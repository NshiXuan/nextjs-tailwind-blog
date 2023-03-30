---
title: 'Java自定义异常抛出并拦截返回给客户端' # 标题
date: '2023/3/30' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java自定义异常抛出并拦截返回给客户端'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2ff2a0330b94d4d935cd2cc26a1b4f5~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

在 `Java` 开发中，如果前端传输的数据不符合要求时，我们就需要抛出异常并返回给前端，这里教大家如何自定义异常并拦截返回给前端

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02e2c237838a42e59b8242cfa143d84c~tplv-k3u1fbpfcp-watermark.image?)

1.用枚举封装返回给前端的状态码 `Code`

```java
public enum Code {
    SERVICE_BUSY(500, "服务繁忙");

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

2. 创建自定义异常类 `MyException`

```java
public class MyException extends RuntimeException {
    private Code commonErr;

    public MyException() {
    }

    public MyException(Code code) {
        this.commonErr = code;
    }

    public Code getCommonErr() {
        return commonErr;
    }

    public void setCommonErr(Code commonErr) {
        this.commonErr = commonErr;
    }

    public static void cast(Code commonErr) {
        throw new MyException(commonErr);
    }
}
```

3. 创建全局拦截异常的类 `GlobalExceptionHandler`

- `@ControllerAdvice`注解在 Java 中实现全局异常处理
- `@ResponseBody`注解用于指示 Spring 将方法返回的对象转换为 HTTP 响应正文
- `@ControllerAdvice` 与 `@ResponseBody` 结合一起使用
- `@RestControllerAdvice` 是 `@ControllerAdvice` 与 `@ResponseBody` 结合，使用了它就不需要再使用 `@ResponseBody` 了
- `@ExceptionHandler(MyException.class)`  表示该方法用于处理  `MyException`  类型的异常。当系统抛出该类型的异常时，该方法将会被调用，并且可以对异常进行处理
- `R` 的封装这里就不展示了，可以看往期的文章

```java
import com.xuecheng.base.model.R;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Slf4j
//@ControllerAdvice
@RestControllerAdvice
public class GlobalExceptionHandler {
    // @ResponseBody
    @ExceptionHandler(MyException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public R myException(MyException e) {
        //记录异常
        log.error("系统异常 --- ", e.getCommonErr().getMsg(), e);

        return R.error(e.getCommonErr().getCode(), e.getCommonErr().getMsg());
    }
}
```

4. 在 `controller` 抛出测试

```java
@RestController
public class TestController {
    @GetMapping("/test")
    public void test() {
        MyException.cast(Code.SERVICE_BUSY);
    }
}
```

结果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7858e4328b724511b811dfd26b8e4718~tplv-k3u1fbpfcp-watermark.image?)
