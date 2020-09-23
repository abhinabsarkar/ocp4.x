# OpenShift Networking 4.x

OpenShift Networking features two main components: 
1. **OpenShift Software Defined Network plug-in** - it handles the communication within the cluster (**default is Open vSwitch (OVS)**)
2. **OpenShift Router plug-in** - it handles the inbound and outbound traffic that is destined to services in the cluster **(default is HAProxy)**

![Alt text](/images/network-overview.jpg)

> The cluster administrator can choose to deploy the cluster by using a third-party SDN from the supported system.

## OpenShift cluster network
OpenShift container Platform uses a software-defined networking approach to provide a unified cluster network that assigns an internal IP address to each pod in the cluster to ensure that all containers within the pod behave as though they were on the same host. 

This pod network is established and maintained by the OpenShift SDN, which configures an overlay network by using Open vSwitch (OVS).

> OVS in OpenShift is the bridge that overlays on top of nodes. All pods (including the router pods) are connected to this bridge.

In OCP 4.x, Cluster Network is the overlay network which OpenShift configures on top of the underlay network. This overlay network provides one logical network for all the pods so that they can talk to each other by their IP address directly even though they may be hosted on different nodes in different network segments.
* clusterNetwork - A list specifying the blocks of IP addresses from which Pod IPs are allocated and the subnet prefix length assigned to each individual node.
* serviceNetwork - A block of IP addresses for services. The OpenShift SDN Container Network Interface (CNI) network provider supports only a single IP address block for the service network.
* machineNetwork - The IP address pools for machines. Refer [link](https://docs.openshift.com/container-platform/4.5/installing/installing_azure/installing-azure-network-customizations.html)

```yaml
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
```

In the default configuration shown above, the cluster network is the 10.128.0.0/14 network (i.e. 10.128.0.0 - 10.131.255.255), and nodes are allocated /23 subnets (i.e., 10.128.0.0/23, 10.128.2.0/23, 10.128.4.0/23 and so on to 10.131.254.0/23). This means that the cluster network has 512 subnets available to assign to nodes, and a given node is allocated 510 addresses that it can assign to the containers running on it. The size and address range of the cluster network are configurable, as is the host subnet size.

Refer the [cidr calculator](https://www.ipaddressguide.com/cidr)

```bash
# cluster network subnet (cidr - 10.128.0.0/23) that is allocated to each node
oc get hostsubnet
NAME       HOST       HOST IP         SUBNET          EGRESS CIDRS   EGRESS IPS
master-0   master-0   10.189.250.8    10.129.0.0/23
master-1   master-1   10.189.250.6    10.130.0.0/23
master-2   master-2   10.189.250.5    10.128.0.0/23
worker-1   worker-1   10.189.251.5    10.128.2.0/23
worker-2   worker-2   10.189.251.6    10.131.0.0/23
worker-3   worker-3   10.189.251.8    10.131.2.0/23
```
> In the above example, the machineNetwork cidr wasn't configured, hence the *HOST IP* address is outside the machineNetwrok cidr mentioned in the yaml above 

*Master nodes do not have access to pods via the cluster network, unless it is explicitly enabled. This is a security feature. The control plane is decoupled from the data plane*

## OpenShift SDN plug-ins
OpenShift provides the following SDN plugins for configuring the network:
1. subnet plug-in - it provides a flat pod network in which every pod can communicate with every other pod and service. This configuration is the default configuration for single-tenant clusters.
2. multitenant plug-in
    * Provides project-level isolation for pods and services
    * Each project receives a unique Virtual Network ID (VNID)
    * Pods can only communicate to Pods that share this VNID
    * Projects that receive VNID 0 can communicate with all other pods, and vice-versa
    * The default project has VNID 0. Services like load balancer have VNID 0, to communicate with all other pods in the cluster and vice versa.
 3. networkpolicy plug-in - allows  project administrators to configure their own isolation policies by using NetworkPolicy objects.

OpenShift SDN maintains a registry of all nodes in the cluster. This registry is stored in etcd. When the system administrator registers a node, OpenShift SDN allocates an unused /23 subnet from the cluster network and stores this subnet in the registry. When a node is removed or deleted from the cluster, the OpenShift SDN frees the corresponding cluster network subnet. This subnet becomes available for future allocations to new nodes.

![Alt text](/images/etcd-node-registry.jpg)

## OpenShift SDN network devices
When a node is added to the cluster, the OpenShift SDN registers it with the registry on master to allocate an open subnet to the node. Then, OpenShift SDN creates and configures the following network devices:
1. br0: Open Virtual Switch (OVS) bridge device to which pod containers are attached.
2. tun0: OVS internal port (port 2 on br0). The tun0 interfaces owns the default gateway and is forwarding traffic to external endpoints outside the OpenShift platform or routing internal traffic to the openvswitch overlay. OpenShift SDN configures netfilter and routing rules to enable access from the cluster subnet to the external network by way of NAT.
3. vxlan_sys_4789: OVS VXLAN device (port 1 on br0), which provides access to containers on remote nodes. It is referred to as vxlan0 in the OVS rules.

## OpenShift network traffic flow
Each time a pod is started on the host, OpenShift SDN:
1. Assigns the pod a free IP address from the node’s cluster subnet (say 10.128.0.0/23).
2. Attaches the host side of the pod’s veth interface pair to the OVS bridge br0.
3. Adds OpenFlow rules to the OVS database to route traffic addressed to the new pod to the correct OVS port.
4. In the case of the ovs-multitenant plug-in, adds OpenFlow rules to tag traffic coming from the pod with the pod’s VNID, and to allow traffic into the pod if the traffic’s VNID matches the pod’s VNID (or is the privileged VNID 0). Non-matching traffic is filtered out by a generic rule.

OpenShift SDN nodes also watch for subnet updates from the SDN master. When a new subnet is added, the node adds OpenFlow rules on br0 so that packets with a destination IP address in the remote subnet go to vxlan0 (port 1 on br0) and thus out onto the network. The ovs-subnet plug-in sends all packets across the VXLAN with VNID 0, but the ovs-multitenant plug-in uses the appropriate VNID for the source container.

![Alt text](/images/network-devices-node.jpg)

### Intra node Pod traffic
If container A and container B are on the same node, then the flow of packets from container A to container B is as follows:

eth0 (in A’s netns) → vethA → br0 → vethB → eth0 (in B’s netns)

![Alt text](/images/intra-node-pod-traffic.jpg)

### Inter node Pod traffic
If container A and container B are on the different nodes, then the flow of packets from container A to container B is as follows:

eth0 (in A’s netns) → vethA → br0 → vxlan0 → network → vxlan0 → br0 → vethB → eth0 (in B’s netns)

![Alt text](/images/inter-node-pod-traffic.jpg)

### Pod to external (egress) traffic
If container A connects to an external host, the traffic looks like:

eth0 (in A’s netns) → vethA → br0 → tun0 → (NAT) → eth0 (physical device) → Internet

![Alt text](/images/pod-to-external-traffic.jpg)

> Almost all packet delivery decisions are performed with OpenFlow rules in the OVS bridge br0, which simplifies the plug-in network architecture and provides flexible routing. In the case of the ovs-multitenant plug-in, this also provides enforceable [network isolation](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/sdn.html#network-isolation-multitenant).



## References
* [Networking in OpenShift - Best explained](https://www.redhat.com/files/summit/session-assets/2018/Network-security-for-apps-on-OpenShift.pdf)
* [OpenShift & Network Security Zones](https://www.openshift.com/blog/openshift-and-network-security-zones-coexistence-approaches)
* [Networking in Kubernetes explained](https://neuvector.com/network-security/advanced-kubernetes-networking/)
* [Understanding kubernetes networking](https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727)
* [Cluster network operator - OCP 4.x](https://docs.openshift.com/container-platform/4.5/networking/cluster-network-operator.html#nw-operator-cr_cluster-network-operator)
* [OpenShift SDN 3.x](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/sdn.html)
* [cidr calculator](https://www.ipaddressguide.com/cidr)
* [Machine Network](https://docs.openshift.com/container-platform/4.5/installing/installing_azure/installing-azure-network-customizations.html)

### Understanding docker0 bridge
1. [Networking using docker0 bridge](https://developer.ibm.com/recipes/tutorials/networking-your-docker-containers-using-docker0-bridge/)
2. [Container networking - video](https://www.youtube.com/watch?v=6v_BDHIgOY8&feature=youtu.be)
3. [Kubernetes networking](https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727)
4. [Network namespaces - video](https://www.youtube.com/watch?v=j_UUnlVC2Ss)