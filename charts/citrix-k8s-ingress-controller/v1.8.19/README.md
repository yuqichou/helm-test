# Citrix Ingress Controller  

[Citrix](https://www.citrix.com/en-in/) provides an Ingress Controller for Citrix ADC MPX (hardware), Citrix ADC VPX (virtualized), and [Citrix ADC CPX](https://docs.citrix.com/en-us/citrix-adc-cpx/13/about.html) (containerized) for [bare metal](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/deployment/baremetal) and [cloud](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/deployment) deployments. It configures one or more Citrix ADC based on the Ingress resource configuration in [Kubernetes](https://kubernetes.io/) or in [OpenShift](https://www.openshift.com) cluster.

## TL;DR;

### For Kubernetes
   ```
   helm repo add citrix https://citrix.github.io/citrix-helm-charts/

   helm install cic citrix/citrix-ingress-controller --set nsIP=<NSIP>,license.accept=yes,loginFileName=<Secret-for-ADC-credentials>
   ```

### For OpenShift

   ```
   helm repo add citrix https://citrix.github.io/citrix-helm-charts/

   helm install cic citrix/citrix-ingress-controller --set nsIP=<NSIP>,license.accept=yes,loginFileName=<Secret-for-ADC-credentials>,openshift=true
   ```

> **Important:**
>
> The `license.accept` argument is mandatory. Ensure that you set the value as `yes` to accept the terms and conditions of the Citrix license.

## Introduction
This Helm chart deploys Citrix ingress controller in the [Kubernetes](https://kubernetes.io) or in the [Openshift](https://www.openshift.com) cluster using [Helm](https://helm.sh) package manager.

### Prerequisites

-  The [Kubernetes](https://kubernetes.io/) version 1.6 or later if using Kubernetes environment.
-  The [Openshift](https://www.openshift.com) version 3.11.x or later if using OpenShift platform.
-  The [Helm](https://helm.sh/) version 3.x or later. You can follow instruction given [here](https://github.com/citrix/citrix-helm-charts/blob/master/Helm_Installation_version_3.md) to install the same.
-  You determine the NS_IP IP address needed by the controller to communicate with Citrix ADC. The IP address might be anyone of the following depending on the type of Citrix ADC deployment:

   -  (Standalone appliances) NSIP - The management IP address of a standalone Citrix ADC appliance. For more information, see [IP Addressing in Citrix ADC](https://docs.citrix.com/en-us/citrix-adc/12-1/networking/ip-addressing.html).

    -  (Appliances in High Availability mode) SNIP - The subnet IP address. For more information, see [IP Addressing in Citrix ADC](https://docs.citrix.com/en-us/citrix-adc/12-1/networking/ip-addressing.html).

    -  (Appliances in Clustered mode) CLIP - The cluster management IP (CLIP) address for a clustered Citrix ADC deployment. For more information, see [IP addressing for a cluster](https://docs.citrix.com/en-us/citrix-adc/12-1/clustering/cluster-overview/ip-addressing.html).

-  You have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator), if you want to view the metrics of the Citrix ADC CPX collected by the [metrics exporter](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics).

-  The user name and password of the Citrix ADC VPX or MPX appliance used as the ingress device. The Citrix ADC appliance needs to have system user account (non-default) with certain privileges so that Citrix ingress controller can configure the Citrix ADC VPX or MPX appliance. For instructions to create the system user account on Citrix ADC, see [Create System User Account for CIC in Citrix ADC](#create-system-user-account-for-cic-in-citrix-adc).

    You can pass user name and password using Kubernetes secrets. Create a Kubernetes secret for the user name and password using the following command:

    ```
       kubectl create secret generic nslogin --from-literal=username='cic' --from-literal=password='mypassword'
    ```

#### Create system User account for Citrix ingress controller in Citrix ADC

Citrix ingress controller configures the Citrix ADC using a system user account of the Citrix ADC. The system user account should have certain privileges so that the CIC has permission configure the following on the Citrix ADC:

-  Add, Delete, or View Content Switching (CS) virtual server
-  Configure CS policies and actions
-  Configure Load Balancing (LB) virtual server
-  Configure Service groups
-  Cofigure SSl certkeys
-  Configure routes
-  Configure user monitors
-  Add system file (for uploading SSL certkeys from Kubernetes)
-  Configure Virtual IP address (VIP)
-  Check the status of the Citrix ADC appliance

> **Note:**
>
> The system user account would have privileges based on the command policy that you define.

To create the system user account, do the following:

1.  Log on to the Citrix ADC appliance. Perform the following:
    1.  Use an SSH client, such as PuTTy, to open an SSH connection to the Citrix ADC appliance.

    2.  Log on to the appliance by using the administrator credentials.

2.  Create the system user account using the following command:

    ```
       add system user <username> <password>
    ```

    For example:

    ```
       add system user cic mypassword
    ```

3.  Create a policy to provide required permissions to the system user account. Use the following command:

    ```
       add cmdpolicy cic-policy ALLOW "(^\S+\s+cs\s+\S+)|(^\S+\s+lb\s+\S+)|(^\S+\s+service\s+\S+)|(^\S+\s+servicegroup\s+\S+)|(^stat\s+system)|(^show\s+ha)|(^\S+\s+ssl\s+certKey)|(^\S+\s+ssl)|(^\S+\s+route)|(^\S+\s+monitor)|(^show\s+ns\s+ip)|(^\S+\s+system\s+file)"
    ```

4.  Bind the policy to the system user account using the following command:

    ```
       bind system user cic cic-policy 0
    ```

## Installing the Chart
Add the Citrix Ingress Controller helm chart repository using command:

```
   helm repo add citrix https://citrix.github.io/citrix-helm-charts/
```

### For Kubernetes:
#### 1. Citrix Ingress Controller
To install the chart with the release name, `my-release`, use the following command:
   ```
   helm install my-release citrix/citrix-ingress-controller --set nsIP=<NSIP>,license.accept=yes,loginFileName=<Secret-for-ADC-credentials>,ingressClass[0]=<ingressClassName>
   ```

> **Note:**
>
> By default the chart installs the recommended [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) roles and role bindings.

The command deploys Citrix ingress controller on Kubernetes cluster with the default configuration. The [configuration](#configuration) section lists the mandatory and optional parameters that you can configure during installation.

#### 2. Citrix Ingress Controller with Exporter
[Metrics exporter](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed along with Citrix ingress controller and collects metrics from the Citrix ADC instances. You can then [visualize these metrics](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/metrics/promotheus-grafana/) using Prometheus Operator and Grafana.

> **Note:**
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator).

Use the following command for this:
   ```
   helm install my-release citrix/citrix-ingress-controller --set nsIP=<NSIP>,license.accept=yes,loginFileName=<Secret-for-ADC-credentials>,ingressClass[0]=<ingressClassName>,exporter.required=true
   ```

### For Openshift:
Add the service account named "cic-k8s-role" to the privileged Security Context Constraints of OpenShift:

   ```
   oc adm policy add-scc-to-user privileged system:serviceaccount:<namespace>:cic-k8s-role
   ```

#### 1. Citrix Ingress Controller
To install the chart with the release name, `my-release`, use the following command:
   ```
   helm install my-release citrix/citrix-ingress-controller --set nsIP=<NSIP>,license.accept=yes,loginFileName=<Secret-for-ADC-credentials>,openshift=true
   ```

The command deploys Citrix ingress controller on your Openshift cluster in the default configuration. The [configuration](#configuration) section lists the mandatory and optional parameters that you can configure during installation.

#### 2. Citrix Ingress Controller with Exporter
[Metrics exporter](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed along with Citrix ingress controller and collects metrics from the Citrix ADC instances. You can then [visualize these metrics](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/metrics/promotheus-grafana/) using Prometheus Operator and Grafana.

> **Note:**
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator)

Use the following command for this:
   ```
   helm install my-release citrix/citrix-ingress-controller --set nsIP=<NSIP>,license.accept=yes,loginFileName=<Secret-for-ADC-credentials>,openshift=true,exporter.required=true
   ```

### Installed components

The following components are installed:

-  [Citrix ingress controller](https://github.com/citrix/citrix-k8s-ingress-controller)
-  [Exporter](https://github.com/citrix/citrix-adc-metrics-exporter) (if enabled)


## CRDs configuration

CRDs gets installed/upgraded automatically when we install/upgrade Citrix ingress controller using Helm. If you do not want to install CRDs, then set the option `crds.install` to `false`. By default, CRDs too get deleted if you uninstall through Helm. This means, even the CustomResource objects created by the customer will get deleted. If you want to avoid this data loss set `crds.retainOnDelete` to `true`.

> **Note:**
> Installing again may fail due to the presence of CRDs. Make sure that you back up all CustomResource objects and clean up CRDs before re-installing Citrix Ingress Controller.

There are a few examples of how to use these CRDs, which are placed in the folder: [Example-CRDs](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds). Refer to them and install as needed, using the following command:
```kubectl create -f <crd-example.yaml>```

### Details of the supported CRDs:

#### authpolicies CRD: 

Authentication policies are used to enforce access restrictions to resources hosted by an application or an API server.

Citrix provides a Kubernetes CustomResourceDefinitions (CRDs) called the [Auth CRD](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/crd/auth) that you can use with the Citrix ingress controller to define authentication policies on the ingress Citrix ADC.

Example file: [auth_example.yaml](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds/auth_example.yaml)
 
#### continuousdeployments CRD  for canary:

Canary release is a technique to reduce the risk of introducing a new software version in production by first rolling out the change to a small subset of users. After user validation, the application is rolled out to the larger set of users. Citrix ADC-Integrated [Canary Deployment solution](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/crd/canary) stitches together all components of continuous delivery (CD) and makes canary deployment easier for the application developers. 

#### httproutes and listeners CRDs for contentrouting:

[Content Routing (CR)](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/crd/contentrouting) is the execution of defined rules that determine the placement and configuration of network traffic between users and web applications, based on the content being sent. For example, a pattern in the URL or header fields of the request.

Example files: [HTTPRoute_crd.yaml](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds/HTTPRoute_crd.yaml), [Listener_crd.yaml](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds/Listener_crd.yaml)

#### ratelimits CRD:

In a Kubernetes deployment, you can [rate limit the requests](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/crd/ratelimit) to the resources on the back end server or services using rate limiting feature provided by the ingress Citrix ADC.

Example files: [ratelimit-example1.yaml](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds/ratelimit-example1.yaml), [ratelimit-example2.yaml](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds/ratelimit-example2.yaml)

#### vips CRD:

Citrix provides a CustomResourceDefinitions (CRD) called [VIP](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/crd/vip) for asynchronous communication between the IPAM controller and Citrix ingress controller.

The IPAM controller is provided by Citrix for IP address management. It allocates IP address to the service from a defined IP address range. The Citrix ingress controller configures the IP address allocated to the service as virtual IP (VIP) in Citrix ADX VPX. And, the service is exposed using the IP address.

When a new service is created, the Citrix ingress controller creates a CRD object for the service with an empty IP address field. The IPAM Controller listens to addition, deletion, or modification of the CRD and updates it with an IP address to the CRD. Once the CRD object is updated, the Citrix ingress controller automatically configures Citrix ADC-specfic configuration in the tier-1 Citrix ADC VPX.

#### rewritepolicies CRD:

In kubernetes environment, to deploy specific layer 7 policies to handle scenarios such as, redirecting HTTP traffic to a specific URL, blocking a set of IP addresses to mitigate DDoS attacks, imposing HTTP to HTTPS and so on, requires you to add appropriate libraries within the microservices and manually configure the policies. Instead, you can use the [Rewrite and Responder features](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/crd/rewrite-responder-policies-deployment.yaml) provided by the Ingress Citrix ADC device to deploy these policies.

Example files: [target-url-rewrite.yaml](https://github.com/citrix/citrix-helm-charts/tree/master/example-crds/target-url-rewrite.yaml)

### Configuration

The following table lists the mandatory and optional parameters that you can configure during installation:

| Parameters | Mandatory or Optional | Default value | Description |
| --------- | --------------------- | ------------- | ----------- |
| license.accept | Mandatory | no | Set `yes` to accept the CIC end user license agreement. |
| image | Mandatory | `quay.io/citrix/citrix-k8s-ingress-controller:1.7.46` | The CIC image. |
| pullPolicy | Mandatory | Always | The CIC image pull policy. |
| loginFileName | Mandatory | N/A | The secret key to log on to the Citrix ADC VPX or MPX. For information on how to create the secret keys, see [Prerequisites](#prerequistes). |
| nsIP | Mandatory | N/A | The IP address of the Citrix ADC device. For details, see [Prerequisites](#prerequistes). |
| nsVIP | Optional | N/A | The Virtual IP address on the Citrix ADC device. |
| nsPort | Optional | 443 | The port used by CIC to communicate with Citrix ADC. You can use port 80 for HTTP. |
| nsProtocol | Optional | HTTPS | The protocol used by CIC to communicate with Citrix ADC. You can also use HTTP on port 80. |
| logLevel | Optional | DEBUG | The loglevel to control the logs generated by CIC. The supported loglevels are: CRITICAL, ERROR, WARNING, INFO, DEBUG and TRACE. For more information, see [Logging](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/configure/log-levels.md).|
| kubernetesURL | Optional | N/A | The kube-apiserver url that CIC uses to register the events. If the value is not specified, CIC uses the [internal kube-apiserver IP address](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#accessing-the-api-from-a-pod). |
| ingressClass | Optional | N/A | If multiple ingress load balancers are used to load balance different ingress resources. You can use this parameter to specify CIC to configure Citrix ADC associated with specific ingress class. For more information on Ingress class, see [Ingress class support](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/configure/ingress-classes/). |
| nodeWatch | Optional | false | Use the argument if you want to automatically configure network route from the Ingress Citrix ADC VPX or MPX to the pods in the Kubernetes cluster. For more information, see [Automatically configure route on the Citrix ADC instance](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/network/staticrouting/#automatically-configure-route-on-the-citrix-adc-instance). |
| defaultSSLCert | Optional | N/A | Default SSL certificate that needs to be used as a non-SNI certificate in Citrix ADC. |
| http2ServerSide | Optional | OFF | Enables HTTP2 for Citrix ADC service group configurations. |
| ipam | Optional | False | Set this argument if you want to use the IPAM controller to automatically allocate an IP address to the service of type LoadBalancer. |
| logProxy | Optional | N/A | Provide Elasticsearch or Kafka or Zipkin endpoint for Citrix observability exporter. |
| nsNamespace | Optional | k8s | The prefix for the resources on the Citrix ADC VPX/MPX. |
| exporter.required | Optional | false | Use the argument, if you want to run the [Exporter for Citrix ADC Stats](https://github.com/citrix/citrix-adc-metrics-exporter) along with CIC to pull metrics for the Citrix ADC VPX or MPX|
| exporter.image    | Optional | `quay.io/citrix/citrix-adc-metrics-exporter:1.4.3` | The Exporter image. |
| exporter.pullPolicy | Optional | Always | The Exporter image pull policy. |
| exporter.ports.containerPort | Optional | 8888 | The Exporter container port. |
| openshift | Optional | false | Set this argument if OpenShift environment is being used. |
| crds.install | Optional | true | Unset this argument if you don't want to install CustomResourceDefinitions which are consumed by CIC. |
| crds.retainOnDelete | Optional | false | Set this argument if you want to retain CustomResourceDefinitions even after uninstalling CIC. This will avoid data-loss of Custom Resource Objects created before uninstallation. |

Alternatively, you can define a YAML file with the values for the parameters and pass the values while installing the chart.

For example:
   ```
   helm install my-release citrix/citrix-ingress-controller -f values.yaml
   ```

> **Tip:** 
>
> The [values.yaml](https://github.com/citrix/citrix-helm-charts/blob/master/citrix-ingress-controller/values.yaml) contains the default values of the parameters.

> **Note:**
> 
> Please provide frontend-ip (VIP) in your application ingress yaml file. For more info refer [this](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/configure/annotations.md).

## Route Addition in MPX/VPX
For seamless functioning of services deployed in the Kubernetes cluster, it is essential that Ingress NetScaler device should be able to reach the underlying overlay network over which Pods are running. 
`feature-node-watch` knob of Citrix Ingress Controller can be used for automatic route configuration on NetScaler towards the pod network. Refer [Static Route Configuration](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/network/staticrouting.md) for further details regarding the same.
By default, `feature-node-watch` is false. It needs to be explicitly set to true if auto route configuration is required.
This can also be achieved by deploying [Citrix Node Controller](https://github.com/citrix/citrix-k8s-node-controller).

For configuring static routes manually on Citrix ADC VPX or MPX to reach the pods inside the cluster follow:
### For Kubernetes:
1. Obtain podCIDR using below options:
   ```
   kubectl get nodes -o yaml | grep podCIDR
   ```
   * podCIDR: 10.244.0.0/24
   * podCIDR: 10.244.1.0/24
   * podCIDR: 10.244.2.0/24

2. Log on to the Citrix ADC instance.

3. Add Route in Netscaler VPX/MPX
   ```
   add route <podCIDR_network> <podCIDR_netmask> <node_HostIP>
   ```
4. Ensure that Ingress MPX/VPX has a SNIP present in the host-network (i.e. network over which K8S nodes communicate with each other. Usually eth0 IP is from this network).

   Example:
   * Node1 IP = 192.0.2.1
   * podCIDR  = 10.244.1.0/24
   * add route 10.244.1.0 255.255.255.0 192.0.2.1

### For OpenShift:
1. Use the following command to get the information about host names, host IP addresses, and subnets for static route configuration.
   ```
    oc get hostsubnet
   ```

2. Log on to the Citrix ADC instance.

3. Add the route on the Citrix ADC instance using the following command.
   ```add route <pod_network> <podCIDR_netmask> <gateway>```

4. Ensure that Ingress MPX/VPX has a SNIP present in the host-network (i.e. network over which OpenShift nodes communicate with each other. Usually eth0 IP is from this network).

    For example, if the output of the `oc get hostsubnet` is as follows:
    * oc get hostsubnet

        NAME            HOST           HOST IP        SUBNET
        os.example.com  os.example.com 192.0.2.1 10.1.1.0/24

    * The required static route is as follows:

           add route 10.1.1.0 255.255.255.0 192.0.2.1

## Uninstalling the Chart
To uninstall/delete the ```my-release``` deployment:

   ```
   helm delete my-release
   ```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Related documentation

- [Citrix ingress controller Documentation](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/)
- [Citrix ingress controller GitHub](https://github.com/citrix/citrix-k8s-ingress-controller)
