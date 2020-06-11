# 飞行前检查清单

## 飞行前检查清单

该`ceph-deploy`工具不在管理节点上的目录中运行 。任何具有网络连接，现代python环境和ssh（例如Linux）的主机都可以工作。

在以下描述中，节点指的是一台机器。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-b490c5d9d3bb6984503b59681d08337aff62e992.png)

### CEPH部署设置

将Ceph存储库添加到`ceph-deploy`管理节点。然后，安装 `ceph-deploy`。

#### DEBIAN / 

对于Debian和Ubuntu发行版，请执行以下步骤：

1. 添加释放密钥：

   ```text
   wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
   ```

2. 将Ceph软件包添加到您的存储库。使用下面的命令并替换`{ceph-stable-release}`为稳定的Ceph版本（例如 `luminous`。），例如：

   ```text
   echo deb https://download.ceph.com/debian-{ceph-stable-release}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
   ```

3. 更新您的存储库并安装`ceph-deploy`：

   ```text
   sudo apt update
   sudo apt install ceph-deploy
   ```

注意 

您也可以使用EU镜像eu.ceph.com来下载软件包，方法是替换`https://ceph.com/`为`http://eu.ceph.com/`

#### RHEL / 

对于CentOS 7，执行以下步骤：

1. 在Red Hat Enterprise Linux 7上，向目标计算机注册 `subscription-manager`，验证您的订阅，并启用“ Extras”存储库以获取软件包依赖性。例如：

   ```text
   sudo subscription-manager repos --enable=rhel-7-server-extras-rpms
   ```

2. 安装并启用企业Linux附加软件包（EPEL）存储库：

   ```text
   sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   ```

   请参阅[EPEL Wiki](https://fedoraproject.org/wiki/EPEL)页面以获取更多信息。

3. `/etc/yum.repos.d/ceph.repo`使用以下命令将Ceph存储库添加到yum配置文件。替换 `{ceph-stable-release}`为稳定的Ceph版本（例如 `luminous`。），例如：

   ```text
   cat << EOM > /etc/yum.repos.d/ceph.repo
   [ceph-noarch]
   name=Ceph noarch packages
   baseurl=https://download.ceph.com/rpm-{ceph-stable-release}/el7/noarch
   enabled=1
   gpgcheck=1
   type=rpm-md
   gpgkey=https://download.ceph.com/keys/release.asc
   EOM
   ```

4. 更新您的存储库并安装`ceph-deploy`：

   ```text
   sudo yum update
   sudo yum install ceph-deploy
   ```

注意 

您也可以使用EU镜像eu.ceph.com来下载软件包，方法是替换`https://ceph.com/`为`http://eu.ceph.com/`

#### OPENSUSE的

Ceph项目当前未发布针对openSUSE的发行RPM，但是默认更新存储库中包含稳定版本的Ceph，因此，只需进行以下操作即可：

```text
sudo zypper install ceph
sudo zypper install ceph-deploy
```

如果发行版已过期，请在[https://bugzilla.opensuse.org/index.cgi中](https://bugzilla.opensuse.org/index.cgi)打开错误， 并可能通过以下存储库之一尝试运气：

1. 锤子：

   ```text
   https://software.opensuse.org/download.html?project=filesystems%3Aceph%3Ahammer&package=ceph
   ```

2. 宝石：

   ```text
   https://software.opensuse.org/download.html?project=filesystems%3Aceph%3Ajewel&package=ceph
   ```

### CEPH节点设置

管理节点必须具有对Ceph节点的无密码SSH访问。当ceph-deploy以用户身份登录到Ceph节点时，该特定用户必须具有无密码`sudo`特权。

#### 安装

我们建议在Ceph节点上（尤其是在Ceph Monitor节点上）安装NTP，以防止时钟漂移引起问题。有关详情，请参见[时钟](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref#clock)。

在CentOS / RHEL上，执行：

```text
sudo yum install ntp ntpdate ntp-doc
```

在Debian / Ubuntu上，执行：

```text
sudo apt install ntp
```

确保启用NTP服务。确保每个Ceph节点使用相同的NTP时间服务器。有关详细信息，请参见[NTP](http://www.ntp.org/)。

#### 安装SSH服务器

对于**所有** Ceph节点，请执行以下步骤：

1. 在每个Ceph节点上安装SSH服务器（如果需要）：

   ```text
   sudo apt install openssh-server
   ```

   要么：

   ```text
   sudo yum install openssh-server
   ```

2. 确保SSH服务器在**所有** Ceph节点上运行。

#### 创建一个CEPH部署用户

该`ceph-deploy`实用程序必须以具有无密码`sudo`特权的用户身份登录到Ceph节点，因为该实用程序需要安装软件和配置文件而不提示输入密码。

最新版本的`ceph-deploy`支持`--username`选项，因此您可以指定没有密码的任何用户`sudo`（包括`root`，尽管**不**建议这样做）。要使用，您指定的用户必须具有对Ceph节点的无密码SSH访问权限，因为它 不会提示您输入密码。`ceph-deploy --username {username}ceph-deploy`

我们建议创建一个特定的用户`ceph-deploy`在**所有**头孢集群中的节点。请**不要**使用“ ceph”作为用户名。在集群统一的用户名可以提高易用性（不要求），但你应该避免使用明显的用户名，因为黑客通常使用蛮力黑客使用它们（例如`root`， `admin`，`{productname}`）。以下过程替换 `{username}`您定义的用户名，描述了如何使用passwordless创建用户`sudo`。

注意 

从[Infernalis版本开始](https://docs.ceph.com/docs/nautilus/releases/infernalis/#infernalis-release-notes)，“ ceph”用户名为Ceph守护程序保留。如果“ ceph”用户已经存在于Ceph节点上，则必须先删除该用户，然后再尝试升级。

1. 在每个Ceph节点上创建一个新用户。

   ```text
   ssh user@ceph-server
   sudo useradd -d /home/{username} -m {username}
   sudo passwd {username}
   ```

2. 对于您添加到每个Ceph节点的新用户，请确保该用户具有 `sudo`特权。

   ```text
   echo "{username} ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/{username}
   sudo chmod 0440 /etc/sudoers.d/{username}
   ```

#### 启用无密码

由于`ceph-deploy`不会提示您输入密码，因此您必须在管理节点上生成SSH密钥，并将公用密钥分发给每个Ceph节点。`ceph-deploy`将尝试为初始监视器生成SSH密钥。

1. 生成SSH密钥，但不使用`sudo`或 `root`用户。将密码短语留空：

   ```text
   ssh-keygen

   Generating public/private key pair.
   Enter file in which to save the key (/ceph-admin/.ssh/id_rsa):
   Enter passphrase (empty for no passphrase):
   Enter same passphrase again:
   Your identification has been saved in /ceph-admin/.ssh/id_rsa.
   Your public key has been saved in /ceph-admin/.ssh/id_rsa.pub.
   ```

2. 将密钥复制到每个Ceph节点，`{username}`并用在[创建Ceph部署用户中创建](https://docs.ceph.com/docs/nautilus/start/quick-start-preflight/#create-a-ceph-deploy-user)的用户名替换。

   ```text
   ssh-copy-id {username}@node1
   ssh-copy-id {username}@node2
   ssh-copy-id {username}@node3
   ```

3. （推荐）修改 管理节点的`~/.ssh/config`文件，`ceph-deploy`以便`ceph-deploy`可以以创建的用户身份登录到Ceph节点，而无需您指定每次执行的时间。这具有简化和使用的额外好处 。替换为您创建的用户名：`--username {username}ceph-deploysshscp{username}`

   ```text
   Host node1
      Hostname node1
      User {username}
   Host node2
      Hostname node2
      User {username}
   Host node3
      Hostname node3
      User {username}
   ```

#### 在启动时启用网络

Ceph OSD彼此对等并通过网络向Ceph Monitors报告。如果`off`默认情况下为网络连接，则在启用网络连接之前，Ceph群集无法在启动过程中联机。

某些发行版（例如CentOS）上的默认配置默认情况下已关闭网络接口。确保在启动过程中，您的网络接口已打开，以便您的Ceph守护程序可以通过网络进行通信。例如，在Red Hat和CentOS上，导航到 `/etc/sysconfig/network-scripts`并确保 `ifcfg-{iface}`文件已`ONBOOT`设置为`yes`。

#### 确保连接性

确保使用`ping`短主机名（）进行连接。根据需要解决主机名解析问题。`hostname -s`

注意 

主机名应解析为网络IP地址，而不是环回IP地址（例如，主机名应解析为IP地址而不是`127.0.0.1`）。如果将管理节点用作Ceph节点，则还应确保将其解析为其主机名和IP地址（即，不解析其环回IP地址）。

#### 需要打开的端口

`6789`默认情况下，Ceph监视器使用端口进行通信。`6800:7300`默认情况下，Ceph OSD在端口范围内进行通信。有关详细信息，请参见[网络配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)。Ceph OSD可以使用多个网络连接与客户端，监视器，用于复制的其他OSD和用于心跳的其他OSD通信。

在某些发行版（例如，RHEL）上，默认防火墙配置相当严格。您可能需要调整防火墙设置以允许入站请求，以便网络中的客户端可以与Ceph节点上的守护程序进行通信。

对于`firewalld`RHEL 7，在公共区域中添加`ceph-mon`用于Ceph Monitor节点的`ceph`服务以及用于Ceph OSD和MDS 的服务，并确保您将设置永久化，以便在重新启动时启用它们。

例如，在监视器上：

```text
sudo firewall-cmd --zone=public --add-service=ceph-mon --permanent
```

在OSD和MDS上：

```text
sudo firewall-cmd --zone=public --add-service=ceph --permanent
```

使用`--permanent`标志完成对firewalld的配置后，您可以立即进行更改，而无需重新启动：

```text
sudo firewall-cmd --reload
```

对于`iptables`，添加`6789`Ceph监视器的端口`6800:7300` 和Ceph OSD的端口。例如：

```text
sudo iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT
```

完成配置后`iptables`，请确保在每个节点上进行更改，以使更改在您的节点重新启动时生效。例如：

```text
/sbin/service iptables save
```

#### TTY 

在CentOS和RHEL上，尝试执行`ceph-deploy`命令时可能会收到错误消息 。如果`requiretty`在您的Ceph节点上默认设置了if ，则通过执行并找到该设置将其禁用。将其更改为或注释掉以确保可以使用通过 [创建Ceph Deploy用户创建的用户进行连接](https://docs.ceph.com/docs/nautilus/start/quick-start-preflight/#create-a-ceph-deploy-user)。`sudo visudoDefaults requirettyDefaults:ceph !requirettyceph-deploy`

注意 

如果进行编辑，请`/etc/sudoers`确保使用 而不是文本编辑器。`sudo visudo`

#### SELINUX的

在CentOS和RHEL上，SELinux `Enforcing`默认设置为。为了简化安装，我们建议将SELinux设置为`Permissive`或完全禁用SELinux，并在加强配置之前确保安装和集群正常运行。要将SELinux设置为`Permissive`，请执行以下操作：

```text
sudo setenforce 0
```

要持久配置SELinux（如果出现SELinux，则建议这样做），请在修改配置文件 `/etc/selinux/config`。

#### 优先级/首选项

确保软件包管理器已安装并启用了优先级/首选项软件包。在CentOS上，您可能需要安装EPEL。在RHEL上，您可能需要启用可选存储库。

```text
sudo yum install yum-plugin-priorities
```

例如，在RHEL 7服务器上，执行以下操作以安装 `yum-plugin-priorities`并启用 `rhel-7-server-optional-rpms` 存储库：

```text
sudo yum install yum-plugin-priorities --enablerepo=rhel-7-server-optional-rpms
```

### 总结[¶](https://docs.ceph.com/docs/nautilus/start/quick-start-preflight/#summary)

这样就完成了快速入门预检。

