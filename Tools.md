# 管理工具

## WEB管理工具

### phpLPADadmin 🌞推荐

老牌LPAD管理工具，界面像是回到了90年代，不过功能齐全免去手写代码的麻烦。

* [Github地址](https://github.com/leenooks/phpLDAPadmin)

下载后解压缩到 `/ioo/site/ldap.domain.com`

```base
$ cp /ioo/site/ldap.domain.com/config/config.php.example /ioo/site/ldap.domain.com/config/config.php
```

```php
$servers = new Datastore();
$servers->newServer('ldap_pla');
$servers->setValue('server','name','My LDAP Server');
$servers->setValue('login','anon_bind',false);
$servers->setValue('login','allowed_dns',array('cn=root,dc=domain,dc=com'));
$servers->setValue('login','attr','dn');
$servers->setValue('server','host','192.168.99.200');
$servers->setValue('server','port',389);
```

phpLPADadmin需要HTTP-Server支持，下面是Nginx配置：

```nginx
server {
    listen       80;
    server_name  ldap.domain.com;
    charset utf8;

    index  index.php;

    location / {
        root   /mnt/hgfs/ioo/tools/phpLDAPadmin-1.2.4/htdocs;
    }
    location ~ \.php$ {
        root           htdocs;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /mnt/hgfs/ioo/tools/phpLDAPadmin-1.2.4/htdocs/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

访问 http://ldap.domain.com 使用 [Setup](Setup.md) 中设置的管理员和密码登录

### LDAP Account Manager

相比phpLPADadmin界面更加简洁，集成phpLPADadmin的部分功能。但灵活度不够并且强制建立用户组。

* [官方网站](https://www.ldap-account-manager.org/lamcms)
* [下载地址](https://www.ldap-account-manager.org/lamcms/releases)

```base
$ bunzip2 ldap-account-manager-6.7.tar.bz2
$ tar -xf ldap-account-manager-6.7.tar
$ cd ldap-account-manager-6.7
$ ./configure --with-httpd-user=ioo --with-httpd-group=ioo --with-web-root=/ioo/site/lam.domain.com/htdocs
$ make install
$ chmod -R 0777 /ioo/site/lam.domain.com/htdocs
$ cp /ioo/site/lam.domain.com/htdocs/config/config.cfg.sample /ioo/site/lam.domain.com/htdocs/config/config.cfg
```
⚠️注意 `chmod 0777` 只是为了快速部署，要根据`LAM文档`设置特定文件夹权限，保证安全性。

LAM需要HTTP-Server支持，下面是Nginx配置：

```nginx
server {
    listen       80;
    server_name  lam.domain.com;
    charset utf8;
    location / {
        root   /ioo/site/lam.domain.com/htdocs;
        index  index.html index.htm index.php;
    }
    location ~ \.php$ {
        root           htdocs;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /ioo/site/lam.domain.com/htdocs/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

访问 http://lam.domain.com 完成`LAM Settings`，添加Server开始管理。

## 客户端管理工具

### Ldap Admin Tool (Professional Editor) 🌞推荐

支持Mac、Windows、Linux，功能强大，力荐！

* [官方网站](https://www.ldapsoft.com)
* [下载地址](https://www.ldapsoft.com/ldapbrowserfree.html)

以下是`LDAPSoft`公司，各产品线的注册码：

```json
// LDAP Admin Tool(Professional Edition)
{
  "email": "myluamas@gmail.com",
  "registrationKey": "VW1wR2RGbFZUWGhqZWtaMFpWZDRNVmxYTVdoak1FSnVZbGRHY0dKRE5XcGlNakZFVFZSck5VOVVRVEZNZWtFeVRIcE5kMDFVWnowPQ==",
  "activationCode": "STARTTIMEGMT1525589975000-4679189975000",
  "type": "LDAP Admin Tool(Professional Edition)"
}

// LDAP Admin Tool(Standard Edition)
{
  "email": "myluamas@gmail.com",
  "registrationKey": "RjFtYUMxc215bHVhbWFzQGdtYWlsLmNvbUMxOTkwNS8wNi8zMDE4",
  "activationCode": "STARTTIMEGMT1525590013000-4679190013000",
  "type": "LDAP Admin Tool(Standard Edition)"
}

// AD Admin & Reporting Tool
{
  "email": "myluamas@gmail.com",
  "registrationKey": "VVZWU1UxSnFSblJaVlUxNFkzcEdkR1ZYZURGWlZ6Rm9ZekJDYm1KWFJuQmlRelZxWWpJeFFsSkdTa1JOVkdzMVQxUkJNVXg2UVRKTWVrMTNUVlJuUFE9PQ==",
  "activationCode": "STARTTIMEGMT1525590027000-4679190027000",
  "type": "AD Admin \u0026 Reporting Tool"
}

// LDAP Plus AD HelpDesk Professional Tool
{
  "email": "myluamas@gmail.com",
  "registrationKey": "VW1wR2RFMTZVVEZXVjJ4dVlVZFNiV0ZIUm10YWJWSjBaVmQ0TVZsWE1XaGpNRUp1WWxkR2NHSkROV3BpTWpBeVRqQm9TV1JZYkd0VFIxWnpZMFJCTVV4NlFUSk1lazEzVFZSblBRPT0=",
  "activationCode": "STARTTIMEGMT1525590040000-4679190040000",
  "type": "LDAP Plus AD HelpDesk Professional Tool"
}

// LDAP Admin & Reporting Tool
{
  "email": "myluamas@gmail.com",
  "registrationKey": "VkVWU1FsVkdTa2ROVnpGb1VYcEdlazFYTVRWaVNGWm9ZbGRHZWxGSFpIUlpWMnh6VEcxT2RtSlZlRVZSVmtKVFVYcEZOVTlVYTNkT1V6aDNUbWs0ZWsxRVJUUT0=",
  "activationCode": "STARTTIMEGMT1525590051000-4679190051000",
  "type": "LDAP Admin \u0026 Reporting Tool"
}

// AD Admin Tool
{
  "email": "myluamas@gmail.com",
  "registrationKey": "VVZWU1IwMVhNV2hSZWtaNlRWY3hOV0pJVm1oaVYwWjZVVWRrZEZsWGJITk1iVTUyWWxWR1JWRjZSVFZQVkd0M1RsTTRkMDVwT0hwTlJFVTA=",
  "activationCode": "STARTTIMEGMT1525590060000-4679190060000",
  "type": "AD Admin Tool"
}
```

### LdapBrowser

`LDAPSoft`公司的免费产品，仅支持Windows且功能有限。

* [官方网站](https://www.ldapsoft.com)
* [下载地址](https://www.ldapsoft.com/ldapbrowserfree.html)

### LdapAdmin

仅支持Windows，优点是小巧、简单，不用安装即可运行。

* [官方网站](http://www.ldapadmin.org)
* [下载地址](http://www.ldapadmin.org/download/ldapadmin.html)

### JXplorer

支持Mac、Windows、Linux等，界面易用性感觉一般，还有企业版本`JXWorkBench`，但没有深入研究。

* [官方网站](http://www.jxplorer.org)
* [下载地址](http://www.jxplorer.org/downloads/users.html)

