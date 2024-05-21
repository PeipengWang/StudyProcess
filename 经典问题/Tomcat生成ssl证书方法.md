keytool生成（jdk自带工具）

keytool -genkey -alias wasTomcatServer -keyalg RSA -keysize 2048 -keypass 123456 -dname "CN=132.224.118.246,OU=ZTE,O=ZTE,L=Nanjing,ST=Jiangsu,C=CN" -keystore server.keystore -storepass 123456789 -sigalg SHA256withRSA -storetype pkcs12 -validity 9000

    其中：

CN=132.224.118.246：ip地址根据实际的服务部署ip地址修改，如果对外提供域名则直接修改为域名。

-keypass 123456：秘钥库密码，该密码与步骤4中server.xml文件的keystorePass="123456"一致。

2、备份原来证书及相关配置文件

     备份tomcat/keystore下的证书

     备份tomcat/conf/server.xml

3、将新生成的证书上传到目录tomcat/keystore

4、修改tomcat/conf/server.xml中的8443端口的配置

server.xml的具体配置说明如下：

<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol" SSLEnabled="true" maxThreads="150" scheme="https" secure="true" keystoreFile="keystore/serverKey.keystore"  keystorePass="123456" clientAuth="false" sslProtocol="TLS"  sslEnabledProtocols="TLSv1.2" ciphers="TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_DHE_RSA_WITH_AES_256_GCM_SHA384" URIEncoding="ISO-8859-1"/>
