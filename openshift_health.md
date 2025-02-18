---

copyright:
  years: 2014, 2019
lastupdated: "2019-10-18"

keywords: oks, iro, openshift, red hat, red hat openshift, rhos, roks, rhoks

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
{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}

# Logging and monitoring
{: #openshift_health}

For cluster metrics and app logging and monitoring, {{site.data.keyword.openshiftlong}} clusters include built-in tools to help you manage the health of your single cluster instance. You can also set up {{site.data.keyword.cloud_notm}} tools for multi-cluster analysis or other use cases, such as {{site.data.keyword.containerlong_notm}} add-ons: {{site.data.keyword.la_full_notm}} and {{site.data.keyword.mon_full_notm}}.
{: shortdesc}

## Understanding options for logging and monitoring
{: #oc_logmet_options}

To help understand when to use the built-in OpenShift tools or {{site.data.keyword.cloud_notm}} integrations, review the following table.

<table summary="The rows are read left to right, with the type of solution in column one, the description of the built-in OpenShift tooling in column two, and the description of the {{site.data.keyword.cloud_notm}} integration in column three.">
    <caption>Comparing OpenShift and {{site.data.keyword.cloud_notm}} logging and monitoring solutions</caption>
    <thead>
      <th>Type</th>
      <th>OpenShift tools</th>
      <th>{{site.data.keyword.cloud_notm}} integrations</th>
    </thead>
    <tbody>
    <tr>
        <td>**Logging**</td>
        <td>**Built-in OpenShift logging tools**:<ul>
          <li>Built-in view of pod logs in the OpenShift web console.</li>
          <li>Built-in pod logs are not configured with persistent storage. You must integrate with a cloud database to back up the logging data and make it highly available, and manage the logs yourself.</li></ul>
          <p class="note">You cannot run the Ansible playbook to deploy the [OpenShift Container Platform Elasticsearch, Fluentd, and Kibana (EFK) stack ![External link icon](../icons/launch-glyph.svg "External link icon")](https://docs.openshift.com/container-platform/3.11/install_config/aggregate_logging.html) because you cannot modify the default configuration of the Red Hat OpenShift on IBM Cloud cluster.</p></td>
        <td>**{{site.data.keyword.la_full_notm}}**:<ul>
          <li>Customizable user interface for live streaming of log tailing, real-time troubleshooting, issue alerts, and log archiving.</li>
          <li>Quick integration with the cluster via a script.</li>
          <li>Aggregated logs across clusters and cloud providers.</li>
          <li>Historical access to logs that is based on the plan you choose.</li>
          <li>Highly available, scalable, and compliant with industry security standards.</li>
          <li>Integrated with {{site.data.keyword.cloud_notm}} IAM for user access management.</li>
          <li>Flexible plans, including a free `Lite` option.</li></ul>
          <br>To get started, see [Setting up LogDNA](#openshift_logdna).</td>
    </tr>
    <tr>
        <td>**Monitoring**</td>
        <td>**Built-in OpenShift monitoring tools**:<ul>
          <li>Built-in Prometheus and Grafana deployments in `openshift-monitoring` project for cluster metrics.</li>
          <li>At-a-glance, real-time view of how your pods consume cluster resources that can be accessed from the OpenShift **Cluster Console**.</li>
          <li>Monitoring is on a per-cluster basis.</li>
          <li>The `openshift-monitoring` project stack is set up in a single zone only. No persistent storage is available to back up or view metric history.</li></ul>
          <br>For more information, see [the OpenShift documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://docs.openshift.com/container-platform/3.11/install_config/prometheus_cluster_monitoring.html).</td>
        <td>**{{site.data.keyword.mon_full_notm}}**:<ul>
          <li>Customizable user interface for a unified look at your cluster metrics, container security, resource usage, alerts, and custom events.</li>
          <li>Quick integration with the cluster via a script.</li>
          <li>Aggregated metrics and container monitoring across clusters and cloud providers for consistent operations enablement.</li>
          <li>Historical access to metrics that is based on the timeline and plan, and ability to capture and download trace files.</li>
          <li>Highly available, scalable, and compliant with industry security standards.</li>
          <li>Integrated with {{site.data.keyword.cloud_notm}} IAM for user access management.</li>
          <li>Free trial to try out the capabilities.</li></ul>
          <br>To get started, see [Setting up Sysdig](#openshift_sysdig).</td>
    </tr>
    </tbody>
</table>

<br />


## Setting up LogDNA and Sysdig add-ons to monitor cluster health
{: #openshift_logdna_sysdig}

Because OpenShift sets up stricter [Security Context Constraints (SCC) ![External link icon](../icons/launch-glyph.svg "External link icon")](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_scc.html) by default than community Kubernetes, you might find that some apps or cluster add-ons that you use on community Kubernetes cannot be deployed on OpenShift in the same way. In particular, many images must run as a `root` user or as a privileged container, which is prevented in OpenShift by default. You can modify the default SCCs by creating privileged security accounts and updating the `securityContext` in the pod specification to use two popular {{site.data.keyword.containerlong_notm}} add-ons: {{site.data.keyword.la_full_notm}} and {{site.data.keyword.mon_full_notm}}.
{: shortdesc}

Before you begin, log in to your cluster as an administrator.
1.  Open your OpenShift console at `https://<master_URL>/console`. For example, `https://c0.containers.cloud.ibm.com:23652/console`.
2.  From the OpenShift web console menu bar, click your profile **IAM#user.name@email.com > Copy Login Command** and paste the copied `oc` login command into your terminal to authenticate via the CLI.
3.  Set up [{{site.data.keyword.la_short}}](#openshift_logdna) and [{{site.data.keyword.mon_short}}](#openshift_sysdig).

<br />


### Setting up LogDNA
{: #openshift_logdna}

Set up a project and privileged service account for {{site.data.keyword.la_full_notm}}. Then, create a {{site.data.keyword.la_short}} instance in your {{site.data.keyword.cloud_notm}} account. To integrate your {{site.data.keyword.la_short}} instance with your OpenShift cluster, you must modify the daemon set that is deployed to use the privileged service account to run as root.
{: shortdesc}

1.  Set up the project and privileged service account for LogDNA.
    1.  As a cluster administrator, create a `logdna-agent` project.
        ```
        oc adm new-project logdna-agent
        ```
        {: pre}
    2.  Target the project so that the subsequent resources that you create are in the `logdna-agent` project namespace.
        ```
        oc project logdna-agent
        ```
        {: pre}
    3.  Create a service account for the `logdna-agent` project.
        ```
        oc create serviceaccount logdna-agent
        ```
        {: pre}
    4.  Add a privileged security context constraint to the service account for the `logdna-agent` project.<p class="note">If you want to check what authorization the `privileged` SCC policy gives the service account, run `oc describe scc privileged`. For more information about SCCs, see the [OpenShift docs ![External link icon](../icons/launch-glyph.svg "External link icon")](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_scc.html).</p>
        ```
        oc adm policy add-scc-to-user privileged -n logdna-agent -z logdna-agent
        ```
        {: pre}
2.  Create your {{site.data.keyword.la_full_notm}} instance in the same resource group as your cluster. Select a pricing plan that determines the retention period for your logs, such as `lite` which retains logs for 0 days. The region does not have to match the region of your cluster. For more information, see [Provisioning an instance](/docs/services/Log-Analysis-with-LogDNA/tutorials?topic=LogDNA-provision) and [Service plans](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-service_plans).
    ```
    ibmcloud resource service-instance-create <service_instance_name> logdna (lite|7-day|14-day|30-day) <region> [-g <resource_group>]
    ```
    {: pre}

    Example command:
    ```
    ibmcloud resource service-instance-create logdna-openshift logdna lite us-south
    ```
    {: pre}

    In the output, note the service instance **ID**, which is in the format `crn:v1:bluemix:public:logdna:<region>:<ID_string>::`.
    ```
    Service instance <name> was created.

    Name:         <name>   
    ID:           crn:v1:bluemix:public:logdna:<region>:<ID_string>::   
    GUID:         <guid>   
    Location:     <region>   
    ...
    ```
    {: screen}    
3.  Get your {{site.data.keyword.la_short}} instance ingestion key. The LogDNA ingestion key is used to open a secure web socket to the LogDNA ingestion server and to authenticate the logging agent with the {{site.data.keyword.la_short}} service.
    1.  Create a service key for your LogDNA instance.
        ```
        ibmcloud resource service-key-create <key_name> Administrator --instance-id <logdna_instance_ID>
        ```
        {: pre}
    2.  Note the **ingestion_key** of your service key.
        ```
        ibmcloud resource service-key <key_name>
        ```
        {: pre}

        Example output:
        ```
        Name:          <key_name>  
        ID:            crn:v1:bluemix:public:logdna:<region>:<ID_string>::    
        Created At:    Thu Jun  6 21:31:25 UTC 2019   
        State:         active   
        Credentials:                                   
                       apikey:                   <api_key_value>      
                       iam_apikey_description:   Auto-generated for key <ID>     
                       iam_apikey_name:          <key_name>       
                       iam_role_crn:             crn:v1:bluemix:public:iam::::role:Administrator      
                       iam_serviceid_crn:        crn:v1:bluemix:public:iam-identity::<ID_string>     
                       ingestion_key:            111a11aa1a1a11a1a11a1111aa1a1a11    
        ```
        {: screen}
4.  Create a Kubernetes secret to store your LogDNA ingestion key for your service instance.
    ```
     oc create secret generic logdna-agent-key --from-literal=logdna-agent-key=<logDNA_ingestion_key>
    ```
    {: pre}
5.  Create a Kubernetes daemon set to deploy the LogDNA agent on every worker node of your OpenShift cluster. The LogDNA agent collects logs with the extension `*.log` and extensionless files that are stored in the `/var/log` directory of your pod. By default, logs are collected from all namespaces, including `kube-system`, and automatically forwarded to the {{site.data.keyword.la_short}} service.
    ```
    oc create -f https://raw.githubusercontent.com/logdna/logdna-agent/master/logdna-agent-ds-os.yaml
    ```
    {: pre}
6.  Edit the LogDNA agent daemon set configuration to include the API and logging endpoints for the region that your {{site.data.keyword.la_short}} instance is in.
    1.  Download a local copy of the configuration file to edit.
        ```
        oc get ds logdna-agent -n logdna-agent -o yaml > logdna-ds.yaml
        ```
        {: pre}
    2.  In the configuration file, add the following specifications, update the `spec.template.spec.containers.env` environment variable values for the `LDAPIHOST` and `LDLOGHOST` with the `<region>`.
        ```
        apiVersion: extensions/v1beta1
        kind: DaemonSet
        ...
        spec:
          ...
            spec:
              containers:
              - env:
                - name: LOGDNA_AGENT_KEY
                  valueFrom:
                    secretKeyRef:
                      key: logdna-agent-key
                      name: logdna-agent-key
                - name: LDAPIHOST
                  value: api.<region>.logging.cloud.ibm.com
                - name: LDLOGHOST
                  value: logs.<region>.logging.cloud.ibm.com
        ```
        {: screen}
    3.  Save the configuration file and apply your changes.
        ```
        oc apply -f logdna-ds.yaml
        ```
        {: pre}
    4.  Verify that the new `logdna-agent` pods on each node are in a **Running** status.
        ```
        oc get pods
        ```
        {: pre}
    5.  Optional: If no logs are sent to your {{site.data.keyword.la_short}} instance, delete any `logdna-agent` pods so that they pick up the configuration change.
        ```
        oc delete pod <logdna-agent-123456>
        ```
        {: pre}
7.  From the [{{site.data.keyword.cloud_notm}} **Observability > Logging** console](https://cloud.ibm.com/observe/logging), in the row for your {{site.data.keyword.la_short}} instance, click **View LogDNA**. The LogDNA dashboard opens. After a few minutes, your cluster's logs are displayed, and you can analyze your logs.

For more information about how to use {{site.data.keyword.la_short}}, see the [Next steps docs](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube#kube_next_steps).

<br />


### Setting up Sysdig
{: #openshift_sysdig}

Create an {{site.data.keyword.mon_full_notm}} instance in your {{site.data.keyword.cloud_notm}} account. To integrate your {{site.data.keyword.mon_short}} instance with your OpenShift cluster, you must run a script that creates a project and privileged service account for the Sysdig agent.
{: shortdesc}

1.  Create your {{site.data.keyword.mon_full_notm}} instance in the same resource group as your cluster. Select a pricing plan that determines the retention period for your logs, such as `lite`. The region does not have to match the region of your cluster. For more information, see [Provisioning an instance](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-provision).
    ```
    ibmcloud resource service-instance-create <service_instance_name> sysdig-monitor (lite|graduated-tier) <region> [-g <resource_group>]
    ```
    {: pre}

    Example command:
    ```
    ibmcloud resource service-instance-create sysdig-openshift sysdig-monitor lite us-south
    ```
    {: pre}

    In the output, note the service instance **ID**, which is in the format `crn:v1:bluemix:public:logdna:<region>:<ID_string>::`.
    ```
    Service instance <name> was created.

    Name:         <name>   
    ID:           crn:v1:bluemix:public:sysdig-monitor:<region>:<ID_string>::   
    GUID:         <guid>   
    Location:     <region>   
    ...
    ```
    {: screen}    
2.  Get your {{site.data.keyword.mon_short}} instance access key. The Sysdig access key is used to open a secure web socket to the Sysdig ingestion server and to authenticate the monitoring agent with the {{site.data.keyword.mon_short}} service.
    1.  Create a service key for your Sysdig instance.
        ```
        ibmcloud resource service-key-create <key_name> Administrator --instance-id <sysdig_instance_ID>
        ```
        {: pre}
    2.  Note the **Sysdig Access Key** and **Sysdig Collector Endpoint** of your service key.
        ```
        ibmcloud resource service-key <key_name>
        ```
        {: pre}

        Example output:
        ```
        Name:          <key_name>  
        ID:            crn:v1:bluemix:public:sysdig-monitor:<region>:<ID_string>::    
        Created At:    Thu Jun  6 21:31:25 UTC 2019   
        State:         active   
        Credentials:                                   
                       Sysdig Access Key:           11a1aa11-aa1a-1a1a-a111-11a11aa1aa11      
                       Sysdig Collector Endpoint:   ingest.<region>.monitoring.cloud.ibm.com      
                       Sysdig Customer Id:          11111      
                       Sysdig Endpoint:             https://<region>.monitoring.cloud.ibm.com  
                       apikey:                   <api_key_value>      
                       iam_apikey_description:   Auto-generated for key <ID>     
                       iam_apikey_name:          <key_name>       
                       iam_role_crn:             crn:v1:bluemix:public:iam::::role:Administrator      
                       iam_serviceid_crn:        crn:v1:bluemix:public:iam-identity::<ID_string>       
        ```
        {: screen}
3.  Run the script to set up an `ibm-observe` project with a privileged service account and a Kubernetes daemon set to deploy the Sysdig agent on every worker node of your Kubernetes cluster. The Sysdig agent collects metrics such as the worker node CPU usage, worker node memory usage, HTTP traffic to and from your containers, and data about several infrastructure components.

    In the following command, replace <code>&lt;sysdig_access_key&gt;</code> and <code>&lt;sysdig_collector_endpoint&gt;</code> with the values from the service key that you created earlier. For <code>&lt;tag&gt;</code>, you can associate tags with your Sysdig agent, such as `role:service,location:us-south` to help you identify the environment that the metrics come from.

    ```
    curl -sL https://ibm.biz/install-sysdig-k8s-agent | bash -s -- -a <sysdig_access_key> -c <sysdig_collector_endpoint> -t <tag> -ac 'sysdig_capture_enabled: false' --openshift
    ```
    {: pre}

    Example output:
    ```
    * Detecting operating system
    * Downloading Sysdig cluster role yaml
    * Downloading Sysdig config map yaml
    * Downloading Sysdig daemonset v2 yaml
    * Creating project: ibm-observe
    * Creating sysdig-agent serviceaccount in project: ibm-observe
    * Creating sysdig-agent access policies
    * Creating sysdig-agent secret using the ACCESS_KEY provided
    * Retreiving the IKS Cluster ID and Cluster Name
    * Setting cluster name as <cluster_name>
    * Setting ibm.containers-kubernetes.cluster.id 1fbd0c2ab7dd4c9bb1f2c2f7b36f5c47
    * Updating agent configmap and applying to cluster
    * Setting tags
    * Setting collector endpoint
    * Adding additional configuration to dragent.yaml
    * Enabling Prometheus
    configmap/sysdig-agent created
    * Deploying the sysdig agent
    daemonset.extensions/sysdig-agent created
    ```
    {: screen}

4.  Verify that the `sydig-agent` pods on each node show that **1/1** pods are ready and that each pod has a **Running** status.
    ```
    oc get pods
    ```
    {: pre}

    Example output:
    ```
    NAME                 READY     STATUS    RESTARTS   AGE
    sysdig-agent-qrbcq   1/1       Running   0          1m
    sysdig-agent-rhrgz   1/1       Running   0          1m
    ```
    {: screen}
5.  From the [{{site.data.keyword.cloud_notm}} **Observability > Monitoring** console](https://cloud.ibm.com/observe/logging), in the row for your {{site.data.keyword.mon_short}} instance, click **View Sysdig**. The Sysdig dashboard opens, and you can analyze your cluster metrics.

For more information about how to use {{site.data.keyword.mon_short}}, see the [Next steps docs](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-kubernetes_cluster#kubernetes_cluster_next_steps).

<br />


### Optional: Cleaning up
{: #openshift_logdna_sysdig_cleanup}

Remove the {{site.data.keyword.la_short}} and {{site.data.keyword.mon_short}} instances from your cluster and {{site.data.keyword.cloud_notm}} account. Note that unless you store the logs and metrics in [persistent storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving), you cannot access this information after you delete the instances from your account.
{: shortdesc}

1.  Clean up the {{site.data.keyword.la_short}} and {{site.data.keyword.mon_short}} instances in your cluster by removing the projects that you created for them. When you delete a project, its resources such as service accounts and daemon sets are also removed.
    ```
    oc delete project logdna-agent
    ```
    {: pre}
    ```
    oc delete project ibm-observe
    ```
    {: pre}
2.  Remove the instances from your {{site.data.keyword.cloud_notm}} account.
    *   [Removing a {{site.data.keyword.la_short}} instance](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-remove).
    *   [Removing a {{site.data.keyword.mon_short}} instance](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-remove).

<br />


## Setting up {{site.data.keyword.cloud_notm}} logging and monitoring tools
{: #openshift_other_logmet}

For more information about other logging and monitoring tools that you can set up, including {{site.data.keyword.cloud_notm}} services, see the following topics in the {{site.data.keyword.containershort_notm}} docs.
{: shortdesc}

* [Choosing a logging solution](/docs/containers?topic=containers-health#logging_overview).
* [Choosing a monitoring solution](/docs/containers?topic=containers-health#view_metrics).
