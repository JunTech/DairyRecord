# win10搭建hadoop环境

## 1.搭建java环境

参考https://www.runoob.com/w3cnote/windows10-java-setup.html



## 2、下载Hadoop

地址为 <https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/stable/>  这里选择的是hadoop-2.7.7.tar.gz

## 3、将其解压到某一文件夹，这里为D:\hadoop\hadoop2.7.7

![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025211125531-1793687676.png)



## 4、添加“HADOOP_HOME”环境变量，并添加到Path环境变量中

![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025211334531-296577533.png)

## 5、修改Hadoop配置文件



​	在这之前你要先下载[sardetushar_gitrepo_download](https://github.com/sardetushar/hadooponwindows/archive/master.zip) ，之后解压，删掉D:\hadoop\hadoop-2.7.7目录下的bin、etc文件夹，用刚刚解压的替换。

​	修改一下D:\hadoop\hadoop-2.7.7\etc\hadoop\hdfs-site.xml

```xml
	<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
       <name>dfs.namenode.name.dir</name>
       <value>/hadoop/data/namenode</value>
    </property>
    <property>
       <name>dfs.datanode.data.dir</name>
       <value>/hadoop/data/datanode</value>
    </property>
</configuration>
```

​	修改一下D:\hadoop\hadoop-2.7.7\etc\hadoop\hadoop-env.cmd

​	修改jdk路径

​	![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025213000500-721139247.png)

## 6、格式化HDFS文件系统，hdfs namenode -format 

![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025213330859-719313837.png)





## 七、在命令行（管理员）将目录指向D:\hadoop\hadoop-2.7.7\sbin，键入“start-all”

![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025213553734-1606757521.png)



Namenode、Datanode、YARN resourcemanager、YARN nodemanager四个进程启动成功，再看一下网站截图：

　　打开[localhost:8088]()

![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025213916171-1066761185.png)

打开localhost:50070

![img](https://images2015.cnblogs.com/blog/751642/201610/751642-20161025213946812-850843577.png)



最后使用“stop-all”,停止hadoop进程！

请放心食用，hadoop环境搭建完成！