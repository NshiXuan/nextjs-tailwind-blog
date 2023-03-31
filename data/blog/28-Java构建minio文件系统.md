---
title: 'Java构建minio文件系统' # 标题
date: '2023/3/31' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java构建minio文件系统'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2ff2a0330b94d4d935cd2cc26a1b4f5~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

## 下载 minio

官网：https://min.io

下载 minio，下载地址在https://dl.min.io/server/minio/release/

## minio 特点

MinIO 构建分布式文件系统，MinIO 是一个非常轻量的服务,可以很简单的和其他应用的结合使用，它兼容亚马逊 S3 云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等。

它一大特点就是轻量，使用简单，功能强大，支持各种平台，单个文件最大 5TB，兼容 Amazon S3 接口，提供了 Java、Python、GO 等多版本 SDK 支持。

MinIO 集群采用去中心化共享架构，每个结点是对等关系，通过 Nginx 可对 MinIO 进行负载均衡访问。

去中心化有什么好处？

在大数据领域，通常的设计理念都是无中心和分布式。Minio 分布式模式可以帮助你搭建一个高可用的对象存储服务，你可以使用这些存储设备，而不用考虑其真实物理位置。

它将分布在不同服务器上的多块硬盘组成一个对象存储服务。由于硬盘分布在不同的节点上，分布式 Minio 避免了单点故障。如下图

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f4ffb8679ca4d8f96adfe28077ce158~tplv-k3u1fbpfcp-watermark.image?)

Minio 使用纠删码技术来保护数据，它是一种恢复丢失和损坏数据的数学算法，它将数据分块冗余的分散存储在各各节点的磁盘上，所有的可用磁盘组成一个集合，上图由 8 块硬盘组成一个集合，当上传一个文件时会通过纠删码算法计算对文件进行分块存储，除了将文件本身分成 4 个数据块，还会生成 4 个校验块，数据块和校验块会分散的存储在这 8 块硬盘上。

使用纠删码的好处是即便丢失一半数量（N/2）的硬盘，仍然可以恢复数据。 比如上边集合中有 4 个以内的硬盘损害仍可保证数据恢复，不影响上传和下载，如果多于一半的硬盘坏了则无法恢复。

## 启动 minio

需要去官网下载 `minio`，下载完后创建 `data` 文件夹（数量随便），如下创建了四个，执行命令运行

```
minio.exe server D:\minio\data1 D:\minio\data2 D:\minio\data3 D:\minio\data4
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/203a4ed743664809b6d95c1290f1698d~tplv-k3u1fbpfcp-watermark.image?)

运行后访问 `9000` 端口，输入用户、密码，默认为 `minioadmin`，创建 `buckets` ，并把 `buckets` 设为 `public`

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52eb8052d25a46fca76541f245107b12~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fa5ee12f7b04ab1af8fa7cbc3dd17ff~tplv-k3u1fbpfcp-watermark.image?)

## Java 操作 minio

### 安装 `Java` 依赖

```xml
<!-- minio -->
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.4.3</version>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.8.1</version>
</dependency>

<!--根据扩展名取mimetype-->
<dependency>
    <groupId>com.j256.simplemagic</groupId>
    <artifactId>simplemagic</artifactId>
    <version>1.17</version>
</dependency>
```

### 上传、删除、下载文件

创建测试类测试

```java
import com.j256.simplemagic.ContentInfo;
import com.j256.simplemagic.ContentInfoUtil;
import io.minio.GetObjectArgs;
import io.minio.MinioClient;
import io.minio.RemoveObjectArgs;
import io.minio.UploadObjectArgs;
import org.apache.commons.codec.digest.DigestUtils;
import org.apache.commons.io.IOUtils;
import org.junit.jupiter.api.Test;
import org.springframework.http.MediaType;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.FilterInputStream;

/**
 * @description 测试minio的sdk
 */
public class MinioTest {
    // 获取minio对象
    MinioClient minioClient =
            MinioClient.builder()
                    .endpoint("http://minio运行的ip地址:9000")
                    .credentials("minioadmin", "minioadmin")
                    .build();

    /**
     * 上传文件
     */
    @Test
    public void test_upload() throws Exception {
        //通过扩展名得到媒体资源类型 mimeType
        //根据扩展名取出mimeType
        ContentInfo extensionMatch = ContentInfoUtil.findExtensionMatch(".mp4");
        String mimeType = MediaType.APPLICATION_OCTET_STREAM_VALUE;//通用mimeType，字节流
        if (extensionMatch != null) {
            mimeType = extensionMatch.getMimeType();
        }

        // 上传文件的参数信息
        UploadObjectArgs uploadObjectArgs = UploadObjectArgs.builder()
                .bucket("testbucket")// 指定上传的桶
                .filename("C:\...\electron.png") // 指定本地上传的文件路径
                .object("test1.png") // 上传后文件的名称 如果是test/test.png 会放在test目录下
                .contentType(mimeType)//设置媒体文件类型
                .build();

        // 上传文件
        minioClient.uploadObject(uploadObjectArgs);

        // todo: 校验文件的完整性
    }

    /**
     * 删除文件
     */
    @Test
    public void test_delete() throws Exception {
        // 要删除文件的参数信息
        RemoveObjectArgs removeObjectArgs = RemoveObjectArgs.builder()
                .bucket("testbucket")
                .object("test1.png")
                .build();

        // 删除文件
        minioClient.removeObject(removeObjectArgs);
    }

    /**
     * 查询文件下载 从minio中下载
     */
    @Test
    public void test_getFile() throws Exception {
        // 需要获取的文件的参数信息
        GetObjectArgs getObjectArgs = GetObjectArgs.builder()
                .bucket("testbucket")
                .object("test1.png")
                .build();

        // 获取文件输入流
        FilterInputStream inputStream = minioClient.getObject(getObjectArgs);

        // 指定输出流
        FileOutputStream outputStream = new FileOutputStream(new File("D:\upload\test1.png"));

        // 把输入流拷贝给输出流
        IOUtils.copy(inputStream, outputStream);

        // 通过md5 校验文件的完整性
        String source_md5 = DigestUtils.md5Hex(inputStream);
        // 获取下载的文件的输入流
        FileInputStream fileInputStream = new FileInputStream(new File("D:\upload\test1.png"));
        String local_md5 = DigestUtils.md5Hex(fileInputStream);
        if (source_md5.equals(local_md5)) {
            System.out.println("下载成功");
        }
    }
}
```

上传成功示例

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0edfa8c005d4658bd3376d7b3a2b2ec~tplv-k3u1fbpfcp-watermark.image?)
