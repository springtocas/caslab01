# 配置Tomcat使用https协议



## 一、环境：

**os:ubuntu18.04**

**jdk:8**

**tomcat:9.0.14**



## 二、为服务器生成证书

```
keytool -genkey -v -alias tomcat -keyalg RSA -keystore /home/yeyalin/soft/https/tomcat/tomcat.keystore -validity 36500
```

- 输入密钥库口令 123456

- 您的名字与姓氏是什么？

  必须是tomcat 部署的域名或ip,本地开发测试应填入**localhost**

- 您的组织单位名称是什么?、您的组织名称是什么?、您所在的城市或区域名称是什么?、您所在的省/市/自治区名称是什么?、该单位的双字母国家/地区代码是什么?

​        这些随便输入，其中该单位的双字母国家/地区代码是CN

- 输入 <tomcat> 的密钥口令 (如果和密钥库口令相同, 按回车): 直接回车

## 三、为客户端生成证书

```
keytool -genkey -v -alias mykey -keyalg RSA -storetype PKCS12 -keystore /home/yeyalin/soft/https/tomcat/mykey.p12
```

- 输入密钥库口令 123456

- 您的名字与姓氏是什么？

  必须是tomcat 部署的域名或ip,本地开发测试应填入**localhost**

- 您的组织单位名称是什么?、您的组织名称是什么?、您所在的城市或区域名称是什么?、您所在的省/市/自治区名称是什么?、该单位的双字母国家/地区代码是什么?

​        这些随便输入，其中该单位的双字母国家/地区代码是CN

## 四、让服务器信任客户端证书

-  导出证书成单独的cer文件：

```
keytool -export -alias mykey -keystore  /home/yeyalin/soft/https/tomcat/mykey.p12  -storetype PKCS12 -storepass 123456 -rfc -file   /home/yeyalin/soft/https/tomcat/mykey.cer
```

- 将该文件导入到服务器的证书库，添加为一个信任证书使用命令如下：

```
keytool -import -v -file /home/yeyalin/soft/https/tomcat/mykey.cer -keystore /home/yeyalin/soft/https/tomcat/tomcat.keystore 
```

- 输入密码：123456

- 通过list命令查看服务器的证书库，可以看到两个证书，一个是服务器的 ，一个是受信任的客户端证书:

```
keytool -list -keystore /home/yeyalin/soft/https/tomcat/tomcat.keystore 
```



## 五、让客户端信任服务端证书



```
keytool  -keystore  /home/yeyalin/soft/https/tomcat/tomcat.keystore  -export -alias tomcat  -file /home/yeyalin/soft/https/tomcat/tomcat.cer
```



## 六、配置Tomcat服务器

- ​     打开Tomcat根目录下的conf/server.xml,找到Connector port="8443"配置段，修改如下：

```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
	maxThreads="150" SSLEnabled="true" scheme="https"
    secure="true"  clientAuth="true" sslProtocol="TLS"
    keystoreFile="/home/yeyalin/soft/https/tomcat/tomcat.keystore" keystorePass="123456"
    truststoreFile="/home/yeyalin/soft/https/tomcat/tomcat.keystore" truststorePass="123456">
</Connector>
```

- 属性说明：

```
clientAuth：是否启用双向认证，true代表启用双向认证。

keystoreFile：服务器证书路径

truststoreFile：用来验证客户端正式的根证书，此例中就是服务器证书。


```



## 七、测试

启动tomcat,浏览器访问https://localhost:8443/

如果出现以下提示，由于是自签证书，不是公立机构设置的证书，浏览器不认可，这时候需要在浏览器中导入客户端证书：

 【浏览器设置】-->【高级】-->【证书管理】-->【导入】-->【选择mykey.p12】

```
此网站无法提供安全连接
localhost** 不接受您的登录证书，或者您可能没有提供登录证书。

- 请尝试联系系统管理员。

ERR_BAD_SSL_CLIENT_AUTH_CERT
```

