# Spring

## 课程导学

![image-20250220000352054](Spring.assets/image-20250220000352054.png)

Spring框架可以说是JAVA最重要的框架，没有之一，业内流传着一句话：JAVA程序员都是spring程序员。此话虽然有点不太谦虚，但是事实几乎如此，可以毫不夸张的说，如果Sun公司赋予JAVA语言生命，那么Spring就是该生命的延续，它会让JAVA进化的更加强大。

![image-20250220000507473](Spring.assets/image-20250220000507473.png)

现在JAVA程序员在找工作时，Spring基础应用并不再是加分项，而是是否能入职的一个合格线。纵观招聘站点上对于JAVA中级以上程序员的招聘要求就会发现，几乎所有的企业都在使用Spring进行项目开发，所以Spring必须要会。

其次中大型互联网公司或者高级开发程序员对Spring的水准要求也不仅仅是简单使用，必须要达到一个精通的水准，所以说需要对Spring进行再次深挖。

企业在进行技术选型时，哪个技术和Spring集成好，集成方便，最终哪个就有可能被选中。可以说，其他技术都有替代方案，而Spring稳坐王位，无惧撼动。总而言之，不会Spring就无法从事JAVA相关工作。

![image-20250220000647464](Spring.assets/image-20250220000647464.png)

1.比如：使用自定义命名空间或**@import注解**结合**Beanpostprocessor**去完成企业通用级别的第三方框架集成方案。

2.学完本套课程，不仅仅能从事基本的Spring开发，还能针对Spring的复杂问题进行解决，亦或是自定义Spring的相关组件。比如：很多课程都在讲解注解，但是只讲解注解的基本使用。这套课程不仅讲注解的基本使用，还要从它背后的原理，实现的机制以及一些细节逐一剖析剖析。

![image-20250220000818584](Spring.assets/image-20250220000818584.png)



## IoC基础容器

![image-20250220002420361](Spring.assets/image-20250220002420361.png)

### 传统Javaweb开发的困惑

```java
//用户账户信息修改业务方法
public void updateUserInfo(User user) {
	try{
		//开启事务
		DaoUtils.openTransacation();
		//获取UserDao执行插入User数据到数据库操作
		UserDao userDao = new UserDaoImpl();
		userDao.updateUserInfo(user);
		//修改成功后，向用户行为日志表中插入一条数据，内容：修改时间等信息
		UserLog userLog = new UserLogImpl();
		userLog.recodeUserUpdate(user);
		//提交事务
		DaoUtils.commit();
	} catch(Exception e) {
		//回滚事务
		DaoUtils.rollback();
		//向异常日志表中插入数据
		ExceptionLog exceptionLog= new ExceptionLogImpl();
		exceptionLog.recodeException(this,e);
	}
}



//用户注册业务方法
public void regist(User user) {
	try{
		//开启事务
		DaoUtils.openTransacation();
		//获取UserDao执行插入User数据到数据库操作
		UserDao userDao = new UserDaoImpl();
		userDao.addUserInfo(user);
		//注册成功后，向用户行为日志表中插入一条数据，内容：时间、用户、注册行为
		UserLog userLog = new UserLogImpl();
		userLog.recodeUserRegister(user);
		//注册成功后，向用户邮箱发送一封激活邮件
		CommonUtils.sendEmail(user);
		//提交事务
		DaoUtils.commit();
	} catch(Exception e) {
		//回滚事务
		DaoUtils.rollback();
		//向异常日志表中插入数据
		ExceptionLog exceptionLog= new ExceptionLogImpl();
		exceptionLog.recodeException(this,e);
	}
}
```

![image-20250220004322904](Spring.assets/image-20250220004322904.png)

分析：

`updateUserInfo`方法的主要业务就是获得一个`userDao`对象，然后通过`userDao`对象调Dao层去更新用户的信息。`regist`方法的主要业务是获得一个`userDao`对象，然后通过`userDao`对象向数据库插入一条数据。
在`updateUserInfo`方法内部需要开启事务，如果更新操作没有问题，需要提交事务，如果发生异常，在`catch`当中需要回滚事务，如果用户的一些行为信息需要进行记录，比如用户更改信息，需要把用户的操作用日志的方式进行记录，所以需要获得用户的日志对象，通过用户日志对象去记录当前用户的操作，比如时间，用户操作等等，如果日志操作发生异常，还要把这个异常的信息也要通过日志的方式进行记录，方便程序员和运维人员在后台进行查看。同样这些过程在`regist`方法当中也有体现。

![image-20250220005236073](Spring.assets/image-20250220005236073.png)

传统Web开发的问题1：

在业务层的业务方法当中，要获取Dao层对应的操作对象，所以直接new一个对象，再去调用对应的方法进行操作。但是它会违背开发的一个原则——偶合度比较紧密。即层与层之间偶合的比较紧密，还有接口与接口对应的具体实现偶合比较紧密。比如：UserDao接口对应UserDaoImpl实现，接口对应具体实现，而接口的行为对应具体实现的行为，如果后期想切换对应的具体实现，只能修改源码。

**解决方法**：

使用设计模式——工厂模式。工厂就是去创建Bean对象，借助这个工厂第三方就可以避免接口与具体实现之间的紧偶合，也可以避免业务层与Dao层之间的紧偶。

![image-20250220012530357](Spring.assets/image-20250220012530357.png)

传统Web开发的问题2：

在业务层的业务方法当中基本都有事务操作和日志操作，这些操作是通用的行为。所以事务和日志操作和业务代码是紧耦合的。

**解决方法**：

这里也是通过第三方去返回一个Bean对象，但是返回并不是Bean对象本身，而是返回Bean的**proxy对象**——**代理对象**，而这个proxy对象会对Bean对象进行增强。比如：上面的`updateUserInfo` 方法属于业务层，也就是属于**UserService**对象的方法，如果UserService对象不是我们自己去new出来，而是通过第三方去创建，但是第三方创建的UserService对象不是普通的Bean对象，而是是一个被增强过的对象，被增强过的Bean对象它内部的方法也是被增强的，即这些方法具备事务和日志的功能，换句话说，事务和日志操作不用我们在方法里面写，让第三方去增强，这样就更加方便一点。



### IoC、DI和AOP思想提出

![image-20250220012711899](Spring.assets/image-20250220012711899.png)

![image-20250220013119124](Spring.assets/image-20250220013119124.png)

程序代码获取Bean对象时不直接new，而找第三方，只不过找第三方需要分情况，有些需要这个Bean对象本身（即不需要事务和日志操作），有些需要增强的Bean对象（即需要事务和日志操作），即让第三方在Bean的基础上使用proxy模式。

1. **IoC思想**：即**控制反转**（或**反转控制**）。它强调的Bean对象的创建权的反转。将Bean对象的创建权反转给第三方，原先程序代码内部要想获取某个Bean对象，直接new即可，而现在让第三方去控制Bean的创建权，如果使用Bean对象，需要找第三方创建，就把Bean的创建权反转出去了。
2. **DI思想**：即**依赖注入**。它强调的是Bean的注入关系（或Bean的设置关系）。比如：上图中，开发过程当中，第一个Bean对象内部需要设置第二个Bean的对象，第一个Bean的对象才能达到想要的效果（就是第一个Bean对象组合或者聚合第二个Bean对象），原先可以通过程序代码找第三方先后创建第一个Bean对象和第二个Bean对象，然后在程序代码当中把第一个Bean对象设置给第一个Bean对象，这是人为去做的。而现在可以把设置动作交给第三方，也就是说第三方在创建第一个Bean对象的时候，也会创建第二个Bean对象，并把第二个Bean对象设置给第一个Bean对象，这样程序代码通过第三方获取第一个Bean对象的时候，第一个Bean对象的内部就已经包含第二个Bean对象的引用。
3. **AOP思想**：即**面向切面编程**思想。面向切面编程比面向对象编程更加高级，面向对象编程是纵向设计一个Bean，而AOP面向切面编程是横向功能抽取的思想，它主要的实现方式就是proxy代理，主要功能是对某一个Bean进行增强。

![image-20250220015107142](Spring.assets/image-20250220015107142.png)

生活中的例子：大厦的基本框架（半成品） => 再进行外形设计 => 一栋大厦。

**软件当中的框架也是个半成品**。比如：在程序开发时，需要100行代码，但是经常被用到的代码可能是50行，不管什么代码都会用到这50行代码，就可以把那这50行代码抽取成一个框架（半成品），下次再写的时候就不需要自己去写，直接在这个框架的基础上做一些特有的定制化的功能。

![image-20250220020050158](Spring.assets/image-20250220020050158.png)

![image-20250220020419510](Spring.assets/image-20250220020419510.png)

现在程序员在开发时一般都不会使用基础的语言进行开发，都会使用基础语言上面的框架进行开发，所以使用框架开发是现在程序员的最基本的一个素养。



### Spring框架的诞生

![image-20250220020607790](Spring.assets/image-20250220020607790.png)



#### spring框架概述

只有思想还是不行，这个思想本身是没法真正用于编码的，需要在思想的基础上去开发出一款框架，这个框架是真正可以用于编码的。Spring就是这么一款框架。

![image-20250220020901714](Spring.assets/image-20250220020901714.png)

Spring是一个开源的，轻量级的框架，它可以简化企业级开发，使用它可以让我们写的代码变得很少，因为框架已经封装了一些功能。Spring本身就是实现IOC、AOP这些思想的框架。Spring框架几乎现在是JAVA开发不可缺少的框架之一，Spring的生态非常完善，不管是JAVA开发的哪个领域，Spring基本都会提供相应的解决方案，但是不管是哪个解决方案，最终都会依附于最基础的**SpringFramework**。

![image-20250220020931066](Spring.assets/image-20250220020931066.png)

[Spring官网](https://www.spring.io)

![image-20250220021531424](Spring.assets/image-20250220021531424.png)

<img src="Spring.assets/image-20250220021936787.png" alt="image-20250220021936787" style="zoom:50%;" />

![image-20250220022154406](Spring.assets/image-20250220022154406.png)

![image-20250220022524447](Spring.assets/image-20250220022524447.png)

![image-20250220022515875](Spring.assets/image-20250220022515875.png)



#### Spring框架的历史

![image-20250220023239312](Spring.assets/image-20250220023239312.png)



#### Spring Framework技术栈图示

![image-20250220023343061](Spring.assets/image-20250220023343061.png)

**Spring Framework技术站:**

- **Test**：Spring提供的一个测试。
- **Core Container**：**核心容器**。内部包括四部分：**Beans**，**Core**，**Context**和**SpEL**。Core本身就是核心，这四个其实可以从四个Jar包看到这四个。就在开发时一般会引入四个jar包，使用maven开发时就是引入四个坐标。在引入Context的时候会自动引入其余三个。Beans代表Bean，因为Spring本身就是管理Beande的；Core代表核心；Context代表上下文；SpEL代表Spring的EL表达式。
- **AOP**和**Aspects**：Aspects表示切面，AOP的一个技术。
- **Data Acess**：**数据访问层**。内部包括JDBBC，ORM框架，OXM，JMS，Transactions事务。
- **Web**：**Web层**。内部包括WebSocket，Servlet，Web，Portiet，mvc。
- 不管是数据访问层还是Web层，都是在核心容器跟AOP基础之上去进行构建的，所以数据访问层和Web层属于上层的建筑。OK，那



#### BeanFactory版本的快速入门

**体验IoC**：

![image-20250220122651788](Spring.assets/image-20250220122651788.png)

**配置清单**：首先程序代码要使用某一个Bean，要找第三方，但是第三方得知道要帮程序代码提前创建Bean对象，然后程序代码需要获取对象的时候再传递给程序代码。所以需要把第三方BeanFactory创建的Bean配置到一个配置清单，也就是就是一个XML文件当中，BeanFactory在加载这个配置文件之后，就能产生对应的Bean，让程序代码能够获取。

开发步骤：

1. 首先第三方BeanFactory不是我们自己写的，它是Spring提供的，所以第一步需要导入Spring对应的坐标。

   ![image-20250220125433499](Spring.assets/image-20250220125433499.png)

   **pom.xml**：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.itheima</groupId>
       <artifactId>spring_ioc_test01</artifactId>
       <version>1.0-SNAPSHOT</version>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>5.3.23</version>
           </dependency>
       </dependencies>
   
   </project>
   ```

2. 产生Bean之前需要准备Bean，所以第二步去编写一个UserService接口和它的实现UserServiceImpl。写一个普通的Bean也可以。

   ```java
   package com.itheima.service;
   
   public interface UserService {
   }
   
   
   
   package com.itheima.service.impl;
   
   import com.itheima.service.UserService;
   
   public class UserServiceImpl implements UserService {
   }
   ```

   

3. 写完Bean之后得将Bean配置到清单当中，最终才能帮程序代码产生Bean，所以第三步需要创建User.xml配置文件，配置文件的名字可以自定义，需要将UserServiceImpl配置到这个xml文件当中。

   **beans.xml**：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!-- 配置UserServiceImpl -->
       <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"></bean>
   </beans>
   ```

   ![image-20250220130454017](Spring.assets/image-20250220130454017.png)

   ![image-20250220131205615](Spring.assets/image-20250220131205615.png)

   * **class**：UserService实现类的全类名。通过全类名知道这个Bean的位置，只有知道这个Bean的位置才能创建这个Bean的对象。因为底层要通过反射拿到全类名去创建对象
   * **id**：最好为标签起一个名字，也就是唯一标识。它的作用是如果要一个配置文件当中配置多个Bean，通过BeanFactory去获取Bean的时候，可以根据id去区分不同的Bean，通过这个名字去获得对应Bean的对象，这里是获取UserServiceImpl的对象。

4. 第四步就是写测试代码，需要通过BeanFactoy去加载配置文件，然后在程序代码当中通过BeanFactory去获取对应的Bean。

   ```java
   package com.itheima.test;
   
   import com.itheima.service.UserService;
   import org.springframework.beans.factory.support.DefaultListableBeanFactory;
   import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
   
   public class BeanFactoryTest {
       public static void main(String[] args) {
           // 1.创建一个工厂对象
           //在Spring当中提供BeanFactory的实现DefaultListableBeanFactory
           DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
   
           // 2.创建一个xml读取器（读取xml配置文件）
           //因为当前使用xml的配置方式，在Spring当中提供BeanDefinitionReader的实现XmlBeanDefinitionReader，参数就是
           //上一步创建的DefaultListableBeanFactory的对象
           XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
   
           // 3.将xml读取器和工厂绑定（因为读取的数据要给工厂），使用xml读取器读取配置文件给工厂
           //使用读取器加载beanDefinitions，里面传递xml配置文件名。
           reader.loadBeanDefinitions("beans.xml");
   
           // 4.根据配置文件中配置的标签id获取Bean实例对象
           UserService userService = (UserService) beanFactory.getBean("userService");
           System.out.println(userService);
       }
   }
   ```



#### BeanFactory版本的快速入门2

**体验DI：**

![image-20250220135615060](Spring.assets/image-20250220135615060.png)

前面通过BeanFactory去产生一个UserService对象，BeanFactory之所以能产生UserService对象，是因为我们把UserService配置到xml配置文件，然后通过BeanFactory去加载xml配置文件。也就意味着我们把需要产生的Bean配置到配置文件当中，BeanFactory就能帮我们去产生对应Bean。

比如：

```java
package com.itheima.dao;

public interface UserDao {
}



package com.itheima.dao.impl;

import com.itheima.dao.UserDao;

public class UserDaoImpl implements UserDao {
}



package com.itheima.test;

import com.itheima.dao.UserDao;
import com.itheima.service.UserService;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;

public class BeanFactoryTest {
    public static void main(String[] args) {
        // 1.创建一个工厂对象
        //在Spring当中提供BeanFactory的实现DefaultListableBeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2.创建一个xml读取器（读取xml配置文件）
        //因为当前使用xml的配置方式，在Spring当中提供BeanDefinitionReader的实现XmlBeanDefinitionReader，参数就是
        //上一步创建的DefaultListableBeanFactory的对象
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);

        // 3.将xml读取器和工厂绑定（因为读取的数据要给工厂），使用xml读取器读取配置文件给工厂
        //使用读取器加载beanDefinitions，里面传递xml配置文件名。
        reader.loadBeanDefinitions("beans.xml");

        // 4.根据配置文件中配置的标签id获取Bean实例对象
        UserService userService = (UserService) beanFactory.getBean("userService");
        System.out.println(userService);

        UserDao userDao = (UserDao) beanFactory.getBean("userDao");
        System.out.println(userDao);
    }
}
```

**beans.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置UserDaoImpl -->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
</beans>
```



创建其他对象也是相同的步骤：先去编写对应Bean，然后将Bean配置到配置文件当中，最后去修改客户端代码获取对应Bean即可。

但是我们在开发时要严格遵守JAVA EE三层架构，业务层要调Dao层，也就是说UserServiceImpl内部如果有方法的话，业务方法内部要用到UserDaoImpl，那么UserServiceImpl内部得维护UserDaoImpl的引用。之前的做法是在UserServiceImpl内部设置对应的setUserDao方法，通过这个setUserDao方法把UserDaoImpl设置进去，现在有了DI思想之后就没有必要这么干。

因为有DI思想的引入，有一个概念叫注入，就是在BeanFactory内部可以把UserDaoImpl直接注入给UserServiceImpl，但是要提前告诉BeanFactory，UserServiceImpl内部需要依赖注入UserDaoImpl，这就需要在xml配置文件当中进行配置。

1. 首先在UserServiceImpl当中提供一个SetUserDao方法，这样BeanFactory在底层操作时才能调用UserServiceImpl的setUserDao方法把UserDaoImpl设置进去。

   ```java
   package com.itheima.service.impl;
   
   import com.itheima.dao.UserDao;
   import com.itheima.service.UserService;
   
   public class UserServiceImpl implements UserService {
       private UserDao userDao;
       //BeanFactory去调用该方法，从容器（BeanFactory）当中获取userDao设置到此处
       public void setUserDao(UserDao userDao) {
   //        System.out.println("BeanFactory去调用该方法，获取userDao设置到此处：" + userDao);
           this.userDao = userDao;
       }
   }
   ```

   但是光有set方法还不行，因为没有告诉BeanFactory。

2. 通过配置的方式告诉BeanFactory要将UserDaoImpl注入到UserServiceImpl当中。

   **beans.xml**:

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!-- 配置UserServiceImpl -->
       <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
           <property name="userDao" ref="userDao"></property>
       </bean>
   
       <!-- 配置UserDaoImpl -->
       <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
   </beans>
   ```

   在xml配置文件当中，在和UserServiceImpl（业务层）关联的bean标签当中配置一个子标签property。

   ![image-20250220145842478](Spring.assets/image-20250220145842478.png)

   - **name**：指的是UserServiceImpl里面的setUserDao方法设置的属性的名称，首字母小写。这里是userDao。

   - **ref**：代表引用。ref的属性值是在BeanFactory或者xml配置文件当中与UserDaoImpl关联的bean标签的id。表示把容器当中的UserDaoImpl设置给UserServiceImpl的setUserDao方法

   - 注意：这里的name和ref的属性值都是userDao，但它们的含义不一样，ref代表引用容器当中id与ref属性值userDao相等且与UserDaoImpl关联的bean标签，设置给UserServiceImpl当中的属性名为name属性值userDao的setUserDao方法。

     ![image-20250220150045372](Spring.assets/image-20250220150045372.png)

3. setUserDao方法执行了，但是在测试方法当中我们没有调用setUserDao方法，也就是说BeanFactory帮我们调用了setUserDao方法，这就是DI依赖注入。

**总结：**

1. IOC是控制反转，反转Bean的创建权。在测试代码当中我们并没有手动去new一个UserServiceImpl或者new一个UserDaoImpl，把new对象的权力反转给BeanFactory，这就是IOC反转Bean的创建权。

2. DI叫依赖注入，测试代码当中获取的UserServiceImpl对象其实默认情况下是个半成品，因为在实际开发当中UserServiceImpl内部要用到UserDaoImpl，如果我们没有配置依赖注入的时候，UserServiceImpl内部没有UserDaoImpl，而现在在配置文件当中通过property标签把容器当中的UserDaoImpl设置给UserServiceImpl的setUserDao方法。UserServiceLmpl里面的方式是setUserDao，xml配置文件当中与UserServiceImpl关联的bean标签里面的property标签的name属性值就是userDao，如果UserServiceLmpl里面的方式是setXxx，xml配置文件当中与UserServiceImpl关联的bean标签里面的property标签的name属性值就是xxx。如果UserServiceImpl是一个功能完整的Bean，它需要依赖UserDaoImpl的注入才能然后完成相应的功能。

   ![image-20250220151639662](Spring.assets/image-20250220151639662.png)

3. 后期在开发时，一般经常会在xml配置文件当中完成单个Bean的配置和多个Bean之间引用，也就是使用set方法注入的配置。



#### BeanFactory版本的依赖注入总结

![image-20250220152928443](Spring.assets/image-20250220152928443.png)

![image-20250220153010223](Spring.assets/image-20250220153010223.png)

![image-20250220153324848](Spring.assets/image-20250220153324848.png)

BeanFactory其实才是Spring整体运行过程当中最核心的一个对象，即使是Application（也叫做Spring容器、应用上下文），它的底层对于Bean的一些操作调用的还是BeanFactory。BeanFactory是Spring框架当中最底层核心的那一部分。



#### ApplicationContext版本的快速入门

![image-20250220154818508](Spring.assets/image-20250220154818508.png)

ApplicationContext比BeanFactory更常用一些。因为它是一个Spring容器，BeanFactory其实本身是个Bean工厂。

![image-20250220155304794](Spring.assets/image-20250220155304794.png)

```java
package com.itheima.test;

import com.itheima.dao.UserDao;
import com.itheima.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 徐梓豪
 * @version 1.0
 * @description: TODO
 * @date 2025/2/20 下午3:55
 */
public class ApplicationContextTest {
    public static void main(String[] args) {
        //直接把类加载路径下的配置文件传进去
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = (UserService) applicationContext.getBean("userService");
        System.out.println(userService);
    }
}
```

**applicationContext.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 配置UserServiceImpl -->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"></property>
    </bean>

    <!-- 配置UserDaoImpl -->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
</beans>
```



#### BeanFactory与ApplicationContext的关系

![image-20250220204541812](Spring.assets/image-20250220204541812.png)

**对比BeanFactory和Spring容器之间的关系：**

1. 从名字上区分：BeanFactory其实内部主要维护的功能是跟Bean产生相关的。而Spring容器内部除了有BeanFactory的功能之外。还有一些其他功能。

2. 我们在开发时，一般情况下，BeanFactory底层的API很难自己去操作，我们都是操作Spring容器ApplicationContext的一些API，ApplicationContext的API再去调用BeanFactory底层的API。

   ![image-20250220220211239](Spring.assets/image-20250220220211239.png)

   ![image-20250220210231445](Spring.assets/image-20250220210231445.png)

   * **ApplicationEventPublisher**：事件发布器，它有事件发布的功能。事件发布就是指可以监听一个事件，然后那边发布事件这里能监听到，即监听机制。
   * **ResourcePatternResolver**：资源解析器。
   * **MessageSource**：消息资源解析器，也是国际化功能。

3. ![image-20250220211358761](Spring.assets/image-20250220220459518.png)

   ![image-20250220211309357](Spring.assets/image-20250220211309357.png)

   ![image-20250220211309357](Spring.assets/image-20250220211309357.png)

   ![image-20250220214001043](Spring.assets/image-20250220214001043.png)

4. 简单来说BeanFactory是延迟加载，调用getBean方法时才去创建Bean对象；而ApplicationContext是立即加载，只要加载配置文件，创建容器，Bean对象就创建好了，而且是在一个map对象内部，使用的时候直接从map对象里面拿即可。

   测试BeanFactory：

   ![image-20250220214930507](Spring.assets/image-20250220214930507.png)

   ![image-20250220215103233](Spring.assets/image-20250220215103233.png)

   测试ApplicationContext：

   ![image-20250220215807601](Spring.assets/image-20250220215807601.png)


![image-20250220221651385](Spring.assets/image-20250220221651385.png)



#### BeanFactory的继承体系

BeanFactory和ApplicationContext都是接口，一般使用的都是它们的实现，因为实现的功能更加强大，平常也是调用实现里面的方法。

![image-20250220225311936](Spring.assets/image-20250220225311936.png)

![image-20250220221945032](Spring.assets/image-20250220221945032.png)

![image-20250220223637596](Spring.assets/image-20250220223637596.png)

* **`beanDefinitionMap -> Map<String, BeanDefinition>`**：中文叫Bean定义Map，简单来说，java是一个面向对象语言，在XML配置文件当中定义了很多bean标签，这些bean标签里面的信息最终也得封装。

  ![image-20250220223707643](Spring.assets/image-20250220223707643.png)

  **注意**：这里说的是封装，而不是根据bean标签的信息创建Bean对象，创建Bean对象是直接使用反射根据bean标签里面的信息创建即可。

  在xml当中配置这些bean标签信息还不能使用，后期得把这些bean标签信息抽取之后封装到一个对象当中，这些bean标签信息的对象称为Bean定义对象。每一个Bean定义对象封装的是xml配置文件当中的每一个bean标签，每一个Bean定义对象最终封装到BeanDefinition对象当中，然后这些BeanDefinition对象最终都存放到beanDefinitionMap当中。

![image-20250220225134474](Spring.assets/image-20250220225134474.png)

![image-20250220230716370](Spring.assets/image-20250220230716370.png)

![image-20250220230014094](Spring.assets/image-20250220230014094.png)



#### ApplicationContext的继承体系

![image-20250220231934586](Spring.assets/image-20250220231934586.png)

ApplicationContext常用的就三个实现类，其实不管是哪个框架的配置，即使是servlet的配置也只有两套方案：一个是xml配置方案，一个注解配置方案：

- 与xml配置方案相关的实现类有两个，当Spring配置使用XML方式配置的时候，Spring容器的客户端使用下面两个实现类，这两个使用场景的区别为：

  - **FileSystemXmlApplicationContext**：如果配置文件不在类加载路径下，比如配置文件放在D盘下，D盘是一个文件系统，就创建FileSystemXmlApplicationContext类的对象去加载配置文件。
  - **ClassPathXmlApplicationContext**：如果配置文件在类加载路径下，比如Maven工程就是类加载路径，配置文件放在resources目录下，就创建ClassPathXmlApplicationContext类的对象去加载配置文件。

- 与注解配置方案相关的实现类有一个，当Spring配置使用注解配置的时候，Spring容器的客户端使用下面的实现类：

  **AnnotationConfigApplicationContext**：当使用注解方式配置，客户端就创建AnnotationConfigApplicationContext类的对象去加载配置文件。



**解释前提条件为什么是只在Spring基础环境下：**

**pom.xml文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>spring_ioc_test01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.23</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>6.0.11</version>
        </dependency>
    </dependencies>

</project>
```

![image-20250220232851731](Spring.assets/image-20250220232851731.png)

如果在pom.xml文件当中配置spring-web的依赖，当Spring跟Web环境集成之后，ApplicationContext的继承体系就会发生变化，比如多了一个Web包：

- **XmlWebApplicationContext**：这个实体类的作用是使用xml配置方式配置的Web环境下的Spring容器——ApplicationContext。
- **AnnotationConfigWebApplicationContext**：这个实体类的作用是使用注解配置方式配置的Web环境下的Spring容器——ApplicationContext。

所以当我们在pom.xml配置文件当中配置其他依赖时，Spring容器的继承体系也会跟着变化。比如在web环境下肯定有属于web环境特质的Spring容器。

![image-20250220233624821](Spring.assets/image-20250220233624821.png)



### 基于xml方式的Spring应用

![image-20250221000112579](Spring.assets/image-20250221000112579.png)



#### SpringBean的配置详解

![image-20250221000735254](Spring.assets/image-20250221000735254.png)

1. `<bean id="" class="">`：`class`属性主要是配置Bean的全限定名。`id`属性是Bean在配置文件当中的一个唯一标识，在后期对Bean进行封装的时候，id在容器当中会转换成Bean的Bean name，也就是Bean的名称。而我们在使用getBean方法时，()里面的参数其实不是id，而是Bean name，Bean的名称，只不过在xml文件当中配置的id后期会转成容器当中的Bean name。

2. `<bean name="">`：name属性用于配置Bean的别名。也就是说一个Bean可以有多个名称。所以在使用getBean方法时，()里面不仅可以传入id（其实应该传入Bean name，但是id后期会转化为Beanname）去获取Bean，也可以传入name别名去获取Bean。

3. `<bean scope="">`：scope属性用于配置Bean的作用范围。Bean的作用范围是指，在默认情况下，每个Bean只存在一个单例Bean对象，也就是说Spring容器对于每个Bean只产生一个Bean对象，放到map容器当中，在获取Bean对象的时候直接返回对应的这个Bean对象，但是有的时候可以修改scope属性，让Spring容器为同一个Bean产生多个Bean对象，就是每次在调用getBean方法时都会给创建一个新的Bean对象返回，这样的话Bean对象就没办法去存放在map容器当中，因为Bean对象的数量不确定，会影响性能。

4. `<bean lazy-init="">`：lazy-init属性用于配置延迟加载。Spring容器ApplicationContext在默认情况下加载配置文件，创建容器之后，Bean就创建好放到一个位置等待你使用getBean()去获取，可以看成提前缓存。但如果某些Bea在配置时，设置lazy-init为true，意味着不需要在加载配置文件的时候就创建Bean对象，而是和Bean工厂一样，什么时候调用getBean()方法的，什么时候创建Bean对象。

5. `<bean init-method="">`：init-method属性用于设置初始化方法。初始化方法是指当Bean创建完毕之后，可能要执行一些初始化的操作，就可以通过配置init-method属性的方式指定这个Bean内部的哪个方法作为初始化方法。这样当Bean创建完毕之后，Spring会自动去调用初始化方法。

   **注意**：构造方法用于构造对象；初始化方法是等对象创建完毕之后，再调用初始化方法，它们是不同的概念，有时间差。

6. `<bean destroy-method="">`：destroy-method属性用于设置销毁方法，表示Bean对象在挂掉之前要执行某些方法。这个属性的用处不是特别大。

7. `<bean autowire="byType">`：autowire属性用于设置自动注入。我们之前是通过配置注入依赖，通过propertes标签和set方法注入。如果不配置情况的下，我们也可以通过设置自动注入的方式从容器当中自动找到对应的Bean注入到另一个Bean中，可以设置自动注入的方式，比如byType表示通过类型自动注入，byName表示通过名字自动注入。如果配置完毕之后，Bean的内部有对应的set方法，Spring会自动的从容器当中找类型匹配的Bean或名字匹配的Bean注入到这个Bean中，就不用每次都人为设置properties属性进行注入。

8. `<bean factory-bean=""factory-method=""/>`：factory-bean和factory-method用于配置Bean的创建方式。



##### beanName和别名配置

![image-20250221012226294](Spring.assets/image-20250221012226294.png)

**关于id属性的细节：**

- id是bean标签的唯一标识，不能重复。

  ![image-20250221022147748](Spring.assets/image-20250221022147748.png)

- getBean方法里面的参数传递的是beanName。

  ![image-20250221022425665](Spring.assets/image-20250221022425665.png)

  如果浅显理解，id和beanName可以认为是一样的，但是在底层，id和beanName不一样，id是xml配置文件当中的bean标签的一个属性，表示bean标签的唯一标识。最终Bean对象进入到Spring容器ApplicationContext当中，Spring容器会把id转化成一个beanName进行存储，在getBean方法的()里面传递的参数是beanName，也就是Bean的名称，而且可以有不同的东西去充当beanName。比如：

  ![image-20250221024047122](Spring.assets/image-20250221024047122.png)

  如果bean标签配置id属性，beanName的值与id属性的值相同。

  如果bean标签不配置id属性，getBean方法会报错，因为bean标签没有设置id属性，使用原来的beanName就会报错。但是注意：==bean标签没有设置id不代表没有beanName==，其实没有配置id属性，还是有beanName的，此时beanName的值和Bean的全限定名相同。

  ![image-20250221025231382](Spring.assets/image-20250221025231382.png)

  ![image-20250221025738381](Spring.assets/image-20250221025738381.png)

  ![image-20250221030144145](Spring.assets/image-20250221030144145.png)

  ![image-20250221031800191](Spring.assets/image-20250221031800191.png)

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <!-- 配置UserServiceImpl -->
      <bean id="userService" name="aaa,bbb,ccc" class="com.itheima.service.impl.UserServiceImpl">
          <property name="userDao" ref="userDao"></property>
      </bean>
      <!-- 配置UserDaoImpl -->
      <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
  </beans>
  ```

  ![image-20250221030545511](Spring.assets/image-20250221030545511.png)
  ![image-20250221030821398](Spring.assets/image-20250221030821398.png)

  ![image-20250221031405483](Spring.assets/image-20250221031405483.png)

  ![image-20250221031625678](Spring.assets/image-20250221031625678.png)

**总结：**

1. 如果在bean标签当中配置了id属性，那么beanName的值就是id属性值，此时调用getBean方法时，可以通过beanName去获取Bean，

   如果设置了name别名，也可以通过别名去获取Bean，但是不能通过Bean的全限定名去获取Bean。

2. 如果在bean标签当中没有配置id属性，但是配置了name属性，也就是别名，那么默认情况下beanName的值就是第一个别名，可以通过beanName去获取Bean，但是不能通过Bean的全限定名去获取Bean。

3. 如果在bean标签当中既没有配置id属性，也没有配置name属性，那么默认情况下beanName的值就是Bean的全限定名，可以根据beanName获取Bean。

4. 一般bean标签配置id属性即可，name别名几乎不用。



##### Bean的作用范围

![image-20250221133013311](Spring.assets/image-20250221133013311.png)

如果没有配置Bean的作用范围，默认作用范围是singleton单例，也就是说只有一个Bean对象，在服务器启动时加载完配置文件，这个对象就创建完毕，并存到map当中，需要用的时候直接从map俩面取即可。但是有的时候我不需要它是单例，想要创建多个Bean对象，此时就可以把scope设置为prototype，prototype叫做原型，也就是说每次调用getBean方法时都创建一个新的Bean对象返回。

- **singleton**：单例，默认值，Spring容器创建时就会进行Bean的实例化，也就是加载配置文件，创建spring容器时，这个对象就创建好了，并存入到容器内部的单例池当中，也就是SingleObjects里面。每次调用getBean方法时都是从单例池（HashMap类型）当中获取相同的Bean实例。

  ![image-20250221134848226](Spring.assets/image-20250221134848226.png)

- **prototype**：原型，Spring容器在初始化时不创建Bean实例，加载配置文件，创建Spring容器时不会创建Bean对象。当每次调用getBean方法时才会创建Bean对象，而且每次调用getBean方法时都会创建一个新的Bean对象，用完之后，由于没有对Bean对象的引用，最终就释放了，释放之后就会被gc垃圾回收。

测试singleton：

![image-20250221140941457](Spring.assets/image-20250221140941457.png)

![image-20250221141303702](Spring.assets/image-20250221141303702.png)

测试prototype：

![image-20250221141550354](Spring.assets/image-20250221141550354.png)

![image-20250221142659005](Spring.assets/image-20250221142659005.png)

一般情况下使用scope属性的默认值，也就是singleton即可。

![image-20250221142844315](Spring.assets/image-20250221142844315.png)

![image-20250221142902519](Spring.assets/image-20250221142902519.png)

![image-20250221144701535](Spring.assets/image-20250221144701535.png)

![image-20250221144751249](Spring.assets/image-20250221144751249.png)

**扩展——scope属性的属性值：**

如果当前是一个基础的Spring环境，scope的取值就只有两个，一个singleton，一个prototype。

![image-20250221143444880](Spring.assets/image-20250221143444880.png)

面试的时候回答这两个基本就可以了。

- singleton：单例，Spring容器创建完，加载完配置文件就会创建Bean对象，并且存储到SingletonObjects单例池当中，每次使用直接从单例池内部拿即可。
- prototype：原型，Spring容器创建完，配置文件加载完，不会立马创建Bean对象，而是在调用getBean方法的时候创建Bean对象，并且不会存入到单例池当中。

更标准的回答是，如果是一个Web环境，scope属性的取值不只上面两种。

**pom.xml文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>spring_ioc_test01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.23</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>6.0.11</version>
        </dependency>
        <!-- 添加springmvc的依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>6.0.11</version>
        </dependency>
    </dependencies>

</project>
```

![image-20250221144307715](Spring.assets/image-20250221144307715.png)

如果是在SpringMVC环境下，scope属性会多出两个属性值：

- **request**：当把scope属性设置为request的时候，Spring容器会把创建的对象放到request域当中。
- **session**：当把scope属性设置为session的时候，Spring容器会把创建的对象放到session域当中。



##### Bean的延迟加载

![image-20250221144831559](Spring.assets/image-20250221144831559.png)

默认情况下ApplicationContext是立即加载，也就是说当这个Spring容器创建完毕，配置文件加载完毕之后马上创建Bean。如果不想让ApplicationContext马上创建Bean，就为bean标签设置lazy-init属性，表示是否延迟加载，设置为true表示延迟加载，ApplicationContext就不会马上创建Bean，而是调用getBean方法时才创建，并且创建完毕之后，把Bean放到singletonObjects单例池当中。

**注意**：lazy-init属性对BeanFactory无效，如果客户端使用BeanFactory，不管是否设置lazy-init属性，就算把lazy-init设置为false，BeanFactory还是延迟加载。

lazy-init属性只对ApplicationContext有效。



测试：

如果bean标签没配置lazy-init属性：

![image-20250221155601392](Spring.assets/image-20250221155601392.png)

如果bean标签配置lazy-init属性：

![image-20250221160037334](Spring.assets/image-20250221160037334.png)

![image-20250221160142725](Spring.assets/image-20250221160142725.png)

![image-20250221160256747](Spring.assets/image-20250221160256747.png)

![image-20250221160357468](Spring.assets/image-20250221160357468.png)

**注意**：设置lazy-init属性为true和设置scope属性为prototype不一样，设置scope属性为prototype，创建的Bean对象是不会存放到SigletonObjects单例池当中，因为可以创建多个相同Bean的对象，用完就销毁了。lazy-init延迟加载是指先不创建Bean，也就是说在Spring容器创建完，配置文件加载完，Bean对象并没有创建，而是等到调用getBean方法的时候创建Bean，创建的Bean对象仍然会存入SingletonObjects单例池当中，后期谁用直接从单例池当中取就行。



##### 初始化方法和销毁方法

![image-20250221160924054](Spring.assets/image-20250221160924054.png)

Bean在创建对象之后可以完成一些基本的初始化操作，我们可以在配置的时候指定初始化方法；同样，在销毁的时候也能指定销毁的方法。

首先需要在Bean中声明一个初始化方法和一个销毁方法，初始化方法和销毁方法的名字自定义，等下配置的时候要使用。

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {
    public void init() {
        System.out.println("初始化方法...");
    }

    public void destroy() {
        System.out.println("销毁方法...");
    }
}
```

只是在Bean中声明还不够，此时它们就是一个普通方法，如果想把它们分别定义成初始化方法和销毁方法，还需要通过xml配置文件去配置，因为初始化方法和销毁方法本身是Spring自动帮我们调用，在配置文件当中为bean标签配置init-method属性用来指定初始化方法，为bean标签配置destroy-method属性，用来指定销毁方法。

**applicationContext.xml文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 配置UserServiceImpl -->
    <bean id="userService" name="aaa,bbb,ccc" class="com.itheima.service.impl.UserServiceImpl" init-method="init" destroy-method="destroy">
        <property name="userDao" ref="userDao"></property>
    </bean>
    <!-- 配置UserDaoImpl -->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
</beans>
```

![image-20250221165427281](Spring.assets/image-20250221165427281.png)

**注意**：先执行Bean的构造方法，因为构造方法代表创建Bean对象，初始化方法是在Bean对象创建完毕之后，Spring自动帮我们执行的方法。

问题：销毁方法没有执行。

原因：因为我们没有显式的去关闭Spring容器。这就涉及到Spring容器当中的Bean对象销毁的时机。容器当中的Bean默认是singleton单例的，当这个容器显式的关闭，也就是容器调用close方法时，如果在xml配置文件当中的bean标签配置了destroy-method属性，那么Spring会帮我们调用对应的Bean的销毁方法。但是如果容器不是显式的销毁，也就是容器没有调用close方法，比如：代码执行完，容器并不知道自己要关闭，所以Spring容器根本就来不及帮我们去调用销毁方法就挂掉。

**注意**：ApplicationContext接口内部没有close关闭方法，得用它的子类，比如ClassPathXmlApplicationContext内部有close方法。

```java
package com.itheima.test;

import com.itheima.dao.UserDao;
import com.itheima.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ApplicationContextTest {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = (UserService) applicationContext.getBean("userService");//()里面是beanName
        System.out.println(userService);
        applicationContext.close();
    }
}
```

Spring容器调用close方法，这个时候Spring容器发现自己要关闭，所以Spring容器在关闭之前把容器内部所有配置了销毁方法的Bean对应的方法都调用一遍。

![image-20250221171138008](Spring.assets/image-20250221171138008.png)

destroy-method属性基本不会使用，init-method属性偶尔用一下，就是当这个Bean对象创建完毕之后要进行初始化操作，我们把要做的事情的一些逻辑代码写到初始化方法当中。

**强调**：Bean的销毁和Bean的销毁方法的调用是两回事。有的时候Spring容器已经没有了，比如ApplicationContext这个对象已经在内存当中没有了，也就是被销毁了，那么ApplicationContext内部维护的那些Bean对象也会被销毁，但是为什么Bean里面的销毁方法没有被调用呢？这是因为Spring还没有执行到调用Bean的销毁方法这一步，这个Spring容器就挂掉了。

所以说对象没有了但是不执行Bean的销毁方法，是因为Spring容器可能还没有执行到销毁方法就挂掉了，对应的nameBean也没有。Bean对象已经销毁了，但是销毁方法Spring容器没有调用，这就是Bean的销毁和Bean的销毁方法的调用的区别。

所以我们需要显式关闭Spring容器，让Spring容器知道它自己马上要挂掉了，让它自己在挂掉之前做一些善后的工作，比如去调用一些Bean的销毁方法。



##### InitializingBean方式

![image-20250221212220382](Spring.assets/image-20250221212220382.png)

初始化的第二种方式：让Bean实现**InitializingBean接口**，在这个接口内部有一个方法——**afterPropertiesSet方法**。在这个方法内部写的一些业务也可以在初始化时被执行，所以当我们实现InitializingBean接口之后，Spring容器在执行时发现Bean实现了这个接口，Spring容器就会自动的帮我们去调用afterPropertiesSet方法。

afterPropertiesSet中文为在属性设置之后执行，简单来说就是：Service层要调用Dao层，即UserServiceImpl内部需要注入UserDaoImpl，Dao注入给Service就是**属性设置**，属性设置执行之后，再调用afterPropertiesSet方法。

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.service.UserService;
import org.springframework.beans.factory.InitializingBean;

public class UserServiceImpl implements UserService, InitializingBean {
    public void init() {
        System.out.println("初始化方法...");
    }

    public void destroy() {
        System.out.println("销毁方法...");
    }


    public UserServiceImpl() {
        System.out.println("UserServiceImpl实例化");
    }

    private UserDao userDao;
    
    public void setUserDao(UserDao userDao) {
        System.out.println("属性设置完毕...");
        this.userDao = userDao;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("afterPropertiesSet执行...");
    }
}
```

afterPropertiesSet方法定义之后，不需要在配置文件当中额外对这个方法进行配置，因为接口对应一个规范，而且Bean又在Spring容器当中，Spring容器要去判定Bean有没有实现InitializingBean接口，如果Bean实现了InitializingBean接口，Spring容器自然会帮我们调用afterPropertiesSet方法。

![image-20250221214252202](Spring.assets/image-20250221214252202.png)

验证afterPropertiesSet方法在属性设置之后执行：

![image-20250221215154303](Spring.assets/image-20250221215154303.png)

**总结：**

Bean实现InitializingBean接口，接口里面的afterPropertiesSet方法跟Bean的生命周期相关。

Bean对象创建完毕之后，想要做一些初始化工作，至少有两种方式：

1. 第一种：Bean定义一个初始化方法，然后在xml配置文件里面的bean标签设置init-method属性指定初始化方法。
2. 第二种：Bean实现InitializingBean接口，然后去重写afterPropertiesSet方法。



##### 实例化Bean的方式-构造方法方式

![image-20250221230409597](Spring.assets/image-20250221230409597.png)

Spring实例化的配置就是Spring帮我们创建Bean的方式，分为两种：第一种是通过反射找构造方法，第二种是工厂方式。

这里有个误区：ApplicationContext.getBean()底层调用的也是BeanFactory.getBean()对应的方法，BeanFactory本身就是一个Bean工厂，也就是说整个过程采用的是工厂模式，但是在Bean工厂内部帮我们创建Bean的时候又分为两种方式：第一种方式就是通过反射找Bean的构造方法帮我们创建Bean对象，Bean工厂返回Bean对象。第二种方式是在Bean工厂内部使用工厂方式帮我们创建Bean，可以认为是工厂内部套工厂，第一层工厂是BeanFactory，也就是DefaultListableBeanFactory内部帮我们创建Bean的时候，可以通过无参构造方式创建Bean，也可以通过工厂方式创建Bean，工厂需要我们去提供，这个自定义的工厂内部可以去创建一个Bean，这个Bean再由Spring容器帮我们调用和管理。

![image-20250221233440149](Spring.assets/image-20250221233440149.png)

如果在xml配置文件当中没有配置**constructor-arg**标签，默认使用无参构造方式创建Bean；如果配置constructor-arg标签，表示使用有参构造方式创建Bean，可以配置多个constructor-arg标签，一个constructor-arg标签对应有参构造方式的一个形参。Spring容器采用哪种方式帮我们创建Bean取决于xml配置文件。

测试使用无参构造方式创建Bean：

![image-20250221235803177](Spring.assets/image-20250221235803177.png)

![image-20250222000150321](Spring.assets/image-20250222000150321.png)

![image-20250222000505300](Spring.assets/image-20250222000505300.png)

这种情况下在xml配置文件当中配置有参构造去创建Bean。

**applicationContext.xml：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userService" name="aaa,bbb,ccc" class="com.itheima.service.impl.UserServiceImpl">
        <constructor-arg name="name" value="haohao"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
        <property name="userDao" ref="userDao"></property>
    </bean>
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
</beans>
```

在bean标签里面嵌套一个子标签constructor-arg。constructor-arg标签里面设置两个属性：

- **name**：name属性值为Bean的有参构造函数里面的形参名。
- **value**：自定义，随便写。

如果Bean的有参构造里面有多个形参，可以在bean标签里面嵌套多个constructor-arg标签，一个constructor-arg标签对应一个形参。

![image-20250222001210065](Spring.assets/image-20250222001210065.png)

![image-20250222001722312](Spring.assets/image-20250222001722312.png)

配置有参构造让Spring容器使用Bean的有参构造去创建对象，在开发当中用的特别少。几乎都是使用默认情况下Bean的无参构造去创建Bean对象。

其实有参构造和无参构造的区别在于xml配置文件当中的bean标签是否嵌套constructor-arg标签。是否配置constructor-arg标签又取决于Bean的构造函数当中的形参个数。

**注意点**：constructor-arg中文叫构造参数，构造参数不仅仅指的是Bean的构造方法的参数，也就是说，constructor-arg子标签不仅仅可以用于配置构造函数的参数，只要是在Bean创建过程中需要的参数，都可以用constructor-arg标签进行传输。比如：使用Bean工厂方式创建Bean的过程中需要参数，这些参数也是构造bean时需要的参数，也属于构造参数，可以用constructor-arg标签进行传入。所以不仅仅只有配置构造方法时使用constructor-arg标签。



##### 静态工厂方法方式

![image-20250222003930671](Spring.assets/image-20250222003930671.png)

好，那我们继续啊，刚才呢，我们已经用这个构造方法啊，去可以配置我们的这个bin是吧，不管是有餐的还是无餐的，它本质都是我们的构造方法。那刨除构造方法之外呢？我们说还有一种就是工厂方式去实例化我们的bin啊，那么看一下工厂方式，工厂方式呢？在这儿我又给大家归结为三种啊。第一种叫共态静态工厂方法。第二种呢，叫实力工厂方法，第三种呢，
呃，实现我们的这个的方式，那第三种啊，在这我一句话半句话说不明白，但第一种第二种啊，我可以稍微然后介绍一下啊。静态工厂方法说白，就是说什么就是我定一个工厂，哎，我定一个工厂就自定义一个工厂，那这个工厂内部呢，提供个静态方法。在这个静态方法内部呢，我去产生一个并啊，最终这个并呢，
哎，我要交给spring容器，然后去帮我管理。OK，那第二种呢？实例工厂方法呢？呃，其实相对静态就是说我有个工厂，这工厂内部的方法呢？是非静态的。啊，那那最终这个非静态方法，然后产生的bin呢，就是返回值啊，最终我交给spring容器，
然后去管理啊。它们其实最主要的区别是在于什么呢？在于静态工厂方法，它是不是方法是静态的啊？非静态工厂方法啊？它的这个这个方法是非静态的，所以我们在配置的时候，我们要知道。静态工厂方法，它在产生这个就是在调用它静态方法时，我这个工厂静态工厂它不需要现有对象。对吧，因为静态方法直接，然后这个匿名去调静态方法，就但实例工厂方法就不一样了，
你这个然后实例工厂必须需要工厂对象。然后再说用工厂对象去调对应的一些方法啊，这是它一个最主要的区别，行吧，那下面我们就配一个吧，咱们配一个啊。呃，我们回到idea当中。呃，首先在这儿呢，我再建一个package。啊，就叫factory吧，叫factory好在这个内部呢，我去创建一个就叫。
叫static吧啊，叫static，然后这个这儿吧，别叫static呢。我叫bin。叫my be factory。factory是吧诶一。可以吧，脉变factory 1好，创建好之后呢？内部我们提供一个静态方法就行。static返回值呢，你随便我们就返回个do吧好吧user dao。啊，就叫user dao。
啊，最终我在这return一个。new一个。有点电影m片。是不是写完了吧？写完之后，现在然后我们去产生b原先是怎么产生的？就原先是不是spring容器，它通过这个权限的名儿。就是那个全包名儿啊，反射创建好对象，放到容器当中是吧？而现在呢，是我们spring spring这个容器帮你去调这个。力的这个静态方法最终将返回的这个对象存储到spring容器当中。
啊，是一个过程吧好，那下面呢？我们主要在于怎么去配置吧？怎么去配右键copy它的权限的名儿？回到我们的配置文件当中。哎，在这。诶，你原先怎么配？现在你还怎么配？你比如原先我们是不是先配个这个这个这个ID？比如我叫叫什么呀？叫user。do.
一行吧，对应的权限里面是它。它那如果你这样配完之后，这跟原先我配个普通的是不是一样的是吧？当你启动时，那spring容器会干什么事儿呢？会按照你的全限定名儿给你创建，创建对象就使用它对象嘛，这个就走的是无三构造对不对，然后呢，创建完对象之后放到容器当中，那放到容器当中。是不是这个bin的有个bin name呀？那此时的bin name是不是叫ID uz doe对不对？但但是呢，
现在我是想找它的方法，所以说在这个地方你加什么呀？加个叫。指定它的那个方法啊，我的方法是不是就叫user du这个方法了？是吧哎，所以说你在这儿就指定这个方法叫那此时当你这样配完之后，磁盘容器当看到时哎在分在这个解析时哦发现哦，你有个factory method，那我知道呢不是。把它变成对象。是找它内部的这个user do这个方法，把这个方法的返回值啊，当做这个对象，再以你指定的ID作为bna me。
存储到容器当中。这能明白吧，好下边我们测试一下啊，先保证我的测试是没有问题的。呃，这些代码我先干掉吧，有容器呢。GE TB我叫什么名字诶？这pay你叫什么名字？是不是叫user doe对不对？好在这user doe。行吧，那就不不强转的啊，不强转的我在这user do我打一下地址就行执行。诶，
稍等。你看是不是没问题，看看别看第一个，第一个是我们那构造打那个service构造打印的画啊，是不是最终我打印没有问题，是不是能创建吧？好能创建，那其实在这个地方呢，我可以再加个断点。走往里走一步。我们看谁呀？我们看那个就是那个当地时对不对？在这有b factory。又找不到了，它下边儿有个bfa x这儿呢，
然后进去，然后有个singlet objects呃，不是这个吧哎，点进去。然后再点你找是不它有个user due？是不是user du 1吧啊？user du 1你再去点，你发现这个user du 1是不是就是user du ipl这对象对不对哎？它对应是这个对象并不是这个工厂对象啊。因为我这种配置spring容器是能识别的。okay，这点儿能听懂吧？但再回到这儿说，这种方式有什么用呢？这么麻烦，
是不是？那叫什么用啊？大家想想啊。想想啊，第一个用处在于原先你在去呃，自己去创建这个病，让死命去管理的话，那你这个病在创建前后。你是没有办法去执行其他代码的。就是这个变，我说马上要创建呢是吧？创建之前我想干点事儿，原先你不好干，但通过这种工厂方式，那就那比如在这个地方。
加个注释，就是说这个bin。创建之前之前我可以要进行一些其他的页面逻辑操作。对不对？那在创建之后呢？是不是也可以进行一些其他业务轨迹操作，比如我把它踢到外边是吧，然后执行完之后最终再把它返回？这能明白吧，所以你这个这个逻辑可以更加灵活一点。这是这是第一个是吧？那第二个原因呢？大家想想你看啊，还是说我们这些是不是都是我自定义的吧？啊，
就是这个b factory也好啊，这个user dom PR也好，是不是都是自定义的是吧啊？大家在想，其实有一些啊，有一些bin是非我自定义的。就什么意思？就是人家炸包已经给你定义好的。就其他炸包啊，给你定义好的，比如说你在开发时，你不可能都是自定义的吧，你是不可能要演一些工具啊，或演一些，然后第三方的一些炸包啊，
对不对？那如果第三方炸包当中的哎，某些病你也需要交给spring去管是吧？你是不是得配置？可能有时候这个病本身它就不是，然后通过有参构造和无参构造去生成的是吧？它可能就是通过某些静态方法产生的，而那些病你在配置时，你就可以用这种。什么呀，这种。静态构造就是静态工厂的方式，把它配到spring容器当中。啊，那打个比方，
比如说。大家知道我们这个什么呢？呃，我在这写一个啊，大家知道我们当时在开发时，我们的gd BC是吧？gd BC。GG BC是不是那个connection怎么去获取的？draw manager.当然，我这没导包啊，就装manager点儿，是不是有个get connection？tln是吧connection，然后这伪代码呢？
因为这没导包，所以都爆红了是吧？哎，大家想哎，就这就这个方法是吧？那比如说它最终的反折是不是connection？是不是connection那如果现在我想让spring哎，帮我去管connection，那connection怎么去产生呢？是不是通过然后一个a的一个方法最终产生的connection呢？对吧，那此时我是不就可以把它看成是一个静态工厂方法产生connection？但这种方式不会这么配啊，你比方讲完之后回去真呃傻呵呵的咔咔的把那个connection要配到容器当中，用这种方式是吧，
我就给大家打个比方，我现在暂时想不出来其他的呢是吧？就是说有很多，然后非我自定义的病它的，然后产生确实是通过，然后这种静态方法产生的，那此时我就可以用。哎，刚才学的这种什么呀？哎，静态工厂的方式进行配置啊。好吧，那就是关于静态工厂方式配置。我们的变啊。



##### 实例工厂方法方式



##### 有参数的静态工厂和实例工厂方法



##### 实现FactoryBean规范延迟Bean实例化





#### Spring的get方法



#### Speing配置非自定义Bean



#### 基于xml方式的Bean的配置预览



#### Bean实例化的基本流程



#### Spring的后处理器



#### Spring Bean的生命周期



#### Spring IoC整体流程总结



#### Spring xml方式整合第三方框架





### 注入方式和注入数据类型



### 注入集合数据类型



### 自动装配



### 命名空间的种类



### beans的profile属性切换环境



### import标签



### alias标签



### 自定义命名空间标签的使用步骤



## AOP面向切面编程



## Spring集成Web环境



## SpringMVC







