---
title: 'Java的JSR303参数校验' # 标题
date: '2023/3/29' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java的JSR303参数校验'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2ff2a0330b94d4d935cd2cc26a1b4f5~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

---

theme: channing-cyan
highlight: a11y-dark

---

## 安装依赖

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7db18c8658140b6826c0383b9ad6686~tplv-k3u1fbpfcp-watermark.image?)

## 常用注解

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/349a9540f93645a48408641215698770~tplv-k3u1fbpfcp-watermark.image?)

## 基本使用

在 `dto` 中使用注解

```java
import lombok.Data;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.Size;

@Data
public class AddCourseDto {
    @NotEmpty(message = "课程名称不能为空")
    private String name;

    @NotEmpty(message = "适用人群不能为空")
    @Size(message = "适用人群内容过少", min = 10)
    private String users;

    @NotEmpty(message = "课程分类不能为空")
    private String mt;

    @NotEmpty(message = "课程分类不能为空")
    private String st;

    @NotEmpty(message = "课程等级不能为空")
    private String grade;

    @NotEmpty(message = "收费规则不能为空")
    private String charge;
    ...
}
```

创建全局异常处理类 `GlobalExceptionHandler` 与异常返回类 `RestErrorResponse`

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98d052d34d314f4bbf0c631a4d042d22~tplv-k3u1fbpfcp-watermark.image?)

- `@ControllerAdvice`注解在 Java 中实现全局异常处理
- `@ResponseBody`注解用于指示 Spring 将方法返回的对象转换为 HTTP 响应正文
- `@ControllerAdvice` 与 `@ResponseBody` 结合一起使用
- `@RestControllerAdvice` 是 `@ControllerAdvice` 与 `@ResponseBody` 结合，使用了它就不需要再使用 `@ResponseBody` 了
- `@ExceptionHandler(MethodArgumentNotValidException.class)`  表示该方法用于处理  `MethodArgumentNotValidException`  类型的异常。当系统抛出该类型的异常时，该方法将会被调用，并且可以对异常进行处理

```java
package com.xuecheng.base.exception;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.http.HttpStatus;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import java.util.ArrayList;
import java.util.List;

@Slf4j
@ControllerAdvice
//@RestControllerAdvice
public class GlobalExceptionHandler {
    ...

    @ResponseBody
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public RestErrorResponse methodArgumentNotValidException(MethodArgumentNotValidException e) {

        BindingResult bindingResult = e.getBindingResult();
        // 获取错误的文本字段 为一个数组
        List<String> errors = new ArrayList<>();
        bindingResult.getFieldErrors().stream().forEach(item -> {
            errors.add(item.getDefaultMessage());
        });

        // 将List的信息拼接
        String errMessage = StringUtils.join(errors, ",");

        // 使@Slf4j的log输出异常
        log.error("系统异常{}", e.getMessage(), errMessage);

        // 返回异常信息
        return R.error(1, errMessage);
    }
}
```

错误返回类 `R`

```java
package com.xuecheng.base.model;

import lombok.Data;

import java.io.Serializable;

@Data
public class R<T> implements Serializable {
    private Integer code;
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

    public static <T> R<T> ok(T data) {
        return new R(0, data, "success");
    }

    public static <T> R<T> success(String msg, T data) {
        return new R(0, data, msg);
    }

    public static <T> R<T> error(Integer code, String msg) {
        return new R(code, msg);
    }
}
```

在 `controller` 中校验参数

- 使用 `@Validated` 注解开启校验

```java
@PostMapping("/course")
public CourseBaseInfoDto createCourseBase(@RequestBody @Validated AddCourseDto addCourseDto) {
    ...
}
```

参数错误结果如下

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/051184cdf7ce47f59bd4bb62aa4bc681~tplv-k3u1fbpfcp-watermark.image?)

## 分组

当我们有多个接口需要同一个 `dto` 时，且 `dto` 的校验信息是不一样的，这时候就需要分组功能了，如下

创建一个分组的类

```java
package com.xuecheng.base.exception;

public class ValidationGroups {
    public interface Inster {
    }


    public interface Update {
    }
}
```

在 `dto` 中使用

```java
@Data
public class AddCourseDto {
    @NotEmpty(message = "新增课程名称不能为空", groups = {ValidationGroups.Inster.class})
    @NotEmpty(message = "修改课程名称不能为空", groups = {ValidationGroups.Update.class})
    private String name;

    ...
}
```

在 `controller` 中使用

```java
@PostMapping("/course")
public CourseBaseInfoDto createCourseBase(@RequestBody
@Validated(ValidationGroups.Update.class) AddCourseDto addCourseDto) {
    ...
}
```

测试结果

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c0af0ef99d24a339cf9316f6addb8e2~tplv-k3u1fbpfcp-watermark.image?)
