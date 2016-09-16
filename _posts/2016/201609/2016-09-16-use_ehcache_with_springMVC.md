---
layout: post
title: use ehcache with springMVC
---
-----
# use ehcache with springMVC
在springMVC中使用ehcache
***
### 创建maven工程
该项目的parent工程：springehcache-parent包含两个module：springehcache-web和springehcache-service。
> parent的pom文件配置如下

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>

      <groupId>xyz.lament</groupId>
        <artifactId>springehcache-parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
      <packaging>pom</packaging>

      <name>springehcache</name>
      <url>http://lament.xyz</url>

      <properties>
        <!-- Generic properties -->
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

        <!-- Web -->
        <jsp.version>2.2</jsp.version>
        <jstl.version>1.2</jstl.version>
        <servlet.version>3.0.1</servlet.version>

        <!-- Spring -->
        <spring.version>4.3.2.RELEASE</spring.version>

        <!-- Logging -->
        <slf4j.version>1.7.7</slf4j.version>

        <!-- ehcache -->
        <ehcache.version>2.9.0</ehcache.version>

        <!-- jackson -->
        <jackson.version>2.8.0</jackson.version>
      </properties>

      <modules>
	      <module>springehcache-web</module>
	      <module>springehcache-service</module>
      </modules>

    </project>
> web的pom文件配置

    <project xmlns="http://maven.apache.org/POM/4.0.0"    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
      <modelVersion>4.0.0</modelVersion>

      <parent>
         <groupId>xyz.lament</groupId>
         <artifactId>springehcache-parent</artifactId>
         <version>0.0.1-SNAPSHOT</version>
      </parent>

      <artifactId>springehcache-web</artifactId>
      <packaging>war</packaging>

      <name>springehcache-web Maven Webapp</name>
      <url>http://maven.apache.org</url>

      <dependencies>
    	<dependency>
    	  <groupId>xyz.lament</groupId>
    	  <artifactId>springehcache-service</artifactId>
    	  <version>${parent.version}</version>
    	</dependency>
        <dependency>
        	<groupId>org.springframework</groupId>
        	<artifactId>spring-webmvc</artifactId>
        	<version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context-support</artifactId>
       	  <version>${spring.version}</version>
    	</dependency>
        <dependency>
        	<groupId>javax.servlet</groupId>
        	<artifactId>javax.servlet-api</artifactId>
        	<version>${servlet.version}</version>
        	<scope>provided</scope>
        </dependency>

        <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-api</artifactId>
          <version>${slf4j.version}</version>
          <scope>compile</scope>
        </dependency>
      </dependencies>
      <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
          <plugin>
        	<groupId>org.apache.maven.plugins</groupId>
        	<artifactId>maven-war-plugin</artifactId>
        	<version>2.6</version>
    	  </plugin>	   
          <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-compiler-plugin</artifactId>
    		<version>3.1</version>
              <configuration>  
                <source>${java.version}</source>  
                <target>${java.version}</target>  
              </configuration>  
          </plugin>
    	</plugins>
      </build>
    </project>

> service的pom配置

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>

      <parent>
         <groupId>xyz.lament</groupId>
         <artifactId>springehcache-parent</artifactId>
         <version>0.0.1-SNAPSHOT</version>
      </parent>

      <artifactId>springehcache-service</artifactId>
      <packaging>jar</packaging>

      <name>springehcache-service</name>
      <url>http://maven.apache.org</url>

      <dependencies>
        <dependency>
        	<groupId>org.springframework</groupId>
        	<artifactId>spring-context</artifactId>
        	<version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>net.sf.ehcache</groupId>
          <artifactId>ehcache</artifactId>
          <version>${ehcache.version}</version>
        </dependency>
        <dependency>
    	  <groupId>com.fasterxml.jackson.core</groupId>
    	  <artifactId>jackson-databind</artifactId>
    	  <version>${jackson.version}</version>
    	</dependency>
      </dependencies>

      <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
      	  <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-compiler-plugin</artifactId>
    		<version>3.1</version>
              <configuration>
                <source>${java.version}</source>  
                <target>${java.version}</target>  
              </configuration>  
          </plugin>
        </plugins>
      </build>
    </project>

### 配置springMVC

> web.xml中配置sevelet

    <web-app xmlns="http://java.sun.com/xml/ns/javaee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"      
          xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
          version="3.0">

    	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
    	<context-param>
    		<param-name>contextConfigLocation</param-name>
    		<param-value>/WEB-INF/spring/root-context.xml</param-value>
    	</context-param>

    	<!-- Creates the Spring Container shared by all Servlets and Filters -->
    	<listener>
    		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    	</listener>

    	<!-- Processes application requests -->
    	<servlet>
    		<servlet-name>appServlet</servlet-name>
    		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    		<init-param>
    			<param-name>contextConfigLocation</param-name>
    			<param-value>/WEB-INF/spring/servlet/servlet-context.xml</param-value>
    		</init-param>
    		<load-on-startup>1</load-on-startup>
    		<async-supported>true</async-supported>
    	</servlet>

    	<servlet-mapping>
    		<servlet-name>appServlet</servlet-name>
    		<url-pattern>/</url-pattern>
    	</servlet-mapping>

    	<!-- Disables Servlet Container welcome file handling. Needed for compatibility with Servlet 3.0 and Tomcat 7.0 -->
    	<welcome-file-list>
    		<welcome-file></welcome-file>
    	</welcome-file-list>

    </web-app>

> 在servlet配置文件中配置spring的一些基本配置，比如扫描的包，并在此文件中配置cache的management

    <?xml version="1.0" encoding="UTF-8"?>
    <beans:beans xmlns="http://www.springframework.org/schema/mvc"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:beans="http://www.springframework.org/schema/beans"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:cache="http://www.springframework.org/schema/cache"
    	xsi:schemaLocation="
    		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
    		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">

    	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->

    	<context:component-scan base-package="xyz.lament.springehcache" />

    	<!-- Enables the Spring MVC @Controller programming model -->
    	<annotation-driven />

    	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources/ directory -->
    	<resources mapping="/resources/**" location="/resources/" />

    	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
    	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    		<beans:property name="prefix" value="/WEB-INF/views/" />
    		<beans:property name="suffix" value=".jsp" />
    	</beans:bean>

    	<cache:annotation-driven cache-manager="cacheManager"/>
    	<beans:bean id="cacheManagerFactory" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
            <beans:property name="configLocation" value="/WEB-INF/ehcache/ehcache.xml"/>
    	</beans:bean>
    	<beans:bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager">
            <beans:property name="cacheManager" ref="cacheManagerFactory"/>
    	</beans:bean>
    </beans:beans>

> 在ehcache.xml中配置需要用到的缓存名称和策略，比如下面的配置文件中定义了一个名为myCache的缓存  
> 配置自定义缓存        
> 	    name：Cache的唯一标识        
> 	    maxElementsInMemory：缓存中允许创建的最大对象数        
> 	    maxElementsOnDisk：磁盘中最大缓存对象数，若是0表示无穷大        
> 	    eternal：Element是否永久有效，一但设置了，timeout将不起作用，对象永不过期。       
> 	    timeToIdleSeconds：缓存数据的钝化时间，也就是在一个元素消亡之前，两次访问时间的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是 0 就意味着元素可以停顿无穷长的时间。        
> 	    timeToLiveSeconds：缓存数据的生存时间，也就是一个元素从构建到消亡的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是0就意味着元素可以停顿无穷长的时间。        
> 	    overflowToDisk：内存不足时，是否启用磁盘缓存。        
> 	    diskPersistent：是否缓存虚拟机重启期数据
> 	    diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒
> 	    diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区        
> 	    memoryStoreEvictionPolicy：缓存满了之后的淘汰算法。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）   

    <?xml version="1.0" encoding="UTF-8"?>

    <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:noNamespaceSchemaLocation="ehcache.xsd">

        <!-- 缓存位置可以是自定义的硬盘地址也可以是JVM默认使用的缓存地址-->
    	<diskStore path="java.io.tmpdir"/>
        <!--<diskStore path="d:\cache"/> -->

        <defaultCache eternal="false"    
            maxElementsInMemory="10000"   
            overflowToDisk="false"    
            timeToIdleSeconds="0"   
            timeToLiveSeconds="0"    
            memoryStoreEvictionPolicy="LFU"/>   

        <cache name="myCache"    
            eternal="false"    
            maxElementsInMemory="10000"   
            overflowToDisk="false"    
            timeToIdleSeconds="0"   
            timeToLiveSeconds="0"    
            memoryStoreEvictionPolicy="LFU"/>   
    </ehcache>

### 在java代码中配置spring的注解
> 在controller中定义restful接口（为了测试方便，此处使用了restful接口，也可以将@ResponseBody去掉，return一个string，在views目录下建对应的jsp页面，如本项目的hello.jsp）

    @Controller
    @RequestMapping("/springehcache")
    public class UseCacheController
    {
    	@Autowired
    	UseCacheService service;

    	@ResponseBody
    	@RequestMapping(value="/user/{uid}",method=RequestMethod.GET)
        public UserBean getUser(@PathVariable("uid") String uid)
    	{
    		UserBean serviceUser = service.getUser(uid);
    		if(serviceUser != null)
    		{
    			System.out.println("get: " + serviceUser);
    		}
    		else
    		{
    			System.out.println("can't get: " + uid);
    		}
            return serviceUser;
        }
    }

> 在Service中使用spring cache的注解实现对数据的缓存，由于未使用数据库，本项目使用了一个static map来保存数据。

    @Service
    public class UseCacheService
    {
    	public UseCacheService()
    	{
    		UserBean initUser1 = new UserBean();
    		initUser1.setUid("1");
    		initUser1.setName("Natalie Portman");
    		initUser1.setAge(35);
    		UserStore.setUser(initUser1);

    		UserBean initUser2 = new UserBean();
    		initUser2.setUid("2");
    		initUser2.setName("Keira Knightley");
    		initUser2.setAge(31);
    		UserStore.setUser(initUser2);

    		UserBean initUser3 = new UserBean();
    		initUser3.setUid("3");
    		initUser3.setName("Gal Gadot");
    		initUser3.setAge(31);
    		UserStore.setUser(initUser3);
    	}

    	@Cacheable(value="myCache")
    	public UserBean getUser(String uid) {
    		UserBean serviceUser = UserStore.getUser(uid);
    		System.out.println("no cache for this user, read from userStore. user: " + serviceUser);
    		return serviceUser;
    	}
    }

### 在tomcat上运行springehcache-web进行测试
> 获取user  
http://localhost:8080/springehcache-web/springehcache/user/3  
    {
    "uid": "3",
    "name": "Gal Gadot",
    "age": 31
    }

> 新增/修改user
![updateUser](https://raw.githubusercontent.com/LamentOfTheHighborne/images/master/springehcache/updateUser.png)

> 测试缓存是否生效
![systemOut.png](https://raw.githubusercontent.com/LamentOfTheHighborne/images/master/springehcache/systemOut.png)

### 完整的项目代码
[springehcache-parent代码](https://git.oschina.net/LamentOfTheHighborne/springehcache)
