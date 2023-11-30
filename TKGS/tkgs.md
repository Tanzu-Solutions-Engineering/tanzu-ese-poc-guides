# Tanzu Kubernetes Grid Service (TKGs)

## Table of Content

- [Tanzu Kubernetes Grid Service (TKGs)](#tanzu-kubernetes-grid-service-tkgs)
  - [Table of Content](#table-of-content)
  - [Prerequisites](#prerequisites)
    - [vSphere Networking](#vsphere-networking)
    - [NSX Networking](#nsx-networking)
    - [Licensing](#licensing)
    - [Content Library](#content-library)
    - [Domain Activations](#domain-activations)
  - [Binaries](#binaries)
  - [CLIs](#clis)
    - [Kubectl](#kubectl)
    - [Kubectl vSphere](#kubectl-vsphere)
  - [Networking](#networking)
    - [Load Balancer Feature Comparison](#load-balancer-feature-comparison)
    - [Supervisor Networking](#supervisor-networking)
    - [Tanzu Kubernetes Cluster Networking](#tanzu-kubernetes-cluster-networking)
    - [DNS Resolve flow](#dns-resolve-flow)
  - [Tanzu Kubernetes Cluster](#tanzu-kubernetes-cluster)
    - [Available Virtual Machine Classes](#available-virtual-machine-classes)
    - [Example Manifest - Advanced](#example-manifest---advanced)
    - [Example Manifest - Simple](#example-manifest---simple)
    - [Pod Security Policy](#pod-security-policy)
  - [Troubleshooting](#troubleshooting)
    - [Support Bundle](#support-bundle)
    - [Show Kubernetes vSphere Objects](#show-kubernetes-vsphere-objects)
    - [vCenter Server Appliance Service Troubleshooting](#vcenter-server-appliance-service-troubleshooting)
    - [DCLI](#dcli)
    - [SSH to Supervisor Control Plane Node](#ssh-to-supervisor-control-plane-node)
    - [Operating with the wcp privileged wcp-vmop-user-domain-cXXXX user](#operating-with-the-wcp-privileged-wcp-vmop-user-domain-cxxxx-user)
    - [SSH into a Tanzu Kubernetes Cluster Nodes using the tkc-ssh-secret](#ssh-into-a-tanzu-kubernetes-cluster-nodes-using-the-tkc-ssh-secret)
    - [SSH into a Tanzu Kubernetes Cluster Nodes using a Jumpbox-Pod](#ssh-into-a-tanzu-kubernetes-cluster-nodes-using-a-jumpbox-pod)
  - [Jumpbox](#jumpbox)
    - [Docker](#docker)
  - [Proxy](#proxy)
    - [PhotonOS](#photonos)
    - [Ubuntu](#ubuntu)
    - [CentOS/RHEL](#centosrhel)
  - [Demo App](#demo-app)
  - [Helpful Resources](#helpful-resources)

## Prerequisites

### vSphere Networking

- VMware vSphere 7 Enterprise Plus and Tanzu Edition license keys
- HA and DRS enabled vSphere cluster (fully automated)
- Shared storage connected to all ESXi Hosts
- NTP configured and working on all components
- DNS server configured, reachable, and working
- 5x free IPs in a row on the management network for the Supervisor Control Plane
- VDS version 7.0.0
- Separate and fully routed workload and management network on different VLANs (third Frontend network is recommended)

### NSX Networking

- VMware vSphere 7 Enterprise Plus and Tanzu Edition license keys
- HA and DRS enabled vSphere cluster (fully automated)
- Shared storage connected to all ESXi hosts
- NTP configured and working on all components
- DNS server configured, reachable, and working
- 5x free IPs in a row on the management network for the Supervisor Control Plane
- VDS version 7.0.0
- at least 1x VLAN and subnet for the overlay network (2x if you don’t have a separate vmnic to spare)
- NSX-T 3.x with a working overlay network
- at least 1x NSX Manager and 1x Edge node (LARGE)
- MTU size of at least 1600
- 2x /27 subnets for ingress and egress traffic

### Licensing

- [Licensing for vSphere with Tanzu](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-9A190942-BDB1-4A19-BA09-728820A716F2.html)
  - VMware vSphere 7 Enterprise Plus with Add-on for Kubernetes license assigned to each host from the Supervisor cluster
  - Tanzu license for the Supervisor cluster
  - You can assign a Tanzu edition license key to multiple Supervisor clusters at a time

- [Prerequisites for Configuring vSphere with Tanzu on a Cluster](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-EE236215-DA4D-4579-8BEB-A693D1882C77.html)
  - vSphere Cluster with at least 3 hosts
    - 4 hosts with vSAN
    - HA and DRS (fully automated) is required
  - vSAN File Services for persistent volumes in ReadWriteMany mode
  - User account has the Modify cluster-wide configuration permission to enable *Workload Management*

Table I: Choose and configure the networking stack

| Functionality | NSX-T Networking | vDS Networking | Comment |
| :--: | :--: | :--: | :--: |
| vSphere Pods  | Yes | No | |
| Tanzu Kubernetes clusters  | Yes | Yes | |
| Embedded Harbor Registry  | Yes | No | |
| Load Balancing  | Yes | Yes, by installing and configuring the NSX Advanced Load Balancer or the HAProxy load balancer. | vSphere with Tanzu supports the NSX Advanced Load Balancer with vSphere 7 U2 and later.

### Content Library

- Subscribed Content Library
  - Subscription URL: <https://wp-content.vmware.com/v2/latest/lib.json>
  - OVF security policy: **OVF default policy**

- Local Content Library
  - Subscription URL: <https://wp-content.vmware.com/v2/latest/lib.json>
  - OVF security policy: **OVF default policy**
  - Synchronization Option
    - Immediately
    - When needed (two options to synchronize: entire library or a single item)

### Domain Activations

- *.tmc.cloud.vmware.com
- *.console.cloud.vmware.com
- *.cloud.vmware.com
- *.projects.registry.vmware.com
- *.registry.vmware.com
- *.registry.pivotal.io
- *.github.com
- *.githubusercontent.com
- *.vmware.com
- *.docker.io
- *.docker.com

## Binaries

Since vSphere with Tanzu (Workload Management) is a vSphere Cluster feature, it's not necessary to download any additional installation binaries.

- [Download vSphere via VMware Customer Connect](https://customerconnect.vmware.com/de/downloads/info/slug/datacenter_cloud_infrastructure/vmware_vsphere/7_0)

## CLIs

### Kubectl

[Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

- Install kubectl binary with curl on Linux

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

- Download a specific `kubectl` version

```shell
curl -LO https://dl.k8s.io/release/v1.23.0/bin/linux/amd64/kubectl
```

- Install using native package management

```shell
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubectl
```

- Autocompletion `kubectl`

```shell
source <(kubectl completion bash)

echo "source <(kubectl completion bash)" >> ~/.bashrc
```

- Set alias for kubectl to just `k`

```shell
echo "alias k='kubectl'" >> ~/.bashrc

complete -F __start_kubectl k
```

### Kubectl vSphere

[Download and Install the Kubernetes CLI Tools for vSphere](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-0F6E45C4-3CB1-4562-9370-686668519FCA.html)

<img src="images/tkgs_kubectl_vsphere_cli.png" align="center" height="300px"/>

- Password as an environment variable:

```bash
export KUBECTL_VSPHERE_PASSWORD=P@ssw0rd

kubectl vsphere login --vsphere-username administrator@vsphere.local --server=(IP or FQDN)
```

## Networking

### Load Balancer Feature Comparison

<img src="images/tkgs_loadbalancer_comparison.png"/>

### Supervisor Networking

<img src="images/tkgs_nsx_architecture.png"/>

- [Supervisor Cluster Networking](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-B156CDA6-B056-4D1C-BBC5-07D1A701E402.html)

<img src="images/tkgs_supervisor_networking.png"/>

### Tanzu Kubernetes Cluster Networking

- [Tanzu Kubernetes Cluster Networking](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-A7756D67-0B95-447D-A645-E2A384BF8135.html)

The table summarizes Tanzu Kubernetes cluster networking features and their implementation.

| Endpoint | Provider |	Description |
| :-- | :--: | --: |
| Pod connectivity |Antrea or Calico | Container network interface for pods. Antrea uses Open vSwitch. Calico uses the Linux bridge with BGP |
| Service type: ClusterIP |	Antrea or Calico | Default Kubernetes service type that is only accessible from within the cluster |
Service type: NodePort | Antrea or Calico | Allows external access through a port opened on each worker node by the Kubernetes network proxy |
Service type: LoadBalancer | NSX-T load balancer, NSX Advanced Load Balancer, HAProxy | For NSX-T, one virtual server per service type definition. For NSX Advanced Load Balancer, refer to that section of this documentation Note: Some load balancing features may not be available with HAProxy, such as support for static IPs |
| Cluster ingress | Third-party ingress controller | Routing for inbound pod traffic; you can use any third-party ingress controller |
| Network policy | Antrea or Calico | Controls what traffic is allowed to and from selected pods and network endpoints. Antrea uses Open vSwitch. Calico uses Linux IP tables |

### DNS Resolve flow

<img src="images/tkgs_dns_resolve_flow.png"/>

## Tanzu Kubernetes Cluster

### Available Virtual Machine Classes

[Virtual Machine Classes for Tanzu Kubernetes Clusters](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-7351EEFF-4EF0-468F-A19B-6CEA40983D3D.html)

| Class | CPU | Memory (GB) | Reserved CPU and Memory |
| :--: | :--: | :--: | :--: |
| guaranteed-8xlarge | 32 | 128 | Yes |
| best-effort-8xlarge | 32 | 128 | No |
| guaranteed-4xlarge | 16 | 128 | Yes |
| best-effort-4xlarge | 16 | 128 | No |
| guaranteed-2xlarge | 8 | 64 | Yes |
| best-effort-2xlarge | 8 | 64 | No |
| guaranteed-xlarge | 4 | 32 | Yes |
| best-effort-xlarge | 4 | 32 | No |
| guaranteed-large | 4 | 16 | Yes |
| best-effort-large | 4 | 16 | No |
| guaranteed-medium | 2 | 8 | Yes |
| best-effort-medium | 2 | 8 | No |
| guaranteed-small | 2 | 4 | Yes |
| best-effort-small | 2 | 4 | No |
| guaranteed-xsmall | 2 | 2 | Yes |
| best-effort-xsmall | 2 | 2 | No |

### Example Manifest - Advanced

[Provisioning Tanzu Kubernetes Clusters Using the Tanzu Kubernetes Grid Service v1alpha2 API](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-3B2102E6-D9AA-4FE6-B3AA-60B450BE8491.html)

Full Example Manifest file to deploy a Tanzu Kubernetes Cluster:

```shell
apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
metadata:
  name: tkgs-v2-cluster-custom
  namespace: tkgs-cluster-ns
spec:
  topology:
    controlPlane:
      replicas: 3
      vmClass: guaranteed-medium
      storageClass: vwt-storage-policy
      volumes:
        - name: etcd
          mountPath: /var/lib/etcd
          capacity:
            storage: 4Gi
      tkr:
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
    nodePools:
    - name: worker-nodepool-a1
      replicas: 3
      vmClass: guaranteed-large
      storageClass: vwt-storage-policy
      volumes:
        - name: containerd
          mountPath: /var/lib/containerd
          capacity:
            storage: 16Gi
      tkr:
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
    - name: worker-nodepool-a2
      replicas: 2
      vmClass: guaranteed-medium
      storageClass: vwt-storage-policy
      tkr:
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
    - name: worker-nodepool-a3
      replicas: 1
      vmClass: guaranteed-small
      storageClass: vwt-storage-policy
      tkr:
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
  settings:
    storage:
      defaultClass: vwt-storage-policy
    network:
      cni:
        name: antrea
      services:
        cidrBlocks: ["198.53.100.0/16"]
      pods:
        cidrBlocks: ["192.0.5.0/16"]
      serviceDomain: managedcluster.local
      proxy:
        httpProxy: http://<user>:<pwd>@<ip>:<port>
        httpsProxy: http://<user>:<pwd>@<ip>:<port>
        noProxy: [10.246.0.0/16,192.168.144.0/20,192.168.128.0/20]
      trust:
        additionalTrustedCAs:
          - name: CompanyInternalCA-1
            data: LS0tLS1C...LS0tCg==
          - name: CompanyInternalCA-2
            data: MTLtMT1C...MT0tPg==
```

### Example Manifest - Simple

```shell
apiVersion: run.tanzu.vmware.com/v1alpha1
kind: TanzuKubernetesCluster
metadata:
  name:
  namespace:
spec:
  distribution:
    version: v1.21.6
  settings:
    network:
      cni:
        name: antrea
      pods:
        cidrBlocks:
        - 193.0.2.0/16
      services:
        cidrBlocks:
        - 195.51.100.0/12
  topology:
    controlPlane:
      class: best-effort-small
      count: 3
      storageClass:
    workers:
      class: best-effort-medium
      count: 3
      storageClass:
```

### Pod Security Policy

[Using Pod Security Policies with Tanzu Kubernetes Clusters](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-CD033D1D-BAD2-41C4-A46F-647A560BAEAB.html)

> Tanzu Kubernetes Grid Service provisions Tanzu Kubernetes clusters with the PodSecurityPolicy Admission Controller enabled. This means that pod security policy is required to deploy workloads. Cluster administrators can deploy pods from their user account to any namespace, and from service accounts to the kube-system namespace. For all other use cases, you must explicitly bind to a PodSecurityPolicy object. Clusters include default pod security policies that you can bind to, or create your own.

```shell
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: psp:privileged
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - vmware-system-privileged
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: all:psp:privileged
roleRef:
  kind: ClusterRole
  name: psp:privileged
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts
  apiGroup: rbac.authorization.k8s.io
```

## Troubleshooting

### Support Bundle

[Collect a Support Bundle for Tanzu Kubernetes Clusters](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-6390E04F-E511-4DC9-B941-FDD0979F2A2A.html)

To troubleshoot Tanzu Kubernetes cluster errors, you can run a utility to collect a diagnostic log bundle. Obtain the utility from here: [KB80949](https://kb.vmware.com/s/article/80949)

### Show Kubernetes vSphere Objects

- `kubectl get virtualmachineimages`
- `kubectl get virtualmachineclasses`
- `kubectl get tanzukubernetescluster`
  - `kubectl get tkc`

### vCenter Server Appliance Service Troubleshooting

Show service status using `vmon-cli`

- List all services: `vmon-cli --list`
- Show status of a service: `vmon-cli --status`
- List dependencies of a service on other services:

```shell
vmon-cli --list-dependencies wcp

cis-license
eam
lookupsvc
sts
trustmanagement
vapi-endpoint
vmware-vpostgres
vpxd
```

- Start a service: `vmon-cli --start`
- Stop a services: `vmon-cli --stop`
- Restart a service: `vmon-cli --restart`

Show service status using `service-control`

```shell
service-control --help
usage: service-control [-h] [--start] [--stop] [--restart] [--status] [--list] [--list-services] [--list-dependencies] [--vmon-profile VMON_PROFILE] [--all] [--ignore] [--verbose] ...

Perform operation on VMware services. By default the services selected are based on current startup profile. This can be overridden by using --all and --vmon-profile options.

positional arguments:
  services              Services on which to operate

optional arguments:
  -h, --help            show this help message and exit
  --start               Perform start operation on VMware services.
  --stop                Perform stop operation on VMware services.
  --restart             Perform restart operation on a single VMware service.
  --status              Get running status of VMware services.
  --list                List all controllable VMware services. This option will soon be deprecated. Please use --list-services.
  --list-services       Lists all controllable VMware services.
  --list-dependencies   Lists dependencies for a given service.
  --vmon-profile VMON_PROFILE
                        Start/Stop services associated with given profile name.
  --all                 Start/Stop all VMware services i.e. core and the default mgmt services).
  --ignore              Ignore errors. Continue with given operation even if errors occur.
  --verbose             Display detailed information.
```

### DCLI

Check vSphere cluster compatibility.

`dcli com vmware vcenter namespacemanagement clustercompatibility list`

`dcli com vmware vcenter namespacemanagement supportbundle`

**List/delete integrated Harbor registry**

It can happen that disabling/deleting the Integrated Harbor Registry ends in an endless loop and the vSphere objects weren't deleted. In this case, the vCenter `dcli` can help.

- List the iHarbor instance:

`dcli +show-unreleased +skip +username administrator@jarvis.tanzu +password 'PASSWORD' com vmware vcenter content registries harbor list`

- delete the iHarbor instance using `dcli`

`dcli +show-unreleased +skip +username administrator@jarvis.tanzu +password 'PASSWORD' com vmware vcenter content registries harbor delete --registry Harbor-57b478ea...`

### SSH to Supervisor Control Plane Node

`ssh` to your vCenter Server Appliance and execut `shell` when prompted.

```shell
# change into the dir where the pwd decryption file is stored
cd /usr/lib/vmware-wcp/

# execute the file to get the pwd
./decryptK8Pwd.py
```

Connect to the Supervisor Node using the decrypted password:

```shell
Connected to PSQL

Cluster: domain-c7:a4b59b96-9e94-4158-95ff-e1dfeb58c6bf
IP: 10.10.13.30
PWD: DHinqsmNdedRcpMEOvlYtfSROSO6C6Bf4VC9mpzIE6QclX2vQqtJuG4O12FDp4Ox3LVxhYTB81Ntv2DM+g4daJTbBdJKw3nTcO/cFSsH7xCZayQNzWwVMdNYpNo5OSS4Gcnex4nGsauJO5HNz+a+AzJjfkICLLRLwCJHxi3tssk=
```

### Operating with the wcp privileged wcp-vmop-user-domain-cXXXX user

In TKGs, you don't have permissions to e.g. configure, or even power-on, power-off the SupervisorVMs or the TKC-VMs. Not even the administrator@vsphere.local can. Even though, there's a way to use the `wcp-vmop-user-domain` user in order to execute those operations.

1. Login into a SupervisorVM (see description above)
2. execute the foloowing commands:

```shell
$ kubectl get secrets wcp-vmop-sa-vc-auth -n vmware-system-vmop -o jsonpath='{.data}' |jq

{
  "password": "PTJ0VC06J15GM01tOEcwfmNd=",
  "username": "d2NwLXZtb3AtdXNlci1kb21haW4tYzIwMzMtN2FjODUzYzItNTIwNy00Y2I0LThhNTItZjQwOGRjNGQ2M2Q4QG1hcms1MC5s="
}
```

3. Decrypt the base64 encoded data to get the username and password

```shell
$ echo d2NwLXZtb3AtdXNlci1kb21haW4tYzIwMzMtN2FjODUzYzItNTIwNy00Y2I0LThhNTItZjQwOGRjNGQ2M2Q4QG1hcms1MC5sYWI=d2NwLXZtb3AtdXNlci1kb21haW4tYzIwMzMtN2FjODUzYzItNTIwNy00Y2I0LThhNTItZjQwOGRjNGQ2M2Q4QG1hcms1MC5s= | base64 -d

$ echo PTJ0VC06J15GM01tOEcwfmNd= | base64 -d
```

4. Login into vSphere using the new credentials, which allows you to execute operations on SupervisorVMs as well as on TKC's

![wcp-vmop-sa-vc](images/tkgs_wcp-vmop-sa-vc_user.png)

### SSH into a Tanzu Kubernetes Cluster Nodes using the tkc-ssh-secret

Reference: [Docs](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-37DC1DF2-119B-4E9E-8CA6-C194F39DDEDA.html)

1. Login to the Supervisor-Cluster

`kubectl vsphere login --insecure-skip-tls-verify --vsphere-username administrator@mark50.lab --server=mark50.jarvis.tanzu`

2. Get the exact secret-name out of the appropriate vSphere-Namespace

```shell
kubectl -n mark50-ns-1 get secrets

NAME                               TYPE                                  DATA   AGE
default-token-j77xs                kubernetes.io/service-account-token   3      121d
mark50-tkc-1-antrea                kubernetes.io/tls                     3      10d
mark50-tkc-1-auth-svc-cert         kubernetes.io/tls                     3      10d
mark50-tkc-1-ca                    Opaque                                2      10d
mark50-tkc-1-ccm-token-6dpjd       kubernetes.io/service-account-token   3      10d
mark50-tkc-1-control-plane-5mmc9   cluster.x-k8s.io/secret               1      10d
mark50-tkc-1-control-plane-jv8xz   cluster.x-k8s.io/secret               1      10d
mark50-tkc-1-control-plane-nnnpp   cluster.x-k8s.io/secret               1      10d
mark50-tkc-1-encryption            Opaque                                1      10d
mark50-tkc-1-etcd                  Opaque                                2      10d
mark50-tkc-1-extensions-ca         kubernetes.io/tls                     3      10d
mark50-tkc-1-kubeconfig            Opaque                                1      10d
mark50-tkc-1-metrics-server-cert   kubernetes.io/tls                     3      10d
mark50-tkc-1-node-zljq9-n8czc      cluster.x-k8s.io/secret               1      122m
mark50-tkc-1-proxy                 Opaque                                2      10d
mark50-tkc-1-pvcsi-token-t7xbb     kubernetes.io/service-account-token   3      10d
mark50-tkc-1-sa                    Opaque                                2      10d
mark50-tkc-1-ssh                   kubernetes.io/ssh-auth                1      10d
mark50-tkc-1-ssh-password          Opaque                                1      10d
mark50-tkc-1-worker-j4v9z-sst4j    cluster.x-k8s.io/secret               1      12s
```

3. Get the base64 encoded ssh password for the TKC nodes

```shell
k -n mark50-ns-1 get secret mark50-tkc-1-ssh-password -o yaml

apiVersion: v1
data:
  ssh-passwordkey: aTcxWG9rbWVpdkllU01xaEJqRFkzTzVnTUZnanZkRXlmL090Q296cFUy=
kind: Secret
metadata:
  creationTimestamp: "2022-04-29T07:18:50Z"
  name: mark50-tkc-1-ssh-password
  namespace: mark50-ns-1
  ownerReferences:
  - apiVersion: run.tanzu.vmware.com/v1alpha2
    kind: TanzuKubernetesCluster
    name: mark50-tkc-1
    uid: 69cac0d0-fd81-4aa6-8f80-011663b8ecb9
  resourceVersion: "89540146"
  selfLink: /api/v1/namespaces/mark50-ns-1/secrets/mark50-tkc-1-ssh-password
  uid: e61530b9-60d1-4475-8301-ccae21028e8d
type: Opaque
```

4. Decode the `ssh-passwordkey`

`echo aTcxWG9rbWVpdkllU01xaEJqRFkzTzVnTUZnanZkRXlmL090Q296cFUy= | base64 -d`

5. `ssh` into the desired TKC node using e.g. the IP (`k -n mark50-ns-1 get virtualmachine -o wide`) and the `vmware-system-user`

### SSH into a Tanzu Kubernetes Cluster Nodes using a Jumpbox-Pod

1. Connect to the Supervisor Cluster.

`kubectl vsphere login --insecure-skip-tls-verify --vsphere-username administrator@mark50.lab --server=mark50.jarvis.tanzu`

2. Specify target TKC Namespace and switch context:

```shell
export NAMESPACE=

kubectl config use-context $NAMESPACE
```

3. View the TKGS-CLUSTER-NAME-ssh secret object:

`kubectl get secrets`


4. Create a vSphere Pod using the following jumpbox.yaml

Replace the namespace value YOUR-NAMESPACE with the vSphere Namespace where the target cluster is provisioned. Replace the secretName value YOUR-CLUSTER-NAME-ssh with name of the target cluster.

```shell
kubectl create -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: jumpbox
  namespace: YOUR-NAMESPACE     #REPLACE
spec:
  containers:
  - image: "photon:3.0"
    name: jumpbox
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "yum install -y openssh-server; mkdir /root/.ssh; cp /root/ssh/ssh-privatekey /root/.ssh/id_rsa; chmod 600 /root/.ssh/id_rsa; while true; do sleep 30; done;" ]
    volumeMounts:
      - mountPath: "/root/ssh"
        name: ssh-key
        readOnly: true
    resources:
      requests:
        memory: 2Gi
  volumes:
    - name: ssh-key
      secret:
        secretName: YOUR-CLUSTER-NAME-ssh     #REPLACE
EOF
```

5. Create an environment variable with the IP address of the target cluster node by running the following set of commands:


`kubectl get virtualmachines`

`export VMNAME=`

`export VMIP=$(kubectl -n $NAMESPACE get virtualmachine/$VMNAME -o jsonpath='{.status.vmIp}')`

`kubectl exec -it jumpbox  /usr/bin/ssh vmware-system-user@$VMIP`

## Jumpbox

- [Download VMware PhotonOS](https://github.com/vmware/photon/wiki/Downloading-Photon-OS)
- [Create a Linux Jump Host VM](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-6DE3E58B-6AE3-4ABD-ABCC-2221B4230908.html)
- [Netshoot Virtual Appliance](https://github.com/josemzr/netshoot-virtual-appliance)
- [Demo Appliance for Tanzu Kubernetes Grid](https://flings.vmware.com/demo-appliance-for-tanzu-kubernetes-grid)

### Docker

[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

- Uninstall old versions: `sudo apt-get remove docker docker-engine docker.io containerd runc`
- Set up the repository:

```bash
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

- Add Docker’s official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

- Install Docker Engine

```shell
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

- Manage Docker as a non-root user

```bash
sudo groupadd docker

sudo usermod -aG docker $USER
```

[Install Docker Engine on RHEL](https://docs.docker.com/engine/install/rhel/)

- Uninstall old versions:

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc
```

or

```bash
sudo yum remove docker-ce docker-ce-cli containerd.io

sudo rm -rf /var/lib/docker

sudo rm -rf /var/lib/containerd
```

- Install using the repository

```bash
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/rhel/docker-ce.repo
```

- Install Docker Engine

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

- Install a specific version of Docker Engine

```bash
yum list docker-ce --showduplicates | sort -r
```

- Start Docker

```bash
sudo systemctl start docker
```

- Manage Docker as a non-root user

```bash
sudo groupadd docker

sudo usermod -aG docker $USER
```

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

## Demo App

Hackazon Shop App with Ingress

```shell
kubectl -n hackazon apply -f - <<EOF
kind: Deployment
apiVersion: apps/v1
metadata:
  name: hackazon-shop
  labels:
    app: hackazon-shop
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hackazon-shop
  template:
    metadata:
      labels:
        app: hackazon-shop
    spec:
      containers:
      - name: hackazon-shop
        image: projects.registry.vmware.com/tanzu_ese_poc/hackazon:1.0
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
---
kind: Service
apiVersion: v1
metadata:
  name: hackazon-svc-lb
  labels:
    app: hackazon-shop
    svc: hackazon-svc-lb
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: hackazon-shop
  type: LoadBalancer
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hackazon-ingress
  labels:
    app: hackazon-shop
#  annotations:
#   kubernetes.io/ingress.class: avi
spec:
  rules:
  - host: hackazon.avi.jarvis.tanzu
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: hackazon-svc-lb
            port:
              number: 80
EOF
```

Ghost Web-App with `pvc`:

```shell
kubectl -n ghost create -f - << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog
  namespace: ghost
  labels:
    app: blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blog
  template:
    metadata:
      labels:
        app: blog
    spec:
      containers:
      - name: blog
        image: projects.registry.vmware.com/tanzu_ese_poc/ghost:4.34.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 2368
        env:
        - name: url
          value: http://my-blog.corp.local
        volumeMounts:
        - mountPath: /var/lib/ghost/content
          name: content
      volumes:
      - name: content
        persistentVolumeClaim:
          claimName: blog-content
---
apiVersion: v1
kind: Service
metadata:
  labels:
    name: blog
  name: blog
  namespace: ghost
spec:
  ports:
    - port: 80
      targetPort: 2368
  selector:
    app: blog
  type: LoadBalancer
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: blog-content
  namespace: ghost
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5Gi
EOF
```

## Helpful Resources

- [Common issues with a vSphere with Tanzu Cluster deployment stuck in Configuring state (83060)](https://kb.vmware.com/s/article/83060?lang=en_US)
- [Tanzu Proof of Concept Guide](https://core.vmware.com/resource/tanzu-proof-concept-guide#poc-guide-overview)
- [Verify and Troubleshoot vSphere 7 with Kubernetes](https://beyondelastic.com/2020/07/17/verify-and-troubleshoot-vsphere-7-with-kubernetes/)
- [vSphere with Tanzu - SupervisorControlPlaneVM stucks in state NotReady](https://rguske.github.io/post/vsphere-with-tanzu-supervisor-control-plane-vm-stucks-in-state-not-ready/)
- [TKGs w ALB Quickstart Guide](https://core.vmware.com/resource/vsphere-tanzu-and-nsx-advanced-load-balancer-quick-start-guide-v703)
- [TKGs Troubleshooting Part I](https://core.vmware.com/blog/tanzu-kubernetes-grid-service-troubleshooting-deep-dive-part-1)
- [TKGs Troubleshooting Part II](https://core.vmware.com/blog/tanzu-kubernetes-grid-service-troubleshooting-deep-dive-part-2)
- [TKGs Troubleshooting Part III](https://core.vmware.com/blog/tanzu-kubernetes-grid-service-troubleshooting-deep-dive-part-3)
