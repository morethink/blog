---
title: SpringMVC数据验证(AOP处理Errors和方法验证)
date: 2017-11-19
tags:
categories: SpringMVC
---

什么是JSR303？

JSR 303 – Bean Validation 是一个数据验证的规范，2009 年 11 月确定最终方案。
Hibernate Validator 是 Bean Validation 的最佳实践。

为什么使用JSR，松耦合，让业务代码的职责更加清晰。

松耦合就是职责更加清晰，每个人都有自己的职责，如果你的代码进行改动，我不用改动或者仅仅少量改动就可以发布和部署。
<!-- more -->

# 准备工作

## maven 配置

```xml
<!-- JSR 303 -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
<!-- Hibernate validator-->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.0.Final</version>
</dependency>
```
## SpringMVC 配置
```xml
<mvc:annotation-driven validator="validator">
</mvc:annotation-driven>
<!-- 配置校验器 -->
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <!-- 校验器，使用Hibernate校验器 -->
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
    <!-- 指定校验使用的资源文件，在文件中配置校验错误信息，如果不指定则默认使用classpath下面的ValidationMessages.properties文件， -->
    <property name="validationMessageSource" ref="messageSource"/>
</bean>
<!-- 校验错误信息配置文件，也可以不配置，直接使用注解中的message即可 -->
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <!-- 资源文件名 -->
    <property name="basenames">
        <list>
            <value>classpath:messageSource</value>
        </list>
    </property>
    <!-- 资源文件编码格式 -->
    <property name="fileEncodings" value="utf-8"/>
    <!-- 对资源文件内容缓存时间，单位秒 -->
    <property name="cacheSeconds" value="120"/>
</bean>
```

# 常用校验注解

| 注解 | 运行时检查 |
|------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| @AssertFalse | 被注解的元素必须为false |
| @AssertTrue | 被注解的元素必须为true |
| @DecimalMax(value) | 被注解的元素必须为一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin(Value) | 被注解的元素必须为一个数字，其值必须大于等于指定的最小值 |
| @Digits(integer=, fraction=) | 被注解的元素必须为一个数字，其值必须在可接受的范围内 |
| @Future | 被注解的元素必须是日期，检查给定的日期是否比现在晚 |
| @Max(value) | 被注解的元素必须为一个数字，其值必须小于等于指定的最大值 |
| @Min(value) | 被注解的元素必须为一个数字，其值必须大于等于指定的最小值 |
| @NotNull | 被注解的元素必须不为null |
| @Null | 被注解的元素必须为null |
| @Past(java.util.Date/Calendar) | 被注解的元素必须过去的日期，检查标注对象中的值表示的日期比当前早 |
| @Pattern(regex=, flag=) | 被注解的元素必须符合正则表达式，检查该字符串是否能够在match指定的情况下被regex定义的正则表达式匹配 |
| @Size(min=, max=) | 被注解的元素必须在制定的范围(数据类型:String, Collection, Map and arrays) |
| @Valid | 递归的对关联对象进行校验, 如果关联对象是个集合或者数组, 那么对其中的元素进行递归校验,如果是一个map,则对其中的值部分进行校验 |
| @CreditCardNumber | 对信用卡号进行一个大致的验证 |
| @Email | 被注释的元素必须是电子邮箱地址 |
| @Length(min=, max=) | 被注解的对象必须是字符串的大小必须在制定的范围内 |
| @NotBlank | 被注解的对象必须为字符串，不能为空，检查时会将空格忽略 |
| @NotEmpty | 被注释的对象必须不为空(数据:String,Collection,Map,arrays) |
| @Range(min=, max=) | 被注释的元素必须在合适的范围内 (数据：BigDecimal, BigInteger, String, byte, short, int, long and 原始类型的包装类 ) |
| @URL(protocol=, host=, port=, regexp=, flags=) | 被注解的对象必须是字符串，检查是否是一个有效的URL，如果提供了protocol，host等，则该URL还需满足提供的条件 |
|   |   |




# Bean验证

首先向我们的bean中添加注解。

```java
public class User {

    @NotEmpty(message = "用户名不为空")
    private String username;
    @NotEmpty(message = "密码不为空")
    private String password;
    // getter 和 setter
}
```

`Controller`中配置:

```java
@RestController
public class UserAction {

    @PostMapping("/login")
    public Result login(@Validated User user, Errors errors) {
        System.out.println(user);
        if (errors.hasErrors()) {
            return ResultUtil.messageResult(errors);
        }
        return ResultUtil.SUCCESS_RESULT;
    }
}
```

我们只需要在要校验的bean前面添加`@Validated`，在需要校验的bean后面添加`Errors`对象来接收校验出错信息即可，然后根据错误信息进行判断和返回错误信息给前端。

**注意**：
`@Validated` 和 `Errors errors` 是成对出现的，并且形参顺序是固定的（一前一后）。也就是所每一个`@Validated`后面必须跟一个`Errors`，需要验证多个bean，后面就跟多个`Errors`。

# AOP处理Errors

如果我们通过JSR来验证bean对象，那么在每个需要验证的方法中都需要处理Error对象，很容易想到可以通过AOP的方式来统一处理错误对象，并且组织错误信息，返回给前端。

通过一个环绕通知对所有的action方法尽心拦截，如果发现有Errors对象存在，就获取所有的错误信息，封装为一个list返回前端。

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.validation.BindingResult;
import org.springframework.validation.Errors;
import org.springframework.validation.ObjectError;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 李文浩
 * @version 2017/10/8.
 */
public class ValidationAdvice {

    /**
     * 切点处理
     *
     * @param pjp
     * @return
     * @throws Throwable
     */
    public Object aroundMethod(ProceedingJoinPoint pjp) throws Throwable {
        Errors errors = null;
        Object[] args = pjp.getArgs();
        if (null != args && args.length != 0) {
            for (Object object : args) {
                if (object instanceof BindingResult) {
                    errors = (BindingResult) object;
                    break;
                }
            }
        }
        if (errors != null && errors.hasErrors()) {
            List<ObjectError> allErrors = errors.getAllErrors();
            List<String> messages = new ArrayList<String>();
            for (ObjectError error : allErrors) {
                messages.add(error.getDefaultMessage());
            }
            return ResultUtil.messageResult(messages);
        }
        return pjp.proceed();
    }

}
```


Spring配置：


```xml
<bean id="validationAdvice" class="studio.jikewang.util.ValidationAdvice" />
<aop:config>
    <aop:pointcut id="validation1" expression="execution(public * studio.jikewang.action.*.*(..))" />
    <aop:aspect id="validationAspect" ref="validationAdvice">
        <aop:around method="aroundMethod" pointcut-ref="validation1" />
    </aop:aspect>
</aop:config>
```



# `@Validated` 和 `@Valid`

- `@Valid`是使用Hibernate Validation的时候使用。
  Java的JSR303声明了这类接口，然后hibernate－validator对其进行了实现。
- `@Validated`是只用Spring Validator校验机制使用。


# 方法参数验证

Spring提供了`MethodValidationPostProcessor`类，用于对方法的校验。

`Controller`中配置:

```java
@RestController
@Validated
public class UserAction {

    @PostMapping("/login")
    public Result login(@NotEmpty(message = "用户名不为空") String username,
                        @NotEmpty(message = "密码不为空")  String password) {
        return ResultUtil.SUCCESS_RESULT;
    }
}
```


xml配置(**最好是配置在`<mvc:annotation-driven validator="validator" />`上面，不然会有未知错误**)如下：

```xml
<bean  class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
</bean>
```

在校验遇到非法的参数时会抛出`ConstraintViolationException`，可以通过`getConstraintViolations`获得所有没有通过的校验`ConstraintViolation`集合，可以通过它们来获得对应的消息。

我们同样使用 `@ExceptionHandler` 捕捉`ConstraintViolationException`异常处理全局异常信息。

然后将所有的错误信息封装好返回给前端。

```java
@RestControllerAdvice
public class MyExceptionHandler {
       @ExceptionHandler(ConstraintViolationException.class)
    public Result handleConstraintViolationException(ConstraintViolationException e) {
        List<String> list = new ArrayList<String>();
        for (ConstraintViolation<?> s : e.getConstraintViolations()) {
            System.out.println(s.getInvalidValue() + ": " + s.getMessage());
            list.add(s.getMessage());
        }
        Result result = new Result();
        result.setStatus("0");
        result.setMessage(list);
        return result;
    }
}
```
# 使用`@Validated`验证list

现在我遇到一个新的需求，我需要前端给我传递一个对象数组，于是我使用一个list去接收，但是无法获得验证信息。

于是将list重新包装一下。

```java
public class ValidList<E> {

    @Valid
    private List<E> list;

    public List<E> getList() {
        return list;
    }

    public void setList(List<E> list) {
        this.list = list;
    }
}
```

`Controller`中配置:

```java
@RestController
public class UserAction {

    @PostMapping("/login")
    public Result login(@Validated ValidList<User> users, Errors errors) {
        System.out.println(users.getList());
        if (errors.hasErrors()) {
            return ResultUtil.messageResult(errors);
        }
        return ResultUtil.SUCCESS_RESULT;
    }
}
```

然后为了只返回第一个验证失败的信息(如果不更改，就会将所有的出错信息返回给前端)，更改`ValidationAdvice`如下：

```java
public class ValidationAdvice {

    /**
     * 切点处理
     *
     * @param pjp
     * @return
     * @throws Throwable
     */
    public Object aroundMethod(ProceedingJoinPoint pjp) throws Throwable {
        Object[] args = pjp.getArgs();
        boolean isValidList = false;
        Errors errors = null;
        if (null != args && args.length != 0) {
            for (Object object : args) {
                if (object instanceof ValidList) {
                    isValidList = true;
                }
                if (object instanceof BindingResult) {
                    errors = (BindingResult) object;
                    break;
                }
            }
        }
        if (errors != null && errors.hasErrors()) {
            List<ObjectError> allErrors = errors.getAllErrors();
            List<String> messages = new ArrayList<String>();
            for (ObjectError error : allErrors) {
                if (isValidList) {
                    messages.add(error.getDefaultMessage());
                    break;
                } else {
                    messages.add(error.getDefaultMessage());
                }
            }
            return ResultUtil.messageResult(messages);
        }
        return pjp.proceed();
    }
}
```

这样即可验证`list`。

# 总结

AOP的思想是贯穿我们的开发的，使用AOP的思想可以大大提高我们的开发效率，减少重复代码。


**参考文档**：
1. [springmvc参数校验-JSR303(Bean Validation）](http://www.jianshu.com/p/fc6c20af759a)
2. [Java Bean Validation 最佳实践](http://www.cnblogs.com/beiyan/p/5946345.html)
