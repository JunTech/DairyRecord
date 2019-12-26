# mybatis逆向工程--项目开发神器

## 1.准备环境

### 1.1开发工具

工具：mysql-java-connetion.jar 下载[地址：点击下载](/download/mysql-connector-java-5.1.48.jar)

​	sql文件： [点击下载sql文件](/download/xdclass.sql)

​	逆向工程文件： [点击下载逆向工程文件](/download/generatorConfig.xml)

​	开发工具： idea 自行下载

### 1.2.准备开发

新建一个spiingboot工程，加入依赖

pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.juntech</groupId>
    <artifactId>juntech-class</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>juntech-class</name>
    <description>juntech课堂--单体架构</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- https://mvnrepository.com/artifact/tk.mybatis/mapper-spring-boot-starter -->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.1.5</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core -->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
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

    <build>

        <plugins>
            <!-- MyBatis 逆向工程 插件 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.6</version>
                <configuration>
                    <!-- 允许移动生成的文件 -->
                    <verbose>true</verbose>
                    <!-- 是否覆盖 -->
                    <overwrite>true</overwrite>
                    <!-- 配置文件 -->
                    <configurationFile>
                        ${basedir}/src/main/resources/generatorConfig.xml
                    </configurationFile>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

在这里，可以引入mysql依赖，引入了就不用在逆向工程文件加入jar包文件了

虽说上面已经有了逆向工程文件和sql,这里还是把逆向文件代码贴出来，防止文件不在了：

generatorConfig.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
		PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
	<!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包-->
	<classPathEntry  location="F:\project\juntech-class\src\main\resources\jar\mysql-connector-java-5.1.48.jar"/>
	<context id="DB2Tables"  targetRuntime="MyBatis3">
		<commentGenerator>
			<property name="suppressDate" value="true"/>
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true"/>
		</commentGenerator>
		<!--数据库链接URL，用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:13306/classroom" userId="root" password="123456">
		</jdbcConnection>
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false"/>
		</javaTypeResolver>
		<!-- 生成模型的包名和位置-->
		<javaModelGenerator targetPackage="top.juntech.bean" targetProject="src/main/java">
			<property name="enableSubPackages" value="true"/>
			<property name="trimStrings" value="true"/>
		</javaModelGenerator>
		<!-- 生成映射文件的包名和位置-->
		<sqlMapGenerator targetPackage="top.juntech.mapper" targetProject="src/main/java">
			<property name="enableSubPackages" value="true"/>
		</sqlMapGenerator>
		<!-- 生成DAO的包名和位置-->
		<javaClientGenerator type="XMLMAPPER" targetPackage="top.juntech.mapper" targetProject="src/main/java">
			<property name="enableSubPackages" value="true"/>
		</javaClientGenerator>
		<!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名-->
		<table tableName="comment" domainObjectName="Comment"
			   enableCountByExample="false" enableUpdateByExample="false"
			   enableDeleteByExample="false" enableSelectByExample="false"
			   selectByExampleQueryId="false">
		</table>
		<table tableName="chapter" domainObjectName="Chapter"
			   enableCountByExample="false" enableUpdateByExample="false"
			   enableDeleteByExample="false" enableSelectByExample="false"
			   selectByExampleQueryId="false">
		</table>
		<table tableName="episode" domainObjectName="Episode"
			   enableCountByExample="false" enableUpdateByExample="false"
			   enableDeleteByExample="false" enableSelectByExample="false"
			   selectByExampleQueryId="false">
		</table>
		<table tableName="user" domainObjectName="User"
			   enableCountByExample="false" enableUpdateByExample="false"
			   enableDeleteByExample="false" enableSelectByExample="false"
			   selectByExampleQueryId="false">
		</table>
		<table tableName="video" domainObjectName="Video"
			   enableCountByExample="false" enableUpdateByExample="false"
			   enableDeleteByExample="false" enableSelectByExample="false"
			   selectByExampleQueryId="false">
		</table>
		<table tableName="video_order" domainObjectName="VideoOrder"
			   enableCountByExample="false" enableUpdateByExample="false"
			   enableDeleteByExample="false" enableSelectByExample="false"
			   selectByExampleQueryId="false">
		</table>
	</context>
</generatorConfiguration>
```



在项目下，添加2个包:mapper,bean

一个用来保存表映射过来生成的bean文件，另外一个用来保存Mapper.java以及Mapper.xml文件的，在这里我把他们放在了同一个文件夹下，方便编写

项目目录结构大致如上：

![项目结构](https://img.vim-cn.com/48/cb276ffd46218a57efc78cd3509d61f51ae78d.png)

然后运行maven命令就行了：mybatis-generator:generate

或者：

1、![](https://img.vim-cn.com/2e/b88ef48d356b9c70715bf7b9b689bdaf3c6751.png)

2.![](https://img.vim-cn.com/75/784c3636df01aac881052ed2bb65c7688af176.png)

点击运行就行了

查看控制台显示：

```
D:\devtools\apache-maven-3.5.4\conf\settings.xml -Dmaven.repo.local=D:\devtools\apache-maven-3.5.4\repository mybatis-generator:generate
[WARNING] 
[WARNING] Some problems were encountered while building the effective settings
[WARNING] Unrecognised tag: 'repositories' (position: START_TAG seen ...</profile>\n    -->\n\n    <repositories>... @229:19)  @ D:\devtools\apache-maven-3.5.4\conf\settings.xml, line 229, column 19
[WARNING] Unrecognised tag: 'repositories' (position: START_TAG seen ...</profile>\n    -->\n\n    <repositories>... @229:19)  @ D:\devtools\apache-maven-3.5.4\conf\settings.xml, line 229, column 19
[WARNING] 
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< top.juntech:juntech-class >----------------------
[INFO] Building juntech-class 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- mybatis-generator-maven-plugin:1.3.6:generate (default-cli) @ juntech-class ---
[INFO] Connecting to the Database
[INFO] Introspecting table comment
[INFO] Introspecting table chapter
[INFO] Introspecting table episode
[INFO] Introspecting table user
[INFO] Introspecting table video
[INFO] Introspecting table video_order
[INFO] Generating Record class for table comment
[INFO] Generating Mapper Interface for table comment
[INFO] Generating SQL Map for table comment
[INFO] Generating Record class for table chapter
[INFO] Generating Mapper Interface for table chapter
[INFO] Generating SQL Map for table chapter
[INFO] Generating Record class for table episode
[INFO] Generating Mapper Interface for table episode
[INFO] Generating SQL Map for table episode
[INFO] Generating Record class for table user
[INFO] Generating Mapper Interface for table user
[INFO] Generating SQL Map for table user
[INFO] Generating Record class for table video
[INFO] Generating Mapper Interface for table video
[INFO] Generating SQL Map for table video
[INFO] Generating Record class for table video_order
[INFO] Generating Mapper Interface for table video_order
[INFO] Generating SQL Map for table video_order
[INFO] Saving file CommentMapper.xml
[INFO] Saving file ChapterMapper.xml
[INFO] Saving file EpisodeMapper.xml
[INFO] Saving file UserMapper.xml
[INFO] Saving file VideoMapper.xml
[INFO] Saving file VideoOrderMapper.xml
[INFO] Saving file Comment.java
[INFO] Saving file CommentMapper.java
[INFO] Saving file Chapter.java
[INFO] Saving file ChapterMapper.java
[INFO] Saving file Episode.java
[INFO] Saving file EpisodeMapper.java
[INFO] Saving file User.java
[INFO] Saving file UserMapper.java
[INFO] Saving file Video.java
[INFO] Saving file VideoMapper.java
[INFO] Saving file VideoOrder.java
[INFO] Saving file VideoOrderMapper.java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.793 s
[INFO] Finished at: 2019-12-18T11:20:13+08:00
[INFO] ------------------------------------------------------------------------
```

在打开工程项目，查看bean目录和mapper目录：

![](https://img.vim-cn.com/2b/8ed178de50142303c5af0572b6ad035027e72c.png)

好了，我们可以进行快乐的开发了，这些bean文件写起来是真的无聊又费事，有了逆向工程，妈妈再也不用担心我的代码了。

## 2.致谢

 感谢各位coder抽出宝贵时间来查看这篇文章，非常感谢！