#Https #Springboot #Keytool 

# 获取证书
## jdk keytool 生成自定义证书
```shell
keytool -genkey -alias httpsTest -dname "CN=localhost,OU=UPlus,O=UIH,L=WH,ST=HB,C=CN" -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 365
```
- 在当前目录生成证书文件 `keystore.p12`；
- 这里的 `CN (Common Name)` 为 `localhost` ，相当于*给 localhost 颁发了证书*。

### 命令含义
- `alias` 别名  
- `keypass` 指定生成密钥的密码  
- `keyalg` 指定密钥使用的加密算法（如 RSA）  
- `keysize` 密钥大小  
- `validity` 过期时间，单位天  
- `keystore` 指定存储密钥的密钥库的生成路径、名称  
- `storepass` 指定访问密钥库的密码

keytool 中 `dname` 字段含义：
1. `C` ：Country，单位所在国家，为两位数的国家缩写，如：CN 表示中国；
2. `ST` ：State/Province，单位所在州或省；
3. `L` ：Locality，单位所在城市/或县区；
4. `O` ：Organization，此网站的单位名称；
5. `OU` ：Organization Unit，下属部门名称，也常常用于显示其他证书相关信息，如证书类型，证书产品名称或身份验证类型或验证内容等；
6. `CN` ：Common Name，网站的域名；


## Linux Openssl 生成证书
1. 安装 Openssl
```shell
yum install openssl openssl-devel -y
```
2. 生成一个RSA密钥 （私钥）
```shell
openssl genrsa -out server.key 2048
```
3. 生成一个证书请求
```shell
openssl req -new -key server.key -out server.csr -subj “/C=CN/ST=HB/L=WH/O=UIH/OU=UPlus/CN=localhost”
```
4. 自己签发证书
```shell
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
5. 转换为pkcs12格式（因为在 Java 中使用证书，需要转换一下格式）
```shell
openssl pkcs12 -export -clcerts -in server.crt -inkey server.key -out keystore.p12
```


# Springboot 配置
- 将上面生成的*证书复制到 springboot 工程 resource 目录*（本例中改名为 `server.keystore`）

## application.yaml 配置
```yaml
server:  
  port: 443 # https默认访问端口  
  ssl:  
  key-store: classpath:server.keystore # 证书存放的位置  
  key-alias: httpsTest # 证书别名  
  key-store-type: PKCS12 # P12证书格式  
  key-store-password: Admin123
```

## 配置 tomcat

```java
@Configuration  
public class CustomTomcatConfig{  

  //springboot2 新变化  
  @Bean  
  public TomcatServletWebServerFactory servletContainer() { 
	  TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {  
		  @Override  
		  protected void postProcessContext(Context context) {  
			  SecurityConstraint securityConstraint = new SecurityConstraint();  
			  securityConstraint.setUserConstraint("CONFIDENTIAL");  
			  SecurityCollection collection = new SecurityCollection();  
			  collection.addPattern("/*");  
			  securityConstraint.addCollection(collection);  
			  context.addConstraint(securityConstraint);  
		  }  
	  };  
	  tomcat.addAdditionalTomcatConnectors(initiateHttpConnector());  
	  return tomcat;  
  }  
  
  /**  
   * initiateHttpConnector http(port 8080) redirect to https(port 8443)  
   *  
  * @return Connector  
   */  
  private Connector initiateHttpConnector() {  
	  Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");  
	  connector.setScheme("http");  
	  connector.setPort(8080);  
	  connector.setSecure(false);  
	  // 设置 http 请求自动条状 HTTPS
	  // 不需要设置 http 自动跳转（限制必须使用https）时，可注释该行
	  connector.setRedirectPort(443);  
	  return connector;  
  }  
}
```


## demo controller
```java
@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}

```

## 结果
1. **tomcat 启动信息**
```
2022-07-15 13:46:36.125  INFO 1164 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 443 (https) 8080 (http) with context path ''
2022-07-15 13:46:36.128  INFO 1164 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 4.638 seconds (JVM running for 5.331)
```
- https 启动在 443 (默认端口，访问时无需带上该端口)
- http 启动在 8080 端口


2. 页面

![[springboot Https 服务响应示意.png]]
3. Http 请求自动跳转 Https

![[http重定向到https.gif]]



## pom
```xml
	<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.1.0.RELEASE</spring-boot.version>
        <tomcat.version>9.0.12</tomcat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>${tomcat.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
	        <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.0</version>
                <configuration>
                    <!--   maven install 时跳过 UT  -->
                    <skipTests>true</skipTests>
                    <!--   maven install 忽略失败的 UT  -->
                    <testFailureIgnore>true</testFailureIgnore>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>com.example.demo.DemoApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

# 异常处理
## UnsatisfiedLinkError
- 错误 stack
```
2022-07-15 14:06:34.607 ERROR 12756 --- [-nio-443-exec-6] org.apache.tomcat.util.net.NioEndpoint   : 

java.lang.UnsatisfiedLinkError: org.apache.tomcat.jni.SSL.renegotiatePending(J)I
	at org.apache.tomcat.jni.SSL.renegotiatePending(Native Method) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.openssl.OpenSSLEngine.getHandshakeStatus(OpenSSLEngine.java:1037) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.openssl.OpenSSLEngine.wrap(OpenSSLEngine.java:458) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at java.base/javax.net.ssl.SSLEngine.wrap(SSLEngine.java:479) ~[na:na]
	at org.apache.tomcat.util.net.SecureNioChannel.handshakeWrap(SecureNioChannel.java:440) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.SecureNioChannel.handshake(SecureNioChannel.java:211) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1394) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128) ~[na:na]
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628) ~[na:na]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) ~[tomcat-embed-core-9.0.14.jar:9.0.14]
	at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]

```

### 原因
- springboot 内嵌 Tomcat 版本的问题；
- 参考 [Spring Boot 2.1.1: UnsatisfiedLinkError: org.apache.tomcat.jni.SSL.renegotiatePending](https://stackoverflow.com/questions/53596134/spring-boot-2-1-1-unsatisfiedlinkerror-org-apache-tomcat-jni-ssl-renegotiatepe)
- 解决：embed tomcat 使用 9.0.12 版本（springboot 版本为 2.1.0.RELEASE）
	- springboot 版本尽量*不高于 2.1.0.RELEASE*


## PKIX path building failed
- 错误 stack

```
org.springframework.web.client.ResourceAccessException: I/O error on GET request for "https://localhost/home": PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target; nested exception is javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:743)
	at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:669)
	at org.springframework.web.client.RestTemplate.getForEntity(RestTemplate.java:337)
```


### 原因
- 缺少


# 参考
