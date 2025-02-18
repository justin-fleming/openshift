---

copyright:
  years: 2014, 2019
lastupdated: "2019-10-07"

keywords: openshift, roks, rhoks, rhos

subcollection: openshift

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"} 
{:codeblock: .codeblock}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:download: .download}
{:preview: .preview}

# Service limitations
{: #openshift_limitations}

Review the following limitations for {{site.data.keyword.openshiftlong}}. For general product limitations, such as API calls or number of pods per worker node, see [{{site.data.keyword.containerlong_notm}} service limitations](/docs/containers?topic=containers-ibm-cloud-kubernetes-service-technology#tech_limits).
{: shortdesc}

## Cluster
{: #oc_limits_cluster}

*   You can create only standard clusters, not free clusters. Instead, you can create a free Kubernetes cluster, and then re-deploy the apps that you try out in the Kubernetes cluster to your OpenShift cluster.
*   [Locations](/docs/openshift?topic=openshift-regions-and-zones) are available in all six worldwide multizone metro and select single zone regions.
*   You cannot create a cluster with worker nodes that run multiple operating systems, such as OpenShift on Red Hat Enterprise Linux and community Kubernetes on Ubuntu.


## Networking
{: #oc_limits_networking}

Private network load balancers (NLBs) cannot be registered with the domain name server (DNS), so the cluster cannot be created with only a private network interface. At least some of the cluster's worker nodes must have both public and private VLANs. You can still create a private service to expose your apps on only the private network.

## Storage
{: #oc_limits_storage}

Because of the way that {{site.data.keyword.cloud_notm}} NFS file storage configures Linux user permissions, you might encounter errors when you use file storage. If so, you might need to configure [OpenShift Security Context Constraints ![External link icon](../icons/launch-glyph.svg "External link icon")](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_scc.html) or use a different storage type.

## Logging and Metrics 
{: #oc_limits_metrics}

**Logging**: You cannot run the Ansible playbook to deploy the [OpenShift Container Platform Elasticsearch, Fluentd, and Kibana (EFK) stack ![External link icon](../icons/launch-glyph.svg "External link icon")](https://docs.openshift.com/container-platform/3.11/install_config/aggregate_logging.html) because you cannot modify the default configuration of the Red Hat OpenShift on IBM Cloud cluster.

**Metrics**: The [built-in Prometheus](/docs/openshift?topic=openshift-openshift_apps#openshift_access_oc_services) alert manager includes two rules that display as active alerts in a `FIRING` state: `KubeControllerManagerDown` and `KubeSchedulerDown`. These components are part of the IBM-managed cluster master, so you can ignore these alerts.

Example alert:
```
alert: KubeControllerManagerDown
expr: absent(up{job="kube-controllers"}
  == 1)
for: 15m
labels:
  severity: critical
annotations:
  message: KubeControllerManager has disappeared from Prometheus target discovery.
```
{: screen}
