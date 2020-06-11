# DASHBOARD

## CEPH仪表板

### 概述

Ceph仪表板是基于Web的内置Ceph管理和监视应用程序，用于管理集群的各个方面和对象。它作为[Ceph Manager守护程序](https://docs.ceph.com/docs/nautilus/mgr/#ceph-manager-daemon)模块实现。

Ceph Luminous随附的原始Ceph仪表板最初是一个简单的只读视图，可查看Ceph集群的各种运行时信息和性能数据。它使用了非常简单的架构来实现最初的目标。但是，对于添加更多基于Web的管理功能的需求不断增长，以使更喜欢使用WebUI而不是使用命令行的用户更易于管理Ceph。

新的[Ceph仪表板](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-dashboard)模块替代了之前的模块，并向Ceph Manager添加了一个内置的基于Web的监视和管理应用程序。这个新插件的体系结构和功能源自[openATTIC Ceph管理和监视工具](https://openattic.org/)，并受其启发。[SUSE](https://www.suse.com/) openATTIC背后的团队积极推动开发，并获得了[Red Hat](https://redhat.com/)等公司和Ceph社区其他成员的大力支持。

仪表板模块的后端代码使用CherryPy框架和自定义REST API实现。WebUI实现基于Angular / TypeScript，合并了来自原始仪表板的功能，并添加了最初为独立版本的openATTIC开发的新功能。Ceph仪表板模块被实现为Web应用程序，该Web应用程序使用托管的Web服务器来可视化有关Ceph群集的信息和统计信息`ceph-mgr`。

#### 功能概述

仪表板提供以下功能：

* **多用户和角色管理**：仪表板支持具有不同权限（角色）的多个用户帐户。可以在命令行和通过WebUI修改用户帐户和角色。有关详细信息，请参见[用户和角色管理](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-user-role-management)。
* **单一登录（SSO）**：显示板支持使用SAML 2.0协议通过外部身份提供程序进行身份验证。有关详细信息，请参阅 [启用单点登录（SSO）](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-sso-support)。
* **SSL / TLS支持**：Web浏览器和仪表板之间的所有HTTP通信都通过SSL进行保护。可以使用内置命令创建自签名证书，但是也可以导入由CA签名和颁发的自定义证书。有关详细信息，请参见[SSL / TLS支持](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-ssl-tls-support)。
* **审核**：仪表盘后端可以配置为在Ceph审核日志中记录所有PUT，POST和DELETE API请求。有关 如何启用此功能的说明，[请](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-auditing)参见[审核API请求](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-auditing)。
* **国际化（I18N）**：仪表板可以在运行时选择的不同语言中使用。

目前，Ceph仪表板能够监视和管理Ceph集群的以下方面：

* **总体群集运行状况**：显示总体群集状态，性能和容量指标。
* **嵌入式Grafana仪表板**：Ceph仪表板能够将[Grafana](https://grafana.com/)仪表板嵌入 许多位置，以显示[Prometheus模块](https://docs.ceph.com/docs/nautilus/mgr/prometheus/#mgr-prometheus)收集的其他信息和性能指标 。有关如何配置此功能的详细信息，请参见[启用Grafana仪表板的嵌入](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-grafana)。
* **群集日志**：显示群集事件和审核日志文件的最新更新。日志条目可以按优先级，日期或关键字进行过滤。
* **主机**：显示与群集关联的所有主机，正在运行的服务以及已安装的Ceph版本的列表。
* **性能计数器**：显示每个正在运行的服务的特定于服务的详细统计信息。
* **监视器**：列出所有MON，其法定状态，打开的会话。
* **监视**：允许创建，重新创建，编辑和终止Prometheus的静音，列出Prometheus的警报配置和当前触发的警报。还显示触发警报的通知。需要配置。
* **配置编辑器**：显示所有可用的配置选项，它们的描述，类型和默认值，并编辑当前值。
* **池**：列出所有Ceph池及其详细信息（例如，应用程序，放置组，复制大小，EC配置文件，CRUSH规则集等）
* **OSD**：列出所有OSD，它们的状态和使用情况统计信息，以及诸如读写操作的属性（OSD映射），元数据，性能计数器和使用情况直方图之类的详细信息。标记OSD向上/向下/移出，清除和重新称重OSD，执行清理操作，修改各种与清理相关的配置选项，选择不同的配置文件以调整回填活动的级别。
* **iSCSI**：列出所有运行TCMU运行器服务，显示所有映像及其性能特征（读/写操作，流量）的主机。创建，修改和删除iSCSI目标（通过`ceph-iscsi`）。有关如何配置此功能的说明，请参阅 [启用iSCSI管理](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-iscsi-management)。
* **RBD**：列出所有RBD图像及其属性（大小，对象，特征）。创建，复制，修改和删除RBD图像。在全局，每个池或每个映像级别定义各种I / O或带宽限制设置。创建，删除和回滚所选图像的快照，保护/取消保护这些快照以防修改。复制或克隆快照，展平克隆的图像。
* **RBD镜像**：启用和配置RBD镜像到远程Ceph服务器。列出所有活动的同步守护程序及其状态，池和RBD映像，包括其同步状态。
* **CephFS**：列出所有活动的文件系统客户端和关联的池，包括其使用情况统计信息。
* **对象网关**：列出所有活动对象网关及其性能计数器。显示和管理（添加/编辑/删除）对象网关用户及其详细信息（例如配额）以及用户的存储桶及其详细信息（例如所有者，配额）。有关配置说明，请参阅[启用对象网关管理前端](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-enabling-object-gateway)。
* **NFS**：通过NFS Ganesha管理CephFS文件系统和RGW S3存储桶的NFS导出。有关如何启用此功能的详细信息，请参见[NFS-Ganesha管理](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#dashboard-nfs-ganesha-management)。
* **Ceph Manager模块**：启用和禁用所有Ceph Manager模块，更改模块特定的配置设置。

#### 支持的浏览器

Ceph仪表板主要使用以下Web浏览器进行测试和开发：

| 浏览器 | 版本号 |
| :--- | :--- |
| [铬](https://www.google.com/chrome/) | 68+ |
| [火狐浏览器](http://www.mozilla.org/firefox/) | 61+ |

尽管Ceph Dashboard可能在较旧的浏览器中运行，但我们不能保证一定会建议您将浏览器更新到最新版本。

### 启用

如果您是`ceph-mgr-dashboard`从分发程序包安装的，则程序包管理系统应该已经安装了所有必需的依赖项。

如果您要从源代码安装Ceph并想从开发环境中启动仪表板，请参阅源代码 目录中的文件`README.rst`和`HACKING.rst`目录`src/pybind/mgr/dashboard`。

在正在运行的Ceph集群中，可以通过以下方式启用Ceph仪表板：

```text
$ ceph mgr module enable dashboard
```

### 配置

#### SSL / TLS支持

默认情况下，与仪表板的所有HTTP连接均使用SSL / TLS保护。

为了使仪表板快速启动并运行，您可以使用以下内置命令生成并安装自签名证书：

```text
$ ceph dashboard create-self-signed-cert
```

请注意，大多数Web浏览器都会抱怨此类自签名证书，并要求在建立与仪表板的安全连接之前进行明确确认。

为了正确保护部署并消除证书警告，应使用由证书颁发机构（CA）颁发的证书。

例如，可以使用类似于以下命令的命令来生成密钥对：

```text
$ openssl req -new -nodes -x509 \
  -subj "/O=IT/CN=ceph-mgr-dashboard" -days 3650 \
  -keyout dashboard.key -out dashboard.crt -extensions v3_ca
```

`dashboard.crt`然后应由CA 对文件签名。完成后，您可以通过运行以下命令为所有Ceph管理器实例启用它：

```text
$ ceph dashboard set-ssl-certificate -i dashboard.crt
$ ceph dashboard set-ssl-certificate-key -i dashboard.key
```

如果出于某种原因要求每个管理器实例使用不同的证书，则可以按如下方式包括实例名称（其中`$name`，`ceph-mgr`实例名称，通常是主机名）：

```text
$ ceph dashboard set-ssl-certificate $name -i dashboard.crt
$ ceph dashboard set-ssl-certificate-key $name -i dashboard.key
```

也可以通过设置以下配置值来禁用SSL：

```text
$ ceph config set mgr mgr/dashboard/ssl false
```

如果仪表板将在不支持上游服务器SSL的代理后面运行，或者在不需要或不需要SSL的其他情况下运行，则这可能很有用。

警告 

禁用SSL时请务必谨慎，因为用户名和密码将以未加密的形式发送到仪表板。

注意 

更改SSL证书和密钥后，您需要手动重新启动Ceph管理器进程。这可以通过运行或禁用并重新启用仪表板模块来完成（这也会触发管理器重新生成自身）：`ceph mgr fail mgr`

```text
$ ceph mgr module disable dashboard
$ ceph mgr module enable dashboard
```

#### 主机名和端口

像大多数Web应用程序一样，仪表板绑定到TCP / IP地址和TCP端口。

默认情况下，`ceph-mgr`禁用SSL时，托管仪表板的守护程序（即当前活动的管理器）将绑定到TCP端口8443或8080。

如果未配置任何特定地址，则该Web应用将绑定到`::`，该地址对应于所有可用的IPv4和IPv6地址。

可以通过群集范围内的配置密钥工具更改这些默认值（因此它们适用于所有管理器实例），如下所示：

```text
$ ceph config set mgr mgr/dashboard/server_addr $IP
$ ceph config set mgr mgr/dashboard/server_port $PORT
$ ceph config set mgr mgr/dashboard/ssl_server_port $PORT
```

由于每个人都`ceph-mgr`拥有自己的仪表板实例，因此也可能需要单独配置它们。可以使用以下命令来更改特定管理器实例的IP地址和端口：

```text
$ ceph config set mgr mgr/dashboard/$name/server_addr $IP
$ ceph config set mgr mgr/dashboard/$name/server_port $PORT
$ ceph config set mgr mgr/dashboard/$name/ssl_server_port $PORT
```

替换`$name`为托管仪表板Web应用程序的ceph-mgr实例的ID。

注意 

该命令将向您显示当前已配置的所有端点。查找密钥以获取用于访问仪表板的URL。`ceph mgr servicesdashboard`

#### 用户名和密码

为了能够登录，您需要创建一个用户帐户并将其与至少一个角色相关联。我们提供了一组可以使用的预定义_系统角色_。有关更多详细信息，请参阅“ [用户和角色管理”](https://docs.ceph.com/docs/nautilus/mgr/dashboard/#user-and-role-management) 部分。

要创建具有管理员角色的用户，可以使用以下命令：

```text
$ ceph dashboard ac-user-create <username> <password> administrator
```

#### 启用对象网关管理前端

要使用仪表板的对象网关管理功能，您将需要提供`system`启用了该标志的用户的登录凭据。

如果您没有用于提供这些凭据的用户，则还需要创建一个：

```text
$ radosgw-admin user create --uid=<user_id> --display-name=<display_name> \
    --system
```

记下这些键`access_key`以及`secret_key`此命令的输出。

也可以使用radosgw-admin获得现有用户的凭证 ：

```text
$ radosgw-admin user info --uid=<user_id>
```

最后，向仪表板提供凭据：

```text
$ ceph dashboard set-rgw-api-access-key <access_key>
$ ceph dashboard set-rgw-api-secret-key <secret_key>
```

在具有单个RGW终结点的典型默认配置中，这就是使Object Gateway管理功能正常工作所要做的。仪表板将尝试通过从Ceph Manager的服务图中获取此信息来自动确定对象网关的主机和端口。

如果使用多个区域，它将自动确定主区域组和主区域中的主机。对于大多数设置来说，这应该足够了，但是在某些情况下，您可能需要手动设置主机和端口：

```text
$ ceph dashboard set-rgw-api-host <host>
$ ceph dashboard set-rgw-api-port <port>
```

除了到目前为止提到的设置之外，还存在以下设置，您可能会遇到必须使用它们的情况：

```text
$ ceph dashboard set-rgw-api-scheme <scheme>  # http or https
$ ceph dashboard set-rgw-api-admin-resource <admin_resource>
$ ceph dashboard set-rgw-api-user-id <user_id>
```

如果在对象网关设置中使用自签名证书，则应在仪表板中禁用证书验证，以避免连接被拒绝，例如，由未知CA签名的证书或与主机名不匹配引起的连接被拒绝：

```text
$ ceph dashboard set-rgw-api-ssl-verify False
```

如果对象网关处理请求的时间太长，并且仪表板发生超时，则可以根据需要设置超时值：

```text
$ ceph dashboard set-rest-requests-timeout <seconds>
```

默认值为45秒。

#### 启用ISCSI管理

Ceph仪表板可以使用由[Ceph iSCSI Gateway](https://docs.ceph.com/docs/nautilus/rbd/iscsi-overview/#ceph-iscsi)的rbd-target-api服务提供的REST API管理iSCSI目标 。请确保已在iSCSI网关上安装并启用了它。

注意 

Ceph仪表板的iSCSI管理功能取决于[ceph-iscsi](https://github.com/ceph/ceph-iscsi)项目的最新版本3 。确保您的操作系统提供了正确的版本，否则仪表板将无法启用管理功能。

如果ceph-iscsi REST API是在HTTPS模式下配置的，并且使用自签名证书，那么您需要配置仪表板，以避免在访问ceph-iscsi API时进行SSL证书验证。

要禁用API SSL验证，请运行以下命令：

```text
$ ceph dashboard set-iscsi-api-ssl-verification false
```

必须使用以下命令定义可用的iSCSI网关：

```text
$ ceph dashboard iscsi-gateway-list
$ ceph dashboard iscsi-gateway-add <scheme>://<username>:<password>@<host>[:port]
$ ceph dashboard iscsi-gateway-rm <gateway_name>
```

#### 启用GRAFANA仪表板的嵌入

Grafana和Prometheus可能会在不久的将来通过Ceph的一些编排工具进行捆绑和安装，但是目前，您将必须手动安装和配置这两者。在首选主机上安装Prometheus和Grafana之后，请继续以下步骤。

1. 通过运行以下命令启用作为Ceph Manager模块附带的Ceph Exporter：

   ```text
   $ ceph mgr module enable prometheus
   ```

可以在[Prometheus模块](https://docs.ceph.com/docs/nautilus/mgr/prometheus/#mgr-prometheus)的文档中找到更多详细信息。

1. 将相应的抓取配置添加到Prometheus。可能看起来像：

   ```text
   global:
     scrape_interval: 5s

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
     - job_name: 'ceph'
       static_configs:
         - targets: ['localhost:9283']
     - job_name: 'node-exporter'
       static_configs:
         - targets: ['localhost:9100']
   ```

2. 将Prometheus作为数据源添加到Grafana
3. 使用以下命令安装vonage-status-panel和grafana-piechart-panel插件：

   ```text
   grafana-cli plugins install vonage-status-panel
   grafana-cli plugins install grafana-piechart-panel
   ```

4. 将仪表板添加到Grafana：

   可以通过导入仪表板json将仪表板添加到Grafana。以下命令可用于下载json文件：

   ```text
   wget https://raw.githubusercontent.com/ceph/ceph/master/monitoring/grafana/dashboards/<Dashboard-name>.json
   ```

   您可以[在此处](https://github.com/ceph/ceph/tree/master/monitoring/grafana/dashboards)找到所有仪表板json 。

   例如，对于ceph-cluster概述，您可以使用：

   ```text
   wget https://raw.githubusercontent.com/ceph/ceph/master/monitoring/grafana/dashboards/ceph-cluster.json
   ```

5. 在/etc/grafana/grafana.ini中配置Grafana 以适应匿名模式：

   ```text
   [auth.anonymous]
   enabled = true
   org_name = Main Org.
   org_role = Viewer
   ```

设置Grafana和Prometheus之后，您将需要配置Ceph仪表板将用来访问Grafana的连接信息。

您需要告诉仪表板哪个URL Grafana实例正在运行/部署：

```text
$ ceph dashboard set-grafana-api-url <grafana-server-url>  # default: ''
```

url的格式为：&lt;协议&gt;：&lt;IP地址&gt;：&lt;端口&gt;

注意 

Ceph仪表板通过`iframe`HTML元素嵌入Grafana仪表板。如果Grafana配置为不支持SSL / TLS，则在启用了仪表盘中的SSL支持（默认配置）的情况下，大多数浏览器都会阻止将不安全内容嵌入到安全网页中。如果如上所述启用嵌入式Grafana仪表板后看不到它们，请查看浏览器的文档以了解如何取消阻止混合内容。或者，考虑在Grafana中启用SSL / TLS支持。

如果在Grafana设置中使用自签名证书，则应在仪表板中禁用证书验证，以避免连接被拒绝，例如，由未知CA签名的证书或与主机名不匹配引起的连接被拒绝：

```text
$ ceph dashboard set-grafana-api-ssl-verify False
```

您也可以直接访问Grafana实例来监视您的集群。

#### 启用单点登录（SSO）

Ceph仪表板支持通过[SAML 2.0](https://en.wikipedia.org/wiki/SAML_2.0)协议对用户进行外部身份验证 。您需要创建用户帐户，并首先将它们与所需的角色相关联，因为授权仍由仪表板执行。但是，身份验证过程可以由现有的身份提供商（IdP）执行。

注意 

Ceph仪表板SSO支持依赖onelogin的 [python-saml](https://pypi.org/project/python-saml/)库。请确保通过使用发行版的软件包管理程序或通过Python的pip安装程序将该库安装在您的系统上。

要在Ceph仪表板上配置SSO，您应该使用以下命令：

```text
$ ceph dashboard sso setup saml2 <ceph_dashboard_base_url> <idp_metadata> {<idp_username_attribute>} {<idp_entity_id>} {<sp_x_509_cert>} {<sp_private_key>}
```

参数：

* **&lt;ceph\_dashboard\_base\_url&gt;**：可访问Ceph仪表板的基本URL（例如https：//cephdashboard.local）
* **&lt;idp\_metadata&gt;**：IdP元数据XML的URL，文件路径或内容（例如https：// myidp / metadata）
* **&lt;idp\_username\_attribute&gt;** _（可选）_：应该用于从身份验证响应中获取用户名的属性。默认为uid。
* **&lt;idp\_entity\_id&gt;** _（可选）_：如果IdP元数据上存在多个实体ID，请使用此ID。
* **&lt;sp\_x\_509\_cert&gt; / &lt;sp\_private\_key&gt;** _（可选）_：Ceph仪表板（服务提供商）用于签名和加密的证书的文件路径或内容。

注意 

SAML请求的发行者值将遵循以下模式： **&lt;ceph\_dashboard\_base\_url&gt;** / auth / saml2 / metadata

要显示当前的SAML 2.0配置，请使用以下命令：

```text
$ ceph dashboard sso show saml2
```

注意 

有关onelogin\_settings的更多信息，请查看[onelogin文档](https://github.com/onelogin/python-saml)。

要禁用SSO：

```text
$ ceph dashboard sso disable
```

要检查是否启用了SSO：

```text
$ ceph dashboard sso status
```

要启用SSO：

```text
$ ceph dashboard sso enable saml2
```

#### 启用PROMETHEUS警报

使用Prometheus进行监视，您必须定义[警报规则](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules)。要管理它们，您需要使用[Alertmanager](https://prometheus.io/docs/alerting/alertmanager)。如果您尚未使用Alertmanager，请强制[安装它](https://github.com/prometheus/alertmanager#install)，以接收和管理来自Prometheus的警报。

仪表板可以通过三种不同方式使用Alertmanager功能：

1. 使用仪表板的通知接收器。
2. 使用Prometheus Alertmanager API。
3. 同时使用两个来源。

这三种方法都会通知您有关警报的信息。如果您同时使用这两种资源，都不会收到两次通知，但是您至少需要使用Alertmanager API来管理静音。

1. 使用仪表板的通知接收器：

   这使您可以从Alertmanager [配置](https://prometheus.io/docs/alerting/configuration/)获取通知。发出通知后，您将在仪表板内收到通知，但您将无法管理警报。

   将仪表板接收器和新路由添加到Alertmanager配置。看起来应该像这样：

   ```text
   route:
     receiver: 'ceph-dashboard'
   ...
   receivers:
     - name: 'ceph-dashboard'
       webhook_configs:
       - url: '<url-to-dashboard>/api/prometheus_receiver'
   ```

   请确保Alertmanager就仪表板而言将您的SSL证书视为有效。有关正确配置的更多信息，请[参见&lt;http\_config&gt;文档](https://prometheus.io/docs/alerting/configuration/#%3Chttp_config%3E)。

2. 使用Prometheus和Alertmanager的API

   这使您可以管理警报和静音。这将启用“群集”菜单项的“监视”部分中的“活动警报”，“所有警报”以及“沉默”选项卡。

   警报可以按名称，作业，严重性，状态和开始时间进行排序。不幸的是，不可能知道Alertmanager根据您的配置通过通知发送警报的时间，这就是为什么仪表板会在警报的任何可见更改时通知用户并通知更改的警报。

   沉默可以按ID，创建者，状态，开始，更新和结束时间进行排序。可以通过多种方式创建沉默，也可以使它们过期。

   1. 从头开始创建
   2. 基于选定的警报
   3. 从沉默中重建
   4. 更新静默（将重新创建静默并使之失效（默认Alertmanager行为））

   要使用它，请指定Alertmanager服务器的主机和端口：

   ```text
   $ ceph dashboard set-alertmanager-api-host <alertmanager-host:port>  # default: ''
   ```

   例如：

   ```text
   $ ceph dashboard set-alertmanager-api-host 'http://localhost:9093'
   ```

   为了能够查看所有已配置的警报，您将需要配置Prometheus API的URL。使用此API，UI还将帮助您验证新的静音是否与相应的警报匹配。

   ```text
   $ ceph dashboard set-prometheus-api-host <prometheus-host:port>  # default: ''
   ```

   例如：

   ```text
   $ ceph dashboard set-prometheus-api-host 'http://localhost:9090'
   ```

   设置主机后，您必须在浏览器窗口中刷新仪表板。

3. 同时使用两种方法

   两种方法的不同行为配置为，它们不应通过弹出讨厌的重复通知而相互干扰。

#### 访问仪表板

现在，您可以使用（启用JavaScript的）网络浏览器访问仪表板，方法是将其指向任何主机名或IP地址以及运行管理器实例的所选TCP端口：例如，`httpS://<$IP>:<$PORT>/`。

然后，应该通过仪表板登录页面向您打招呼，要求您提供先前定义的用户名和密码。 如果您想在以后访问仪表板时跳过用户名/密码请求，**请**选中“ **保持登录状态”**复选框。

### 用户和角色管理

#### 用户帐户

Ceph仪表板支持管理多个用户帐户。每个用户帐户都包含一个用户名，一个密码（使用加密形式存储的密码`bcrypt`），一个可选名称和一个可选电子邮件地址。

用户帐户存储在MON的配置数据库中，并且在所有ceph-mgr实例之间全局共享。

我们提供了一组CLI命令来管理用户帐户：

* _显示用户_：

  ```text
  $ ceph dashboard ac-user-show [<username>]
  ```

* _创建用户_：

  ```text
  $ ceph dashboard ac-user-create <username> [<password>] [<rolename>] [<name>] [<email>]
  ```

* _删除用户_：

  ```text
  $ ceph dashboard ac-user-delete <username>
  ```

* _修改密码_：

  ```text
  $ ceph dashboard ac-user-set-password <username> <password>
  ```

* _修改用户（名称和电子邮件）_：

  ```text
  $ ceph dashboard ac-user-set-info <username> <name> <email>
  ```

#### 用户角色和权限

用户帐户还与一组角色相关联，这些角色定义了用户可以访问哪些仪表板功能。

仪表板功能/模块在_安全范围内_分组。安全范围是预定义的和静态的。当前可用的安全范围是：

* **主机**：包括与`Hosts`菜单项相关的所有功能。
* **config-opt**：包括与Ceph配置选项管理相关的所有功能。
* **池**：包括与池管理有关的所有功能。
* **osd**：包括与OSD管理相关的所有功能。
* **monitor**：包括与Monitor管理相关的所有功能。
* **rbd-image**：包括与RBD图像管理相关的所有功能。
* **rbd-mirroring**：包括与RBD镜像管理相关的所有功能。
* **iscsi**：包括与iSCSI管理相关的所有功能。
* **rgw**：包括与Rados Gateway管理相关的所有功能。
* **cephfs**：包括与CephFS管理相关的所有功能。
* **manager**：包括与Ceph Manager管理相关的所有功能。
* **log**：包括与Ceph日志管理相关的所有功能。
* **grafana**：包括与Grafana代理相关的所有功能。
* **prometheus**：包括与Prometheus警报管理相关的所有功能。
* **仪表板设置**：允许更改仪表板设置。

一个_角色_指定了一组之间的映射的_安全范围_和一组 _权限_。有四种类型的权限：

* **读**
* **创建**
* **更新**
* **删除**

请参阅以下基于Python词典的角色规范示例：

```text
# example of a role
{
  'role': 'my_new_role',
  'description': 'My new role',
  'scopes_permissions': {
    'pool': ['read', 'create'],
    'rbd-image': ['read', 'create', 'update', 'delete']
  }
}
```

上面的角色指示用户具有与池管理相关的功能的_读取_和_创建_权限，并且对与RBD图像管理相关的功能具有完全权限。

仪表板已经提供了一组预定义的角色，我们称为 _系统角色_，并且可以在全新的Ceph仪表板安装中立即使用。

系统角色列表为：

* **管理员**：为所有安全范围提供完全权限。
* **只读**：为除仪表板设置以外的所有安全范围提供_读取_权限。
* **block-manager**：提供_rbd-image_， _rbd-mirroring_和_iscsi_范围的完整权限。
* **rgw-manager**：为_rgw_范围提供完整权限
* **cluster-manager**：提供对_host_，_osd_， _monitor_，_manager_和_config-opt_范围的完整权限。
* **pool-manager**：提供_池_范围的完整权限。
* **cephfs-manager**：提供对_cephfs_范围的完整权限。

可以通过以下命令检索当前可用角色的列表：

```text
$ ceph dashboard ac-role-show [<rolename>]
```

也可以使用CLI命令创建新角色。用于管理角色的可用命令如下：

* _创建角色_：

  ```text
  $ ceph dashboard ac-role-create <rolename> [<description>]
  ```

* _删除角色_：

  ```text
  $ ceph dashboard ac-role-delete <rolename>
  ```

* _将作用域权限添加到角色_：

  ```text
  $ ceph dashboard ac-role-add-scope-perms <rolename> <scopename> <permission> [<permission>...]
  ```

* _从角色中删除范围权限_：

  ```text
  $ ceph dashboard ac-role-del-perms <rolename> <scopename>
  ```

要将角色与用户相关联，可以使用以下CLI命令：

* _设置用户角色_：

  ```text
  $ ceph dashboard ac-user-set-roles <username> <rolename> [<rolename>...]
  ```

* _向用户添加角色_：

  ```text
  $ ceph dashboard ac-user-add-roles <username> <rolename> [<rolename>...]
  ```

* _从用户删除角色_：

  ```text
  $ ceph dashboard ac-user-del-roles <username> <rolename> [<rolename>...]
  ```

#### 用户和自定义角色创建示例

在本节中，我们显示了用于创建用户帐户所需命令的完整示例，该命令应能够管理RBD映像，查看和创建Ceph池，并且对任何其他作用域具有只读访问权限。

1. _创建用户_：

   ```text
   $ ceph dashboard ac-user-create bob mypassword
   ```

2. _创建角色并指定范围权限_：

   ```text
   $ ceph dashboard ac-role-create rbd/pool-manager
   $ ceph dashboard ac-role-add-scope-perms rbd/pool-manager rbd-image read create update delete
   $ ceph dashboard ac-role-add-scope-perms rbd/pool-manager pool read create
   ```

3. _将角色与用户相关联_：

   ```text
   $ ceph dashboard ac-user-set-roles bob rbd/pool-manager read-only
   ```

### 代理配置

在具有多个ceph-mgr实例的Ceph集群中，仅在当前活动的ceph-mgr守护程序上运行的仪表板将处理传入的请求。访问当前处于待机状态的任何其他ceph-mgr实例上的仪表板的TCP端口将对当前活动的管理器的仪表板URL执行HTTP重定向（303）。这样，您可以将浏览器指向任何ceph-mgr实例，以便访问仪表板。

如果要建立一个固定的URL来访问仪表板，或者不想直接连接到管理器节点，则可以设置一个代理，该代理将传入的请求自动转发到当前活动的ceph-mgr实例。

#### 配置URL前缀

如果要通过反向代理配置访问仪表板，则可能希望使用URL前缀对其进行服务。要使仪表板使用包含前缀的超链接，可以设置以下 `url_prefix`设置：

```text
ceph config set mgr mgr/dashboard/url_prefix $PREFIX
```

因此您可以通过访问仪表板`http://$IP:$PORT/$PREFIX/`。

#### 禁用重定向

如果仪表板位于诸如[HAProxy之](https://www.haproxy.org/)类的负载均衡代理的后面，则 您可能希望禁用重定向行为，以防止将内部（无法解析的）URL发布到前端客户端的情况。使用以下命令来使仪表板响应HTTP错误（默认为500），而不是重定向到活动的仪表板：

```text
$ ceph config set mgr mgr/dashboard/standby_behaviour "error"
```

要将设置重置为默认的重定向行为，请使用以下命令：

```text
$ ceph config set mgr mgr/dashboard/standby_behaviour "redirect"
```

#### 配置错误状态码

禁用重定向行为后，您要自定义备用仪表板的HTTP状态代码。为此，您需要运行以下命令：

```text
$ ceph config set mgr mgr/dashboard/standby_error_status_code 503
```

#### HAPROXY示例配置

在下面，您将找到使用[HAProxy](https://www.haproxy.org/)进行SSL / TLS传递的示例配置 。

请注意，该配置在以下条件下有效。如果仪表板进行故障转移，则前端客户端可能会收到HTTP重定向（303）响应，并将重定向到无法解析的主机。当两次HAProxy运行状况检查期间发生故障转移时，就会发生这种情况。在这种情况下，先前处于活动状态的仪表板节点现在将以303响应，该指向新的活动节点。为避免这种情况，您应该考虑在备用节点上禁用重定向行为。

```text
defaults
  log global
  option log-health-checks
  timeout connect 5s
  timeout client 50s
  timeout server 450s

frontend dashboard_front
  mode http
  bind *:80
  option httplog
  redirect scheme https code 301 if !{ ssl_fc }

frontend dashboard_front_ssl
  mode tcp
  bind *:443
  option tcplog
  default_backend dashboard_back_ssl

backend dashboard_back_ssl
  mode tcp
  option httpchk GET /
  http-check expect status 200
  server x <HOST>:<PORT> check-ssl check verify none
  server y <HOST>:<PORT> check-ssl check verify none
  server z <HOST>:<PORT> check-ssl check verify none
```

### 审核API请求

REST API能够将PUT，POST和DELETE请求记录到Ceph审核日志中。默认情况下，此功能是禁用的，但可以使用以下命令启用：

```text
$ ceph dashboard set-audit-api-enabled <true|false>
```

如果启用，则每个请求记录以下参数：

* [from-](https://[::1]:44410/)请求的来源，例如[https：// \[:: 1\]：44410](https://[::1]:44410/)
* path-REST API路径，例如/ api / auth
* 方法-例如PUT，POST或DELETE
* 用户-用户名，否则为“无”

默认情况下，启用请求有效负载（参数及其值）的日志记录。执行以下命令以禁用此行为：

```text
$ ceph dashboard set-audit-api-log-payload <true|false>
```

日志条目可能如下所示：

```text
2018-10-22 15:27:01.302514 mgr.x [INF] [DASHBOARD] from='https://[::ffff:127.0.0.1]:37022' path='/api/rgw/user/klaus' method='PUT' user='admin' params='{"max_buckets": "1000", "display_name": "Klaus Mustermann", "uid": "klaus", "suspended": "0", "email": "klaus.mustermann@ceph.com"}'
```

### NFS-GANESHA管理

Ceph仪表板可以管理使用CephFS或RadosGW作为其后备存储的[NFS Ganesha](http://nfs-ganesha.github.io/)导出。

要在Ceph仪表板中启用此功能，关于NFS-Ganesha服务的配置方式需要满足一些假设。

仪表板管理存储在Ceph群集上的RADOS对象中的NFS-Ganesha配置文件。NFS-Ganesha必须将其部分配置存储在Ceph集群中。

这些配置文件必须遵循一些约定。约定。每个导出块必须存储在其自己的名为的RADOS对象中`export-<id>`，该对象 `<id>`必须与`Export_ID`导出配置的属性匹配。然后，对于每个NFS-Ganesha服务守护程序，应该存在一个名为的RADOS对象`conf-<daemon_id>`，其中`<daemon_id>`是一个任意字符串，应唯一标识该守护程序实例（例如，守护程序运行所在的主机名）。每个`conf-<daemon_id>`对象都包含指向NFS-Ganesha守护程序应服务的导出的RADOS URL。这些URL的格式为：

```text
%url rados://<pool_name>[/<namespace>]/export-<id>
```

无论是`conf-<daemon_id>`和`export-<id>`对象必须存储在同一个RADOS泳池/命名空间。

#### 在仪表板配置NFS-象头

要在Ceph仪表板中启用对NFS-Ganesha导出的管理，我们只需要告诉仪表板，配置对象存储在RADOS池和命名空间中。然后，Ceph仪表板可以遵循上述命名约定来访问对象。

用于配置NFS-Ganesha配置对象位置的仪表板命令为：

```text
$ ceph dashboard set-ganesha-clusters-rados-pool-namespace <pool_name>[/<namespace>]
```

运行上述命令后，Ceph仪表板便能够找到NFS-Ganesha配置对象，并且我们可以开始通过Web UI管理导出。

#### 支持多个NFS-GANESHA集群

Ceph仪表板还支持管理属于不同NFS-Ganesha集群的NFS-Ganesha导出。NFS-Ganesha群集是共享相同导出的一组NFS-Ganesha服务守护程序。不同的NFS-Ganesha群集是独立的，彼此之间不共享导出配置。

每个NFS-Ganesha群集都应将其配置对象存储在不同的RADOS池/命名空间中，以使配置彼此隔离。

要指定每个NFS-Ganesha集群的配置位置，我们可以使用与上述相同的命令，但使用不同的值模式：

```text
$ ceph dashboard set-ganesha-clusters-rados-pool-namespace <cluster_id>:<pool_name>[/<namespace>](,<cluster_id>:<pool_name>[/<namespace>])*
```

该`<cluster_id>`是应该唯一标识NFS-象头神集群的任意字符串。

当使用多个NFS-Ganesha群集配置Ceph仪表板时，Web UI将自动允许选择导出所属的群集。

### 插件

仪表板插件允许以模块化和松散耦合的方式扩展仪表板的功能。

#### 功能切换

该插件允许按需启用或禁用Ceph仪表板的某些功能。禁用某个功能时：

* 它的前端元素（网页，菜单项，图表等）将被隐藏。
* 其关联的REST API端点将拒绝任何其他请求（404，未找到错误）。

该插件的主要目的是允许对仪表板公开的工作流进行临时自定义。另外，它可以允许以最小的配置负担和不影响服务的方式动态启用实验性功能。

可以启用/禁用的功能列表为：

* **区块（RBD）**：
  * 影像管理： `rbd`
  * 镜像： `mirroring`
  * iSCSI： `iscsi`
* **文件系统（Cephfs）**：`cephfs`
* **对象（RGW）**：（`rgw`包括守护程序，用户和存储桶管理）。

默认情况下，所有功能都启用。

要检索功能及其当前状态的列表：

```text
$ ceph dashboard feature status
Feature 'cephfs': 'enabled'
Feature 'iscsi': 'enabled'
Feature 'mirroring': 'enabled'
Feature 'rbd': 'enabled'
Feature 'rgw': 'enabled'
```

要启用或禁用单个或多个功能的状态：

```text
$ ceph dashboard feature disable iscsi mirroring
Feature 'iscsi': disabled
Feature 'mirroring': disabled
```

功能状态更改后，API REST端点会立即响应该更改，而对于前端UI元素，最多可能需要20秒才能反映出来。

