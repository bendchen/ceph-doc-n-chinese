# 飞行前检查清单

## 飞行前检查清单

0.60版中的新功能。

此**预检清单**将帮助您准备与之配合使用的管理节点 `ceph-deploy`以及与无密码`ssh`和 一起使用的服务器节点`sudo`。

使用部署Ceph之前`ceph-deploy`，需要确保首先在管理节点和运行Ceph守护程序的节点上进行了一些设置。

### 安装操作系统

在您的节点上安装Debian或Ubuntu的最新版本（例如16.04 LTS）。有关操作系统的更多详细信息，或要使用Debian或Ubuntu以外的其他操作系统，请参阅[OS Recommendations](https://docs.ceph.com/docs/nautilus/start/os-recommendations)。

### 安装SSH服务器

该`ceph-deploy`实用程序需要`ssh`，因此您的服务器节点需要SSH服务器。

```text
sudo apt-get install openssh-server
```

### 创建用户

在运行Ceph守护程序的节点上创建用户。

小费 

我们建议用户名是蛮力攻击者也不会轻易猜测（例如，比其他的东西`root`，`ceph`等等）。

```text
ssh user@ceph-server
sudo useradd -d /home/ceph -m ceph
sudo passwd ceph
```

`ceph-deploy`将软件包安装到您的节点上。这意味着您创建的用户需要无密码`sudo`特权。

注意 

出于安全原因，我们**不**建议启用`root`密码。

要向用户提供完全特权，请将以下内容添加到中 `/etc/sudoers.d/ceph`。

```text
echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
sudo chmod 0440 /etc/sudoers.d/ceph
```

### 配置

使用对运行Ceph守护程序的每个节点的无密码SSH访问来配置您的管理机器（将密码短语留空）。

```text
ssh-keygen
Generating public/private key pair.
Enter file in which to save the key (/ceph-client/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /ceph-client/.ssh/id_rsa.
Your public key has been saved in /ceph-client/.ssh/id_rsa.pub.
```

将密钥复制到每个运行Ceph守护程序的节点：

```text
ssh-copy-id ceph@ceph-server
```

修改您的管理节点的〜/ .ssh / config文件，以使其在未指定用户名时默认以您创建的用户身份登录。

```text
Host ceph-server
        Hostname ceph-server.fqdn-or-ip-address.com
        User ceph
```

### 安装CEPH- 

要安装`ceph-deploy`，请执行以下操作：

```text
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
echo deb http://ceph.com/debian-dumpling/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
sudo apt-get update
sudo apt-get install ceph-deploy
```

### 确保连接性

确保您的联系节点连接到网络，你的服务器节点（例如，保证`iptables`，`ufw`或可能阻止连接，流量转发，等等，让你所需要的其他工具）。

完成此飞行前检查清单后，就可以开始使用了 `ceph-deploy`。

