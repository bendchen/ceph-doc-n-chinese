# PUBSUB同步模块

## PUBSUB同步模块

Nautilus版本中的新功能。

内容

此同步模块为对象存储库修改事件提供发布和订阅机制。事件被发布到预定义的主题中。可以订阅主题，并可以从中提取事件。需要确认事件。同样，事件将过期并在一段时间后消失。

推送通知机制也存在，目前支持HTTP，AMQP0.9.1和Kafka端点。在这种情况下，事件将被推送到终结点，然后再将事件存储在Ceph中。如果事件仅应推送到端点，而无需存储在Ceph中，则应使用[Bucket Notification](https://docs.ceph.com/docs/nautilus/radosgw/notifications)机制而不是pubsub sync模块。

用户可以创建不同的主题。主题实体由其用户和名称定义。用户只能管理自己的主题，并且只能订阅其拥有的存储桶发布的事件。

为了发布特定存储桶的事件，需要创建一个通知实体。可以在事件类型的子集或所有事件类型上创建通知（默认）。任何特定主题可以有多个通知，并且同一主题可以用于多个通知。

还可以定义对主题的订阅。任何特定主题都可以有多个订阅。

REST API已被定义为pubsub机制提供配置和控制接口。该API有两种风格，一种是S3兼容的，一种不是。两种口味可以一起使用，尽管建议使用与S3兼容的一种。与S3兼容的API与存储桶通知机制中使用的API相似。

事件以RGW对象的形式存储在特殊用户的特殊存储区中。无法直接访问事件，但是需要使用新的REST API来提取和确认事件。

* [S3存储桶通知兼容性](https://docs.ceph.com/docs/nautilus/radosgw/s3-notification-compatibility/)

### [PUBSUB区域配置](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module/#id2)

pubsub同步模块要求在多[站点](https://docs.ceph.com/docs/nautilus/radosgw/multisite)环境中创建一个新区域。首先，必须存在一个主区域，然后应创建一个辅助区域。在创建辅助区域时，其层类型必须设置为`pubsub`：

```text
# radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --endpoints={http://fqdn}[,{http://fqdn}] \
                            --sync-from-all=0 \
                            --sync-from={master-zone-name} \
                            --tier-type=pubsub
```

#### [PUBSUB区域配置参数](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module/#id3)

```text
{
    "tenant": <tenant>,             # default: <empty>
    "uid": <uid>,                   # default: "pubsub"
    "data_bucket_prefix": <prefix>  # default: "pubsub-"
    "data_oid_prefix": <prefix>     #
    "events_retention_days": <days> # default: 7
}
```

* `tenant` （串）

pubsub控件用户的租户。

* `uid` （串）

pubsub控件用户的uid。

* `data_bucket_prefix` （串）

存储区名称的前缀，将创建该存储区名称以存储特定主题的事件。

* `data_oid_prefix` （串）

存储事件的oid前缀。

* `events_retention_days` （整数）

保留未确认事件的天数。

#### 通过CLI配置参数

可以使用以下命令来设置层配置：

```text
# radosgw-admin zone modify --rgw-zonegroup={zone-group-name} \
                             --rgw-zone={zone-name} \
                             --tier-config={key}={val}[,{key}={val}]
```

在`key`配置中的处指定需要更新的配置变量（从上面的列表中），并`val`指定其新值。例如，将pubsub控件用户设置`uid`为`user_ps`：

```text
# radosgw-admin zone modify --rgw-zonegroup={zone-group-name} \
                             --rgw-zone={zone-name} \
                             --tier-config=uid=pubsub
```

可以使用删除配置字段`--tier-config-rm={key}`。

### PUBSUB性能统计信息

pubsub同步模块和通知机制之间共享相同的计数器。

* `pubsub_event_triggered`：运行事件计数器，但至少要关联一个主题
* `pubsub_event_lost`：对具有相关主题和订阅但未存储或推送到任何订阅的事件进行计数
* `pubsub_store_ok`：运行计数器，用于所有订阅的存储事件
* `pubsub_store_fail`：对于所有订阅，事件的运行计数器无法存储
* `pubsub_push_ok`：为所有订阅运行计数器，将事件成功推送到其端点
* `pubsub_push_fail`：对于所有订阅，事件的运行计数器未能推送到其端点
* `pubsub_push_pending`：衡量推送到端点但尚未确认或尚未确认的事件的值

注意 

`pubsub_event_triggered`和`pubsub_event_lost`每个事件被递增，而： `pubsub_store_ok`，`pubsub_store_fail`，`pubsub_push_ok`，`pubsub_push_fail`，的每存储/推动作开始，每个订阅。

### PUBSUB的REST API 

小费 

只能将PubSub REST调用发送到属于PubSub区域的RGW

#### 主题

**创建一个话题**

这将创建一个新主题。两种API都需要创建主题。可选地，可以为主题提供推送端点参数，稍后将在创建S3兼容通知时使用该参数。成功请求后，响应将包含主题ARN，该主题可在以后与S3兼容的通知请求中用来引用此主题。要更新主题，请使用用于主题创建的相同命令，并使用现有主题的主题名称和不同的端点值。

小费 

必须重新创建与该主题相关联的任何与S3兼容的通知，以使主题更新生效

```text
PUT /topics/<topic-name>[?OpaqueData=<opaque data>][&push-endpoint=<endpoint>[&amqp-exchange=<exchange>][&amqp-ack-level=none|broker][&verify-ssl=true|false][&kafka-ack-level=none|broker][&use-ssl=true|false][&ca-location=<file path>]]
```

请求参数：

* push-endpoint：向其发送推送通知的端点的URI
* OpaqueData：在主题配置中设置不透明数据，并将其添加到由ropic触发的所有通知中

端点URI可能包含参数，具体取决于端点的类型：

* HTTP端点

> * URI： `http[s]://<fqdn>[:<port]`
> * 端口默认为：80/443，用于HTTP / S
> * verify-ssl：指示服务器证书是否由客户端验证（默认为“ true”）

* AMQP0.9.1端点

> * URI： `amqp://[<user>:<password>@]<fqdn>[:<port>][/<vhost>]`
> * 用户/密码默认为：访客/访客
> * 用户/密码只能通过HTTPS提供。否则，主题创建请求将被拒绝
> * 端口默认为：5672
> * vhost默认为：“ /”
> * amqp-exchange：交换局必须存在并且能够基于主题路由消息（AMQP0.9.1的强制参数）
> * amqp-ack级别：不需要end2end acking，因为消息在传递到最终目的地之前可能会保留在代理中。存在两种确认方法：
>
> > * “无”：如果发送给代理，则消息被视为“已发送”
> > * “经纪人”：如果消息被经纪人确认，则邮件被视为“已传递”（默认）

* Kafka端点

> * URI： `kafka://[<user>:<password>@]<fqdn>[:<port]`
> * 如果`use-ssl`设置为“ true”，则将使用安全连接与代理进行连接（默认情况下为“ false”）
> * 如果`ca-location`提供了，并且使用了安全连接，则将使用指定的CA（而非默认的CA）对代理进行身份验证
> * 用户/密码只能通过HTTPS提供。否则，主题创建请求将被拒绝
> * 用户/密码只能与一起提供`use-ssl`，否则，与代理的连接将失败
> * 端口默认为：9092
> * kafka-ack级别：不需要end2end acking，因为消息在传递到最终目的地之前可能会保留在代理中。存在两种确认方法：
>
> > * “无”：如果发送给代理，则消息被视为“已发送”
> > * “经纪人”：如果消息被经纪人确认，则邮件被视为“已传递”（默认）

响应中的主题ARN将具有以下格式：

```text
arn:aws:sns:<zone-group>:<tenant>:<topic>
```

**获取主题信息**

返回有关特定主题的信息。这包括对该主题的订阅以及推送端点信息（如果提供）。

```text
GET /topics/<topic-name>
```

响应将具有以下格式（JSON）：

```text
{
    "topic":{
        "user":"",
        "name":"",
        "dest":{
            "bucket_name":"",
            "oid_prefix":"",
            "push_endpoint":"",
            "push_endpoint_args":"",
            "push_endpoint_topic":""
        },
        "arn":""
        "opaqueData":""
    },
    "subs":[]
}
```

* topic.user：创建主题的用户名
* 名称：主题名称
* dest.bucket\_name：未使用
* dest.oid\_prefix：未使用
* dest.push\_endpoint：在符合S3的通知的情况下，此值将用作推送端点URL
* 如果推端点URL包含用户/密码信息，则必须通过HTTPS发出请求。如果不是，则主题获取请求将被拒绝
* dest.push\_endpoint\_args：在符合S3的通知的情况下，此值将用作推送端点args
* dest.push\_endpoint\_topic：在符合S3的通知的情况下，此值将保存发送到端点的主题名称（可能与内部主题名称不同）
* topic.arn：主题ARN
* 订阅：与此主题相关的订阅列表

**删除主题**

```text
DELETE /topics/<topic-name>
```

删除指定的主题。

**列出主题**

列出用户定义的所有主题。

```text
GET /topics
```

* 如果推端点URL包含用户/密码信息，则在任何主题中，都必须通过HTTPS发出请求。否则，主题列表请求将被拒绝

#### 符合S3的通知

详细说明：[BUCKET OPERATIONS](https://docs.ceph.com/docs/nautilus/radosgw/s3/bucketops/)。

注意

* 通知创建还将创建用于推送/拉动事件的订阅
* 生成的订阅名称将与通知ID相同，以后可用于通过订阅API来获取和确认事件。
* 删除通知将删除所有生成的订阅
* 如果存储桶删除隐式删除了通知，关联的订阅将不会自动删除（删除的存储桶的任何事件仍可访问），并且必须使用订阅删除API显式删除
* 不支持基于元数据（这是S3的扩展）的过滤，并且这些规则将被忽略
* 不支持基于标签的过滤（这是S3的扩展），这些规则将被忽略

#### [不符合S3的通知](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module/#id13)

**创建通知**

这将为特定主题的主题创建一个发布者。

```text
PUT /notifications/bucket/<bucket>?topic=<topic-name>[&events=<event>[,<event>]]
```

请求参数：

* topic-name：主题名称
* 事件：事件类型（字符串），中的一种：`OBJECT_CREATE`，`OBJECT_DELETE`，`DELETE_MARKER_CREATE`

**删除通知信息**

将发布者从特定存储桶中删除到特定主题中。

```text
DELETE /notifications/bucket/<bucket>?topic=<topic-name>
```

请求参数：

* topic-name：主题名称

注意 

删除存储桶时，其上定义的所有通知也会被删除

**名单通知**

列出在存储桶上定义了关联事件的所有主题。

```text
GET /notifications/bucket/<bucket>
```

响应将具有以下格式（JSON）：

```text
{"topics":[
   {
      "topic":{
         "user":"",
         "name":"",
         "dest":{
            "bucket_name":"",
            "oid_prefix":"",
            "push_endpoint":"",
            "push_endpoint_args":"",
            "push_endpoint_topic":""
         }
         "arn":""
      },
      "events":[]
   }
 ]}
```

#### [订阅](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module/#id17)

**创建订阅**

创建一个新的订阅。

```text
PUT /subscriptions/<sub-name>?topic=<topic-name>[?push-endpoint=<endpoint>[&amqp-exchange=<exchange>][&amqp-ack-level=none|broker][&verify-ssl=true|false][&kafka-ack-level=none|broker][&ca-location=<file path>]]
```

请求参数：

* topic-name：主题名称
* push-endpoint：向其发送推送通知的端点的URI

端点URI可能包含参数，具体取决于端点的类型：

* HTTP端点

> * URI： `http[s]://<fqdn>[:<port]`
> * 端口默认为：80/443，用于HTTP / S
> * verify-ssl：指示服务器证书是否由客户端验证（默认为“ true”）

* AMQP0.9.1端点

> * URI： `amqp://[<user>:<password>@]<fqdn>[:<port>][/<vhost>]`
> * 用户/密码默认为：访客/访客
> * 端口默认为：5672
> * vhost默认为：“ /”
> * amqp-exchange：交换局必须存在并且能够基于主题路由消息（AMQP0.9.1的强制参数）
> * amqp-ack级别：不需要end2end acking，因为消息在传递到最终目的地之前可能会保留在代理中。存在两种确认方法：
>
> > * “无”：如果发送给代理，则消息被视为“已发送”
> > * “经纪人”：如果消息被经纪人确认，则邮件被视为“已传递”（默认）

* Kafka端点

> * URI： `kafka://[<user>:<password>@]<fqdn>[:<port]`
> * 如果`ca-location`提供，则将使用安全连接与代理进行连接
> * 用户/密码只能通过HTTPS提供。否则，主题创建请求将被拒绝
> * 用户/密码只能与一起提供`ca-location`。否则，主题创建请求将被拒绝
> * 端口默认为：9092
> * kafka-ack级别：不需要end2end acking，因为消息在传递到最终目的地之前可能会保留在代理中。存在两种确认方法：
>
> > * “无”：如果发送给代理，则消息被视为“已发送”
> > * “经纪人”：如果消息被经纪人确认，则邮件被视为“已传递”（默认）

**获取订阅信息**

返回有关特定订阅的信息。

```text
GET /subscriptions/<sub-name>
```

响应将具有以下格式（JSON）：

```text
{
    "user":"",
    "name":"",
    "topic":"",
    "dest":{
        "bucket_name":"",
        "oid_prefix":"",
        "push_endpoint":"",
        "push_endpoint_args":"",
        "push_endpoint_topic":""
    }
    "s3_id":""
}
```

* 用户：创建订阅的用户名
* 名称：订阅名称
* topic：与订阅相关联的主题的名称
* dest.bucket\_name：存储事件的桶的名称
* dest.oid\_prefix：存储在存储桶中的事件的oid前缀
* dest.push\_endpoint：在符合S3的通知的情况下，此值将用作推送端点URL
* 如果推端点URL包含用户/密码信息，则必须通过HTTPS发出请求。如果不是，则主题获取请求将被拒绝
* dest.push\_endpoint\_args：在符合S3的通知的情况下，此值将用作推送端点args
* dest.push\_endpoint\_topic：在符合S3的通知的情况下，此值将保存发送到端点的主题名称（可能与内部主题名称不同）
* s3\_id：对于符合S3的通知，它将保留创建订阅的通知名称

**删除订阅**

删除订阅。

```text
DELETE /subscriptions/<sub-name>
```

#### [事件](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module/#id21)

**拉事件**

拉事件发送到特定的订阅。

```text
GET /subscriptions/<sub-name>?events[&max-entries=<max-entries>][&marker=<marker>]
```

请求参数：

* marker：事件列表的分页标记，如果未指定，则从最早的开始
* max-entries：要返回的最大事件数

响应将保存有关当前标记的信息以及是否还有其他未提取的事件：

```text
{"next_marker":"","is_truncated":"",...}
```

响应的实际内容取决于创建订阅的方式。如果订阅是通过S3兼容的通知创建的，则事件将具有S3兼容的记录格式（JSON）：

```text
{"Records":[
    {
        "eventVersion":"2.1"
        "eventSource":"aws:s3",
        "awsRegion":"",
        "eventTime":"",
        "eventName":"",
        "userIdentity":{
            "principalId":""
        },
        "requestParameters":{
            "sourceIPAddress":""
        },
        "responseElements":{
            "x-amz-request-id":"",
            "x-amz-id-2":""
        },
        "s3":{
            "s3SchemaVersion":"1.0",
            "configurationId":"",
            "bucket":{
                "name":"",
                "ownerIdentity":{
                    "principalId":""
                },
                "arn":"",
                "id":""
            },
            "object":{
                "key":"",
                "size":"0",
                "eTag":"",
                "versionId":"",
                "sequencer":"",
                "metadata":[],
                "tags":[]
            }
        },
        "eventId":"",
        "opaqueData":"",
    }
]}
```

* awsRegion：zonegroup
* eventTime：指示事件触发时间的时间戳
* eventName：`s3:ObjectCreated:`或`s3:ObjectRemoved:`
* userIdentity：不支持
* requestParameters：不支持
* responseElements：不支持
* s3.configurationId：为事件创建订阅的通知ID
* s3.bucket.name：存储桶名称
* s3.bucket.ownerIdentity.principalId：存储桶的所有者
* s3.bucket.arn：桶的ARN
* s3.bucket.id：存储桶的ID（S3通知API的扩展）
* s3.object.key：对象密钥
* s3.object.size：不支持
* s3.object.eTag：对象etag
* s3.object.version：存储桶版本化的情况下的对象版本
* s3.object.sequencer：单调增加的每个对象更改的标识符（十六进制格式）
* s3.object.metadata：不支持（S3通知API的扩展）
* s3.object.tags：不支持（S3通知API的扩展）
* s3.eventId：事件的唯一ID，可用于确认（S3通知API的扩展）
* s3.opaqueData：在主题配置中设置不透明数据，并将其添加到由ropic触发的所有通知中（S3通知API的扩展）

如果订阅不是通过非S3兼容的通知创建的，则事件将具有以下事件格式（JSON）：

```text
 {"events":[
    {
        "id":"",
        "event":"",
        "timestamp":"",
        "info":{
            "attrs":{
                "mtime":""
            },
            "bucket":{
                "bucket_id":"",
                "name":"",
                "tenant":""
            },
            "key":{
                "instance":"",
                "name":""
            }
        }
    }
]}
```

* id：事件的唯一ID，可用于确认
* 事件：一：`OBJECT_CREATE`，`OBJECT_DELETE`，`DELETE_MARKER_CREATE`
* 时间戳：指示事件发送时间的时间戳
* info.attrs.mtime：指示事件触发时间的时间戳
* info.bucket.bucket\_id：存储桶的ID
* info.bucket.name：存储桶的名称
* info.bucket.tenant：存储桶所属的承租人
* info.key.instance：存储桶版本化的情况下的对象版本
* info.key.name：对象密钥

**确认事件**

确认事件，以便可以将其从订阅历史记录中删除。

```text
POST /subscriptions/<sub-name>?ack&event-id=<event-id>
```

请求参数：

* event-id：要确认的事件的ID

