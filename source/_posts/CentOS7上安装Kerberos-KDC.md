---
title: CentOS7上安装Kerberos KDC
date: 2020-04-24 17:29:41
tags: kerberos
---

### yum安装软件
首先通过yum安装kerberos所需要的软件

```bash
yum install -y krb5-server krb5-workstation pam_krb5
```

### 修改服务端配置
```bash
cd /var/kerberos/krb5kdc
```
我们需要修改该目录下的两个配置文件：
* kdc.conf
* kadm5.acl

假设我们的realm为KDC.COM，修改`kadm5.acl`为如下内容：
```bash
*/admin@KDC.COM     *
```
修改`kdc.conf`为如下内容：
```bash
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 KDC.COM = {
  #master_key_type = aes256-cts
  max_renewable_life = 7d
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```

### 修改客户端配置
我们用作KDC服务的服务器也将会是一个Kerberos客户端，我们需要修改客户端配置文件`/etc/krb5.conf`，文件内容如下：
```bash
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = /etc/pki/tls/certs/ca-bundle.crt
 default_realm = KDC.COM
 default_ccache_name = KEYRING:persistent:%{uid}
 default_tgs_enctypes = aes128-cts arcfour-hmac-md5 des-cbc-crc des-cbc-md5 des-hmac-sha1 aes256-cts
 default_tkt_enctypes = aes128-cts arcfour-hmac-md5 des-cbc-crc des-cbc-md5 des-hmac-sha1 aes256-cts
 permitted_enctypes = aes256-cts-hmac-sha1-96 des3-cbc-sha1 arcfour-hmac-md5 des-cbc-crc des-cbc-md5 des-cbc-md4
 allow_weak_crypto = yes

[realms]
 KDC.COM = {
  kdc = kdc.hostname
  admin_server = kdc.hostname
 }

[domain_realm]
 .kdc.com = KDC.COM
 kdc.com = KDC.COM
```

### 创建KDC数据库
```bash
kdb5_util create -s -r KDC.COM
```

### 开启Kerberos
```bash
systemctl start krb5kdc kadmin
systemctl enable krb5kdc kadmin
```

接下来需要创建kadmin的管理员账号：
```bash
kadmin.local
kadmin.local: addprinc root/admin
kadmin.local: quit
```

### 修改SSH客户端配置

修改`/etc/ssh/ssh_config`，添加如下配置：
```bash
GSSAPIAuthentication yes
GSSAPIDelegateCredentials yes
```

执行：
```bash
authconfig  --enablekrb5 --update
systemctl reload sshd
```
