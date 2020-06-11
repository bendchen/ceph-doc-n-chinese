# 角色

## 角色

角色类似于用户，并具有附加的权限策略，该权限策略确定角色可以做什么或不能做什么。角色可以由需要它的任何身份承担。如果用户担任角色，则将一组动态创建的临时凭据返回给用户。角色可用于将访问权限委派给没有权限访问某些s3资源的用户，应用程序和服务。

以下radosgw-admin命令可用于创建/删除/更新角色以及与角色相关联的权限。

### 创建一个角色

要创建角色，请执行以下操作：

```text
radosgw-admin role create --role-name={role-name} [--path=="{path to the role}"] [--assume-role-policy-doc={trust-policy-document}]
```

#### 请求参数

`role-name`描述

角色名称。类型

串

`path`描述

角色的路径。默认值为斜杠（/）。类型

串

`assume-role-policy-doc`描述

信任关系策略文档，用于授予实体承担该角色的权限。类型

串

例如：

```text
radosgw-admin role create --role-name=S3Access1 --path=/application_abc/component_xyz/ --assume-role-policy-doc=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Principal\":\{\"AWS\":\[\"arn:aws:iam:::user/TESTER\"\]\},\"Action\":\[\"sts:AssumeRole\"\]\}\]\}
```

```text
{
  "id": "ca43045c-082c-491a-8af1-2eebca13deec",
  "name": "S3Access1",
  "path": "/application_abc/component_xyz/",
  "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
  "create_date": "2018-10-17T10:18:29.116Z",
  "max_session_duration": 3600,
  "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
}
```

### 删除角色

要删除角色，请执行以下操作：

```text
radosgw-admin role rm --role-name={role-name}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

例如：

```text
radosgw-admin role rm --role-name=S3Access1
```

注意：仅当角色未附加任何权限策略时，才能删除该角色。

### 获得角色

要获取有关角色的信息，请执行以下操作：

```text
radosgw-admin role get --role-name={role-name}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

例如：

```text
radosgw-admin role get --role-name=S3Access1
```

```text
{
  "id": "ca43045c-082c-491a-8af1-2eebca13deec",
  "name": "S3Access1",
  "path": "/application_abc/component_xyz/",
  "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
  "create_date": "2018-10-17T10:18:29.116Z",
  "max_session_duration": 3600,
  "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
}
```

### 列出角色

要列出具有指定路径前缀的角色，请执行以下操作：

```text
radosgw-admin role list [--path-prefix ={path prefix}]
```

#### 请求参数

`path-prefix`描述

过滤角色的路径前缀。如果未指定，则列出所有角色。类型

串

例如：

```text
radosgw-admin role list --path-prefix="/application"
```

```text
[
  {
      "id": "3e1c0ff7-8f2b-456c-8fdf-20f428ba6a7f",
      "name": "S3Access1",
      "path": "/application_abc/component_xyz/",
      "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
      "create_date": "2018-10-17T10:32:01.881Z",
      "max_session_duration": 3600,
      "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
  }
]
```

### 更新角色的假定角色策略文档

要修改角色的承担角色策略文档，请执行以下操作：

```text
radosgw-admin role modify --role-name={role-name} --assume-role-policy-doc={trust-policy-document}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

`assume-role-policy-doc`描述

信任关系策略文档，用于授予实体承担该角色的权限。类型

串

例如：

```text
radosgw-admin role modify --role-name=S3Access1 --assume-role-policy-doc=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Principal\":\{\"AWS\":\[\"arn:aws:iam:::user/TESTER2\"\]\},\"Action\":\[\"sts:AssumeRole\"\]\}\]\}
```

```text
{
  "id": "ca43045c-082c-491a-8af1-2eebca13deec",
  "name": "S3Access1",
  "path": "/application_abc/component_xyz/",
  "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
  "create_date": "2018-10-17T10:18:29.116Z",
  "max_session_duration": 3600,
  "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER2\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
}
```

在上面的示例中，我们在其承担角色策略文档中将Principal从TESTER修改为TESTER2。

### 添加/更新附加到角色的策略

要添加或更新附加到角色的内联策略，请执行以下操作：

```text
radosgw-admin role policy put --role-name={role-name} --policy-name={policy-name} --policy-doc={permission-policy-doc}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

`policy-name`描述

策略名称。类型

串

`policy-doc`描述

权限策略文档。类型

串

例如：

```text
radosgw-admin role-policy put --role-name=S3Access1 --policy-name=Policy1 --policy-doc=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Action\":\[\"s3:*\"\],\"Resource\":\"arn:aws:s3:::example_bucket\"\}\]\}
```

在上面的示例中，我们将策略“ Policy1”附加到角色“ S3Access1”，该策略允许对“ example\_bucket”执行所有s3操作。

### 列出附加到角色的权限策略名称

要列出附加到角色的权限策略的名称，请执行以下操作：

```text
radosgw-admin role policy get --role-name={role-name}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

例如：

```text
radosgw-admin role-policy list --role-name=S3Access1
```

```text
[
  "Policy1"
]
```

### 获取附加到角色的权限策略

要获得附加到角色的特定权限策略，请执行以下操作：

```text
radosgw-admin role policy get --role-name={role-name} --policy-name={policy-name}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

`policy-name`描述

策略名称。类型

串

例如：

```text
radosgw-admin role-policy get --role-name=S3Access1 --policy-name=Policy1
```

```text
{
  "Permission policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"s3:*\"],\"Resource\":\"arn:aws:s3:::example_bucket\"}]}"
}
```

### 删除附加到角色的策略

要删除附加到角色的权限策略，请执行以下操作：

```text
radosgw-admin role policy rm --role-name={role-name} --policy-name={policy-name}
```

#### 请求参数

`role-name`描述

角色名称。类型

串

`policy-name`描述

策略名称。类型

串

例如：

```text
radosgw-admin role-policy get --role-name=S3Access1 --policy-name=Policy1
```

**REST API操纵角色**

除了上面的radosgw-admin命令之外，还可以使用以下REST API来操纵角色。有关请求参数及其说明，请参阅上面的部分。

为了调用REST管理API，需要创建一个具有管理员权限的用户。

```text
radosgw-admin --uid TESTER --display-name "TestUser" --access_key TESTER --secret test123 user create
radosgw-admin caps add --uid="TESTER" --caps="roles=*"
```

### 创建一个角色

例：：

POST“ &lt;主机名&gt;？Action = CreateRole＆RoleName = S3Access＆Path = / application\_abc / component\_xyz /＆AssumeRolePolicyDocument = {“版本”：“ 2012-10-17”，“声明”：\[{“效果”：“允许”，“委托人”： {“ AWS”：\[“ arn：aws：iam ::: user / TESTER”\]}，“动作”：\[“ sts：AssumeRole”\]}}}}”

```text
<role>
  <id>8f41f4e0-7094-4dc0-ac20-074a881ccbc5</id>
  <name>S3Access</name>
  <path>/application_abc/component_xyz/</path>
  <arn>arn:aws:iam:::role/application_abc/component_xyz/S3Access</arn>
  <create_date>2018-10-23T07:43:42.811Z</create_date>
  <max_session_duration>3600</max_session_duration>
  <assume_role_policy_document>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["arn:aws:iam:::user/TESTER"]},"Action":["sts:AssumeRole"]}]}</assume_role_policy_document>
</role>
```

### 删除角色

例：：

POST“ &lt;主机名&gt;？操作= DeleteRole＆RoleName = S3Access”

注意：仅当角色未附加任何权限策略时，才能删除该角色。

### 获得角色

例：：

POST“ &lt;主机名&gt;？Action = GetRole＆RoleName = S3Access”

```text
<role>
  <id>8f41f4e0-7094-4dc0-ac20-074a881ccbc5</id>
  <name>S3Access</name>
  <path>/application_abc/component_xyz/</path>
  <arn>arn:aws:iam:::role/application_abc/component_xyz/S3Access</arn>
  <create_date>2018-10-23T07:43:42.811Z</create_date>
  <max_session_duration>3600</max_session_duration>
  <assume_role_policy_document>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["arn:aws:iam:::user/TESTER"]},"Action":["sts:AssumeRole"]}]}</assume_role_policy_document>
</role>
```

### 列出角色

例：：

POST“ &lt;主机名&gt;？Action = ListRoles＆RoleName = S3Access＆PathPrefix = /应用程序”

```text
<role>
  <id>8f41f4e0-7094-4dc0-ac20-074a881ccbc5</id>
  <name>S3Access</name>
  <path>/application_abc/component_xyz/</path>
  <arn>arn:aws:iam:::role/application_abc/component_xyz/S3Access</arn>
  <create_date>2018-10-23T07:43:42.811Z</create_date>
  <max_session_duration>3600</max_session_duration>
  <assume_role_policy_document>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["arn:aws:iam:::user/TESTER"]},"Action":["sts:AssumeRole"]}]}</assume_role_policy_document>
</role>
```

### 更新假设角色策略文档

例：：

POST“ &lt;主机名&gt;？Action = UpdateAssumeRolePolicy＆RoleName = S3Access＆PolicyDocument = {“版本”：“ 2012-10-17”，“声明”：\[{“ Effect”：“ Allow”，“ Principal”：{“ AWS”：\[“ arn：aws：iam ::: user / TESTER2“\]}，”动作“：\[” sts：AssumeRole“\]}\]}}”

### 添加/更新附加到角色的策略

例：：

POST“ &lt;主机名&gt;？Action = PutRolePolicy＆RoleName = S3Access＆PolicyName = Policy1＆PolicyDocument = {“版本”：“ 2012-10-17”，“声明”：\[{“效果”：“允许”，“操作”：\[\[s3：CreateBucket ”\]，“资源”：“ arn：aws：s3 ::: example\_bucket”}\]}”

### 列出附加到角色的权限策略名称

例：：

POST“ &lt;主机名&gt;？操作= ListRolePolicies＆RoleName = S3Access”

```text
<PolicyNames>
  <member>Policy1</member>
</PolicyNames>
```

### 获取附加到角色的权限策略

例：：

POST“ &lt;主机名&gt;？操作= GetRolePolicy＆RoleName = S3Access＆PolicyName = Policy1”

```text
<GetRolePolicyResult>
  <PolicyName>Policy1</PolicyName>
  <RoleName>S3Access</RoleName>
  <Permission_policy>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["s3:CreateBucket"],"Resource":"arn:aws:s3:::example_bucket"}]}</Permission_policy>
</GetRolePolicyResult>
```

### 删除附加到角色的策略

例：：

POST“ &lt;主机名&gt;？操作= DeleteRolePolicy＆RoleName = S3Access＆PolicyName = Policy1”

