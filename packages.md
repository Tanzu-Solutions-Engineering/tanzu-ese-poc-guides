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
  - [Install kapp-controller](#install-kapp-controller)
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
  - [Airgapped Installation](#airgapped-installation)
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

## CLIs

### Tanzu Cli

**Download relevent package from** https://customerconnect.vmware.com/

Docs - [Install the Tanzu CLI and Other Tools](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-install-cli.html)

```shell
# unpack the tar ball
tar -xvf tanzu-cli-bundle-linux-amd64.tar

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

## Install kapp-controller

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
| :-- | :-- |
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

## Airgapped Installation

Kapp Controller preps:

```shell
# pull and push the kapp-controller image into the destination registry
docker pull projects.registry.vmware.com/tkg/kapp-controller:v0.23.0_vmware.1
# tag the kapp-ctrl image to be prepared for the image push
docker tag projects.registry.vmware.com/tkg/kapp-controller:v0.23.0_vmware.1 harbor.jarvis.tanzu/tanzu/kapp-controller:v0.23.0_vmware.1
# push the image to the destination registry
docker push harbor.jarvis.tanzu/tanzu/kapp-controller:v0.23.0_vmware.1
```

Download the Harbor certificate and add it (if necessary) to your Docker config.

```shell
# enter your Harbor registry URL (w/o https)
read REGISTRY
# create a folder named like your registry
sudo mkdir -p /etc/docker/certs.d/$REGISTRY
# download the certificate
sudo wget -O /etc/docker/certs.d/$REGISTRY/ca.crt https://$REGISTRY/api/v2.0/systeminfo/getcert --no-check-certificate
# restart docker daemon
systemctl restart docker
```

Packages bundle preps:

```shell
# use the imgpkg copy command to copy the bundle into a tar ball on your jumpbox
imgpkg copy -b projects.registry.vmware.com/tkg/packages/standard/repo:v1.4.0 --to-tar ~/Downloads/packages.tar
# use the imgpkg copy command to `push` the content into your private container registry
imgpkg copy --tar packages-images.tar --to-repo harbor.jarvis.tanzu/packages/packages-images --registry-ca-cert-path=ca.cer --registry-username=admin --registry-password='VMware1!'
```

Edit the kapp controller config to trust your private container registry.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  # Name must be `kapp-controller-config` for kapp controller to pick it up
  name: kapp-controller-config
  # Namespace must match the namespace kapp-controller is deployed to
  namespace: tkg-system
data:
  # A cert chain of trusted ca certs. These will be added to the system-wide
  # cert pool of trusted ca's (optional)
  #  caCerts: |
  #    -----BEGIN CERTIFICATE-----
  #    Certificate 1
  #    -----END CERTIFICATE-----
  #    -----BEGIN CERTIFICATE-----
  #    Certificate 2
  #    -----END CERTIFICATE-----
 
  # The url/ip of a proxy for kapp controller to use when making network
  # requests (optional)
  httpProxy: "http://ip-address:port"
 
  # The url/ip of a tls capable proxy for kapp controller to use when
  # making network requests (optional)
  httpsProxy: "http://ip-address:port"
 
  # A comma delimited list of domain names which kapp controller should
  # bypass the proxy for when making requests (optional)
  noProxy: "localhost,127.0.0.1,kubernetes.default.svc,.svc,cluster.local,.local,195.51.100.0/12"
 
  # A comma delimited list of hostnames for which kapp controller should
  # skip TLS verification (optional)
  #dangerousSkipTLSVerify: "cert-manager-webhook.cert-manager.svc,cert-manager-webhook"
```

Add the repo using tanzu cli: `tanzu package repository add repo --url harbor.jarvis.tanzu/packages/packages:v1.4.0 --namespace private-repo`

## Resources

- [Install the Tanzu CLI and Other Tools](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-install-cli.html)
- [Install and Configure Packages](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.4/vmware-tanzu-kubernetes-grid-14/GUID-packages-index.html)
- [Tanzu Packages Explained](https://beyondelastic.com/2022/01/04/tanzu-packages-explained/)
- [Deploying Tanzu Packages using Tanzu Mission Control Catalog](https://rguske.github.io/post/deploying-tanzu-packages-using-tanzu-mission-control-catalog/)
- [Configure vSphere with Tanzu behind a Proxy plus TKG Extensions](https://beyondelastic.com/2021/06/21/configure-vsphere-with-tanzu-behind-a-proxy-plus-tkg-extensions/)
