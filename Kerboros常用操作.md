# Kerboros 常用操作

[TOC]

### 文章目录

- [登录Kerberos: kadmin.local](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#Kerberos_kadminlocal_1)
- [查看已存在凭据：list_principals, listprincs, get_principals, getprincs](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#list_principals_listprincs_get_principals_getprincs_40)
- [添加凭据：add_principal, addprinc, ank](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#add_principal_addprinc_ank_54)
- [修改凭据密码：change_password, cpw](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#change_password_cpw_62)
- [删除凭据：delete_principal, delprinc](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#delete_principal_delprinc_69)
- [认证用户：kinit](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#kinit_76)
- [查看当前认证用户：klist](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#klist_81)
- [删除当前认证的缓存](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#_91)
- [切换Hadoop组内用户票据](https://blog.csdn.net/Happy_Sunshine_Boy/article/details/102801386#Hadoop_97)



# 登录Kerberos: kadmin.local

- 使用kadmin.local命令登录

```linux
[root@manager ~]# kadmin.local 
Authenticating as principal root/admin@BIGDATA with password.
kadmin.local:  ?	# 查看命令列表i
Available kadmin.local requests:

add_principal, addprinc, ank
                         Add principal
delete_principal, delprinc
                         Delete principal
modify_principal, modprinc
                         Modify principal
rename_principal, renprinc
                         Rename principal
change_password, cpw     Change password
get_principal, getprinc  Get principal
list_principals, listprincs, get_principals, getprincs
                         List principals
add_policy, addpol       Add policy
modify_policy, modpol    Modify policy
delete_policy, delpol    Delete policy
get_policy, getpol       Get policy
list_policies, listpols, get_policies, getpols
                         List policies
get_privs, getprivs      Get privileges
ktadd, xst               Add entry(s) to a keytab
ktremove, ktrem          Remove entry(s) from a keytab
lock                     Lock database exclusively (use with extreme caution!)
unlock                   Release exclusive database lock
purgekeys                Purge previously retained old keys from a principal
get_strings, getstrs     Show string attributes on a principal
set_string, setstr       Set a string attribute on a principal
del_string, delstr       Delete a string attribute on a principal
list_requests, lr, ?     List available requests.
quit, exit, q            Exit program.

1234567891011121314151617181920212223242526272829303132333435
```

# 查看已存在凭据：list_principals, listprincs, get_principals, getprincs

```linux
kadmin.local:  listprincs
HTTP/manager.bigdata@BIGDATA
HTTP/master.bigdata@BIGDATA
HTTP/worker.bigdata@BIGDATA
K/M@BIGDATA
activity_analyzer/manager.bigdata@BIGDATA
activity_explorer/manager.bigdata@BIGDATA
admin/admin@BIGDATA
ambari-qa-gaia@BIGDATA
ambari-server-gaia@BIGDATA
……
1234567891011
```

# 添加凭据：add_principal, addprinc, ank

```linux
kadmin.local:  ank test@BIGDATA
WARNING: no policy specified for test@BIGDATA; defaulting to no policy
Enter password for principal "test@BIGDATA": 
Re-enter password for principal "test@BIGDATA": 
Principal "test@BIGDATA" created.
12345
```

# 修改凭据密码：change_password, cpw

```linux
kadmin.local:  cpw test@BIGDATA
Enter password for principal "test@BIGDATA": 
Re-enter password for principal "test@BIGDATA": 
Password for "test@BIGDATA" changed.
1234
```

# 删除凭据：delete_principal, delprinc

```linux
kadmin.local:  delprinc test@BIGDATA
Are you sure you want to delete the principal "test@BIGDATA"? (yes/no): yes
Principal "test@BIGDATA" deleted.
Make sure that you have removed this principal from all ACLs before reusing.
1234
```

# 认证用户：kinit

```linux
[root@manager ~]# kinit admin/admin@BIGDATA
Password for admin/admin@BIGDATA:
12
```

# 查看当前认证用户：klist

```linux
[root@manager ~]# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: admin/admin@BIGDATA

Valid starting       Expires              Service principal
10/30/2019 00:21:12  10/31/2019 00:21:12  krbtgt/BIGDATA@BIGDATA
	renew until 11/06/2019 00:21:12
1234567
```

# 删除当前认证的缓存

```linux
[root@manager ~]# kdestroy
[root@manager ~]# klist
klist: No credentials cache found (filename: /tmp/krb5cc_0)
123
```

# 切换Hadoop组内用户票据

- 根据keytab文件查询用户

```linux
klist -kt /etc/security/keytabs/hdfs.headless.keytab
1
```

- 切换票据

```linux
kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdv4@BIGDATA
```