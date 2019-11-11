---
title: mybatis传多类型不同参数
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
date: 2019-11-11 14:37:37
summary: mybatis传多类型不同参数
categories: mybatis
tags: mybatis
keywords: mybatis
---

# mybatis传多类型不同参数

mybatis传参数

如果全是字符串可以采用map来传，如果是整型和字符串怎么传

```java
public User test(int a,String b)
```

## 1.可以改为这样，用@Param来传

```java
public User test(@Param("a")int a,@Param("b")String b)
```

在mapper.xml中写

```sql
<select id="***" resultType="***.**.User">
	select * form * where a = #{a} and b=#{b}
</select>
```

就会报找不到参数错误,然后将参数改为如下就可以了

```sql
<select id="***" resultType="***.**.User">
	select * form * where a = #{param1} and b=#{param2}
</select>
```

## 2.可以使用对象，将参数封装进参数对象***RequestVO或者 InAO

### 2.1创建一个对象ParamInAO

```java
@Setter
@Getter
@ToString(supercall=true)
public class ParamInAO{
    private String b;
    private Integer a;
}
```

### 2.2 将参数封装进对象

```java
ParamInAO in = new ParamInAO();
in.setA(a);
in.setB(b);
```

### 2.3 mapper文件

假如是***Mapper.java,这里使用通用mapper,当然这要看你自己项目需要

```java
tk.Mapper
public interface ***Mapper extends Mapper<User>{
    public User get****(PramInAO in);
    
    public void update(ParamInAO in);
}
```

2.4 mapper.xml

```sql
<select id="***" resultType="***.**.User" parameterType="**.**.ParamInAO">
	select * form * where a = #{param1} and b=#{param2}
</select>

<update id="***" parameterType="**.**.ParamInAO">
	update table set a=#{a},b={b};
</update>
```

这样就大功告成了

# [Mybatis传多个参数（三种解决方案） mapper.xml的sql语句修改！]

# 第一种

```
Public User selectUser(String name,String area);
```

对应的Mapper.xml  

```
<select id="selectUser" resultMap="BaseResultMap">
    select  *  from user_user_t   where user_name = #{0} and user_area=#{1}
</select>
```

其中，#{0}代表接收的是dao层中的第一个参数，#{1}代表dao层中第二参数，更多参数一致往后加即可。

 

# 第二种

此方法采用Map传多参数.

Dao层的函数方法

```sql
Public User selectUser(Map paramMap);
```

 

对应的Mapper.xml

```SQL
<select id=" selectUser" resultMap="BaseResultMap">
   select  *  from user_user_t   where user_name = #{userName，jdbcType=VARCHAR} and user_area=#{userArea,jdbcType=VARCHAR}
</select>
```

 

 

# 第三种

Dao层的函数方法

```sql
Public User selectUser(@param(“userName”)Stringname,@param(“userArea”)String area);
```

对应的Mapper.xml

```sql
<select id=" selectUser" resultMap="BaseResultMap">
   select  *  from user_user_t   where user_name = #{userName，jdbcType=VARCHAR} and user_area=#{userArea,jdbcType=VARCHAR}
</select> 
```