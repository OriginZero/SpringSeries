# Spring整合Mybatis

## 整合Mybatis 方式一

### 1. 导入相关jar包

+ junit	
+ mybatis
+ mybatis-spring
+ mysql驱动
+ spring mvc
+ spring-jdbc
+ aop织入

> pom.xml 所需要的jar包

```xml
<dependencies>
    <!--junit-->
    <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>

    <!--mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.4</version>
    </dependency>

    <!--mybatis-spring MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中-->
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.4</version>
    </dependency>

    <!--spring 全家桶-->
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.5.RELEASE</version>
    </dependency>

    <!--mysql 驱动-->
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.9-rc</version>
    </dependency>

    <!--spring 操作数据库 需要 spring jdbc-->
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.5.RELEASE</version>
    </dependency>

    <!--spring 依赖 zhi'ru织入包-->
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.5</version>
    </dependency>

</dependencies>
```



### 2. 编写核心配置文件

> 编写spring注入mybatis配置，建议放入spring-dao.xml文件中

#### 1.DataSource

> 在Spring中编写数据源

```xml
<!--第一步 设置数据源 org.springframework.jdbc.datasource.DriverManagerDataSource-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <!--驱动包名-->
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <!--数据库连接url-->
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8&amp;useSSL=false"/>
    <!--数据库用户名-->
    <property name="username" value="zero"/>
    <!--数据库密码-->
    <property name="password" value="zero"/>
</bean>
```



#### 2.SqlSessionFactory

> 通过SqlSessionFactoryBean在Spring配置中加入SqlSessionFactory

```xml

<!--第二步 设置sqlSessionFactory-->
<!--通过SqlSessionFactoryBean获得sqlSessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--注入数据库数据源-->
    <property name="dataSource" ref="dataSource"/>
    <!--配置MyBaties全局配置文件:mybatis-config.xml-->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!--绑定mapper文件-->
    <property name="mapperLocations" value="classpath:mapper/*.xml"/>
</bean>
```



#### 3.sqlSessionTemplate

> 通过sqlSessionTemplate获取sqlSession

```xml
<!--第三步 通过sqlSessionTemplate把sqlSession到springzhong-->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <!--通过构造器注入属性-->
    <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
```

> spring-dao.xml 全部配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--第一步 设置数据源 org.springframework.jdbc.datasource.DriverManagerDataSource-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--驱动包名-->
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <!--数据库连接url-->
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8&amp;useSSL=false"/>
        <!--数据库用户名-->
        <property name="username" value="zero"/>
        <!--数据库密码-->
        <property name="password" value="zero"/>
    </bean>

    <!--第二步 设置sqlSessionFactory-->
    <!--通过SqlSessionFactoryBean获得sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--注入数据库数据源-->
        <property name="dataSource" ref="dataSource"/>
        <!--配置MyBaties全局配置文件:mybatis-config.xml-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!--绑定mapper文件-->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>

    <!--第三步 通过sqlSessionTemplate把sqlSession到springzhong-->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <!--通过构造器注入属性-->
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
</beans>
```



#### 4.给Dao层接口添加实现类

> 实现Dao层接口

```java
// UserMapper接口
public interface UserMapper {
    // 查询所有用户
    List<User> selectAllUser();
}

// UserMapper实现类
public class UserMapperImpl implements UserMapper {
    // SqlSession对象线程安全的
    private SqlSession sqlSession;

    // 需要通过Spring的Set方式属性注入SqlSession属性
    // SqlSession是通过SqlSessionTemplate多态形成的
    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    //实现方法
    public List<User> selectAllUser() {
        return sqlSession.getMapper(UserMapper.class).selectAllUser();
    }
}
```



#### 5. 整合Spring全局配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--导入其他spring配置-->
    <import resource="classpath:spring/spring-dao.xml"/>

</beans>
```



#### 6.将实现类注入到Spring中

```xml
<!--注入userMapper-->
<bean id="userMapper" class="com.zhang.mapper.UserMapperImpl">
    <property name="sqlSession" ref="sqlSession"/>
</bean>
```

> applicationContext.xml完整配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--导入其他spring配置-->
    <import resource="classpath:spring/spring-dao.xml"/>

    <!--注入userMapper-->
    <bean id="userMapper" class="com.zhang.mapper.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```



### 3. 测试

```java
@Test
public void selectAllUserTest(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserMapper userMapper = applicationContext.getBean("userMapper", UserMapper.class);
    for (User user : userMapper.selectAllUser()) {
        System.out.println(user);
    }
}
```





## 整合Mybatis 方式二

> 主要是Dao层实现类通过继承SqlSessionDaoSupport来提供SqlSessionTemplate，从而获得SqlSession。

```java
//想要getSqlSession()需要继承SqlSessionDaoSupport
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {
    public List<User> selectAllUser() {
        // 继承SqlSessionDaoSupport后UserDaoImpl拥有getSqlSession()方法可以用于获取SqlSessionTemplate
        return getSqlSession().getMapper(UserMapper.class).selectAllUser();
    }
}
```

> 注入sqlSessionFactory属性

```xml
<!--方式二 直接注入属性sqlSessionFactory-->
<bean id="userMapper2" class="com.zhang.mapper.UserMapperImpl2">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```

> 步骤参考方式一，唯一不同在于获取SqlSessionTemplate的方式，方式一要注入sqlSessionTemplate，而方式二直接通过继承SqlSessionDaoSupport获取getSqlSession()方法，减少一个步骤。



# SSM整合

## 1、数据环境

```sql
CREATE DATABASE `ssmbuild`;

USE `ssmbuild`;

DROP TABLE IF EXISTS `books`;

CREATE TABLE `books` (
    `bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
    `bookName` VARCHAR(100) NOT NULL COMMENT '书名',
    `bookCounts` INT(11) NOT NULL COMMENT '数量',
    `detail` VARCHAR(200) NOT NULL COMMENT '描述',
    KEY `bookID` (`bookID`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT  INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`)VALUES
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```





## 2、创建项目解决基本环境

+ 添加WEB支持

+ pom.xml依赖

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://maven.apache.org/POM/4.0.0"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>com.zhang</groupId>
        <artifactId>SSMDemoIntegrate</artifactId>
        <version>1.0-SNAPSHOT</version>
    
        <dependencies>
            <!--测试-->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.13</version>
            </dependency>
            <!--servlet相关-->
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>4.0.1</version>
            </dependency>
            <dependency>
                <groupId>javax.servlet.jsp</groupId>
                <artifactId>jsp-api</artifactId>
                <version>2.2</version>
            </dependency>
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>jstl</artifactId>
                <version>1.2</version>
            </dependency>
            <!--mysql-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.19</version>
            </dependency>
            <!--数据库连接池-->
            <dependency>
                <groupId>com.mchange</groupId>
                <artifactId>c3p0</artifactId>
                <version>0.9.5.5</version>
            </dependency>
            <!--mybatis-->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>3.5.4</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>2.0.4</version>
            </dependency>
            <!--spring-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>5.2.5.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId>
                <version>5.2.5.RELEASE</version>
            </dependency>
        </dependencies>
    </project>
    ```

    

## 3、创建包文件、配置文件

包：java下

+ com.zhang.pojo
+ com.zhang.dao
+ com.zhang.service
+ com.zhang.controller



配置文件：resources下

+ dataSource.propertis // 数据库连接信息 url、用户名、密码

    ```
    jdbc.driver=com.mysql.cj.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    jdbc.username=zero
    jdbc.password=zero
    ```

    

+ mybatis-config.xml // mybatis配置文件

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
    
    </configuration>
    ```

    

+ applicationContext.xml // spring配置文件

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns="http://www.springframework.org/schema/beans"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">
        <!--导入其他spring配置文件依赖-->
        <import resource="classpath:spring/*.xml"/>
    </beans>
    ```



+ spring/spring-dao.xml  //spring配置dao层相关配置
+ spring/spring-mvc.xml //spring配置web层相关配置
+ spring/spring-service.xml //spring配置service层相关配置



## 4、Mybatis层相关配置

### 配置myabtis

> 在mybatis-config.xml中进行配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--开启自带日志功能-->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <typeAliases>
        <!--自动扫描pojo下的类并设置别名-->
        <package name="com.zhang.pojo"/>
    </typeAliases>
</configuration>
```



### 创建实体类、dao层接口

pojo:

> 创建实体类对象

```java
package com.zhang.pojo;

@Component
public class Books {
    private int bookID;
    private String bookName;
    private int bookCounts;
    private String detail;

    //getter and setter other
}

```

dao:

> 实体类对象dao层Mapper接口

```java
package com.zhang.dao;

@Repository
public interface BooksMapper {

    /**
     * @param books Books对象
     * @return 添加成功后返回1
     */
    int addBook(Books books);

    /**
     * @param id 删除书籍的具体id
     * @return 删除成功后1
     */
    int deleteBookId(@Param("bookID") int id);

    /**
     * @param books 要更新的书籍对象
     * @return 更新成功后返回 1
     */
    int updateBook(Books books);

    /**
     * @param bookid 要查询的书id
     * @return 返沪书籍对象
     */
    Books selectBookById(int bookid);

    /**
     * @return 返回所有Books信息
     */
    List<Books> selectAllBooks();

    /**
     * @param bookName 要查询的书籍名称
     * @return 返回查询到的所有书籍
     */
    List<Books> selectBookByName(String bookName);
}

```



### 编写接口对应Mapper.xml文件

mapper/booksMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.zhang.dao.BooksMapper">
    <!--添加一本书-->
    <insert id="addBook" parameterType="books">
        insert into books (bookName, bookCounts, detail)
        values (#{bookName}, #{bookCounts}, #{detail});
    </insert>

    <!--根据id删除一本书-->
    <delete id="deleteBookId" parameterType="_int">
        delete from books
        where bookID = #{bookID};
    </delete>

    <!--更新一本书-->
    <update id="updateBook" parameterType="books">
        update books
        set bookName = #{bookName}, bookCounts = #{bookCounts}, detail = #{detail}
        where bookID = #{bookID};
    </update>

    <!--根据id查询书籍-->
    <select id="selectBookById" parameterType="_int" resultType="books">
        select * from books
        where bookID = #{bookid};
    </select>

    <!--查询所有书籍-->
    <select id="selectAllBooks" resultType="books">
        select * from books;
    </select>

    <select id="selectBookByName" resultType="books">
        SELECT * FROM books
        <where>
            <if test="book != null and bookName != ''">
                bookName LIKE "%"#{bookName}"%"
            </if>
        </where>
    </select>
</mapper>
```



### 编写Service层和需求类

service接口：

```java
package com.zhang.service;

@Service
public interface BookService {

    /**
     * @param books Books对象
     * @return 添加成功后返回1
     */
    int addBook(Books books);

    /**
     * @param id 删除书籍的具体id
     * @return 删除成功后1
     */
    int deleteBookId(@Param("bookID") int id);

    /**
     * @param books 要更新的书籍对象
     * @return 更新成功后返回 1
     */
    int updateBook(Books books);

    /**
     * @param bookId 要查询的书籍名
     * @return 返沪书籍对象
     */
    Books selectBookById(int bookId);

    /**
     * @return 返回所有Books信息
     */
    List<Books> selectAllBooks();

    List<Books> selectBookByName(String bookName);
}

```



实现类：

```java
package com.zhang.service;

@Service
public class BookServiceImpl implements BookService {
    @Autowired
    //调用dao层的操作，设置一个set接口，方便Spring管理
    private BooksMapper booksMapper;

    public void setBooksMapper(BooksMapper booksMapper) {
        this.booksMapper = booksMapper;
    }

    public int addBook(Books books) {
        return booksMapper.addBook(books);
    }

    public int deleteBookId(int id) {
        return booksMapper.deleteBookId(id);
    }

    public int updateBook(Books books) {
        return booksMapper.updateBook(books);
    }

    public Books selectBookById(int bookId) {
        return booksMapper.selectBookById(bookId);
    }

    public List<Books> selectAllBooks() {
        return booksMapper.selectAllBooks();
    }

    public List<Books> selectBookByName(String bookName) {
        return booksMapper.selectBookByName(bookName);
    }
}

```



## 5、Spring整合Mybatis

> + 配置Spring整合mybatis，这里使用c3p0连接池；
> + 在spring/spring-dao.xml下编写相关配置；

spring/spring-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:comtext="http://www.springframework.org/schema/context"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置mybatis-->

    <!--关联数据库文件夹-->
    <comtext:property-placeholder location="classpath:dataSource.propertis"/>

    <!--数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- c3p0连接池的私有属性 -->
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="10000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>

    <!--配置SqlSessionFactory-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath:/mapper/*.xml"/>
    </bean>

    <!--配置扫描Dao接口包，动态实现Dao接口注入到spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--如果你需要指定 sqlSessionFactory 或 sqlSessionTemplate，那你应该要指定的是 bean 名而不是 bean 的引用-->
        <!--因此要使用 value 属性而不是通常的 ref 属性-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--给出需要扫描Dao接口包-->
        <property name="basePackage" value="com.zhang.dao"/>
    </bean>
</beans>
```



## 6、Spring整合Service层

> 这边idea遇到一问题，就算applicationContext.xml导入了其他配置文件。
>
> 但是spring-service.xml配置文件还是无法找到其他配置文件中的bean
>
> 解决方法：这边导入需要的配置文件
>
> <import resource="spring-dao.xml"/>

spring配置上下文依赖：

![image-20200413181039996](Spring%E6%95%B4%E5%90%88.assets/image-20200413181039996.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--要使用dao.xml配置的bean、idea有延迟会爆红-->
    <import resource="spring-dao.xml"/>

    <!--扫描service 装配bean的注解-->
    <context:component-scan base-package="com.zhang.service"/>

    <!--这边选着注解-->
    <!--ref="booksMapper"报错idea延迟、需要把dao.xml配置在这边引入-->
    <!--<bean id="bookServiceImpl" class="com.zhang.service.BookServiceImpl">-->
    <!--<property name="booksMapper" ref="booksMapper"/>-->
    <!--</bean>-->

    <!--配置事务管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```



## 7、Spring MVC整合

spring/spring-mvc.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.zhang.controller"/>
    <!--开启注解支持-->
    <mvc:annotation-driven/>
    <!--静态资源过滤-->
    <mvc:default-servlet-handler/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```



web.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--Spring MVC DispatcherServlet-->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!--一定要注意:我们这里加载的是总的配置文件，之前被这里坑了！-->
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--使用过滤器更改编码-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--Session过期时间-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>
```



> 配置完成。
>
> 在Controler层中完成其他代码。