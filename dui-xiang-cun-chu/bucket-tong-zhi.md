# BUCKET通知

## [桶通知](https://docs.ceph.com/docs/nautilus/radosgw/notifications/#id1)

Nautilus版本中的新功能。

桶通知提供了一种在桶上发生某些事件时将信息发送到radosgw之外的机制。当前，通知可以发送到：HTTP，AMQP0.9.1和Kafka端点。

请注意，如果另外将事件存储在Ceph中，或者将事件推送到端点，则应使用[PubSub模块](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module)而不是存储桶通知机制。

用户可以创建不同的主题。主题实体由其用户和名称定义。用户只能管理自己的主题，并且只能将其与自己拥有的存储桶相关联。

为了发送有关特定存储桶事件的通知，需要创建一个通知实体。可以在事件类型的子集或所有事件类型上创建通知（默认）。通知还可以基于前缀/后缀和/或键的正则表达式匹配来过滤掉事件。以及，附加到对象或对象标签的元数据属性。任何特定主题可以有多个通知，并且同一主题可以用于多个通知。

定义了REST API，以为存储桶通知机制提供配置和控制接口。此API与pubsub同步模块中定义为S3兼容的API相似。

* S3存储桶通知兼容性

### 通知性能统计信息

pubsub同步模块和存储桶通知机制之间共享相同的计数器。

* `pubsub_event_triggered`：运行事件计数器，但至少要关联一个主题
* `pubsub_event_lost`：对与主题相关但未推送到任何端点的事件进行计数
* `pubsub_push_ok`：针对所有通知运行计数器，以将事件成功推送到其端点
* `pubsub_push_fail`：针对所有通知的运行计数器，未能将事件推送到其端点
* `pubsub_push_pending`：衡量推送到端点但尚未确认或尚未确认的事件的值

注意 

`pubsub_event_triggered`和`pubsub_event_lost`每个事件被递增，而： `pubsub_push_ok`，`pubsub_push_fail`被每推动作开始，每个通知。

### 桶通知REST 

#### 主题

**创建一个话题**

这将创建一个新主题。应该为该主题提供推送端点参数，稍后将在创建通知时使用这些参数。成功请求后，响应将包含主题ARN，以后可以将其用于在通知请求中引用此主题。要更新主题，请使用用于主题创建的相同命令，并使用现有主题的主题名称和不同的端点值。

TIP： 需要重新创建任何与该主题相关联的通知，以使主题更新生效

```text
POST
Action=CreateTopic
&Name=<topic-name>
&push-endpoint=<endpoint>
[&Attributes.entry.1.key=amqp-exchange&Attributes.entry.1.value=<exchange>]
[&Attributes.entry.2.key=amqp-ack-level&Attributes.entry.2.value=none|broker]
[&Attributes.entry.3.key=verify-ssl&Attributes.entry.3.value=true|false]
[&Attributes.entry.4.key=kafka-ack-level&Attributes.entry.4.value=none|broker]
[&Attributes.entry.5.key=use-ssl&Attributes.entry.5.value=true|false]
[&Attributes.entry.6.key=ca-location&Attributes.entry.6.value=<file path>]
[&Attributes.entry.7.key=OpaqueData&Attributes.entry.7.value=<opaque data>]
```

请求参数：

* push-endpoint：向其发送推送通知的端点的URI
* OpaqueData：在主题配置中设置不透明数据，并将其添加到由ropic触发的所有通知中
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

注意

* 特定参数的键/值不必位于同一行或任何特定顺序，而必须使用相同的索引
* 属性索引不需要是顺序的或从任何特定值开始
* [AWS Create Topic](https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html)具有有关终端节点属性格式的详细说明。但是，在本例中，使用了不同的键和值

响应将具有以下格式：

```text
<CreateTopicResponse xmlns="https://sns.amazonaws.com/doc/2010-03-31/">
    <CreateTopicResult>
        <TopicArn></TopicArn>
    </CreateTopicResult>
    <ResponseMetadata>
        <RequestId></RequestId>
    </ResponseMetadata>
</CreateTopicResponse>
```

响应中的主题ARN将具有以下格式：

```text
arn:aws:sns:<zone-group>:<tenant>:<topic>
```

**获取主题信息**

返回有关特定主题的信息。这包括推送端点信息（如果提供）。

```text
POST
Action=GetTopic&TopicArn=<topic-arn>
```

响应将具有以下格式：

```text
<GetTopicResponse>
    <GetTopicRersult>
        <Topic>
            <User></User>
            <Name></Name>
            <EndPoint>
                <EndpointAddress></EndpointAddress>
                <EndpointArgs></EndpointArgs>
                <EndpointTopic></EndpointTopic>
            </EndPoint>
            <TopicArn></TopicArn>
            <OpaqueData></OpaqueData>
        </Topic>
    </GetTopicResult>
    <ResponseMetadata>
        <RequestId></RequestId>
    </ResponseMetadata>
</GetTopicResponse>
```

* 用户：创建主题的用户名
* 名称：主题名称
* EndPoinjtAddress：推送端点URL
* 如果端点URL包含用户/密码信息，则必须通过HTTPS发出请求。如果不是，则主题获取请求将被拒绝
* EndPointArgs：推入终点参数
* EndpointTopic：应该发送到端点的主题名称（与上面的主题名称不同）
* TopicArn：主题ARN

**删除主题**

```text
POST
Action=DeleteTopic&TopicArn=<topic-arn>
```

删除指定的主题。请注意，删除已删除的主题应该是无操作且不会失败。

响应将具有以下格式：

```text
<DeleteTopicResponse xmlns="https://sns.amazonaws.com/doc/2010-03-31/">
    <ResponseMetadata>
        <RequestId></RequestId>
    </ResponseMetadata>
</DeleteTopicResponse>
```

**列出主题**

列出用户定义的所有主题。

```text
POST
Action=ListTopics
```

响应将具有以下格式：

```text
<ListTopicdResponse xmlns="https://sns.amazonaws.com/doc/2010-03-31/">
    <ListTopicsRersult>
        <Topics>
            <member>
                <User></User>
                <Name></Name>
                <EndPoint>
                    <EndpointAddress></EndpointAddress>
                    <EndpointArgs></EndpointArgs>
                    <EndpointTopic></EndpointTopic>
                </EndPoint>
                <TopicArn></TopicArn>
                <OpaqueData></OpaqueData>
            </member>
        </Topics>
    </ListTopicsResult>
    <ResponseMetadata>
        <RequestId></RequestId>
    </ResponseMetadata>
</ListTopicsResponse>
```

* 如果端点URL包含用户/密码信息，则在任何主题中，都必须通过HTTPS发出请求。否则，主题列表请求将被拒绝

#### 通知

详细说明：BUCKET OPERATIONS。

注意

* “中止分段上传”请求未发出通知
* “删除多个对象”请求不会发出通知
* “启动分段上传”和“ POST对象”请求都将发出`s3:ObjectCreated:Post`通知

#### 事件

这些事件采用JSON格式（无论实际端点如何），并且与使用pubsub sync模块推送或拉出的S3兼容事件共享相同的结构。

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
                "id:""
            },
            "object":{
                "key":"",
                "size":"",
                "eTag":"",
                "versionId":"",
                "sequencer": "",
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
* eventName：有关受支持事件的列表，请参阅：[S3通知兼容性](https://docs.ceph.com/docs/nautilus/radosgw/s3-notification-compatibility)
* userIdentity.principalId：触发更改的用户
* requestParameters.sourceIPAddress：不支持
* responseElements.x-amz-request-id：原始更改的请求ID
* responseElements.x\_amz\_id\_2：对其进行更改的RGW
* s3.configurationId：创建事件的通知ID
* s3.bucket.name：存储桶名称
* s3.bucket.ownerIdentity.principalId：存储桶的所有者
* s3.bucket.arn：桶的ARN
* s3.bucket.id：存储桶的ID（S3通知API的扩展）
* s3.object.key：对象密钥
* s3.object.size：对象大小
* s3.object.eTag：对象etag
* s3.object.version：存储桶版本化的情况下的对象版本
* s3.object.sequencer：单调增加的每个对象更改的标识符（十六进制格式）
* s3.object.metadata：在对象上设置的任何元数据，发送方式为：`x-amz-meta-`（S3通知API的扩展）
* s3.object.tags：objcet上设置的任何标签（S3通知API的扩展）
* s3.eventId：事件的唯一ID，可用于确认（S3通知API的扩展）
* s3.opaqueData：在主题配置中设置不透明数据，并将其添加到由ropic触发的所有通知中（S3通知API的扩展）

