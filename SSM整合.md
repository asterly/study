# ssM整合环境搭建

​	ssm即spring、springmvc、mybatis的合称，也是javaEE企业级开发框架的轻量级实现，相对于原有的SSH开发人员有了更多的的选择，提升了开发的人员对于框架的使用的自主性。





## 1、配置文件



### 1.1 、spring-servlrt.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
   http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context-3.0.xsd
   http://www.springframework.org/schema/mvc
   http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--<context:component-scan base-package="com.asterary" />-->
    <!-- 1、配置映射器与适配器 -->
    <mvc:annotation-driven >
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
    <!--2 配置自动扫描的包 -->
    <context:component-scan base-package="cn.asterly"/>

    <!-- 4 静态资源默认servlet配置 -->
    <mvc:default-servlet-handler/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/view/" />
        <property name="suffix" value=".jsp" />
    </bean>

</beans>
```



### 1.2、myBatis

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">

	<!-- 读取属性配置 -->
	<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<value>classpath:jdbc.properties</value>
		</property>
	</bean>
	
	<!-- 配置数据源 -->
	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
		<property name="url" value="${url}"></property>
		<property name="username" value="${username}"></property>
		<property name="password" value="${password}"></property>
		<property name="initialSize" value="${initialSize}"></property>
		<property name="maxActive" value="${maxActive}"></property>
		<property name="minIdle" value="${minIdle}"></property>
		<property name="maxWait" value="${maxWait}"></property>
	</bean>
	
	<!-- 配置SqlSessionFactory > SqlSession > getMapper -->
	<bean id="sqlSessionFactory" name="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="typeAliasesPackage" value="cn.asterly.entity"></property>
		<property name="mapperLocations" value="classpath:cn/asterly/dao/*.xml"></property>
	</bean>
	
	<!-- 配置Mapper扫描器 扫描Mapper文件以及对应接口产生实现类的bean -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 指定Mapper类所在的包 -->
		<property name="basePackage" value="cn.asterly.dao"></property>
		<!-- 指定SqlSessionFactory对应的bean名称 -->
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
	</bean>
	
	<!-- 用于处理事务的切面bean 由spring提供的事务管理器 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- 对指定包进行受管组件扫描 -->
	<context:component-scan base-package="cn.asterly.*"/>
	
	<!-- 配置aspectJ 自动产生代理 -->
	<aop:aspectj-autoproxy proxy-target-class="true"/>
	
	<!-- 启用基于注解的声明式事务配置 -->
	<tx:annotation-driven transaction-manager="txManager"/>
    
</beans>
```



###1.3、mybatis-Config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
</configuration>
```



### 1.4 、jdbc.properties

```properties
url=jdbc:mariadb://127.0.0.1:3300/news
username=root
password=ikun2018
initialSize=5
maxActive=200
minIdle=2
maxWait=60000
```



### 1.5、 log4j.properties

```properties
### \u8BBE\u7F6E###
log4j.rootLogger = DEBUG,ERROR,A

###  ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

##
log4j.appender.INFO=org.apache.log4j.ConsoleAppender
log4j.appender.INFO.File=src/info.log
log4j.appender.INFO.Append=true
log4j.appender.INFO.layout=org.apache.log4j.PatternLayout
log4j.apperder.INFO.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%c]-[%p] %m%n
##
#
log4j.appender.A=org.apache.log4j.ConsoleAppender
log4j.appender.A.layout=org.apache.log4j.PatternLayout
log4j.appender.A.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%c]-[%p] %m%n

#
log4j.appender.B=org.apache.log4j.FileAppender
log4j.appender.B.File=/src/log.log
log4j.appender.B.layout=org.apache.log4j.SimpleLayout

#\u8F93\u51FA\u5230E\u76D8\u7684log.html\u6587\u4EF6
log4j.appender.C=org.apache.log4j.RollingFileAppender
log4j.appender.C.File=src/log.html
log4j.appender.C.MaxFileSize=1000KB
log4j.appender.C.MaxBackupIndex=10
log4j.appender.C.layout=org.apache.log4j.HTMLLayout

### ##
log4j.appender.D = org.apache.log4j.ConsoleAppender
log4j.appender.D.File=/src/debug.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

#log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
#log4j.appender.D.File = src/log.log
#log4j.appender.D.Append = true
#log4j.appender.D.Threshold = DEBUG 
#log4j.appender.D.layout = org.apache.log4j.PatternLayout
#log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n


log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File =src/error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

```



### 1.6、generator.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
  <context id="context1">
  <commentGenerator>
  	<property name="suppressAllComments" value="true"/>
  </commentGenerator>
    <jdbcConnection connectionURL="jdbc:mysql://120.79.235.35:3300/news" 
    driverClass="com.mysql.jdbc.Driver" 
    password="lk1234" 
    userId="root" />
    
<!--     生成DTO(配置DTO) -->
    <javaModelGenerator 
    targetPackage="cn.asterly.entity" 
    targetProject="cidida" />
    
<!--     配置映设文件 -->
    <sqlMapGenerator 
		  targetPackage="cn.asterly.dao" 
		  targetProject="cidida" />
		  
<!-- 		  DAO接口生成的目录 -->
    <javaClientGenerator 
    	targetPackage="cn.asterly.dao" 
    	targetProject="cidida" 
    	type="XMLMAPPER" />
    	
    	
<!--     <table  tableName="advert" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--     <table  tableName="comment" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="dept" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="essay" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="files" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="gam" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="grade" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="login" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="manager" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="problem" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="redactor" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="reply" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="share" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="user" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
<!--         <table  tableName="webinfo" -->
<!--     		enableCountByExample="false" -->
<!-- 			enableUpdateByExample="false" -->
<!--     		enableSelectByExample="false" -->
<!-- 			enableDeleteByExample="false" -->
<!--     ></table> -->
        <table  tableName="types"
    		enableCountByExample="false"
			enableUpdateByExample="false"
    		enableSelectByExample="false"
			enableDeleteByExample="false"
    ></table>
  </context>
</generatorConfiguration>

```



## 2、环境搭建

​	项目的环境基于maven搭建，主要包含的模块有API、DAO、MANAGE、SERVICE、MODEL。API主要负责为移动端提供数据接口服务。DAO对应Mapper接口层,MANAGE则是负责前台页面的控制处理，SERVICE对应传统的SERVICE层，提供处理方法，model主要的javaBean的实体类

### 2.1、创建父工程Parent

​	file-->new-->project-->选择maven，选择JDK版本-->填写项目坐标-->填写项目名称和选择项目存储位置。

在pom.xml添加如下代码：

```xml
    <!-- 父工程 标识该项目为一个父项目-->
    <packaging>pom</packaging>
```

​	抽取公共依赖，如版本号，单元测试插件，公共包引入

​	在父项目pom.xml中写入如下代码：

```xml
    <!--公共版本号-->
    <properties>
        <junit.version>4.11</junit.version>
        <!-- spring版本号 -->
        <spring.version>4.3.21.RELEASE</spring.version>
        <!-- mybatis版本号 -->
        <mybatis.version>3.4.6</mybatis.version>
        <mybatis-spring.version>1.3.2</mybatis-spring.version>
        <mybatis-generator-core.version>1.3.7</mybatis-generator-core.version>
        <!-- mysql ,mariadb驱动版本号 -->
        <mysql-driver.version>5.1.35</mysql-driver.version>
        <mariadb-driver.version>2.3.0</mariadb-driver.version>
        <alibaba-druid.version>1.1.10</alibaba-druid.version>
        <!--json工具-->
        <alibaba-fastjson.version>1.2.6</alibaba-fastjson.version>
        <google-gson.version>2.8.5</google-gson.version>
        <!-- log4j日志包版本号 -->
        <slf4j.version>1.7.12</slf4j.version>
        <log4j.version>1.2.17</log4j.version>
    </properties>

<!--公共依赖-->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

```



### 2.2创建子工程API

​	在父工程项目的右键-->new-->new module-->选择maven-->勾选**create form archetype** -->选择maven-archetype-quickstart -->填写坐标和名称-->在父项目子项目名成之间加"**-**"

​	在项目创建后的pom.xml文件中的“<artifactId>cicada-manage</artifactId>”后加入如下代码

```xml
<packaging>jar</packaging>
<!---表示 项目会打包成jar包-->
```



​	manage模块在选择模板时选择maven-archetype-webapp

​	manage需要使用springmvc控制器，在pom.xml文件中引入需要的jar包

```xml
    <!--1.添加spring依赖-->
    <dependencies>
        <!--aop-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--aspects-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--context-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--beans-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--core-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--expression-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--jdbc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--orm-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--tx-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--webmvc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
```



