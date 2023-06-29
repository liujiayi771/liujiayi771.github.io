---
title: Presto使用OpenLDAP进行登录认证
date: 2020-06-04 15:17:44
tags: LDAP
---
presto对接OpenLDAP只支持ldaps，需要OpenLDAP开启SSL，本文将首先介绍OpenLDAP如何开启SSL。

## OpenLDAP开启SSL
OpenLDAP开启SSL需要生成LDAP证书，有两种证书生成方式：
1. 自签名证书。该方式较为简单，LDAP客户端需要进行设置不进行证书校验；
2. CA签名证书，可以使用内部CA或外部CA签名证书。需要将给ldap server证书签名的CA证书放在客户端的`/etc/openldap/cacerts/`目录中，以便LDAP客户端可以验证证书。同时，客户端需要在`/etc/openldap/ldap.conf`中配置`TLS_CACERT /etc/openldap/cacerts/ca.cert.pem`

两种方式任选一种即可。

### 自签名证书
在`/etc/openldap/certs/`目录中创建LDAP server自签名证书：
```bash
openssl req -new -x509 -nodes -out /etc/openldap/certs/ldap.crt -keyout /etc/openldap/certs/ldap.key -days 1460
```

填写信息中比较重要的是Common Name，需要填写安装OpenLDAP的主机的hostname，其他信息可以不填写：
```bash
Country Name (2 letter code) [XX]: CN
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []: localhost
Email Address []:
```
<!-- more -->
创建完成后，在`/etc/openldap/certs`目录下会产生`ldap.key`和`ldap.crt`，修改这两个文件的属主：
```bash
chown ldap:ldap /etc/openldap/certs/ldap.*
```

之后，我们通过ldif文件修改配置的方式修改OpenLDAP中关于证书信息的配置项，新建`certs.ldif`文件，内容如下：
```bash
dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/ldap.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldap.key
```

在修改配置前，我们先看一下原有配置的内容：
```bash
slapcat -b "cn=config" | egrep "olcTLSCertificateKeyFile|olcTLSCertificateFile"
```

使用`certs.ldif`修改配置的命令为：
```bash
ldapmodify -Y EXTERNAL  -H ldapi:/// -f certs.ldif
```

修改完成后可以使用上述检查配置的命令检查一下修改是否生效了。

### CA签名证书
在`/etc/pki/CA`目录中创建`serial`和`index.txt`文件用于记录生成证书过程：
```bash
cd /etc/pki/CA
echo 0001 > serial
touch index.txt
```
#### 1. 生成CA所使用的密钥
```bash
openssl genrsa -aes256 -out /etc/pki/CA/private/ca.key.pem
```
输入秘钥的密码。

#### 2. 创建CA证书
```bash
openssl req -new -x509 -days 3650 -key /etc/pki/CA/private/ca.key.pem -extensions v3_ca -out /etc/pki/CA/certs/ca.cert.pem
```
创建证书需要输入一些信息，Country Name可以输入CN，State or Province Name、Locality Name、Organization Name、Organizational Unit Name我们可以填入一个固定的字符串，比如“ZZ”，在后面创建ldap server的证书时，填写的内容需要与这里保持一致，最终使用CA签名时才能签名成功，Common Name需要填写安装OpenLDAP的主机的hostname，其他没有说明的地方可以不用填写。
> 注意：CN（Common Name）和主机名必须要匹配

```bash
Enter pass phrase for /etc/pki/CA/private/ca.key.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:ZZ
Locality Name (eg, city) [Default City]:ZZ
Organization Name (eg, company) [Default Company Ltd]:ZZ
Organizational Unit Name (eg, section) []:ZZ
Common Name (eg, your name or your server's hostname) []:localhost
Email Address []:
```

#### 3. 创建ldap server使用的秘钥
```bash
openssl genrsa -out /etc/pki/CA/private/ldap.key
```

#### 4. 创建ldap server的证书文件
```bash
openssl req -new -key /etc/pki/CA/private/ldap.key -out /etc/pki/CA/certs/ldap.csr
```
输入信息需要和创建CA证书时保持一致，CN填入OpenLDAP安装主机的hostname。

#### 5. 使用CA对ldap server的证书进行签名
```bash
openssl ca -keyfile /etc/pki/CA/private/ca.key.pem -cert /etc/pki/CA/certs/ca.cert.pem -in /etc/pki/CA/certs/ldap.csr -out /etc/pki/CA/certs/ldap.crt
```

这时候已经完成了ldap server证书由我们创建的CA进行签名的过程，此时`index.txt`文件中的内容应该已经发生了变化，使用`cat /etc/pki/CA/index.txt`进行查看。我们也可以使用`openssl`来验证ldap server证书是否被CA正确签名：
```bash
openssl verify -CAfile /etc/pki/CA/certs/ca.cert.pem /etc/pki/CA/certs/ldap.crt
```

我们需要将证书、秘钥文件拷贝到`/etc/openldap/certs`目录中，将CA证书放到`/etc/openldap/cacerts/`目录中：
```bash
cp -v /etc/pki/CA/certs/* /etc/openldap/certs/
cp -v /etc/pki/CA/private/ldap.key /etc/openldap/certs/
mkdir /etc/openldap/cacerts/
cp -v /etc/pki/CA/certs/ca.cert.pem /etc/openldap/cacerts/
```
修改目录属主：
```bash
chown -R ldap:ldap /etc/openldap/certs
chown -R ldap:ldap /etc/openldap/cacerts
```

#### 6. 修改OpenLDAP配置生效CA签名证书
新建`certs.ldif`文件，其内容如下：
```bash
dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/ldap.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldap.key
-
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/cacerts/ca.cert.pem
```
执行如下命令进行修改：
```bash
ldapmodify -Y EXTERNAL -H ldapi:// -f certs.ldif
```
使用如下命令确保生效：
```bash
slapcat -b "cn=config" | egrep "olcTLSCertificateFile|olcTLSCertificateKeyFile|olcTLSCACertificateFile"
```

### 配置OpenLDAP开启SSL监听
修改配置文件`/etc/sysconfig/slapd`，添加`ldaps`：
```bash
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
```
#### 如果是CA签名证书
修改`/etc/openldap/ldap.conf`中的配置，添加服务端CA证书的存储路径：
```
#TLS_CACERTDIR /etc/openldap/certs
TLS_CACERT /etc/openldap/cacerts/ca.cert.pem
```
#### 如果是自签名证书
自签名证书需要修改`/etc/openldap/ldap.conf`中的配置，关闭校验：
```bash
#TLS_CACERTDIR /etc/openldap/certs
TLS_REQCERT never
```
重启OpenLDAP：
```bash
systemctl restart slapd
```
重启完成后，首先检查ldaps所使用的636端口是否已经启动：
```bash
netstat -nlp | grep 636
```
可以使用如下命令来测试ldaps连接：
```bash
ldapwhoami -x -H ldaps://<ldap_server>:636
```
如果能够成功显示anonymous，说明成功通过SSL连上了OpenLDAP server，同时OpenLDAP server是支持匿名访问的（老版本Presto连接ldap需要ldap开启匿名访问）。

也可以使用如下简单认证的方式来验证ldaps连接是否正确（该方法不能测试匿名连接，如果上面方法失败，但是这个方法成功，说明ldaps连接没问题，但是匿名访问不被允许，若Presto版本对接ldap必须使用匿名访问，则需要配置OpenLDAP ACL或删除disallow匿名访问的配置）：
```bash
ldapwhoami -x -w "<passwd>" -D <user_dn> -H ldaps://<ldap_server>:636
```

## 配置Presto支持LDAP认证
Presto支持LDAP认证，客户端必须以HTTPS的方式访问presto server。开启LDAP认证只需要对presto coordinator进行配置使其支持SSL/TLS，presto worker不需要进行修改，只有客户端和presto coordinator之间的连接需要认证。

### 导入ldap server证书到java cacerts
Presto服务要想通过ldaps与ldap server通信，必须将ldap server的证书导入java cacerts，从源码上看Presto服务会直接读取java的cacerts：
```bash
keytool -import -alias ldap_server -file /etc/openldap/certs/ldap.crt -keystore ${JAVA_HOME}/jre/lib/security/cacerts -storepass changeit
```

### 生成presto server秘钥
配置presto coordinator支持SSL/TLS，需要生成presto server的秘钥（keystore）文件。

#### 1. 生成presto server私钥：
```bash
keytool -genkey -v -keystore presto-private.keystore -alias presto-private -keyalg RSA -dname "CN=<hostname>, OU=, O=, L=, ST=, C=CN" -validity 20000 -keypass <passwd> -storepass <passwd>
```
这里CN（Common Name）必须填写presto server所在主机的hostname，其他的可以不需要填写。

#### 2. 导出证书文件
```bash
keytool -export -alias presto-private -keystore presto-private.keystore -file presto-public.cer -storepass <passwd>
```

#### 3. 由证书文件生成公钥给客户端使用
```bash
keytool -import -alias presto-public -file presto-public.cer -keystore presto.keystore -storepass <passwd>
```
此处`<passwd>`可与私钥不同，后续客户端登录时需要显示输入该密码

### 修改Presto coordinator配置
修改`etc/config.properties`文件，添加如下配置：
```bash
http-server.authentication.type=PASSWORD
http-server.https.enabled=true
http-server.https.port=8443

http-server.https.keystore.path=<private_keystore_path>
http-server.https.keystore.key=<keystore_passwd>
```
修改`etc/password-authenticator.properties`文件，添加如下配置：
```bash
password-authenticator.name=ldap
ldap.url=ldaps://<ldap_server>:636
ldap.user-bind-pattern=uid=${USER},dc=example,dc=com
```
`ldap.user-bind-pattern`需要填写登录用户如何映射到ldap的dn，其中`${USER}`表示登录的用户名，即presto cli登录时`--username`参数填写的内容。

配置完成后需要重启Presto coordinator使配置生效，之后便可以使用客户端登录，进行ldap认证了。

### Presto cli登录配置
使用如下命令连接presto server，相比没有ldap认证时候需要填写keystore、user、password等内容：
```bash
presto --server https://<presto_server>:<https_port> --keystore-path <public_keystore_path> --keystore-password <public_keystore_passwd> --catalog hive --user <ldap_username> --password
```
输入密码之后将可以进入presto命令行。若密码输入错误，也可以进入presto命令行，但是执行presto命令会出现认证失败的报错。输入正确的ldap用户名和密码，执行`show schemas;`，若能正确输出结果说明配置正确。


## OpenLDAP客户端通过ldaps访问ldap服务
下面也顺便把ldap客户端通过ldaps访问ldap服务的方法说一下，也分为自签名证书和CA签名证书。
客户端需要安装：
```bash
yum install -y openldap-clients nss-pam-ldapd
```
#### 1. 使用autoconfig生成客户端配置
```bash
authconfig --enableldap --enableldapauth --ldapserver=ldaps://<ldap_server> --ldapbasedn="<base_dn>" --enablemkhomedir --disableldaptls --update
```

### 自签名证书需要完成的步骤
修改`/etc/nslcd.conf`配置
```bash
tls_reqcert allow
```

### CA签名证书
需要将ldap server的CA前面证书`/etc/openldap/cacerts/ca.cert.pem`拷贝到客户端，放在`/etc/openldap/cacerts/`目录中。
通过如下命令给证书生成hash：
```bash
/etc/pki/tls/misc/c_hash /etc/openldap/cacerts/ca.cert.pem
```
会输出证书的hash值，需要做一个名称为hash值软链放在`/etc/openldap/cacerts/`目录中，假设hash值输出为`6822c9d7.0`，则软链为：
```bash
ln -s /etc/openldap/cacerts/ca.cert.pem /etc/openldap/cacerts/6822c9d7.0
```

重启ldap客户端服务：
```bash
systemctl restart nslcd
```

以上的方式比较正式，如果嫌麻烦，也可以简单配置`/etc/openldap/ldap.conf`跳过证书校验：
```bash
#TLS_CACERTDIR /etc/openldap/certs
TLS_REQCERT never
```
CA签名证书的方式也可以将CA证书拷贝到客户端之后，修改`/etc/openldap/ldap.conf`：
```bash
#TLS_CACERTDIR /etc/openldap/certs
TLS_CACERT /etc/openldap/cacerts/ca.cert.pem
```
达到的效果是类似的，最后也是用如下命令在客户端上进行校验：
```bash
ldapwhoami -x -H ldaps://<ldap_server>:636
```
输出`anonymous`说明连接成功，在关闭匿名访问的情况下，用如下方式输入ldap的dn和密码进行校验：
```bash
ldapwhoami -x -w "<passwd>" -D <user_dn> -H ldaps://<ldap_server>:636
```
