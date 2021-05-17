# SSM框架：

# 一、Spring：

## 1、Spring基本介绍：

### 1.1、Spring简介：

- Spring是分层javaSE/EE应用full_stack轻量级开源框架，一**ioc**（反转控制）和**aop**（面向切面编程）为内核。
- Spring提供了展现层（Spring-MVC）和持久层（Spring-JDBCTemplate）以及业务层事务管理等众多企业级应用技术，还整合了开源世界众多著名的第三方框架和类库，逐渐成为使用最多的javaEE企业应用开源框架。

### 1.2、Spring的优势：

- 方便解耦，简化开发：通过Spring提供的ioc容器，可以将对象间的依赖关系交给Spring来处理，避免硬编程带来的过多耦合。用户不再为单例模式类、属性文件解析等很底层的需求编写代码，可以更加专注于上层的应用。
- Aop编程的支持：通过Spring的Aop功能，方便进行Aop编程许多不容易用oop实现的功能可以轻松通过aop实现
- 声明式事物的支持：可以将我们从繁琐的事务管理中解放出来，通过声明式事务管理，提高开发效率和质量
- 方便程序的测试：继承了Junit单元测试，方便代码测试。
- 方便集成优秀的框架：Spring对各种优秀框架（struts、Hibernate、Hessian、Quartz）的支持。
- 降低javaEE API的使用难度。
- Spring源码是经典学习模范。

### 1.3、Spring的体系结构：

​	![image-20210513224128024](SSM框架.assets/image-20210513224128024.png)

## 2、Spring快速入门：

### 2.1、Spring程序开发步骤：

- 导入Spring开发基本包坐标。

- 编写Dao接口和实现类。

- 创建Spring配置文件xml。

- 在Spring配置文件中配置UserDaoImpl。
- 使用Spring的API获取Bean对象。
- 代码演示：

``` java

//	接口：
package com.gzhu.dao;

public interface UserDao {
    public void save();
}

//	实现类：
package com.gzhu.dao.impl;

import com.gzhu.dao.UserDao;

public class UserDaoImpl implements UserDao {
    @Override
    public void save(){
        System.out.println("save running");
    }
}

//	配置文件：applicationContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="UserDao" class="com.gzhu.dao.impl.UserDaoImpl"></bean>
</beans>
        
//	测试代码：
import com.gzhu.dao.UserDao;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.context.ApplicationContext;

public class UserDemo {
    public static void main(String[] args) {
        //	获取配置文件对象
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        //	根据Bean的id获取对应的UserDao对象
        UserDao userDao = ( UserDao ) app.getBean("UserDao");
        //	调用UserDao对象的方法save
        userDao.save();
    }
}

```

### 2.2、Spring配置文件：

#### 2.2.1、Bean标签的基本配置：

- 用于配置对象交给Spring来创建。
- 默认情况下，它调用类中的无参数构造创建对象，若没有无参构造，则创建失败。
- 基本属性：
  - id：Bean实例在Spring容器中的唯一标识。
  - class：Bean的u全限定名称。

#### 2.2.2、Bean标签的范围配置：

- scope：指对象的作用范围，取值如下：
  - singleton：默认值，单例的
  - prototype：多例的
  - request：Web项目中，Spring创建一个Bean对象，将对象存放到request域中。
  - session：Web项目中，Spring创建一个Bean对象，将对象存放到session域中。
  - global session：Web项目中，应用在portlet环境，如果没有portlet环境，则global session相当于session。

- 代码演示：

``` java

// 配置文件中Bean的scope属性的取值：
<bean id="UserDao" class="com.gzhu.dao.impl.UserDaoImpl" scope="prototype"></bean>

//	测试代码：
public class SpringTest {
    @Test
    //  测试scope属性
    public void test1(){
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao1 = ( UserDao ) app.getBean("UserDao");
        UserDao userDao2 = ( UserDao ) app.getBean("UserDao");
        //	scope值为prototype，所以创建的对象不是一个对象
        System.out.println(userDao1 == userDao2);			//	false
    }
}
```

- 当scope的取值为singleton时：
  - Bean的实例化对象只有一个。
  - Bean的实例化时机，当Spring的核心文件被加载时，创建对象（无参构造的方式）。
  - Bean的生命周期：
    - 对象创建：当应用加载，容器被创建时，对象就被创建。
    - 对象运行：只要容器在，对象就一直活着。
    - 对象销毁：当应用卸载，容器被销毁时，对象就被销毁。
- 当scope的取值为prototype时：
  - Bean的实例化对象有多个。
  - Bean对象的创建时机：当使用getBean方法时，创建对象。
  - Bean的生命周期：
    - 对象创建:使用getBean时创建对象。
    - 对象运行：只要对象在使用，就一直活着。
    - 对象销毁：只要对象长时间不使用，就会被java回收机制回收。

#### 2.2.4、Bean生命周期配置：

- init-method：指定对象创建之后的首先执行的初始化方法。
- destory-method：指定类中的销毁方法的名称。

#### 2.2.5、Bean的实例化的三种方式：

- 无参构造方法实例化对象：

- 工厂静态方法实例化对象：

  - 创建一个工厂类，类里面定义一个创建UserDao对象的【**静态**】方法，通过工厂类，创建UserDao对象。
  - 代码演示：

  ``` java
  工厂类：
  package com.gzhu.factory;
  public class StaticFactory {
  
      public static UserDao getUserDao(){
          return new UserDaoImpl();
      }
  }
  
  <!--工厂静态方法实例化对象-->
      <!--<bean id = "UserDao" class="com.gzhu.factory.StaticFactory" factory-method="getUserDao"></bean>-->
  ```

- 工厂示例方法实例化对象：

  - 定义一个工厂类，在类里面定义一个成员方法创建UserDao对象。
  - 在配置文件中需要先创建工厂类对象，才可以调用其中的方法，创建UserDao对象。
  - 代码演示：

  ``` java
  工厂类：
  com.gzhu.factory;
  public class DynameicFactory {
  
      public UserDao getUserDao(){
          return new UserDaoImpl();
      }
  }
  
  //	配置文件：
  	<!--工厂实例方法实例化对象-->
      <bean id = "factory" class="com.gzhu.factory.DynameicFactory" ></bean>
      <bean id="UserDao" factory-bean="factory" factory-method="getUserDao" />
  ```

#### 2.2.6、Bean的依赖注入：

- 应用场景：在实际开发中，一个类的成员变量可能是另一个类对象，Spring支持依赖注入，将一个类对象注入到另一个类对象。

- Bean依赖注入的两种方式：

  - set方法：

    - 代码演示：

    ``` java
    //	UserService类：
    public class UserServiceImpl implements UserService {
    
        private UserDao userDao;
    
        public void setUserDao(UserDao userDao) {
            this.userDao = userDao;
        }
    
        @Override
        public void save() {
            userDao.save();
        }
    }
    //	配置文件：
    //	注意：<property>标签的name属性的值为setXXX方法set后面的单词xXX（首字母要小写）！！！
    <bean id = "UserService" class="com.gzhu.service.impl.UserServiceImpl">
        <property name="userDao" ref="UserDao"></property>
    </bean>
    ```

    

  - 构造函数：

    - 代码演示：

    ``` java
    //	UserService类：
    public class UserServiceImpl implements UserService {
    
        private UserDao userDao;
    
        public UserServiceImpl(UserDao userDao) {
            this.userDao = userDao;
        }
        @Override
        public void save() {
            userDao.save();
        }
    }
    
    //	配置文件：
    <bean id = "UserService" class="com.gzhu.service.impl.UserServiceImpl">
            <constructor-arg name="userDao" ref="UserDao"></constructor-arg>
    </bean>
    ```

- Spring支持的依赖注入类型主要分为以下三种（以set方式演示）：

  - 基本数据类型：

  ``` java
  <bean id = "UserService" class="com.gzhu.service.impl.UserServiceImpl">
      <property name="userDao" value="UserDao"></property>
  </bean>
  ```

  - 引用类型：

  ``` java
  <bean id = "UserService" class="com.gzhu.service.impl.UserServiceImpl">
      <property name="userDao" ref="UserDao"></property>
      <property name="age" ref="22"></property>
  </bean>
  ```

  - 集合数据类型：

  ``` java
  <bean id = "UserService" class="com.gzhu.service.impl.UserServiceImpl">
      <list>
      	<value>...</value>
      	<value>...</value>
      	...
      </list>
  </bean>
  ```


#### 2.2.7、Spring引入其他配置文件：

- 在实际开发中，Spring的配置文件内容非常多，放在一起管理起来比较繁琐，Spring支持将部分配置文件拆解到其他配置文件种，而在主配置文件中使用import标签进行加载。
- 格式：

``` java
<import resource="配置文件的文件名"/>
```

### 2.3、Spring相关API：

#### 2.3.1、ApplicationContext的继承体系：

- ApplicationContext：接口类型，代表应用上下文，可以通过其实例获取Spring容器中的Bean对象。

- ApplicationContext的实现类：
  - ClassPathXmlApplicationContext：适用于从类的根路径下加载配置文件。
  - FileSystemXmlApplicationContext：适用于从硬盘上加载配置文件，配置文件可以在磁盘上的任意位置。
  - AnnotationConfigApplicationContext：当时用注解配置容器对象时，需要使用此类来创建Spring容器，用它来读取注解。

#### 2.3.2、getBean方法：

- getBean的两个重载方法：

``` java
public Object getBean(String id):根据容器中的id属性获取对象。
public<T> T getBean（Class<T> RequiredType）：根据字节码文件对象创建对象。
```

- 应用场景：
  - 当参数为字符串时，表示根据Spring容器中的id属性获取对象实例，返回的是Object，需要强转。
  - 当参数是class字节码文件对象时，表示根据类型从容其中获取Bean实例对象，当容器中同一个类型的对象有多个Bean时，此方法会报错（不知道获取哪一个Bean实例）。

### 2.4、Spring配置数据源（连接池）：

#### 2.4.1、数据源（连接池）的作用：

- 数据源是为了提高程序的性能而出现的。
- 事先实例化数据源，初始化部分连接资源。
- 使用连接资源时从数据源中获取。
- 使用完毕后将链连接资源归还给数据源。

#### 2.4.2、常见的数据源：

- DBCP、C3P0、BoneCP、Druid等。

#### 2.4.3、Spring配置数据源：

- 传统方式获取数据源对象：

``` java
@Test
    //  测试手动创建c3p0数据源
    public void test1()throws Exception{
        ComboPooledDataSource datasource = new ComboPooledDataSource();
        datasource.setDriverClass("com.mysql.cj.jdbc.Driver");
        datasource.setJdbcUrl("jdbc:mysql://localhost:3306/bjpowernode?useSSL=FALSE&serverTimezone=UTC");
        datasource.setUser("root");
        datasource.setPassword("832870@Cmy");
        Connection connection = datasource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

- 配置文件方式获取数据源对象：

``` java
//	配置文件：
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/bjpowernode?useSSL=FALSE&&serverTimezone=UTC
jdbc.userName=root
jdbc.password=832870@Cmy
//	main方法：
@Test
    //  测试手动创建c3p0数据源（配置文件形式）
    public void test3()throws Exception{
        //  读取配置文件
        ResourceBundle rb = ResourceBundle.getBundle("jdbc");
        String driver = rb.getString("jdbc.driver");
        String url = rb.getString("jdbc.url");
        String userName = rb.getString("jdbc.userName");
        String password = rb.getString("jdbc.password");
        //  创建连接池（数据源）对象
        ComboPooledDataSource datasource = new ComboPooledDataSource();
        datasource.setDriverClass(driver);
        datasource.setJdbcUrl(url);
        datasource.setUser(userName);
        datasource.setPassword(password);
        //  使用数据源
        Connection connection = datasource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

- 利用spring获取数据源对象：

``` java
//	配置文件：
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/bjpowernode?useSSL=FALSE&amp;serverTimezone=UTC"></property>
        <property name="user" value="root"></property>
        <property name="password" value="832870@Cmy"></property>
</bean>
    
//main方法：
@Test
    //  测试使用spring容器创建数据源
    public void test4()throws Exception{
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        ComboPooledDataSource dataSource = app.getBean(ComboPooledDataSource.class);
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

#### 2.4.4、spring加载外部配置文件：

- 应用场景：现在有一个mysql的配置文件properties和一个spring配置文件，要求spring配置文件能加载properties配置文件的内容，来获取mysql的信息。

- 加载步骤：

  - 在spring配置文件中添加context标签：

  ``` java
  //	添加以下两行代码
  xmlns:context="http://www.springframework.org/schema/context"
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
  ```

- 将properties通过context标签加载到spring容器中：

``` java
<!--加载外部的配置文件-->
<context:property-placeholder location="classpath:jdbc.properties"/>
```

- 新的配置文件：

``` java
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="user" value="${jdbc.userName}"></property>
        <property name="password" value="${jdbc.password}"></property>
</bean>
```

### 2.5、Spring注解开发：

#### 2.5.1、Spring原始注解：

- Spring是轻代码重配置的开发框架，配置文件比较繁琐，影响开发效率，所以注解开发是一种趋势，代替xml配置文件可以简化配置，提高开发效率。
- 常用的个原始注解及其说明：
  - @Component:使用在类上用于实例化Bean
  - @Controller:使用在Web层实例化Bean。
  - @Service:使用在service层实例化Bean。
  - @Repository:使用在dao层实例化Bean。
  - @Autowired:使用在字段（成员变量）上用于根据类型依赖注入。
  - @Qualifier:结合@Autowired使用，用于根据名称实现依赖注入。
  - @Resource：相当于@Autowired+@Qualifier，按照名称进行注入。
  - @Value:注入普通属性。
  - @Scope:标注标签的作用范围。
  - @PostConstruct:相当于原始bean标签的init-method属性，设置初始化方法。
  - @Predestory:相当于原始bean标签的destory-method方法，设置销毁方法。
- 代码演示：

``` java
//	dao层：
//  <bean id="userDao" class="cn.gzhu.dao.impl.UserDaoImpl"/>
@Component("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("save running...");
    }
}

//	service层：
/*
    <bean id = "UserService" class="cn.gzhu.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"></property>
    </bean>
 */
@Component("UserService")       //  参数代表原来的id值
public class UserServiceImpl implements UserService {

    @Autowired
    @Qualifier("userDao")           //  参数表示需要注入的类
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save() {
        userDao.save();
    }
}

//	新的配置文件：告知Spring容器从哪一个包下读取注解
<!--
        需要使用context标签，告知spring容器使用注解实现依赖注入
        base-package（基本包）：表示扫描哪一个包下面的注解。
    -->
    <context:component-scan base-package="cn.gzhu"/>

//	web层（主方法）：
public class UserController {
    public static void main(String[] args) {
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserServiceImpl userService = app.getBean(UserServiceImpl.class);
        userService.save();
    }
}
```

#### 2.5.2、Spring新注解：

- 常用的新注解：
  - @Configuration：用于指定当前类是一个Spring配置类，当创建容器时会从该类上加载注解。
  - @ComponentScan：用于指定Spring在初始化容器时要扫描的包，作用和Spring的xml文件中的标签：**<context:componnet-scan base-package="待扫描的基本包"/>**一样。
  - @Bean：使用在【**方法**】上，标注将该方法的返回值存储到Spring容器中。
  - @PropertySource：用于加载properties配置文件中的数据。
  - @Import：用于导入其他配置类。
- 新注解的使用，代码演示（对比传统xml配置文件）：

``` java
/*
    Spring配置类：代替xml配置文件
 */
@Configuration          //  表示这是一个Spring核心配置类
//  <context:component-scan base-package="cn.gzhu"/>
@ComponentScan("cn.gzhu")       //  参数表示要扫描的包
//  <context:property-placeholder location="classpath:jdbc.properties"/>
@PropertySource("classpath:jdbc.properties")            //  加载外部的properties配置文件
public class SpringConfiguration {
    @Value("${jdbc.driver}")        //  该标签表示注入普通属性
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.user}")
    private String user;
    @Value("${jdbc.password}")
    private String password;

    //  该标签将方法的返回值放到Spring配置文件中
    @Bean("dataSource")     //  参数作为改返回值在Spring容器中的id
    public DataSource getDatasource()throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(user);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

### 2.6、Spring整合Junit：

#### 2.6.1、Spring集成Junit测试步骤：

- 导入spring集成Junit坐标。
- 使用@Runwith注解代替原来的运行期。
- 使用@ConfigurationContext指定配置文件或者配置类。
- 使用@Autowire注入需要测试的对象。
- 创建测试方法进行测试。

- 代码实现：

``` java
@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration("classpath:applicationContext.xml")
@ContextConfiguration(classes = SpringConfiguration.class)
public class SpringJunitTest {

    @Autowired
    private UserService userService;

    @Autowired
    private DataSource dataSource;

    @Test
    public void test()throws SQLException {
        userService.save();
        System.out.println(dataSource.getConnection());
    }
}
```

## 3.Aop:

### 3.1、Aop简介：

#### 3.1.1、什么是Aop：

- Aop：意思是【**面向切面编程**】，是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。

#### 3.1.2、Aop的作用与优势;

- 作用：在程序运行期间，在不修改源码的情况下可以对方法的功能进行增强。
- 优势：减少重复代码，提高开发效率，并且便于维护。

#### 3.2.3、Aop的底层实现：

- 实际上，Aop的底层是通过Spring提供的【**动态代理技术**】实现的，在运行期间，Spring通过动态代理技术动态生成代理对象，代理对象方法执行时进行功能的介入，再去调用目标对象的方法，进而完成功能的增强。

#### 3.2.4、Aop动态代理技术：

- 常用的动态代理技术：
  - JDK代理：基于接口的动态代理。
  - cglib代理：基于父类的动态代理技术。

#### 3.2.5、基于Jdk的动态代理实现（传统方式）：

- 代码演示：

``` java
public class ProxyTest {
    public static void main(String[] args) {
        //  创建目标对象
        final Target target = new Target();
        //  创建增强的对象
        final Advice advice = new Advice();

        /*
            三个参数：
                目标对象的字节码文件的类加载器
                目标对象的实现接口
                另外一个不知道
         */
        TargetInterface proxy = (TargetInterface) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        advice.before();            //  前置增强
                        method.invoke(target, args);
                        advice.after();             //  后置增强
                        return null;
                    }
                }
        );
        proxy.save();
    }
}
```

#### 3.2.6、基于cglib的动态代理实现（传统方式）：

- 代码演示：

``` java
public class Proxy_cglib_Test {
    public static void main(String[] args) {
        //  创建目标对象
        final Target target = new Target();
        //  创建增强对象
        final Advice advice = new Advice();
        //  返回值就是动态生成的代理对象，基于cglib
        //  1.创建增强器
        Enhancer enhancer = new Enhancer();
        //  2.设置父类（目标类）
        enhancer.setSuperclass(Target.class);
        //  3.设置回调
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                advice.before();    //  前置方法
                Object invoke = method.invoke(target, args);
                advice.after();//  后置方法
                return invoke;
            }
        });
        //  4.创建代理对象
        Target proxy = (Target) enhancer.create();
        proxy.save();
    }
}
```

### 3.2、Aop的使用：

#### 3.2.1、Aop的相关术语：

- 目标对象（Target）：代理的目标对象。
- 代理对象（Proxy）：一个类被Aop织入增强后就会产生一个代理对象。
- 连接点（Joinpoint）：连接点在Springt中表示可以被增强的点（方法）。
- 切入点（pointcut）：切入点指我们需要对哪些连接点进行拦截、增强。
- 通知/增强（Advice）：通知就是拦截连接点后的要做的事情就是通知（增强）。
- 切面（Aspect）：是切入点和通知的结合。
- 织入（Weaving）：指将增强应用到目标对象生成代理对象的过程。

#### 3.2.2、Aop开发需要明确的事项：

- 需要编写的内容：
  - 编写核心业务代码（目标对象的目标方法（连接点））。
  - 编写切面类，切面类中有通知（增强方法）。
  - 在配置文件中，配置织入关系，即将那些通知与哪些连接点进行结合
- Aop技术实现的内容：
  - Spring框架监听切入点（方法）的执行，一旦发现有切入点执行，则会使用代理机制，动态生成一个代理对象，根据通知类别，在代理对象的对应位置，将通知对应的功能织入，完成完整的代码逻辑。
- Aop底层使用的哪一种代理方式：
  - 在spring中，框架类会根据目标对象是否实现了接口来决定使用哪一种代理方式。

#### 3.2.3、Xml配置文件实现Aop：

- 实现步骤：

  - 在applicationContext.xml配置文件中引入aop标签

  ``` java
  xmlns:aop="http://www.springframework.org/schema/aop"
  http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
  ```

  - 编写切面类和目标类：

  ``` java
  /*
      切面类：里面包含增强方法
   */
  public class MyAspect {
      public void before(){
          System.out.println("前置增强。。。。");
      }
  
      /**
       * 环绕增强
       * @param poj 切入点
       */
      public Object arround(ProceedingJoinPoint poj) throws Throwable {
          System.out.println("前置增强。。。。");
          Object result = poj.proceed();
          System.out.println("后置增强。。。。");
          return result;
      }
  }
  //	待增强的类：
  public class Target implements TargetInterface {
      @Override
      public void save() {
          System.out.println("save running...");
      }
  }
  ```
  - 在xml文件中配置对象

  ``` java
  <!--配置目标对象-->
  <bean id="target" class="cn.gzhu.proxy.jdk.aop.Target"></bean>
  <!--配置切面-->
  <bean id="myaspect" class="cn.gzhu.proxy.jdk.aop.MyAspect"></bean>
  ```

  - 配置织入

  ``` java
  <aop:config>
          <!--介个标签表示配置切面类-->
          <aop:aspect ref="myaspect">
              <aop:pointcut id="myPointcut" expression="execution(* cn.gzhu.proxy.jdk.aop.*.*(..))"/>
              <!--介个标签的:before表示通知的类型,method属性指定增强方法（通知），pointcut指定需要被增强的方法（切入点）-->
              <!--<aop:before method="before" pointcut="execution(public void cn.gzhu.proxy.jdk.aop.Target.save())"/>-->
              <!--<aop:around method="arround" pointcut="execution(* cn.gzhu.proxy.jdk.aop.*.*(..))"/>-->
              <aop:around method="arround" pointcut-ref="myPointcut"/>
          </aop:aspect>
  </aop:config>
  ```

  

- 切点表达式：

  - 表达式语法：

  ``` java
  execution([修饰符] 返回值类型 包名.类名.方法名（参数列表）)
  ```

  - 访问修饰符可以省略不写。
  - 返回值、包名、类名、方法名都可以使用*代表任意。
  - 包名与类名之间使用一个.代表当前包下的类，两个.代表当前包及其子包下的类。
  - 参数列表可以使用两个点，代表任意个数、任意类型的参数列表。

- 通知的类型：

  - 格式：<aop:XXX .........>：XXX表示通知的类型，常用的有以下几种：
  - 前置通知\<aop:before>：用于配置前置通知，指定增强的方法在切入点方法之前执行。
  - 后置通知\<aop:after-returning>：用于配置后置通知，指定增强的方法在切入点方法之后执行。
  - 环绕通知\<aop:around>：用于配置环绕通知，指定的增强方法在切入点之前、之后都会执行。
  - 异常抛出通知\<aop:throwing>：用于配置抛出异常通知，指定的增强方法在切入点方法抛出异常时执行。
  - 最终通知\<aop:after>：用于配置最终通知，无论增强方法是否执行有异常都会执行。

- 切点表达式的抽取：

  - 格式：

  ``` java
  <aop:pointcut id="自定义一个id" expression="切点表达式"></aop:pointcut>
  ```

#### 3.2.4、基于注解的Aop开发：

- 快速入门：
  - 创建目标接口和目标类。
  - 创建切面类。
  - 将目标类和切面类的创建权交给spring容器。
  - 在切面类中使用注解配置织入关系。
  - 在配置文件中开启组件扫描和Aop的自动代理。
  - 测试。
- 代码演示;

``` java
//	目标类：
<!--配置目标对象-->
    <bean id="target" class="cn.gzhu.proxy.jdk.aop.Target"></bean>
 */
@Component("target")
public class Target implements TargetInterface {
    @Override
    public void save() {
        System.out.println("save running...");
    }
}
//	切面类：
/*
<!--配置切面-->
    <bean id="myaspect" class="cn.gzhu.proxy.jdk.aop.MyAspect"></bean>
 */
@Component("myAspect")
/*
    <!--配置织入：告诉Spring容器那些方法（切入点）需要哪些增强（通知）-->
    <aop:config>
        <!--介个标签表示配置切面类-->
        <aop:aspect ref="myaspect">
            <aop:pointcut id="myPointcut" expression="execution(* cn.gzhu.proxy.jdk.aop.*.*(..))"/>
            <!--介个标签的:before表示通知的类型,method属性指定增强方法（通知），pointcut指定需要被增强的方法（切入点）-->
            <!--<aop:before method="before" pointcut="execution(public void cn.gzhu.proxy.jdk.aop.Target.save())"/>-->
            <!--<aop:around method="arround" pointcut="execution(* cn.gzhu.proxy.jdk.aop.*.*(..))"/>-->
            <aop:around method="arround" pointcut-ref="myPointcut"/>
        </aop:aspect>
    </aop:config>
 */
@Aspect
public class MyAspect {
    @Before("execution(* cn.gzhu.proxy.jdk.anno.*.*(..))")
    public void before(){
        System.out.println("前置增强。。。。");
    }
    public Object arround(ProceedingJoinPoint poj) throws Throwable {
        System.out.println("前置增强。。。。");
        Object result = poj.proceed();
        System.out.println("后置增强。。。。");
        return result;
    }
}
//	测试类：
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext-anno.xml")
public class AopAnnoTest {
    @Autowired
    private TargetInterface target;

    @Test
    public void test(){
        target.save();
    }
}
```

## 4.JDBCTemplate：

### 4.1、Spring的JDBCTemplate概述：

- 他是Spring框架提供的一个对象，是对原始繁琐的jdbc API对象的简单封装。Spring框架为我们提供了很多操作模板类（jdbcTemplate、ReidsTemplate、HiberbateTemplate、JmsTemplate）。

### 4.2、JDBCTemplate模板的使用:

#### 4.2.1、JDBCTemplate的开发步骤：

- 导入Spring-jdbc和Spring-tx坐标。
- 创建数据库和实体。
- 创建JDBCTemplate对象。
- 执行数据库操作。

#### 4.2.2、传统方式使用jdbcTemplate：

- 实现步骤：
  - 配置数据库相关信息，获取数据源对象。
  - 创建jdbcTemplate对象，将数据源对象作为jdbcTemplate的setDataSource方法的参数设置进去。
  - 执行相应的sql语句。
- 代码演示：

``` java
@Test
    //  测试jdbcTemplate开发步骤
    public void test1() throws PropertyVetoException {
        //  创建数据源对象
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/bjpowernode?useSSL=FALSE&serverTimezone=UTC");
        dataSource.setUser("root");
        dataSource.setPassword("832870@Cmy");
        //  创建jdbcTemplate对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //  设置数据源，直到数据库在哪
        jdbcTemplate.setDataSource(dataSource);
        //  执行更新sql语句：？，？为占位符。
        int row = jdbcTemplate.update("insert into account values (?, ?)", "Cmy", 500);
        System.out.println(row);
    }
```

#### 4.2.3、Spring产生jdbcTemplate对象（xml配置文件形式）：

- 实现步骤：
  - 在Spring容器中创建dataSource和JdbcTemplate对象。
  - 利用property标签将dataSource注入jdbcTemplate对象。
  - 加载配置文件，获取jdbcTemplate对象。
  - 执行sql语句，对数据库进行操作。
- 代码实现：

``` java
//	配置文件：
<!--读取properties文件-->
    <context:property-placeholder location="jdbc.properties"/>
    <!--数据源对象-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
	</bean>
    <!--配置jdbcTemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
//  测试jdbc的使用（xml配置文件形式）
    public void test2(){
        ApplicationContext app = new ClassPathXmlApplicationContext("applicatioContext.xml");
        JdbcTemplate jdbcTemplate = app.getBean(JdbcTemplate.class);
        int row = jdbcTemplate.update("insert into account values (?, ?)", "草木灰", 10000);
        System.out.println(row);
    }        
//	测试hdbctemplate的使用:查询。
 	@Test
    public void testQueryAll(){
        //  查询全部元素，并将查询结果封装到实体类Account中。
        List<Account> list = jdbcTemplate.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
        System.out.println(list);
    }
        
```

## 5.Spring的声明式事务控制：

### 5.1、编程式事务控制：

#### 5.1.1、编程事务控制相关的对象：

- PlatformTransactionManager接口：是Spring事务管理器，它里面提供了我们常用的个事务控制方法：
  - 方法：
    - TransactionStatus getTransaction（TransactionDefination defination）：获取事务的状态信息：
    - void commit（TransactionStatus status）：提交事务。
    - void rollback（TransactionStatus status）：回滚事务。
  - 【注意】：
    - PlatformTransactionManager是一个接口，不同的Dao层有不同的实现。
    - spring提供的实现类主要有两个：DataSourceTransactionManager、HibernateTransactionManager。
- TransactionDefinition是事务的定义信息对象，里面有如下方法：
  - 方法：
    - int getIsolationLevel():获取事务的隔离级别。
    - int getpropogationBehavior():获取事务的传播行为
    - int getTimeout():获取超时时间。
    - boolean isReadOnly():是否只读。
  - 事物的隔离级别：
    - 读未提交（ISOLATION_READ_UNCOMMITTED）.
    - 读已提交（ISOLATION_READ_COMMITTED）
    - 可重复读(ISOLATION_REPEATABLE_READ)
    - 串行化读(ISOLATION_SERIALIZABLE)
- transactionStatus接口:提供的是事务的具体运行状态，方法简介如下：
  - 方法：
    - boolean hasSavePoint():是否存储回滚点。
    - boolean isCompleted():事务是否已完成。
    - boolean isNewTransaction():是否是新事务。
    - boolean isRollbackOnly():事务是否回滚。

### 5.2、基于xml文件的声明式事务控制：

#### 5.2.1、什么是声明式事务控制：

- Spring的声明式事务控制：顾名思义就是使用声明的方式来处理事务。这里的声明，就是指在配置文件中声明，用在Spring配置文件中声明式的处理事务来代替代码处理事务。

#### 5.2.2、声明式事务控制的作用：

- 事务管理不侵入开发的组件（事务控制与业务代码分隔开）。
- 在不需要事务控制的时候，只需要在配置文件中修改相应的配置即可，不需要改变代码重新编译，这样维护起来比较方便
- 【注意】：Spring声明事务的底层原理就是Aop。

### 5.3、基于注解的声明式事务控制：

























