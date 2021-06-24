---
layout:     post
title:      SpringBoot
subtitle:   SpringBoot基础、整合其他技术（shiro、spring-security、Junit、Jpa、Mybatis、德鲁伊、pagehelper、Thymeleaf、Swagger）
date:       2021-06-16
author:     墙纸
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
- SpringBoot
- shiro
- spring-security
- Junit
- Jpa
- Mybatis
- 德鲁伊
- pagehelper
- Thymeleaf
- Swagger
---

ja一、SpringBoot快速入门

## 1、创建SpringBoot项目

### 1、创建项目

 看看idea能怎么优雅的创建一个springboot项目

（1）创建一个项目选择spring initializr![1624446123415](img\springboot\1624446123415.png)

（2）工程名spring-boot-test

![1624446265945](..\img\springboot\1624446265945.png)

（3）勾选需要的组件

![1624446330491](..\img\springboot\1624446330491.png)

![1624446396886](..\img\springboot\1624446396886.png)

> **写一个controller**

```java
@Conreoller
public class UserController{
    @RequestMapping("/")
    @ResponseBody
    public String test(){
        return "Hello springboot";
    }
}
```

> **运行引导类的main方法**

![1624450883807](..\img\springboot\1624450883807.png)

结果为OK！

![1624450958467](..\img\springboot\1624450958467.png)

其他的都一样是不是很快是不是很爽！

### 2、分析一下

1、springboot工程本质就是个maven工程，他有自己的依赖和自己的插件

![1624451058231](..\img\springboot\1624451058231.png)

重点：都一样！

思考你能不能从一个空工程建立一个springboot工程

（1）建一个maven工程

（2）引入依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-devtools</artifactId>
             <optional>true</optional>
             <scope>true</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                	<fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

（3）写一个main方法即可，main方法hi固定的

```java
@SpringBootApplication
public class SpringBootSolrApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootSolrApplication.class, args);
    }
}
```

## 2、SpringBoot工程热部署

（1）我们在开发中反复修改类、页面等资源，每次修改后都是需要重新启动才生效，这样每次启动都很麻烦，浪费了大量的时间，我们可以在修改代码后不重启就能生效，在 pom.xml
中添加如下配置就可以实现这样的功能，我们称之为热部署。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
	<scope>true</scope>
</dependency>
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

（2）IDEA中配置当我们修改了类文件后，idea不会自动编译，得修改idea设置。

File-Settings-Compiler-Build Project automatically

![1624451804299](..\img\springboot\1624451804299.png)

ctrl + shift + alt + / ,选择Registry,勾上 Compiler autoMake allow when app running

![1624451849728](..\img\springboot\1624451849728.png)

![1624452265642](..\img\springboot\1624452265642.png)

> 思考和我们以前的项目有什么不同，直观的两点

1. 以main方法启动，貌似不需要tomcat

2. 没有什么配置文件，xml都没有了

3. 没有了配置文件我们怎么组装呢？

# 二、SpringBoot基础分析

## 1、丢了的tomcat

### 内嵌tomcat原理

> pom

```xml
<dependencies>
	<dependency>
		<groupId>org.apache.tomcat.embed</groupId>
		<artifactId>tomcat-embed-core</artifactId>
		<version>9.0.43</version>
	</dependency>
	<dependency>
		<groupId>org.apache.tomcat.embed</groupId>
		<artifactId>tomcat-embed-jasper</artifactId>
		<version>9.0.43</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>5.2.9.RELEASE</version>
	</dependency>
</dependencies>
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.8.1</version>
			<configuration>
				<target>1.8</target>
				<source>1.8</source>
				<encoding>utf-8</encoding>
			</configuration>
		</plugin>
	</plugins>
</build>
```

> 一个main方法

```java
public class SpringBootApplication{
	public static void run() {
		Tomcat tomcat = new Tomcat();
		Context context = tomcat.addWebapp("/app", "D:/img/");
		AnnotationConfigWebApplicationContext  applicationContext= new AnnotationConfigWebApplicationContext();
		applicationContext.register(AppConfig.class);
		DispatcherServlet dispatcherServlet = new DispatcherServlet(applicationContext);
		Wrapper wrapper = tomcat.addServlet("/app", "app", dispatcherServlet);
		wrapper.addMapping("/*");
        try{
			Connector connector = new Connector(); connector.setPort(8080);
             tomcat.getService().addConnector(connector);
             tomcat.start();
			tomcat.getServer().await();
        } catch (LifecycleException e){
            e.prinrStackTrace();
        }
    }
}
```

```java
@ComponentScan("cn.itnanls")
public class AppConfig {
    
}
```

## 2、配置文件

### 1、真的没有配置文件了吗？其实也有

![1624452994705](..\img\springboot\1624452994705.png)

例如：怎么配一个端口呢？

![1624453035061](..\img\springboot\1624453035061.png)

### 2、猜一猜，估计是根据约定以编码的方式写了代码里边

同时根据上边的配置我们大致明白了，对于配置而言一般是如下情况

```java
if(配置里有){
	return 配置;
}else{
	retuen 默认;
}
```

### 3、思考一下spring里边的bean是怎么注入的？

通常：spring注入bean有几种方式

可以直接注入Bean（User）  也可以注入factorybean（UserFactorybean）

（1）配置文件    不说了  <bean> 标签 。

（2）扫包

@Component

@Configuration     @Bean

@Service  @Controller  @Repository

@Import

```java
// 直接引入一个类
@Import(User.class) 3	// 引入一个配置类
@Import(UserConfiguration.class)
//   引入一个实现了ImportSelector接口的类，返回全类型数组，数组的类都可以引入
@Import(UserSelector.class)
public class UserSelector implements ImportSelector {
	@Override
	public String[] selectImports(AnnotationMetadata importingClassMetadata){
		return new String[]{"cn.itnanls.User"};
	}
}
```

但是前提是你扫描的包下有这些注解！

那你引入的第三方的组件，你的工程并没有扫描到，怎么用啊？

1、自己写配置类，当然可以

2、但是事实上很多组件连配置类都不用写，为什么？ 重要特性----自动装配（starter）后边讲

> 配置文件真的可以没有吗？

1、你想改一个端口，真没要修改代码吗？

2、框架知道你的数据库连接吗？

3、修改连接真的要修改代码，重新编译吗？

> 配置文件真的都有用吗？

1. spring的配置文件太重，我们不想要的是哪些不会经常变动的而已。

2. 但事实上，我们使用的大量组件都需要和spring结合，只要结合就需要注入自己的bean。

3. 到了后来你会发现，配置文件里有了fastjson的配置，druid的配置，mybatis的配置，springmvc 的配置....

4. 然而，这些配置往往配置一次以后很少会在此改变。

5. 和装配相关的配置往往没有那么重要！

> 那合理的做法是什么呢？

1、我们预留出一个单独的配置文件，去配置那些必须的或是需要修改的配置。

2、其他的配置我们使用java方式去配置，如果组件能够自己自动配置就更好了，springboot也是这么做    的。

>  约定大于配置 mysql启动的端口你知道是多少吗？ tomcat启动的端口你知道吗？

为什么springboot启动默认就在8080端口，你配置了吗？ 因为是默认，因为约定俗成。

当所有组件都有默认的配置，都有你知我知的约定，那他具体怎么和spring整合，我们就明白了。 举个例子，比如整合thymleaf，默认他们试图解析器前缀是“classpath:/templates/”,后缀是“.html” 那么在整合时，我们即使什么也不配，我也知道。

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

加一个依赖就搞定了。

但是有一天我想修改他的配置怎么办，有办法。

springboot是提供有了一个配置文件的。

> SpringBoot是 基于约定 的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的 话，就可以使用application.properties或者application.yml（application.yaml）进行配置。

SpringBoot默认会从Resources目录下加载application.properties或 application.yml（application.yaml）文件

其中，application.properties文件是键值对类型的文件，之前一直在使用，所以此处不在对properties 文件的格式进行阐述。除了properties文件外，SpringBoot还可以使用yml文件进行配置，下面对yml文 件进行讲解。

### 1）配置文件语法

> application.yml配置文件 
>
> yml配置文件简介

YML文件格式是YAML (YAML Aint Markup Language)编写的文件格式，YAML是一种直观的能够被电脑 识别的的数据数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不 同的编程语言程序导入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP等。YML文件是以数据为核心 的，比传统的xml方式更加简洁。

YML文件的扩展名可以使用.yml或者.yaml。

> yml配置文件的语法 配置普通数据

- 语法： key: value
- 示例代码：
- name: haohao
- 注意：value之前有一个空格

> 配置对象数据

- 语法：

​	key:

​		key1: value1 

​		key2: value2

​	或者：

​	key: {key1: value1,key2: value2}

- 示例代码：

```
person:
	name: haohao
	age: 31
	addr: beijing
#或者
person: {name: haohao,age: 31,addr: beijing}
```

注意：key1前面的空格个数不限定，在yml语法中，相同缩进代表同一个级别

> 配置Map数据 

同上面的对象写法

> 配置数组（List、Set）数据

- 语法：

  ​	key:

  ​		-  value1

  ​		-  value2

  或者：

  ​	key: [value1,value2]

- 示例代码：

```yml
city:
	- beijing
	- tianjin
	- shanghai
	- chongqing
#或者
city:  [beijing,tianjin,shanghai,chongqing]

#集合中的元素是对象形式 student:
	- name: zhangsan 
	  age: 18
	  score: 100
	- name: lisi 
	  age: 28
	  score: 88
	- name: wangwu 
	  age: 38
	  score: 90
```

`注意`：value1与之间的  - 之间存在一个空格

### 2）常用配置

 

```yml
# EMBEDDED SERVER CONFIGURATION (ServerProperties) 
server.port=8080 # Server HTTP port.
server.servlet.context-path= demo # Context path of the application.
server.servlet.path=/ # Path of the main dispatcher servlet.

# SPRING MVC (WebMvcProperties)
spring.mvc.view.prefix= # Spring MVC view prefix. 
spring.mvc.view.suffix= # Spring MVC view suffix.

# DATASOURCE (DataSourceAutoConfiguration & DataSourceProperties) 
spring.datasource.driver-class-name= # Fully qualified name of the JDBC driver. Auto-detected based on the URL by default. 
spring.datasource.password= # Login password of the database. 
spring.datasource.url= # JDBC URL of the database.
spring.datasource.username= # Login username of the database.
```

我们可以通过配置application.poperties  或者 application.yml 来修改SpringBoot的默认配置

例如：

application.properties文件

```
server.port=8888
server.servlet.context-path=demo
```

application.yml文件

```
server:
	port: 8888
	servlet:
		context-path: /demo
```

### 3）读取配置

> 使用注解@Value 

我们可以通过@Value注解将配置文件中的值映射到一个Spring

管理的Bean的字段上 例如：

application.properties配置如下：

```properties
person:
	name: zhangsan
	age: 18
```

或者，application.yml配置如下：

```yml
person:
	name: zhangsan
	age: 18
```

实体Bean代码如下：

```java
@Controller
public class QuickStartController {

	@Value("${person.name}") 
	private String name;
	@Value("${person.age}") 
	private Integer age;

	@RequestMapping("/quick")
	@ResponseBody
	public String quick(){
		return "springboot 访问成功! name="+name+",age="+age;
	}
}

```

浏览器访问地址：http://localhost:8080/quick  结果如下：

> 使用注解@ConfigurationProperties映射

通过注解@ConfigurationProperties(prefix="配置文件中的key的前缀")可以将配置文件中的配置自动与 实体进行映射

application.properties配置如下：

```properties
person:
	name: zhangsan
	age: 18
```

或者，application.yml配置如下：

```yml
person:
	name: zhangsan
	age: 18
```

实体Bean代码如下：

```java
@Controller
@ConfigurationProperties(prefix = "person") 
public class QuickStartController {
	private String name; private Integer age;

	@RequestMapping("/quick")
	@ResponseBody
	public String quick(){
		return "springboot 访问成功! name="+name+",age="+age;
	}

	public void setName(String name) { 
		this.name = name;
	}

	public void setAge(Integer age) { 
		this.age = age;
	}
}
```

浏览器访问地址：http://localhost:8080/quick  结果如下：

注意：使用@ConfigurationProperties方式可以进行配置文件与实体字段的自动映射，但需要字段必须提供set方法才可以，而使用@Value注解修饰的字段不需要提供set方法

## 3、依赖分析                                                                    

### (1)   分析spring-boot-starter-parent

按住Ctrl点击pom.xml中的spring-boot-starter-parent，跳转到了spring-boot-starter-parent的

pom.xml，xml配置如下（只摘抄了部分重点配置）：

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-dependencies</artifactId>
	<version>2.4.3</version>
</parent>
```

发现parent中还有parent（spring-boot-dependencies）

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-dependencies</artifactId>
	<version>2.4.3</version>
</parent>
<artifactId>spring-boot-starter-parent</artifactId>
<packaging>pom</packaging>
<name>spring-boot-starter-parent</name>
<description>Parent pom providing dependency and plugin management for applications built with Maven</description>
<properties>
	<java.version>1.8</java.version>
	<resource.delimiter>@</resource.delimiter>
	<maven.compiler.source>${java.version}</maven.compiler.source>
	<maven.compiler.target>${java.version}</maven.compiler.target>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<project.reporting.outputEncoding>UTF- 8</project.reporting.outputEncoding>
</properties>
<url>https://spring.io/projects/spring-boot</url>
<licenses>
	<license>
		<name>Apache License, Version 2.0</name>
		<url>[https://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)</url>
	</license>
</licenses>
<developers>
	<developer>
		<name>Pivotal</name>
			<email>[info@pivotal.io](mailto:info@pivotal.io)</email>
			<organization>Pivotal Software, Inc.</organization>
			<organizationUrl>[https://www.spring.io](http://www.spring.io/)</organizationUrl>
	</developer>
</developers>
<scm>
	<url>https://github.com/spring-projects/spring-boot</url>
</scm>
<build>
	<resources>
		<resource>
			<directory>${basedir}/src/main/resources</directory>
			<filtering>true</filtering>
		<includes>
			<include>**/application*.yml</include>
			<include>**/application*.yaml</include>
			<include>**/application*.properties</include>
		</includes>
	</resource>
	<resource>
		<directory>${basedir}/src/main/resources</directory>
		<excludes>
			<exclude>**/application*.yml</exclude>
			<exclude>**/application*.yaml</exclude>
			<exclude>**/application*.properties</exclude>
		</excludes>
	</resource>
</resources>
```

按住Ctrl点击pom.xml中的spring-boot-starter-dependencies，跳转到了spring-boot-starter- dependencies的pom.xml，xml配置如下（只摘抄了部分重点配置）：

```
<properties>
	<activemq.version>5.15.12</activemq.version>
	<commons-codec.version>1.14</commons-codec.version>
	<commons-dbcp2.version>2.7.0</commons-dbcp2.version>
	<commons-lang3.version>3.10</commons-lang3.version>
	<commons-pool.version>1.6</commons-pool.version>
	<commons-pool2.version>2.8.0</commons-pool2.version>
	<ehcache.version>2.10.6</ehcache.version>
	<ehcache3.version>3.8.1</ehcache3.version>
	<elasticsearch.version>7.6.2</elasticsearch.version>
	<freemarker.version>2.3.30</freemarker.version>
	<hibernate.version>5.4.15.Final</hibernate.version>
	<hibernate-validator.version>6.1.5.Final</hibernate-validator.version>
	<hikaricp.version>3.4.5</hikaricp.version>
	<jedis.version>3.3.0</jedis.version>
	<junit.version>4.13</junit.version>
	<junit-jupiter.version>5.6.2</junit-jupiter.version>
	<kafka.version>2.5.0</kafka.version>
	<maven-compiler-plugin.version>3.8.1</maven-compiler-plugin.version>
	<maven-dependency-plugin.version>3.1.2</maven-dependency-plugin.version>
	<mysql.version>8.0.20</mysql.version>
	<neo4j-ogm.version>3.2.11</neo4j-ogm.version>
	<netty.version>4.1.49.Final</netty.version>
	<netty-tcnative.version>2.0.30.Final</netty-tcnative.version>
	<quartz.version>2.3.2</quartz.version>
	<servlet-api.version>4.0.1</servlet-api.version>
	<slf4j.version>1.7.30</slf4j.version>
	<snakeyaml.version>1.26</snakeyaml.version>
	<solr.version>8.5.1</solr.version> 30 ...
</properties>
<dependencyManagement>
...
</dependencyManagement>
```

从上面的spring-boot-starter-dependencies的pom.xml中我们可以发现，一部分坐标的版本、依赖管 理、插件管理已经定义好，所以我们的SpringBoot工程继承spring-boot-starter-parent后已经具备版本 锁定等配置了。所以起步依赖的作用就是进行依赖的传递。

springboot帮助我们整合了市场上绝大多数多的常用组件，并且使用了组合适版本，尽量规避了因版本 不兼容而导致的问题。

### (2)  分析spring-boot-starter-web

按住Ctrl点击pom.xml中的spring-boot-starter-web，跳转到了spring-boot-starter-web的pom.xml，xml配置如下（只摘抄了部分重点配置）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd>" 
         xmlns="http://maven.apache.org/POM/4.0.0"   
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <!-- This module   was also published with a richer   model, Gradle metadata,   -->   
    <!-- which should   be used instead. Do not delete   the following line which   -->   
    <!-- is to indicate to Gradle or any Gradle   module metadata file consumer   -->   
    <!-- that they   should prefer consuming it instead. -->   
    <!-- do_not_remove: published-with-gradle-metadata -->   
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.3.0.RELEASE</version>
    <name>spring-boot-starter-web</name>
    <description>Starter for building web,including RESTful,applications using Spring MVC.Uses Tomcat as the default embedded container</description>
    <url>https://spring.io/projects/spring-boot</url>
    <organization>
        <name>Pivotal   Software, Inc.</name>
        <url>https://spring.io</url>
    </organization>
    <licenses>
        <license>
            <name>Apache   License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0></url>
        </license>
    </licenses>
    <developers>
        <developer>
            <name>Pivotal</name>
            <email>info@pivotal.io</email>
            <organization>Pivotal Software, Inc.</organization>
            <organizationUrl>https://www.spring.io</organizationUrl>
        </developer>
    </developers>
    <scm>
        <connection>scm:git:git://github.com/spring-projects/spring- boot.git</connection>
        <developerConnection>scm:git:ssh://git@github.com/spring- projects/spring-boot.git</developerConnection>
        <url>https://github.com/spring-projects/spring-boot</url>
    </scm>
    <issueManagement>  
    	<system>GitHub</system>
        <url>https://github.com/spring-projects/spring-boot/issues</url>
    </issueManagement>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>2.3.0.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
		<version>2.3.0.RELEASE</version>
		<scope>compile</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-json</artifactId>
		<version>2.3.0.RELEASE</version>
		<scope>compile</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-tomcat</artifactId>
		<version>2.3.0.RELEASE</version>
		<scope>compile</scope>
	</dependency>
	<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<scope>compile</scope>
	</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<scope>compile</scope>
		</dependency>
	</dependencies>
</project>
```

从上面的spring-boot-starter-web的pom.xml中我们可以发现，spring-boot-starter-web就是将web开 发要使用的spring-web、spring-webmvc等坐标进行了“打包”，这样我们的工程只要引入spring-boot- starter-web起步依赖的坐标就可以进行web开发了，同样体现了依赖传递的作用。每一项都可以点进去看看！

# 三 、web配置                                             

## 1 、自定义欢迎页                                                              

默认回去static中中找index.html

```
classpath:/static/index.html
classpath:/public/index.html
```

## 2 、配置错误页                                                                 

在static目录下新建error目录

放入对应的错误页面就行了如404.html     500.html

为什么放在这里就行，应为约定

## 3 、自定义favicon                                                                    

浏览器左上角的图标可以放在静态资源下static中，

随便找个网站http://www.bitbug.net/生成了放进去就行了

> 体会一点感受： 

约定的重要性，你我都服从约定，很多代码就不用写了。 你知道垃圾要扔到垃圾箱，那么处理垃圾就变得简单了。 spring做的就是制定垃圾要扔到垃圾箱的规则。

## 4 、定制banner                                                                        

<http://patorjk.com/software/taag/>

<https://www.degraeve.com/img2txt.php>

创建Banner文件

src/main/resource/banner.txt

```text
${AnsiColor.BRIGHT_RED}
                                  
                              _ooOoo_                               
                             o8888888o                              
                             88" . "88                              
                             (| -_- |)                              
                             O\  =  /O                              
                          ____/`---'\____                           
                        .'  \\|     |//  `.                         
                       /  \\|||  :  |||//  \                        
                      /  _||||| -:- |||||_  \                       
                      |   | \\\  -  /'| |   |                       
                      | \_|  `\`---'//  |_/ |                       
                      \  .-\__ `-. -'__/-.  /                       
                    ___`. .'  /--.--\  `. .'___                     
                 ."" '<  `.___\_<|>_/___.' _> \"".                  
                | | :  `- \`. ;`. _/; .'/ /  .' ; |    Buddha       
                \  \ `-.   \_\_`. _.'_/_/  -' _.' /                 
  ================-.`___`-.__\ \___  /__.-'_.'_.-'================  
                              `=--=-'                    hjw        
Spring Boot Version: ${spring-boot.version}${spring-boot.formatted-version}
```

从上面的内容中可以看到，还使用了一些属性设置：

​	${AnsiColor.BRIGHT_RED}：设置控制台中输出内容的颜色，可以自定义，具体参考

​	org.springframework.boot.ansi.AnsiColor

​	${spring-boot.version}：Spring  Boot的版本号

​	{spring-boot.formatted-version}：格式化后的{spring-boot.version}版本信息

不妨自己从run方法进去看看源码：

```java
public static void main(String[] args) {
	SpringApplication.run(SpringbootStudyApplication.class,  args);
}
```

原来在这里定义了。

```java
SpringApplicationBannerPrinter类中：
static final String BANNER_LOCATION_PROPERTY = "spring.banner.location";
static final String BANNER_IMAGE_LOCATION_PROPERTY =
"spring.banner.image.location";
static final String DEFAULT_BANNER_LOCATION = "banner.txt";
```

## 5 、其他的服务器配置如jetty                                      

在某个博客看到改Jetty的好处，也真是我现在开发的项目后面要用长连接

好处： 1、Jetty适合长连接应用，就是聊天类的长

连接：  2、Jetty更轻量级。这是相对Tomcat而言的。

3、jetty更灵活，体现在其可插拔性和可扩展性，更易于开发者对Jetty本身进行二次开发，定制一个适 合自身需求的Web  Server。

4、使用Jetty，需要在spring-boot-starter-web排除spring-boot-starter-tomcat，因为SpringBoot默认 使用tomcat

对于配置内置服务器的springBoot，都必定会配置

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

以上配置springBoot的启动web服务器，但默认是Tomcat

所以呢，要配置为jetty要去掉默认tomcat配置

 

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

并且加上jetty启动

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

## 6 、资源定义                                                                    

默认，就用默认

```
classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
```

> 如果需要修改

第一种：在配置文件中进行配置

```yml
#静态资源访问路径
spring.mvc.static-path-pattern=/img/**
#静态资源映射路径，注意会覆盖
spring.mvc.static-path-pattern=/**
spring.web.resources.static-locations[0]=classpath:/img/
spring.web.resources.static-locations[1]=classpath:/static/
spring.web.resources.static-locations[2]=classpath:/public/
```

点击配置文件发现，源码就是读取static-locations来设置资源目录的：

```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { 
	"classpath:/META-INF/resources/",
	"classpath:/resources/", 
	"classpath:/static/", 
	"classpath:/public/" 
};
```

## 7 、web配置（重点）                                                      

WebMvcConfigurer配置类其实是 Spring 内部的一种配置方式，采用 JavaBean 的形式来代替传统的 配置文件形式进行针对框架个性化定制，可以自定义一些Handler，Interceptor，ViewResolver，MessageConverter。基于java-based方式的spring       mvc配置，需要创建一个**配置**类并实现

**WebMvcConfigurer** 接口；

在Spring Boot 1.5版本都是靠重写**WebMvcConfigurerAdapter**的方法来添加自定义拦截器，消息转 换器等。SpringBoot 2.0 后，该类被标记为@Deprecated（弃用）。官方推荐直接实现 WebMvcConfigurer或者直接继承WebMvcConfigurationSupport，方式一实现WebMvcConfigurer接 口（推荐），方式二继承WebMvcConfigurationSupport类。

**常用的方法**：

```
/**
* @author zn
* @date 2021/3/3
*/
@Configuration
public class ApplicationConfiguration implements WebMvcConfigurer {
	/**
 	 * 定义拦截器 
 	 * @param registry   
 	 */   
 	@Override   
 	public void addInterceptors(InterceptorRegistry registry) { 
 		registry.addInterceptor(new   LoginInterception()).addPathPatterns("/**");   
 	}           
 	/**   
 	 * 定义资源，不覆盖默认   
 	 * @param registry   
 	 */   
 	 @Override   
 	 public void addResourceHandlers(ResourceHandlerRegistry   registry) {
 	 	registry.addResourceHandler("/img/**").addResourceLocations("classpath:/im g/"); 
 	 	registry.addResourceHandler("/file/**").addResourceLocations("file:D:/img/ ");   
 	 } 
 	 /**   
 	  * 后端解决跨域的配置   
 	  * @param registry   
 	  */   
 	 @Override   
 	 public void addCorsMappings(CorsRegistry   registry) { 
 	 	registry.addMapping("/app/**").allowedOrigins("*");   
 	 }           
 	 /**   
 	  * 视图跳转控制器   
 	  * 相当于写了controller   
 	  * @param registry   
 	  */  
     @Override   
     public void addViewControllers(ViewControllerRegistry registry) {
     	registry.addViewController("/user").setViewName("user");   
     }           
     /** 
     	<dependency>   
     		<groupId>com.alibaba</groupId>   
     		<artifactId>fastjson</artifactId>   
     		<version>1.2.46</version>   
     	</dependency>   
       * 配置消息转化器   
       * @param converters   
       */   
      @Override   
      public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
      	FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
      	FastJsonConfig config = new FastJsonConfig();
      	 config.setSerializerFeatures(   
      	 	// 保留map空的字段
			SerializerFeature.WriteMapNullValue,
			// 将String类型的null转成""
			SerializerFeature.WriteNullStringAsEmpty,
			// 将Number类型的null转成0
			SerializerFeature.WriteNullNumberAsZero,
			// 将List类型的null转成[]
			SerializerFeature.WriteNullListAsEmpty,
			// 将Boolean类型的null转成false
			SerializerFeature.WriteNullBooleanAsFalse,
			// 避免循环引用
			SerializerFeature.DisableCircularReferenceDetect
		);
		converter.setFastJsonConfig(config);
		converter.setDefaultCharset(Charset.forName("UTF-8"));
		List<MediaType> mediaTypeList = new ArrayList<>();
		//   解决中文乱码问题，相当于在Controller上的@RequestMapping中加了个属性produces = "application/json"
		mediaTypeList.add(MediaType.APPLICATION_JSON);
		converter.setSupportedMediaTypes(mediaTypeList); 81
		//将fastjson添加到视图消息转换器列表内
		converters.add(converter); 84   
	}
	/**
	 * 配置视图解析器
	 * @param registry
 	 */
 	 @Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
	}
	/**

	 * 参数解析器
	 * @param resolvers
	 */
	@Override
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {

	}

	/**
	 * 配置返回值处理器
	 * @param handlers
	 */
	 @Override
	public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {

	}

}
```

## 8、打包

> jar包

 因为是maven项目![1624457688276](..\img\springboot\1624457688276.png)

标志打包为war

```xml
<packaging>war</packaging>
```

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
		</exclusion>
		<exclusion>
			<groupId>org.springframework.boot</groupId> 
			<artifactId>spring-boot-starter-tomcat</artifactId>   
		</exclusion>   
	</exclusions>   
</dependency>   
<dependency>   
	<groupId>org.springframework.boot</groupId>   
	<artifactId>spring-boot-starter-tomcat</artifactId>   
<!--打包的时候可以不用包进去，别的设施会提供。事实上该依赖理论上可以参与编译，测试，运 行等周期。   相当于compile，但是打包阶段做了exclude操作-->   
	<scope>provided</scope>   
</dependency>
```

 

 

```java
/**
 * @author AiLi
 * @date 2021/6/7 
 */   
@Component   
public class ServletInitializer extends SpringBootServletInitializer {
	@Override   
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		return  builder.sources(MySpringBootApplication.class);   
	}   
}   
```

 SpringBootServletInitializer的执行过程，简单来说就是通过SpringApplicationBuilder构建并封装 SpringApplication对象，并最终调用SpringApplication的run方法的过程。

spring boot就是为了简化开发的，也就是用注解的方式取代了传统的xml配置。 SpringBootServletInitializer就是原有的web.xml文件的替代。 使用了嵌入式Servlet,默认是不支持jsp。

SpringBootServletInitializer 可以使用外部的Servlet容器，使用步骤：

- 1.必须创建war项目，需要创建好web项目的目录。
- 2.嵌入式Tomcat依赖scope指定provided。
- 3.编写SpringBootServletInitializer类子类,并重写configure方法。

```
public class ServletInitializer extends SpringBootServletInitializer {
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) { 
		return  application.sources(SpringBoot04WebJspApplication.class);
	}
}
```

4.启动服务器。

jar包和war包启动区别

- jar包:执行SpringBootApplication的run方法,启动IOC容器,然后创建嵌入式Servlet容器

**war**包: 先是启动**Servlet**服务器,服务器启动**Springboot**应用(**springBootServletInitizer**),然后启动IOC 容器

**SpringBootServletInitializer**实例执行**onStartup**方法的时候会通过**createRootApplicationContext**方法 来执行run方法，接下来的过程就同以jar包形式启动的应用的**run**过程一样了，在内部会创建**IOC**容器并 返回，只是以**war**包形式的应用在创建**IOC**容器过程中，不再创建**Servlet**容器了。

tomcat 

1、ServletContainerInitializer

2、META-INF/services/javax.servlet.ServletContainerInitializer  找  spi 调用 onstart

![1624458113611](..\img\springboot\1624458113611.png)

3、servletcontext  配置servlet  filter listener

## 9、启动系统任务                                                            

有两个接口**CommandLineRunner**和**ApplicationRunner**，实现了这两项接口的类会在系统启动后自动 调用执行run方法

```java
/**
 * @author AiLi
 * @date 2020/6/7
 */
@Component
@Order(1)
public class CommandRunnerOne implements CommandLineRunner {
	//args是获取的main方法传入的参数
	@Override
	public void run(String... args) throws Exception {
		//这里可以做一些事情比如redis预热，系统检查等工作 
		System.out.println("commandOne---->" + Arrays.toString(args));
	}
}

@Component
@Order(2)
public class CommandRunnerTwo implements CommandLineRunner {
	//args是获取的main方法传入的参数
	@Override
	public void run(String... args) throws Exception {
		//这里可以做一些事情比如redis预热，系统检查等工作 
		System.out.println("commandTwo---->" + Arrays.toString(args));
	}
}
```

>  测试

另一个和这个接口用法一样，只是参数不同，有兴趣的同学可以研究一下。![1624458323328](img\springboot\1624458323328.png)

## 10、配置文件的读取顺序                                                

如果在不同的目录中存在多个配置文件，它的读取顺序是：

1、config/application.properties（项目根目录中config目录下） 

2、config/application.yml 

3、application.properties（项目根目录下）

4、application.yml 

5、resources/config/application.properties（项目resources目录中config目录下） 

6、resources/config/application.yml 

7、resources/application.properties（项目的resources目录下） 

8、resources/application.yml

> 有啥好处，打包后我们可以再jar包之外放置配置文件，随时修改

```xml
 /**   
  *  {@link EnvironmentPostProcessor}   that configures the   context environment by loading   
  *  properties from   well known file   locations. By default properties will be loaded from   
  *  'application.properties' and/or   'application.yml' files in the following locations:   
  *    <ul>   
  *    <li>file:./config/</li>   
  *    <li>file:./config/{@literal *}/</li>   
  *    <li>file:./</li>   
  *    <li>classpath:config/</li>   
  *    <li>classpath:</li>   
  * </ul>   
  * The list is ordered by precedence (properties defined in locations higher the list
  * override those defined in lower locations).
  */
@Deprecated
public class ConfigFileApplicationListener implements EnvironmentPostProcessor, SmartApplicationListener, Ordered {

```

## 11、SpringBoot Profile多环境配置

> 多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml    例如：

application-dev.yml 

application-prod.yml 

application-test.yml

默认使用application.properties的配置; yml支持多文

> 档块方式

```yml
spring:
	profiles:
		active: dev #指定使用哪个环境 
---
server:
	port: 8082
spring:
	profiles: dev 
---
server:
	port: 8083
spring:
	profiles: test 
---
server:
	port: 8084
spring:
	profiles: prod
```

> 激活指定Profile

​	在配置文件中指定 spring.profiles.active=dev

​	命令行：java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev;

​	可以直接在测试的时候，配置传入命令行参数

​	 虚拟机参数：-Dspring.profiles.active=dev

> 优先级 

命令行参数>JVM参数>配置文件

## 1 2、原生的servlet内容                                                   

### （1）第一种方式

> 由于springboot基于servlet3.0+,内嵌tomcat容器 因此无法像之前一样通过web.xml中配置Filter

```java
/**
 * @author itnanls
 * @date 2021/3/9
 */
@WebServlet(urlPatterns = "/user")
public class UserServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		System.out.println("创建了servlet");
	}
}
/**
 * @author itnanls
 */
@SpringBootApplication
@ServletComponentScan
public class SpringbootStudyApplication { public static void main(String[] args) {
	SpringApplication.run(SpringbootStudyApplication.class,  args);
}
                                         
/**                                         /**
 * @author itnanls
 * @date 2021/3/9
 */
@WebFilter("/*")
public class UserFilter implements Filter {
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, 			ServletException {
		System.out.println("来了过滤器！");
		doFilter(request,response,chain);
	}
}
/**
 * @author itnanls
 */
@SpringBootApplication
@ServletComponentScan
public class SpringbootStudyApplication { public static void main(String[] args) {
	SpringApplication.run(SpringbootStudyApplication.class,  args);
}
/**
* @author zn
* @date 2021/3/9
*/
@WebListener
public class UserListener implements HttpSessionListener {
    @Override
	public void sessionCreated(HttpSessionEvent se) { 
        System.out.println("创建了一个session");
	}
}

@SpringBootApplication
@ServletComponentScan
public class SpringbootStudyApplication { public static void main(String[] args) {
	SpringApplication.run(SpringbootStudyApplication.class,  args);
}

```

### （2）第二种方式

```java
 /**
  * @author itnanls 3 * @date 2021/3/9
  */
@Configuration
public class ServletConfiguration { 7
	@Bean
	public ServletRegistrationBean<UserServlet> servletRegistrationBean(){
		return new ServletRegistrationBean<>(new UserServlet(),"/user"); 11 
	}

	@Bean
	public FilterRegistrationBean<UserFilter> filterRegistrationBean(){
		FilterRegistrationBean<UserFilter> register = new FilterRegistrationBean<>();
		register.setFilter(new UserFilter());
		register.setUrlPatterns(Arrays.asList("/admin/*","/user/*"));
		return register; 19 
	}

	@Bean
	public ServletListenerRegistrationBean<UserListener> servletListenerRegistrationBean(){
		return new ServletListenerRegistrationBean<>(new UserListener()); 24 }
	}
```

## 13、观察者设计模式                                                       

定义一个自定义事件，继承ApplicationEvent类

```java
/**
 * 定义事件
 *
 */
public class MyApplicationEvent extends ApplicationEvent {
	private static final long serialVersionUID = 1L;
	public MyApplicationEvent(Object source) {
		super(source);	
	}
}

/**
 * 定义事件的监听
 *
 */
@Component
public class MyApplicationListener implements ApplicationListener<MyApplicationEvent>  {
	public void onApplicationEvent(MyApplicationEvent event) {
		System.out.println("接收到事件："+event.getClass());
	}
}
```

主类测试：

```java
@SpringBootApplication 
public class Application {   
	public static void main(String[] args) { 
		SpringApplication application = new SpringApplication(Application.class);   
		ConfigurableApplicationContext context =application.run(args);   
		//发布事件   
		context.publishEvent(new MyApplicationEvent(new Object())); 
		context.close();   
	}   
}   
```



## 14、定时任务

> 开启定时任务

```java
@EnableScheduling
public class MySpringBootApplication
```

> 写代码

```java
/**
 * @author AiLi
 * @date 2021/6/7
 */
@Component
public class MySchedule {
	@Scheduled(fixedDelay = 3000) 
    public void fixedDelay(){
		System.out.println("fixedDelay:"+new  Date());
	}

	@Scheduled(fixedRate = 3000) 
    public void fixedRate(){
		System.out.println("fixedRate:"+new  Date());
	}


	@Scheduled(initialDelay = 1000,fixedDelay = 2000) 
    public void initialDelay(){
		System.out.println("initialDelay:"+new  Date());
	}

	@Scheduled(cron = "0 * * * * ?") 
    public void cron(){
	System.out.println("cron:"+new  Date());
	}
}
```

 结果

```
fixedRate:Sun Jun 07 11:16:17 CST 2020
fixedDelay:Sun Jun 07 11:16:17 CST 2020
2020-06-07 11:16:17.337	INFO 11928 --- [			main]
com.example.MySpringBootApplication	:	Started	MySpringBootApplication
in 6.028 seconds (JVM running for 8.211)
initialDelay:Sun Jun 07 11:16:18 CST 2020			
fixedRate:Sun Jun 07 11:16:20 CST 2020			
fixedDelay:Sun Jun 07 11:16:20 CST 2020			
initialDelay:Sun Jun 07 11:16:20 CST 2020			
initialDelay:Sun Jun 07 11:16:22 CST 2020			
fixedRate:Sun Jun 07 11:16:23 CST 2020			
fixedDelay:Sun Jun 07 11:16:23 CST 2020			
initialDelay:Sun Jun 07 11:16:24 CST 2020			
fixedRate:Sun Jun 07 11:16:26 CST 2020			
fixedDelay:Sun Jun 07 11:16:26 CST 2020	
initialDelay:Sun Jun 07 11:16:26 CST 2020
initialDelay:Sun Jun 07 11:16:28 CST 2020
fixedRate:Sun Jun 07 11:16:29 CST 2020
fixedDelay:Sun Jun 07 11:16:29 CST 2020
initialDelay:Sun Jun 07 11:16:30 CST 2020
```

> 区别

1、fixedDelay控制方法执行的间隔时间，是以上一次方法执行完开始算起，如上一次方法执行阻塞住 了，那么直到上一次执行完，并间隔给定的时间后，执行下一次。

2、fixedRate是按照一定的速率执行，是从上一次方法执行开始的时间算起，如果上一次方法阻塞住 了，下一次也是不会执行，但是在阻塞这段时间内累计应该执行的次数，当不再阻塞时，一下子把这些 全部执行掉，而后再按照固定速率继续执行。

3、cron表达式可以定制化执行任务，但是执行的方式是与fixedDelay相近的，也是会按照上一次方法结 束时间开始算起。

4、initialDelay 。如： @Scheduled(initialDelay = 10000,fixedRate = 15000 这个定时器就是在上一个的基础上加了一个initialDelay = 10000 意思就是在容器启动后,延迟10秒后再 执行一次定时器,以后每15秒再执行一次该定时器

> quartz自学

# 五 、SpringBoot与整合其他技术                   

## 1 、SpringBoot整合Junit                                                                

添加Junit的起步依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

 编写测试类

 

```java
package com.xinzhi.test;

import  com.xinzhi.MySpringBootApplication;
import com.xinzhi.domain.User;
import com.xinzhi.mapper.UserMapper;
import org.junit.Test;
import org.junit.runner.RunWith;
import  org.springframework.beans.factory.annotation.Autowired;
import  org.springframework.boot.test.context.SpringBootTest;
import  org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

@SpringBootTest
class StudySpringbootApplicationTests {

	@Autowired 
    private Cat cat;

	@Test
	void contextLoads() { 
        System.out.println(cat);
	}
}
```

> 控制台打印信息

## 2、整合Spring Data JPA                                                                     

 JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映 射关系，并将运行期的实体[对象持久化](https://baike.baidu.com/item/%E5%AF%B9%E8%B1%A1%E6%8C%81%E4%B9%85%E5%8C%96/7316192)到数据库中。

JPA是需要Provider来实现其功能的，Hibernate就是JPA Provider中很强的一个，应该说无人能出其 右。从功能上来说，JPA就是Hibernate功能的一个子集。Hibernate 从3.2开始，就开始兼容JPA。 Hibernate3.2获得了Sun TCK的JPA(Java Persistence API) 兼容认证。

> 添加Spring Data JPA的起步依赖

```xml
<!-- springBoot JPA的起步依赖 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

> 添加数据库驱动依赖

```xml
<!-- MySQL连接驱动 -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

> 在application.properties中配置数据库和jpa的相关属性

```
spring:   
	#通用的数据源配置 
	datasource:   
		url: jdbc:mysql://localhost:3306/ssm?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
		driver-class-name: com.mysql.cj.jdbc.Driver username: root
		password: root 
	jpa:
		#这个参数是在建表的时候，将默认的存储引擎切换为 InnoDB 用的 
		database-platform:    org.hibernate.dialect.MySQL5InnoDBDialect   
		#配置在日志中打印出执行的SQL语句信息.
		show-sql: true
		hibernate: 
			#配置指明在程序启动的时候要删除并且创建实体类对应的表
			ddl-auto: create
ddl-auto可选参数
create  启动时删数据库中的表，然后创建，退出时不删除数据表
create-drop 启动时删数据库中的表，然后创建，退出时删除数据表 如果表不存在报错
update  如果启动时表格式不一致则更新表，原有数据保留
validate 项目启动表结构进行校验 如果不一致则报错
```

>  创建实体配置实体

```
@NoArgsConstructor
@Data
@Entity
@Table(name = "user") public class User {
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)//主键生成策略
	@Column(name="id") 
	private Integer id;
	@Column(name = "user_name") 
	private String username;
	@Column(name = "password") 
	private String password;
	@Column(name = "birthday")
	private Date birthday;
}
```

> 主键生成策略

**JPA**提供的四种标准用法为**TABLE**,**SEQUENCE**,**IDENTITY**,**AUTO**.

1、**TABLE**：使用一个特定的数据库表格来保存主键。

使用一个特定的数据库表格来保存主键,持久化引擎通过关系数据库的一张特定的表格来生成主键,这种策 略的好处就是不依赖于外部环境和数据库的具体实现,在不同数据库间可以很容易的进行移植,但由于其不 能充分利用数据库的特性,所以不会优先使用。

2、**SEQUENCE**：根据底层数据库的序列来生成主键，条件是数据库支持序列。

在某些数据库中,不支持主键自增长,比如Oracle,其提供了一种叫做"序列(sequence)"的机制生成主键。 此时,GenerationType.SEQUENCE就可以作为主键生成策略。该策略的不足之处正好与TABLE相反,由于 只有部分数据库(Oracle,PostgreSQL,DB2)支持序列对象,MYsql不支持序列,所以该策略一般不应用于其 他数据库。类似的,该策略一般与另外一个注解一起使用@SequenceGenerator,@SequenceGenerator 注解指定了生成主键的序列.然后JPA会根据注解内容创建一个序列(或使用一个现有的序列)。如果不指定 序列,则会自动生成一个序列SEQ_GEN_SEQUENCE。

3、**IDENTITY**：主键由数据库自动生成（主要是自动增长型）

此种主键生成策略就是通常所说的主键自增长,数据库在插入数据时,会自动给主键赋值,比如MySQL可以 在创建表时声明"auto_increment" 来指定主键自增长。该策略在大部分数据库中都提供了支持(指定方 法或关键字可能不同),但还是有少数数据库不支持,所以可移植性略差。使用自增长主键生成策略是只需 要声明strategy = GenerationType.IDENTITY即可。

4、**AUTO**：主键由程序控制。

把主键生成策略交给持久化引擎(persistence engine),持久化引擎会根据数据库在以上三种主键生成策略 中选择其中一种。此种主键生成策略比较常用,由于JPA默认的生成策略就是GenerationType.AUTO,所以 使用此种策略时.可以显式的指定@GeneratedValue(strategy    =   GenerationType.AUTO)也可以直接

@**GeneratedValue**。

>  编写**UserRepository**

```java
public interface UserRepository extends JpaRepository<User,Long>{
	public List<User> findAll();
}
```

> 编写测试类

```java
@SpringBootTest(classes=MySpringBootApplication.class)
public class JpaTest {
	@Autowired
	private UserRepository userRepository;

	@Test
	public void test(){
	List<User> users = userRepository.findAll();
		System.out.println(users);
	}
}
```

> 控制台打印信息

 

 

**HQL**是**Hibernate**    **Query**   **Language**的缩写，提供更加丰富灵活、更为强大的查询能力;HQL更接近SQL

语句查询语法。

​	对查询条件进行了[面向对象](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1)封装，符合编程人员的思维方式，格式：from + 类名 + 类对象 + where + 对象的属性

​	**区分大小写,关键字不区分大小写 从下标0开始计算位置(hibernate5之后不支持) 支持命名参数**

```java
@Query("select u from User1 u where u.username = :aaa")
List<User1> findByUsername(@Param("aaa") String aaa);
```

如果想使用原生sql

```java
@Query(value	=	"select	*	from	t_users	where	username	=?",nativeQuery	=	true)
```

分页处理，一切皆对象

```java
Sort sort = Sort.by(Sort.Direction.DESC,"id");
Pageable pageable = PageRequest.of(0,3,sort);
Page<User> all = userDao.findAll(pageable);
```

如图1所示，会议提前关闭问题

**spring**整合**hibernate**或者**spring** **boot**里使用**jpa**，本质都是**hibernate**的**sesison**操作数据库，默认 **sesison**会提前关闭，报延迟加载..异常，要等到后台值返回到视图层之后，才关闭**sesison**才合理;

**spring** **boot**里这样设置即可，延续**session**到返回视图层

##  3、 SpringBoot整合Mybatis                                         

> 添加Mybatis的起步依赖

```xml
<!--mybatis起步依赖-->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>2.1.2</version>
</dependency>
<!-- MySQL连接驱动 -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```

> 添加数据库连接信息

在application.properties中添加数据量的连接信息

```yml
#DB Configuration: 
spring:   
	datasource:   
		url: jdbc:mysql://localhost:3306/ssm?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai   
	driver-class-name: com.mysql.cj.jdbc.Driver
	username: root
	password: root   
```

> 创建user表

在test数据库中创建user表

```sql
-- ----------------------------
-- Table structure for `user`
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`username` varchar(50) DEFAULT NULL,
`password` varchar(50) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', 'zhangsan', '123');
INSERT INTO `user` VALUES ('2', 'lisi', '123');

```

>  创建实体Bean

```java
/**
 * @author AiLi
 * @date 2021/6/5
 */
@Data
@AllArgsConstructor
@NoArgsConstructor 
public class User {

	private int id;
	private String username; 
	private String password;
}
```

> 编写Mapper

```java
@Mapper
public interface UserMapper {
	public List<User> queryUserList(); 
}
```

注意：@Mapper标记该类是一个mybatis的mapper接口，可以被spring boot自动扫描到spring上下文 中

> 配置Mapper映射文件 

在src\main\resources\mapper路径下加入**UserMapper**.**xml**配置文件"

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper   PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">   
<mapper    namespace="com.xinzhi.dao.UserMapper"> 
    <select   id="queryUserList" resultType="user">
        select * from user   
    </select>
</mapper>
```

> 在application.properties中添加mybatis的信息

```yml
#spring集成Mybatis环境
mybatis:
	type-aliases-package: com.example.entity
	mapper-locations: mapper/*Mapper.xml
```

> service层

```java
/**
 * @author AiLi
 * @date 2020/6/5
 */
public interface IUserService {
	/**
	 * 获取用户信息
	 * @return
	 */
	List<User> getAllUsers();
}
@Service
public class UserServiceImpl implements IUserService {

	@Resource
	private UserMapper userMapper;

	@Override
	public List<User> getAllUsers() { 
        return userMapper.queryUserList();
    }
}
```

> 编写测试Controller

```java
/**
 * @author AiLi
 * @date 2021/6/5
 */
@Controller
@RequestMapping("/user") 
public class UserController {
	@Resource
	private IUserService userService;

	@GetMapping
	@ResponseBody
	public List<User> getUsers(){
		return userService.getAllUsers();
	}
}
```

> 测试

## 4、德鲁伊数据源                                                            

```java
/**
 * @author AiLi
 * @date 2021/6/5
 */
@Configuration
public class DruidConfig {
	@ConfigurationProperties(prefix = "spring.datasource")
	@Bean
	public DataSource druid() {
		return new DruidDataSource();
	}
}
```

>  配置

```yml
spring: 
	datasource:
		username: root 
		password: root
		driver-class-name: com.mysql.cj.jdbc.Driver 
		url:  jdbc:mysql://127.0.0.1:3306/ssm?
characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStateme nts=true
		type:  com.alibaba.druid.pool.DruidDataSource
		#	数据源其他配置
		initialSize: 5
		minIdle: 5
		maxActive: 20
		maxWait: 60000
		timeBetweenEvictionRunsMillis: 60000
		minEvictableIdleTimeMillis: 300000 validationQuery: SELECT 1 FROM DUAL testWhileIdle: true
		testOnBorrow: false testOnReturn: false poolPreparedStatements: true
		#	配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
		filters: stat,wall maxPoolPreparedStatementPerConnectionSize: 20 
		useGlobalDataSourceStat: true 					
		connectionProperties:
druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

```

 重新运行

看后台已经切换了 还有监控呢？ 完善配置文件

```java
/**
 * @author AiLi
 * @date 2020/6/5
 */
@Configuration
public class DruidConfig {
	/**
	 * 注入数据源
 	 * @return
	 */
	@ConfigurationProperties(prefix = "spring.datasource")
	@Bean
	public DataSource druid() {
		return new DruidDataSource();
	}
	/**
	 * 配置监控
	 * @return
	 */
	@Bean
	public ServletRegistrationBean statViewServlet(){ 
		ServletRegistrationBean bean = new ServletRegistrationBean(newStatViewServlet(), "/druid/*");
		HashMap<String, String> map = new HashMap<>(2); 
         map.put("loginUsername","xinzhi"); 			
		map.put("loginPassword","123456"); 
         bean.setInitParameters(map);
		return bean;
	}

	@Bean
	public FilterRegistrationBean webStatFilter(){ 
		FilterRegistrationBean<Filter> bean = new 
		bean.setFilter(new WebStatFilter()); 
         HashMap<String, String> map = new HashMap<>(8); 		
		map.put("exclusions","*.js"); 
         bean.setInitParameters(map); 
         bean.setUrlPatterns(Arrays.asList("/*"));
		return bean;
	}
}
```

​                      }                     

搞定

## 5、pagehelper                                                         

为了方便分页

> 引入依赖

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.2.10</version>
</dependency>
```

> 在application.yml中做如下配置

```yml
# 分页配置
pagehelper:
	helper-dialect: mysql
	reasonable: true
	support-methods-arguments: true
	params: count=countSql
```

> 在代码中使用,（service或controller）

```java
//这行是重点，表示从pageNum页开始，每页pageSize条数据
PageHelper.startPage(pageNum,pageSize);
List<Tools> list = toolsMapper.findAll();
PageInfo<Tools> pageInfo = new PageInfo<Tools>(list);
return  ServerResponse.createBySuccess("查询成功",pageInfo);
```

> 开启日志

```yml
mybatis:
	mapper-locations: mapper/*.xml
	type-aliases-package:  cn.itnanls.studyspringboot.entity
	configuration:
		map-underscore-to-camel-case: true
		log-impl:  org.apache.ibatis.logging.stdout.StdOutImpl
```

## 6 、Thymeleaf模板引擎                                                  

###### Thymelaf整合SpringBoot

> 在pom.xml文件引入thymeleaf

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

> 在application.properties（application.yml）文件中配置thymeleaf
>  

```yml
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.check-template-location=true
spring.thymeleaf.suffix=.html
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.content-type=text/html
spring.thymeleaf.mode=HTML5
spring.thymeleaf.cache=false
```

> 新建编辑控制层代码HelloController，在request添加了name属性，返回到前端hello.html再使 用thymeleaf取值显示。

```java
@GetMapping("/login")
public String toLogin(HttpServletRequest request){
	request.setAttribute("name","zhangsan");
	return "login";
}
```

> 新建编辑模板文件，在resources文件夹下的templates目录，用于存放HTML等模板文件，在这新 增hello.html，添加如下代码。

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
	<meta charset="UTF-8"/>
	<title>springboot-thymeleaf  demo</title>
</head>
<body>
	<p th:text="'hello, ' + ${name} + '!'" />
</body>
</html>
```

切记：使用Thymeleaf模板引擎时，必须在html文件上方添加该行代码使用支持Thymeleaf。

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

5. 启动项目，访问http://localhost:8080/user/login，看到如下显示证明SpringBoot整合Thymeleaf 成功。

博客学习该模板引擎语法

<https://blog.csdn.net/qq_24598601/article/details/89190411>

>  路径映射 如果觉得写controller太麻烦一次性多映射一些常用的地址

```java
@Override
protected void addViewControllers(ViewControllerRegistry registry) {
	registry.addViewController("/login").setViewName("login.html");
	registry.addViewController("/register").setViewName("register.html");
	super.addViewControllers(registry);
}
```

## 7 、集成Swagger                                                            

#### （1）POM添加依赖

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-boot-starter</artifactId>
	<version>3.0.0</version>
</dependency>
```

**（**2**）**Application**启动应用类上面加入**@EnableOpenApi**注解**

```java
@EnableOpenApi
```

**（3）**Swagger3Config**的配置**

```java
/**
 * @author AiLi
 * @date 2021/3/11
 */
@Configuration
public class Swagger3Config {
	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.OAS_30)
				.apiInfo(apiInfo())
				.select()
				//.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
				.apis(RequestHandlerSelectors.basePackage("cn.itnanls.springbootstudy.contro ller"))
				.paths(PathSelectors.any())
				.build();
	}

	private ApiInfo apiInfo() { 
		return new ApiInfoBuilder()
				.title("itnanls自学网")
				.description("一个给大家学习的网站")
				.contact(new Contact("itnanls", "www.itnanls.cn", "510180298@qq.com"))
				.version("1.0")
				.build();
	}
}
```

![1624462099486](..\img\springboot\1624462099486.png)
 **（**4**）完善**controller

```
 

 
/**
 * @author AiLi 
 * @date 2021/3/11
 */
@RestController
@RequestMapping("user")
@Api(tags = "用户信息的curd操作api")
public class UserController {
	@GetMapping("{id}")
	@ApiOperation("根据id获取用户")
	@ApiImplicitParam(name = "id",value = "用户的id",paramType = "path",dataType = "Integer")
	@ApiResponse(code = 200,message = "成功")
	public String getUser(@PathVariable Integer id){
		return id + "号员工：zhangsan"; 
     }
     
	@PostMapping
	@ApiOperation("保存用户")
	@ApiImplicitParams({
	@ApiImplicitParam(name = "username", value = "用户的用户名", paramType = "query", dataType = "String"),
	@ApiImplicitParam(name = "password", value = "用户的密码")
	})
	@ApiResponses({
	@ApiResponse(code = 200,message = "成功"),
	@ApiResponse(code = 2001,message = "用户已经存在"),
	@ApiResponse(code = 2002,message = "密码不合法") 28   })
	public String getUser(String username,String password){
	return "Success";
	}
}
```

![1624462335236](..\img\springboot\1624462335236.png)

#### （5）换皮肤

访问地址：http://localhost:8080/docs.html

```xml
<dependency>
	<groupId>com.github.caspar-chen</groupId>
	<artifactId>swagger-ui-layer</artifactId>
	<version>1.1.3</version>
</dependency>
```

## 8、SpringBoot整合Redis

**Lettuce** 和 **Jedis** 都是**Redis**的**client**，所以他们都可以连接 Redis Server。 Jedis在实现上是直接连接的Redis Server，如果在多线程环境下是非线程安全的。  每个线程都去拿自己的   Jedis  实例，当连接数量增多时，资源消耗阶梯式增大，连接成本就较高了。

Lettuce的连接是基于Netty的，Netty 是一个多线程、事件驱动的 I/O 框架。连接实例可以在多个线程 间共享，当多线程使用同一连接实例时，是线程安全的。 所以，一个多线程的应用可以使用同一个连接实例，而不用担心并发线程的数量。 当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。

通过异步的方式可以让我们更好的利用系统资源，而不用浪费线程等待网络或磁盘I/O。 所以 Lettuce 可以帮助我们充分利用异步的优势。

> 添加redis的起步依赖

```xml
<!-- 配置使用redis启动器 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

> 配置redis的连接信息

```yml
#Redis
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

>  注入RedisTemplate测试redis操作

jackson的核心方法：

```yml
String userJson = objectMapper.writeValueAsString(new User());
System.out.println(userJson);
User user = objectMapper.readValue(userJson, User.class);
System.out.println(user);
```

```java
/**
* @author AiLi
* @date 2021/6/6
*/
@SpringBootTest
public class RedisTest {

	@Resource
	private UserMapper userMapper;

	@Autowired
	private RedisTemplate<String, String> redisTemplate;

	@Test
	public void test() throws JsonProcessingException {
		BoundHashOperations<String, Object, Object> hash = redisTemplate.boundHashOps("user:1");
		hash.put("username","zhangsan"); hash.put("age","12");
		hash.put("password","1233");
		BoundValueOperations<String, String> userList = redisTemplate.boundValueOps("user:list");

		//从redis缓存中获得指定的数据
		String usersJson = userList.get();
		//如果redis中没有数据的话 
		if(null==usersJson){
			//查询数据库获得数据
			List<User> users = userMapper.queryUserList();
			//转换成json格式字符串
			ObjectMapper om = new ObjectMapper(); usersJson = om.writeValueAsString(users);
			//将数据存储到redis中，下次在查询直接从redis中获得数据，不用在查询数据库
			redisTemplate.boundValueOps("user:list").set(usersJson);

			System.out.println("===============从数据库获得数据===============");
		}else{
			System.out.println("===============从redis缓存中获得数据===============");
		}

			System.out.println(usersJson);

	}
}
```

注入自定义的redisTemplate，因为有一个需求我们经常想直接以String为key，value是Object去存储

```java
@Bean
public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
	RedisTemplate<String,Object> redisTemplate = new RedisTemplate<>();
	// 设置不同的序列化方式 redisTemplate.setKeySerializer(RedisSerializer.string()); 			
	redisTemplate.setValueSerializer(RedisSerializer.json()); 	
	redisTemplate.setHashKeySerializer(RedisSerializer.string()); 
	redisTemplate.setHashValueSerializer(RedisSerializer.json());
	// 设置redis的连接工厂，自动装配时已经注入了 
	redisTemplate.setConnectionFactory(factory); return redisTemplate;
}
```

```java
@Autowired
private RedisTemplate<String,Object> redisTemplate; 
	@Test
	void contextLoads() {
	ValueOperations<String, Object> opsForValue = redisTemplate.opsForValue();
	opsForValue.set("user",new User());
}
```

## 9 、集成shiro（先学shiro）                                            

shiro是web开发中常用的使用安全管理框架，通过shiro-spring-boot-web-starter方式集成Shiro到 springboot2可以简化配置。

#### （1）引包

maven方式在项目pom.xml中引入shiro starter包的坐标，这里引用了1.4.1版本

```xml
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring-boot-web-starter</artifactId>
	<version>1.7.1</version>
</dependency>
```

#### （2）配置

**starter**已经做了很多自动配置工作，具体可以参考ShiroAutoConfiguration.java、 ShiroBeanAutoConfiguration.java和ShiroWebAutoConfiguration.java这几个文件。

这里使用新建shiroConfig.java类方式进行shiro配置。主要配置Realm、url过滤器、密码匹配器和安全 管理器这几个组件就可以让shiro正常工作。

```java
@Configuration
public class shiroConfig {// 配置自定义Realm
	@Bean
	public UserRealm userRealm() {
		UserRealm userRealm = new UserRealm();
		userRealm.setCredentialsMatcher(credentialsMatcher()); //配置使用哈希 密码匹配
		return userRealm;
    }
	// 配置url过滤器
 	@Bean
 	public ShiroFilterChainDefinition shiroFilterChainDefinition() {
		DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
		chainDefinition.addPathDefinition("/captcha",  "anon");
		chainDefinition.addPathDefinition("/logout","anon");
		chainDefinition.addPathDefinition("/layuiadmin/**",  "anon");
    	chainDefinition.addPathDefinition("/druid/**",  "anon");
 		chainDefinition.addPathDefinition("/api/**",  "anon");
		// all other paths require a logged in user
 		chainDefinition.addPathDefinition("/login","anon");
		chainDefinition.addPathDefinition("/**",  "authc");
		return chainDefinition;
    }
		// 设置用于匹配密码的CredentialsMatcher
	@Bean
    public HashedCredentialsMatcher credentialsMatcher() {
         HashedCredentialsMatcher credentialsMatcher = new
		HashedCredentialsMatcher(); credentialsMatcher.setHashAlgorithmName(Sha256Hash.ALGORITHM_NAME);
		// 散列算法，这里使用更安全的sha256算法
		credentialsMatcher.setStoredCredentialsHexEncoded(false);	// 数据库 存储的密码字段使用HEX还是BASE64方式加密
		credentialsMatcher.setHashIterations(1024);	// 散列迭代次数
		return credentialsMatcher;
	}

	// 配置security并设置userReaml，避免xxxx required a bean named 'authorizer' that could not be found.的报错
	@Bean
	public SessionsSecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager(); 			
        securityManager.setRealm(userRealm()); 
        return securityManager;
    }
}
```

#### （3）登录页配置

shiro默认的登录页是/login.jsp，需要在项目配置文件application.yml中修改默认登录页等配置。）

```yml
shiro:
	loginUrl: /toLogin
	successUrl: /
```

#### （4）自定义realm

```java
/**
 * @author itnanls
 * @date 2021/3/10
 */
public class UserRealm extends AuthorizingRealm {
	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		String username = (String) SecurityUtils.getSubject().getPrincipal();
		SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
		// 角色
		Set<String> roles = new HashSet<>(); Set<String> permissions = new HashSet<>();
		// 权限
		// 测试用权限
		if ("admin".equals(username)) { 
			roles.add("admin"); 
			permissions.add("op:write");
		} else {
			roles.add("user");
			permissions.add("op:read");
		}
		authorizationInfo.setRoles(roles);
		authorizationInfo.setStringPermissions(permissions);
		return authorizationInfo;
	}
 	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
		String username = (String) token.getPrincipal(); 30	// 从数据库查出user
		// User user = userDao.getByUsername(username);
		User user = new User();
		if (user == null) {
			throw new UnknownAccountException(); // 账号不存在
		}
		if (user.getStatus() != 0) {
			throw new LockedAccountException();	// 账号被锁定
		}
		String salt = user.getSalt();
		SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user,
		user.getPassword(),
		ByteSource.Util.bytes(salt),
		getName());
		return authenticationInfo;
	}
}
```

#### （5）登录方法
 

```java
@ResponseBody
@PostMapping("login")
public JsonResult doLogin(String username, String password, HttpServletRequest request) {
	try {
		UsernamePasswordToken token = new UsernamePasswordToken(username, password);
		SecurityUtils.getSubject().login(token);
		//addLoginRecord(getLoginUserId(),  request);	// 记录登录信息
		return R.ok("登录成功").put("data", map);
	} catch (IncorrectCredentialsException ice) { 
		return R.error("密码错误");
	} catch (UnknownAccountException uae) { 
		return R.error("账号不存在");
	} catch (LockedAccountException e) { 
		return R.error("账号被锁定");
	} catch (ExcessiveAttemptsException eae) {
		return  JsonResult.error("操作频繁，请稍后再试");
	}
}
```

（6）接口权限验证

```java
@RequiresRoles("user")
@GetMapping("/user/list.html") 
public String userList() {
	return "user/user/list";
}
@RequiresPermissions("op:read")
@GetMapping("user/userform.html") 
public String userForm() {
	return "user/user/userform";
}
```

#### （7）shiro默认过滤器

shiro提供和多个默认的过滤器，我们可以用这些过滤器来配置过滤指定url的访问权限。） 常见过滤器

|                   |                                |                                                              |
| ----------------- | ------------------------------ | ------------------------------------------------------------ |
| anon              | AnonymousFilter                | 指定url可以匿名访问                                          |
| authc             | FormAuthenticationFilter       | 指定url需要form表单登录，默认会 从请求中获取username、 password,rememberMe等参数并 尝试登录，如果登录不了就会跳转到   loginUrl配置的路径。我们也可以用 这个过滤器做默认的登录逻辑，但是 一般都是我们自己在控制器写登录逻 辑的，自己写的话出错返回的信息都 可以定制嘛。 |
| logout            | LogoutFilter                   | 登出过滤器，配置指定url就可以实 现退出功能，非常方便         |
| noSessionCreation | NoSessionCreationFilter        | 禁止创建会话                                                 |
| perms             | PermissionsAuthorizationFilter | 需要指定权限才能访问                                         |
| roles             | RolesAuthorizationFilter       | 需要指定角色才能访问                                         |
| ssl               | SslFilter                      | 需要https请求才能访问                                        |
| user              | UserFilter                     | 需要已登录或“记住我”的用户才能访 问                          |


 

#### （8）shiro常用注解

可以在控制器类上使用 常见注解

@RequiresGuest 只有游客可以访问

@RequiresAuthentication  需要登录才能访问

@RequiresUser    已登录的用户或“记住我”的用户能访问

@RequiresRoles     已登录的用户需具有指定的角色才能访问

@RequiresPermissions 已登录的用户需具有指定的权限才能访问

#### （9）整合thymleaf

1、添加依赖

```xml
<dependency>
	<groupId>com.github.theborakompanioni</groupId>
	<artifactId>thymeleaf-extras-shiro</artifactId>
	<version>2.0.0</version>
</dependency>
```

2、在ShiroConfig中配置ShiroDialect

```java
/**
 * shiro方言	支持shiro标签
 * @return
 */
@Bean
public ShiroDialect shiroDialect() {
	return new ShiroDialect();
}
```

3、html中导入shiro标签

```html
<!DOCTYPE html>
<html lang="en"
xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>
	<meta charset="UTF-8">
	<title>itnanls</title>
</head>
<body>
	<form id="loginForm" >
		Username: <input type="text" name="username"/> <br/> Password: <input type="password" name="password"/><br/>
		<input type="checkbox" name="rememberMe" value="true"/>Remember Me<br/>
		<input type="button" id="loginBtn" value="login">
	</form>
<script   src="https://cdn.bootcdn.net/ajax/libs/jquery/2.2.4/jquery.min.js">
</script>
</body>
</html>
```

登录测试！

##  10、整合spring-security                                               

1、加入页面映射

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
	public void addViewControllers(ViewControllerRegistry registry) { 		 
		registry.addViewController("/home").setViewName("home"); 	
		registry.addViewController("/").setViewName("home"); 
		registry.addViewController("/hello").setViewName("hello"); 
		registry.addViewController("/login").setViewName("login");
	}
}
```

2、映入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-test</artifactId>
	<scope>test</scope>
</dependency>
```

3、编写配置类

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception { 
		http.authorizeRequests()
			.antMatchers("/",  "/home").permitAll()
			.anyRequest().authenticated()
			.and()
			.formLogin()
			.loginPage("/login")
			.permitAll()
			.and()
			.logout()
			.permitAll();
	}

	@Bean
	@Override
	public UserDetailsService userDetailsService() { 
		UserDetails user = User.withDefaultPasswordEncoder()
				.username("user")
				.password("password")
				.roles("USER")
				.build();

		return new InMemoryUserDetailsManager(user);
	}
}
```

authenticated() 保护URL，需要用户登录

permitAll()   指定URL无需保护，一般应用与静态资源文件

hasRole(String role) 限制单个角色访问，角色将被增加 “ROLE_” .所以”ADMIN” 将和

“ROLE_ADMIN”进行比较.

hasAuthority(String authority) 限制单个权限访问 hasAnyRole(String… roles)允许多个角色访问. hasAnyAuthority(String… authorities) 允许多个权限访问.

access(String attribute) 该方法使用 SpEL表达式, 所以可以创建复杂的限制. hasIpAddress(String ipaddressExpression) 限制IP地址或子网

 4、login.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras- springsecurity3">
<head>
	<title>Spring Security Example </title>
</head>
<body>
	<div th:if="${param.error}"> 
		Invalid username and password.
	</div>
	<div th:if="${param.logout}"> 
		You have been logged out.
	</div>
	<form th:action="@{/login}" method="post">
		<div><label> User Name : <input type="text" name="username"/>
		</label></div>
		<div><label> Password: <input type="password" name="password"/>
		</label></div>
		<div><input type="submit" value="Sign In"/></div>
	</form>
</body>
</html>
```

5、hello.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras- springsecurity3">
<head>
	<title>Hello World!</title>
</head>
<body>
	<h1 th:inline="text">Hello [[${#httpServletRequest.remoteUser}]]!
	</h1>
	<form th:action="@{/logout}" method="post">
		<input type="submit" value="Sign Out"/>
	</form>
</body>
</html>
```

6、自定义

认证流程

 ![1624463843374](..\img\springboot\1624463843374.png)

```java
/**
* @author AiLi
* @date 2021/3/11
*/
@Service
public class SpringDataUserDetailsService implements UserDetailsService {
	@Override
	public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
		return User
		.withUsername("zhangsan")

		.password("$2a$10$qsqPRUfW5LQUXEQrgOTqbuGMpuRFDqs2up15ZvNm33LA9QXRZfUma")
		.roles("admin")
		.authorities("user:add").build();
	}
}
```

密码编码器

```java
@Bean
public PasswordEncoder passwordEncoder(){
	return new BCryptPasswordEncoder();
}
```

获取当前用户的方法

```java
/**
 * 获取当前登录用户名
 * @return 
 */
private String getUsername(){
	SecurityContext securityContext = SecurityContextHolder.getContext();
	return  securityContext.getAuthentication().getName();
}
```

我们可以在任何 @Configuration 实例上使用 @EnableGlobalMethodSecurity(prePostEnabled = true)

注释来启用基于注解的安全性。

> 方法注解

###### @Secured注释用于指定方法上的角色列表。因此，如果用户至少具有一个指定的角色，则用户能访问 该方法。

我们定义一个getUsername方法：

 

```java
@Secured("ROLE_VIEWER")
public String getUsername() {
	SecurityContext securityContext = SecurityContextHolder.getContext();
	return  securityContext.getAuthentication().getName();
}
```

我们可以传入多个

```java
@Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername(String username) {
	return  userRoleRepository.isValidUsername(username);
}
```

###### @RoleAllowed注释是JSR-250对@Secured注释的等效注释。

基本上，我们可以像@Secured一样使用@RoleAllowed注释。

```java
@RolesAllowed("ROLE_VIEWER")
	public String getUsername2() {
}
@RolesAllowed({ "ROLE_VIEWER", "ROLE_EDITOR" })
	public boolean isValidUsername2(String username) {
}
```

同样，只有当用户至少具有ROLE_VIEWER或ROLER_EDITOR角色之一时，用户才能调用isValidUsername2。

###### @PreAuthorize和@PostAuthorize注释都提供基于表达式的访问控制。 因此，可以使用SpEL（Spring Expression Language）编写。

**@PreAuthorize****注释在进入方法之前检查给定的表达式，而****@PostAuthorize****注释在执行方法后验证它 并且可能改变结果**。

现在，让我们声明一个getUsernameInUpperCase方法，如下所示：

```java
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
	return getUsername().toUpperCase();
}
```

```java
@PreAuthorize("hasRole('ROLE_VIEWER') or hasRole('ROLE_EDITOR')")
public boolean isValidUsername3(String username) {
	//...
}
```

而且，我们实际上可以使用method参数作为表达式的一部分：

```java
@PreAuthorize("#username == authentication.principal.username")
public String getMyRoles(String username) {
	//...
}
```

这里，只有当参数username的值与当前主体的用户名相同时，用户才能调用getMyRoles方法。

###### 值得注意的是，@ PreAuthorize表达式可以替换为@PostAuthorize表达式。


 

**让我们重写getMyRoles**：

```java
@PostAuthorize("#username == authentication.principal.username")
public String getMyRoles2(String username) {
	//...
}
```

但是，在上一个示例中，授权将在执行目标方法后延迟。

###### 此外，@ PostAuthorize注释提供了访问方法结果的能力：

```java
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
	return  userRoleRepository.loadUserByUserName(username);
}
```

在此示例中，如果返回的CustomUser的用户名等于当前身份验证主体的昵称，则loadUserDetail方法 会成功执行。  

# 六、自动装配                                               

## 1 、原理                                                                           

1）SpringBoot启动的时候加载主配置类，开启了自动配置功能@EnableAutoConfiguration

2）@EnableAutoConfiguration作用:

利用EnableAutoConfigurationImporttSelector给容器中导入一些组件； 可以插件selectImports()方法的内容；

List configurations = getCandidateConfigurations(annotationMetadata,arrtibutes)获取候选的 配置

SpringFactoriesLoader.loadFactoryNames()会扫描所有jar包类路径下的META-INF/spring.factories文 件

把扫描到的这些文件的内容包装成properties对象，从properties中获取到

EnableAutoConfiguration.class类(类名)对应的值,然后把他们添加在容器中。

**将类路径下META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到了容  器中**

```factories
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfigurtion,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactor yAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguratio  n,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoC onfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientH ttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClie ntAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoCon figuration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryA utoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfig uration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfigur ation,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfigurati on,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactive AutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAu toConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessaging AutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfigura tion,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplat eAutoConfiguration

# Failure analyzers
org.springframework.boot.diagnostics.FailureAnalyzer=\
org.springframework.boot.autoconfigure.diagnostics.analyzer.NoSuchBeanDefin itionFailureAnalyzer,\
org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAn alyzer,\
org.springframework.boot.autoconfigure.jdbc.HikariDriverConfigurationFailur eAnalyzer,\
org.springframework.boot.autoconfigure.session.NonUniqueSessionRepositoryFa ilureAnalyzer

# Template availability providers
org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvide  r=\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailab ilityProvider,\
org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilit yProvider,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvaila bilityProvider,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabil ityProvider,\
org.springframework.boot.autoconfigure.web.servlet.JspTemplateAvailabilityProvider
```

每一个这样的AutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做配置

3）每一个自动配置类进行自动配置功能；

4）以HttpEncodingAutoConfiguration (Http编码自动配置)为例解释自动配置原理；

```java
//表示这是一个配置类没以前编写的配置文件一样，也可以给容器中添加组件
@Configuration
//启动指定类的ConfigurationProperties功能，将配置文件中对应的值和 HttpEncodingAutoConfigurationProperties绑定起来；
@EnableConfigurationProperties({HttpProperties.class})
//Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置 就会生效(即判断当前应用是否是web应用)
@ConditionalOnWebApplication( type = Type.SERVLET)
//判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码处理的过滤器
@ConditionalOnClass({CharacterEncodingFilter.class})
@ConditionalOnProperty(prefix = "spring.http.encoding", value = {"enabled"}, matchIfMissing = true)
public class HttpEncodingAutoConfiguration{
//给容器中添加一个组件
@Bean
@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() { 
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());

        filter.setForceRequestEncoding(
            this.properties.shouldForce(
                org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));

        filter.setForceResponseEncoding(
            this.properties.shouldForce(
                org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
		return filter;
	}
}
根据当前不同的条件判断，决定这个配置类是否生效？
一旦这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的
properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的.
```

 5）所有在配置文件中能配置的属性都是在xxxxPropertites类中封装着，配置文件能配置什么就可以参 照某个功能对应的这个属性类；

```java
//从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(
prefix = "spring.http" 4	)
public class HttpProperties {
private boolean logRequestDetails;
private final HttpProperties.Encoding encoding = new HttpProperties.Encoding();
```

## 2、自定义starter

> 新建hello-spring-boot-starter工程

> 1.pom.xml

 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                    https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.xinzhi</groupId>
	<artifactId>hello-spring-boot-starter</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>hello-spring-boot-starter</name>
	<description>Demo project for Spring Boot</description> 16
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure</artifactId>
		</dependency>
	</dependencies>
</project>
```

同时删除 启动类、resources下的文件，test文件。

> HelloProperties

```java
/**
* @author AiLi
* @date 2020/6/5
*/
@ConfigurationProperties(prefix = "itnanls.user") 
public class UserProperties {
	private String username = "张三";
	private String password = "123";
	public String getUsername() { 
		return username;
	}

	public void setUsername(String username) { 
		this.username = username;
	}

	public String getPassword() { 
		return password;
	}

	public void setPassword(String password) { 
		this.password = password;
	}
}
```

> HelloService

```java
/**
* @author AiLi
* @date 2020/6/5
*/
@Component
public class User {
	@Resource
	private UserProperties userProperties;

	public void say(){
		System.out.println(userProperties.getUsername()+"的密码 是："+userProperties.getPassword());
	}
}
```

> UserAutoConfiguration

```java
/**
* @author AiLi
* @date 2020/6/5
*/
@Configuration
@ConditionalOnClass(User.class)
@ConditionalOnProperty(prefix = "xinzhi.user",value = "enable",matchIfMissing = true)
@EnableConfigurationProperties(UserProperties.class) 
public class UserAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean public User greeter() {
		return new User();
	}
}
```

#### 5. spring.factories

在 **resources** 下创建文件夹 **META-INF** 并在 **META-INF** 下创建文件 **spring.factories** ，内容如下：

```factories
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
cn.itnanls.config.UserAutoConfiguration
```

到这儿，我们的配置自定义的starter就写完了

## 三 、测试自定义starter                                                           

我们创建个项目 **hello-spring-boot-starter-test**，来测试系我们写的stater。

### 1.  pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                    https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>spring-boot-initializr</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-boot-initializr</name>
    <description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>com.xinzhi</groupId>
			<artifactId>hello-spring-boot-starter</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

**2.**  **HelloController**

```java
@Controller
public class UserController {
	@Resource
	private User user;

	@GetMapping("/user")
	@ResponseBody
	public void getUser(){
		user.say();
	}
}
```

### 3.  修改application.properties

```properties
xinzhi.user.username=李四
xinzhi.user.password=abc
```



需要的组件