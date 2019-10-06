# 安装

首先安装 openldap、servers和clients三个软件包，openldap系统可能会自带。

```bash
$ yum -y install openldap openldap-servers openldap-clients

$ cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
$ chown -R ldap:ldap /var/lib/ldap/
$ chmod 0600 /var/lib/ldap/DB_CONFIG

$ systemctl enable slapd
$ systemctl start slapd
$ systemctl status slapd
```

