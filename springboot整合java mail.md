# springboot整合java mail

## 1.准备工作

登录进qq邮箱，进入设置

![](https://img.vim-cn.com/55/b0625dfcbd28938c525b3aa771d188381fa885.png)

接下来打开

![](https://img.vim-cn.com/a7/e57a4120d967dae139aa4afce879755c4c89cb.png)

打开设置：

![](https://img.vim-cn.com/46/25bebf6c6698ac185ee5a31518e5aaec4c96fa.png)

记录下授权码，有用的

## 2.进行开发

### 2.1环境搭建

pom.xml

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
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
```

test--->用于单元测试

springbootmailstater--->用于邮件发送服务



applicaiton.properties

```
server.port=8080

#mail
spring.mail.host=smtp.qq.com
spring.mail.port=465
spring.mail.username=859240195@qq.com
spring.mail.protocol=smtp
spring.mail.password=你的授权码
spring.mail.default-encoding=utf-8
spring.mail.properties.mail.debug=true
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
```

mail的基本配置，没什么好说的，把授权码放在password那里，就可以了

在这里port可以设置为465或者587，都可以

### 2.2 发送简单邮件

```
package top.juntech.springbootmail.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/13 13:40
 * @ClassName 类名
 * @Descripe 发送简单邮件
 */
@Service
public class MailService {
    @Autowired
    JavaMailSender javaMailSender;

    public void sendSimpleMail(String from,String to,String cc,String subject,String content){
        /*
        * 邮件信息
        * */
        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
        simpleMailMessage.setFrom(from);
        simpleMailMessage.setTo(to);
        simpleMailMessage.setCc(cc);
        simpleMailMessage.setSubject(subject);
        simpleMailMessage.setText(content);
        /*
        * 发送邮件
        * */
        javaMailSender.send(simpleMailMessage);
    }
}

```

代码解释：

​	javamailsender是springboot在mailsenderpropertiesConfiguration类中配置好的

​	sendSimplemail的5个参数分别是：发送者，接收者，抄送这，邮件主题，邮件内容

​	简单邮件可以直接构建一个simplemailmessage对象进行配置，配置完成后通过javamailsender发送出去

### 2.3单元测试

```
package top.juntech.springbootmail;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import top.juntech.springbootmail.service.MailService;

@SpringBootTest
class SpringbootMailApplicationTests {
    @Autowired
    private MailService mailService;

    @Test
    void contextLoads() {
        mailService.sendSimpleMail(
                "859240195@qq.com",
                "zhoujay1997@gmail.com",
                "859240195@qq.com",
                "测试邮件主题",
                "测试邮件内容"
        );
    }

}

```

运行单元测试，打开控制台，打印日志

```
RCPT TO:<zhoujay1997@gmail.com>
250 Ok
RCPT TO:<859240195@qq.com>
250 Ok
DEBUG SMTP: Verified Addresses
DEBUG SMTP:   zhoujay1997@gmail.com
DEBUG SMTP:   859240195@qq.com
DATA
354 End data with <CR><LF>.<CR><LF>
Date: Fri, 13 Dec 2019 14:03:21 +0800 (CST)
From: 859240195@qq.com
To: zhoujay1997@gmail.com
Cc: 859240195@qq.com
Message-ID: <66845334.0.1576217001699@Ryan>
Subject: =?UTF-8?B?5rWL6K+V6YKu5Lu25Li76aKY?=
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: base64

5rWL6K+V6YKu5Lu25YaF5a65
.
250 Ok: queued as 
DEBUG SMTP: message successfully delivered to mail server
QUIT
```

在查看邮箱：

![](https://img.vim-cn.com/96/8923f82ea33d283c81eb753eaed3c2fd9fa899.png)

### 2.4发送带附件的邮件

```
    /*
    * 发送带附件的邮件
    * */
    public void sendAttacheFileMail(String from, String to, String cc, String subject, String content, File file){
        try{
            MimeMessage mimeMessage = javaMailSender.createMimeMessage();
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper();
            mimeMessageHelper.setFrom(from);
            mimeMessageHelper.setTo(to);
            mimeMessageHelper.setText(content);
            mimeMessageHelper.setCc(cc);
            mimeMessageHelper.setSubject(subject);
            mimeMessageHelper.addAttachment(file.getName(),file);
            javaMailSender.send(mimeMessage);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

加上这段代码就可以发送带附件的邮件 了