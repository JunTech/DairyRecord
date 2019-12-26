# 使用Jwt实现token认证

## 1.准备环境

pom.xml依赖

```
 <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
```

## 2.编写工具类

JwtUtils.java

```
package top.juntech.utils;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;
import top.juntech.bean.User;

import java.util.Date;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/18 17:11
 * @ClassName 类名
 * @Descripe 生成token,和解析token
 */
@Component
public class JwtUtils {

    public final static String SUBJECT = "juntech_class";
    public static final long EXPIRE = 1000*60*60*24*7;
    public static final String APPSECRET = "juntech";


    /**
     * @param user
     * @return String
     * @Descript 生成token
     */
    public static String generateJsonWebToken(User user){
        if(user ==null || user.getId()==null||user.getName()==null||user.getHeadImg()==null){
            return null;
        }
        String token =Jwts.builder().setSubject(SUBJECT)
                .claim("id",user.getId())
                .claim("name",user.getName())
                .claim("img",user.getHeadImg())
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis()+EXPIRE))
                .signWith(SignatureAlgorithm.HS256,APPSECRET).compact();

        return token;
    }


    /**
     * 校验token
     * @param token
     * @return
     */
    public static Claims checkJwt(String token){
        try {
            final Claims body = Jwts.parser().setSigningKey(APPSECRET).parseClaimsJws(token).getBody();
            return body;
        }catch (Exception e){
            return null;
        }

    }
}

```

## 3.编写测试类

```
package top.juntech;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import top.juntech.bean.User;
import top.juntech.utils.JwtUtils;

@SpringBootTest
class JuntechClassApplicationTests {

    @Test
    void contextLoads() {
    }

    @Test
    void testJwt(){
        User user = new User();
        user.setId(999);
        user.setName("ryan");
        user.setHeadImg("www.juntech.top");

        String s = JwtUtils.generateJsonWebToken(user);
        System.out.println("token = "+s);
    }

    @Test
    void testCheck(){
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqdW50ZWNoX2NsYXNzIiwiaWQiOjk5OSwibmFtZSI6InJ5YW4iLCJpbWciOiJ3d3cuanVudGVjaC50b3AiLCJpYXQiOjE1NzY2NjEyNzEsImV4cCI6MTU3NzI2NjA3MX0.2vNn0m9Ojz_Iq-Rksi1GG_9S2CaDzppo2foyfnO3vok";
        Claims claims = JwtUtils.checkJwt(token);
        if(claims!=null){
            String name = (String)claims.get("name");
            int id = (int)claims.get("id");
            String img = (String)claims.get("img");
            System.out.println(name+":"+id+":"+img);
        }

    }
}

```

## 4.进行单元测试

运行测试类，查看控制台：

```
生成token的方法：token = eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqdW50ZWNoX2NsYXNzIiwiaWQiOjk5OSwibmFtZSI6InJ5YW4iLCJpbWciOiJ3d3cuanVudGVjaC50b3AiLCJpYXQiOjE1NzY2NjQ5MzYsImV4cCI6MTU3NzI2OTczNn0.57fNdevyJhUC9kVbxMV9JCtTuf8rG13GvOjbzZLO4fo
解析token的方法：ryan:999:www.juntech.top
```

## 5.结束

感谢大家的观看