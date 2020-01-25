---
title: Netty URL路由方案探讨
date: 2018-09-27
tags: Netty
categories: Java
---

最近在用Netty做开发，需要提供一个http web server，供调用方调用。采用Netty本身提供的`HttpServerCodec` handler进行Http协议的解析，但是需要自己提供路由。

最开始是通过对Http method及uri 采用多层if else 嵌套判断的方法路由到真正的controller类：
```Java
String uri = request.uri();
HttpMethod method = request.method();
if (method == HttpMethod.POST) {
    if (uri.startsWith("/login")) {
        //url参数解析，调用controller的方法
    } else if (uri.startsWith("/logout")) {
        //同上
    }
} else if (method == HttpMethod.GET) {
    if (uri.startsWith("/")) {

    } else if (uri.startsWith("/status")) {

    }
}
```
<!-- more -->

在只需提供`login`及`logout`API时，代码可以完成功能，可是随着API的数量越来越多，需要支持的方法及uri越来越多，`else if` 越来越多，代码越来越复杂。

![time-for-change](https://images.morethink.cn/time-for-change.jpg "是时候考虑重构了")

在阿里开发手册中也提到过：

![](https://images.morethink.cn/e1a6f2bd638f0baf5aba0d7a63e77230.png "重构多层else if")

因此首先考虑采用状态设计模式及策略设计模式重构。

# 状态模式

## 状态模式的角色：

- state状态
表示状态，定义了根据不同状态进行不同处理的接口，该接口是那些处理内容依赖于状态的方法集合，对应实例的state类
- 具体的状态
实现了state接口，对应daystate和nightstate
- context
context持有当前状态的具体状态的实例，此外，他还定义了供外部调用者使用的状态模式的接口。

首先我们知道每个http请求都是由method及uri来唯一标识的，所谓路由就是通过这个唯一标识定位到controller类的中的某个方法。

因此把HttpLabel作为状态
```Java
@Data
@AllArgsConstructor
public class HttpLabel {
    private String uri;
    private HttpMethod method;
}

```

状态接口：
```Java
public interface Route {
    /**
     * 路由
     *
     * @param request
     * @return
     */
    GeneralResponse call(FullHttpRequest request);
}
```
为每个状态添加状态实现：
```java
public void route() {
    //单例controller类
    final DemoController demoController = DemoController.getInstance();
    Map<HttpLabel, Route> map = new HashMap<>();
    map.put(new HttpLabel("/login", HttpMethod.POST), demoController::login);
    map.put(new HttpLabel("/logout", HttpMethod.POST), demoController::login);
}
```

接到请求，判断状态，调用不同接口：
```java
public class ServerHandler extends SimpleChannelInboundHandler<FullHttpRequest> {
    @Override
    public void channelRead0(ChannelHandlerContext ctx, FullHttpRequest request) {
        String uri = request.uri();
        GeneralResponse generalResponse;
        if (uri.contains("?")) {
            uri = uri.substring(0, uri.indexOf("?"));
        }
        Route route = map.get(new HttpLabel(uri, request.method()));
        if (route != null) {
            ResponseUtil.response(ctx, request, route.call(request));
        } else {
            generalResponse = new GeneralResponse(HttpResponseStatus.BAD_REQUEST, "请检查你的请求方法及url", null);
            ResponseUtil.response(ctx, request, generalResponse);
        }
    }
}
```

使用状态设计模式重构代码，在增加url时只需要网map里面put一个值就行了。

# Netty实现类似SpringMVC路由

后来看了 [JAVA反射+运行时注解实现URL路由](https://yuerblog.cc/2018/03/08/java-router-with-annotation/) 发现反射+注解的方式很优雅，代码也不复杂。

下面介绍Netty使用反射实现URL路由。

**路由注解**：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestMapping {
    /**
     * 路由的uri
     *
     * @return
     */
    String uri();

    /**
     * 路由的方法
     *
     * @return
     */
    String method();
}
```

`json格式的body`：
```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestBody {

}
```
**异常类(用于全局异常处理,实现 `@ControllerAdvice` 异常处理)**：

```java
@Data
public class MyRuntimeException extends RuntimeException {

    private GeneralResponse generalResponse;

    public MyRuntimeException(String message) {
        generalResponse = new GeneralResponse(HttpResponseStatus.INTERNAL_SERVER_ERROR, message);
    }

    public MyRuntimeException(HttpResponseStatus status, String message) {
        generalResponse = new GeneralResponse(status, message);
    }

    public MyRuntimeException(GeneralResponse generalResponse) {
        this.generalResponse = generalResponse;
    }

    @Override
    public synchronized Throwable fillInStackTrace() {
        return this;
    }
}

```

扫描classpath下带有`@RequestMapping`注解的方法，将这个方法放进一个路由Map：`Map<HttpLabel, Action<GeneralResponse>> httpRouterAction`，key为上面提到过的Http唯一标识  `HttpLabel`，value为通过反射调用的方法：
```java
@Slf4j
public class HttpRouter extends ClassLoader {

    private Map<HttpLabel, Action<GeneralResponse>> httpRouterAction = new HashMap<>();

    private String classpath = this.getClass().getResource("").getPath();

    private Map<String, Object> controllerBeans = new HashMap<>();

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String path = classpath + name.replaceAll("\\.", "/");
        byte[] bytes;
        try (InputStream ins = new FileInputStream(path)) {
            try (ByteArrayOutputStream out = new ByteArrayOutputStream()) {
                byte[] buffer = new byte[1024 * 5];
                int b = 0;
                while ((b = ins.read(buffer)) != -1) {
                    out.write(buffer, 0, b);
                }
                bytes = out.toByteArray();
            }
        } catch (Exception e) {
            throw new ClassNotFoundException();
        }
        return defineClass(name, bytes, 0, bytes.length);
    }

    public void addRouter(String controllerClass) {
        try {
            Class<?> cls = loadClass(controllerClass);
            Method[] methods = cls.getDeclaredMethods();
            for (Method invokeMethod : methods) {
                Annotation[] annotations = invokeMethod.getAnnotations();
                for (Annotation annotation : annotations) {
                    if (annotation.annotationType() == RequestMapping.class) {
                        RequestMapping requestMapping = (RequestMapping) annotation;
                        String uri = requestMapping.uri();
                        String httpMethod = requestMapping.method().toUpperCase();
                        // 保存Bean单例
                        if (!controllerBeans.containsKey(cls.getName())) {
                            controllerBeans.put(cls.getName(), cls.newInstance());
                        }
                        Action action = new Action(controllerBeans.get(cls.getName()), invokeMethod);
                        //如果需要FullHttpRequest，就注入FullHttpRequest对象
                        Class[] params = invokeMethod.getParameterTypes();
                        if (params.length == 1 && params[0] == FullHttpRequest.class) {
                            action.setInjectionFullhttprequest(true);
                        }
                        // 保存映射关系
                        httpRouterAction.put(new HttpLabel(uri, new HttpMethod(httpMethod)), action);
                    }
                }
            }
        } catch (Exception e) {
            log.warn("{}", e);
        }
    }

    public Action getRoute(HttpLabel httpLabel) {
        return httpRouterAction.get(httpLabel);
    }
}
```
通过反射调用`controller` 类中的方法：
```java
@Data
@RequiredArgsConstructor
@Slf4j
public class Action {
    @NonNull
    private Object object;
    @NonNull
    private Method method;

    private List<Class> paramsClassList;

    public GeneralResponse call(Object... args) {
        try {
            return (GeneralResponse) method.invoke(object, args);
        } catch (InvocationTargetException e) {
            Throwable targetException = e.getTargetException();
            //实现 `@ControllerAdvice` 异常处理，直接抛出自定义异常
            if (targetException instanceof MyRuntimeException) {
                return ((MyRuntimeException) targetException).getGeneralResponse();
            }
            log.warn("method invoke error: {}", e);
            return new GeneralResponse(HttpResponseStatus.INTERNAL_SERVER_ERROR, String.format("Internal Error: %s", ExceptionUtils.getRootCause(e)), null);
        } catch (IllegalAccessException e) {
            log.warn("method invoke error: {}", e);
            return new GeneralResponse(HttpResponseStatus.INTERNAL_SERVER_ERROR, String.format("Internal Error: %s", ExceptionUtils.getRootCause(e)), null);
        }
    }
}
```

`ServerHandler.java`处理如下：
```java
public void channelRead0(ChannelHandlerContext ctx, FullHttpRequest request) {
    String uri = request.uri();
    GeneralResponse generalResponse;
    if (uri.contains(DELIMITER)) {
        uri = uri.substring(0, uri.indexOf(DELIMITER));
    }
    //根据不同的请求API做不同的处理(路由分发)
    Action action = httpRouter.getRoute(new HttpLabel(uri, request.method()));
    if (action != null) {
        String s = request.uri();
        if (request.headers().get(HttpHeaderNames.CONTENT_TYPE.toString()).equals(HttpHeaderValues.APPLICATION_X_WWW_FORM_URLENCODED.toString())) {
            s = s + "&" + request.content().toString(StandardCharsets.UTF_8);
        }
        QueryStringDecoder queryStringDecoder = new QueryStringDecoder(s);
        Map<String, List<String>> parameters = queryStringDecoder.parameters();
        Class[] classes = action.getMethod().getParameterTypes();
        Object[] objects = new Object[classes.length];
        for (int i = 0; i < classes.length; i++) {
            Class c = classes[i];
            //处理@RequestBody注解
            Annotation[] parameterAnnotation = action.getMethod().getParameterAnnotations()[i];
            if (parameterAnnotation.length > 0) {
                for (int j = 0; j < parameterAnnotation.length; j++) {
                    if (parameterAnnotation[j].annotationType() == RequestBody.class &&
                            request.headers().get(HttpHeaderNames.CONTENT_TYPE.toString()).equals(HttpHeaderValues.APPLICATION_JSON.toString())) {
                        objects[i] = JsonUtil.fromJson(request, c);
                    }
                }
                //处理数组类型
            } else if (c.isArray()) {
                String paramName = action.getMethod().getParameters()[i].getName();
                List<String> paramList = parameters.get(paramName);
                if (CollectionUtils.isNotEmpty(paramList)) {
                    objects[i] = ParamParser.INSTANCE.parseArray(c.getComponentType(), paramList);
                }
            } else {
                //处理基本类型和string
                String paramName = action.getMethod().getParameters()[i].getName();
                List<String> paramList = parameters.get(paramName);
                if (CollectionUtils.isNotEmpty(paramList)) {
                    objects[i] = ParamParser.INSTANCE.parseValue(c, paramList.get(0));
                } else {
                    objects[i] = ParamParser.INSTANCE.parseValue(c, null);
                }
            }
        }
        ResponseUtil.response(ctx, HttpUtil.isKeepAlive(request), action.call(objects));
    } else {
        //错误处理
        generalResponse = new GeneralResponse(HttpResponseStatus.BAD_REQUEST, "请检查你的请求方法及url", null);
        ResponseUtil.response(ctx, HttpUtil.isKeepAlive(request), generalResponse);
    }
}
```

`DemoController` 方法配置：
```java
@RequestMapping(uri = "/login", method = "POST")
    public GeneralResponse login(@RequestBody User user, FullHttpRequest request,
                                 String test, Integer test1, int test2,
                                 long[] test3, Long test4, String[] test5, int[] test6) {
        System.out.println(test2);
        log.info("/login called,user: {} ,{} ,{} {} {} {} {} {} {} {} ", user, test, test1, test2, test3, test4, test5, test6);
        return new GeneralResponse(null);
    }
```

测试结果如下：

![](https://images.morethink.cn/774f6db42a43417eac2bd893ed6b657e.png "发送信息")

`netty-route` 得到结果如下:

```shell
user=User(username=hah, password=dd),test=111,test1=null,test2=0,test3=[1],test4=null,test5=[d,a, 1],test6=[1, 2]
```
完整代码在 https://github.com/morethink/Netty-Route
