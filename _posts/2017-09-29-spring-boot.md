---
layout: post
layout: post
title: "SPRING BOOT - SpringApplication"
subtitle: "SpringApplication 동작 이해"
categories: spring spring-boot
tags: spring spring-boot
description: "Spring-boot의 SpringApplication 동작 이해"
comments: true
toc: true
asset-root: /assets/2016-09-09-spring-boot
---


# SpringApplication 동작 이해
문서 작성 기준 버전 `spring-boot version : 1.5.2.RELEASE`

## SpringApplication 생성 및 실행 방법
* SpringApplicationBuilder 통해서 생성하게 되면 세밀한 설정이 가능하다.
```java
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		//빌더를 이용한 방법
		SpringApplicationBuilder applicationBuilder = new SpringApplicationBuilder(Application.class);
		SpringApplication application = applicationBuilder
				.build();
		application.run(args);

		//static 메소드를 이용한 방법
		SpringApplication.run(Application.class,args);
	}
}
```

## 단계별 설명
### 초기화 단계
* SpringApplication 객체가 생성되는 시점
* 웹 환경인지 여부 조사
  - 하기 클래스들이 클래스 경로상에 존재하는지 확인
```java
  javax.servlet.Servlet
  org.springframework.web.context.ConfigurableWebApplicationContext
```
* ApplicationContextInitializer 설정
  - 현재 클래스 로더에서 모든 `META-INF/spring.factories` 파일들에서
  - `ApplicationContextInitializer` 클래스 경로키로 설정된 클래스들을 인스턴스화 한다
  - spring-boot.jar 파일 안에 있는 `META-INF/spring.factories` 파일의 내용 일부분
```yml
# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.context.embedded.ServerPortInfoApplicationContextInitializer
```
  - 관련 스프링 코드
```java
SpringFactoriesLoader.loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader)
```

* ApplicationListener 설정
  - Initializer 설정하는 동작 원리와 같다.

### 실행 단계
* SpringApplication.run 메소드 코드
```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		FailureAnalyzers analyzers = null;
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args); //#1
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext(); // #2
			analyzers = new FailureAnalyzers(context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner); // #3
			refreshContext(context); // #4
			afterRefresh(context, applicationArguments); // #5
			listeners.finished(context, null);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			return context;
		}
		catch (Throwable ex) {
			handleRunFailure(context, listeners, analyzers, ex);
			throw new IllegalStateException(ex);
		}
	}
```
* SpringApplication 실행 단계는 SpringApplicationRunListener 가 제공하는 기능 단위와 일치한다

```java
/**
	 * Called immediately when the run method has first started. Can be used for very
	 * early initialization.
	 */
	void starting();

	/**
	 * Called once the environment has been prepared, but before the
	 * {@link ApplicationContext} has been created.
	 * @param environment the environment
	 */
	void environmentPrepared(ConfigurableEnvironment environment);

	/**
	 * Called once the {@link ApplicationContext} has been created and prepared, but
	 * before sources have been loaded.
	 * @param context the application context
	 */
	void contextPrepared(ConfigurableApplicationContext context);

	/**
	 * Called once the application context has been loaded but before it has been
	 * refreshed.
	 * @param context the application context
	 */
	void contextLoaded(ConfigurableApplicationContext context);

	/**
	 * Called immediately before the run method finishes.
	 * @param context the application context or null if a failure occurred before the
	 * context was created
	 * @param exception any run exception or null if run completed successfully.
	 */
	void finished(ConfigurableApplicationContext context, Throwable exception);
```

#### #1 SpringApplicationRunListener 생성
* `초기화 단계`에서 설정한 ApplicationListener 등록 처럼 `SpringApplicationRunListener.class` 타입으로 지정된 클래스들을 인스턴스화 한다

#### #2 ApplicationContext 생성
* 기본적으로 초기화 단계에서 웹 환경인지 체크한 결과에 따라 아래 둘 중 한개로 생성한다.
  - org.springframework.context.annotation.AnnotationConfigApplicationContext
  - org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext(WEB 환경)
* ConfigurableApplicationContext 상속한 클래스 중 하나를 별도로 설정할 수 있다

#### #3 prepareContext
* ApplicationContextInitializer 들의 초기화 메소드 호출
* SpringApplicationRunListener.contextPrepared 호출
* SpringBoot 전용 하기 Bean생성
  - "springApplicationArguments","springBootBanner"
* BeanDefinitionLoader 생성 후 load() 함수 호출
* SpringApplicationRunListener.contextLoaded 호출

#### #4 refreshContext(개인적으로 가장 어려운 부분)
* 실제 Bean 생성이 되는 부분
  - 정확하게 알지는 못하지만 BeanDefinitionLoader를 통해서 BeanDefinition 생성
  - BeanDefinition 에 있는 Bean 생성을 위한 정보를 토대로 Bean을 생성함

#### #5 afterRefresh
* 생성된 ApplicationContext에 생성된 Bean 중 아래 타입에 해당되는 Bean을 획득 후 실행
  - ApplicationRunner.class
  - CommandLineRunner.class
* Runner Bean 들은 지정된 순서에 의해 실행된다

## Spring Boot (SpringApplication)지원 라이브러리 또는 어플리케이션 개발 방법

* `META-INF/spring.factories` 파일을 생성하고 `ApplicationContextInitializer`,`ApplicationListener` 또는 `EnableAutoConfiguration`관련 팩토리 클래스들을 나열함으로 중간중간 `SpringApplication`이 동작하는 과정에 개입을 할 수 있다.
