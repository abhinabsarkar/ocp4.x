# OCP 4.x Enterprise Network Architecture
The below diagram shows the enterprise network architecture for OCP4.x on Azure & the different interfacing systems on-premise.

![Alt text](/images/ocp4.x-ipi-network-architecture-azure-enterprise.jpg)

Some of the interfacing systems that OCP4.x is integrated with are hosted on Azure, where as some are hosted on-premise. The component integration is listed below:

### Authentication
By default, only a kubeadmin user exists on OCP cluster. The ldap identity provider is configured to validate user names and passwords against an LDAPv3 server, using simple bind authentication.

### DevOps
The OpenShift Container Platform is configured with the following DevOps toolsets. All of these are hosted in Azure.
* Bitbucket - Source & version control repository
* Jenkins - Jenkins provides building, testing, and deploying, facilitating continuous integration and continuous delivery.
* Nexus - Docker registry

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
