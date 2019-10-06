# 卸载

停止服务

```base
$ systemctl stop slapd
```

删除软件包

```base
$ yum -y remove openldap-servers openldap-clients
```

删除配置文件

```base
$ rm -rf /etc/openldap/slapd.d
```

删除数据库文件

```base
$ rm -rf /var/lib/ldap
```


