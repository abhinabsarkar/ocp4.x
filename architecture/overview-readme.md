# OpenShift 4.x overview

OpenShift Container Platform uses Red Hat Enterprise Linux CoreOS (RHCOS), a container-oriented operating system that combines some of the best features and functions of the CoreOS and Red Hat Atomic Host operating systems. RHCOS is specifically designed for running containerized applications from OpenShift Container Platform and works with new tools to provide fast installation, Operator-based management, and simplified upgrades.

RHCOS includes:
1. Ignition, which OpenShift Container Platform uses as a firstboot system configuration for initially bringing up and configuring machines.
2. CRI-O, fully replaces the Docker Container Engine, which was used in OpenShift Container Platform 3.x. It is a Kubernetes native container runtime implementation that integrates closely with the operating system to deliver an efficient and optimized Kubernetes experience. CRI-O provides facilities for running, stopping, and restarting containers.
3. Kubelet, the primary node agent for Kubernetes that is responsible for launching and monitoring containers.

In OpenShift Container Platform 4.4, you must use RHCOS for all control plane machines, but you can use Red Hat Enterprise Linux (RHEL) as the operating system for compute machines, which are also known as worker machines. If you choose to use RHEL workers, you must perform more system maintenance than if you use RHCOS for all of the cluster machines.

## Simplified installation & update process
For clusters that use RHCOS for all machines, updating, or upgrading, OpenShift Container Platform is a simple, highly-automated process. Because OpenShift Container Platform completely controls the systems and services that run on each machine, including the operating system itself, from a central control plane, upgrades are designed to become automatic events. If your cluster contains RHEL worker machines, the control plane benefits from the streamlined update process, but you must perform more tasks to upgrade the RHEL machines.

## OpenShift Container Platform lifecycle
The following figure illustrates the basic OpenShift Container Platform lifecycle:
* Creating an OpenShift Container Platform cluster
* Managing the cluster
* Developing and deploying applications
* Scaling up applications

![Alt text](/images/ocp-overview.jpg)