---
layout: post
title: "SPRING BOOT - 시작하기"
subtitle: "나는 왜 스프링부트를 사용하는가?"
categories: spring spring-boot
tags: spring spring-boot
description: "Spring boot 시작하기"
comments: true
asset-root: /assets/2016-09-09-spring-boot
---

# Spring Boot - 나는 왜 스프링부트를 사용하는가?

> 관련 자료 위치 :
> http://projects.spring.io/spring-boot/
> http://docs.spring.io/spring-boot/docs/1.4.0.RELEASE/reference/htmlsingle
> Sample Project
> https://github.com/nomimic/spring-boot-sample

## 1. Spring boot 의 간략 설명
스프링부트는 쉽게 단독실행 및 production 등급의 스프링 어플리케이션을 만들고 그냥 실행 할 수 있게 해주는 프레임워크이다. 대부분의 스프링부트 기반의 어플리케이션들은 최소한의 스프링 설정을 필요로 한다.
(스프링부트가 있다고 해서, 쉽게 개발되는건 물론 아니다. 다만 스프링부트는 개발자의 손을 좀더 덜어줄뿐..)

### 주요 기능
* Create stand-alone Spring applications
* Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
* Provide opinionated 'starter' POMs to simplify your Maven configuration
* Automatically configure Spring whenever possible
* Provide production-ready features such as metrics, health checks and externalized configuration
* Absolutely no code generation and no requirement for XML configuration

### Quick Start

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.0.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

```java
package hello;

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@Controller
@EnableAutoConfiguration
public class SampleController {

    @RequestMapping("/")
    @ResponseBody
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleController.class, args);
    }
}

```
## 2. 웹 어플리케이션 개발 시작해보자
``pom.xml`` 에 아래 내용을 추가한다.
```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>1.4.0.RELEASE</version>
			<scope>import</scope>
			<type>pom</type>
		</dependency>
	</dependencies>
</dependencyManagement>
```

``dependencies``에 아래 항목만 추가하면 기본적인 웹 개발 준비는 끝난다.

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<!-- 테스트 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
	</dependency>

</dependencies>
```

### 기본 메인 코드

``nomimic/springboot/starter/Application.java``
```java
package nomimic.springboot.starter;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Created by Lucas Yonghee Lee on 2016. 9. 5..
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}

```
#### 실행
``
$ java -jar ${JAR_FILE}
``

![boot run]({{site.url}}{{page.asset-root}}/spring-boot-run.png){:height="600px"}

### Rest Controller 추가

``nomimic/springboot/starter/reset/model/User.java``
```java
package nomimic.springboot.starter.rest.message;

/**
 * Created by Lucas Yonghee Lee on 2016. 9. 9..
 */
public class User {

    private String name;

    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```



``nomimic/springboot/starter/rest/controller/Users.java``
```java
package nomimic.springboot.starter.rest.controller;

import nomimic.springboot.starter.rest.message.User;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by Lucas Yonghee Lee on 2016. 9. 7..
 */
@RestController
public class Users {

    @GetMapping(name = "/api/users")
    public List<User> getUsers() {
        List<User> users = new ArrayList<User>();

        users.add(new User("KIM",10));
        users.add(new User("LEE",10));

        return users;
    }
}

```

#### 확인
``curl http://localhost:8080/api/users``
```json
[
  {
    "name": "KIM",
    "age": 10
  },
  {
    "name": "LEE",
    "age": 10
  }
]
```

## 3. 어노테이션 설명
### @SpringBootApplication
이 어노테이션은 아래의 항목들을 포함하는 어노테이션이다. 이 중 ``@EnableAutoConfiguration`` 이 스프링부트에서 제공하는 자동 설정 기능과 관련된 어노테이션이다.

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class))

```

### @EnableAutoConfiguration
스프링부트는 어노테이션 존재시 classpath 내에서 META-INF/spring.factories 파일을 로딩 후
org.springframework.boot.autoconfigure.EnableAutoConfiguration에 나열된 클래스들을 이용하여 자동 설정을 진행한다. 스프링부트를 지원하는 라이브러리 등을 제작시 자동 설정 기능을 지원할려면 해당 라이브러리 classpath에 ``META-INF/spring.factories`` 파일을 만들고 아래와 같이 자동 설정 기능을 지원할 클래스명을 기입하면 된다.

``예시) META-INF/spring.factories``
```java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
.....
```

#### 자동 설정 기능 제외

실행 로그에 보면 hiddenHttpMethodFilter, httpPutFormContentFilter 등이 자동 설정 기능에 의해서 로드되어진 것을 볼 수 있다. 이 필터들의 설정은 ``WebMvcAutoConfiguration`` 이 처리한다. 그러므로 이 설정을 제외하고 싶으면 아래와 같이 어노테이션에 추가해주면 간단하게 제외 시킬 수 있다.

```java
@SpringBootApplication(exclude = {WebMvcAutoConfiguration.class})
```


## 4. Production-ready features(spring-boot-actuator)
스프링부트는 개발도 중요하지만, 실제 운영시에 도움이 될만한 기능들(health check, metrics 등)을 제공하며 기본으로 HTTP 기반에서 확인 할 수 있도록 ``Endpoint``들을 제공한다.
``Remote Shell(ssh) 로도 확인 가능하지만 추가적인 작업이 필요함.``

이 기능을 사용할려면 간단하게 ==pom.xml==에 아래와 같이 추가만 하면 된다.
참고(http://docs.spring.io/spring-boot/docs/1.4.0.RELEASE/reference/htmlsingle/#production-ready)
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Endpoint(동작 확인)
스프링부트( 1.4.0 기준)는 여러 endpoint 를 제공하지만 여기서는 간단하게 몇가지 확인해보자.자세한 항목은 레퍼런스 문서를 참고 바란다.

health check ``curl http://localhost:8080/health``
```json
{"status":"UP","diskSpace":{"status":"UP","total":419046809600,"free":238174081024,"threshold":10485760}}
```
metrics ``curl http://localhost:8080/metrics``
```json
{"mem":561082,"mem.free":287116,"processors":8,"instance.uptime":102244,"uptime":107115,"systemload.average":2.11962890625,"heap.committed":486912,"heap.init":262144,"heap.used":199795,"heap":3728384,"nonheap.committed":76032,"nonheap.init":2496,"nonheap.used":74171,"nonheap":0,"threads.peak":15,"threads.daemon":13,"threads.totalStarted":19,"threads":15,"classes":9466,"classes.loaded":9466,"classes.unloaded":0,"gc.ps_scavenge.count":6,"gc.ps_scavenge.time":64,"gc.ps_marksweep.count":2,"gc.ps_marksweep.time":160,"httpsessions.max":-1,"httpsessions.active":0,"gauge.response.health":3.0,"counter.status.200.health":2}
```

### Enpoint 비 활성화 방법
원하지 않는 Enpoint 제거 방법은 간단하게 ``application.properties`` 파일을 classpath: 에 추가한 후 아래와 같이 프로퍼티를 설정해주면 간단하게 제거할 수 있다.
```
endpoints.metrics.enabled=false
```

## 5.Filter 다루기
전통적인 웹 개발시에는 필터를 ``web.xml``에 등록하고 사용을 했을 것이다. 그런데 Spring Boot 에서는 ``web.xml``이 필요하지 않다. 그럼 Spring Boot 에서 어떻게 Filter 들을 다루는지 확인해 보자.

### 간단하게 먼저 Filter 를 만들어 보자.

``nomimic/springboot/starter/filter/MyFilter.java``
```java
package nomimic.springboot.starter.filter;

import javax.servlet.*;
import java.io.IOException;

/**
 * Created by Lucas Yonghee Lee on 2016. 9. 9..
 */
public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("MyFilter : " + request.getRemoteAddr());

        chain.doFilter(request,response);
    }

    @Override
    public void destroy() {

    }
}


```

### Filter 를 등록하자
자바 Config 방식으로 설정을 해보자.
``nomimic/springboot/starter/config/FilterConfig.java``
```java
package nomimic.springboot.starter.config;

import nomimic.springboot.starter.filter.MyFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by Lucas Yonghee Lee on 2016. 9. 9..
 */
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean myFilterBean() {
        FilterRegistrationBean myFilterBean = new FilterRegistrationBean();

        myFilterBean.setFilter(new MyFilter());

        return myFilterBean;
    }
}
```

필터가 동작하는지 확인해보자.

![filter check]({{site.url}}{{page.asset-root}}/filter-check.png){:height="600px"}

정상적으로 필터가 등록되고 콘솔창에서 동작하는지를 확인할 수 있다.

### Filter 순서를 조정하자

FilterConfig 파일에서 ``setOrder``함수를 이용하면 필터의 순서를 조정 할 수 있다.
```java
@Bean
    public FilterRegistrationBean myFilterBean() {
        FilterRegistrationBean myFilterBean = new FilterRegistrationBean();

        myFilterBean.setFilter(new MyFilter());

        myFilterBean.setOrder(Ordered.HIGHEST_PRECEDENCE);

        return myFilterBean;
    }
```
![filter order]({{site.url}}{{page.asset-root}}/filter-order.png){:height="600px"}
