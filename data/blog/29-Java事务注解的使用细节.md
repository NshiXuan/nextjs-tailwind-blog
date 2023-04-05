---
title: 'Java事务注解的使用细节' # 标题
date: '2023/4/5' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java事务注解的使用细节'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2ff2a0330b94d4d935cd2cc26a1b4f5~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

在 Java 中经常遇到一个非事务方法调同类一个事务方法，事务无法控制，这是为什么呢？

- 因为事务是通过代理加 aop 实现的

如下：为什么不把事务放在 `uploadFile` 方法上，因为 `addMediaFilesToMinIO` 需要通过网络连接 `minio` ，如果网络不好连不上，事务就会占据数据库比较长的时间，这是没有必要的，所以我们把事务放在操作数据库的方法 `addMediaFilesToDb` 上

那么下面在 `uploadFile` 调用 `addMediaFilesToDb` 方法，事务能实现吗，不能，因为事务本身是通过代理对象来调用的，而 `MediaFileServiceImpl` 本身 `this` 并不是代理对象

```java
...
import org.springframework.transaction.annotation.Transactional;

@Service
public class MediaFileServiceImpl implements MediaFileService {

    @Override
    public UploadFileResultDto uploadFile(Long companyId, UploadFileParamsDto uploadFileParamsDto, String localFilePath) {
        ...
        // 6.上传文件到minio
        boolean result = addMediaFilesToMinIO(localFilePath, mimeType, bucket_mediafiles, objectName);
        ...
        // 7.将文件信息保存到数据库
        MediaFiles mediaFiles = addMediaFilesToDb(companyId, fileMd5, uploadFileParamsDto, bucket_mediafiles, objectName);
        // 相当于this.addMediaFilesToDb()
        ...
    }


    /**
     * @description 将文件信息添加到文件表
     */
    @Transactional
    public MediaFiles addMediaFilesToDb(Long companyId, String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String objectName) {
            mediaFiles = new MediaFiles();

            ...映射数据

            //插入数据库
            int insert = mediaFilesMapper.insert(mediaFiles);
            ...
    }
}
```

所以，我们要怎么使上面的方法实现事务呢？注入本身即可

- 在 `service` 中定义 `addMediaFilesToDb` 方法，使注入自己的 `service` 能够调用

```java
public interface MediaFileService {
    public UploadFileResultDto uploadFile(Long companyId, UploadFileParamsDto uploadFileParamsDto, String localFilePath);

    public MediaFiles addMediaFilesToDb(Long companyId, String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String objectName);
}
```

- 在 `impl` 中使用实现事务，用 `currentProxy` 调用 `addMediaFilesToDb` 即可

```java
...
import org.springframework.transaction.annotation.Transactional;

@Service
public class MediaFileServiceImpl implements MediaFileService {
    // 注入自己实现代理，springboot注入会自动给我们包裹代理
    @Autowired
    MediaFileService currentProxy;


    @Override
    public UploadFileResultDto uploadFile(Long companyId, UploadFileParamsDto uploadFileParamsDto, String localFilePath) {
        ...
        // 7.将文件信息保存到数据库
        MediaFiles mediaFiles = currentProxy.addMediaFilesToDb(companyId, fileMd5, uploadFileParamsDto, bucket_mediafiles, objectName);
        ...
    }

    @Transactional
    public MediaFiles addMediaFilesToDb(Long companyId, String fileMd5, UploadFileParamsDto uploadFileParamsDto, String bucket, String objectName) {
            mediaFiles = new MediaFiles();

            ...映射数据

            //插入数据库
            int insert = mediaFilesMapper.insert(mediaFiles);
            ...
    }
}
```
