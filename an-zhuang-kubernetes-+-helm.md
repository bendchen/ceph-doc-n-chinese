# 安装（KUBERNETES + HELM）

## 安装（KUBERNETES + HELM）

通过[ceph-helm](https://github.com/ceph/ceph-helm/)项目，您可以在Kubernetes环境中部署Ceph。本文档假定Kubernetes环境可用。

### 当前限制

> * 公共网络和集群网络必须相同
> * 如果存储类用户ID不是admin，则必须在Ceph集群中手动创建该用户并在Kubernetes中创建其秘密
> * ceph-mgr只能运行1个副本

### 安装并启动

可以按照以下顺序安装。

Helm通过从本地Kubernetes配置文件中读取找到Kubernetes集群；确保已下载该文件并可供`helm`客户端访问。

必须为您的Kubernetes集群配置并运行Tiller服务器，并且本地Helm客户端必须已连接到该服务器。查看Helm文档的[init](https://github.com/kubernetes/helm/blob/master/docs/helm/helm_init.md)可能会有所帮助。要在本地运行Tiller并将Helm连接到它，请运行：

```text
$ helm init
```

ceph-helm项目默认情况下使用本地Helm存储库来存储图表。要启动本地Helm存储库服务器，请运行：

```text
$ helm serve &
$ helm repo add local http://localhost:8879/charts
```

### 将CEPH-HELM添加到HELM本地存储库

```text
$ git clone https://github.com/ceph/ceph-helm
$ cd ceph-helm/ceph
$ make
```

### 配置您的头孢集群

创建一个`ceph-overrides.yaml`将包含您的Ceph配置的。该文件可能存在于任何地方，但是对于该文档，将假定其位于用户的主目录中：

```text
$ cat ~/ceph-overrides.yaml
network:
  public:   172.21.0.0/20
  cluster:   172.21.0.0/20

osd_devices:
  - name: dev-sdd
    device: /dev/sdd
    zap: "1"
  - name: dev-sde
    device: /dev/sde
    zap: "1"

storageclass:
  name: ceph-rbd
  pool: rbd
  user_id: k8s
```

注意 

如果未设置日志，它将与设备位于同一位置

注意 

该`ceph-helm/ceph/ceph/values.yaml`文件包含可以设置的选项的完整列表

### 创建CEPH集群名称空间

默认情况下，ceph-helm组件假定它们将在`ceph`Kubernetes命名空间中运行。要创建名称空间，请运行：

```text
$ kubectl create namespace ceph
```

### 配置RBAC权限

Kubernetes&gt; = v1.6使RBAC成为默认的准入控制器。ceph-helm为每个组件提供RBAC角色和权限：

```text
$ kubectl create -f ~/ceph-helm/ceph/rbac.yaml
```

该`rbac.yaml`文件假定Ceph集群将部署在`ceph`名称空间中。

### 标签小块

需要设置以下标签来部署Ceph集群：

* ceph-mon =已启用
* ceph-mgr =已启用
* ceph-osd =已启用
* ceph-osd-device- &lt;名称&gt; =已启用

该`ceph-osd-device-<name>`标签是基于我们定义的osd\_devices名称值创建`ceph-overrides.yaml`。在上面的示例中，我们将使用以下两个标签：`ceph-osd-device-dev-sdb`和`ceph-osd-device-dev-sdc`。

对于每个Ceph Monitor：

```text
$ kubectl label node <nodename> ceph-mon=enabled ceph-mgr=enabled
```

对于每个OSD节点：

```text
$ kubectl label node <nodename> ceph-osd=enabled ceph-osd-device-dev-sdb=enabled ceph-osd-device-dev-sdc=enabled
```

### CEPH部署

运行helm install命令以部署Ceph：

```text
$ helm install --name=ceph local/ceph --namespace=ceph -f ~/ceph-overrides.yaml
NAME:   ceph
LAST DEPLOYED: Wed Oct 18 22:25:06 2017
NAMESPACE: ceph
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                    TYPE    DATA  AGE
ceph-keystone-user-rgw  Opaque  7     1s

==> v1/ConfigMap
NAME              DATA  AGE
ceph-bin-clients  2     1s
ceph-bin          24    1s
ceph-etc          1     1s
ceph-templates    5     1s

==> v1/Service
NAME      CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
ceph-mon  None            <none>       6789/TCP  1s
ceph-rgw  10.101.219.239  <none>       8088/TCP  1s

==> v1beta1/DaemonSet
NAME              DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR                                     AGE
ceph-mon          3        3        0      3           0          ceph-mon=enabled                                  1s
ceph-osd-dev-sde  3        3        0      3           0          ceph-osd-device-dev-sde=enabled,ceph-osd=enabled  1s
ceph-osd-dev-sdd  3        3        0      3           0          ceph-osd-device-dev-sdd=enabled,ceph-osd=enabled  1s

==> v1beta1/Deployment
NAME                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
ceph-mds              1        1        1           0          1s
ceph-mgr              1        1        1           0          1s
ceph-mon-check        1        1        1           0          1s
ceph-rbd-provisioner  2        2        2           0          1s
ceph-rgw              1        1        1           0          1s

==> v1/Job
NAME                                 DESIRED  SUCCESSFUL  AGE
ceph-mgr-keyring-generator           1        0           1s
ceph-mds-keyring-generator           1        0           1s
ceph-osd-keyring-generator           1        0           1s
ceph-rgw-keyring-generator           1        0           1s
ceph-mon-keyring-generator           1        0           1s
ceph-namespace-client-key-generator  1        0           1s
ceph-storage-keys-generator          1        0           1s

==> v1/StorageClass
NAME     TYPE
ceph-rbd  ceph.com/rbd
```

helm install的输出向我们展示了将要部署的不同类型的资源。

将使用Pod 创建一个名为`ceph-rbd`type 的StorageClass 。这些将使创建PVC时自动配置RBD。首次映射时，RBD也将被格式化。所有RBD将使用ext4文件系统。不支持该选项。默认情况下，RBD将使用图像格式2和分层。您可以在值文件中覆盖以下storageclass的默认值：`ceph.com/rbdceph-rbd-provisionerceph.com/rbdfsType`

```text
storageclass:
  name: ceph-rbd
  pool: rbd
  user_id: k8s
  user_secret_name: pvc-ceph-client-key
  image_format: "2"
  image_features: layering
```

使用以下命令检查所有Pod是否都在运行。这可能需要几分钟：

```text
$ kubectl -n ceph get pods
NAME                                    READY     STATUS    RESTARTS   AGE
ceph-mds-3804776627-976z9               0/1       Pending   0          1m
ceph-mgr-3367933990-b368c               1/1       Running   0          1m
ceph-mon-check-1818208419-0vkb7         1/1       Running   0          1m
ceph-mon-cppdk                          3/3       Running   0          1m
ceph-mon-t4stn                          3/3       Running   0          1m
ceph-mon-vqzl0                          3/3       Running   0          1m
ceph-osd-dev-sdd-6dphp                  1/1       Running   0          1m
ceph-osd-dev-sdd-6w7ng                  1/1       Running   0          1m
ceph-osd-dev-sdd-l80vv                  1/1       Running   0          1m
ceph-osd-dev-sde-6dq6w                  1/1       Running   0          1m
ceph-osd-dev-sde-kqt0r                  1/1       Running   0          1m
ceph-osd-dev-sde-lp2pf                  1/1       Running   0          1m
ceph-rbd-provisioner-2099367036-4prvt   1/1       Running   0          1m
ceph-rbd-provisioner-2099367036-h9kw7   1/1       Running   0          1m
ceph-rgw-3375847861-4wr74               0/1       Pending   0          1m
```

注意 

由于我们未使用`ceph-rgw=enabled`或标记任何节点，因此MDS和RGW Pod处于待处理状态 `ceph-mds=enabled`

所有Pod运行之后，请从一个星期一开始检查Ceph集群的状态：

```text
$ kubectl -n ceph exec -ti ceph-mon-cppdk -c ceph-mon -- ceph -s
cluster:
  id:     e8f9da03-c2d2-4ad3-b807-2a13d0775504
  health: HEALTH_OK

services:
  mon: 3 daemons, quorum mira115,mira110,mira109
  mgr: mira109(active)
  osd: 6 osds: 6 up, 6 in

data:
  pools:   0 pools, 0 pgs
  objects: 0 objects, 0 bytes
  usage:   644 MB used, 5555 GB / 5556 GB avail
  pgs:
```

### 配置波德从CEPH的使用PERSISTENTVOLUME 

为在中定义的k8s用户创建一个密钥环`~/ceph-overwrite.yaml`，并将其转换为base64：

```text
$ kubectl -n ceph exec -ti ceph-mon-cppdk -c ceph-mon -- bash
# ceph auth get-or-create-key client.k8s mon 'allow r' osd 'allow rwx pool=rbd'  | base64
QVFCLzdPaFoxeUxCRVJBQUVEVGdHcE9YU3BYMVBSdURHUEU0T0E9PQo=
# exit
```

编辑`ceph`名称空间中存在的用户密码：

```text
$ kubectl -n ceph edit secrets/pvc-ceph-client-key
```

使用自己的值将base64值添加到键值并保存：

```text
apiVersion: v1
data:
  key: QVFCLzdPaFoxeUxCRVJBQUVEVGdHcE9YU3BYMVBSdURHUEU0T0E9PQo=
kind: Secret
metadata:
  creationTimestamp: 2017-10-19T17:34:04Z
  name: pvc-ceph-client-key
  namespace: ceph
  resourceVersion: "8665522"
  selfLink: /api/v1/namespaces/ceph/secrets/pvc-ceph-client-key
  uid: b4085944-b4f3-11e7-add7-002590347682
type: kubernetes.io/rbd
```

我们将创建一个在默认名称空间中使用RBD的Pod。将用户机密从`ceph`名称空间复制到`default`：

```text
$ kubectl -n ceph get secrets/pvc-ceph-client-key -o json | jq '.metadata.namespace = "default"' | kubectl create -f -
secret "pvc-ceph-client-key" created
$ kubectl get secrets
NAME                  TYPE                                  DATA      AGE
default-token-r43wl   kubernetes.io/service-account-token   3         61d
pvc-ceph-client-key   kubernetes.io/rbd                     1         20s
```

创建并初始化RBD池：

```text
$ kubectl -n ceph exec -ti ceph-mon-cppdk -c ceph-mon -- ceph osd pool create rbd 256
pool 'rbd' created
$ kubectl -n ceph exec -ti ceph-mon-cppdk -c ceph-mon -- rbd pool init rbd
```

重要 

Kubernetes使用RBD内核模块将RBD映射到主机。发光需要CRUSH\_TUNABLES 5（珠宝）。这些可调参数的最低内核版本为4.5。如果您的内核不支持这些可调参数，请运行`ceph osd crush tunables hammer`

重要 

由于RBD映射在主机系统上。主机需要能够解析由kube-dns服务管理的ceph-mon.ceph.svc.cluster.local名称。要获取kube-dns服务的IP地址，请运行`kubectl -n kube-system get svc/kube-dns`

创建一个PVC：

```text
$ cat pvc-rbd.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ceph-pvc
spec:
  accessModes:
   - ReadWriteOnce
  resources:
    requests:
       storage: 20Gi
  storageClassName: ceph-rbd

$ kubectl create -f pvc-rbd.yaml
persistentvolumeclaim "ceph-pvc" created
$ kubectl get pvc
NAME       STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
ceph-pvc   Bound     pvc-1c2ada50-b456-11e7-add7-002590347682   20Gi       RWO           ceph-rbd        3s
```

您可以检查已在集群上创建了RBD：

```text
$ kubectl -n ceph exec -ti ceph-mon-cppdk -c ceph-mon -- rbd ls
kubernetes-dynamic-pvc-1c2e9442-b456-11e7-9bd2-2a4159ce3915
$ kubectl -n ceph exec -ti ceph-mon-cppdk -c ceph-mon -- rbd info kubernetes-dynamic-pvc-1c2e9442-b456-11e7-9bd2-2a4159ce3915
rbd image 'kubernetes-dynamic-pvc-1c2e9442-b456-11e7-9bd2-2a4159ce3915':
    size 20480 MB in 5120 objects
    order 22 (4096 kB objects)
    block_name_prefix: rbd_data.10762ae8944a
    format: 2
    features: layering
    flags:
    create_timestamp: Wed Oct 18 22:45:59 2017
```

创建一个将使用PVC的Pod：

```text
$ cat pod-with-rbd.yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: busybox
      image: busybox
      command:
        - sleep
        - "3600"
      volumeMounts:
      - mountPath: "/mnt/rbd"
        name: vol1
  volumes:
    - name: vol1
      persistentVolumeClaim:
        claimName: ceph-pvc

$ kubectl create -f pod-with-rbd.yaml
pod "mypod" created
```

检查Pod：

```text
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
mypod     1/1       Running   0          17s
$ kubectl exec mypod -- mount | grep rbd
/dev/rbd0 on /mnt/rbd type ext4 (rw,relatime,stripe=1024,data=ordered)
```

### 记录

可以通过命令访问OSD和Monitor日志。监视器具有多个日志记录流，每个流都可以从运行在ceph-mon Pod中的容器访问。`kubectl logs [-f]`在ceph-mon Pod中有3个容器在运行：

* ceph-mon，等同于裸机上的ceph-mon.hostname.log
* cluster-audit-log-tailer，等效于裸机上的ceph.audit.log
* cluster-log-tailer，等效于裸机上的ceph.log或 `ceph -w`

可通过`--container`或`-c`选项访问每个容器。例如，要访问cluster-tail-log，可以运行：

```text
$ kubectl -n ceph logs ceph-mon-cppdk -c cluster-log-tailer
```

