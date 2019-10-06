# 配置

🚩下面操作建议使用`root`用户操作

## 创建管理员密码

使用slappasswd命令创建管理员密码

```base
$ slappasswd -s bootioo
{SSHA}i5iFrmJ9Kfm84qv9J/20KMETlUqBhIjk
```

⚠️注意，新版OpenLDAP不再有slapd.conf配置文件，也不能直接修改`/etc/openldap/slapd.d`中的配置，需要通过`.ldif`文件配合`lpadadd`和`lpadmodify`命令修改。

```base
$ vi ~/chrootpw.ldif #文件名随意
```

```ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}i5iFrmJ9Kfm84qv9J/20KMETlUqBhIjk
```

```base
$ ldapadd -Y EXTERNAL -H ldapi:/// -f ~/chrootpw.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
```

## 禁止匿名用户登陆

```base
$ vi ~/disable_anon.ldif
```

```ldif
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon

dn: cn=config
changetype: modify
add: olcRequires
olcRequires: authc

dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcRequires
olcRequires: authc
```

```baes
$ ldapadd -Y EXTERNAL -H ldapi:/// -f ~/disable_anon.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"

modifying entry "cn=config"

modifying entry "olcDatabase={-1}frontend,cn=config"
```

## 导入基本Schema

```base
$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=inetorgperson,cn=schema,cn=config"
```

## 设置Domain Name

创建目录管理者密码

```base
$ slappasswd -s bootioo
{SSHA}OJ0EjQfH6j5xKCt0GPSqvBoLL3s34oO5
```

```base
$ vi ~/chdomain.ldif #文件名随意
```

```ldif
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=root,dc=domain,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=domain,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=root,dc=domain,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}OJ0EjQfH6j5xKCt0GPSqvBoLL3s34oO5

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange
  by dn="cn=root,dc=domain,dc=com" write
  by anonymous auth
  by self auth
  by * none
olcAccess: {1}to dn.base=""
  by * read
olcAccess: {2}to *
  by dn="cn=root,dc=domain,dc=com" write
  by * read
```

```base
$ ldapmodify -Y EXTERNAL -H ldapi:/// -f ~/chdomain.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"
```

## 导入basedomain.ldif文件

```base
$ vi ~/basedomain.ldif #文件名随意
```

```ldif
dn: dc=domain,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: domain.com
dc: domain

dn: cn=root,dc=domain,dc=com
objectClass: organizationalRole
cn: root

dn: ou=users,dc=domain,dc=com
objectClass: organizationalUnit
ou: users

dn: ou=groups,dc=domain,dc=com
objectClass: organizationalUnit
ou: groups
```

```base
$ ldapadd -x -D cn=root,dc=domain,dc=com -W -f ~/basedomain.ldif
Enter LDAP Password: ##此处需要输入管理密码
adding new entry "dc=domain,dc=com"

adding new entry "cn=root,dc=domain,dc=com"

adding new entry "ou=users,dc=domain,dc=com"

adding new entry "ou=groups,dc=domain,dc=com"
```

## 添加memberOf模块

该模块的作用是，当为`groupOfNames`类型的组添加用户时，为用户添加名称为`memberOf`的属性，而值是组的`DN`。配合`refint`模块可以在用户删除和更名时同步更新。

以应用后台管理员验证为例，我们有一个名为cms的应用，在openldap中建立`groupOfNames`类型的组，其DN为`cn=cms,ou=groups,dc=domain,dc=com`，并且设置其`memeber`（可以设置多个）。管理员登陆时通过`memberOf=cn=cms,ou=groups,dc=domain,dc=com`过滤出隶属于`cms`组的会员进行验证。

⚠️另外多说一点，我们的用户以web登录为主，因此添加`inetOrgPerson`类型的用户即可。

### memberof

```base
$ vi ~/memberof.ldif #文件名随意
```

```ldif
dn: cn=module,cn=config
cn: module
objectClass: olcModuleList
olcModuleLoad: memberof
olcModulePath: /usr/lib64/openldap

dn: olcOverlay={0}memberof,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfNames
olcMemberOfMemberAD: member
olcMemberOfMemberOfAD: memberOf
```

```base
$ ldapadd -Q -Y EXTERNAL -H ldapi:/// -f ~/memberof.ldif
adding new entry "cn=module,cn=config"

adding new entry "olcOverlay={0}memberof,olcDatabase={2}hdb,cn=config"
```

### refint

```base
$ vi ~/refint1.ldif #文件名随意
```

```ldif
dn: cn=module{0},cn=config
add: olcmoduleload
olcmoduleload: refint
```

```base
$ ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f ~/refint1.ldif
modifying entry "cn=module{0},cn=config"
```

```base
$ vi ~/refint2.ldif #文件名随意
```

```ldif
dn: olcOverlay={1}refint,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: {1}refint
olcRefintAttribute: memberof member manager owner
```

```base
$ ldapadd -Q -Y EXTERNAL -H ldapi:/// -f ~/refint2.ldif
adding new entry "olcOverlay={1}refint,olcDatabase={2}hdb,cn=config"
```

## 使设置生效

```base
$ systemctl restart slapd
```

## 测试

```bash
$ ldapsearch -D "cn=root,dc=domain,dc=com" -w "bootioo" -h 192.168.99.179 -p 389 -b 'ou=groups,dc=domain,dc=com' -s base'(objectclass=*)'
```

<!--cn=Manager,dc=my-domain,dc=com ldap123-->

