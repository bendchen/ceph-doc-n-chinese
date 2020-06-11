# LDAP认证

## LDAP认证

珠宝的新版本。

您可以将Ceph Object Gateway身份验证委派给LDAP服务器。

### 它是如何工作

Ceph对象网关从令牌中提取用户LDAP凭据。使用用户名构造搜索过滤器。Ceph对象网关使用配置的服务帐户在目录中搜索匹配的条目。如果找到条目，则Ceph对象网关会尝试使用令牌中的密码绑定到找到的专有名称。如果凭据有效，则绑定将成功，并且Ceph对象网关将授予访问权限。

您可以通过将搜索基础设置为特定组织单位或指定自定义搜索过滤器（例如，要求特定的组成员身份，自定义对象类或属性）来限制允许的用户。

### 要求

* **LDAP或Active Directory：** Ceph对象网关可访问的运行中的LDAP实例
* **服务帐户：**具有搜索权限的Ceph对象网关将使用的LDAP凭据
* **用户帐户：** LDAP目录中的至少一个用户帐户
* **请勿使LDAP和本地用户重叠：**本地用户和使用LDAP进行身份验证的用户不应使用相同的用户名。Ceph对象网关无法区分它们，并将它们视为同一用户。

### 完整性检查

使用该`ldapsearch`实用工具来验证服务帐户或LDAP连接：

```text
# ldapsearch -x -D "uid=ceph,ou=system,dc=example,dc=com" -W \
-H ldaps://example.com -b "ou=users,dc=example,dc=com" 'uid=*' dn
```

注意 

确保使用与Ceph配置文件中相同的LDAP参数来消除可能的问题。

### 配置CEPH对象网关以使用LDAP认证

Ceph配置文件中的以下参数与LDAP身份验证相关：

* `rgw_ldap_uri`：指定要使用的LDAP服务器。确保使用该 `ldaps://<fqdn>:<port>`参数不通过网络传输明文凭证。
* `rgw_ldap_binddn`：Ceph对象网关使用的服务帐户的专有名称（DN）
* `rgw_ldap_secret`：服务帐户的密码
* `rgw_ldap_searchdn`：指定目录信息树中用于搜索用户的基础。这可能是您用户的组织单位，也可能是某些特定的组织单位（OU）。
* `rgw_ldap_dnattr`：在构造的搜索过滤器中用于匹配用户名的属性。根据您的目录信息树（DIT），可能为`uid`或`cn`。生成的过滤器字符串将为`cn=some_username`。
* `rgw_ldap_searchfilter`：如果未指定，则Ceph Object Gateway会使用该`rgw_ldap_dnattr` 设置自动构建搜索过滤器。使用此参数以非常灵活的方式缩小允许的用户列表。有关详细信息，请参阅“ _使用自定义搜索过滤器限制用户访问”部分_

### 使用自定义搜索过滤器限制用户访问权限

有两种使用`rgw_search_filter`参数的方法：

#### 指定局部过滤器以进一步限制构造的搜索过滤器

部分过滤器的示例：

```text
"objectclass=inetorgperson"
```

Ceph对象网关将照常使用来自令牌的用户名和的值生成搜索过滤器`rgw_ldap_dnattr`。然后将构造的过滤器与`rgw_search_filter` 属性中的部分过滤器组合。根据用户名和设置，最终的搜索过滤器可能变为：

```text
"(&(uid=hari)(objectclass=inetorgperson))"
```

因此，`hari`只有在LDAP目录中找到用户，对象类为`inetorgperson`且指定了有效密码的用户才能被授予访问权限。

#### 指定一个完整的过滤器

完整的过滤器必须包含一个`@USERNAME@`令牌，该令牌将在身份验证尝试期间替换为用户名。`rgw_ldap_dnattr` 在这种情况下，不再使用该参数。例如，要将有效用户限制为特定组，请使用以下过滤器：

```text
"(&(uid=@USERNAME@)(memberOf=cn=ceph-users,ou=groups,dc=mycompany,dc=com))"
```

注意 

`memberOf`在LDAP搜索中使用属性需要您特定LDAP服务器实现的服务器端支持。

### 生成用于LDAP身份验证的访问令牌

该`radosgw-token`实用程序根据LDAP用户名和密码生成访问令牌。它将输出一个base-64编码的字符串，它是访问令牌。

```text
# export RGW_ACCESS_KEY_ID="<username>"
# export RGW_SECRET_ACCESS_KEY="<password>"
# radosgw-token --encode --ttype=ldap
```

注意 

对于Active Directroy，请使用`--ttype=ad`参数。

重要 

访问令牌是base-64编码的JSON结构，并包含LDAP凭据作为明文。

### 测试访问权限

使用您喜欢的S3客户端并将令牌指定为访问密钥。

