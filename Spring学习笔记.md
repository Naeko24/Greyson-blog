# Spring学习笔记

## 一、Spring入门案例

1. 通过Idea创建maven项目
2. 配置spring配置文件ApplicationContext.xml
3. 编写接口及实现类
- IaccountDao

        /**
         * 账户的持久层接口
         */
        public interface IAccountDao {
        
            /**
             * 模拟保存账户
             */
            void saveAccount();
        }

- IaccountService

        /**
         * 账户业务层的接口
         */
        public interface IAccountService {
        
            /**
             * 模拟保存账户
             */
            void saveAccount();
        }

- AccountDaoImpl

        /**
         * 账户的持久层实现类
         */
        public class AccountDaoImpl implements IAccountDao {
        
            public  void saveAccount(){
        
                System.out.println("保存了账户");
            }
        }

- AccountServiceImpl

        /**
         * 账户的业务层实现类
         */
        public class AccountServiceImpl implements IAccountService {
        
            private IAccountDao accountDao = new AccountDaoImpl();
        
            public void  saveAccount(){
                accountDao.saveAccount();
            }
        }

4. 编写测试类Client

ApplicationContext的三个常用实现类：

- ClassPathXmlApplicationContext： 它可以加载路径下的配置文件，要求配置文件必须在路径下，否则加载不了

        ApplicationContext ac = new ClassPathXmlApplicationContext("bea
        ns.xml");

- FileSyetemXmlApplicationContext：它可以加载磁盘下任意路径下的配置文件（必须有访问权限）

    加载方式如下：

        ApplicationContext ac = new FileSystemXmlApplicationContext("C:\\user\\greyson\\...")

- AnnotationConfigApplicationContext：它是用于读取注解创建容器的

        /**
         * 模拟一个表现层，用于调用业务层
         */
        public class Client {
            /**
             *
             *获取IOC的核心容器，并根据id获取对象
             * @param args
             */
            public static void main(String[] args) {
                ApplicationContext ac = new ClassPathXmlApplicationContext("beans.xml");
                // 两种不同的方式获取Bean对象
                IAccountService as = (IAccountService) ac.getBean("accountService");
                IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);
                System.out.println(as);
                System.out.println(adao);
        //        as.saveAccount();
            }
        }

核心容器的两个接口引发出来的问题

- ApplicationContext：它在创建核心容器时，创建对象采取的策略是采用立即加载的方式，也就是说，只要一读取完配置文件就马上创建配置文件中配置的对象
    - 单例对象适用
    - 开发中常采用此接口
- BeanFactory:它在构建核心容器时，创建对象的策略是采用延迟加载的方式，什么时候获取id对象了，什么时候就创建对象。
    - 多例对象适用

## 二、Spring中Bean的细节

### 1.三种创建bean对象的方式

(1) 使用默认构造函数创建

在spring的配置文件中，使用id和class属性之后，且没有其他属性和标签时，采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建。

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl"></bean>

(2) 使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入spring容器）,如下

    /**
     *模拟一个工厂类，该类可能存在于jar包中，无法通过修改源码的方式来提供默认构造函数
     * 
     */
    public class InstanceFactory {
        public IAccountService getAccountService() {
            return new AccountServiceImpl();
        }
    }

配置方式如下：

    <bean id = "instanceFactory" class = "com.itheima.factory.InstanceFactory"></bean>
        <bean id = "accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>

(3) 使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器），如下：

    public class StaticFactory {
        public  static IAccountService getAccountService() {
    
            return new AccountServiceImpl();
        }
    }

配置方式如下：

    <bean id = "accountService" class = "com.itheima.factory.StaticFactory" factory-method="getAccountService"></bean>

### 2.bean的作用范围调整

(1) bean标签的scope属性

作用：用于指定bean的作用范围

取值：常用的就是单例和多例

- singletond : 单例的（default）
- prototype : 多例的
- request : 作用于web应用的请求范围
- session : 作用于web应用的会话范围
- global-session : 作用于集群的会话范围（全局会话范围），当不是集群范围时，它就是session

![](session-988affbd-bff9-48fc-bb1d-5e135fe32082.png)

(2) bean对象的声明周期

单例对象：

- 出生：当容器创建时发生
- 活着：只要容器还在对象就一直活着
- 死亡：容器销毁，对象消亡

总结：单例对象的声明周期和容器相同

多例对象：

- 出生：当我们使用对象时spring框架为我们创建
- 活着：对象只要是在使用过程中就活着
- 死亡：当对象长时间不用，且没有别的对象引用时，由Java的GC回收

## 三、依赖注入（Dependency Injection）

### 1.概述

IOC的作用：减低程序间的耦合（即依赖关系）

在当前类需要用到其他类的对象，由spring为我们提供，而我们在配置文件中说明依赖关系的维护，这种方式就称为依赖注入。

能注入的数据：

- 基本类型和String
- 其他bean类型（在配置文件中或者注解中配置过的bean）
- 复杂类型/集合类型

### 2.注入方式

1.第一种：使用构造函数提供

使用的标签：constructor-arg

标签所在位置：bean标签的内部

标签中的属性：

- type : 用于指定要注入的数据类型，该类型也是构造函数中某个或某些参数的类型
- index : 用于指定要注入的数据给构造函数中指定索引位置的参数赋值，索引的位置时从0开始
- name(常用) : 用于指定给构造函数中指定名称的参数赋值
- value : 用于提供基本类型和String类型的数据
- ref : 用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器出现过的bean对象

例：

    public class AccountServiceImpl implements IAccountService {
        // 如果时经常变化的数据不适用于依赖注入，此处仅为演示
        private String name;
        private Integer age;
        private Date birthday;
    
        public AccountServiceImpl(String name, Integer age, Date birthday){
            this.name = name;
            this.age = age;
            this.birthday = birthday;
        }
    
        public void  saveAccount() {
            System.out.println("service中的saveaccount()执行了");
        }
    
    }

测试类：

    public static void main(String[] args) {
            //1.获取核心容器对象
            ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
            //2.根据id获取Bean对象
            IAccountService as  = (IAccountService)ac.getBean("accountService");
            as.saveAccount();
        }

配置如下：

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
            <constructor-arg name = "name" value="taylor"></constructor-arg>
            <constructor-arg name = "age" value = "23"></constructor-arg>
            <constructor-arg name = "birthday" ref = "now"></constructor-arg>
        </bean>
    
        <bean id = "now" class = "java.util.Date"></bean>

特点：在获取bean对象时，注入数据是必须的操作，否则无法操作成功

弊端：改变了bean对象的实例化方式，使我们在用不到这些数据的情况下也必须提供带参构造函数，因此开发中较少使用此方法，除非避无可避

2.第二种：使用set方法提供（更常用的方式）

使用的标签：property

出现的位置：bean标签的内部

标签的属性：

name : 用于指定注入时所使用的set方法

value : 用于提供基本类型和String类型的数据

ref : 用于指定其他的bean类型数据，它指的就是在Spring容器中出现过的bean对象

优势：创建对象时没有明确的限制，可以直接使用默认构造函数

弊端：如果有某个成员必须有值，是有可能set方法没有执行

(1) 基本类型和String的注入方式

    public class AccountServiceImpl implements IAccountService {
        // 如果时经常变化的数据不适用于依赖注入，此处仅为演示
        private String name;
        private Integer age;
        private Date birthday;
    
        public void setName(String name) {
            this.name = name;
        }
    
        public void setAge(Integer age) {
            this.age = age;
        }
    
        public void setBirthday(Date birthday) {
            this.birthday = birthday;
        }
    
        public void  saveAccount() {
            System.out.println("service中的saveaccount()执行了" + name + "," + age + "," +birthday);
        }
    
    }

配置如下：

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
            <property name="name" value ="taylor"></property>
            <property name="age" value="21"></property>
            <property name="birthday" ref="now"></property>
        </bean>
    
        <bean id = "now" class = "java.util.Date"></bean>

测试类同上

(2) 复杂集合类型的注入方式

用于给List结构集合注入的标签

- list
- array
- set

用于给map结构集合注入的标签

- map
- properties

结构相同，标签可以互换，因此开发中只要记住两组标签即可

类：

    public class AccountServiceImpl implements IAccountService {
    
        private String[] myStrs;
        private List<String> myList;
        private Set<String> mySet;
        private Map<String, String> myMap;
        private Properties myProps;
    
        public void setMyStrs(String[] myStrs) {
            this.myStrs = myStrs;
        }
    
        public void setMyList(List<String> myList) {
            this.myList = myList;
        }
    
        public void setMySet(Set<String> mySet) {
            this.mySet = mySet;
        }
    
        public void setMyMap(Map<String, String> myMap) {
            this.myMap = myMap;
        }
    
        public void setMyProps(Properties myProps) {
            this.myProps = myProps;
        }
    
        public void  saveAccount() {
            System.out.println(Arrays.toString(myStrs));
            System.out.println(myList);
            System.out.println(myMap);
            System.out.println(mySet);
            System.out.println(myProps);
        }
    
    }

配置如下：

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
            <!--以下三个标签是等价的，set未列出-->
            <property name="myList">
                <list>
                    <value>aaa</value>
                    <value>bbb</value>
                </list>
            </property>
    
            <property name="myStrs">
                <array>
                    <value>aaa</value>
                    <value>bbbb</value>
                </array>
            </property>
    
            <property name="mySet">
                <array>
                    <value>aaa</value>
                    <value>bbbb</value>
                </array>
            </property>
    
            <!--以下两种方式等价-->
            <property name="myMap">
                <map>
                    <!--以下两种配置方式都可以-->
                    <entry key="testA" value="aaa"></entry>
                    <entry key="testA">
                        <value>bbb</value>
                    </entry>
                </map>
            </property>
    
            <property name="myProps">
                <props>
                    <prop key="testB">bbb</prop>
                </props>
            </property>
        </bean>

3.第三种：使用注解提供

**如何使用？**

第一步：在类或方法的前面加上注解关键字

第二步：引入约束,注意此处约束多了xmlns:context...

第三步：添加配置文件，告知spring在创建容器时要扫描的包，配置所需的标签不是在bean约束中，而是一个名称为context的名称孔家和约束中,完整配置如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    
        <context:annotation-config/>
        <context:component-scan base-package="com.itheima"></context:component-scan>
    </beans>

**有哪些注解？**

(1)用于创建对象的

作用：等同于xml配置文件中编写一个<bean>标签

@Component

形式：@Component(value=" ")/@Component(" ")

作用：用于把当前类对象存入spring容器中

属性：

value : 用于指定bean的id，当我们不写的时候，它的默认值是当前类名，且首字母改小写;当值只有一个的时候可以省略

以下三个注解的作用与@Component完全一样，它们是spring提供的更明确的划分，使三层对象更加清晰

- @Controller  用于表现层
- @Service       用于业务层
- @Repository 用于持久层

(2) 用于注入数据的

作用：等同于在<bean>标签中写一个<property>标签

@Autowired

作用：自动按照类型注入，只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功如果IOC容器中没有任何bean的类型和要注入的变量类型匹配，则报错

出现位置：可以是变量上，也可以是方法上，

细节：在使用注解注入时，set方法就不是必须的了

@Qualifier

作用：在按照类型注入的基础上再按照名称注入，它在给类成员注入时不能单独使用，但是在给方法参数注入          时可以。

属性：

value : 用于指定注入的bean的id

@Resource

作用：直接按照bean的id注入，可以直接使用

属性：

name : 用于指定bean的id

等同于@Autowired+@Qualifier

以上三个注入都只能注入其他bean类型的数据，而基本类型和String类型的数据无法使用上述注解实现。另外，集合类型的注入只能通过xml配置文件实现

@Value

作用：用于注入基本类型和String类型的数据

属性：

value : 用于指定数据的值，它可以使用spring中Spel(即spring的el表达式)

Spel的写法：${表达式}

**操作实例：**

接口如下：

    public interface IAccountDao {
    
        void saveAccount();
    }

    public interface IAccountService {
    
        /**
         * 模拟保存账户
         */
        void saveAccount();
    }

实现类：

    @Service("accountService")
    public class AccountServiceImpl implements IAccountService {
    
        @Autowired
        @Qualifier("accountDao2")
        private IAccountDao accountDao = null;
    
    
        public void  saveAccount() {
            accountDao.saveAccount();
        }
    
    }

    @Repository("accountDao1")
    public class AccountDaoImpl implements IAccountDao {
    
        public void  saveAccount() {
            System.out.println("对象创建了111");
        }
    
    }

    @Repository("accountDao2")
    public class IAccountDaoImpl2 implements IAccountDao{
    
        public void  saveAccount() {
            System.out.println("对象创建了222");
        }
    }

测试类：

    public static void main(String[] args) {
            //1.获取核心容器对象
            ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
            //2.根据id获取Bean对象
            IAccountService as  = (IAccountService)ac.getBean("accountService");
    
            as.saveAccount();
        }
    }

如上，AccountDaoImpl1和AccountDaoImpl2实现接口IAccountDao,两个类中分别实现了不同的saveAccount()方法，AccountServiceImpl实现接口IAccountService,其中调用了IAccountDao接口。AccountServiceImpl通过注解关键字Autowired去spring容器中寻找accountDao,再根据Qualifier配置的value找到两个dao的实现类中与之相匹配的Repository的值。

(3) 用于改变范围的

作用：等同于在<bean>标签中使用scope属性

@Scope

作用：用于指定bean的作用范围

属性：

value : 指定范围的取值，同xml中值，常用为singleton, prototype

(4) 和生命周期相关（了解）

作用：等同于在<bean>标签中使用init-method和destroy-method

@PreDestory

作用：用于指定销毁方法

@Postcontrust

作用：用于指定初始化方法