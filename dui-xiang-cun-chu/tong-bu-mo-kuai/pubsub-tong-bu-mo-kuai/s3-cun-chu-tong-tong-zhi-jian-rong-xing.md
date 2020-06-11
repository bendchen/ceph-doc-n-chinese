# S3存储桶通知兼容性

## S3存储桶通知兼容性

Ceph的[存储桶通知](https://docs.ceph.com/docs/nautilus/radosgw/notifications)和[PubSub模块](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module) API遵循[AWS S3存储桶通知API](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html)。但是，存在一些差异，如下所列。

注意 

兼容性因使用上述哪种机制而异

### 支持的目标

AWS支持：**SNS**，**SQS**和**Lambda**作为可能的目的地（AWS内部目的地）。当前，我们支持：**HTTP / S**和**AMQP**。并且还支持拉出和确认存储在Ceph中的事件（作为内部目的地）。

我们正在使用**SNS** ARN来表示**HTTP / S**和**AMQP**目的地。

### 通知配置

不支持以下标记（及其内部的标记）：

| 标签 | 重做 |
| :--- | :--- |
| `<QueueConfiguration>` | 不需要，我们将所有目的地都视为SNS |
| `<CloudFunctionConfiguration>` | 不需要，我们将所有目的地都视为SNS |

### REST API扩展

Ceph的存储桶通知API具有以下扩展：

* 使用`DELETE`动词删除特定通知或存储桶中的所有通知

> * 在S3中，删除存储桶或在存储桶上设置空通知时，所有通知都将被删除

* 获取有关特定通知的信息（当存储桶中存在多个通知时）
  * 在S3中，只能提取存储桶上的所有通知
* 除了基于对象键的前缀/后缀进行过滤之外，我们还支持：
  * 基于正则表达式匹配的过滤
  * 根据附加到对象的元数据属性进行过滤
  * 根据对象标签进行过滤
* 过滤重叠是允许的，因此同一事件可以作为不同的通知发送

### 不支持的字段在事件记录

发送给存储桶通知的记录遵循“ [事件消息结构”中](https://docs.aws.amazon.com/AmazonS3/latest/dev/notification-content-structure.html)描述的格式。但是，在不同的部署选项（Notification / PubSub）下，以下字段可能为空发送：

| 领域 | 通知 | PubSub | 描述 |
| :--- | :--- | :--- | :--- |
| `userIdentity.principalId` | 支持的 | 不支持 | 触发事件的用户的身份 |
| `requestParameters.sourceIPAddress` | 不支持 | 触发事件的客户端的IP地址 |  |
| `requestParameters.x-amz-request-id` | 支持的 | 不支持 | 触发事件的请求ID |
| `requestParameters.x-amz-id-2` | 支持的 | 不支持 | 触发事件的RGW的IP地址 |
| `s3.object.size` | 支持的 | 不支持 | 对象的大小 |

### 事件类型

| 事件 | 通知 | PubSub |
| :--- | :--- | :--- |
| `s3:ObjectCreated:*` | 支持的 |  |
| `s3:ObjectCreated:Put` | 支持的 | 在`s3:ObjectCreated:*`级别上受支持 |
| `s3:ObjectCreated:Post` | 支持的 | 不支持 |
| `s3:ObjectCreated:Copy` | 支持的 | 在`s3:ObjectCreated:*`级别上受支持 |
| `s3:ObjectCreated:CompleteMultipartUpload` | 支持的 | 在`s3:ObjectCreated:*`级别上受支持 |
| `s3:ObjectRemoved:*` | 支持的 | 仅支持以下特定事件 |
| `s3:ObjectRemoved:Delete` | 支持的 |  |
| `s3:ObjectRemoved:DeleteMarkerCreated` | 支持的 |  |
| `s3:ObjectRestore:Post` | 不适用于Ceph |  |
| `s3:ObjectRestore:Complete` | 不适用于Ceph |  |
| `s3:ReducedRedundancyLostObject` | 不适用于Ceph |  |

### 主题配置

对于存储桶通知，主题管理API将从[AWS Simple Notification Service API](https://docs.aws.amazon.com/sns/latest/api/API_Operations.html)派生。请注意，大多数API不适用于Ceph，并且仅实现以下操作：

> * `CreateTopic`
> * `DeleteTopic`
> * `ListTopics`

我们还对主题配置进行了以下扩展：

> * 在这种情况下，`GetTopic`我们允许获取特定主题，而不是所有用户主题
> * 在 `CreateTopic`
>
> > * 我们允许设置端点属性
> > * 我们允许设置不透明数据，将在通知中发送到端点

