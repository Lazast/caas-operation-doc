# ldap

LDAP

1. ldap简介 ldap是轻量目录访问协议，英文全称是Lightweight Directory Access Protocol，一般都简称为ldap。ldap以树状结构存储数据，非常适合存储大量数据，而且数据不经常修改，但是需要快速查询的场景，比如业务系统的鉴权操作。 ldap支持TCP/IP协议，分为server 端和client端，server端用来存储数据，client用来增删查改数据，监听端口为389，加密端口为636。其数据存储结构完全体现了树状的特点： ldap树形结构 dn：表示ldap内条目具体位置,比如 dn: cn=admin,ou=users,dc=baifendian,dc=com； dc：表示条目所属的区域； ou：表示组织单元； cn/uid：表示一条记录具体的名字；
2. caas生产环境ldap安装

   caas生产环境中ldap是以容器的形式运行，所以安装ldap前必须具备docker环境，安装命令如下：

   ```bash
   docker run --name ldap1 --add-host ldap2.general.com:{{ groups.storages[1] }} \
     --restart=always \
     --hostname ldap1.general.com \
     --env LDAP_BASE_DN="dc=general,dc=com" \
     --env LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://ldap1.general.com:389','ldap://ldap2.general.com:3389']" \
     --env LDAP_REPLICATION=true \
     --env LDAP_TLS_VERIFY_CLIENT="never" \
     --env LDAP_DOMAIN="general.com" \
     --env LDAP_ADMIN_PASSWORD="generalcom" \
     -v /caas_data/ldap_data/ldap_config:/etc/ldap/slapd.d \
     -v /caas_data/ldap_data/ldap_data:/var/lib/ldap \
     -v /etc/localtime:/etc/localtime:ro \
     -p 3389:389 \
     --detach osixia/openldap:1.2.1 \
     --loglevel debug \
     --copy-service
   ```

   脚本解释：

   --add-host ldap2.general.com:：往容器内添加hosts，这样对方ldap才能同步，也可以直接将ldap1.general.com和ldap2.general.com在集群内做域名解析。

   LDAP\_REPLICATION\_HOSTS="\#PYTHON2BASH: 这一部分是两个ldap之间互相同步的关键。

   -v：挂载本地卷防止ldap容器挂了之后数据出现丢失的现象。

   -p 3389:389 ：3389表示ldap容器的监听端口，389是haproxy负载ldap的监听端口。

3. ldap命令 ldap分为server端和client端，如果要进行增删查改需要安装openldap，这个包提供几个比较常见的命令，ldapaddd（增加用户），ldapdelete（删除用户），ldapsearch（查找用户），ldapmodify（修改用户）。
   * ldapadd 示例1: 使用ldif文件添加用户

     ```text
     [root@storage1 ~]# ldapadd -x -H ldap://127.0.0.1:389 -D "cn=admin,dc=general,dc=com" -w generalcom -f /tmp/adduser.ldif 
     adding new entry "cn=baifendian001,ou=users,dc=general,dc=com"
     ```

     ldif文件格式如下：

     ```text
     [root@storage1 ~]# cat adduser.ldif 
     dn: cn=baifendian001,ou=users,dc=general,dc=com
     objectClass: person
     objectClass: organizationalPerson
     objectClass: inetOrgPerson
     cn: baifendian001
     sn: os
     displayName: os user
     mail: user@deploy.com
     userPassword: baifendian001
     ```

     示例2:

     直接在命令模式下添加用户

     ```text
     [root@storage1 ~]# ldapadd -x -H ldap://127.0.0.1:389 -D "cn=admin,dc=general,dc=com" -w generalcom
     dn: cn=baifendian002,ou=users,dc=general,dc=com
     objectClass: person
     objectClass: organizationalPerson
     objectClass: inetOrgPerson
     cn: baifendian002
     sn: os
     displayName: os user
     mail: user@deploy.com
     userPassword: baifendian002

     adding new entry "cn=baifendian006,ou=users,dc=general,dc=com"
     ```

* ldapdelete

  ```text
  [root@storage1 ~]# ldapdelete -x -H ldap://127.0.0.1:389  -D "cn=admin,dc=general,dc=com" -w generalcom "cn=baifendian006,ou=users,dc=general,dc=com"
  ```

  注意：使用ldapdelete时被删除的必须是dn。

  * ldapsearch 示例1： 查询整个ldap数据

    ```text
    以管理员admin权限查询ldap用户信息
    [root@storage1 ~]# ldapsearch -x -H ldap://127.0.0.1:389 -b "dc=general,dc=com" -D "cn=admin,dc=general,dc=com" -w generalcom
    ```

    示例2: 查询指定条目

    ```text
    查询cn=pp13staging用户信息
    [root@storage1 ~]# ldapsearch -x -H ldap://127.0.0.1:389 -b "dc=general,dc=com" -D "cn=admin,dc=general,dc=com" -w generalcom "cn=pp13staging"
    ```

* ldapmodify

  ```text
  [root@storage1 ~]# ldapmodify -x -H ldap://127.0.0.1:389 -D cn=admin,dc=general,dc=com -w generalcom -f /tmp/adduser.ldif 
  modifying entry "cn=baifendian002,ou=users,dc=general,dc=com"
  ```

  使用ldif文件改变用户属性，ldif文件如下：

  ```text
  [root@storage1 ~]# cat /tmp/adduser.ldif 
  dn: cn=baifendian002,ou=users,dc=general,dc=com
  changetype: modify
  replace: mail
  mail: 110@baifendian.com
  -
  delete: displayName
  -
  ```

  ​

