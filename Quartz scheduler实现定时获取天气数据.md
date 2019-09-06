# Quartz Scheduler实现定时获取天气数据

开发环境：springboot Quartz starter、quartz scheduler

## 1、实现数据同步

在上一节我们已经集成了redis，提升了天气服务的高并发能力，这一节我们基于上一节的代码进行2次开发，实现数据的同步

### 1.1 创建job包用于存放任务

引入quartz 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

创建WeatherDataSyncJob.java

继承QuartzJobBean

```java
package top.juntech.singleweatherforecastsys.job;

import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.quartz.QuartzJobBean;
//天气数据同步任务
public class WeatherDataSyncJob extends QuartzJobBean {
    private final static Logger logger = LoggerFactory.getLogger(WeatherDataSyncJob.class);
    @Override
    protected void executeInternal(org.quartz.JobExecutionContext jobExecutionContext) throws JobExecutionException {
        logger.info("启动定时任务.....");
    }
}
```

创建Quartzconfig

```java
package top.juntech.singleweatherforecastsys.config;

import org.quartz.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import top.juntech.singleweatherforecastsys.job.WeatherDataSyncJob;

@Configuration
public class QuartzConfig {
//    JobDetail
    @Bean
    public JobDetail weatherDataSyncJobJobDetail(){
        return JobBuilder.newJob(WeatherDataSyncJob.class).withIdentity("weatherDataSyncJob")
                .storeDurably().build();

    }

//    Trigger
    @Bean
    public Trigger weatherDataSyncTrigger(){
        SimpleScheduleBuilder simpleScheduleBuilder = SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(2).repeatForever();
        return TriggerBuilder.newTrigger().forJob(weatherDataSyncJobJobDetail())
                .withIdentity("trigger").withSchedule(simpleScheduleBuilder).build();
    }
}
```

启动main主程序，显示：

```java
2019-08-23 22:15:14.655  INFO 10896 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : 启动定时任务.....
2019-08-23 22:15:14.677  INFO 10896 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-08-23 22:15:14.679  INFO 10896 --- [  restartedMain] .j.s.SingleWeatherForecastSysApplication : Started SingleWeatherForecastSysApplication in 3.227 seconds (JVM running for 4.486)
2019-08-23 22:15:16.242  INFO 10896 --- [eduler_Worker-2] t.j.s.job.WeatherDataSyncJob             : 启动定时任务.....
2019-08-23 22:15:18.241  INFO 10896 --- [eduler_Worker-3] t.j.s.job.WeatherDataSyncJob             : 启动定时任务.....
2019-08-23 22:15:20.242  INFO 10896 --- [eduler_Worker-4] t.j.s.job.WeatherDataSyncJob             : 启动定时任务.....
2019-08-23 22:15:22.241  INFO 10896 --- [eduler_Worker-5] t.j.s.job.WeatherDataSyncJob             : 启动定时任务.....
```

每2s执行一次任务，

## 2、准备城市数据

获取cityList.xml文件，用于保存city数据。由于学习需要，此处就只拿广东省的一些城市作为样本数据进行实验。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c c1="0">
<d d1="101280101" d2="广州" d3="guangzhou" d4="广东"/>
<d d1="101280102" d2="番禺" d3="panyu" d4="广东"/>
<d d1="101280103" d2="从化" d3="conghua" d4="广东"/>
<d d1="101280104" d2="增城" d3="zengcheng" d4="广东"/>
<d d1="101280105" d2="花都" d3="huadu" d4="广东"/>
<d d1="101280201" d2="韶关" d3="shaoguan" d4="广东"/>
<d d1="101280202" d2="乳源" d3="ruyuan" d4="广东"/>
<d d1="101280203" d2="始兴" d3="shixing" d4="广东"/>
<d d1="101280204" d2="翁源" d3="wengyuan" d4="广东"/>
<d d1="101280205" d2="乐昌" d3="lechang" d4="广东"/>
<d d1="101280206" d2="仁化" d3="renhua" d4="广东"/>
<d d1="101280207" d2="南雄" d3="nanxiong" d4="广东"/>
<d d1="101280208" d2="新丰" d3="xinfeng" d4="广东"/>
<d d1="101280209" d2="曲江" d3="qujiang" d4="广东"/>
<d d1="101280210" d2="浈江" d3="chengjiang" d4="广东"/>
<d d1="101280211" d2="武江" d3="wujiang" d4="广东"/>
<d d1="101280301" d2="惠州" d3="huizhou" d4="广东"/>
<d d1="101280302" d2="博罗" d3="boluo" d4="广东"/>
<d d1="101280303" d2="惠阳" d3="huiyang" d4="广东"/>
<d d1="101280304" d2="惠东" d3="huidong" d4="广东"/>
<d d1="101280305" d2="龙门" d3="longmen" d4="广东"/>
<d d1="101280401" d2="梅州" d3="meizhou" d4="广东"/>
<d d1="101280402" d2="兴宁" d3="xingning" d4="广东"/>
<d d1="101280403" d2="蕉岭" d3="jiaoling" d4="广东"/>
<d d1="101280404" d2="大埔" d3="dabu" d4="广东"/>
<d d1="101280406" d2="丰顺" d3="fengshun" d4="广东"/>
<d d1="101280407" d2="平远" d3="pingyuan" d4="广东"/>
<d d1="101280408" d2="五华" d3="wuhua" d4="广东"/>
<d d1="101280409" d2="梅县" d3="meixian" d4="广东"/>
<d d1="101280501" d2="汕头" d3="shantou" d4="广东"/>
<d d1="101280502" d2="潮阳" d3="chaoyang" d4="广东"/>
<d d1="101280503" d2="澄海" d3="chenghai" d4="广东"/>
<d d1="101280504" d2="南澳" d3="nanao" d4="广东"/>
<d d1="101280601" d2="深圳" d3="shenzhen" d4="广东"/>
<d d1="101280701" d2="珠海" d3="zhuhai" d4="广东"/>
<d d1="101280702" d2="斗门" d3="doumen" d4="广东"/>
<d d1="101280703" d2="金湾" d3="jinwan" d4="广东"/>
<d d1="101280800" d2="佛山" d3="foshan" d4="广东"/>
<d d1="101280801" d2="顺德" d3="shunde" d4="广东"/>
<d d1="101280802" d2="三水" d3="sanshui" d4="广东"/>
<d d1="101280803" d2="南海" d3="nanhai" d4="广东"/>
<d d1="101280804" d2="高明" d3="gaoming" d4="广东"/>
<d d1="101280901" d2="肇庆" d3="zhaoqing" d4="广东"/>
<d d1="101280902" d2="广宁" d3="guangning" d4="广东"/>
<d d1="101280903" d2="四会" d3="sihui" d4="广东"/>
<d d1="101280905" d2="德庆" d3="deqing" d4="广东"/>
<d d1="101280906" d2="怀集" d3="huaiji" d4="广东"/>
<d d1="101280907" d2="封开" d3="fengkai" d4="广东"/>
<d d1="101280908" d2="高要" d3="gaoyao" d4="广东"/>
<d d1="101281001" d2="湛江" d3="zhanjiang" d4="广东"/>
<d d1="101281002" d2="吴川" d3="wuchuan" d4="广东"/>
<d d1="101281003" d2="雷州" d3="leizhou" d4="广东"/>
<d d1="101281004" d2="徐闻" d3="xuwen" d4="广东"/>
<d d1="101281005" d2="廉江" d3="lianjiang" d4="广东"/>
<d d1="101281006" d2="赤坎" d3="chikan" d4="广东"/>
<d d1="101281007" d2="遂溪" d3="suixi" d4="广东"/>
<d d1="101281008" d2="坡头" d3="potou" d4="广东"/>
<d d1="101281009" d2="霞山" d3="xiashan" d4="广东"/>
<d d1="101281010" d2="麻章" d3="mazhang" d4="广东"/>
<d d1="101281101" d2="江门" d3="jiangmen" d4="广东"/>
<d d1="101281103" d2="开平" d3="kaiping" d4="广东"/>
<d d1="101281104" d2="新会" d3="xinhui" d4="广东"/>
<d d1="101281105" d2="恩平" d3="enping" d4="广东"/>
<d d1="101281106" d2="台山" d3="taishan" d4="广东"/>
<d d1="101281107" d2="蓬江" d3="pengjiang" d4="广东"/>
<d d1="101281108" d2="鹤山" d3="heshan" d4="广东"/>
<d d1="101281109" d2="江海" d3="jianghai" d4="广东"/>
<d d1="101281201" d2="河源" d3="heyuan" d4="广东"/>
<d d1="101281202" d2="紫金" d3="zijin" d4="广东"/>
<d d1="101281203" d2="连平" d3="lianping" d4="广东"/>
<d d1="101281204" d2="和平" d3="heping" d4="广东"/>
<d d1="101281205" d2="龙川" d3="longchuan" d4="广东"/>
<d d1="101281206" d2="东源" d3="dongyuan" d4="广东"/>
<d d1="101281301" d2="清远" d3="qingyuan" d4="广东"/>
<d d1="101281302" d2="连南" d3="liannan" d4="广东"/>
<d d1="101281303" d2="连州" d3="lianzhou" d4="广东"/>
<d d1="101281304" d2="连山" d3="lianshan" d4="广东"/>
<d d1="101281305" d2="阳山" d3="yangshan" d4="广东"/>
<d d1="101281306" d2="佛冈" d3="fogang" d4="广东"/>
<d d1="101281307" d2="英德" d3="yingde" d4="广东"/>
<d d1="101281308" d2="清新" d3="qingxin" d4="广东"/>
<d d1="101281401" d2="云浮" d3="yunfu" d4="广东"/>
<d d1="101281402" d2="罗定" d3="luoding" d4="广东"/>
<d d1="101281403" d2="新兴" d3="xinxing" d4="广东"/>
<d d1="101281404" d2="郁南" d3="yunan" d4="广东"/>
<d d1="101281406" d2="云安" d3="yunan" d4="广东"/>
<d d1="101281501" d2="潮州" d3="chaozhou" d4="广东"/>
<d d1="101281502" d2="饶平" d3="raoping" d4="广东"/>
<d d1="101281503" d2="潮安" d3="chaoan" d4="广东"/>
<d d1="101281601" d2="东莞" d3="dongguan" d4="广东"/>
<d d1="101281701" d2="中山" d3="zhongshan" d4="广东"/>
<d d1="101281801" d2="阳江" d3="yangjiang" d4="广东"/>
<d d1="101281802" d2="阳春" d3="yangchun" d4="广东"/>
<d d1="101281803" d2="阳东" d3="yangdong" d4="广东"/>
<d d1="101281804" d2="阳西" d3="yangxi" d4="广东"/>
<d d1="101281901" d2="揭阳" d3="jieyang" d4="广东"/>
<d d1="101281902" d2="揭西" d3="jiexi" d4="广东"/>
<d d1="101281903" d2="普宁" d3="puning" d4="广东"/>
<d d1="101281904" d2="惠来" d3="huilai" d4="广东"/>
<d d1="101281905" d2="揭东" d3="jiedong" d4="广东"/>
<d d1="101282001" d2="茂名" d3="maoming" d4="广东"/>
<d d1="101282002" d2="高州" d3="gaozhou" d4="广东"/>
<d d1="101282003" d2="化州" d3="huazhou" d4="广东"/>
<d d1="101282004" d2="电白" d3="dianbai" d4="广东"/>
<d d1="101282005" d2="信宜" d3="xinyi" d4="广东"/>
<d d1="101282006" d2="茂港" d3="maogang" d4="广东"/>
<d d1="101282101" d2="汕尾" d3="shanwei" d4="广东"/>
<d d1="101282102" d2="海丰" d3="haifeng" d4="广东"/>
<d d1="101282103" d2="陆丰" d3="lufeng" d4="广东"/>
<d d1="101282104" d2="陆河" d3="luhe" d4="广东"/>
</c>
```

将xml文件保存在resource文件夹下，接下来，读取城市列表及id.

创建一个实体类接收xml读取出来的数据：city.java

将xml解析映射到javabean的过程如下：

由于要从xml文件中读取数据，则需要用到工具类javax.xml.*,这个包是java自带的。

```java
package top.juntech.singleweatherforecastsys.vo;

import lombok.Data;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlRootElement;
import java.io.Serializable;

@Data
@XmlRootElement(name = "d")
public class City implements Serializable {
    @XmlAttribute(name = "d1")
    private String cityId;
    @XmlAttribute(name = "d2")
    private String cityName;
    @XmlAttribute(name = "d3")
    private String cityCode;
    @XmlAttribute(name = "d4")
    private String province;
}

```

创建cityList来存储city集合：

```java
package top.juntech.singleweatherforecastsys.vo;

import lombok.Data;

import javax.xml.bind.annotation.*;
import java.io.Serializable;
import java.util.List;

@Data
@XmlRootElement(name = "c")
@XmlAccessorType(XmlAccessType.FIELD)
public class CityList implements Serializable {
    @XmlElement(name = "d")
    private List<City> cityList;
}

```

创建工具类XmlBuilder

作用：将xml数据转化为对象

```java
package top.juntech.singleweatherforecastsys.utils;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.Unmarshaller;
import java.io.Reader;
import java.io.StringReader;

public class XmlBuilder {

    public static Object xmlStrToObject(Class<?> clazz, String xmlStr) throws Exception {
        Object xmlObject = null;
        Reader reader = null;

        JAXBContext context = JAXBContext.newInstance(clazz);
        Unmarshaller unmarshaller = context.createUnmarshaller();

        reader = new StringReader(xmlStr);
        xmlObject = unmarshaller.unmarshal(reader);

        if (null != reader) {
            reader.close();
        }

        return xmlObject;
    }
}
```

## 3、编写City的service

```java
package top.juntech.singleweatherforecastsys.service;

import top.juntech.singleweatherforecastsys.vo.City;

import java.util.List;

public interface CityDataService {
    public List<City> listCity() throws  Exception;
}
```

定义listcity接口用来显示city 集合

```java
package top.juntech.singleweatherforecastsys.service.impl;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;
import top.juntech.singleweatherforecastsys.service.CityDataService;
import top.juntech.singleweatherforecastsys.utils.XmlBuilder;
import top.juntech.singleweatherforecastsys.vo.City;
import top.juntech.singleweatherforecastsys.vo.CityList;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.List;

@Service
public class CityDataServiceImpl implements CityDataService {
    @Override
    public List<City> listCity() throws Exception {
//        读取xml文件
        Resource resource = new ClassPathResource("citylist.xml");
        BufferedReader bf = new BufferedReader(new InputStreamReader(((ClassPathResource) resource).getInputStream(),"utf-8"));
        StringBuffer sb = new StringBuffer();
        String line = "";
        while((line = bf.readLine()) != null){
            sb.append(line);
        }
        bf.close();

        //        xml转为java对象
        CityList cityList = (CityList) XmlBuilder.xmlStrToObject(CityList.class, sb.toString());
        System.out.println(cityList);
        return cityList.getCityList();
    }
}

```

读取xml文件属性映射到javabean

在weatherDataService接口文件中加入方法：

```java
//    根据城市id来同步数据
    void syncDateByCityId(String cityId);
```

实现：

```
    private  void saveWeatherData(String uri){
        String key = uri;
        String strBody = null;

        ObjectMapper objectMapper =  new ObjectMapper();
        objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        WeatherResponse weatherResponse = null;
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();

            ResponseEntity<String> respString = restTemplate.getForEntity(uri, String.class);
            if (respString.getStatusCodeValue() == 200) {
                strBody = respString.getBody();
            }
//                将数据写入缓存
            ops.set(key, strBody, TIME_OUT, TimeUnit.SECONDS);


    }
}
```

重写job任务：定时任务

```java
package top.juntech.singleweatherforecastsys.job;

import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;
import top.juntech.singleweatherforecastsys.service.CityDataService;
import top.juntech.singleweatherforecastsys.service.WeatherDataService;
import top.juntech.singleweatherforecastsys.vo.City;

import java.util.List;

//天气数据同步任务
public class WeatherDataSyncJob extends QuartzJobBean {
    private final static Logger logger = LoggerFactory.getLogger(WeatherDataSyncJob.class);
    @Autowired
    private CityDataService cityDataService;

    @Autowired
    private WeatherDataService weatherDataService;

    @Override
    protected void executeInternal(org.quartz.JobExecutionContext jobExecutionContext) throws JobExecutionException {
        logger.info("启动定时任务.....");
//        获取城市id列表
        List<City> cityList = null;

        try{
            cityList = cityDataService.listCity();
        }catch (Exception e){
            logger.error("出错啦");
//            e.printStackTrace();
        }
//        便利城市id获取天气
        for (City city: cityList) {
            String cityId = city.getCityId();
            logger.info("weather data sync job,cityId : "+cityId);
            weatherDataService.syncDateByCityId(cityId);
        }
        logger.info("结束定时任务.....");
    }
}
```

config：

```java
package top.juntech.singleweatherforecastsys.config;

import org.quartz.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import top.juntech.singleweatherforecastsys.job.WeatherDataSyncJob;

@Configuration
public class QuartzConfig {
//    JobDetail
    @Bean
    public JobDetail weatherDataSyncJobJobDetail(){
        return JobBuilder.newJob(WeatherDataSyncJob.class).withIdentity("weatherDataSyncJob")
                .storeDurably().build();

    }

//    Trigger
    @Bean
    public Trigger weatherDataSyncTrigger(){
//        半小时拉取一次数据服务
        SimpleScheduleBuilder simpleScheduleBuilder = SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(1800).repeatForever();
        return TriggerBuilder.newTrigger().forJob(weatherDataSyncJobJobDetail())
                .withIdentity("trigger").withSchedule(simpleScheduleBuilder).build();
    }
}
```

设置为半小时拉取一次天气预报接口的数据服务。

ok，到此就完成了定时任务的编写，并存入redis.

## 4、运行程序

运行主程序，控制台显示如下：

```json

2019-08-23 23:21:14.693  INFO 10696 --- [  restartedMain] org.quartz.impl.StdSchedulerFactory      : Quartz scheduler 'quartzScheduler' initialized from an externally provided properties instance.
2019-08-23 23:21:14.693  INFO 10696 --- [  restartedMain] org.quartz.impl.StdSchedulerFactory      : Quartz scheduler version: 2.3.1
2019-08-23 23:21:14.693  INFO 10696 --- [  restartedMain] org.quartz.core.QuartzScheduler          : JobFactory set to: org.springframework.scheduling.quartz.SpringBeanJobFactory@51a42b8f
2019-08-23 23:21:14.731  INFO 10696 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-08-23 23:21:14.750  INFO 10696 --- [  restartedMain] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now
2019-08-23 23:21:14.750  INFO 10696 --- [  restartedMain] org.quartz.core.QuartzScheduler          : Scheduler quartzScheduler_$_NON_CLUSTERED started.
2019-08-23 23:21:14.758  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : 启动定时任务.....
2019-08-23 23:21:14.779  INFO 10696 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-08-23 23:21:14.782  INFO 10696 --- [  restartedMain] .j.s.SingleWeatherForecastSysApplication : Started SingleWeatherForecastSysApplication in 3.313 seconds (JVM running for 4.563)
CityList(cityList=[City(cityId=101280101, cityName=广州, cityCode=guangzhou, province=广东), City(cityId=101280102, cityName=番禺, cityCode=panyu, province=广东), City(cityId=101280103, cityName=从化, cityCode=conghua, province=广东), City(cityId=101280104, cityName=增城, cityCode=zengcheng, province=广东), City(cityId=101280105, cityName=花都, cityCode=huadu, province=广东), City(cityId=101280201, cityName=韶关, cityCode=shaoguan, province=广东), City(cityId=101280202, cityName=乳源, cityCode=ruyuan, province=广东), City(cityId=101280203, cityName=始兴, cityCode=shixing, province=广东), City(cityId=101280204, cityName=翁源, cityCode=wengyuan, province=广东), City(cityId=101280205, cityName=乐昌, cityCode=lechang, province=广东), City(cityId=101280206, cityName=仁化, cityCode=renhua, province=广东), City(cityId=101280207, cityName=南雄, cityCode=nanxiong, province=广东), City(cityId=101280208, cityName=新丰, cityCode=xinfeng, province=广东), City(cityId=101280209, cityName=曲江, cityCode=qujiang, province=广东), City(cityId=101280210, cityName=浈江, cityCode=chengjiang, province=广东), City(cityId=101280211, cityName=武江, cityCode=wujiang, province=广东), City(cityId=101280301, cityName=惠州, cityCode=huizhou, province=广东), City(cityId=101280302, cityName=博罗, cityCode=boluo, province=广东), City(cityId=101280303, cityName=惠阳, cityCode=huiyang, province=广东), City(cityId=101280304, cityName=惠东, cityCode=huidong, province=广东), City(cityId=101280305, cityName=龙门, cityCode=longmen, province=广东), City(cityId=101280401, cityName=梅州, cityCode=meizhou, province=广东), City(cityId=101280402, cityName=兴宁, cityCode=xingning, province=广东), City(cityId=101280403, cityName=蕉岭, cityCode=jiaoling, province=广东), City(cityId=101280404, cityName=大埔, cityCode=dabu, province=广东), City(cityId=101280406, cityName=丰顺, cityCode=fengshun, province=广东), City(cityId=101280407, cityName=平远, cityCode=pingyuan, province=广东), City(cityId=101280408, cityName=五华, cityCode=wuhua, province=广东), City(cityId=101280409, cityName=梅县, cityCode=meixian, province=广东), City(cityId=101280501, cityName=汕头, cityCode=shantou, province=广东), City(cityId=101280502, cityName=潮阳, cityCode=chaoyang, province=广东), City(cityId=101280503, cityName=澄海, cityCode=chenghai, province=广东), City(cityId=101280504, cityName=南澳, cityCode=nanao, province=广东), City(cityId=101280601, cityName=深圳, cityCode=shenzhen, province=广东), City(cityId=101280701, cityName=珠海, cityCode=zhuhai, province=广东), City(cityId=101280702, cityName=斗门, cityCode=doumen, province=广东), City(cityId=101280703, cityName=金湾, cityCode=jinwan, province=广东), City(cityId=101280800, cityName=佛山, cityCode=foshan, province=广东), City(cityId=101280801, cityName=顺德, cityCode=shunde, province=广东), City(cityId=101280802, cityName=三水, cityCode=sanshui, province=广东), City(cityId=101280803, cityName=南海, cityCode=nanhai, province=广东), City(cityId=101280804, cityName=高明, cityCode=gaoming, province=广东), City(cityId=101280901, cityName=肇庆, cityCode=zhaoqing, province=广东), City(cityId=101280902, cityName=广宁, cityCode=guangning, province=广东), City(cityId=101280903, cityName=四会, cityCode=sihui, province=广东), City(cityId=101280905, cityName=德庆, cityCode=deqing, province=广东), City(cityId=101280906, cityName=怀集, cityCode=huaiji, province=广东), City(cityId=101280907, cityName=封开, cityCode=fengkai, province=广东), City(cityId=101280908, cityName=高要, cityCode=gaoyao, province=广东), City(cityId=101281001, cityName=湛江, cityCode=zhanjiang, province=广东), City(cityId=101281002, cityName=吴川, cityCode=wuchuan, province=广东), City(cityId=101281003, cityName=雷州, cityCode=leizhou, province=广东), City(cityId=101281004, cityName=徐闻, cityCode=xuwen, province=广东), City(cityId=101281005, cityName=廉江, cityCode=lianjiang, province=广东), City(cityId=101281006, cityName=赤坎, cityCode=chikan, province=广东), City(cityId=101281007, cityName=遂溪, cityCode=suixi, province=广东), City(cityId=101281008, cityName=坡头, cityCode=potou, province=广东), City(cityId=101281009, cityName=霞山, cityCode=xiashan, province=广东), City(cityId=101281010, cityName=麻章, cityCode=mazhang, province=广东), City(cityId=101281101, cityName=江门, cityCode=jiangmen, province=广东), City(cityId=101281103, cityName=开平, cityCode=kaiping, province=广东), City(cityId=101281104, cityName=新会, cityCode=xinhui, province=广东), City(cityId=101281105, cityName=恩平, cityCode=enping, province=广东), City(cityId=101281106, cityName=台山, cityCode=taishan, province=广东), City(cityId=101281107, cityName=蓬江, cityCode=pengjiang, province=广东), City(cityId=101281108, cityName=鹤山, cityCode=heshan, province=广东), City(cityId=101281109, cityName=江海, cityCode=jianghai, province=广东), City(cityId=101281201, cityName=河源, cityCode=heyuan, province=广东), City(cityId=101281202, cityName=紫金, cityCode=zijin, province=广东), City(cityId=101281203, cityName=连平, cityCode=lianping, province=广东), City(cityId=101281204, cityName=和平, cityCode=heping, province=广东), City(cityId=101281205, cityName=龙川, cityCode=longchuan, province=广东), City(cityId=101281206, cityName=东源, cityCode=dongyuan, province=广东), City(cityId=101281301, cityName=清远, cityCode=qingyuan, province=广东), City(cityId=101281302, cityName=连南, cityCode=liannan, province=广东), City(cityId=101281303, cityName=连州, cityCode=lianzhou, province=广东), City(cityId=101281304, cityName=连山, cityCode=lianshan, province=广东), City(cityId=101281305, cityName=阳山, cityCode=yangshan, province=广东), City(cityId=101281306, cityName=佛冈, cityCode=fogang, province=广东), City(cityId=101281307, cityName=英德, cityCode=yingde, province=广东), City(cityId=101281308, cityName=清新, cityCode=qingxin, province=广东), City(cityId=101281401, cityName=云浮, cityCode=yunfu, province=广东), City(cityId=101281402, cityName=罗定, cityCode=luoding, province=广东), City(cityId=101281403, cityName=新兴, cityCode=xinxing, province=广东), City(cityId=101281404, cityName=郁南, cityCode=yunan, province=广东), City(cityId=101281406, cityName=云安, cityCode=yunan, province=广东), City(cityId=101281501, cityName=潮州, cityCode=chaozhou, province=广东), City(cityId=101281502, cityName=饶平, cityCode=raoping, province=广东), City(cityId=101281503, cityName=潮安, cityCode=chaoan, province=广东), City(cityId=101281601, cityName=东莞, cityCode=dongguan, province=广东), City(cityId=101281701, cityName=中山, cityCode=zhongshan, province=广东), City(cityId=101281801, cityName=阳江, cityCode=yangjiang, province=广东), City(cityId=101281802, cityName=阳春, cityCode=yangchun, province=广东), City(cityId=101281803, cityName=阳东, cityCode=yangdong, province=广东), City(cityId=101281804, cityName=阳西, cityCode=yangxi, province=广东), City(cityId=101281901, cityName=揭阳, cityCode=jieyang, province=广东), City(cityId=101281902, cityName=揭西, cityCode=jiexi, province=广东), City(cityId=101281903, cityName=普宁, cityCode=puning, province=广东), City(cityId=101281904, cityName=惠来, cityCode=huilai, province=广东), City(cityId=101281905, cityName=揭东, cityCode=jiedong, province=广东), City(cityId=101282001, cityName=茂名, cityCode=maoming, province=广东), City(cityId=101282002, cityName=高州, cityCode=gaozhou, province=广东), City(cityId=101282003, cityName=化州, cityCode=huazhou, province=广东), City(cityId=101282004, cityName=电白, cityCode=dianbai, province=广东), City(cityId=101282005, cityName=信宜, cityCode=xinyi, province=广东), City(cityId=101282006, cityName=茂港, cityCode=maogang, province=广东), City(cityId=101282101, cityName=汕尾, cityCode=shanwei, province=广东), City(cityId=101282102, cityName=海丰, cityCode=haifeng, province=广东), City(cityId=101282103, cityName=陆丰, cityCode=lufeng, province=广东), City(cityId=101282104, cityName=陆河, cityCode=luhe, province=广东)])
2019-08-23 23:21:14.814  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280101
2019-08-23 23:21:15.266  INFO 10696 --- [eduler_Worker-1] io.lettuce.core.EpollProvider            : Starting without optional epoll library
2019-08-23 23:21:15.267  INFO 10696 --- [eduler_Worker-1] io.lettuce.core.KqueueProvider           : Starting without optional kqueue library
2019-08-23 23:21:16.177  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280102
2019-08-23 23:21:16.226  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280103
2019-08-23 23:21:16.311  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280104
2019-08-23 23:21:16.407  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280105
2019-08-23 23:21:16.515  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280201
2019-08-23 23:21:16.546  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280202
2019-08-23 23:21:16.625  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280203
2019-08-23 23:21:16.687  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280204
2019-08-23 23:21:16.787  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280205
2019-08-23 23:21:16.877  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280206
2019-08-23 23:21:16.940  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280207
2019-08-23 23:21:17.001  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280208
2019-08-23 23:21:17.094  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280209
2019-08-23 23:21:17.189  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280210
2019-08-23 23:21:17.277  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280211
2019-08-23 23:21:17.337  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280301
2019-08-23 23:21:17.363  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280302
2019-08-23 23:21:17.448  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280303
2019-08-23 23:21:17.551  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280304
2019-08-23 23:21:17.660  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280305
2019-08-23 23:21:17.781  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280401
2019-08-23 23:21:17.841  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280402
2019-08-23 23:21:17.909  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280403
2019-08-23 23:21:17.970  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280404
2019-08-23 23:21:18.062  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280406
2019-08-23 23:21:18.138  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280407
2019-08-23 23:21:18.200  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280408
2019-08-23 23:21:18.291  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280409
2019-08-23 23:21:18.389  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280501
2019-08-23 23:21:18.414  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280502
2019-08-23 23:21:18.504  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280503
2019-08-23 23:21:18.565  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280504
2019-08-23 23:21:18.639  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280601
2019-08-23 23:21:18.650  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280701
2019-08-23 23:21:18.691  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280702
2019-08-23 23:21:18.758  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280703
2019-08-23 23:21:18.821  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280800
2019-08-23 23:21:18.861  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280801
2019-08-23 23:21:18.920  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280802
2019-08-23 23:21:19.008  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280803
2019-08-23 23:21:19.034  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280804
2019-08-23 23:21:19.106  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280901
2019-08-23 23:21:19.130  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280902
2019-08-23 23:21:19.196  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280903
2019-08-23 23:21:19.274  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280905
2019-08-23 23:21:19.355  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280906
2019-08-23 23:21:19.574  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280907
2019-08-23 23:21:19.629  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101280908
2019-08-23 23:21:19.728  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281001
2019-08-23 23:21:19.792  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281002
2019-08-23 23:21:19.867  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281003
2019-08-23 23:21:19.962  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281004
2019-08-23 23:21:20.046  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281005
2019-08-23 23:21:20.118  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281006
2019-08-23 23:21:20.319  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281007
2019-08-23 23:21:20.382  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281008
2019-08-23 23:21:20.476  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281009
2019-08-23 23:21:20.569  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281010
2019-08-23 23:21:20.639  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281101
2019-08-23 23:21:20.679  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281103
2019-08-23 23:21:20.742  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281104
2019-08-23 23:21:20.805  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281105
2019-08-23 23:21:20.884  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281106
2019-08-23 23:21:20.947  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281107
2019-08-23 23:21:21.046  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281108
2019-08-23 23:21:21.127  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281109
2019-08-23 23:21:21.220  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281201
2019-08-23 23:21:21.301  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281202
2019-08-23 23:21:21.396  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281203
2019-08-23 23:21:21.454  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281204
2019-08-23 23:21:21.548  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281205
2019-08-23 23:21:21.612  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281206
2019-08-23 23:21:21.677  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281301
2019-08-23 23:21:21.703  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281302
2019-08-23 23:21:21.766  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281303
2019-08-23 23:21:21.853  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281304
2019-08-23 23:21:21.948  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281305
2019-08-23 23:21:22.082  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281306
2019-08-23 23:21:22.280  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281307
2019-08-23 23:21:22.664  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281308
2019-08-23 23:21:22.752  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281401
2019-08-23 23:21:22.778  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281402
2019-08-23 23:21:22.839  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281403
2019-08-23 23:21:22.894  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281404
2019-08-23 23:21:22.985  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281406
2019-08-23 23:21:23.150  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281501
2019-08-23 23:21:23.179  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281502
2019-08-23 23:21:23.261  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281503
2019-08-23 23:21:23.337  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281601
2019-08-23 23:21:23.362  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281701
2019-08-23 23:21:23.451  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281801
2019-08-23 23:21:23.478  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281802
2019-08-23 23:21:23.591  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281803
2019-08-23 23:21:23.674  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281804
2019-08-23 23:21:23.733  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281901
2019-08-23 23:21:23.799  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281902
2019-08-23 23:21:23.882  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281903
2019-08-23 23:21:23.943  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281904
2019-08-23 23:21:24.002  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101281905
2019-08-23 23:21:24.072  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282001
2019-08-23 23:21:24.097  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282002
2019-08-23 23:21:24.159  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282003
2019-08-23 23:21:24.222  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282004
2019-08-23 23:21:24.629  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282005
2019-08-23 23:21:24.695  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282006
2019-08-23 23:21:24.785  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282101
2019-08-23 23:21:24.889  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282102
2019-08-23 23:21:24.979  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282103
2019-08-23 23:21:25.044  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : weather data sync job,cityId : 101282104
2019-08-23 23:21:25.192  INFO 10696 --- [eduler_Worker-1] t.j.s.job.WeatherDataSyncJob             : 结束定时任务.....

```

至此，就将数据存入至redis数据库中了，可大大提升访问该程序的并发能力及访问速度。

请放心食用！

## 5、congratulations

祝贺你学到了新知识，也感谢你观看本文！