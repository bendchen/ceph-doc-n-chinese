# 对象存储

## CEPH对象网关

[Ceph对象网关](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-object-gateway)是一个对象存储接口，建立在该对象之上， `librados`为应用程序提供了通往Ceph存储集群的RESTful网关。[Ceph对象存储](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-object-storage)支持两个接口：

1. **与S3兼容：为**对象存储功能提供与Amazon S3 RESTful API的大部分子集兼容的接口。
2. **兼容Swift：为**对象存储功能提供与OpenStack Swift API的大部分子集兼容的接口。

Ceph对象存储使用Ceph对象网关守护进程（`radosgw`），该守护进程是用于与Ceph存储群集进行交互的HTTP服务器。由于它提供与OpenStack Swift和Amazon S3兼容的接口，因此Ceph对象网关具有自己的用户管理。Ceph对象网关可以将数据存储在用于存储来自Ceph文件系统客户端或Ceph块设备客户端的数据的同一Ceph存储群集中。S3和Swift API共享一个公共的名称空间，因此您可以使用一个API编写数据，而使用另一个API检索数据。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-50d12451eb76c5c72c4574b08f0320b39a42e5f1.png)

