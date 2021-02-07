# kerboros 入坑指南

[TOC]

### 原理介绍

kerberos主要是用来做网络通信时候的身份认证，最主要的特点就是“复杂”。所以在入坑kerberos之前，最好先熟悉一下其原理。这里推荐一些别人写的文章内容来进行简单汇总：

1. 链接
    [kerberos认证原理](https://blog.csdn.net/wulantian/article/details/42418231)
    [用对话场景来解释kerbeors的设计过程](https://blog.csdn.net/dog250/article/details/5468741)

2. 简图

   

   ![img](https:////upload-images.jianshu.io/upload_images/6469473-e704be80985da1cc.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/939/format/webp)

   kerberos认证流程简图

### 几个概念的补充

1. principal
    认证的主体，简单来说就是"用户名"
2. realm
    realm有点像编程语言中的namespace。在编程语言中，变量名只有在某个"namespace"里才有意义。同样的，一个principal只有在某个realm下才有意义。
    所以realm可以看成是principal的一个"容器"或者"空间"。相对应的，principal的命名规则是"what_name_you_like@realm"。
    在kerberos, 大家都约定成俗用大写来命名realm, 比如"EXAMPLE.COM"
3. password
    某个用户的密码，对应于kerberos中的master_key。password可以存在一个keytab文件中。所以kerberos中需要使用密码的场景都可以用一个keytab作为输入。
4. credential
    credential是“证明某个人确定是他自己/某一种行为的确可以发生”的凭据。在不同的使用场景下， credential的具体含义也略有不同：
   - 对于某个principal个体而言，他的credential就是他的password。
   - 在kerberos认证的环节中，credential就意味着各种各样的ticket。

### 搭建一个KDC的环境

想要理解Kerberos，最好的方式是自己搭建一个KDC server。同时，这对于开发过程中进行测试也是非常有帮助的。

这里用ubuntu 16.04为例，简单搭一个kdc server

#### 安装下载



```csharp
apt-get install krb5-admin-server krb5-kdc krb5-user krb5-config
```

其中，这几个软件包的作用分别是：

- krb5-admin-server: kdc管理员程序，可以让使用者远程管理kdc数据库。
- krb5-kdc：kdc主程序
- krb5-user: kerberos的一些客户端命令，用来获取、查看、销毁ticket等等。

#### 配置

**/etc/krb5.conf**



```cpp
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 default_realm = EXAMPLE.COM
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 400
 renew_lifetime = 600
 forwardable = true
 udp_preference_limit = 1

[realms]
 EXAMPLE.COM = {
   kdc = 127.0.0.1:88
   admin_server = 127.0.0.1
 }
```

这个配置kdc，kerberos客户端，以及调用kerberos api时都会使用到。
 几个重要的配置项目包括：

- ticket_lifetime和renew_lifetime：指定了kdc授权ticket的过期时长，和允许更新现有ticket的时长。
- realms的section：指定了kdc和admin_server的路径

**/etc/krb5kdc/kdc.conf**



```jsx
[kdcdefaults]
    kdc_ports = 750,88

[realms]
    EXAMPLE.COM = {
        database_name = /etc/krb5kdc/example/principal
        admin_keytab = FILE:/etc/krb5kdc/example/kadm5.keytab
        acl_file = /etc/krb5kdc/example/kadm5.acl
        key_stash_file = /etc/krb5kdc/example/stash
        kdc_ports = 750,88
        max_life = 10h 0m 0s
        max_renewable_life = 7d 0h 0m 0s
        master_key_type = des3-hmac-sha1
        supported_enctypes = aes256-cts:normal arcfour-hmac:normal des3-hmac-sha1:normal des-cbc-crc:normal des:normal des:v4 des:norealm des:onlyrealm des:afs3
        default_principal_flags = +preauth
    }
```

这是kdc的专属配置，可以根据自己的需求修改下kdc数据库的存放目录。我都统一放/etc/krb5kdc/example目录下了。对于这个目录，自己需要提前建立好。

#### 创建数据库和principal

1. 使用kdb5_util创建数据库，从而可以存放principal相关的信息



```css
kdb5_util create -r EXAMPLE.COM -s
```

1. 使用kadmin.local来添加principal



```tsx
root@weijiesun-kubuntu:/etc/krb5kdc# kadmin.local
Authenticating as principal root/admin@EXAMPLE.COM with password.
kadmin.local:  add_principal test-server/myhost@EXAMPLE.COM
WARNING: no policy specified for test-server/myhost@EXAMPLE.COM; defaulting to no policy
Enter password for principal "test-server/myhost@EXAMPLE.COM":
Re-enter password for principal "test-server/myhost@EXAMPLE.COM":
Principal "test-server/myhost@EXAMPLE.COM" created.
kadmin.local:  add_principal test-client/myhost@EXAMPLE.COM
WARNING: no policy specified for test-client/myhost@EXAMPLE.COM; defaulting to no policy
Enter password for principal "test-client/myhost@EXAMPLE.COM":
Re-enter password for principal "test-client/myhost@EXAMPLE.COM":
Principal "test-client/myhost@EXAMPLE.COM" created.
kadmin.local:  ktadd -k /etc/krb5.keytab test-server/myhost@EXAMPLE.COM
Entry for principal test-server/myhost@EXAMPLE.COM with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/etc/krb5.keytab.
Entry for principal test-server/myhost@EXAMPLE.COM with kvno 2, encryption type arcfour-hmac added to keytab WRFILE:/etc/krb5.keytab.
Entry for principal test-server/myhost@EXAMPLE.COM with kvno 2, encryption type des3-cbc-sha1 added to keytab WRFILE:/etc/krb5.keytab.
Entry for principal test-server/myhost@EXAMPLE.COM with kvno 2, encryption type des-cbc-crc added to keytab WRFILE:/etc/krb5.keytab.
kadmin.local:  ktadd -k /etc/krb5.keytab test-client/myhost@EXAMPLE.COM
Entry for principal test-client/myhost@EXAMPLE.COM with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/etc/krb5.keytab.
Entry for principal test-client/myhost@EXAMPLE.COM with kvno 2, encryption type arcfour-hmac added to keytab WRFILE:/etc/krb5.keytab.
Entry for principal test-client/myhost@EXAMPLE.COM with kvno 2, encryption type des3-cbc-sha1 added to keytab WRFILE:/etc/krb5.keytab.
Entry for principal test-client/myhost@EXAMPLE.COM with kvno 2, encryption type des-cbc-crc added to keytab WRFILE:/etc/krb5.keytab.
kadmin.local:  q
root@weijiesun-kubuntu:/etc/krb5kdc#
```

这里，我们创建了两个新的用户：test-server/[myhost@EXAMPLE.COM](mailto:myhost@EXAMPLE.COM)和test-client/[myhost@EXAMPLE.COM](mailto:myhost@EXAMPLE.COM)。并且把他们的密钥放到/etc/krb5.keytab这一keytab文件下。

1. 启动kdc



```ruby
root@weijiesun-kubuntu:/etc/krb5kdc# service krb5-kdc start
 * Starting Kerberos KDC krb5kdc                                                                                                                                                       [ OK ]
root@weijiesun-kubuntu:/etc/krb5kdc# service krb5-admin-server start
 * Starting Kerberos administrative servers kadmind                                                                                                                                    [ OK ]
```

当然，对于不同发行版，这一步执行的命令可能会不太一样。

1. 用kinit验证KDC是否启动成功



```bash
kinit -k -t /etc/krb5.keytab test-client/myhost@EXAMPLE.COM
```

kinit对应的是向kdc获取TGT的步骤。它会向/etc/krb5.conf中指定的kdc server来发送请求。
 如果TGT请求成功，你就可以用klist看到它。



```ruby
weijiesun@weijiesun-kubuntu ~ $ klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: test-client/myhost@EXAMPLE.COM

Valid starting       Expires              Service principal
2018-08-17T13:50:25  2018-08-17T13:57:05  krbtgt/EXAMPLE.COM@EXAMPLE.COM
        renew until 2018-08-17T14:00:25
```

### 把kerberos认证流程嵌到项目中

在真正使用kerberos进行身份认证时，我们一般不直接使用kerberos的接口。而是会使用诸如GSSAPI或者SASL等更通用的一些标准接口。之所以这么做，是因为：

- kerberos的接口更琐碎
- SASL和GSSAPI都是IETF标准，他们对身份认证这一行为做了更宏观的抽象。从而可以灵活的接入不同的认证方案。

#### GSSAPI

GSSAPI的其流程基本和kerberos类似。在现实应用中，你基本可以假设GSSAPI就是kerberos本身。在我们最常使用的kerberos实现[MIT kerberos](http://web.mit.edu/kerberos/)中，GSSAPI的接口也已经是一个内置项了。

比较推荐MIT官方给出的一个gssapi的sample，是学习kerberos完整认证过程一个非常好的例子。里面也有详细的文档教大家如何使用：

- http://www.kerberos.org/software/samples/GSSExample.tgz

MIT kerberos项目内置的gss-sample也可以拿来学习kerberos的认证过程：

- https://github.com/krb5/krb5/tree/master/src/appl/gss-sample

#### SASL

[SASL](https://tools.ietf.org/html/rfc2222)是一个更加通用的身份认证接口，其接口在设计上可以兼容很多主流的认证方案。很多项目在做身份认证的时候，也是采用的SASL接口和流程。

SASL本身也有很多的实现，我在做小米的[Pegasus](https://github.com/XiaoMi/pegasus)项目时，采用的是[cyrus-sasl](https://github.com/cyrusimap/cyrus-sasl)。

想把sasl集成到项目中，并且使用gssapi作为其中的认证方案，其实不是特别的容易。这里给出一些要点：

- 从设计上而言，SASL上是一个client/server之间进行认证的接口。对于**向KDC获取TGT**这一行为，SASL没有预留接口。事实上，SASL在做第一步认证时，就假设用户已经获取到TGT了。所以这里需要我们自己实现获取TGT的代码，也就是kinit的过程。
- SASL的用户名和kerberos的principal命名风格不是完全相同的。在kerberos的标准中，principal的命名方式为name1/name2/name3/.../nameN@realm; 而在SASL中，用户名的命名方式为username/FQDN。因而，在使用SASL的时候，一定要把username和FQDN分开传给sasl的认证接口，从而构造成kerberos的principal。
- 认真学习cyrus-sasl的[认证示例](https://github.com/cyrusimap/cyrus-sasl/tree/master/sample)，并参考这篇[文档](https://www.cyrusimap.org/sasl/sasl/gssapi.html)