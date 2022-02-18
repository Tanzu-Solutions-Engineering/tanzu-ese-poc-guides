# Tanzu Packages

Tanzu Kubernetes Grid includes two types of packages:

**Core packages** are automatically installed and managed by Tanzu Kubernetes Grid. These packages are located in the tanzu-core package repository.
**User-managed packages** are installed and managed by you. These packages are located in the tanzu-standard package repository.

Both the tanzu-core and the tanzu-standard package repositories are automatically enabled in every cluster.

## Table of Content

- [Tanzu Packages](#tanzu-packages)
  - [Table of Content](#table-of-content)
  - [Prerequisites](#prerequisites)
    - [Domain Activations](#domain-activations)
  - [List of User-Managed Packages](#list-of-user-managed-packages)
  - [CLIs](#clis)
    - [Tanzu Cli](#tanzu-cli)
    - [kubectl](#kubectl)
    - [imgpkg](#imgpkg)
  - [Install kapp-controller (online)](#install-kapp-controller-online)
  - [Install Tanzu Package Repository](#install-tanzu-package-repository)
  - [Install Cert-Manager](#install-cert-manager)
  - [Install Contour](#install-contour)
  - [Install Multus-CNI](#install-multus-cni)
  - [Delete a Package](#delete-a-package)
  - [Troubleshooting](#troubleshooting)
    - [Temporary Pause Lifecycle Management](#temporary-pause-lifecycle-management)
  - [Proxy](#proxy)
    - [PhotonOS](#photonos)
    - [Ubuntu](#ubuntu)
    - [CentOS/RHEL](#centosrhel)
  - [Internet Restricted Installation](#internet-restricted-installation)
    - [Registry Certificate Required](#registry-certificate-required)
    - [Kapp Controller Preperations](#kapp-controller-preperations)
    - [Install kapp-controller (offline)](#install-kapp-controller-offline)
    - [Kapp-Controller Authentication with the Private Registry](#kapp-controller-authentication-with-the-private-registry)
    - [Package Bundles Preperations](#package-bundles-preperations)
    - [Optional: kapp-controller secret vs. configMap](#optional-kapp-controller-secret-vs-configmap)
  - [Resources](#resources)

## Prerequisites

### Domain Activations

- *.tmc.cloud.vmware.com
- *.console.cloud.vmware.com
- *.cloud.vmware.com
- *.projects.registry.vmware.com
- *.registry.vmware.com
- *.vmware.com

## List of User-Managed Packages

| Function | Package Package | Repository |
| :--: | :--: | :--: |
| Certificate management | cert-manager | tanzu-standard |
| Container networking | multus-cni | tanzu-standard |
| Container registry |harbor |tanzu-standard |
| Ingress control | contour | tanzu-standard |
| Log forwarding | fluent-bit | tanzu-standard |
| Monitoring | grafana | tanzu-standard |
| Monitoring | prometheus | tanzu-standard |
| Service discovery | external-dns | tanzu-standard |

> Table column *Repository*, is the name of the package repository (`tanzu-standard`) which is configured by default in namespace `tanzu-package-repo-global` and which normally is configured to use URL `projects.registry.vmware.com/tkg/packages/standard/repo`.

## CLIs

### Tanzu Cli

**Download relevent package from** https://customerconnect.vmware.com/

Docs - [Install the Tanzu CLI and Other Tools](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-install-cli.html)

```shell
# unpack the tarball
tar -xvf tanzu-cli-bundle-linux-amd64.tar

# unpacked tarball
.
├── cluster
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-cluster-linux_amd64
├── core
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-core-linux_amd64
├── imgpkg-linux-amd64-v0.10.0+vmware.1.gz
├── kapp-linux-amd64-v0.37.0+vmware.1.gz
├── kbld-linux-amd64-v0.30.0+vmware.1.gz
├── kubernetes-release
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-kubernetes-release-linux_amd64
├── login
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-login-linux_amd64
├── management-cluster
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-management-cluster-linux_amd64
├── manifest.yaml
├── package
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-package-linux_amd64
├── pinniped-auth
│   ├── plugin.yaml
│   └── v1.4.1
│       └── tanzu-pinniped-auth-linux_amd64
├── vendir-linux-amd64-v0.21.1+vmware.1.gz
└── ytt-linux-amd64-v0.34.0+vmware.1.gz

# install the tanzu cli
sudo install cli/core/v1.4.0/tanzu-core-linux_amd64 /usr/local/bin/tanzu

# version check
tanzu version

# tanzu cli update
tanzu update

# remove existing plugins from any previous CLI installations
tanzu plugin clean

# install all the plugins for this release
tanzu plugin install --local cli all

# check plugin installation status
tanzu plugin list
```

### kubectl

```shell
# unpack the binary
gzip -d kubectl-linux-v1.21.2+vmware.1.gz

# make it executable
chmod +x kubectl-linux-v1.21.2+vmware.1

# move the executable to your /usr/local/bin
sudo mv kubectl-linux-v1.21.2+vmware.1 /usr/local/bin/kubectl

# version check
kubectl version
```

### imgpkg

`imgpkg` is required for deploying Tanzu Kubernetes Grid in Internet-restricted environments and when building your own machine images. It is also required when configuring the Harbor package.

```shell
# change dir into the cli folder of the unpacked Tanzu cli
cd cli

# unpack .gz file
gunzip imgpkg-linux-amd64-v0.10.0+vmware.1.gz

# make it executable
chmod ugo+x imgpkg-linux-amd64-v0.10.0+vmware.1

# move the binary into /usr/local/bin
mv ./imgpkg-linux-amd64-v0.10.0+vmware.1 /usr/local/bin/imgpkg
```

## Install kapp-controller (online)

```shell
# login TKC
kubectl vsphere login --insecure-skip-tls-verify --vsphere-username administrator@mark50.lab --server=mark50.jarvis.tanzu --tanzu-kubernetes-cluster-name mark50-tkc-01 --tanzu-kubernetes-cluster-namespace mark50-ns-01

# switch into the right context
kubectl config use-context mark50-tkc-01

# export kapp controller version
#Check the available versions: https://github.com/vmware-tanzu/carvel-kapp-controller/releases
export KAPP_VERSION=v0.23.0

# install kapp controller
kubectl apply -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/download/$KAPP_VERSION/release.yml
```

## Install Tanzu Package Repository

```shell
# create a namespace for the packages
kubectl create ns tanzu-package-repo-global

# add the repository
tanzu package repository add tanzu-standard --url projects.registry.vmware.com/tkg/packages/standard/repo:v1.4.0 -n tanzu-package-repo-global

# display available Tanzu packages
tanzu package available list -n tanzu-package-repo-global

# display Tanzu package repository
tanzu package repository get tanzu-standard -n tanzu-package-repo-global
```

## Install Cert-Manager

```shell
# display available Tanzu packages
tanzu package available list -n tanzu-package-repo-global

# list cert-manager package availability
tanzu package available list cert-manager.tanzu.vmware.com -n tanzu-package-repo-global

# check a specific cert-manager version
tanzu package available get cert-manager.tanzu.vmware.com/1.1.0+vmware.1-tkg.2 -n tanzu-package-repo-global

# install cert-manager version
tanzu package install cert-manager -p cert-manager.tanzu.vmware.com -v 1.1.0+vmware.1-tkg.2 -n tanzu-package-repo-global

# validate cert-manager is successfully running
tanzu package installed list -n tanzu-package-repo-global
```

## Install Contour

```shell
# list contour package availability
tanzu package available list contour.tanzu.vmware.com -n tanzu-package-repo-global

# check a specific contour version
tanzu package available get contour.tanzu.vmware.com/1.17.1+vmware.1-tkg.1 -n tanzu-package-repo-global

# install contour version
tanzu package install contour -p contour.tanzu.vmware.com -v 1.17.1+vmware.1-tkg.1 -n tanzu-package-repo-global --values-file contour-data-values.yaml
```

## Install Multus-CNI

```shell
# list multus package availability
tanzu package available list multus-cni.tanzu.vmware.com -n tanzu-package-repo-global

# check a specific multus version
tanzu package available get multus-cni.tanzu.vmware.com/3.7.1+vmware.1-tkg.1 -n tanzu-package-repo-global

# install multus version
tanzu package install multus-cni -p multus-cni.tanzu.vmware.com -v 3.7.1+vmware.1-tkg.1 -n tanzu-package-repo-global --values-file multus-data-values.yaml
```

## Delete a Package

The tanzu package installed delete command deletes a user-managed package.

`tanzu package installed delete INSTALLED-PACKAGE-NAME -n INSTALLED-PACKAGE-NAMESPACE`

## Troubleshooting

| Command | Description |
| :--: | :--: |
|`kubectl get packageinstall CORE-PACKAGE-NAME -n tkg-system -o yaml`| Check the PackageInstall CR in your target cluster. For example, kubectl get packageinstall antrea -n tkg-system -o yaml. |
| `kubectl get app CORE-ADD-ON-NAME -n tkg-system -o yaml` | Check the App CR in your target cluster. For example, kubectl get app antrea -n tkg-system -o yaml. |
| `kubectl get cluster CLUSTER-NAME -n CLUSTER-NAMESPACE -o jsonpath={.metadata.labels.tanzuKubernetesRelease}` | In the management cluster, check if the TKr label of your target cluster points to the correct TKr. |
| `kubectl get tanzukubernetesrelease TKR-NAME` | Check if the TKr is present in the management cluster. |
| `kubectl get configmaps -n tkr-system -l 'tanzuKubernetesRelease=TKR-NAME'` | Check if the BoM ConfigMap corresponding to your TKr is present in the management cluster. |
| `kubectl get app CLUSTER-NAME-kapp-controller -n CLUSTER-NAMESPACE` | For workload clusters, check if the kapp-controller App CR is present in the management cluster. |
| `kubectl logs deployment/tanzu-addons-controller-manager -n tkg-system` | Check tanzu-addons-manager logs in the management cluster. |
| `kubectl get configmap -n tkg-system | grep CORE-ADD-ON-NAME-ctrl` | Check if your updates to the add-on secret have been applied. The sync period is 5 minutes. |

### Temporary Pause Lifecycle Management

If you need to temporary pause lifecycle management for a core package, you can use the commands below. To pause secret reconciliation, run the following command against the management cluster:

`kubectl patch secret/CLUSTER-NAME-CORE-ADD-ON-NAME-addon -n CLUSTER-NAMESPACE -p '{"metadata":{"annotations":{"tkg.tanzu.vmware.com/addon-paused": ""}}}' --type=merge`

After you run this command, tanzu-addons-manager stops reconciling the secret. To pause PackageInstall CR reconciliation, run the following command against your target cluster:

`kubectl patch packageinstall/CORE-PACKAGE-NAME -n tkg-system -p '{"spec":{"paused":true}}' --type=merge`

After you run this command, kapp-controller stops reconciling the PackageInstall and corresponding App CR.

If you want to temporary modify the resources of a core add-on, pause secret reconciliation first and then pause PackageInstall CR reconciliation. After you unpause lifecycle management, tanzu-addons-manager and kapp-controller resume secret and PackageInstall CR reconciliation:

- To unpause secret reconciliation, remove `tkg.tanzu.vmware.com/addon-paused` from the secret annotations.
- To unpause PackageInstall CR reconciliation, update the PackageInstall CR with `{"spec":{"paused":false}}` or remove the variable.

## Proxy

General tips:

**Special character handling:**

```shell
Literal backslash characters (\) need to be doubled escape them as shown below.

# export http_proxy=http://DOMAIN\\USERNAME:PASSWORD@SERVER:PORT/
When the username or password uses the @ symbol, add a backslash (\) before the @ – for example:

# export http_proxy=http://DOMAIN\\USERN\@ME:PASSWORD@SERVER:PORT
or

# export http_proxy=http://DOMAIN\\USERNAME:P\@SSWORD@SERVER:PORT
```

**NO_PROXY:**

> Configure `NO_PROXY` to ensures that traffic destined to internal addresses won’t get forwarded to the proxy.

### PhotonOS

[PhotonOS Wiki](https://github.com/vmware/photon/wiki/)

Proxy configuration for VMware Photon OS. There are multiplie places in which a proxy can be defined, including in the Kubernetes configuration, or specifically for the tdnf package manager.

`vim /etc/sysconfig/proxy`

`tdnf` is using HTTPS as a default!

### Ubuntu

**Temporary:**

Check if Proxy settings are set:

`env | grep proxy`

Set settings temporary (settings belonging to one shell session!):

`export HTTP_PROXY=http://<user>:<pass>@<proxy>:<port>/`
`export HTTPS_PROXY=http://<user>:<pass>@<proxy>:<port>/`
`export NO_PROXY=localhost,127.0.0.1,::1`

Without user: `export HTTP_PROXY=http://SERVER:PORT/`

**Permanent for All Users**:

`sudo vi /etc/environment`

Update the file with the same information listed above.

**Setting Up Proxy for `apt`**

`sudo vi /etc/apt/apt.conf`

```shell
Acquire::http::Proxy "http://[username]:[password]@ [proxy-web-or-IP-address]:[port-number]";
Acquire::https::Proxy "http://[username]:[password]@ [proxy-web-or-IP-address]:[port-number]";
```

### CentOS/RHEL

Check if Proxy settings are set:

`echo $http_proxy`

**Temporary**

Without user: `export http_proxy=http://SERVER:PORT/`

With user: `export http_proxy=http://USERNAME:PASSWORD@SERVER:PORT/`

With a Domain user: `export http_proxy=http://DOMAIN\\USERNAME:PASSWORD@SERVER:PORT/`

**Permanent:**

`echo "http_proxy=http://proxy.example.com:3128/" > /etc/environment`

> Note that unlike a shell script in /etc/profile.d described in the next section, the /etc/environment file is NOT a shell script and applies to all processes without a shell.
> Source: [How to Configure Proxy in CentOS/RHEL/Fedora](https://www.thegeekdiary.com/how-to-configure-proxy-server-in-centos-rhel-fedora/)

**Configuring proxy for processes with SHELL**

For bash and sh users, add the export line given above into a new file called `/etc/profile.d/http_proxy.sh` file:

`echo "export http_proxy=http://proxy.example.com:3128/" > /etc/profile.d/http_proxy.sh`

**Setting Up Proxy for `yum`**

```shell
vi /etc/yum.conf

proxy=http://proxy.example.com:3128
proxy_username=yum-user
proxy_password=qwerty
```

## Internet Restricted Installation

### Registry Certificate Required

Download the Harbor certificate and add it (if necessary) to your Docker config.

```shell
# enter the FQDN of your Harbor instance (e.g. harbor.jarvis.tanzu)
read REGISTRY

# download the certificate
sudo wget -O ~/Downloads/ca.crt https://$REGISTRY/api/v2.0/systeminfo/getcert --no-check-certificate
```

Add it to your Docker config:

```shell
# create a folder named like your registry
sudo mkdir -p /etc/docker/certs.d/$REGISTRY

# download the certificate
sudo wget -O /etc/docker/certs.d/$REGISTRY/ca.crt https://$REGISTRY/api/v2.0/systeminfo/getcert --no-check-certificate

# restart docker daemon
systemctl restart docker

# login
docker login harbor.jarvis.tanzu

Username: admin
Password:
Login Succeeded
```

### Kapp Controller Preperations

```shell
# pull and push the kapp-controller image into the destination registry
docker pull projects.registry.vmware.com/tkg/kapp-controller:v0.23.0_vmware.1

# tag the kapp-ctrl image to be prepared for the image push
docker tag projects.registry.vmware.com/tkg/kapp-controller:v0.23.0_vmware.1 harbor.jarvis.tanzu/tanzu/kapp-controller:v0.23.0_vmware.1

# push the image to the destination registry
docker push harbor.jarvis.tanzu/tanzu/kapp-controller:v0.23.0_vmware.1

# list images
docker images

REPOSITORY                                           TAG                IMAGE ID       CREATED        SIZE
harbor.jarvis.tanzu/tanzu-packages/kapp-controller   v0.23.0_vmware.1   0076b17e8c71   5 months ago   639MB
projects.registry.vmware.com/tkg/kapp-controller     v0.23.0_vmware.1   0076b17e8c71   5 months ago   639MB

# push the image to the destination registry
docker push harbor.jarvis.tanzu/tanzu-packages/kapp-controller:v0.23.0_vmware.1
```

### Install kapp-controller (offline)

In order to install the kapp-controller to our Kubernetes cluster, a deployment manifest file has to be created first. Create a new file called e.g. `kapp-controller.yaml` and add the provided specifications from the [docs](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-packages-prep-tkgs-kapp.html#kapp-controller).

Open the file and jump to line number `1278`. By using `vim` for example, you can simply enable line numbers. Just enter `:set numbers` and jump to the line by executing `:1278`. Otherwise, simply search `/` for `image: projects`.

Replace the original repository and image with yours.

```yaml
# replace the old image url
[...]

1278         image: harbor.jarvis.tanzu/tanzu-packages/kapp-controller:v0.23.0_vmware.1
1279         name: kapp-controller
1280         ports:

[...]
```

```shell
kubectl apply -f kapp-controller.yaml
```

### Kapp-Controller Authentication with the Private Registry

Edit the kapp controller config to trust your private container registry.

**Step 1:** Edit the kapp-controller `configMap`:

```shell
# edit configMap
k -n tkg-system edit cm kapp-controller-config
```

**Step 2:** Add your certificate data under section `caCerts` like shown in my example below:

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  caCerts: |
    -----BEGIN CERTIFICATE-----
    MIIDWzCCAkOgAwIBAgIRAMODWpLzIWy3JocU4JBtIrIwDQYJKoZIhvcNAQELBQAw
    LTEXMBUGA1UEChMOUHJvamVjdCBIYXJib3IxEjAQBgNVBAMTCUhhcmJvciBDQTAe
    Fw0yMjAxMTExMDEwMDlaFw0zMjAxMDkxMDEwMDlaMC0xFzAVBgNVBAoTDlByb2pl
    Y3QgSGFyYm9yMRIwEAYDVQQDEwlIYXJib3IgQ0EwggEiMA0GCSqGSIb3DQEBAQUA

    [...]
    -----END CERTIFICATE-----
  dangerousSkipTLSVerify: ""
  httpProxy: ""
  httpsProxy: ""
  noProxy: ""
kind: ConfigMap
metadata:
  annotations:
    kapp.k14s.io/change-group: apps.kappctrl.k14s.io/kapp-controller-config
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"caCerts":"","dangerousSkipTLSVerify":"","httpProxy":"","httpsProxy":"","noProxy":""},"kind":"ConfigMap","metadata":{"annotations":{"kapp.k14s.io/change-group":"apps.kappctrl.k14s.io/kapp-controller-config"},"name":"kapp-controller-config","namespace":"tkg-system"}}
  creationTimestamp: "2022-02-08T12:31:35Z"
  name: kapp-controller-config
  namespace: tkg-system
  resourceVersion: "1168617"
  uid: 73e9cc55-02d4-4790-a7ee-eaa38b15c894
```

Save your adjustments `:wq`.

**Step 3:** Restart/`delete` the kapp-controller pod in order to let the changes take effect:

```shell
# delete the kapp-controller pod
k -n tkg-system delete pod kapp-controller-5fd59df9dd-xmvmj
```

**Step 4:** Add the URL, which is pointing to your private package repository and use the namespace `tanzu-package-repo-global`:

```shell
# add the offline package repository
tanzu package repository add tanzu-packages-offline --url harbor.jarvis.tanzu/tanzu-packages/tanzu-packages:v1.4.0 -n tanzu-package-repo-global
```

To let the kapp-controller authenticate with your private registry, you have to create a Kubernetes `secret` which in turn has to be referenced in the `PackageRepository` custom resource (CR).

**Step 1:** Create the Kubernetes `secret`:

```shell
# create a k8s docker-registry secret to authenticate with your registry
kubectl -n tanzu-package-repo-global create secret docker-registry harbor-creds --docker-server='harbor.jarvis.tanzu' --docker-username='admin' --docker-password='your-password' --docker-email='rguske@vmware.com'
```

```shell
# validate the creation of the secret
k get secrets -n tanzu-package-repo-global

NAME                                                    TYPE                                  DATA   AGE
cert-manager-tanzu-package-repo-global-sa-token-6n57n   kubernetes.io/service-account-token   3      3h6m
default-token-wzv5l                                     kubernetes.io/service-account-token   3      24h
harbor-creds
```

**Step 2:**

Adjust the `PackageRepository` CR accordingly to use the new `secret` and to ultimately authenticate with your registry:

```shell
k -n tanzu-package-repo-global edit packagerepositories.packaging.carvel.dev tanzu-packages-offline

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
  creationTimestamp: "2022-02-09T10:11:02Z"
  finalizers:
  - finalizers.packagerepository.packaging.carvel.dev/delete
  generation: 1
  name: tanzu-packages-offline
  namespace: tanzu-package-repo-global
  resourceVersion: "1464972"
  uid: d42032e9-5534-4503-a065-469c3434a94d
spec:
  fetch:
    imgpkgBundle:
      image: harbor.jarvis.tanzu/tanzu-packages/tanzu-packages:v1.4.0
      secretRef:
        name: harbor-creds
[...]
```

**Step 3:**

Validate that the changes has taken effect and that the kapp-controller can successfully `reconcile` the repository. The status for the package repository should have changed from `Reconcile failed:` to `Reconcile succeeded`.

```shell
# validate the configuration of the repository
tanzu package repository list -n tanzu-package-repo-global

- Retrieving repositories...
  NAME                    REPOSITORY                                         TAG     STATUS               DETAILS
  tanzu-packages-offline  harbor.jarvis.tanzu/tanzu-packages/tanzu-packages  v1.4.0  Reconcile succeeded
```

**Step 4:** Also, validate the available (offline) packages:

```shell
# check package availability
tanzu package available list -n tanzu-package-repo-global

- Retrieving available packages...
  NAME                           DISPLAY-NAME  SHORT-DESCRIPTION                                                                                           LATEST-VERSION
  cert-manager.tanzu.vmware.com  cert-manager  Certificate management                                                                                      1.1.0+vmware.1-tkg.2
  contour.tanzu.vmware.com       Contour       An ingress controller                                                                                       1.17.1+vmware.1-tkg.1
  external-dns.tanzu.vmware.com  external-dns  This package provides DNS synchronization functionality.                                                    0.8.0+vmware.1-tkg.1
  fluent-bit.tanzu.vmware.com    fluent-bit    Fluent Bit is a fast Log Processor and Forwarder                                                            1.7.5+vmware.1-tkg.1
  grafana.tanzu.vmware.com       grafana       Visualization and analytics software                                                                        7.5.7+vmware.1-tkg.1
  harbor.tanzu.vmware.com        Harbor        OCI Registry                                                                                                2.2.3+vmware.1-tkg.1
  multus-cni.tanzu.vmware.com    multus-cni    This package provides the ability for enabling attaching multiple network interfaces to pods in Kubernetes  3.7.1+vmware.1-tkg.1
  prometheus.tanzu.vmware.com    prometheus    A time series database for your metrics                                                                     2.27.0+vmware.1-tkg.1
```

### Package Bundles Preperations

```shell
# use the imgpkg copy command to copy the bundle into a tar ball on your jumpbox
imgpkg copy -b projects.registry.vmware.com/tkg/packages/standard/repo:v1.4.0 --to-tar ~/Downloads/packages.tar

# use the imgpkg copy command to `push` the content into your private container registry
imgpkg copy --tar packages-images.tar --to-repo harbor.jarvis.tanzu/packages/packages-images --registry-ca-cert-path=ca.cer --registry-username=admin --registry-password='$PASSWORD'
```

### Optional: kapp-controller secret vs. configMap

Instead of making adjustments to the `configMap` of the kapp-controller, a creation of a Kubernetes `secret` can be used as an alternative. I validated both ways successfully. A more detailed description can be found on the official Carvel documentation here: [Configuring the Controller](https://carvel.dev/kapp-controller/docs/v0.32.0/controller-config/#controller-configuration-spec)

Here's my example I've tested:

```yaml
apiVersion: v1
kind: Secret
metadata:
  # Name must be `kapp-controller-config` for kapp controller to pick it up
  name: kapp-controller-config
  # Namespace must match the namespace kapp-controller is deployed to
  namespace: tkg-system
stringData:
  # A cert chain of trusted ca certs. These will be added to the system-wide
  # cert pool of trusted ca's (optional)
  caCerts: |
    -----BEGIN CERTIFICATE-----
    MIIDWzCCAkOgAwIBAgIRAMODWpLzIWy3JocU4JBtIrIwDQYJKoZIhvcNAQELBQAw

    [...]
    -----END CERTIFICATE-----
  # The url/ip of a proxy for kapp controller to use when making network
  # requests (optional)
  httpProxy: ""
  # The url/ip of a tls capable proxy for kapp controller to use when
  # making network requests (optional)
  httpsProxy: ""
  # A comma delimited list of domain names which kapp controller should
  # bypass the proxy for when making requests (optional)
  noProxy: ""
  # A comma delimited list of domain names for which kapp controller, when
  # fetching images or imgpkgBundles, will skip TLS verification. (optional)
  dangerousSkipTLSVerify: ""
```

## Resources

- [Install the Tanzu CLI and Other Tools](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-install-cli.html)
- [Install and Configure Packages](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-packages-index.html)
- [Tanzu Packages Explained](https://beyondelastic.com/2022/01/04/tanzu-packages-explained/)
- [Deploy VMware Tanzu Packages from a private Container Registry](https://rguske.github.io/post/deploy-tanzu-packages-from-a-private-registry/)
- [Deploying Tanzu Packages using Tanzu Mission Control Catalog](https://rguske.github.io/post/deploying-tanzu-packages-using-tanzu-mission-control-catalog/)
- [Configure vSphere with Tanzu behind a Proxy plus TKG Extensions](https://beyondelastic.com/2021/06/21/configure-vsphere-with-tanzu-behind-a-proxy-plus-tkg-extensions/)
