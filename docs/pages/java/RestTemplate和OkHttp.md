## RestTemplate
> 是 Spring 提供的用于访问Rest服务的客户端， RestTemplate 提供了多种便捷访问远程Http服务的方法,能够大大提高客户端的编写效率。
#### 添加初始化配置
> （也可以不配，有默认的）--注意RestTemplate只有初始化配置，没有什么连接池
```java
@Configuration
public class ApiConfig {
    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        return new RestTemplate(factory);
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        //默认的是JDK提供http连接，需要的话可以//通过setRequestFactory方法替换为例如Apache HttpComponents、Netty或//OkHttp等其它HTTP library。
        factory.setReadTimeout(5000);//单位为ms
        factory.setConnectTimeout(5000);//单位为ms
        return factory;
    }
}
```
#### get请求
```java
// 1-getForObject()
User user1 = this.restTemplate.getForObject(uri, User.class);

// 2-getForEntity()
ResponseEntity<User> responseEntity1 = this.restTemplate.getForEntity(uri, User.class);
HttpStatus statusCode = responseEntity1.getStatusCode();
HttpHeaders header = responseEntity1.getHeaders();
User user2 = responseEntity1.getBody();
  
// 3-exchange()
RequestEntity requestEntity = RequestEntity.get(new URI(uri)).build();
ResponseEntity<User> responseEntity2 = this.restTemplate.exchange(requestEntity, User.class);
User user3 = responseEntity2.getBody();
```
##### 方式一：
```java
Notice notice = restTemplate.getForObject("http://fantj.top/notice/list/{1}/{2}", Notice.class, 1, 5);
```
##### 方式二：
```java
Map<String,String> map = new HashMap();
map.put("start","1");
map.put("page","5");
Notice notice = restTemplate.getForObject("http://fantj.top/notice/list/", Notice.class, map);
```
#### psot方式
```java
// 1-postForObject()
User user1 = this.restTemplate.postForObject(uri, user, User.class);

// 2-postForEntity()
ResponseEntity<User> responseEntity1 = this.restTemplate.postForEntity(uri, user, User.class);

// 3-exchange()
RequestEntity<User> requestEntity = RequestEntity.post(new URI(uri)).body(user);
ResponseEntity<User> responseEntity2 = this.restTemplate.exchange(requestEntity, User.class);
```
##### 方式一：
```java
String url = "http://demo/api/book/";
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
String requestJson = "{...}";
HttpEntity<String> entity = new HttpEntity<String>(requestJson,headers);
String result = restTemplate.postForObject(url, entity, String.class);
System.out.println(result);
```
##### 方式二：
```java
String url = "http://demo/api/book/";
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
MultiValueMap<String, String> map= new LinkedMultiValueMap<>();
map.add("email", "844072586@qq.com");

HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(map, headers);
ResponseEntity<String> response = restTemplate.postForEntity( url, request, String.class);
System.out.println(response.getBody());
```
#### 上传和下载
##### 上传
```java
String url = "http://restAddr/lyy-test/service/upload";
String filePath = "E:/fileServer/client/upload/upload.png";
FileSystemResource resource = new FileSystemResource(new File(filePath));
MultiValueMap<String, Object> param = new LinkedMultiValueMap<>();
param.add("file", resource);
param.add("fileName", "xiaolian.png");

String string = rest.postForObject(url, param, String.class);
```
##### 下载
```java
HttpHeaders headers = new HttpHeaders();
ResponseEntity<byte[]> response = restTemplate.exchange(url, HttpMethod.GET, new HttpEntity<byte[]>(headers), byte[].class);
byte[] result = response.getBody();
inputStream = new ByteArrayInputStream(result);

outputStream = new FileOutputStream(new File(filePath));

int len = 0;
byte[] buf = new byte[1024];
while ((len = inputStream.read(buf, 0, 1024)) != -1) {
	outputStream.write(buf, 0, len);
}
outputStream.flush();
```

## OkHttp3
#### 引入jar包
```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.14.2</version>
</dependency>
```
#### util工具类
```java
import lombok.extern.slf4j.Slf4j;
import okhttp3.*;

import javax.net.ssl.*;
import java.io.IOException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.TimeUnit;

@Slf4j
public class OkHttpUtils {
    public final static int READ_TIMEOUT = 100;
    public final static int CONNECT_TIMEOUT = 60;
    public final static int WRITE_TIMEOUT = 60;
    public static final MediaType JSON = MediaType.parse("application/json; charset=utf-8");
    private OkHttpClient mOkHttpClient;

    private OkHttpUtils() {
        okhttp3.OkHttpClient.Builder ClientBuilder = new okhttp3.OkHttpClient.Builder();
        ClientBuilder.readTimeout(READ_TIMEOUT, TimeUnit.SECONDS);//读取超时
        ClientBuilder.connectTimeout(CONNECT_TIMEOUT, TimeUnit.SECONDS);//连接超时
        ClientBuilder.writeTimeout(WRITE_TIMEOUT, TimeUnit.SECONDS);//写入超时
        //支持HTTPS请求，跳过证书验证
        ClientBuilder.sslSocketFactory(createSSLSocketFactory());
        ClientBuilder.hostnameVerifier((hostname, session) -> true);
        mOkHttpClient = ClientBuilder.build();
    }

    /**
     * get请求，同步方式，获取网络数据，是在主线程中执行的，需要新起线程，将其放到子线程中执行
     *
     * @param url
     * @return
     */
    public Response getData(String url) {
        //1 构造Request
        Request.Builder builder = new Request.Builder();
        Request request = builder.get().url(url).build();
        //2 将Request封装为Call
        Call call = mOkHttpClient.newCall(request);
        //3 执行Call，得到response
        Response response = null;
        try {
            response = call.execute();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return response;
    }

    /**
     * post请求，同步方式，提交数据，是在主线程中执行的，需要新起线程，将其放到子线程中执行
     *
     * @param url
     * @param bodyParams
     * @return
     */
    public Response postData(String url, Map<String, String> bodyParams) {
        //1构造RequestBody
        RequestBody body = setRequestBody(bodyParams);
        //2 构造Request
        Request.Builder requestBuilder = new Request.Builder();
        Request request = requestBuilder.post(body).url(url).build();
        //3 将Request封装为Call
        Call call = mOkHttpClient.newCall(request);
        //4 执行Call，得到response
        Response response = null;
        try {
            response = call.execute();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return response;
    }

    /**
     * get请求，异步方式，获取网络数据，是在子线程中执行的，需要切换到主线程才能更新UI
     *
     * @param url
     * @param myNetCall
     * @return
     */
    public void getDataAsyn(String url, final MyNetCall myNetCall) {
        //1 构造Request
        Request.Builder builder = new Request.Builder();
        Request request = builder.get().url(url).build();
        //2 将Request封装为Call
        Call call = mOkHttpClient.newCall(request);
        //3 执行Call
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                myNetCall.failed(call, e);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                myNetCall.success(call, response);

            }
        });
    }

    /**
     * post请求，异步方式，提交数据，是在子线程中执行的，需要切换到主线程才能更新UI
     *
     * @param url
     * @param bodyParams
     * @param myNetCall
     */
    public void postDataAsyn(String url, Map<String, String> bodyParams, final MyNetCall myNetCall) {
        //1构造RequestBody
        RequestBody body = setRequestBody(bodyParams);
        //2 构造Request
        Request.Builder requestBuilder = new Request.Builder();
        Request request = requestBuilder.post(body).url(url).build();
        //3 将Request封装为Call
        Call call = mOkHttpClient.newCall(request);
        //4 执行Call
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                myNetCall.failed(call, e);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                myNetCall.success(call, response);

            }
        });
    }

    /**
     * post的请求参数，构造RequestBody
     *
     * @param BodyParams
     * @return
     */
    private RequestBody setRequestBody(Map<String, String> BodyParams) {
        RequestBody body = null;
        okhttp3.FormBody.Builder formEncodingBuilder = new okhttp3.FormBody.Builder();
        if (BodyParams != null) {
            Iterator<String> iterator = BodyParams.keySet().iterator();
            String key = "";
            while (iterator.hasNext()) {
                key = iterator.next().toString();
                formEncodingBuilder.add(key, BodyParams.get(key));
                log.info("post http", "post_Params===" + key + "====" + BodyParams.get(key));
            }
        }
        body = formEncodingBuilder.build();
        return body;

    }

    /**
     * 生成安全套接字工厂，用于https请求的证书跳过
     *
     * @return
     */
    public SSLSocketFactory createSSLSocketFactory() {
        SSLSocketFactory ssfFactory = null;
        try {
            SSLContext sc = SSLContext.getInstance("TLS");
            sc.init(null, new TrustManager[]{new TrustAllCerts()}, new SecureRandom());
            ssfFactory = sc.getSocketFactory();
        } catch (Exception e) {
        }
        return ssfFactory;
    }

    public String postJson(String url, String json) throws IOException {
        RequestBody body = RequestBody.create(JSON, json);
        Request request = new Request.Builder()
                .url(url)
                .post(body)
                .build();
        Response response = mOkHttpClient.newCall(request).execute();
        if (response.isSuccessful()) {
            return response.body().string();
        } else {
            throw new IOException("Unexpected code " + response);
        }
    }

    public void postJsonAsyn(String url, String json, final MyNetCall myNetCall) throws IOException {
        RequestBody body = RequestBody.create(JSON, json);
        //2 构造Request
        Request.Builder requestBuilder = new Request.Builder();
        Request request = requestBuilder.post(body).url(url).build();
        //3 将Request封装为Call
        Call call = mOkHttpClient.newCall(request);
        //4 执行Call
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                myNetCall.failed(call, e);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                myNetCall.success(call, response);

            }
        });
    }

    /**
     * 自定义网络回调接口
     */
    public interface MyNetCall {
        void success(Call call, Response response) throws IOException;

        void failed(Call call, IOException e);
    }

    /**
     * 用于信任所有证书
     */
    class TrustAllCerts implements X509TrustManager {
        @Override
        public void checkClientTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {

        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    }
}
```