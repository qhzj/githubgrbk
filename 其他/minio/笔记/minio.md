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







# 操作



## 判断桶是否存在

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





## 获取存储桶的信息列表

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





## 获取预览url

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





## 下载文件对象

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





## 上传文件对象

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







## 将给定流作为存储桶中的对象上传

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





## 删除对象

```Java
minioClient.removeObject(
        RemoveObjectArgs.builder().bucket("testdir").object("2312.zip").build());
```





## 获取对象的对象信息和元数据

```java
//返回的数据，桶名称，文件名称，最后修改的时间，文件大小
StatObjectResponse statObjectResponse = minioClient.statObject(
        StatObjectArgs.builder().bucket("testdir").object("111.mp4").build());
```





## 注意事项

1. minio上传文件小于5M时，返回的ObjectWriteResponse中的region为null,只能通过获取url来访问预览文件
2. minio上传不同格式文件有些文件不支持预览的原因是因为浏览器不知道该类型的文件预览展示
   1. 解决方法，前端修改，或者后端转格式





## 文档路径

[Java Client API Reference — MinIO Object Storage for Linux](https://min.io/docs/minio/linux/developers/java/API.html)











