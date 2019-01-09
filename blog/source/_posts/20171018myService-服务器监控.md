---
title: 服务器可视化监控（AdminLTE + Spring BOOT + JPA）
date: 2017-10-18 11:29:32
tags: 从硬件开始搭建自己的服务器
---
<!-- 标签别名 -->
{% cq %} 一声梧叶一声秋，一点芭蕉一点愁，三更归梦三更后。<br>落灯花，棋未收，叹新丰逆旅淹留。枕上十年事，江南二老忧，都到心头。<br/><br/><strong>—— 徐再思《水仙子·夜雨》</strong> {% endcq %}
<!-- more -->
查看服务器一些基本情况，如内存使用率，磁盘占用率等等，都需要分别敲Linux命令来获取，而且数据不能直观显示。既然这样，不如自己造个轮子监控自己的服务器咯。


## CubieBoard ##

服务器环境 Debian（Ubuntu）。
这是我的项目结构。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w65xj67j308b0h4q39.jpg)

## Spring Boot ##
``Spring Boot`` 用起来简单粗暴。
这里先贴上 ``pom.xml``
```
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.7.RELEASE</version>
  </parent>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- 对 JSP 的支持 -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-jasper</artifactId>
      <scope>provided</scope>
    </dependency>

    <!-- 加入这个包后就可以使用 外置 tomcat -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
    </dependency>

    <!-- 数据库 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <!-- druid数据连接池 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.0.18</version>
    </dependency>

    <!-- Json包 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.16</version>
    </dependency>

    <!-- Test -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <!-- 实现热部署 -->
 <!--   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <optional>true</optional>
    </dependency>-->

    <!-- lombok -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.16.10</version>
      <scope>provided</scope>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <!-- 热部署 必须配置 -->
       <!-- <configuration>
          <fork>true</fork>
        </configuration>-->
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
          <useSystemClassLoader>false</useSystemClassLoader>
        </configuration>
      </plugin>
    </plugins>
  </build>

  <profiles>
    <profile>
      <id>java7</id>
      <activation>
        <jdk>7</jdk>
      </activation>
      <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
              <skipTests>true</skipTests>
            </configuration>
          </plugin>
        </plugins>
      </build>
```
在 ``Spring Boot`` 中是不推荐使用Jsp，而是推荐使用模板引擎。

- Thymeleaf(Spring官方推荐)


- FreeMarker


- Velocity


- Groovy


- Mustache

但是个人还是习惯用 Jsp 来开发页面，这就要添加 Jsp 的支持。
在 ``pom.xml`` 中要添加这两个依赖。

```
 <!-- 对 JSP 的支持 -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-jasper</artifactId>
      <scope>provided</scope>
    </dependency>
```

接着在 ``Spring Boot`` 的主函数中要继承 ``SpringBootServletInitializer`` 类并且重写 ``configure`` 方法。

```java

@SpringBootApplication
@EnableScheduling
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {

        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```

为了页面的安全，一般会把 Jsp 页面放到 ``WEB-INF`` 文件内。直接的地址访问是无法访问到 ``WEB-INF`` 内的文件，因此要添加 ``SpringMVC`` 的地址前后缀来获取页面。
在项目的 ``resources`` 文件下新建 ``application.properties`` 配置文件，文件中添加 ``SpringMVC`` 配置，配置文件中还有 ``JPA`` 数据库的配置，一并贴出。

```
spring.mvc.view.prefix: /WEB-INF/views/
spring.mvc.view.suffix: .jsp
application.message: Hello Phil

#server.port=9090

# 数据库访问配置
# 主数据源，默认的
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/monitor
spring.datasource.username=
spring.datasource.password=
spring.datasource.driverClassName=com.mysql.jdbc.Driver

下面为连接池的补充设置，应用到上面所有数据源中
# 初始化大小，最小，最大
spring.datasource.initialSize=5  
spring.datasource.minIdle=5  
spring.datasource.maxActive=20  
# 配置获取连接等待超时的时间
spring.datasource.maxWait=60000  
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.timeBetweenEvictionRunsMillis=60000  
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.minEvictableIdleTimeMillis=300000  
spring.datasource.validationQuery=SELECT 1 FROM DUAL  
spring.datasource.testWhileIdle=true  
spring.datasource.testOnBorrow=false  
spring.datasource.testOnReturn=false  
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.poolPreparedStatements=true  
spring.datasource.maxPoolPreparedStatementPerConnectionSize=20  
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.filters=stat,wall,log4j  
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000  
# 合并多个DruidDataSource的监控数据
#spring.datasource.useGlobalDataSourceStat=true

#JPA Configuration:
spring.jpa.database=MYSQL
# Show or not log for each sql query
spring.jpa.show-sql=false
spring.jpa.generate-ddl=true  
# Hibernate ddl auto (create, create-drop, update)
spring.jpa.hibernate.ddl-auto=update
#spring.jpa.database-platform=org.hibernate.dialect.MySQL5Dialect
spring.jpa.hibernate.naming_strategy=org.hibernate.cfg.ImprovedNamingStrategy  
#spring.jpa.database=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
```

在添加了 Jsp 支持后，如果是用 ``Spring Boot`` 内部自带的 tomcat 启动项目，一定要 ``cd`` 到项目的目录下，然后用下面的命令来启动，否则会报错。
```
mvn spring-boot:run
```
这里有一个支持Jsp的 ``Spring Boot`` 的[案例](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-jsp)。

在服务器中为了方便配置及调试，可以设置外置 tomcat 。只需在 ``pom.xml`` 文件中加上一个依赖即可。

```
 <!-- 加入这个包后就可以使用 外置 tomcat -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
    </dependency>
```

页面难免要访问一些静态资源还有脚本，而 ``Spring Boot`` 都已经做好了映射了... 
在项目的 ``resources`` 下，下面的目录顺序代表着 ``Spring Boot`` 扫描静态资源目录的顺序。
```
META-INF/resources > resources > static > public
```

到这里， ``Spring Boot`` 配置基本完成。
提供一个快速生成  ``Spring Boot`` 项目Demo的[网站](https://start.spring.io/) 。

## JPA ##

看项目结构会发现，没有 DaoImpl 类。贴上 ``ServiceInfoDao`` 说话。

```java
public interface ServiceInfoDao extends JpaRepository<ServiceInfo, String> {

    @Query("from ServiceInfo s where id between (select max(id) from s)-14 and (select max(id) from s)")
    List<ServiceInfo> findNewInfo();

}
```

继承的 ``JpaRepository`` 类中已经帮我们实现了 find ， save 等数据操作，而自定义的方法只需加上 ``Query`` 注解也可实现。这点真的太舒服。

## Druid ##

``Druid`` 据说是现在最好的数据连接池。

|关键功能|Druid|BoneCP|DBCP|C3P0|Proxool|JBoss|
|-------|:---:|:----:|:--:|:--:|:-----:|:---:|
|LRU|是|否|是|否|是|是|
|PSCache|是|是|是|是|否|是|
|PSCache-Oracle-Optimized|是|否|否|否|否|否|
|ExceptionSorter|是|否|否|否|否|是|
|监控|是|否|否|否|是|否|
|扩展|是|否|否|否|否|否|



- LRU
LRU是一个性能关键指标，特别Oracle，每个Connection对应数据库端的一个进程，如果数据库连接池遵从LRU，有助于数据库服务器优化，这是重要的指标。在测试中，Druid、DBCP、Proxool、JBoss是遵守LRU的。BoneCP、C3P0则不是。BoneCP在mock环境下性能可能好，但在真实环境中则就不好了。



- PSCache
PSCache是数据库连接池的关键指标。在Oracle中，类似SELECT NAME FROM USER WHERE ID = ?这样的SQL，启用PSCache和不启用PSCache的性能可能是相差一个数量级的。Proxool是不支持PSCache的数据库连接池，如果你使用Oracle、SQL Server、DB2、Sybase这样支持游标的数据库，那你就完全不用考虑Proxool。



- PSCache-Oracle-Optimized
Oracle 10系列的Driver，如果开启PSCache，会占用大量的内存，必须做特别的处理，启用内部的EnterImplicitCache等方法优化才能够减少内存的占用。这个功能只有DruidDataSource有。如果你使用的是Oracle Jdbc，你应该毫不犹豫采用DruidDataSource。



- ExceptionSorter
ExceptionSorter是一个很重要的容错特性，如果一个连接产生了一个不可恢复的错误，必须立刻从连接池中去掉，否则会连续产生大量错误。这个特性，目前只有JBossDataSource和Druid实现。Druid的实现参考自JBossDataSource。

不多粘贴，配置起来。先添加 ``Druid`` 的依赖。
```
<!-- druid数据连接池 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.0.18</version>
    </dependency>
```

 ``Druid`` 需要配置下面两个类。

``DruidStatViewServlet``
```java
@SuppressWarnings("serial")
@WebServlet(urlPatterns = "/druid/*",
        initParams={
                @WebInitParam(name="allow",value="127.0.0.1"),// IP白名单 (没有配置或者为空，则允许所有访问)
                @WebInitParam(name="deny",value=""),// IP黑名单 (存在共同时，deny优先于allow)
                @WebInitParam(name="loginUsername",value="admin"),// 用户名
                @WebInitParam(name="loginPassword",value="admin"),// 密码
                @WebInitParam(name="resetEnable",value="false")// 禁用HTML页面上的“Reset All”功能
        })
public class DruidStatViewServlet extends StatViewServlet {
}
```

``DruidStatFilter``
```java
@WebFilter(filterName="druidWebStatFilter",urlPatterns="/*",
        initParams={
                @WebInitParam(name="exclusions",value="*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")// 忽略资源
        })
public class DruidStatFilter extends WebStatFilter {
}

```
在 地址上输入 ``${你的项目地址}/druid/index.html`` 即可进入 ``Druid`` 的监控界面。

## AdminLTE ##

[AdminLTE](https://adminlte.io/) 是基于 [Bootstrap](http://www.bootcss.com/) 的前端页面框架。

项目将在[GitHub](https://github.com/lanzm/CubieBoard2)上持续更新维护。