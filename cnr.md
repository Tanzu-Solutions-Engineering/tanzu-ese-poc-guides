# VMware Tanzu Cloud Native Runtimes

This guide was written for Cloud Native Runtime version 1.3


| Release | Details |
|:--|:-:|
| Version | v1.3.0 |
| Release date | July 12, 2022 |
| Knative Serving | 1.3.2 |
| Knative Eventing | 1.3.2 |
| Knative Eventing RabbitMQ Integration | 1.3.1 |
| Knative cert-manager Integration | 1.3.0 |
| Knative Serving Contour Integration | 1.3.0 |
| VMware Tanzu Sources for Knative | 1.3.0 |
| TriggerMesh Sources from Amazon Web Services (SAWS) | 1.6.0 |
| vSphere Event Sources | 1.3.0 |

## Resources

### Documentation

VMware Docs: [Cloud Native Runtimes](https://docs.vmware.com/en/Cloud-Native-Runtimes-for-VMware-Tanzu/index.html)

[Download CNR](https://network.pivotal.io/products/serverless/)

### Confluence

[Getting Started CNR](https://confluence.eng.vmware.com/pages/viewpage.action?spaceKey=CNA&title=Cloud+Native+Runtimes+-+Getting+Started+Guide)

### Github

[VMware Tanzu Sources for Knative](https://github.com/vmware-tanzu/sources-for-knative)


### Demo

[Demo](https://via.vmw.com/cloud-native-runtime-demo)

## Requirements and Prequisuits

**Read carefully the provided Requiremens and Prerequisites!**

[VMware Tanzu Network and container image registry requirements](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-prerequisites.html)

**Important:**

> vSphere with Tanzu v7.0 U3a (not compatible with Tanzu Application Platform v1.0.0 or earlier).
For vSphere with Tanzu, pod security policies must be configured so that Tanzu Application Platform controller pods can run as root.

Create the PSP:

```shell
kubectl create clusterrolebinding default-tkg-admin-privileged-binding \
--clusterrole=psp:vmware-system-privileged \
--group=system:authenticated
```

## Installation on a Tanzu Kubernetes Cluster provisioned using TKGs

### Kapp-Controller Requirement

Since CNR will be installed as a Tanzu Package, an existing `kapp-controller` installation is a prerequisite.

* Install [kapp-controller](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.6/vmware-tanzu-kubernetes-grid-16/GUID-packages-prep-tkgs-kapp.html)

> Link to Tanzu Packages Documentation version 1.6

You can also follow the guidance [HERE](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/packages.md).

Kapp-Controller installed in namespace `tkg-system`:

```shell
k get po -A

NAMESPACE                      NAME                                                             READY   STATUS    RESTARTS        AGE
cert-manager                   cert-manager-b9df787cd-wspb8                                     1/1     Running   0               3d22h
cert-manager                   cert-manager-cainjector-79db995c8c-gf6x6                         1/1     Running   0               3d22h
cert-manager                   cert-manager-webhook-54b6545cd7-lc879                            1/1     Running   0               3d22h
kube-system                    antrea-agent-g4d2m                                               2/2     Running   0               3d23h
kube-system                    antrea-agent-mnjjp                                               2/2     Running   0               3d23h
kube-system                    antrea-agent-zv8zf                                               2/2     Running   0               3d23h
kube-system                    antrea-controller-bb59f5fbf-z9krp                                1/1     Running   0               3d23h
kube-system                    antrea-resource-init-65b586c9db-jp5pw                            1/1     Running   0               3d23h
kube-system                    coredns-5f64c4fff8-798k8                                         1/1     Running   0               3d23h
kube-system                    coredns-5f64c4fff8-qq9rk                                         1/1     Running   0               3d23h
kube-system                    docker-registry-mark50-tkc-2-control-plane-q6x7n                 1/1     Running   0               3d23h
kube-system                    docker-registry-mark50-tkc-2-node-pool-1-76n2c-d9d9bdf97-47gxm   1/1     Running   0               3d23h
kube-system                    docker-registry-mark50-tkc-2-node-pool-1-76n2c-d9d9bdf97-flccl   1/1     Running   0               3d23h
kube-system                    etcd-mark50-tkc-2-control-plane-q6x7n                            1/1     Running   0               3d23h
kube-system                    kube-apiserver-mark50-tkc-2-control-plane-q6x7n                  1/1     Running   0               3d23h
kube-system                    kube-controller-manager-mark50-tkc-2-control-plane-q6x7n         1/1     Running   1 (3d23h ago)   3d23h
kube-system                    kube-proxy-42w6t                                                 1/1     Running   0               3d23h
kube-system                    kube-proxy-4rwwm                                                 1/1     Running   0               3d23h
kube-system                    kube-proxy-q467j                                                 1/1     Running   0               3d23h
kube-system                    kube-scheduler-mark50-tkc-2-control-plane-q6x7n                  1/1     Running   1 (3d23h ago)   3d23h
kube-system                    metrics-server-774bc4dc99-bxhdm                                  1/1     Running   1 (3d23h ago)   3d23h
tkg-system                     kapp-controller-5d5947dccc-fcsxz                                 1/1     Running   0               3d23h
vmware-system-auth             guest-cluster-auth-svc-cxfrb                                     1/1     Running   0               3d23h
vmware-system-cloud-provider   guest-cluster-cloud-provider-84c56c5b74-54dkm                    1/1     Running   1 (3d23h ago)   3d23h
vmware-system-csi              vsphere-csi-controller-79dd5878fb-m2kqc                          6/6     Running   4 (3d23h ago)   3d23h
vmware-system-csi              vsphere-csi-node-lmj9t                                           3/3     Running   0               3d23h
vmware-system-csi              vsphere-csi-node-p7fd6                                           3/3     Running   2 (3d23h ago)   3d23h
vmware-system-csi              vsphere-csi-node-rdtg4                                           3/3     Running   0               3d23h
```

> **NOTE:** If your TKC was installed via Tanzu Mission Control, the `kapp-controller` is already installed during the cluster deployment.

### Secretgen-Controller

The `secretgen-controller` will be leveraged to e.g. distribute `secrets` across namespaces.

* Installation using the `kapp` cli:

```shell
kapp deploy -a sg -f https://github.com/vmware-tanzu/carvel-secretgen-controller/releases/latest/download/release.yml
```

* Installation using `kubectl`:

```shell
kubectl apply -f https://github.com/vmware-tanzu/carvel-secretgen-controller/releases/latest/download/release.yml
```

### Relocate Installation Image Bundles

* Login to VMware's registry using your [PivNet](https://login.run.pivotal.io/login) creds:

```shell
docker login registry.tanzu.vmware.com

Username: rguske@vmware.com
Password:
WARNING! Your password will be stored unencrypted in /home/rguske/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

* Export the appropriate data for the relocating steps:

```shell
export INSTALL_REGISTRY_USERNAME=admin \
export INSTALL_REGISTRY_PASSWORD='VMware1!' \
export INSTALL_REGISTRY_HOSTNAME=harbor-app.jarvis.tanzu \
export TAP_VERSION=1.2.2 \
export INSTALL_REPO=tap
```

* Kick off the relocation of the TAP image bundle:

```shell
imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:${TAP_VERSION} \
--to-repo ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/tap-packages
--registry-ca-cert-path '/etc/docker/certs.d/harbor-app.jarvis.tanzu/ca.crt'
```

I had to add the `--registry-ca-cert-path` option as well in order to not receive a certificate related error message.

> Errors during the relocation:

```shell
copy | importing 177 images...

 900.85 MiB / 6.74 GiB [==========================================>-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------]  13.05% 9.93 MiB/s 09m50 900.85 MiB / 6.74 GiB [=================>----------------------------------------------------------------------------------------------------------------------]  13.05% 9.72 MiB/s 09m48scopy | Error: Error uploading images: Patch "https://harbor.jarvis.tanzu/v2/tap/tap-packages/blobs/uploads/24ed1fcd-ac74-434f-8bcb-381a9c89ccc9?_state=lZOm5Z0BZLqYhELV35tXp5mPuz_KDFBV35bb9i9fnvx7Ik5hbWUiOiJ0YXAvdGFwLXBhY2thZ2VzIiwiVVVJRCI6IjI0ZWQxZmNkLWFjNzQtNDM0Zi04YmNiLTM4MWE5Yzg5Y2NjOSIsIk9mZnNldCI6MCwiU3RhcnRlZEF0IjoiMjAyMi0wOS0xOVQwODozMzozNi41MjM5MDI3MzNaIn0%3D": http: read on closed response body
 132.82 MiB / 6.74 GiB [==>-----------------------------------------------------------------------------------------------------------------------------------]   1.92% 1.16 MiB/s 1h35m32scopy | Error: Error uploading images: Patch "https://harbor.jarvis.tanzu/v2/tap/tap-packages/blobs/uploads/f0c2c2f6-4aa6-49e5-9bf1-8b3201082c9c?_state=vd0qWzaMF6rTeuWObBKwb-hshmiUfMuaIqsakdyCJ6R7Ik5hbWUiOiJ0YXAvdGFwLXBhY2thZ2VzIiwiVVVJRCI6ImYwYzJjMmY2LTRhYTYtNDllNS05YmYxLThiMzIwMTA4MmM5YyIsIk9mZnNldCI6MCwiU3RhcnRlZEF0IjoiMjAyMi0wOS0xOVQwODozNDoxMS41MzMwNDA2MzVaIn0%3D": http: read on closed response body
 215.47 MiB / 6.74 GiB [====>---------------------------------------------------------------------------------------------------------------------------------]   3.12% 1.41 MiB/s 1h18m30scopy | Error: Error uploading images: Patch "https://harbor.jarvis.tanzu/v2/tap/tap-packages/blobs/uploads/2e6705e2-da89-4bda-ba10-099b7691d59b?_state=t23X3khIS4y_FV6PFaWwQXntYbEKJlBn9oIZ4AXmJJ17Ik5hbWUiOiJ0YXAvdGFwLXBhY2thZ2VzIiwiVVVJRCI6IjJlNjcwNWUyLWRhODktNGJkYS1iYTEwLTA5OWI3NjkxZDU5YiIsIk9mZnNldCI6MCwiU3RhcnRlZEF0IjoiMjAyMi0wOS0xOVQwODozNDozOS41MDcyODk4NDlaIn0%3D": http: read on closed response body
 703.18 MiB / 6.74 GiB [=============>--------------------------------------------------------------------------------------------------------------------------]  10.19% 3.09 MiB/s 33m14scopy | Error: Error uploading images: Patch "https://harbor.jarvis.tanzu/v2/tap/tap-packages/blobs/uploads/19af7e4a-a4ef-4922-8880-c5b50ac9133f?_state=eSue9aSk7k3FLtdFxmjdfZzV80a803-OZPcRiqouy0l7Ik5hbWUiOiJ0YXAvdGFwLXBhY2thZ2VzIiwiVVVJRCI6IjE5YWY3ZTRhLWE0ZWYtNDkyMi04ODgwLWM1YjUwYWM5MTMzZiIsIk9mZnNldCI6MCwiU3RhcnRlZEF0IjoiMjAyMi0wOS0xOVQwODozNTo1MC4yODE3NjA5MTFaIn0%3D": http: read on closed response body
 946.59 MiB / 6.74 GiB [==================>---------------------------------------------------------------------------------------------------------------------]  13.72% 3.03 MiB/s 32m40scopy | Error: Error uploading images: Patch "https://harbor.jarvis.tanzu/v2/tap/tap-packages/blobs/uploads/d1b514e2-464b-4044-944a-70de98bad014?_state=agpACqQWkK8TjtcrpdbZv0H9PJkfyS_2yQ7-SVWlDDt7Ik5hbWUiOiJ0YXAvdGFwLXBhY2thZ2VzIiwiVVVJRCI6ImQxYjUxNGUyLTQ2NGItNDA0NC05NDRhLTcwZGU5OGJhZDAxNCIsIk9mZnNldCI6MCwiU3RhcnRlZEF0IjoiMjAyMi0wOS0xOVQwODozNzoxOC41NTIzMzI3MjRaIn0%3D": http: read on closed response body
 946.59 MiB / 6.74 GiB [==================>----------------------------------------------------------------------------------------------------------------------]  13.72% 3.02 MiB/s 5m13s

copy | done uploading images
Error: Retried 5 times: Patch "https://harbor.jarvis.tanzu/v2/tap/tap-packages/blobs/uploads/d1b514e2-464b-4044-944a-70de98bad014?_state=agpACqQWkK8TjtcrpdbZv0H9PJkfyS_2yQ7-SVWlDDt7Ik5hbWUiOiJ0YXAvdGFwLXBhY2thZ2VzIiwiVVVJRCI6ImQxYjUxNGUyLTQ2NGItNDA0NC05NDRhLTcwZGU5OGJhZDAxNCIsIk9mZnNldCI6MCwiU3RhcnRlZEF0IjoiMjAyMi0wOS0xOVQwODozNzoxOC41NTIzMzI3MjRaIn0%3D": http: read on closed response body
```

* Create a `registry-secret` for the installation:

```shell
tanzu secret registry add tap-registry \
--username ${INSTALL_REGISTRY_USERNAME} --password ${INSTALL_REGISTRY_PASSWORD} \
--server ${INSTALL_REGISTRY_HOSTNAME} \
--export-to-all-namespaces --yes --namespace tap-install
```

* Add the new Package URL:

```shell
tanzu package repository add tanzu-tap-repository \
--url ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/tap-packages:$TAP_VERSION \
--namespace tap-install \
--registry-ca-cert-path '/etc/docker/certs.d/harbor.jarvis.tanzu/ca.crt'
```

* Validate the repo:

```shell
tanzu package repository get tanzu-tap-repository \
--namespace tap-install
```

* Retrieve available packages versions:

```shell
tanzu package available list
--namespace tap-install
```

* Retrieve value schema for Cloud Native Runtimes:

```shell
tanzu package available get cnrs.tanzu.vmware.com/1.3.0
--values-schema -n tap-install > cnr-values.yaml
```

```yaml
---
default_tls_secret: # Optional: Overrides the config-contour configmap in namespace knative-serving.
domain_name: "" # Default domain name for Knative Services.
domain_template: # specifies the golang text template string to use when constructing the Knative service's DNS name.
lite.enable: # Not recommended for production. Set to "true" to reduce CPU and Memory resource requests for all CNR Deployments, Daemonsets, and Statefulsets by half. On by default when "provider" is set to "local".
nodeport.enable: # Set to "true" to change CNR deployed envoy Service in contour-external from Type Loadbalancer to Type NodePort. This has no effect if "ingress.external" is set. On by default when "provider" is set to "local".
pdb.enable: # Set to true to enable Pod Disruption Budget. If provider local is set to "local", the PDB will be disabled automatically.
provider: # Kubernetes cluster provider. To be specified if deploying CNR on TKGs or on a local Kubernetes cluster provider.
domain_config: # Overrides the config-domain configmap in namespace knative-serving. Must be valid YAML.
ingress.external.namespace: # Only valid if a Contour instance already present in the cluster. Specify a namespace where an existing Contour is installed on your cluster (for external services) if you want CNR to use your Contour instance.
ingress.internal.namespace: # Only valid if a Contour instance already present in the cluster. Specify a namespace where an existing Contour is installed on your cluster (for internal services) if you want CNR to use your Contour instance.
ingress.reuse_crds: "" # Only valid if a Contour instance already present in the cluster. Set to "true" if you want CNR to re-use the cluster's existing Contour CRDs.
```

**My configuration:**

```yaml
---
  domain_name: cnr.jarvis.tanzu
  provider: tkgs
```

* Install CNR and Contour (bundled):

```shell
tanzu package install cloud-native-runtimes \
-p cnrs.tanzu.vmware.com \
-v 1.3.0 -n tap-install \
-f cnr-values.yaml \
--poll-timeout 30m
```


* Show only the installed objects:

```shell
k get pods -A

NAMESPACE                      NAME                                                             READY   STATUS      RESTARTS       AGE
contour-external               contour-68b67b5b7-4xhkh                                          1/1     Running     0              4m50s
contour-external               contour-68b67b5b7-pw6ks                                          1/1     Running     0              4m50s
contour-external               contour-certgen-v1.19.1--1-z7thw                                 0/1     Completed   0              4m52s
contour-external               envoy-5lmfl                                                      2/2     Running     0              4m52s
contour-external               envoy-slvbv                                                      2/2     Running     0              4m52s
contour-internal               contour-699f6d875-c8tlw                                          1/1     Running     0              4m49s
contour-internal               contour-699f6d875-r888q                                          1/1     Running     0              4m49s
contour-internal               contour-certgen-v1.19.1--1-xlb2j                                 0/1     Completed   0              4m52s
contour-internal               envoy-jb9kr                                                      2/2     Running     0              4m52s
contour-internal               envoy-llmmq                                                      2/2     Running     0              4m52s
knative-eventing               eventing-controller-749b8dbbfb-49rg2                             1/1     Running     0              4m55s
knative-eventing               eventing-webhook-7b7df888dd-8bp6v                                1/1     Running     0              4m40s
knative-eventing               eventing-webhook-7b7df888dd-ljcgf                                1/1     Running     0              4m55s
knative-eventing               imc-controller-7dbc68cf9c-h9rkk                                  1/1     Running     0              4m55s
knative-eventing               imc-dispatcher-6f66bc4798-zlrnd                                  1/1     Running     0              4m54s
knative-eventing               mt-broker-controller-584fd47768-p7dtm                            1/1     Running     0              4m54s
knative-eventing               mt-broker-filter-586fdd76b7-hv5mm                                1/1     Running     0              4m54s
knative-eventing               mt-broker-ingress-5fd5468875-bfb96                               1/1     Running     0              4m54s
knative-eventing               rabbitmq-broker-controller-79bb84f4d7-bnqnd                      1/1     Running     0              4m54s
knative-eventing               rabbitmq-broker-webhook-6df568746b-lm5p6                         1/1     Running     0              4m53s
knative-eventing               sugar-controller-576977c489-g4m8p                                1/1     Running     0              4m53s
knative-serving                activator-7ccc567ff6-8hvc6                                       1/1     Running     0              4m38s
knative-serving                activator-7ccc567ff6-cp2rq                                       1/1     Running     0              4m51s
knative-serving                activator-7ccc567ff6-kw2fs                                       1/1     Running     0              4m38s
knative-serving                autoscaler-7bb5cdc789-7f57b                                      1/1     Running     0              4m51s
knative-serving                autoscaler-hpa-545657b595-zd8hc                                  1/1     Running     0              4m50s
knative-serving                controller-6c9bf85686-6m79k                                      1/1     Running     0              4m51s
knative-serving                domain-mapping-66f5f49bd7-rlwlk                                  1/1     Running     0              4m51s
knative-serving                domainmapping-webhook-66895b6b55-6png8                           1/1     Running     0              4m51s
knative-serving                net-certmanager-controller-7486776b7-5nrll                       1/1     Running     0              4m53s
knative-serving                net-certmanager-webhook-6fc47d4bfc-4rhlj                         1/1     Running     0              4m52s
knative-serving                net-contour-controller-7ddf8c66fc-5vj6b                          1/1     Running     0              4m49s
knative-serving                webhook-85ccd4ff77-hmxnd                                         1/1     Running     0              4m52s
knative-serving                webhook-85ccd4ff77-kd4k6                                         1/1     Running     0              4m38s
knative-sources                rabbitmq-controller-manager-8b564c8fb-zcmk6                      1/1     Running     0              4m53s
knative-sources                rabbitmq-webhook-695fbc5fc8-f7vdg                                1/1     Running     0              4m53s
triggermesh                    aws-event-sources-controller-7b695bf687-lwhwl                    1/1     Running     0              4m55s
vmware-sources                 webhook-9646cd45d-vhkg8                                          1/1     Running     0              4m53s
```

* Create a first Broker in namespace `vmware-functions`:

```shell
kubectl -n vmware-functions create -f - <<EOF
---
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
  name: default
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
EOF
```

### Tanzu Sources for Knative

Install KN vSphere Plugin: [`kn` Plugins](https://github.com/knative/client/tree/8b8b56581c63901b8a73734f002d6f372ed83819/docs/plugins)

[Manually install a plugin](https://knative.dev/docs/client/kn-plugins/#list-of-knative-plugins)


* Create an `Auth-Secret`:

```shell
kn vsphere auth create \
--namespace vmware-functions \
--username veba-ro@mark50.lab \
--password 'VMware1!' \
--name vsphere-credentials \
--verify-url https://mark50-vcsa.jarvis.tanzu \
--verify-insecure
```

* Create the first `vSphereSource`. This source will send all vCenter Events to our InMemory Channel Broker (`default`):

```shell
kn vsphere source create \
--namespace vmware-functions \
--name mark50-vcsa-broker \
--vc-address https://mark50-vcsa.jarvis.tanzu \
--skip-tls-verify \
--secret-ref vsphere-credentials \
--sink-uri http://broker-ingress.knative-eventing.svc.cluster.local/vmware-functions/default \
--encoding json
```

* The second `source` will send the events to an application which is called Sockeye.

Deploy Sockeye for the vSphere events. Due to the Docker Pull Rate Limit, I've pulled the Sockeye image and pushed it to my private Harbor instance. Consequently, the deployment yaml had to be adjusted properly.

Download the manifest [here](https://github.com/n3wscott/sockeye/releases/download/v0.7.0/release.yaml)

Furthermore, I've configured the `minScale` as well as `maxScale` to 1 in order to always have one instance running (not scale to 0).

```yaml
[...]
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "1"
        autoscaling.knative.dev/minScale: "1"
[...]
```

`k -n vmware-functions apply -f sockeye.yaml`

* create a second `vSphere-source` for Sockeye:

```shell
kn vsphere source create \
--namespace vmware-functions \
--name mark50-vcsa-sockeye \
--vc-address https://mark50-vcsa.jarvis.tanzu \
--skip-tls-verify \
--secret-ref vsphere-credentials \
--sink-uri http://sockeye.vmware-functions.cnr.jarvis.tanzu \
--encoding json
```

* This is an example of an vSphere event which was created based on a VM powered off operation

```shell
specversion: 1.
type: com.vmware.vsphere.VmPoweredOffEvent.
source: mark50-vcsa.jarvis.tanzu
id: 855809
time: 2022-09-20T07:06:09.522Z
datacontenttype: application/xml
Extensions,
eventclass: vsphereapiversion: 7.0.3.0
```
