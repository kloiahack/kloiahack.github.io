---
layout: post
title:  "Spring Boot - static 변수에서 @Value Annotation 사용"
date:   2020-06-04 22:53:21 +0900
categories: springboot java
---

Spring Boot에서는 초기화 과정에서 컴포넌트를 주입할 때, 어플리케이션에 대한 Key/Value 형태의 설정을 클래스 내 변수에 값을 넣어주는 @Value Annotation이 존재한다. 이러한 설정은 application.properties 또는 application.yml 과 같은 파일에서 다음과 같은 형식으로 관리할 수 있다.

예) application.properties
```
application.version = v1.0.2
```
예) application.yml
```
application
    version: v1.0.2
```

이러한 방식을 사용하여 아마존 서비스와 같이 다른 3rd party 서비스를 사용할 때 Access Key 또는 Secret Key 같은 설정을 유용하게 할 수 있다. 또한, Spring Boot는 Profile 별로 설정 파일을 분리하여 관리할 수 있다. 이와 같이 설정 파일에 정의한 값을 사용하기 위하여 Spring Boot에서는 *@Value* annotation 을 제공하고 있다. 

# static 변수에 대하여 @Value Annotation 사용

서론에서 설명한 것과 같이 Spring Boot의 설정 파일에 등록되어 있는 값을 주입된 컴포넌트에서 사용하기 위하여 *@Value* annotation를 사용한다. 클래스 인스턴스 내에서 @Value annotation 으로 decorate 된 변수를 어렵지 않게 사용할 수 있다. 하지만, *static 변수* 에서 다음과 같이 @Value annotation 을 사용한다면 잘못된 결과를 초래할 수 있다.

예) EncryptionUtil.java
```java
@Component
public class EncryptionUtil {

    @Value("${enc.key}"})
    public static String key;

}
```

위와 같이 작성된 EncryptionUtil 클래스 내 정의되어 있는 static 변수 *key*를 사용하려고 *EncryptionUtil.key* 로 접근을 하게 된다면 항상 *null* 이 반환 될 것이다. 이는 static 변수에 대하여 @Value annotation 이 동작하지 않는다. 

이를 해결하기 위해서는 static 이 아닌 setter 메소드를 추가하여 static 변수에 직접적으로 값을 넣을 수 있도록 하면 된다. 

예) 수정된 EncryptionUtil.java
```java
@Component
public class EncryptionUtil {

    public static String key;

    @Value("${enc.key}"})
    public void setKey(String value) {
        key = value;
    }

}
```