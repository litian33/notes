#内容概要
记录ElasticSearch中设置密码认证以及使用SSL证书的方式，以及趟过的坑。

## 开启用户名密码认证
> 6.x版本默认集成了x-pack工具，使用此工具实现认证
> 
### 开启方式
首先，需要开启x-pack：
```
curl -H "Content-Type:application/json" -XPOST  http://127.0.0.1:9200/_xpack/license/start_trial?acknowledge=true
```

然后，查看es控制台，看看是否有启用成功的信息；

之后，还需要修改es配置文件elasticsearch.yml，在最后增加如下内容：
```
 xpack.security.enabled: true
```

然后，重启es


### 修改用户信息
使用下面命令设置密码：

```
bin/elasticsearch-setup-passwords interactive
```

因为这是一个集成包，所以需要设置一堆用户的密码信息，即使使用不到。建议可以通过以下命令，单独修改一个用户的密码：
```
curl -H "Content-Type:application/json" -XPOST -u elastic 'http://127.0.0.1:9200/_xpack/security/user/elastic/_password' -d '{ "password" : "mypassword" }' 
```

### 启用head带密码访问（不安全，不建议使用）
修改elasticsearch.yml文件，增加如下信息：
```
 http.cors.enabled: true
 http.cors.allow-origin: "*"
 http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
```

然后，就可以在访问URL中携带用户名和密码信息：
```
http://127.0.0.1:9200/?auth_user=elastic&auth_password=mypassword
```


### Java Client使用方式
```java
final CredentialsProvider credentialsProvider =
    new BasicCredentialsProvider();
credentialsProvider.setCredentials(AuthScope.ANY,
    new UsernamePasswordCredentials("user", "password"));

RestClientBuilder builder = RestClient.builder(
    new HttpHost("localhost", 9200))
    .setHttpClientConfigCallback(new HttpClientConfigCallback() {
        @Override
        public HttpAsyncClientBuilder customizeHttpClient(
                HttpAsyncClientBuilder httpClientBuilder) {
            return httpClientBuilder
                .setDefaultCredentialsProvider(credentialsProvider);
        }
    });
```


## 开启SSL

### 生成证书

首先，生成CA证书：
```
bin/elasticsearch-certutil ca
```

然后，生成认证证书：
```
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --name localhost --ip 127.0.0.1
```
> 上面的命令行中的name和ip参数可选，它用于指定证书中的主机验证信息，在本地测试环境最好指定，以免踩坑。

之后，将生成的证书文件xxx.p12放在指定路径，如config/certs/xxx.p12

### SSL配置

修改elasticsearch.yml文件，增加证书信息配置：
```ini
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/xxx.p12
xpack.security.http.ssl.truststore.path: certs/xxx.p12

# 指定服务端验证方式，默认为full
xpack.ssl.verification_mode: certificate
```

如果keyStore设置了访问密码，则需要在ES中更新密码信息：
```
bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

然后，重启es

### Java Client使用方式

```java
SSLContext sslContext = SSLContexts.custom()
                                    .loadTrustMaterial(new File(path), "password".toCharArray())
                                    .build();
                            httpClientBuilder.setSSLContext(sslContext);
```

> 需要注意的是，访问时不要使用localhost:9200的方式，而是使用127.0.0.1:9200，否则会出现证书检查不通过的情况。

