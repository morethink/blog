---
title: Spring事务不回滚原因分析
date: 2017-05-14
tags: Spring
categories: Spring
---

Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。

在我完成一个项目的时候，遇到了一个Spring事务不回滚的问题，通过aspectJ和@Transactional注解都无法完成对于事务的回滚，经过查看博客和文档
1. 默认回滚RuntimeException
2. Service内部方法调用
3. Spring父子容器覆盖

代码已经上传到 https://github.com/morethink/transactional

<!-- more -->
# 异常
下面是@Transactional的注释文档，下面有个If no rules are relevant to the exception,it will be treated like {@link  org.springframework.transaction.interceptor.DefaultTransactionAttribute} (rolling back on runtime exceptions).
默认会使用`RuntimeException`,那为什么**Spring默认回滚RuntimeException**，因为Java把Exception分为两种。
1. checked Exception：Exception类本身，以及Exception的子类中除了"运行时异常"之外的其它子类都属于被检查异常。
2. unchecked Exception： RuntimeException和Error都属于未检查异常。

![](https://images.morethink.cn/java-exception.jpg)

```
/**
 * Describes transaction attributes on a method or class.
 *
 * <p>This annotation type is generally directly comparable to Spring's
 * {@link org.springframework.transaction.interceptor.RuleBasedTransactionAttribute}
 * class, and in fact {@link AnnotationTransactionAttributeSource} will directly
 * convert the data to the latter class, so that Spring's transaction support code
 * does not have to know about annotations. If no rules are relevant to the exception,
 * it will be treated like
 * {@link org.springframework.transaction.interceptor.DefaultTransactionAttribute}
 * (rolling back on runtime exceptions).
 *
 * <p>For specific information about the semantics of this annotation's attributes,
 * consult the {@link org.springframework.transaction.TransactionDefinition} and
 * {@link org.springframework.transaction.interceptor.TransactionAttribute} javadocs.
 *
 * @author Colin Sampaleanu
 * @author Juergen Hoeller
 * @author Sam Brannen
 * @since 1.2
 * @see org.springframework.transaction.interceptor.TransactionAttribute
 * @see org.springframework.transaction.interceptor.DefaultTransactionAttribute
 * @see org.springframework.transaction.interceptor.RuleBasedTransactionAttribute
 */
```

因此，如果发生的不是RuntimeException，而你有没有配置rollback for ，那么，异常就不会回滚。

# service 内部方法调用

**就是一个没有开启事务控制的方法调用一个开启了事务控制方法，不会事务回滚。**

**AccountService类**
```
@Service
public class AccountService {

    @Autowired
    private AccountDao accountDao;

    /**
     * 完成转钱业务,transfer方法开启事务
     *
     * @param out
     * @param in
     * @param money
     */
    @Transactional(isolation = Isolation.DEFAULT, propagation = Propagation.REQUIRED)
    public void transfer(String out, String in, double money) {
        Account account = new Account();
        account.setName(out);
        account.setMoney(money);
        accountDao.out(account);
        int i = 1 / 0;
        account.setName(in);
        accountDao.in(account);
    }

    /**
     * 完成转钱业务,transferProxy方法没有开启事务
     *
     * @param out
     * @param in
     * @param money
     */
    public void transferProxy(String out, String in, double money) {
        System.out.println("调用transfer方法 开始");
        transfer(out, in, money);
        System.out.println("调用transfer方法 结束");
    }
}
```
**AccountAction类**
```
@RestController
public class AccountAction {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/transfer")
    public String transfer(String out, String in, double money) {
        accountService.transfer(out, in, money);
        return "transfer";
    }

    @RequestMapping("/transferProxy")
    public String transferProxy(String out, String in, double money) {
        accountService.transferProxy(out, in, money);
        return "transfer";
    }
}
```
1. 通过transferProxy方法调用transfer方法时
    ```
    Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
    调用transfer方法 开始
    Creating a new SqlSession
    SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@a0dbf4] was not registered for synchronization because synchronization is not active
    JDBC Connection [com.mchange.v2.c3p0.impl.NewProxyConnection@6386ed [wrapping: com.mysql.jdbc.JDBC4Connection@9f2009]] will not be managed by Spring
    ==>  Preparing: update account set money = money - ? where name = ?
    ==> Parameters: 100.0(Double), aaa(String)
    <==    Updates: 1
    Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@a0dbf4]
    ```
    发现没有开启事务
2. 直接调用transfer方法时
    ```
    Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
    Creating a new SqlSession
    Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@238be2]
    JDBC Connection [com.mchange.v2.c3p0.impl.NewProxyConnection@a502e0 [wrapping: com.mysql.jdbc.JDBC4Connection@3dbe42]] will be managed by Spring
    ==>  Preparing: update account set money = money - ? where name = ?
    ==> Parameters: 100.0(Double), aaa(String)
    <==    Updates: 1
    Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@238be2]
    Transaction synchronization deregistering SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@238be2]
    Transaction synchronization closing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@238be2]
    ```
我们都知道Spring事务管理是通过AOP代理实现的，可是那么什么条件会使得AOP代理开启？通过查看
[Sprin官方文档](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#transaction)，发现只有把整个Service设为事务控制时，才会进行AOP代理。如果我们通过一个没有事务的transferProxy方法去调用有事务的transfer方法，是通过this引用进行调用，没有开启事务，即使发生了RuntimeException也不会回滚。

![](https://images.morethink.cn/tx.png)

然后


# Spring父子容器覆盖

Spring容器优先加载由ServletContextListener（对应applicationContext.xml）产生的父容器，而SpringMVC（对应mvc_dispatcher_servlet.xml）产生的是子容器。子容器Controller进行扫描装配时装配的@Service注解的实例是没有经过事务加强处理，即没有事务处理能力的Service，而父容器进行初始化的Service是保证事务的增强处理能力的。如果不在子容器中将Service exclude掉，此时得到的将是原样的无事务处理能力的Service，因为在多上下文的情况下，如果同一个bean被定义两次，后面一个优先。

当我们在applicationContext.xml,spring-mvc.xml都配置如下扫描包时，spring-mvc.xml中的service就会覆盖applicationContext.xml中的service。

```
context:component-scan base-package="net.morethink"/>
```

注意：当我们使用JUnit测试的时候，不会出现这种情况。
JUnit配置如下
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml", "classpath:dispatcher-servlet.xml"})
@WebAppConfiguration
public class AccountActionTest {


    protected MockMvc mockMvc;
    @Autowired
    protected WebApplicationContext wac;

    @Before()  //这个方法在每个方法执行之前都会执行一遍
    public void setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();  //初始化MockMvc对象
    }

    @Test
    public void testTransfer() throws Exception {
        String responseString = mockMvc.perform(
                get("/transfer")    //请求的url,请求的方法是get
                        .contentType(MediaType.APPLICATION_FORM_URLENCODED)  //数据的格式
                        .param("out", "aaa")
                        .param("in", "bbb")
                        .param("money", "100")
        ).andExpect(status().isOk())    //返回的状态是200
//                .andDo(print())         //打印出请求和相应的内容
                .andReturn().getResponse().getContentAsString();   //将相应的数据转换为字符串
        System.out.println(responseString);
    }

    @Test
    public void testTransferProxy() throws Exception {
        String responseString = mockMvc.perform(
                get("/transferProxy")    //请求的url,请求的方法是get
                        .contentType(MediaType.APPLICATION_FORM_URLENCODED)  //数据的格式
                        .param("out", "aaa")
                        .param("in", "bbb")
                        .param("money", "100")
        ).andExpect(status().isOk())    //返回的状态是200
//                .andDo(print())         //打印出请求和相应的内容
                .andReturn().getResponse().getContentAsString();   //将相应的数据转换为字符串
        System.out.println(responseString);
    }
}
```
**可能因为JUnit不会产生父子容器。**

**还有可能是其它配置文件出错，例如，连接池配置为多例**

参考文档
1. [Spring声明式事务为何不回滚](http://www.jianshu.com/p/f5fc14bde8a0)
2. [Spring中@Transactional事务回滚（含实例详细讲解，附源码）](http://www.importnew.com/19489.html)
2. [深入研究java.lang.ThreadLocal类](http://blog.csdn.net/c289054531/article/details/9187081)
3. [Spring单实例、多线程安全、事务解析](http://blog.csdn.net/c289054531/article/details/9196053)
