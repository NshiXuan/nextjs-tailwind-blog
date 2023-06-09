---
title: 'Java构建minio文件系统实现分片上传' # 标题
date: '2023/3/31' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java构建minio文件系统实现分片上传'
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

### 开发上传图片、pdf、文档接口

1.配置上传文件大小

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8970b7171a10437b89638d71a25ebe90~tplv-k3u1fbpfcp-watermark.image?)

```yml
# 设置最大上传文件的大小为10MB
spring.servlet.multipart.max-file-size=10MB
# 设置单个文件最大上传大小为5MB
spring.servlet.multipart.max-request-size=5MB
```

2.配置 `minio` 的信息，启动类的 `application.yml`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4628fd62da0a4b7d99b85101e59a091d~tplv-k3u1fbpfcp-watermark.image?)

```yml
## minio配置
minio:
  endpoint: http://ip地址:9000
  accessKey: minioadmin
  secretKey: minioadmin
  bucket:
    files: mediafiles
    videofiles: videofiles
```

3.配置 `config`

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/405a9c025c8948b7b89a43bcd1f9cef6~tplv-k3u1fbpfcp-watermark.image?)

```java
package com.xuecheng.media.config;

import io.minio.MinioClient;
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@Data
public class MinioConfig {
    @Value("${minio.endpoint}")
    private String endpoint;
    @Value("${minio.accessKey}")
    private String accessKey;
    @Value("${minio.secretKey}")
    private String secretKey;

    @Bean
    public MinioClient minioClient() {
        MinioClient minioClient =
                MinioClient.builder()
                        .endpoint(endpoint)
                        .credentials(accessKey, secretKey)
                        .build();
        return minioClient;
    }
}
```

4.`control`

```java
import com.xuecheng.media.model.dto.QueryMediaParamsDto;
import com.xuecheng.media.model.dto.UploadFileParamsDto;
import com.xuecheng.media.model.dto.UploadFileResultDto;
import com.xuecheng.media.model.po.MediaFiles;
import com.xuecheng.media.service.MediaFileService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;

/**
 * @description 媒资文件管理接口
 */
@RestController
public class MediaFilesController {
    @Autowired
    MediaFileService mediaFileService;

    // consumes指定上传类型
    // @RequestPart("filedata")指定上传名称
    @RequestMapping(value = "/upload/coursefile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public UploadFileResultDto upload(@RequestPart("file") MultipartFile file) throws IOException {
        // 1.映射dto
        UploadFileParamsDto uploadFileParamsDto = new UploadFileParamsDto();
        // 文件名称
        uploadFileParamsDto.setFilename(file.getOriginalFilename());
        // 文件大小
        uploadFileParamsDto.setFileSize(file.getSize());
        // 文件类型
        uploadFileParamsDto.setFileType("001001");

        // 2.创建一个临时文件 以获取文件路径
        File tempFile = File.createTempFile("minio", "temp");
        file.transferTo(tempFile);
        // 取出文件的绝对路径
        String absolutePath = tempFile.getAbsolutePath();

        Long companyId = 1232141425L;

        // 3.调用service的上传接口
        return mediaFileService.uploadFile(companyId, uploadFileParamsDto, absolutePath);
    }

}
```

5.`service`

```java
import com.xuecheng.media.model.dto.QueryMediaParamsDto;
import com.xuecheng.media.model.dto.UploadFileParamsDto;
import com.xuecheng.media.model.dto.UploadFileResultDto;
import com.xuecheng.media.model.po.MediaFiles;

public interface MediaFileService {
    /**
     * 上传文件
     *
     * @param companyId           机构id
     * @param uploadFileParamsDto 文件信息
     * @param localFilePath       文件本地路径
     * @return UploadFileResultDto
     */
    public UploadFileResultDto uploadFile(Long companyId, UploadFileParamsDto uploadFileParamsDto, String localFilePath);

    public MediaFiles addMediaFilesToDb(Long companyId, String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String objectName);
}
```

6.`impl`

```java
import com.j256.simplemagic.ContentInfo;
import com.j256.simplemagic.ContentInfoUtil;
import com.xuecheng.base.exception.XueChengPlusException;
import com.xuecheng.base.model.PageParams;
import com.xuecheng.base.model.PageResult;
import com.xuecheng.media.mapper.MediaFilesMapper;
import com.xuecheng.media.model.dto.QueryMediaParamsDto;
import com.xuecheng.media.model.dto.UploadFileParamsDto;
import com.xuecheng.media.model.dto.UploadFileResultDto;
import com.xuecheng.media.model.po.MediaFiles;
import com.xuecheng.media.service.MediaFileService;
import io.minio.MinioClient;
import io.minio.UploadObjectArgs;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.digest.DigestUtils;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.io.File;
import java.io.FileInputStream;
import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.util.Date;
import java.util.List;

@Slf4j
@Service
public class MediaFileServiceImpl implements MediaFileService {
    @Autowired
    MediaFilesMapper mediaFilesMapper;

    @Autowired
    MinioClient minioClient;

    @Autowired
    MediaFileService currentProxy;

    // 获取桶名称
    @Value("${minio.bucket.files}")
    private String bucket_mediafiles;

    // 根据扩展名获取mimeType
    private String getMimeType(String extension) {
        if (extension == null) {
            extension = "";
        }
        //根据扩展名取出mimeType
        ContentInfo extensionMatch = ContentInfoUtil.findExtensionMatch(extension);
        String mimeType = MediaType.APPLICATION_OCTET_STREAM_VALUE;//通用mimeType，字节流
        if (extensionMatch != null) {
            mimeType = extensionMatch.getMimeType();
        }
        return mimeType;
    }

    /**
     * 将文件上传到minio
     *
     * @param localFilePath 文件本地路径
     * @param mimeType      媒体类型
     * @param bucket        桶
     * @param objectName    文件名称
     * @return
     */
    public boolean addMediaFilesToMinIO(String localFilePath, String mimeType, String bucket, String objectName) {
        try {
            UploadObjectArgs uploadObjectArgs = UploadObjectArgs.builder()
                    .bucket(bucket)//桶
                    .filename(localFilePath) //指定本地文件路径
                    .object(objectName)// 文件名称 也可以放在子目录下
                    .contentType(mimeType)//设置媒体文件类型
                    .build();
            //上传文件
            minioClient.uploadObject(uploadObjectArgs);
            log.debug("上传文件到minio成功,bucket:{},objectName:{},错误信息:{}", bucket, objectName);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            log.error("上传文件出错,bucket:{},objectName:{},错误信息:{}", bucket, objectName, e.getMessage());
        }
        return false;
    }

    // 获取文件默认存储目录路径 年/月/日
    private String getDefaultFolderPath() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        String folder = sdf.format(new Date()).replace("-", "/") + "/";
        return folder;
    }

    // 获取文件的md5
    private String getFileMd5(File file) {
        try (FileInputStream fileInputStream = new FileInputStream(file)) {
            String fileMd5 = DigestUtils.md5Hex(fileInputStream);
            return fileMd5;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    // 上传文件
    @Override
    public UploadFileResultDto uploadFile(Long companyId, UploadFileParamsDto uploadFileParamsDto, String localFilePath) {
        // 1.文件名
        String filename = uploadFileParamsDto.getFilename();

        // 2.先得到扩展名
        String extension = filename.substring(filename.lastIndexOf("."));

        // 3.通过扩展名得到mimeType
        String mimeType = getMimeType(extension);

        // 4.子目录
        String defaultFolderPath = getDefaultFolderPath();

        // 5.文件的md5值
        String fileMd5 = getFileMd5(new File(localFilePath));
        String objectName = defaultFolderPath + fileMd5 + extension;

        // 6.上传文件到minio
        boolean result = addMediaFilesToMinIO(localFilePath, mimeType, bucket_mediafiles, objectName);
        if (!result) {
            myException.cast("上传文件失败");
        }

        // 7.将文件信息保存到数据库
        MediaFiles mediaFiles = currentProxy.addMediaFilesToDb(companyId, fileMd5, uploadFileParamsDto, bucket_mediafiles, objectName);
        if (mediaFiles == null) {
            myException.cast("上传文件后保存信息入库失败");
        }

        // 8.映射返回对象
        UploadFileResultDto uploadFileResultDto = new UploadFileResultDto();
        BeanUtils.copyProperties(mediaFiles, uploadFileResultDto);

        return uploadFileResultDto;
    }

    @Transactional
    public MediaFiles addMediaFilesToDb(Long companyId, String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String objectName) {
        //将文件信息保存到数据库
        MediaFiles mediaFiles = mediaFilesMapper.selectById(fileMd5);
        if (mediaFiles == null) {
            mediaFiles = new MediaFiles();

            ...映射po数据

            //插入数据库
            int insert = mediaFilesMapper.insert(mediaFiles);
            if (insert <= 0) {
                log.debug("向数据库保存文件失败,bucket:{},objectName:{}", bucket, objectName);
                return null;
            }
            return mediaFiles;
        }
        return mediaFiles;
    }
}
```

## 分片上传

### Java 开发分片上传接口

1. 定义测试表

```sql
create table test_minio.media_files
(
    id        varchar(32)   not null comment '主键'
        primary key,
    filename  varchar(255)  null comment '文件名称',
    file_type varchar(12)   null comment '文件类型 （图片、文档、视频）',
    file_id   varchar(32)   null comment '文件id',
    file_path varchar(512)  null comment '文件存储目录',
    url       varchar(1024) null comment '文件访问地址',
    file_size bigint        null comment '文件大小'
)
    comment '文件信息表';
```

2.定义 `dto` , `po` 就不展示了

```java
package com.minio.model.dto;

import lombok.Data;
import lombok.ToString;

@Data
@ToString
public class UploadFileParamsDto {
    /**
     * 文件名称
     */
    private String filename;

    /**
     * 文件类型（文档，音频，视频）
     */
    private String fileType;

    /**
     * 文件大小
     */
    private Long fileSize;
}
```

3.`controller`

```java
package com.minio.controller;

import com.minio.model.dto.R;
import com.minio.model.dto.UploadFileParamsDto;
import com.minio.model.po.MediaFiles;
import com.minio.service.MediaFilesService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.util.List;

@RestController
public class MinioController {
    @Autowired
    MediaFilesService mediaFilesService;

    /**
     * 获取全部文件
     */
    @GetMapping("/files")
    public R<List<MediaFiles>> getFiles() {
        List<MediaFiles> files = mediaFilesService.getFiles();
        return R.success(files);
    }

    /**
     * 直接上传文件 不涉及分块
     *
     * @param file
     * @return
     * @throws Exception
     */
    @PostMapping("/upload/files")
    // @RequestPart("file") 注解表示将 HTTP 请求中的一个文件作为请求参数传递给方法，并绑定到对应的方法参数上。
    public R<MediaFiles> uplaodFile(@RequestPart("file") MultipartFile file) throws Exception {
        // 1.映射dto
        UploadFileParamsDto uploadFileParamsDto = new UploadFileParamsDto();
        // 文件名称
        uploadFileParamsDto.setFilename(file.getOriginalFilename());
        // 文件大小
        uploadFileParamsDto.setFileSize(file.getSize());
        // 文件类型
        uploadFileParamsDto.setFileType("图片");

        // 2.创建一个临时文件 以获取文件路径
        File tempFile = File.createTempFile("minio", "temp");
        file.transferTo(tempFile);
        // 取出文件的绝对路径
        String absolutePath = tempFile.getAbsolutePath();

        // 3.调用service
        MediaFiles mediaFiles = mediaFilesService.uploadFile(uploadFileParamsDto, absolutePath);

        return R.success(mediaFiles);
    }

    /**
     * 上传分块
     *
     * @param fileMd5    原始文件的md5
     * @param chunk      分块文件
     * @param chunkIndex 分块序号
     * @return
     * @throws Exception
     */
    @PostMapping("/upload/uploadchunk")
    public R<Boolean> uploadChunk(@RequestParam("fileMd5") String fileMd5, @RequestParam("chunk") MultipartFile chunk, @RequestParam("chunkIndex") int chunkIndex) throws Exception {
        // 1.创建临时文件获取文件路径
        File tempFile = File.createTempFile("minio", "temp");
        chunk.transferTo(tempFile);
        // 取出文件的绝对路径
        String localFilePath = tempFile.getAbsolutePath();


        // 3.调用service上传分块
        Boolean result = mediaFilesService.uploadChunk(fileMd5, chunkIndex, localFilePath);
        return R.success(result);
    }

    /**
     * 检查分块
     *
     * @param fileMd5    原始文件的md5
     * @param chunkIndex 分块序号
     */
    @PostMapping("/upload/checkchunk")
    public R<Boolean> checkChunk(@RequestParam("fileMd5") String fileMd5, @RequestParam("chunkIndex") int chunkIndex) {
        Boolean result = mediaFilesService.checkChunk(fileMd5, chunkIndex);
        return R.success(result);
    }

    /**
     * 合并文件
     *
     * @param fileMd5    原始文件的md5
     * @param fileName   原始文件的名称
     * @param chunkTotal 分块总数
     */
    @PostMapping("/upload/mergechunks")
    public R<Boolean> mergeChunk(@RequestParam("fileMd5") String fileMd5, @RequestParam("fileName") String fileName, @RequestParam("chunkTotal") int chunkTotal) {
        // 1.映射文件参数信息
        UploadFileParamsDto uploadFileParamsDto = new UploadFileParamsDto();
        uploadFileParamsDto.setFilename(fileName);
        uploadFileParamsDto.setFileType("视频");

        Boolean res = mediaFilesService.mergeChunk(fileMd5, chunkTotal, uploadFileParamsDto);
        return R.success(res);
    }
}
```

4.封装工具类 `Utils`

```java
package com.minio.utils;

import com.j256.simplemagic.ContentInfo;
import com.j256.simplemagic.ContentInfoUtil;
import org.apache.commons.codec.digest.DigestUtils;
import org.springframework.http.MediaType;

import java.io.File;
import java.io.FileInputStream;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Utils {
    // 根据扩展名取出mimeType
    public static String getMimeType(String extension) {
        if (extension == null) {
            extension = "";
        }
        ContentInfo extensionMatch = ContentInfoUtil.findExtensionMatch(extension);
        String mimeType = MediaType.APPLICATION_OCTET_STREAM_VALUE;//通用mimeType，字节流
        if (extensionMatch != null) {
            mimeType = extensionMatch.getMimeType();
        }
        return mimeType;
    }

    //格式化年/月/日 用来作为文件存储目录 分片不用这个 分片用md5
    public static String getFormatTime() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        String folder = sdf.format(new Date()).replace("-", "/") + "/";
        return folder;
    }

    // md5加密文件
    public static String getFileMd5(File file) {
        try (FileInputStream fileInputStream = new FileInputStream(file)) {
            String fileMd5 = DigestUtils.md5Hex(fileInputStream);
            return fileMd5;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

5.`service`

```java
package com.minio.service;

import com.minio.model.dto.UploadFileParamsDto;
import com.minio.model.po.MediaFiles;

import java.util.List;

public interface MediaFilesService {

    /**
     * 获取全部文件信息
     */
    List<MediaFiles> getFiles();

    /**
     * 上传文件
     *
     * @param uploadFileParamsDto 文件信息
     * @param localFilePath       本地文件路径
     */
    MediaFiles uploadFile(UploadFileParamsDto uploadFileParamsDto, String localFilePath);

    /**
     * 上传分块
     *
     * @param fileMd5            原始文件的md5
     * @param chunkIndex         分块序号
     * @param localChunkFIlePath 分块文件本地路径
     * @return
     */
    Boolean uploadChunk(String fileMd5, int chunkIndex, String localChunkFIlePath);

    /**
     * 检查分块
     *
     * @param fileMd5    原始文件md5
     * @param chunkIndex 分块序号
     */
    Boolean checkChunk(String fileMd5, int chunkIndex);

    /**
     * 合并分块
     *
     * @param fileMd5             原始文件md5 用来获取分块目录
     * @param chunkTotal          分块总数
     * @param uploadFileParamsDto 文件参数
     * @return
     */
    Boolean mergeChunk(String fileMd5, int chunkTotal, UploadFileParamsDto uploadFileParamsDto);

    /**
     * 文件信息入库
     *
     * @param fileMd5             原始文件md5 用来做文件的id
     * @param uploadFileParamsDto 文件参数
     * @param bucket              存储的桶
     * @param filePath            文件存储路径
     * @return
     */
    MediaFiles addFilesToDb(String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String filePath);
}
```

6.`impl`

```java
package com.minio.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.minio.exception.Code;
import com.minio.exception.MyException;
import com.minio.mapper.MediaFilesMapper;
import com.minio.model.dto.UploadFileParamsDto;
import com.minio.model.po.MediaFiles;
import com.minio.service.MediaFilesService;
import com.minio.utils.Utils;
import io.minio.*;
import io.minio.messages.DeleteError;
import io.minio.messages.DeleteObject;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.io.File;
import java.io.FilterInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@Slf4j
@Service
public class MediaFilesServiceImpl implements MediaFilesService {
    @Autowired
    MediaFilesMapper mediaFilesMapper;

    @Autowired
    MinioClient minioClient;

    @Autowired
    MediaFilesService currentProxy;


    //存储普通文件
    @Value("${minio.bucket.files}")
    private String bucket_files;

    //存储视频
    @Value("${minio.bucket.video}")
    private String bucket_video;

    @Override
    public List<MediaFiles> getFiles() {
        LambdaQueryWrapper<MediaFiles> wrapper = new LambdaQueryWrapper<>();
        return mediaFilesMapper.selectList(wrapper);
    }

    /**
     * 上传文件
     *
     * @param uploadFileParamsDto 文件信息
     * @param localFilePath       本地文件路径
     */
    @Override
    public MediaFiles uploadFile(UploadFileParamsDto uploadFileParamsDto, String localFilePath) {
        // 1.获取文件名
        String filename = uploadFileParamsDto.getFilename();

        // 2.获取扩展名
        // lastIndexOf() 方法查找字符串中最后一个出现指定字符的位置，并返回该位置到字符串结尾的子字符串
        String extension = filename.substring(filename.lastIndexOf("."));

        // 3.通过扩展名拿到mimeType
        String mimeType = Utils.getMimeType(extension);

        // 4.配置存储文件在minio的目录 年/月/日
        String formatTime = Utils.getFormatTime();

        // 5.获取文件的md5值作为存储在minio的文件名称
        String fileMd5 = Utils.getFileMd5(new File(localFilePath));

        // 6.配置文件存储路径 目录(年/月/日)+名称+后缀名
        String filePath = formatTime + fileMd5 + extension;

        // 7.上传到minio
        boolean result = uploadFilesToMinIO(localFilePath, mimeType, bucket_files, filePath);
        if (!result) {
            MyException.cast(Code.UPLOAD_ERROR);
        }

        // 8.将数据保存到数据库
        MediaFiles mediaFiles = currentProxy.addFilesToDb(fileMd5, uploadFileParamsDto, bucket_files, filePath);
        if (mediaFiles == null) {
            MyException.cast(Code.SERVICE_BUSY);
        }

        return mediaFiles;
    }

    /**
     * 上传分块
     *
     * @param fileMd5            原始文件的md5
     * @param chunkIndex         分块序号
     * @param localChunkFIlePath 分块文件本地路径
     * @return
     */
    @Override
    public Boolean uploadChunk(String fileMd5, int chunkIndex, String localChunkFIlePath) {
        // 1.根据md5获取文件存储路径 MD5的前面两个字符作为目录 如 a/b/abcxxxxxxxx/chunk/i
        String chunkFilePath = getChunkFilePath(fileMd5) + chunkIndex;

        // 2.获取mimeType
        String mimeType = Utils.getMimeType(null);

        // 3.将分块文件上传到minio
        boolean b = uploadFilesToMinIO(localChunkFIlePath, mimeType, bucket_video, chunkFilePath);
        if (!b) {
            MyException.cast(Code.UPLOAD_ERROR);
            return false;
        }

        return true;
    }

    /**
     * 检查分块
     *
     * @param fileMd5    原始文件md5
     * @param chunkIndex 分块序号
     * @return
     */
    @Override
    public Boolean checkChunk(String fileMd5, int chunkIndex) {
        // 1.获取minio存储分块的目录路径
        String chunkFilePath = getChunkFilePath(fileMd5);

        // 2.获取文件参数信息
        GetObjectArgs getObjectArgs = GetObjectArgs.builder()
                .bucket(bucket_video)
                .object(chunkFilePath + chunkIndex)
                .build();

        // 3.检查minio中分块是否存在
        try {
            FilterInputStream inputStream = minioClient.getObject(getObjectArgs);
            if (inputStream != null) {
                return true;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return false;
    }

    /**
     * 合并分块
     *
     * @param fileMd5             原始文件md5 用来获取分块目录
     * @param chunkTotal          分块总数
     * @param uploadFileParamsDto 文件参数
     * @return
     */
    @Override
    public Boolean mergeChunk(String fileMd5, int chunkTotal, UploadFileParamsDto uploadFileParamsDto) {
        // 1.获取全部分块文件的信息
        List<ComposeSource> sources = new ArrayList<>();
        // 获取分块目录
        String chunkFilePath = getChunkFilePath(fileMd5);
        for (int i = 0; i < chunkTotal; i++) {
            // 指定分块文件的信息
            ComposeSource composeSource = ComposeSource.builder()
                    .bucket(bucket_video)
                    .object(chunkFilePath + i)
                    .build();
            sources.add(composeSource);
        }

        // 2.合并分块
        // 2.1 获取源文件名称作为
        String filename = uploadFileParamsDto.getFilename();
        // 2.2 获取扩展名
        String extension = filename.substring(filename.lastIndexOf("."));
        // 2.3 获取合并后的文件路径
        String objectName = fileMd5.substring(0, 1) + "/" + fileMd5.substring(1, 2) + "/" + fileMd5 + "/" + fileMd5 + extension;

        // 2.4 定义合并参数
        ComposeObjectArgs composeObjectArgs = ComposeObjectArgs.builder()
                .bucket(bucket_video)
                .object(objectName) // 合并后文件名称
                .sources(sources) // 指定源文件
                .build();

        // 2.5 合并文件 minio默认的分块大小为5M
        try {
            minioClient.composeObject(composeObjectArgs);
        } catch (Exception e) {
            e.printStackTrace();
            log.error("合并文件出错，bucket:{},mergeName:{},错误信息:{}", bucket_video, objectName, e.getMessage());
            MyException.cast(Code.SERVICE_BUSY);
            return false;
        }

        // todo: 3.校验合并后的源文件是否一致，视频上传才成功 如何校验

        // 4.将文件信息入库
        MediaFiles mediaFiles = currentProxy.addFilesToDb(fileMd5, uploadFileParamsDto, bucket_video, objectName);
        if (mediaFiles == null) {
            log.error("文件信息插入数据库失败 mergeChunk failed");
            MyException.cast(Code.SERVICE_BUSY);
            return false;
        }

        // 5.清理分块文件
        clearChunkFiles(chunkFilePath, chunkTotal);

        return true;
    }

    /**
     * 清除分块文件
     *
     * @param chunkFilePath 分块文件路径
     * @param chunkTotal    分块文件总数
     */
    private void clearChunkFiles(String chunkFilePath, int chunkTotal) {
        Iterable<DeleteObject> objects = Stream.iterate(0, i -> ++i).limit(chunkTotal).map(i -> new DeleteObject(chunkFilePath + i)).collect(Collectors.toList());

        RemoveObjectsArgs removeObjectsArgs = RemoveObjectsArgs.builder().bucket(bucket_video).objects(objects).build();
        Iterable<Result<DeleteError>> results = minioClient.removeObjects(removeObjectsArgs);
        //要想真正删除
        results.forEach(f -> {
            try {
                DeleteError deleteError = f.get();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
    }

    /**
     * 得到分块文件的目录
     *
     * @param fileMd5 文件md5
     * @return
     */
    private String getChunkFilePath(String fileMd5) {
        return fileMd5.substring(0, 1) + "/" + fileMd5.substring(1, 2) + "/" + fileMd5 + "/" + "chunk" + "/";
    }

    /**
     * 将文件上传到minio
     *
     * @param localFilePath 文件本地路径
     * @param mimeType      媒体类型
     * @param bucket        桶
     * @param filePath      文件路径
     * @return
     */
    public boolean uploadFilesToMinIO(String localFilePath, String mimeType, String bucket, String filePath) {
        try {
            UploadObjectArgs uploadObjectArgs = UploadObjectArgs.builder()
                    .bucket(bucket)//桶
                    .filename(localFilePath) //指定本地文件路径
                    .object(filePath)// 文件名 放在子目录下
                    .contentType(mimeType)//设置媒体文件类型
                    .build();

            //上传文件
            minioClient.uploadObject(uploadObjectArgs);
            log.debug("上传文件到minio成功,bucket:{},objectName:{},错误信息:{}", bucket, filePath);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            log.error("上传文件出错,bucket:{},objectName:{},错误信息:{}", bucket, filePath, e.getMessage());
        }
        return false;
    }

    /**
     * 添加文件到数据库
     *
     * @param fileMd5             文件md5值
     * @param uploadFileParamsDto 上传文件的信息
     * @param bucket              桶
     * @param filePath            文件存储路径
     */
    @Transactional
    public MediaFiles addFilesToDb(String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String filePath) {
        //将文件信息保存到数据库
        LambdaQueryWrapper<MediaFiles> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(MediaFiles::getFileId, fileMd5);

        MediaFiles mediaFiles = mediaFilesMapper.selectOne(wrapper);
        if (mediaFiles == null) {
            mediaFiles = new MediaFiles();
            BeanUtils.copyProperties(uploadFileParamsDto, mediaFiles);
            // 文件路径
            mediaFiles.setFilePath(filePath);
            // 文件id
            mediaFiles.setFileId(fileMd5);
            // 文件地址
            mediaFiles.setUrl("http://192.168.31.32:9000" + "/" + bucket + "/" + filePath);
            //插入数据库
            mediaFiles.setId(null);

            int insert = mediaFilesMapper.insert(mediaFiles);
            if (insert <= 0) {
                log.debug("向数据库保存文件失败,bucket:{},objectName:{}", bucket, filePath);
                return null;
            }
            return mediaFiles;
        }
        return mediaFiles;
    }
}
```

### 前端分片上传

[React+TS 实现分片上传 - 掘金 (juejin.cn)](https://juejin.cn/post/7219651766706798629)
