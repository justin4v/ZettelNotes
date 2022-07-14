#Springboot #Https

# 背景
本系统接口都是http的，调用第三方接口，因为做了安全性校验，所以不能通过RestTemplate调用

# 解决
- 重写覆盖 `SimpleClientHttpRequestFactory` 抽象类的 `prepareConnection` 方法

```java
package com.minstone.apprBase.common.utils.http.rest;

import org.apache.http.conn.ssl.SSLContexts;
import org.apache.http.conn.ssl.TrustStrategy;
import org.springframework.http.client.SimpleClientHttpRequestFactory;

import javax.net.ssl.HttpsURLConnection;
import javax.net.ssl.SSLContext;
import java.net.HttpURLConnection;
import java.security.KeyStore;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

/**
 * 兼容调Https接口
 * @Author mazq
 * @Date 2020/06/04 17:16
 * @Param
 * @return
 */
public class HttpsClientRequestFactory extends SimpleClientHttpRequestFactory {
 
    @Override
    protected void prepareConnection(HttpURLConnection connection, String httpMethod) {
        try {
            if (!(connection instanceof HttpsURLConnection)) {// http协议
                //throw new RuntimeException("An instance of HttpsURLConnection is expected");
                super.prepareConnection(connection, httpMethod);
            }
            if (connection instanceof HttpsURLConnection) {// https协议，修改协议版本
                KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());
                // 信任任何链接
                TrustStrategy anyTrustStrategy = new TrustStrategy() {
                    @Override
                    public boolean isTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {
                        return true;
                    }
                };
                SSLContext ctx = SSLContexts.custom().useTLS().loadTrustMaterial(trustStore, anyTrustStrategy).build();
                ((HttpsURLConnection) connection).setSSLSocketFactory(ctx.getSocketFactory());
                HttpsURLConnection httpsConnection = (HttpsURLConnection) connection;
                super.prepareConnection(httpsConnection, httpMethod);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```
- 关键代码，`new RestTemplate(new HttpsClientRequestFactory());`
- 对应工具类参考：

```java
package com.minstone.apprBase.common.utils.http.rest;

import com.minstone.apprBase.common.utils.web.WebUtil;
import org.springframework.http.*;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

/**
 * <pre>
 *      RestTemplate 远程调用工具类
 * </pre>
 *
 * <pre>
 * @author mazq
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2020/06/01 11:38  修改内容:
 * </pre>
 */
@Component
public class RestTemplateUtils {


    public static RestTemplate geTemplate(){
        return new RestTemplate(new HttpsClientRequestFactory());
    }

    /**
     * GET请求调用方式
     * @Author mazq
     * @Date 2020/06/01 13:47
     * @Param [url, responseType, uriVariables]
     * @return org.springframework.http.ResponseEntity<T>
     */
    public static <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables) {
        return geTemplate().getForEntity(url, responseType, uriVariables);
    }

    /**
     * POST请求调用方式
     * @Author mazq
     * @Date 2020/06/01 13:47
     * @Param [url, headers, body, responseType]
     * @return org.springframework.http.ResponseEntity<T>
     */
    public static <T> ResponseEntity<T> postForEntity(String url,HttpHeaders headers, Object requestBody , Class<T> responseType ){

        HttpEntity<Object> requestEntity = new HttpEntity<Object>(requestBody, headers);
        return  geTemplate().postForEntity(url, requestEntity, responseType);
    }

    /**
     * PUT请求调用方式
     * @Author mazq
     * @Date 2020/06/01 13:35
     * @param url 请求URL
     * @param headers 请求头参数
     * @param requestBody 请求参数体
     * @param responseType 返回对象类型
     * @param uriVariables URL中的变量，与Map中的key对应
     * @return ResponseEntity 响应对象封装类
     */
    public static <T> ResponseEntity<T> put(String url, HttpHeaders headers, Object requestBody, Class<T> responseType, Map<String, ?> uriVariables) {
        HttpEntity<Object> requestEntity = new HttpEntity<Object>(requestBody, headers);
        return geTemplate().exchange(url, HttpMethod.PUT, requestEntity, responseType, uriVariables);
    }

    /**
     * DELETE请求调用方式
     * @Author mazq
     * @Date 2020/06/01 13:37
     * @param url 请求URL
     * @param headers 请求头参数
     * @param requestBody 请求参数体
     * @param responseType 返回对象类型
     * @param uriVariables URL中的变量，按顺序依次对应
     * @return ResponseEntity 响应对象封装类
     */
    public static <T> ResponseEntity<T> delete(String url, HttpHeaders headers, Object requestBody, Class<T> responseType, Object... uriVariables) {
        HttpEntity<Object> requestEntity = new HttpEntity<Object>(requestBody, headers);
        return geTemplate().exchange(url, HttpMethod.DELETE, requestEntity, responseType, uriVariables);
    }

    /**
     * 通用调用方式
     * @Author mazq
     * @Date 2020/06/01 13:37
     * @Param [url, method, requestEntity, responseType, uriVariables]
     * @return org.springframework.http.ResponseEntity<T>
     */
    public static <T> ResponseEntity<T> exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType, Map<String, ?> uriVariables) {
        return geTemplate().exchange(url, method, requestEntity, responseType, uriVariables);
    }
}
```

# 参考
1. [SpringBoot系列之RestTemplate调https接口 ](https://juejin.cn/post/6844904199214350344)