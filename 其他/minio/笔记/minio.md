# minio介绍

MinIO 是一款基于 Go 语言的高性能、可扩展、云原生支持、操作简单、开源的分布式对象存储产品。

官网：https://min.io/



## 特点

* 高性能：作为高性能对象存储，在标准硬件条件下它能达到55GB/s的读、35GG/s的写速率
* 可扩容：不同MinIO集群可以组成联邦，并形成一个全局的命名空间，并跨越多个数据中心
* 云原生：容器化、基于K8S的编排、多租户支持
* Amazon S3兼容：Minio使用Amazon S3 v2 / v4 API。可以使用Minio SDK，Minio Client，AWS SDK和AWS CLI访问Minio服务器。
* 可对接后端存储: 除了Minio自己的文件系统，还支持DAS、 JBODs、NAS、Google云存储和Azure Blob存储。
* SDK支持: 基于Minio轻量的特点，它得到类似Java、Python或Go等语言的sdk支持
* Lambda计算: Minio服务器通过其兼容AWS SNS / SQS的事件通知服务触发Lambda功能。支持的目标是消息队列，如Kafka，NATS，AMQP，MQTT，Webhooks以及Elasticsearch，Redis，Postgres和MySQL等数据库。
* 有操作页面
* 功能简单: 这一设计原则让MinIO不容易出错、更快启动
* 支持纠删码：MinIO使用纠删码，在最高冗余度配置下，即使丢失1/2的磁盘也能恢复数据





## 相关名词概念

Object：存储到minio的基本对象，如文件，字节流等。

Bucket（桶）：用来存储Object的逻辑空间。每个Bucket之间的数据是互相隔离的。对于客户端而言，就相当于存放文件的顶层文件夹。

Drlve（驱动器）：存储数据的磁盘，在MinIO启动时，以参数的方式传入。MinIO中所有的对象数据都会存在Drive里。

Set：即一组 Drive的集合，分布式部署根据集群规模自动划分一个或者多个Set，每个Set中的Drive 分布在不同位置。一个对象存储在一个Set上。 作者：爱玩游戏的白白狗 https://www.bilibili.com/read/cv20353378 出处：bilibili



# 安装



## Windows环境

### 1：minio文件下载

下载地址：https://dl.min.io/server/minio/release/windows-amd64/minio.ex



### 2：创建目录

与minin.exe同级创建data目录

![image-20230517095727307](minio.assets/image-20230517095727307.png)



### 3：启动

在minio.exe目录下使用cmd命令

```
-- minio.exe server data的路径（相对路径决定路径都可以）
minio.exe server D:\develop\software\minio\data
或者
minio.exe server data
```



![image-20230517095937236](minio.assets/image-20230517095937236.png)

红字表明：警告：说你使用的是默认的，建议你可以修改



### 4：访问

访问地址：http://127.0.0.1:9000

根据打印日志来













## Linux环境



### 1：创建目录

```
mkdir /home/minio
mkdir /home/minio/data
```



### 2：下载安装包

切换到/home/minio路径下

```
wget https://dl.min.io/server/minio/release/linux-amd64/minio
```

或者可以直接点击链接下载到本地，在拖动上传到/home/minio路径下

下载地址：https://dl.min.io/server/minio/release/linux-amd64/minio



### 3：赋予下载下来的minio文件权限

```shell
-- 给指定文件赋予权限 x为执行权限
chmod +x minio
```

这一步需要切换到/home/minio路径下，

注意：是给下载下来的minio文件赋予权限，而不是给创建的minio文件夹赋予权限



### 4：启动

```shell
./minio server /home/minio/data
```

这一步也要在/home/minio目录下



### 5：启动后打印

* 一些连接信息

![image-20230512160516435](minio.assets/image-20230512160516435.png)



### 6：添加映射

如果出现下列的打印，就是需要添加映射

![img](minio.assets/b9e81f41c04a44ba8cd64d6bc2ad7db1.png)



```
./minio server --console-address '0.0.0.0:9999'  /home/minio/data
 
nohup ./minio server --console-address '0.0.0.0:9999'  /home/minio/data &  #后台启动
```

![img](minio.assets/9138a92c5ba043a6b6e8c7d6e388f1f6.png)



### 7：修改密码

警告说的是建议修改账号密码 默认账号密码为minioadmin端口为9000

```
export MINIO_ACCESS_KEY=XXXXXX
export MINIO_SECRET_KEY=XXXXXX
```



### 8：开放端口

这一步根据打印出来的端口信息进行更改，

```shell
-- 打开9000端口
firewall-cmd --zone=public --add-port=9000/tcp --permanent
-- 打开9999端口
firewall-cmd --zone=public --add-port=9999/tcp --permanent
 -- 防火墙重载
firewall-cmd --reload
```



### 9：访问

![image-20230512161637734](minio.assets/image-20230512161637734.png)



### 10：过程遇到的问题



# Java Clinet

## 依赖

```xml
    <dependencies>
        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>8.4.6</version>
            <exclusions>
                <exclusion>
                    <groupId>com.squareup.okhttp3</groupId>
                    <artifactId>okhttp</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>4.9.0</version>
        </dependency>
        
        <dependency>
            <groupId>me.tongfei</groupId>
            <artifactId>progressbar</artifactId>
            <version>0.5.3</version>
        </dependency>
    </dependencies>


```



## 配置文件

```yml
minio:
  endPoint: http://127.0.0.1:9000
  accessKey: minioadmin
  secretKey: minioadmin
  bucket: work
```





```java
package com.zbf.bean;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "minio")
public class MinioProperties {

    private String endPoint;
    private String accessKey;
    private String secretKey;
}

```



```java
package com.zbf.config;

import com.zbf.bean.MinioProperties;
import io.minio.MinioClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MinioConfig {

    @Autowired(required = false)
    public MinioProperties minioProperties;

    @Bean
    public MinioClient getMinioClient(){

        MinioClient build = MinioClient.builder().endpoint(minioProperties.getEndPoint())
                .credentials(minioProperties.getAccessKey(), minioProperties.getSecretKey()).build();
        return build;
    }
}

```



## Java操作



### 判断桶是否存在

```java
public static boolean judgmentBucketExist(String bucket){
        MinioClient minioClient = getMinioClient();
        boolean b = true;
        try {
            b = minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucket).build());
        } catch (Exception e) {
            e.printStackTrace();
        }
        return b;
    }
```





### 获取存储桶的信息列表

```java
public static List<Bucket> getBuckets(){

    List<Bucket> bucketList = null;
    try {
        bucketList = getMinioClient().listBuckets();
    } catch (Exception e) {
        e.printStackTrace();
    }
    for (Bucket bucket : bucketList) {
        System.out.println(bucket.creationDate() + ", " + bucket.name());
    }
    return bucketList;
}
```

![image-20230517101449757](minio.assets/image-20230517101449757.png)



### 获取预览url

```java 
Map<String, String> reqParams = new HashMap<String, String>();
reqParams.put("response-content-type", "image/png");

String url =
        minioClient.getPresignedObjectUrl(
                GetPresignedObjectUrlArgs.builder()
                        .method(Method.GET)
                        //桶
                        .bucket("testdir")
                        //文件对象
                        .object("QQ浏览器截图20210811185209.png")
                        //设置图片预览的过期时间
                        .expiry(10, TimeUnit.SECONDS)
                        //设置请求参数,也可以不设置,可以表明请求的媒体类型等
                        .extraQueryParams(reqParams)
                        .build());
System.out.println(url);
```





### 下载文件对象

```java
minioClient.downloadObject(
        DownloadObjectArgs.builder()
                //桶名
                .bucket("testdir")
                //文件对象
                .object("QQ浏览器截图20210811185209.png")
                //下载下来的文件路径,包含文件名称
                .filename("D:\\data\\123.png")
                .build());
```





### 上传文件对象

> 将文件中的内容作为存储桶中的对象上传

```java
minioClient.uploadObject(
        UploadObjectArgs.builder()
                //桶
                .bucket("testdir")
                //上传到minio的文件名,ewew/123123.zip,/前表示文件夹
                .object("12312.txt")
                //需要上传的本地文件路径
                .filename("D:\\桌面\\临时\\临时.txt")
                //文件类型,也可以不指定
                .contentType("video/mp4")
                .build());
```







### 将给定流作为存储桶中的对象上传

> 将给定的流作为存储桶中的对象上传

> 与uploadObject不同的是，上传的文件通过share路径打开是下载链接，而不是预览链接
>
> 且不支持getPresignedObjectUrl获取预览链接

> 大型文件使用这个对象，



```Java
        ObjectWriteResponse testdir = minioClient.putObject(
                PutObjectArgs.builder()
                        //桶
                        .bucket("testdir")
                        //上传到minio中的路径文件名
                        .object("2312.txt")
                        //流，流大小
                        .stream(inputStream, inputStream.available(), -1)
                        //文件类型，可以不给
//                        .contentType("video/mp4")
                        .build());

//返回的ObjectWriteResponse，存储着桶名称，文件路径等
```





### 删除对象

```Java
minioClient.removeObject(
        RemoveObjectArgs.builder().bucket("testdir").object("2312.zip").build());
```





### 获取对象的对象信息和元数据

```java
//返回的数据，桶名称，文件名称，最后修改的时间，文件大小
StatObjectResponse statObjectResponse = minioClient.statObject(
        StatObjectArgs.builder().bucket("testdir").object("111.mp4").build());
```





## 注意事项

1. minio上传文件小于5M时，返回的ObjectWriteResponse中的region为null,只能通过获取url来访问预览文件
2. minio上传不同格式文件有些文件不支持预览的原因是因为浏览器不知道该类型的文件预览展示
   1. 解决方法，前端修改，或者后端转格式





# 文档路径

[Java Client API Reference — MinIO Object Storage for Linux](https://min.io/docs/minio/linux/developers/java/API.html)











