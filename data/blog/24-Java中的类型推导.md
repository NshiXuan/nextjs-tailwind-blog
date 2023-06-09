---
title: 'Java中的类型推导与定义返回模型R' # 标题
date: '2023/3/29' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java中的类型推导与定义返回模型R'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2ff2a0330b94d4d935cd2cc26a1b4f5~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

泛型是 `Java` 中常用的功能，同样通过参数推导出类型也是常用的，那么什么时候能够通过参数来推导出类型呢？

- 如 `public R<T> success(String msg, T data)` ，该方法没有定义为泛型方法，`T` 是没有被定义，编译器无法推导出它的类型
- 如 `public <T> R<T> success(String msg, T data)`，第一个 `<T>` 表示这是一个泛型方法，此时通过参数就可以推导出 `data` 的类型并赋给 `T`

练习：封装接口返回值类型 `R`

```java
import com.minio.exception.Code;
import lombok.Data;

import java.io.Serializable;

@Data
public class R<T> implements Serializable {
    private Integer code; // 0代表成功 -1代表失败
    private T data;
    private String msg;

    public R() {
    }

    public R(Integer code, T data, String msg) {
        this.code = code;
        this.data = data;
        this.msg = msg;
    }

    public R(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public static <T> R<T> success() {
        return new R<>(0, "success");
    }

    public static <T> R<T> success(T data) {
        return new R<>(0, data, "success");
    }

    public static <T> R<T> success(String msg, T data) {
        return new R<>(0, data, msg);
    }

    public static <T> R<T> success(Code code) {
        return new R<>(code.getCode(), code.getMsg());
    }

    public static <T> R<T> success(Code code, T data) {
        return new R<>(code.getCode(), data, code.getMsg());
    }

    public static <T> R<T> error(String msg) {
        return new R<>(-1, msg);
    }

    public static <T> R<T> error(Code code) {
        return new R<>(code.getCode(), code.getMsg());
    }

    public static <T> R<T> error(Integer code, String msg) {
        return new R<>(code, msg);
    }
}
```

- `Code`

```java
public enum Code {
    SERVICE_BUSY(500, "服务繁忙"), // 枚举分隔不能使用; 需要使用,
    UPLOAD_ERROR(1001, "上传失败");

    private Integer code;
    private String msg;

    private Code(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public Integer getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```
