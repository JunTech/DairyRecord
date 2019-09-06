# HttpUtils-----httpclient爬虫

将httpclient爬虫功能封装在httpUtils类里面：

httpUtils.java

创建连接池管理器

```java
package top.juntech.autohome.utils;

import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HttpUtils {
//    编写连接池管理器
    public PoolingHttpClientConnectionManager pm(){
//        创建连接池管理器
        PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
//        设置最大连接数
        cm.setMaxTotal(200);
//        设置最大连接主机数
        cm.setDefaultMaxPerRoute(20);
//        关闭失效连接
        cm.closeExpiredConnections();
//        返回连接池
        return cm;
    }

}
```



ApiService.java

进行httpclient一些功能的封装:下载图片和解析html

```java
package top.juntech.autohome.service.impl;

import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import top.juntech.autohome.service.ServiceApi;
import top.juntech.autohome.utils.HttpUtils;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.UUID;

@Service
public class ApiServiceImpl implements ServiceApi {
    @Autowired
    private HttpUtils httpUtils;


    @Override
    public String getHtml(String uri) {
//        CloseableHttpClient client = HttpClients.createDefault();
        CloseableHttpClient client = HttpClients.custom().setConnectionManager(httpUtils.pm()).build();
        HttpGet httpGet = new HttpGet();
        httpGet.setHeader("User-Agen","Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.19 Safari/537.36");
        httpGet.setConfig(this.getRequestConfig());
        CloseableHttpResponse response = null;
        try {
            response = client.execute(httpGet);
            if(response.getStatusLine().getStatusCode() == 200){
                String html = "";
                if(response.getEntity() != null){
                    html = EntityUtils.toString(response.getEntity(), "utf8");
                    System.out.println(html);

                }
                return html;
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                //关闭resp
                response.close();

            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return null;
    }

//下载图片
    @Override
    public String getImg(String uri) {
        CloseableHttpClient client = HttpClients.custom().setConnectionManager(httpUtils.pm()).build();
        HttpGet httpGet = new HttpGet();
        httpGet.setHeader("User-Agen","Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.19 Safari/537.36");
        httpGet.setConfig(this.getRequestConfig());
        CloseableHttpResponse response = null;

        try {
            response = client.execute(httpGet);
            if (response.getStatusLine().getStatusCode() == 200) {
               String contentType = response.getEntity().getContentType().getValue();
               String extname = contentType.split("/")[1];
                //        使用UUID生成图片名
                UUID uuid = UUID.randomUUID();
                String imageName = uuid.toString() + extname;
                //声明输出的文件
                OutputStream outputStream = new FileOutputStream(new File("C:\\Users\\Ryan\\Downloads\\images\\"+imageName));
//                输出文件
                response.getEntity().writeTo(outputStream);
//                返回imageName
                return imageName;

            }

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                //关闭resp
                response.close();

            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return null;
    }

//超时配置
    public RequestConfig getRequestConfig(){
        RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(10000)
                .setConnectTimeout(500)
                .setConnectionRequestTimeout(1000).build();
        return requestConfig;
    }
}
```

