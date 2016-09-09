---
title: CAS单点登录配置
date: 2016-06-12 16:02:44
tags:
- CAS
categories:
- 服务器

---

## 生成证书

1. 新建一个cas文件夹
2. 用以下命令生成server.keystore
```bash
jinyaoyuan:cas jinyaoyuan$ keytool -genkey -alias server -keyalg RSA -keypass mysql123 -storepass mysql123 -keystore server.keystore
您的名字与姓氏是什么?
  [Unknown]:  zzkt.sso.com
您的组织单位名称是什么?
  [Unknown]:  zzkt.sso.com
您的组织名称是什么?
  [Unknown]:  zzkt.sso.com
您所在的城市或区域名称是什么?
  [Unknown]:  beijing
您所在的省/市/自治区名称是什么?
  [Unknown]:  beijing
该单位的双字母国家/地区代码是什么?
  [Unknown]:  cn
CN=zzkt.sso.com, OU=zzkt.sso.com, O=zzkt.sso.com, L=beijing, ST=beijing, C=cn是否正确?
  [否]:  Y
```
	**<font color="#A52A2A">其中名字与姓氏一定要是cas服务器对应的域名，不能是IP</font>**

3. 导出证书
```bash
jinyaoyuan:cas jinyaoyuan$ keytool -export -alias server -storepass mysql123 -file server.cer -keystore server.keystore
存储在文件 <server.cer> 中的证书
```

4. 导入证书
```bash
jinyaoyuan:cas jinyaoyuan$ keytool -import -trustcacerts -alias server -file server.cer -keystore cacerts -storepass mysql123
所有者: CN=zzkt.sso.com, OU=zzkt.sso.com, O=zzkt.sso.com, L=beijing, ST=beijing, C=cn
发布者: CN=zzkt.sso.com, OU=zzkt.sso.com, O=zzkt.sso.com, L=beijing, ST=beijing, C=cn
序列号: 4406dd99
有效期开始日期: Sun Jun 12 16:19:09 CST 2016, 截止日期: Sat Sep 10 16:19:09 CST 2016
证书指纹:
	 MD5: 6F:AB:1B:6D:30:45:14:42:B9:09:4A:12:61:AA:42:CB
	 SHA1: F8:2E:22:34:A4:E4:C6:8E:61:7D:90:AC:99:2A:08:AC:A3:4C:1A:1C
	 SHA256: B5:17:93:49:60:F6:17:6C:0A:93:2A:5F:00:C5:E7:F7:33:B1:CE:D3:DC:A0:9F:C5:43:FF:68:CD:67:A2:F8:9A
	 签名算法名称: SHA256withRSA
	 版本: 3
扩展:
#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 9E F9 E0 9A 0A F4 E8 9C   B4 24 C6 85 44 C3 30 F7  .........$..D.0.
0010: 5F 24 B5 4C                                        _$.L
]
]
是否信任此证书? [否]:  Y
证书已添加到密钥库中
```

5. 完成上述步骤后cas文件下会生成server.keystore、server.cer、cacerts三个文件，将生成的cacerts文件替换到$JAVA_HOME/jre/lib/security/cacerts文件，**<font color="#A52A2A">替换前最好备份一下，这个文件有可能影响其他程序的正常运行</font>**。

6. 
-genkey      
在用户主目录中创建一个默认文件".keystore",还会产生一个mykey的别名，mykey中包含用户的公钥、私钥和证书
-alias       产生别名
-keystore    指定密钥库的名称(产生的各类信息将不在.keystore文件中
-keyalg      指定密钥的算法
-validity    指定创建的证书有效期多少天
-keysize     指定密钥长度
-storepass   指定密钥库的密码
-keypass     指定别名条目的密码
-dname   指定证书拥有者信息
-list  显示密钥库中的证书信息
-v           显示密钥库中的证书详细信息
-export      将别名指定的证书导出到文件
-file        参数指定导出到文件的文件名
-delete      删除密钥库中某条目
-keypasswd   修改密钥库中指定条目口令
-import      将已签名数字证书导入密钥库

## CAS配置

1. 将生成的server.keystore放入%TOMCAT_HOME%\conf下。修改%TOMCAT_HOME%\conf\server.xml文件
 	
		 <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="http" secure="true"
               clientAuth="false" sslProtocol="TLS" keystoreFile="./conf/server.keystore" 
		keystorePass="mysql123"/>
