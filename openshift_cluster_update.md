---

copyright:
  years: 2014, 2019
lastupdated: "2019-11-04"

keywords: openshift, roks, rhoks, rhos, version, upgrade

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



# Updating clusters, worker nodes, and cluster components
{: #update}

You can install updates to keep your {{site.data.keyword.openshiftlong}} clusters up-to-date.
{:shortdesc}

## Updating the master
{: #master}

Periodically, {{site.data.keyword.openshiftshort}} releases [major, minor, or patch updates](/docs/openshift?topic=openshift-openshift_versions). Updates can affect the API server version or other components in your master. IBM updates the patch version, but you must update the master major and minor versions.
{:shortdesc}

**How do I know when to update the master?**</br>
You are notified in the {{site.data.keyword.cloud_notm}} console and CLI when updates are available, and can also check the [supported versions](/docs/openshift?topic=openshift-openshift_versions) page.

**Can my worker nodes run a later version than the master?**</br>
Your worker nodes cannot run a later `major.minor` Kubernetes version than the master. First, [update your master](#update_master) to the latest Kubernetes version. Then, [update the worker nodes](#worker_node) in your cluster.

Worker nodes can run later patch versions than the master, such as patch versions that are specific to worker nodes for security updates.

**How are patch updates applied?**</br>
By default, patch updates for the master are applied automatically over the course of several days, so a master patch version might show up as available before it is applied to your master. The update automation also skips clusters that are in an unhealthy state or have operations currently in progress. Occasionally, IBM might disable automatic updates for a specific master fix pack, such as a patch that is only needed if a master is updated from one minor version to another. In any of these cases, you can [check the versions changelog](/docs/openshift?topic=openshift-openshift_changelog) for any potential impact and choose to safely use the `ibmcloud oc cluster master update` [command](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_cluster_update) yourself without waiting for the update automation to apply.

Unlike the master, you must update your workers for each patch version.

**What happens during the master update?**</br>
Your master is highly available with three replica master pods. The master pods have a rolling update, during which only one pod is unavailable at a time. Two instances are up and running so that you can access and change the cluster during the update. Your worker nodes, apps, and resources continue to run.

For clusters that run previous versions of Kubernetes, when you update the Kubernetes API server, the API server is down for about 5 - 10 minutes. During the update, you cannot access or change the cluster. However, worker nodes, apps, and resources that cluster users deployed are not modified and continue to run.

**Can I roll back the update?**</br>
No, you cannot roll back a cluster to a previous version after the update process takes place. Be sure to use a test cluster and follow the instructions to address potential issues before you update your production master.

**What process can I follow to update the master?**</br>
The following diagram shows the process that you can take to update your master.

![Master update best practice](/images/update-tree.png)

Figure 1. Updating Kubernetes master process diagram

{: #update_master}
Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

To update the Kubernetes master _major_ or _minor_ version:

1.  Review the [Kubernetes changes](/docs/containers?topic=containers-cs_versions) and make any updates marked _Update before master_.

2.  Update your API server and associated master components by using the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/login) or running the CLI `ibmcloud oc cluster master update` [command](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_cluster_update).

3.  Wait a few minutes, then confirm that the update is complete. Review the API server version on the {{site.data.keyword.cloud_notm}} clusters dashboard or run `ibmcloud oc cluster ls`.

4.  Install the version of the [`oc cli`](/docs/containers?topic=containers-cs_cli_install#kubectl) that matches the API server version that runs in the master. [Kubernetes does not support ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/setup/release/version-skew-policy/) `oc` client versions that are two or more versions apart from the server version (n +/- 2).

When the master update is complete, you can update your worker nodes

<br />



## Updating worker nodes
{: #worker_node}

You received a notification to update your worker nodes. What does that mean? As security updates and patches are put in place for the API server and other master components, you must be sure that the worker nodes remain in sync.
{: shortdesc}



**What happens to my apps during an update?**</br>
If you run apps as part of a deployment on worker nodes that you update, the apps are rescheduled onto other worker nodes in the cluster. These worker nodes might be in a different worker pool, or if you have stand-alone worker nodes, apps might be scheduled onto stand-alone worker nodes. To avoid downtime for your app, you must ensure that you have enough capacity in the cluster to carry the workload.

**How can I control how many worker nodes go down at a time during the update?**</br>
If you need all your worker nodes to be up and running, consider [resizing your worker pool](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_worker_pool_resize) or [adding stand-alone worker nodes](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_worker_add) to add more worker nodes. You can remove the additional worker nodes after the update is completed.

In addition, you can create a Kubernetes config map that specifies the maximum number of worker nodes that can be unavailable at a time during the update. Worker nodes are identified by the worker node labels. You can use IBM-provided labels or custom labels that you added to the worker node.

**What if I choose not to define a config map?**</br>
When the config map is not defined, the default is used. By default, a maximum of 20% of all of your worker nodes in each cluster can be unavailable during the update process.

### Prerequisites
{: #worker-up-prereqs}

Before you update your worker nodes, review the prerequisite steps.
{: shortdesc}

Updates to worker nodes can cause downtime for your apps and services. Your worker node machine is reimaged, and data is deleted if not [stored outside the pod](/docs/openshift?topic=openshift-storage_planning#persistent_storage_overview).
{: important}

- [Log in to your account. If applicable, target the appropriate resource group. Set the context for your cluster.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)
- [Update the master](#master). The worker node version cannot be higher than the API server version that runs in your Kubernetes master.
- Make any changes that are marked with _Update after master_ in the [Kubernetes clusters](/docs/containers?topic=containers-cs_versions) or [OpenShift clusters](/docs/openshift?topic=openshift-openshift_versions) version preparation guides.
- If you want to apply a patch update, review the [Kubernetes clusters](/docs/containers?topic=containers-changelog) or [OpenShift clusters](/docs/openshift?topic=openshift-openshift_versions) version changelog.
- Consider [adding more worker nodes](/docs/openshift?topic=openshift-add_workers) so that your cluster has enough capacity to rescheduling your workloads during the update.
- Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

### Updating worker nodes in the CLI with a configmap
{: #worker-up-configmap}

Set up a configmap to perform a rolling update of your worker nodes.
{: shortdesc}

1.  Complete the [prerequisite steps](#worker-up-prereqs).
2.  List available worker nodes and note their private IP address.

    ```
    ibmcloud oc worker ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

3. View the labels of a worker node. You can find the worker node labels in the **Labels** section of your CLI output. Every label consists of a `NodeSelectorKey` and a `NodeSelectorValue`.
   ```
   oc describe node <private_worker_IP>
   ```
   {: pre}

   Example output:
   ```
   Name:               10.184.58.3
   Roles:              <none>
   Labels:             arch=amd64
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/region=us-south
                    failure-domain.beta.kubernetes.io/zone=dal12
                    ibm-cloud.kubernetes.io/encrypted-docker-data=true
                    ibm-cloud.kubernetes.io/iaas-provider=softlayer
                    ibm-cloud.kubernetes.io/machine-type=u3c.2x4.encrypted
                    kubernetes.io/hostname=10.123.45.3
                    privateVLAN=2299001
                    publicVLAN=2299012
   Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
   CreationTimestamp:  Tue, 03 Apr 2018 15:26:17 -0400
   Taints:             <none>
   Unschedulable:      false
   ```
   {: screen}

4. Create a config map and define the unavailability rules for your worker nodes. The following example shows four checks, the `zonecheck.json`, `regioncheck.json`, `defaultcheck.json`, and a check template. You can use these example checks to define rules for worker nodes in a specific zone (`zonecheck.json`), region (`regioncheck.json`), or for all worker nodes that do not match any of the checks that you defined in the config map (`defaultcheck.json`). Use the check template to create your own check. For every check, to identify a worker node, you must choose one of the worker node labels that you retrieved in the previous step.  

   For every check, you can set only one value for <code>NodeSelectorKey</code> and <code>NodeSelectorValue</code>. If you want to set rules for more than one region, zone, or other worker node labels, create a new check. Define up to 10 checks in a config map. If you add more checks, they are ignored.
   {: note}

   Example:
   ```
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: ibm-cluster-update-configuration
     namespace: kube-system
   data:
    drain_timeout_seconds: "120"
    zonecheck.json: |
      {
        "MaxUnavailablePercentage": 30,
        "NodeSelectorKey": "failure-domain.beta.kubernetes.io/zone",
        "NodeSelectorValue": "dal13"
      }
    regioncheck.json: |
      {
        "MaxUnavailablePercentage": 20,
        "NodeSelectorKey": "failure-domain.beta.kubernetes.io/region",
        "NodeSelectorValue": "us-south"
      }
    defaultcheck.json: |
      {
        "MaxUnavailablePercentage": 20
      }
    <check_name>: |
      {
        "MaxUnavailablePercentage": <value_in_percentage>,
        "NodeSelectorKey": "<node_selector_key>",
        "NodeSelectorValue": "<node_selector_value>"
      }
   ```
   {: codeblock}

   <table summary="The first row in the table spans both columns. The rest of the rows should be read left to right, with the parameter in column one and the description that matches in column two.">
   <caption>ConfigMap components</caption>
    <thead>
      <th colspan=2><img src="images/idea.png" alt="Idea icon"/> Understanding the components </th>
    </thead>
    <tbody>
      <tr>
        <td><code>drain_timeout_seconds</code></td>
        <td> Optional: The timeout in seconds to wait for the [drain ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) to complete. Draining a worker node safely removes all existing pods from the worker node and reschedules the pods onto other worker nodes in the cluster. Accepted values are integers in the range 1 - 180. The default value is 30.</td>
      </tr>
      <tr>
        <td><code>zonecheck.json</code></br><code>regioncheck.json</code></td>
        <td>Two checks that define a rule for a set of worker nodes that you can identify with the specified <code>NodeSelectorKey</code> and <code>NodeSelectorValue</code>. The <code>zonecheck.json</code> identifies worker nodes based on their zone label, and the <code>regioncheck.json</code> uses the region label that is added to every worker node during provisioning. In the example, 30% of all worker nodes that have <code>dal13</code> as their zone label and 20% of all the worker nodes in <code>us-south</code> can be unavailable during the update.</td>
      </tr>
      <tr>
        <td><code>defaultcheck.json</code></td>
        <td>If you do not create a config map or the map is configured incorrectly, the Kubernetes default is applied. By default, only 20% of the worker nodes in the cluster can be unavailable at a time. You can override the default value by adding the default check to your config map. In the example, every worker node that is not specified in the zone and region checks (<code>dal13</code> or <code>us-south</code>) can be unavailable during the update. </td>
      </tr>
      <tr>
        <td><code>MaxUnavailablePercentage</code></td>
        <td>The maximum number of nodes that are allowed to be unavailable for a specified label key and value, which is specified as a percentage. A worker node is unavailable during the deploying, reloading, or provisioning process. The queued worker nodes are blocked from updating if it exceeds any defined maximum unavailable percentages. </td>
      </tr>
      <tr>
        <td><code>NodeSelectorKey</code></td>
        <td>The label key of the worker node for which you want to set a rule. You can set rules for the default labels that are provided by IBM, as well as on worker node labels that you created. <ul><li>If you want to add a rule for worker nodes that belong to one worker pool, you can use the <code>ibm-cloud.kubernetes.io/machine-type</code> label. </li><li> If you have more than one worker pool with the same machine type, use a custom label. </li></ul></td>
      </tr>
      <tr>
        <td><code>NodeSelectorValue</code></td>
        <td>The label value that the worker node must have to be considered for the rule that you define. </td>
      </tr>
    </tbody>
   </table>

5. Create the configuration map in your cluster.
   ```
   oc apply -f <filepath/configmap.yaml>
   ```
   {: pre}

6.  Verify that the config map is created.
    ```
    oc get configmap --namespace kube-system
    ```
    {: pre}

7.  Update the worker nodes.

    ```
    ibmcloud oc worker update --cluster <cluster_name_or_ID> --worker <worker_node1_ID> --worker <worker_node2_ID>
    ```
    {: pre}

8. Optional: Verify the events that are triggered by the config map and any validation errors that occur. The events can be reviewed in the  **Events** section of your CLI output.
   ```
   oc describe -n kube-system cm ibm-cluster-update-configuration
   ```
   {: pre}

9. Confirm that the update is complete by reviewing the Kubernetes version of your worker nodes.  
   ```
   oc get nodes
   ```
   {: pre}

10. Verify that you do not have duplicate worker nodes. In some cases, older clusters might list duplicate worker nodes with a **`NotReady`** status after an update. To remove duplicates, see [troubleshooting](/docs/containers?topic=containers-cs_troubleshoot_clusters#cs_duplicate_nodes).

Next steps:
-   Repeat the update process with other worker pools.
-   Inform developers who work in the cluster to update their `oc` CLI to the version of the Kubernetes master.
-   If the Kubernetes dashboard does not display utilization graphs, [delete the `kube-dashboard` pod](/docs/containers?topic=containers-cs_troubleshoot_health#cs_dashboard_graphs).

### Updating worker nodes in the console
{: #worker_up_console}

After you set up the config map for the first time, you can then update worker nodes by using the {{site.data.keyword.cloud_notm}} console.
{: shortdesc}

To update worker nodes from the console:
1.  Complete the [prerequisite steps](#worker-up-prereqs) and [set up a config map](#worker_node) to control how your worker nodes are updated.
2.  From the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/) menu ![Menu icon](../icons/icon_hamburger.svg "Menu icon"), click **OpenShift**.
3.  From the **Clusters** page, click your cluster.
4.  From the **Worker Nodes** tab, select the check box for each worker node that you want to update. An action bar is displayed over the table header row.
5.  From the action bar, click **Update**.

<br />



## Updating flavors (machine types)
{: #machine_type}

You can update the flavors, or machine types, of your worker nodes by adding new worker nodes and removing the old ones. For example, if your cluster has deprecated `x1c` or older Ubuntu 16 `x2c` worker node flavors, create Ubuntu 18 worker nodes that use flavors with `x3c` in the names.
{: shortdesc}

Before you begin:
- [Log in to your account. If applicable, target the appropriate resource group. Set the context for your cluster.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)
- If you store data on your worker node, the data is deleted if not [stored outside the worker node](/docs/openshift?topic=openshift-storage_planning#persistent_storage_overview).
- Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

To update flavors:

1. List available worker nodes and note their private IP address.
   - **For worker nodes in a worker pool**:
     1. List available worker pools in your cluster.
        ```
        ibmcloud oc worker-pool ls --cluster <cluster_name_or_ID>
        ```
        {: pre}

     2. List the worker nodes in the worker pool. Note the **ID** and **Private IP**.
        ```
        ibmcloud oc worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
        ```
        {: pre}

     3. Get the details for a worker node and note the zone, the private and the public VLAN ID.
        ```
        ibmcloud oc worker get --cluster <cluster_name_or_ID> --worker <worker_ID>
        ```
        {: pre}

   - **Deprecated: For stand-alone worker nodes**:
     1. List available worker nodes. Note the **ID** and **Private IP**.
        ```
        ibmcloud oc worker ls --cluster <cluster_name_or_ID>
        ```
        {: pre}

     2. Get the details for a worker node and note the zone, the private VLAN ID, and the public VLAN ID.
        ```
        ibmcloud oc worker get --cluster <cluster_name_or_ID> --worker <worker_ID>
        ```
        {: pre}

2. List available flavors in the zone.
   ```
   ibmcloud oc flavors --zone <zone>
   ```
   {: pre}

3. Create a worker node with the new machine type.
   - **For worker nodes in a worker pool**:
     1. Create a worker pool with the number of worker nodes that you want to replace.
        * Classic clusters:
          ```
          ibmcloud oc worker-pool create classic --name <pool_name> --cluster <cluster_name_or_ID> --machine-type <flavor> --size-per-zone <number_of_workers_per_zone>
          ```
          {: pre}
        * VPC clusters:
          ```
          ibmcloud oc worker-pool create vpc-classic <pool_name> --cluster <cluster_name_or_ID> --flavor <flavor> --size-per-zone <number_of_workers_per_zone> --vpc-id <VPC_ID>
          ```
          {: pre}

     2. Verify that the worker pool is created.
        ```
        ibmcloud oc worker-pool ls --cluster <cluster_name_or_ID>
        ```
        {: pre}

     3. Add the zone to your worker pool that you retrieved earlier. When you add a zone, the worker nodes that are defined in your worker pool are provisioned in the zone and considered for future workload scheduling. If you want to spread your worker nodes across multiple zones, choose a [multizone-capable zone](/docs/containers?topic=containers-regions-and-zones#zones).
       * Classic clusters:
         ```
         ibmcloud oc zone add classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
         ```
         {: pre}
       * VPC clusters:
         ```
         ibmcloud oc zone add vpc-classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --subnet-id <vpc_subnet_id>
         ```
         {: pre}

   - **Deprecated: For stand-alone worker nodes**:
       ```
       ibmcloud oc worker add --cluster <cluster_name> --machine-type <flavor> --workers <number_of_worker_nodes> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
       ```
       {: pre}

4. Wait for the worker nodes to be deployed. When the worker node state changes to **Normal**, the deployment is finished.
   ```
   ibmcloud oc worker ls --cluster <cluster_name_or_ID>
   ```
   {: pre}
5.  To prevent downtime, reschedule the apps from the old worker nodes before you delete the old worker nodes.
    1.  Mark the worker node as unschedulable in a process that is known as cordoning. When you cordon a worker node, you make it unavailable for future pod scheduling. Use the **Private IP** of the worker node that you retrieved earlier, which is the worker node name in Kubernetes.
        ```
        oc cordon <private_IP_address_of_worker_node>
        ```
        {: pre}
    2.  Verify that pod scheduling is disabled for the worker node by checking that the status is **SchedulingDisabled**.
        ```
        oc get nodes
        ```
        {: pre}
    3.  Force pods to be removed from your worker node and rescheduled onto remaining worker nodes in the cluster. This process can take a few minutes.
        ```
        oc drain <worker_name>
        ```
        {: pre}
6. Remove the old worker node. **Note**: If you are removing a flavor that is billed monthly (such as bare metal), you are charged for the entire the month.
   - **For worker nodes in a worker pool**:
     1. Remove the worker pool with the old machine type. Removing a worker pool removes all worker nodes in the pool in all zones. This process might take a few minutes to complete.
        ```
        ibmcloud oc worker-pool rm --worker-pool <pool_name> --cluster <cluster_name_or_ID>
        ```
        {: pre}

     2. Verify that the worker pool is removed.
        ```
        ibmcloud oc worker-pool ls --cluster <cluster_name_or_ID>
        ```
        {: pre}

   - **Deprecated: For stand-alone worker nodes**:
      ```
      ibmcloud oc worker rm --cluster <cluster_name> --worker <worker_node>
      ```
      {: pre}

7. Verify that the worker nodes are removed from your cluster.
   ```
   ibmcloud oc worker ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

8. Repeat these steps to update other worker pools or stand-alone worker nodes to different flavors.

## Updating cluster components
{: #components}

Your Red Hat OpenShift on IBM Cloud cluster comes with components, such as Ingress, that are installed automatically when you provision the cluster. By default, these components are updated automatically by IBM. However, you can disable automatic updates for some components and manually update them separately from the master and worker nodes.
{: shortdesc}

**What default components can I update separately from the cluster?**</br>
You can optionally disable automatic updates for the following components:
* [Fluentd for logging](#logging-up)
* [Ingress application load balancer (ALB)](#alb)

**Are there components that I can't update separately from the cluster?**</br>

Yes. Your cluster is deployed with the following managed components and associated resources that cannot be changed, except to scale pods or edit configmaps for certain performance benefits. If you try to change one of these deployment components, their original settings are restored on a regular interval when they are updated with the cluster master. However, note that resources that you create that are associated with these components, such as Calico network policies that you create to be implemented by the Calico deployment components, are not updated.

* `calico`
* `coredns`
* `coredns-autoscaler`
* `heapster`
* `ibm-file-plugin`
* `ibm-storage-watcher`
* `ibm-keepalived-watcher`
* `kube-dns-amd64`
* `kube-dns-autoscaler`
* `kubernetes-dashboard`
* `metrics-server`
* `vpn`

**Can I install other plug-ins or add-ons than the default components?**</br>
Yes. Red Hat OpenShift on IBM Cloud provides other plugin-ins and add-ons that you can choose from to add capabilities to your cluster. For example, you might want to [use Helm charts](/docs/containers?topic=containers-helm#public_helm_install) to install the [block storage plug-in](/docs/openshift?topic=openshift-block_storage#install_block) or [strongSwan VPN](/docs/openshift?topic=openshift-vpn#vpn-setup). You must update these Helm charts separately by following the instructions in the Helm chart readmes.

### Managing automatic updates for Fluentd
{: #logging-up}

In order to change your logging or filter configurations, the Fluentd component must be at the latest version. By default, automatic updates to the component are enabled.
{: shortdesc}

You can manage automatic updates of the Fluentd component in the following ways. **Note**: To run the following commands, you must have the [**Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform) for the cluster.

* Check whether automatic updates are enabled by running the `ibmcloud oc logging autoupdate get --cluster <cluster_name_or_ID>` [command](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_log_autoupdate_get).
* Disable automatic updates by running the `ibmcloud oc logging autoupdate disable` [command](/docs/openshift?topic=openshift-kubernetes-service-cli#cs_log_autoupdate_disable).
* If automatic updates are disabled, but you need to change your configuration, you have two options:
    * Turn on automatic updates for your Fluentd pods.
        ```
        ibmcloud oc logging autoupdate enable --cluster <cluster_name_or_ID>
        ```
        {: pre}
    * Force a one-time update when you use a logging command that includes the `--force-update` option. **Note**: Your pods update to the latest version of the Fluentd component, but Fluentd does not update automatically going forward.
        Example command:

        ```
        ibmcloud oc logging config update --cluster <cluster_name_or_ID> --id <log_config_ID> --type <log_type> --force-update
        ```
        {: pre}

### Managing automatic updates for Ingress ALBs
{: #alb}

Control when the Ingress application load balancer (ALB) component is updated.
{: shortdesc}

When the Ingress ALB component is updated, the `nginx-ingress` and `ingress-auth` containers in all ALB pods are updated to the latest build version. By default, automatic updates to ALBs are enabled. Updates are performed on a rolling basis so that your Ingress ALBs don't experience any downtime. When a pod restarts after the update is applied, a [readiness check](/docs/openshift?topic=openshift-ingress-settings#readiness-check) prevents the ALB pod from attempting to route traffic requests until all of the Ingress resource files are parsed. This readiness check prevents request loss during ALB pod updates and can take up to 5 minutes.

If you disable automatic updates, you are responsible for updating your ALBs. As updates become available, you are notified in the CLI when you run the `ibmcloud oc alb ls` or `alb autoupdate get` commands.

When you update the major or minor Kubernetes version of your cluster, IBM automatically makes necessary changes to the Ingress deployment, but does not change the build version of your Ingress ALBs. You are responsible for checking the compatibility of the latest Kubernetes versions and your Ingress ALBs' images.
{: note}

Before you begin:

1. Verify that your ALBs are running.
    ```
    ibmcloud oc alb ls
    ```
    {: pre}

2. Check the status of automatic updates for the Ingress ALB component.
    ```
    ibmcloud oc alb autoupdate get --cluster <cluster_name_or_ID>
    ```
    {: pre}

    Example output when automatic updates are enabled:
    ```
    Retrieving automatic update status of application load balancer (ALB) pods in cluster mycluster...
    OK
    Automatic updates of the ALB pods are enabled in cluster mycluster
    ALBs are at the latest version in cluster mycluster
    ```
    {: screen}

    Example output when automatic updates are disabled:
    ```
    Retrieving automatic update status of application load balancer (ALB) pods in cluster mycluster...
    OK
    Automatic updates of the ALB pods are disabled in cluster mycluster
    ALBs are not at the latest version in cluster mycluster. To view the current version, run 'ibmcloud oc alb ls'.
    ```
    {: screen}

3. Verify the current **Build** version of your ALB pods.
    ```
    ibmcloud oc alb ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

    Example output:
    ```
    ALB ID                                            Enabled   Status     Type      ALB IP          Zone    Build                           ALB VLAN ID   NLB Version
    private-crdf253b6025d64944ab99ed63bb4567b6-alb2   false     disabled   private   10.xxx.xx.xxx   dal10   ingress:411/ingress-auth:315*   2294021       -
    public-crdf253b6025d64944ab99ed63bb4567b6-alb2    true      enabled    public    169.xx.xxx.xxx  dal10   ingress:411/ingress-auth:315*   2234945       -

    * An update is available for the ALB pods. Review any potentially disruptive changes for the latest version before you update: https://cloud.ibm.com/docs/containers?topic=containers-update#alb
    ```
    {: screen}

You can manage automatic updates of the Ingress ALB component in the following ways. **Note**: To run the following commands, you must have the [**Editor** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform) for the cluster.
* Disable automatic updates.
    ```
    ibmcloud oc alb autoupdate disable --cluster <cluster_name_or_ID>
    ```
    {: pre}
* Manually update your Ingress ALBs.
    1. If an update is available and you want to update your ALBs, first check the [changelog for the latest version of the Ingress ALB component](/docs/containers?topic=containers-cluster-add-ons-changelog#alb_changelog) to verify any potentially disruptive changes.
    2. Force a one-time update of your ALB pods. All ALB pods in the cluster are updated to the latest build version. You cannot update an individual ALB or choose which build to update ALBs to. Automatic updates remain disabled.
        ```
        ibmcloud oc alb update --cluster <cluster_name_or_ID>
        ```
        {: pre}
* If your ALB pods were recently updated, but a custom configuration for your ALBs is affected by the latest build, you can roll back the update to the build that your ALB pods were previously running. **Note**: After you roll back an update, automatic updates for ALB pods are disabled.
    ```
    ibmcloud oc alb rollback --cluster <cluster_name_or_ID>
    ```
    {: pre}
* Re-enable automatic updates. Whenever the next build becomes available, the ALB pods are automatically updated to the latest build.
    ```
    ibmcloud oc alb autoupdate enable --cluster <cluster_name_or_ID>
    ```
    {: pre}

<br />





