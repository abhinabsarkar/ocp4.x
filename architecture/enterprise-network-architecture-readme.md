# OCP 4.x Enterprise Network Architecture
The below diagram shows the enterprise network architecture for OCP4.x on Azure & the different interfacing systems on-premise.

![Alt text](/images/ocp4.x-ipi-network-architecture-azure-enterprise-updated.jpg)

Some of the interfacing systems that OCP4.x is integrated with are hosted on Azure, where as some are hosted on-premise. The component integration is listed below:

### Authentication
By default, only a kubeadmin user exists on OCP cluster. The ldap identity provider is configured to validate user names and passwords against an LDAPv3 server, using simple bind authentication.

### DevOps
The OpenShift Container Platform is configured with the following DevOps toolsets. All of these are hosted in Azure.
* Bitbucket - Source & version control repository
* Jenkins - Jenkins provides building, testing, and deploying, facilitating continuous integration and continuous delivery.
* Nexus - Docker registry
* HashiCorp Vault - HashiCorp Vault is a centralized secrets management middleware. Developers and operators can just talk to an API and Vault takes care of encryption, key management, key rotation, and more. It helps in adopting DevSecOps framework which seeks to remove manual processes and allow operations in a low/zero-trust network. 
> Refer this [link](https://www.hashicorp.com/resources/how-to-get-security-buy-in-from-developers/) for building secure development lifecycle and zero-trust networks

The below diagram shows the workflow of Kubernetes Auth Method for OpenShift platform which is using Vault hosted externally. The details can be found in this [link](https://medium.com/hashicorp-engineering/vault-kubernetes-auth-method-for-openshift-9b9155590a6d) & this [link](https://itnext.io/dynamic-vault-secrets-agent-sidecar-on-kubernetes-cc0ce3e54a94)

![Alt text](/images/vault-k8s-auth-method.jpg)

**Vault Injector** - The Vault Injector is a Mutating Webhook that will inject a Init container and a sidecar container into your pod for secrets management. The Init container will pull the secrets before your application container starts and put it into a shared volume that will be mounted in your app container. The Init container then terminates. The sidecar will run along side your app container and keep the secret fresh in the mounted volume. If the secret is updated in Vault, the sidecar container will update it on the file system in your app container. The Mutating Webhook is triggered by adding annotations to your deployment/pod manifest.

![Alt image](/images/vault-injector.jpg)

> For more details refer this [link](https://itnext.io/dynamic-vault-secrets-agent-sidecar-on-kubernetes-cc0ce3e54a94)

### Monitoring & Alerting
OpenShift Container Platform includes a pre-configured and self-updating monitoring stack that is based on the Prometheus open source project and its wider eco-system. It provides monitoring of cluster components and ships with a set of alerts to immediately notify the cluster administrator about any occurring problems and a set of Grafana dashboards. Refer the below link for details
* [Cluster monitoring](https://docs.openshift.com/container-platform/4.4/monitoring/cluster_monitoring/about-cluster-monitoring.html)

Cluster Monitoring Operator (CMO), which watches over the deployed monitoring components and resources, and ensures that they are always up to date. The Prometheus Operator (PO) creates, configures, and manages Prometheus and Alertmanager instances. It also automatically generates monitoring target configurations based on familiar Kubernetes label queries.

![Alt text](/images/prometheus-cluster-monitoring.jpg)

Configuring most OpenShift framework components, including the Prometheus Cluster Monitoring stack, happens post-installation.

Alertmanager is used to send alerts by configuring it with the following components
* SMTP server - Sends email using the alertmanager to the cluster administrators
* TrueSight - It monitors emails sent to a mailbox by the alertmanager & then create incidents in Service-Now.
* Service-Now - The incidents created in Service-Now are sent to the respective queues of the concerned teams.

### Logging
OpenShift forward logs using the Fluentd forward protocol. The details can be found below:
* [Forward logs to 3rd party system](https://docs.openshift.com/container-platform/4.4/logging/config/cluster-logging-external.html)
* [OCP log forwarding API](https://www.openshift.com/blog/forwarding-logs-to-splunk-using-the-openshift-log-forwarding-api)

![Alt text](/images/fluentd-fwd-logs-splunk.jpg)

The logs are forwarded to the following 

* Splunk -  It is the external log aggregation system for *applications* running on OCP. Splunk has HEC (HTTP Event Collector) endpoint for each instance like On-prem, GCP & Azure (Prod/UAT/Dev). HEC uses a token-based authentication model and send data & application events to Secure HTTP (HTTPS) protocols. For more details please refer Splunk documentation - [link](https://docs.splunk.com/Documentation/Splunk/7.3.0/Data/UsetheHTTPEventCollector). Each instance have different indexer like On-prem, GCP & Azure (Prod/UAT/Dev) and connect with common search head.

    Below is an example of Splunk configuration at OpenShift end.

    Index Name :- 
    app_logs - Event Index - 10GB
    Azure Dev HEC Details:
    HEC end point: https://splunk-host.domain.com
    Port - 8088
    HEC Token: 12sd4b2321-a012-4be7-ac34-f3840902393

* ArcSight - It is a cyber security product that  provides big data security analytics for security information and event management (SIEM), and log management. It helps customers identify and prioritize security threats, organize and track incident response activities, and simplify audit and compliance activities. All the syslogs are forwarded to ArcSight. Refer this link - [Forwarding logs using syslog protocol](https://docs.openshift.com/container-platform/4.3/logging/config/cluster-logging-external.html#cluster-logging-collector-syslog_cluster-logging-external) 

### Backup
The etcd backup is done on the backup server. The Netbackup client is installed on the backup server, which will back up the etcd data from the backup server in cloud to on-premise as per the schedule configured.

### Security
Qualys scanner tool is used for real-time identification of vulnerabilities in the OpenShift Virtual Machines.
