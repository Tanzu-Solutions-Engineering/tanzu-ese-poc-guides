# Table of Content

- [Table of Content](#table-of-content)
  - [SMB CSI Driver for Kubernetes](#smb-csi-driver-for-kubernetes)
  - [Preperations](#preperations)
    - [1. Make Images Offline Available](#1-make-images-offline-available)
    - [2. Role-Based-Access-Control](#2-role-based-access-control)
    - [3. CSI-SMB-Driver Installation](#3-csi-smb-driver-installation)
    - [4. Create Image Pull Secret](#4-create-image-pull-secret)
    - [5. Controller Deployment](#5-controller-deployment)
    - [6. DaemonSet Node](#6-daemonset-node)
  - [Installation](#installation)
  - [Testing](#testing)
    - [1. Create SMB Access Secret](#1-create-smb-access-secret)
    - [2. Create PersistentVolume](#2-create-persistentvolume)
    - [3. Create a Deployment to Validate the Access](#3-create-a-deployment-to-validate-the-access)
    - [4. Final Test](#4-final-test)

## SMB CSI Driver for Kubernetes

Source on Github: [csi-driver-smb](https://github.com/kubernetes-csi/csi-driver-smb)

This driver allows Kubernetes to access SMB server on both Linux and Windows nodes, csi plugin name: `smb.csi.k8s.io`. The driver requires existing and already configured SMB server, it supports dynamic provisioning of Persistent Volumes via Persistent Volume Claims by creating a new sub directory under SMB server.

The default implementation is in Namespace `kube-system`. Decide to go with the default or to use a different Namespace like e.g. `csi-smb-provisioner` (namespace creation description in RBAC manifest included).

## Preperations

### 1. Make Images Offline Available

Since the customer environment is completely air-gapped, the necessary images which are used for the `Deployment` as well as for the `DaemonSet` manifests, have to be offline available first, before applying the files.

In this specific customer environment, the images were shared as a tarball (`*.tar`) upfront.

- docker pull k8s.gcr.io/sig-storage/csi-provisioner:v2.2.2
- docker save docker save k8s.gcr.io/sig-storage/csi-provisioner:v2.2.2 > csi-provisioner-v222.tar
- docker pull k8s.gcr.io/sig-storage/livenessprobe:v2.5.0
- docker save k8s.gcr.io/sig-storage/livenessprobe:v2.5.0 > livenessprobe-v250.tar
- docker pull k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.4.0
- docker save k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.4.0 > csi-node-driver-registrar-v240.tar
- docker pull mcr.microsoft.com/k8s/csi/smb-csi:v1.5.0
- docker save mcr.microsoft.com/k8s/csi/smb-csi:v1.5.0 > smb-csi-v150.tar

The customer had to use `docker load -i` in order to have the images local on the jumpstation available. After this step, all images had to be `tag`ged properly first before being `push`ed to the private repository.

Example:

- `docker images`

```shell
REPOSITORY                                                   TAG                       IMAGE ID       CREATED         SIZE
k8s.gcr.io/sig-storage/csi-node-driver-registrar             v2.4.0                    f45c8a305a0b   4 months ago    19.8MB
```

- `docker tag f45c8a305a0b harbor01.jarvis.tanzu/csi-smb-repo/csi-node-driver-registrar:v2.4.0`
- `docker push harbor01.jarvis.tanzu/csi-smb-repo/csi-node-driver-registrar:v2.4.0`

After this step is done, adjust the following Deployment and DaemonSet manifests accordingly to pull the images from the customer Harbor (registry) instance using the `imagePullSecret`.

### 2. Role-Based-Access-Control

Create a new file named `rbac-csi-smb-controller.yaml` in order to create the necessary `ServiceAccount`, `ClusterRole` as well as the `ClusterRoleBinding`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: csi-smb-provisioner
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: csi-smb-controller-sa
  namespace: csi-smb-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: smb-external-provisioner-role
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["csinodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: smb-csi-provisioner-binding
subjects:
  - kind: ServiceAccount
    name: csi-smb-controller-sa
    namespace: csi-smb-provisioner
roleRef:
  kind: ClusterRole
  name: smb-external-provisioner-role
  apiGroup: rbac.authorization.k8s.io
```

### 3. CSI-SMB-Driver Installation

Create a new `CSIdriver` manifest file and apply it:

```yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: smb.csi.k8s.io
spec:
  attachRequired: false
  podInfoOnMount: true
```

### 4. Create Image Pull Secret

In order to allow the TKC Worker-Nodes pulling images from the private registry, the creation of a Kubernetes secret has to be done first.

`kubectl -n csi-smb-provisioner create secret docker-registry harbor-creds --docker-server='harbor.jarvis.tanzu' --docker-username='admin' --docker-password='$PASSWORD'`

The new secret (`imagePullSecrets`) has to be added to the `Deployment` as well as to the `DaemonSet` manifest files (Step 5 & 6).

### 5. Controller Deployment

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: csi-smb-controller
  namespace: csi-smb-provisioner
spec:
  replicas: 2
  selector:
    matchLabels:
      app: csi-smb-controller
  template:
    metadata:
      labels:
        app: csi-smb-controller
    spec:
      dnsPolicy: ClusterFirstWithHostNet
      serviceAccountName: csi-smb-controller-sa
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      tolerations:
        - key: "node-role.kubernetes.io/master"
          operator: "Exists"
          effect: "NoSchedule"
        - key: "node-role.kubernetes.io/controlplane"
          operator: "Exists"
          effect: "NoSchedule"
      containers:
        - name: csi-provisioner
          image: k8s.gcr.io/sig-storage/csi-provisioner:v2.2.2
          args:
            - "-v=2"
            - "--csi-address=$(ADDRESS)"
            - "--leader-election"
          env:
            - name: ADDRESS
              value: /csi/csi.sock
          volumeMounts:
            - mountPath: /csi
              name: socket-dir
          resources:
            limits:
              cpu: 1
              memory: 300Mi
            requests:
              cpu: 10m
              memory: 20Mi
        - name: liveness-probe
          image: k8s.gcr.io/sig-storage/livenessprobe:v2.5.0
          args:
            - --csi-address=/csi/csi.sock
            - --probe-timeout=3s
            - --health-port=29642
            - --v=2
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
          resources:
            limits:
              cpu: 1
              memory: 100Mi
            requests:
              cpu: 10m
              memory: 20Mi
        - name: smb
          image: mcr.microsoft.com/k8s/csi/smb-csi:v1.5.0
          imagePullPolicy: IfNotPresent
          args:
            - "--v=5"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--metrics-address=0.0.0.0:29644"
          ports:
            - containerPort: 29642
              name: healthz
              protocol: TCP
            - containerPort: 29644
              name: metrics
              protocol: TCP
          livenessProbe:
            failureThreshold: 5
            httpGet:
              path: /healthz
              port: healthz
            initialDelaySeconds: 30
            timeoutSeconds: 10
            periodSeconds: 30
          env:
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /csi
              name: socket-dir
          resources:
            limits:
              memory: 200Mi
            requests:
              cpu: 10m
              memory: 20Mi
      imagePullSecrets:
      - name: harbor-creds
      volumes:
        - name: socket-dir
          emptyDir: {}
```

### 6. DaemonSet Node

Adjust the image reference accordingly to also match the image repository as well as to use the `imagePullSecret` like used before.

```yaml
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: csi-smb-node
  namespace: csi-smb-provisioner
spec:
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: csi-smb-node
  template:
    metadata:
      labels:
        app: csi-smb-node
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-node-critical
      tolerations:
        - operator: "Exists"
      containers:
        - name: liveness-probe
          volumeMounts:
            - mountPath: /csi
              name: socket-dir
          image: k8s.gcr.io/sig-storage/livenessprobe:v2.5.0
          args:
            - --csi-address=/csi/csi.sock
            - --probe-timeout=3s
            - --health-port=29643
            - --v=2
          resources:
            limits:
              memory: 100Mi
            requests:
              cpu: 10m
              memory: 20Mi
        - name: node-driver-registrar
          image: k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.4.0
          args:
            - --csi-address=$(ADDRESS)
            - --kubelet-registration-path=$(DRIVER_REG_SOCK_PATH)
            - --v=2
          livenessProbe:
            exec:
              command:
                - /csi-node-driver-registrar
                - --kubelet-registration-path=$(DRIVER_REG_SOCK_PATH)
                - --mode=kubelet-registration-probe
            initialDelaySeconds: 30
            timeoutSeconds: 15
          env:
            - name: ADDRESS
              value: /csi/csi.sock
            - name: DRIVER_REG_SOCK_PATH
              value: /var/lib/kubelet/plugins/smb.csi.k8s.io/csi.sock
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: registration-dir
              mountPath: /registration
          resources:
            limits:
              memory: 100Mi
            requests:
              cpu: 10m
              memory: 20Mi
        - name: smb
          image: mcr.microsoft.com/k8s/csi/smb-csi:v1.5.0
          imagePullPolicy: IfNotPresent
          args:
            - "--v=5"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--nodeid=$(KUBE_NODE_NAME)"
            - "--metrics-address=0.0.0.0:29645"
          ports:
            - containerPort: 29643
              name: healthz
              protocol: TCP
          livenessProbe:
            failureThreshold: 5
            httpGet:
              path: /healthz
              port: healthz
            initialDelaySeconds: 30
            timeoutSeconds: 10
            periodSeconds: 30
          env:
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
            - name: KUBE_NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /csi
              name: socket-dir
            - mountPath: /var/lib/kubelet/
              mountPropagation: Bidirectional
              name: mountpoint-dir
          resources:
            limits:
              memory: 200Mi
            requests:
              cpu: 10m
              memory: 20Mi
      imagePullSecrets:
      - name: harbor-creds
      volumes:
        - hostPath:
            path: /var/lib/kubelet/plugins/smb.csi.k8s.io
            type: DirectoryOrCreate
          name: socket-dir
        - hostPath:
            path: /var/lib/kubelet/
            type: DirectoryOrCreate
          name: mountpoint-dir
        - hostPath:
            path: /var/lib/kubelet/plugins_registry/
            type: DirectoryOrCreate
          name: registration-dir
```

## Installation

Now that the preperations are done, `apply` all the created manifest files to Kubernetes. 

- kubectl apply -f rbac-csi-smb-controller.yaml
- kubectl apply -f csi-smb-driver.yaml
- kubectl apply -f csi-smb-controller.yaml
- kubectl apply -f csi-smb-node.yaml

## Testing

The foloowing test will be done in Namespace `smb-test`.

`kubectl create ns smb-test`.

### 1. Create SMB Access Secret

In order to authenticate to a SMB share, create a `secret`first:

`kubectl -n smb-test create secret generic smb-creds --from-literal username=USERNAME --from-literal password="PASSWORD"`

> add --from-literal domain=DOMAIN-NAME for domain support

### 2. Create PersistentVolume

The deployment, which will be created in the next step, will have the SMB share accessible through a `PersistentVolume` and the associated `PersistentVolumeClaim`.

Beginning with the `PersistentVolume`:

> Edit `source` in `volumeAttributes`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-smb
  namespace: smb-test
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  mountOptions:
    - dir_mode=0777
    - file_mode=0777
    - vers=3.0
  csi:
    driver: smb.csi.k8s.io
    readOnly: false
    volumeHandle: unique-volumeid  # make sure it's a unique id in the cluster
    volumeAttributes:
      source: "//smb-server-address/sharename"
    nodeStageSecretRef:
      name: smb-creds
      namespace: smb-test
```

Create the PersitentVolume:

`kubectl apply -f pvc-smb-static.yaml`

Continueing with the `PersitstentVolumeClaim` to bound the volume.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-smb
  namespace: smb-test
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  volumeName: pv-smb
  storageClassName:
```

Create the `pvc` by executing `kubectl apply -f pvc-smb.yaml`.

Validate that the `STATUS` is in state `Bound`:

```shell
kubectl -n smb-test get pv,pvc

NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
persistentvolume/pv-smb   50Gi       RWX            Retain           Bound    smb-test/pvc-smb                           2m22s

NAME                            STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc-smb   Bound    pv-smb   50Gi       RWX                           20s
```

### 3. Create a Deployment to Validate the Access

Make the `nginx` image (`image: mcr.microsoft.com/oss/nginx/nginx:1.19.5`) offline available:

- `docker pull mcr.microsoft.com/oss/nginx/nginx:1.19.5`
- `docker save mcr.microsoft.com/oss/nginx/nginx:1.19.5 > nginx-1195.tar`
- `docker load -i nginx-1195.tar`
- `docker tag ...`
- `docker push ...`

Create the Nginx deployment manifest file `nginx-deployment.yaml`

> Adjust the `image:` section!

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: deployment-smb
  namespace: smb-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      name: deployment-smb
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
        - name: deployment-smb
          image: mcr.microsoft.com/oss/nginx/nginx:1.19.5
          command:
            - "/bin/bash"
            - "-c"
            - set -euo pipefail; while true; do echo $(date) >> /mnt/smb/outfile; sleep 1; done
          volumeMounts:
            - name: smb
              mountPath: "/mnt/smb"
              readOnly: false
      volumes:
        - name: smb
          persistentVolumeClaim:
            claimName: pvc-smb
  strategy:
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
    type: RollingUpdate
```

Create the deployment with `kubectl apply -f deployment.yaml`.

### 4. Final Test

Validate that the volume was mounted and the Pod has access to the SMB share: 
`kubectl exec -it nginx-smb -- df -h`

```shell
kubectl -n smb-test exec -it deployment-smb-547588d59c-8q9kl -- df -h
Filesystem              Size  Used Avail Use% Mounted on
overlay                  16G   11G  4.5G  70% /
tmpfs                    64M     0   64M   0% /dev
tmpfs                   2.0G     0  2.0G   0% /sys/fs/cgroup
//dc.jarvis.tanzu/data   90G   65G   26G  72% /mnt/smb
/dev/root                16G   11G  4.5G  70% /etc/hosts
shm                      64M     0   64M   0% /dev/shm
tmpfs                   2.0G   12K  2.0G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                   2.0G     0  2.0G   0% /proc/acpi
tmpfs                   2.0G     0  2.0G   0% /sys/firmware
```

Write a file into the folder:

`kubectl -n smb-test exec -it deployment-smb-547588d59c-8q9kl -- touch /mnt/smb/test.txt`

Show the folder content:

```shell
kubectl -n smb-test exec -it deployment-smb-547588d59c-8q9kl -- ls -la /mnt/smb
total 12
drwxrwxrwx 2 root root    0 Mar 31 14:05 .
drwxr-xr-x 1 root root 4096 Mar 31 14:06 ..
-rwxrwxrwx 1 root root    0 Mar 31 14:10 test.txt
```
