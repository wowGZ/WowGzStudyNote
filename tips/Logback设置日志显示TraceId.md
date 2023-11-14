# Logback设置日志显示TraceId

## 问题场景： 

logback可以通过配置在日志中显示TraceId来记录相关日志流水。

试想如下场景：当你要通过一个接口的代码执行过程中找到你所需要的日志时，你是不是一个一个通过代码日志中的关键字然后一次一次的在海量的日志中检索。

如果是这样的话，我们将花费过量的时间去查找日志排查问题。

那么如果你的日志如下图所示，就可以通过检索traceId来筛选出一个完整流程中你所打印的所有日志。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22461487/1654595687717-a26679c4-8283-413e-a93d-19a12165be4b.png)



## 处理前准备

 

目前只研究了logback的配置方式，其他的日志框架可以自行了解一下

需要准备：

●logback.xml的配置文件

●一个ID生成器的工具类

●一个注解

●一个切面

代码可参考如下：

ID生成器，可以自己实现一个雪花算法的ID生成器也可

```java
import com.alibaba.fastjson2.JSONObject;
import com.cdeledu.domain.ServiceResult;
import lombok.extern.slf4j.Slf4j;

/**
 * @ClassName IdCenterUtil
 * @Author wowgz
 * @Date 2022/6/6 9:52
 * @Description 调取id生成器获取唯一id
 */
@Slf4j
public class IdCenterUtil {

    private static final String default_namespace = "cdelPay";

    private static final String createId_url = "http://";

    public static Long createId() {
        return createId(null);
    }

    public static Long createId(String nameSpace){
        try{
            nameSpace = nameSpace == null ? IdCenterUtil.default_namespace : nameSpace;
            String resultStr = OkHttpUtil.getInstance().postJson(IdCenterUtil.createId_url + nameSpace, "");
            ServiceResult<Long> result = JSONObject.parseObject(resultStr, ServiceResult.class);
            if(!result.isSuccess()){
                return -1L;
            }
            return result.getResult();
        }catch(Exception e){
            log.error("Exception happened while post to IdCenter service - Exception - {} - Stack Trace - {}",
                    e.getMessage(), e.getStackTrace());
            return -1L;
        }
    }
}
```

注解

```java
import java.lang.annotation.*;

@Target({ ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TraceLog {
}
```

切面

```java
import com.cdel.pay.util.IdCenterUtil;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.slf4j.MDC;
import org.springframework.stereotype.Component;
import org.aspectj.lang.annotation.Aspect;

/**
 * @ClassName TraceLogAspect
 * @Author Guo Zhen
 * @Date 2022/6/7 11:43
 * @Description 生成日志traceID，用于做日志追踪
 */
@Aspect
@Component
@Slf4j
public class TraceLogAspect {

    /**
     * 日志前缀
     */
    private static final String logPrefix = "[Trace Id] - ";

    /**
     * trace id
     */
    private static final String TRACE_ID = "traceId";

    @Around("@annotation(com.cdel.pay.annotation.TraceLog)")
    public Object trace(ProceedingJoinPoint joinPoint) {

        // 生成并设置id到当前线程
        if (null == MDC.get(TRACE_ID)) {
            String traceId = logPrefix + IdCenterUtil.createId() + " - ";
            MDC.put(TRACE_ID, traceId);
        }
        Object result = null;
        try {
            result = joinPoint.proceed();
        } catch (Throwable e) {
            log.error("[Trace Id] - {} - [Exception] - {} - [Stack Trace] - {}",
                    MDC.get(TRACE_ID), e.getMessage(), e.getStackTrace());
        }
        // 移除 id
        MDC.remove(TRACE_ID);
        return result;
    }

}
```

## 处理和使用

 

1. 调整logback.xml的配置文件

如图：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22461487/1654596099802-87cdcc3b-d740-4aa3-ba83-83a62f21dd50.png)

原有的日志格式是

> %d %p - %C[%L] - [%t] - %m %n

可以在%m前加上%mdc{traceId}

> %d %p - %C[%L] - [%t] - %mdc{traceId}%m %n

关于logback的通配符可以参考官方文档 https://logback.qos.ch/manual/layouts.html

2. 使用，只需要在你所需要的方法上标注之前准备的注解即可，如图：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22461487/1654596360194-52d41b9e-a78e-43c8-918e-650eafc8adae.png)