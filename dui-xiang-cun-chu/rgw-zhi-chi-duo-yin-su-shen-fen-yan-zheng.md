# RGW支持多因素身份验证

## RGW支持多因素身份验证

Mimic版本中的新功能。

S3多因素身份验证（MFA）功能允许用户在删除某些存储桶上的对象时要求使用一次性密码。这些存储桶需要配置为启用版本控制和MFA，这可以通过S3 API完成。

可以通过radosgw-admin将基于时间的一次性密码令牌分配给用户。每个令牌都有一个秘密种子和分配给它的序列ID。令牌已添加到用户，可以列出并删除，也可以重新同步。

### 多站点

在用户的元数据上设置了MFA ID时，实际的MFA一次性密码配置位于本地区域的osds中。因此，在多站点环境中，建议对不同的区域使用不同的令牌。

### 术语

- `TOTP`：基于时间的一次性密码

- ：代表TOTP令牌ID的字符串`token serial`

- ：用于计算TOTP的秘密种子`token seed`

- ：用于生成TOTP的时间分辨率`totp seconds`

- ：验证令牌时在当前令牌之前和之后检查的TOTP令牌的数量`totp window`

- ：TOTP令牌在特定时间的有效值`totp pin`

### 管理员命令

#### 创建一个新的MFA TOTP令牌

```text
# radosgw-admin mfa create --uid=<user-id> \
                           --totp-serial=<serial> \
                           --totp-seed=<seed> \
                           [ --totp-seed-type=<hex|base32> ] \
                           [ --totp-seconds=<num-seconds> ] \
                           [ --totp-window=<twindow> ]
```

#### 列出MFA TOTP令牌

```text
# radosgw-admin mfa list --uid=<user-id>
```

#### 显示MFA TOTP令牌

```text
# radosgw-admin mfa get --uid=<user-id> --totp-serial=<serial>
```

#### 删除MFA TOTP令牌

```text
# radosgw-admin mfa remove --uid=<user-id> --totp-serial=<serial>
```

#### 检查MFA TOTP令牌

测试TOTP令牌引脚，这是验证TOTP正常运行所必需的。

```text
# radosgw-admin mfa check --uid=<user-id> --totp-serial=<serial> \
                          --totp-pin=<pin>
```

#### 重新同步MFA TOTP令牌

为了重新同步TOTP令牌（在时间偏斜的情况下）。这需要给两个连续的引脚供电：前一个引脚和当前引脚。

```text
# radosgw-admin mfa resync --uid=<user-id> --totp-serial=<serial> \
                           --totp-pin=<prev-pin> --totp=pin=<current-pin>
```

