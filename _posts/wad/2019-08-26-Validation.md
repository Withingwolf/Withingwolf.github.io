---
layout: post
title: Validation 校验时遇到的问题
date:  2019-08-26 12:54:00 +0900
description: 欢迎
img: post-2.jpg # Add image post (optional)
tags: [Validation,validate]
author: withing
jekyll: true
---

# Validation构建时遇到的问题

在通过ValidatorFactory构建Validation时,出现了如下错误。

```java
javax.validation.ValidationException: HV000183: Unable to initialize 'javax.el.ExpressionFactory'. Check that you have the EL dependencies on the classpath, or use ParameterMessageInterpolator instead
```

### 问题产生原因

ValidatorFactory构建Validation的时候,采用的是工厂方法去搜寻可用的Provider,一般来说使用的是HibernateValidator。在构建的过程中,会使用到tomcat里的EL表达式包。那么在单元测试或者其他不用tomcat启动亦或者使用tomcat6及以下版本启动应用的方式下,就出现了无法实例化ExpressionFactory类的情况。

### 解决方案

1. 在没用使用到EL表达式内容的情况下，更改构建参数，去掉EL包依赖。

   ```java
   import org.hibernate.validator.messageinterpolation.ParameterMessageInterpolator;
   import javax.validation.Validation;
   import javax.validation.Validator;
   import javax.validation.ValidatorFactory;
   
   ValidatorFactory factory = Validation.byDefaultProvider()
                   .configure()
                   .messageInterpolator(new ParameterMessageInterpolator())
                   .buildValidatorFactory();
   ```

   

2. 解决EL依赖相关问题

   + 添加EL依赖包

     ```xml
     <dependency>
         <groupId>javax.el</groupId>
         <artifactId>javax.el-api</artifactId>
         <version>3.0.0</version><!--一定要3.x.x,否则没有ELManager-->
     </dependency>
     <dependency>
        <groupId>org.glassfish.web</groupId>
        <artifactId>javax.el</artifactId>
        <version>2.2.4</version>
     </dependency>
     ```

   + 或者添加tomcat相关依赖

     ```xml
     <dependency>
         <groupId>org.apache.tomcat</groupId>
         <artifactId>tomcat-el-api</artifactId>
         <version>8.5.24</version>
         <scope>test</scope> <!--本人是用于单元测试使用-->
     </dependency>
     <dependency>
         <groupId>org.apache.tomcat</groupId>
         <artifactId>tomcat-jasper-el</artifactId>
         <version>8.5.24</version>
         <scope>test</scope>
     </dependency>
     ```

#### 额外注意:

别显式声明jsp-api依赖,即使要声明也要声明2.2及以上版本，否则会尝试依赖冲突。

``` xml
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.1</version>
    <scope>provided</scope>
</dependency>
<!-- jsp-api依赖包需要改成2.2及以上版本-->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>
```

