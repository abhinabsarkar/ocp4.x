# Key terminologies & definitions

## Machine management in OCP 
[Machine management - RH docs](https://docs.openshift.com/container-platform/4.11/machine_management/index.html)
* Machines - Node in OCP. In Azure, it is VMs.
* MachineSets - Groups of machines. MachineSets are to machines as ReplicaSets are to Pods. If you need more machines or must scale them down, you change the replicas field on the MachineSet to meet your compute need.
* MachineAutoscaler - This resource automatically scales machines in a cloud. You can set the minimum and maximum scaling boundaries for nodes in a specified MachineSet, and the MachineAutoscaler maintains that range of nodes. The MachineAutoscaler object takes effect after a ClusterAutoscaler object exists. Both ClusterAutoscaler and MachineAutoscaler resources are made available by the ClusterAutoscalerOperator.
* ClusterAutoscaler -  In the OpenShift Container Platform implementation, it is integrated with the Machine API by extending the MachineSet API. You can set cluster-wide scaling limits for resources such as cores, nodes, memory, GPU, and so on.

## Build strategies
[Build strategy - RH docs](https://docs.openshift.com/container-platform/4.11/cicd/builds/build-strategies.html)
* Docker build - build a container image from a Dockerfile
* Source-to-image (S2I) - injects application source into a container image and assembling a new image

## Image stream
[Image streams - RH docs](https://docs.openshift.com/container-platform/4.11/openshift_images/image-streams-manage.html)
* An image stream and its associated tags provide an abstraction for referencing container images from within OpenShift Container Platform. Image stream can trigger builds and deployments when a new image is pushed to the registry.

