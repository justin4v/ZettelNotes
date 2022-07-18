#Springboot #Https

# 背景
本系统接口都是http的，调用第三方接口，因为做了安全性校验，所以不能通过RestTemplate调用

# 解决
- 自定义 RestTemplate 配置

```java
@Configuration
public class CustomRestConfig {

  @Bean
  public RestTemplate customRestTemplate()
      throws KeyStoreException, NoSuchAlgorithmException, KeyManagementException {
    // 实现了 org.apache.http.ssl.TrustStrategy，重写了 isTrusted(X509Certificate[] chain, String authType)
    // 总是返回 true
    TrustStrategy acceptingTrustStrategy = (X509Certificate[] chain, String authType) -> true;

    SSLContext sslContext = org.apache.http.ssl.SSLContexts.custom()
        .loadTrustMaterial(null, acceptingTrustStrategy)
        .build();

    SSLConnectionSocketFactory csf = new SSLConnectionSocketFactory(sslContext);

    CloseableHttpClient httpClient = HttpClients.custom()
        .setSSLSocketFactory(csf)
        .build();

    HttpComponentsClientHttpRequestFactory requestFactory =
        new HttpComponentsClientHttpRequestFactory();

    requestFactory.setHttpClient(httpClient);
    RestTemplate restTemplate = new RestTemplate(requestFactory);
    return restTemplate;
  }

  // 标准的 RestTemplate
  @Bean
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }
}
```
# 参考
1. [SpringBoot系列之RestTemplate调https接口 ](https://juejin.cn/post/6844904199214350344)