# CoreOS
Red Hat Enterprise Linux CoreOS (RHCOS) combines the quality standards of Red Hat Enterprise Linux (RHEL) with the automated, remote upgrade features from Container Linux. RHCOS is the only supported operating system for OpenShift Container Platform control plane, or master, machines. While RHCOS is the default operating system for all cluster machines, you can create compute machines, which are also known as worker machines, that use RHEL as their operating system.

![Alt text](/images/rhel-coreos.jpg)

There are two general ways RHCOS is deployed in OpenShift Container Platform 4.4
1. Installer Provisioned Infrastructure (IPI) - If you install your cluster on infrastructure that the cluster provisions, RHCOS images are downloaded to the target platform during installation, and suitable Ignition config files, which control the RHCOS configuration, are used to deploy the machines.

    ![Alt text](/images/ipi.jpg)

2. User Provisioned Infrastructure (UPI) - If you install your cluster on infrastructure that you manage, you must follow the installation documentation to obtain the RHCOS images, generate Ignition config files, and use the Ignition config files to provision your machines.

    ![Alt text](/images/upi.jpg)

Key RHCOS features
The following list describes key features of the RHCOS operating system:

1. Based on RHEL: The underlying operating system consists primarily of RHEL components.
2. Controlled immutability: Management is performed remotely from the OpenShift Container Platform cluster. When you set up your RHCOS machines, you can modify only a few system settings. This controlled immutability allows OpenShift Container Platform to store the latest state of RHCOS systems in the cluster so it is always able to create additional machines and perform updates based on the latest RHCOS configurations.
3. CRI-O container runtime: It incorporates the CRI-O container engine instead of the Docker container engine. CRI-O offers a smaller footprint and reduced attack surface than is possible with container engines that offer a larger feature set.
4. Set of container tools: 
    * The podman CLI tool supports many container runtime features, such as running, starting, stopping, listing, and removing containers and container images.
    * The skopeo CLI tool can copy, authenticate, and sign images.
    * The crictl CLI tool to work with containers and pods from the CRI-O container engine.
5. rpm-ostree upgrades: RHCOS features transactional upgrades using the rpm-ostree system. Updates are delivered by means of container images and are part of the OpenShift Container Platform update process. When deployed, the container image is pulled, extracted, and written to disk, then the bootloader is modified to boot into the new version. The machine will reboot into the update in a rolling manner to ensure cluster capacity is minimally impacted.
6. Updated through MachineConfigOperator: In OpenShift Container Platform, the Machine Config Operator handles operating system upgrades. Instead of upgrading individual packages, as is done with yum upgrades, rpm-ostree delivers upgrades of the OS as an atomic unit. The new OS deployment is staged during upgrades and goes into effect on the next reboot. If something goes wrong with the upgrade, a single rollback and reboot returns the system to the previous state. RHCOS upgrades in OpenShift Container Platform are performed during cluster updates.