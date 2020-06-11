# 获取编译好的包

## 获取编译好的包

要安装Ceph和其他启用软件，您需要从Ceph存储库中检索软件包。按照本指南获取软件包；然后，进入“ [安装Ceph对象存储”](https://docs.ceph.com/docs/nautilus/install/install-storage-cluster)。

### 获取包

有两种获取软件包的方法：

* **添加存储库：**添加存储库是获取软件包的最简单方法，因为在大多数情况下，软件包管理工具将为您检索软件包和所有启用软件。但是，要使用此方法，群集中的每个 [Ceph节点](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-node)必须具有Internet访问权限。
* **手动下载软件包：**如果您的环境不允许[Ceph节点](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-node)访问Internet，则**手动**下载软件包是安装Ceph的便捷方法。

### 要求

所有Ceph部署都需要Ceph软件包（开发除外）。您还应该添加密钥和推荐的软件包。

* **密钥：（推荐）**无论您是添加存储库还是手动下载软件包，都应下载密钥以验证软件包。如果您没有获得密钥，则可能会遇到安全警告。有关详细信息，请参见[添加密钥](https://docs.ceph.com/docs/nautilus/install/get-packages/#add-keys)。
* **Ceph ：（必需）**所有Ceph部署都需要Ceph发行包，但使用开发包的部署除外（仅开发，QA和最新边缘部署）。有关详细信息，请参见[添加Ceph](https://docs.ceph.com/docs/nautilus/install/get-packages/#add-ceph)。
* **Ceph开发：（可选）**如果您正在为Ceph开发，测试Ceph开发版本，或者想要Ceph开发的前沿功能，则可以获取Ceph开发包。有关详细信息，请参见 [添加Ceph开发](https://docs.ceph.com/docs/nautilus/install/get-packages/#add-ceph-development)。

如果您打算手动下载软件包，请参见“ [下载软件包”一](https://docs.ceph.com/docs/nautilus/install/get-packages/#download-packages)节。

### 添加关键

将密钥添加到系统的受信任密钥列表中，以避免出现安全警告。对于主要版本（例如`hammer`，`jewel`，`luminous`）和开发版本（`release-name-rc1`，`release-name-rc2`），使用`release.asc`密钥。

#### APT 

要安装`release.asc`密钥，请执行以下操作：

```text
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
```

#### RPM 

要安装`release.asc`密钥，请执行以下操作：

```text
sudo rpm --import 'https://download.ceph.com/keys/release.asc'
```

### 加入头孢

发布存储库使用`release.asc`密钥来验证软件包。要使用高级软件包工具（APT）或经过修改的Yellowdog Updater（YUM）安装Ceph软件包，必须添加Ceph存储库。

您可以在以下位置找到Debian / Ubuntu（与APT一起安装）的发行版：

```text
https://download.ceph.com/debian-{release-name}
```

您可以在以下位置找到CentOS / RHEL和其他版本（随YUM一起安装）的发行版：

```text
https://download.ceph.com/rpm-{release-name}
```

Ceph的主要版本汇总在：[发行索引](https://docs.ceph.com/docs/nautilus/releases/#id1)

第二个主要发行版被认为是长期稳定（LTS）。关键的错误修正将向后移植到LTS版本中，直到它们退役为止。由于不再维护已淘汰的发行版，因此我们建议用户定期升级其群集-最好升级到最新的LTS版本。

小费 

对于非美国用户：从您附近可以下载Ceph的地方可能有一个镜子。有关更多信息，请参见：[Ceph Mirrors](https://docs.ceph.com/docs/nautilus/install/mirrors)。

#### DEBIAN软件包

将Ceph软件包存储库添加到系统的APT来源列表中。对于较新版本的Debian / Ubuntu，请在命令行上调用以获取简短的代号，然后替换以下命令。`lsb_release -sc{codename}`

```text
sudo apt-add-repository 'deb https://download.ceph.com/debian-luminous/ {codename} main'
```

对于早期的Linux发行版，您可以执行以下命令：

```text
echo deb https://download.ceph.com/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

对于较早的Ceph版本，`{release-name}`用name 替换为Ceph版本的名称。您可以在命令行上调用以获取简短的代号，然后替换以下命令。`lsb_release -sc{codename}`

```text
sudo apt-add-repository 'deb https://download.ceph.com/debian-{release-name}/ {codename} main'
```

对于较旧的Linux发行版，请用`{release-name}`发行版名称替换：

```text
echo deb https://download.ceph.com/debian-{release-name}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

对于开发发行版软件包，将我们的软件包存储库添加到您系统的APT来源列表中。有关受支持的Debian和Ubuntu版本的完整列表，请参见[测试的Debian存储库](https://download.ceph.com/debian-testing/dists)。

```text
echo deb https://download.ceph.com/debian-testing/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

小费 

对于非美国用户：从您附近可以下载Ceph的地方可能有一个镜子。有关更多信息，请参见：[Ceph Mirrors](https://docs.ceph.com/docs/nautilus/install/mirrors)。

#### RPM软件包

对于主要版本，您可以在`/etc/yum.repos.d` 目录中添加一个Ceph条目。创建一个`ceph.repo`文件。在下面的例子中，替换 `{ceph-release}`用头孢的主要版本（例如`hammer`，`jewel`，`luminous`等），并`{distro}`与你的Linux发行版（如，`el7`等）。您可以查看[https://download.ceph.com/rpm-{ceph-release](https://download.ceph.com/rpm) } /目录，以查看Ceph支持哪些发行版。某些Ceph软件包（例如EPEL）必须优先于标准软件包，因此必须确保设置了 `priority=2`。

```text
[ceph]
name=Ceph packages for $basearch
baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/$basearch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc
```

对于特定的软件包，您可以通过按名称下载发行软件包来检索它们。我们的开发过程每3-4周生成一次Ceph的新版本。这些软件包的移动速度比主要版本快。开发软件包具有快速集成的新功能，同时在发布之前仍需进行数周的质量检查。

存储库软件包将存储库详细信息安装在您的本地系统上，以用于`yum`。用`{distro}`Linux发行 `{release}`版和Ceph的特定版本替换：

```text
su -c 'rpm -Uvh https://download.ceph.com/rpms/{distro}/x86_64/ceph-{release}.el7.noarch.rpm'
```

您可以直接从以下位置下载RPM：

```text
https://download.ceph.com/rpm-testing
```

小费 

对于非美国用户：从您附近可以下载Ceph的地方可能有一个镜子。有关更多信息，请参见：[Ceph Mirrors](https://docs.ceph.com/docs/nautilus/install/mirrors)。

### 添加CEPH开发

如果您正在开发Ceph并且需要部署和测试特定的Ceph分支，请确保首先删除主要版本的存储库条目。

#### DEB包

我们会在Ceph源代码存储库中为当前开发分支自动构建Ubuntu软件包。这些软件包仅适用于开发人员和质量检查人员。

将软件包存储库添加到系统的APT来源列表中，但替换`{BRANCH}`为您要使用的分支（例如wip-hack，master）。请参阅[萨满页面](https://shaman.ceph.com/)以获取我们构建的发行版的完整列表。

```text
curl -L https://shaman.ceph.com/api/repos/ceph/{BRANCH}/latest/ubuntu/$(lsb_release -sc)/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list
```

注意 

如果存储库尚未准备好，将返回HTTP 504

使用的`latest`URL中，意味着它会找出哪个是最后提交已建成。或者，可以指定特定的sha1。对于Ubuntu Xenial和Ceph的master分支，它看起来像：

```text
curl -L https://shaman.ceph.com/api/repos/ceph/master/53e772a45fdf2d211c0c383106a66e1feedec8fd/ubuntu/xenial/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list
```

警告 

两周后将不再提供开发资料库。

#### RPM软件包

对于当前的开发分支，您可以在`/etc/yum.repos.d`目录中添加一个Ceph条目 。该[萨满页面](https://shaman.ceph.com/)可用于检索回购文件的完整细节。可以通过HTTP请求来检索它，例如：

```text
curl -L https://shaman.ceph.com/api/repos/ceph/{BRANCH}/latest/centos/7/repo/ | sudo tee /etc/yum.repos.d/shaman.repo
```

使用的`latest`URL中，意味着它会找出哪个是最后提交已建成。或者，可以指定特定的sha1。对于CentOS 7和Ceph的master分支，它看起来像：

```text
curl -L https://shaman.ceph.com/api/repos/ceph/master/53e772a45fdf2d211c0c383106a66e1feedec8fd/centos/7/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list
```

警告 

两周后将不再提供开发资料库。

注意 

如果存储库尚未准备好，将返回HTTP 504

### 下载包

如果要在没有Internet访问的环境中尝试在防火墙后面安装，则必须在尝试安装之前检索软件包（已镜像所有必需的依赖项）。

#### DEBIAN软件包

Ceph需要其他第三方库。

* libaio1
* libsnappy1
* libcurl3
* 卷曲
* libgoogle-perftools4
* google-perftools
* libleveldb1

存储库软件包将存储库详细信息安装在您的本地系统上，以用于`apt`。替换`{release}`为最新的Ceph版本。替换 `{version}`为最新的Ceph版本号。`{distro}`用您的Linux发行版代号替换。替换`{arch}`为CPU体系结构。

```text
wget -q https://download.ceph.com/debian-{release}/pool/main/c/ceph/ceph_{version}{distro}_{arch}.deb
```

#### RPM软件包

Ceph需要其他附加的第三方库。要添加EPEL存储库，请执行以下操作：

```text
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

Ceph需要以下软件包：

* 活泼的
* 级别数据库
* 磁盘
* python-argparse
* gperftools-libs

当前为RHEL / CentOS7（`el7`）平台构建了软件包。存储库软件包将存储库详细信息安装在您的本地系统上，以用于`yum`。`{distro}`用您的发行版替换。

```text
su -c 'rpm -Uvh https://download.ceph.com/rpm-luminous/{distro}/noarch/ceph-{version}.{distro}.noarch.rpm'
```

例如，对于CentOS 7（`el7`）：

```text
su -c 'rpm -Uvh https://download.ceph.com/rpm-luminous/el7/noarch/ceph-release-1-0.el7.noarch.rpm'
```

您可以直接从以下位置下载RPM：

```text
https://download.ceph.com/rpm-luminous
```

对于较早的Ceph版本，`{release-name}`用name 替换为Ceph版本的名称。您可以在命令行上调用以获取简短的代号。`lsb_release -sc`

```text
su -c 'rpm -Uvh https://download.ceph.com/rpm-{release-name}/{distro}/noarch/ceph-{version}
```

