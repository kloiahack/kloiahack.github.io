---
layout: post
title:  "Spring Boot에서 Amazon MQ의 RabbitMQ 연동 (1)"
date:   2021-01-10 23:50:55 +0900
categories: springboot aws rabbitmq java
---

[RabbitMQ](https://www.rabbitmq.com/)는 오픈소스 메세지 브로커 소프트웨어이다. 메세지 브로커를 사용하여 다양한 어플리케이션 간 통신을 지원할 수 있다. 언어와 상관없이 AMQP(Advanced Message Queuing Protocol)을 사용하므로 확장성이 좋다고 할 수 있다. 메세지 브로커는 RabbitMQ 뿐만 아니라 Redis, Apache ActiveMQ 등 다양한 소프트웨어가 존재한다. 각 소프트웨어마다 차이점이 있지만, RabbitMQ의 특징만 정리한다. RabbitMQ는 메세지 전달을 위한 큐를 가지고 있다. 큐는 하나만 구성이 될 수 있으며, 여러개의 큐를 관리할 수 있다. 뿐만 아니라 네임스페이스를 만들어서 큐를 그룹핑 할 수 있다. 토픽이라는 것이 있으며 MQTT(Message Queuing Telemetry Transport)와 같은 방식으로 메세지를 전달할 수 있다. 추후 RabbitMQ 에 대한 내용은 [RabbitMQ Tutorial](https://www.rabbitmq.com/getstarted.html)을 정리할 예정이다.

AWS 에서는 AmazonMQ 제품을 제공하고 있다. AmazonMQ는 Apache ActiveMQ만 지원을 하였지만, 2020년 11월 4일 RabbitMQ를 추가적으로 [지원](https://aws.amazon.com/ko/about-aws/whats-new/2020/11/announcing-amazon-mq-rabbitmq/)하였다. AmazonMQ는 메세지 브로커를 쉽게 구축하고 운영할 수 있도록 해준다. 또한, 높은 확장성과 안정성을 제공한다. 본 글에서는 스프링 부트에서 RabbitMQ를 연동하는 방법을 정리하려고 한다. 

### RabbitMQ 브로커 생성

AWS에 접속하고 [Amazon MQ](https://ap-northeast-2.console.aws.amazon.com/amazon-mq/home?region=ap-northeast-2#/first-run) 페이지로 이동한다. '시작하기' 버튼을 눌러서 브로커를 만들기 위한 페이지로 이동한다. 브로커는 'RabbitMQ'를 선택한다. 배포 모드는 '단일 인스턴스 브로커'와 '클러스터 배포'가 있지만, 지금은 개발/테스트 용도이므로 '단일 인스턴스 브로커'를 선택한다. 다음으로 넘어가면 설정 구성을 할 수 있는데, 브로커 이름은 적당하게 지어주고 인스턴스 유형은 'mq.t3.micro'를 선택한다. 마지막으로 RabbitMQ 액세스를 할 수 있는 사용자 이름과 비밀번호를 입력한다. 이때 입력한 정보는 잘 기억을 하고 있어야 추후에 관리자 페이지 접속 및 브로커 서버에 접속이 가능하다. 추가적인 설정을 보면 기본적으로 액세스 유형이 '퍼블릭 액세스'로 설정되어 있다. 추후 실제 운영을 하기 위해서는 VPC 내에서만 접속이 가능한 '프라이빗 액세스'로 만드는 것이 좋다. 이와 같이 설정 후 생성하는데 약 5~10분 정도 시간 소요된다. 이때 커피 한 잔 내려오면 적절하다.

![image](/assets/imgs/Pasted image 20210110224507.png)

브로커가 정상적으로 생성이 됐을 경우에는 위와 같은 화면을 볼 수 있을 것이다. 'RabbitMQ 웹 콘솔' URL을 클릭하게 되면, RabbitMQ 관리자 페이지에 접속할 수 있다. 접속 정보는 브로커 생성 시 입력한 사용자 이름과 비밀번호이다. 

### 스프링 부트와 RabbitMQ 연동

기존 스프링 부트 프로젝트가 없는 경우 [Spring initializr](https://start.spring.io)에서 프로젝트를 쉽게 생성할 수 있다. 

![image](/assets/imgs/Pasted image 20210110225415.png)

기존 프로젝트가 있는 경우에는 다음과 같이 build.gradle 파일에서의 dependencies 를 추가한다. 

```gradle
implementation 'org.springframework.boot:spring-boot-starter-amqp'
```

또한, 내부적으로 jackson 라이브러리를 사용하여 json을 오브젝트로 변환하는 기능을 사용하려고 한다. 다음과 같이 build.gradle 파일에 jackson에서 제공하는 databind 라이브러리도 추가한다. 


```gradle
implementation 'com.fasterxml.jackson.core:jackson-databind'
```

본인이 사용하는 에디터를 가지고 스프링 부트 프로젝트를 연다. 다음과 같이 application.yml 에서 RabbitMQ 접속 정보를 입력한다. application.properties는 해당하는 파일 양식으로 작성하면 된다.

```yaml
spring:  
 rabbitmq:  
    host: <Your Host>
    port: 5671  
	username: <Your UserName>  
    password: <Your Password>
    virtual-host: /vesta
    ssl:  
      enabled: true
 listener:  
      simple:  
        acknowledge-mode: manual
```

위에서 rabbitmq 필드의 host, username, password는 브로커 생성 시 입력했던 사용자 정보와 비밀번호를 입력하면 된다. listener에서 acknowledge-mode가 manual인데, 스프링 부트에서 RabbitMQ Listener에서 메세지를 정상적으로 처리가 됐다는 ack을 수동으로 보내겠다는 의미이다. 이것에 대한 내용은 뒤에 설명한다. 기본적으로 Amazon MQ에서는 SSL을 지원하는 브로커로 생성된다. 따라서, ssl.enabled 필드를 true 로 설정해야 한다. 이를 설정하지 않는 경우 브로커 접속이 되지 않으며, 다음과 같은 오류메세지가 발생한다. 

```
org.springframework.amqp.AmqpIOException: java.io.IOException
	at org.springframework.amqp.rabbit.support.RabbitExceptionTranslator.convertRabbitAccessException(RabbitExceptionTranslator.java:70) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.connection.AbstractConnectionFactory.createBareConnection(AbstractConnectionFactory.java:602) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory.createConnection(CachingConnectionFactory.java:723) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.connection.ConnectionFactoryUtils.createConnection(ConnectionFactoryUtils.java:214) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.core.RabbitTemplate.doExecute(RabbitTemplate.java:2128) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.core.RabbitTemplate.execute(RabbitTemplate.java:2101) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.core.RabbitTemplate.execute(RabbitTemplate.java:2081) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.core.RabbitAdmin.getQueueInfo(RabbitAdmin.java:407) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.core.RabbitAdmin.getQueueProperties(RabbitAdmin.java:391) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.attemptDeclarations(AbstractMessageListenerContainer.java:1883) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.redeclareElementsIfNecessary(AbstractMessageListenerContainer.java:1864) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.initialize(SimpleMessageListenerContainer.java:1345) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:1191) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]
Caused by: java.io.IOException: null
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:129) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:125) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.impl.AMQConnection.start(AMQConnection.java:396) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:1139) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:1087) ~[amqp-client-5.10.0.jar:5.10.0]
	at org.springframework.amqp.rabbit.connection.AbstractConnectionFactory.connectAddresses(AbstractConnectionFactory.java:638) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.connection.AbstractConnectionFactory.connect(AbstractConnectionFactory.java:613) ~[spring-rabbit-2.3.2.jar:2.3.2]
	at org.springframework.amqp.rabbit.connection.AbstractConnectionFactory.createBareConnection(AbstractConnectionFactory.java:565) ~[spring-rabbit-2.3.2.jar:2.3.2]
	... 12 common frames omitted
Caused by: com.rabbitmq.client.ShutdownSignalException: connection error
	at com.rabbitmq.utility.ValueOrException.getValue(ValueOrException.java:66) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.utility.BlockingValueOrException.uninterruptibleGetValue(BlockingValueOrException.java:36) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.impl.AMQChannel$BlockingRpcContinuation.getReply(AMQChannel.java:502) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.impl.AMQConnection.start(AMQConnection.java:326) ~[amqp-client-5.10.0.jar:5.10.0]
	... 17 common frames omitted
Caused by: java.net.SocketException: Connection reset
	at java.base/java.net.SocketInputStream.read(SocketInputStream.java:186) ~[na:na]
	at java.base/java.net.SocketInputStream.read(SocketInputStream.java:140) ~[na:na]
	at java.base/java.io.BufferedInputStream.fill(BufferedInputStream.java:252) ~[na:na]
	at java.base/java.io.BufferedInputStream.read(BufferedInputStream.java:271) ~[na:na]
	at java.base/java.io.DataInputStream.readUnsignedByte(DataInputStream.java:293) ~[na:na]
	at com.rabbitmq.client.impl.Frame.readFrom(Frame.java:91) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.impl.SocketFrameHandler.readFrame(SocketFrameHandler.java:184) ~[amqp-client-5.10.0.jar:5.10.0]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:665) ~[amqp-client-5.10.0.jar:5.10.0]
	... 1 common frames omitted
```

이제 스프링 부트에서 RabbitMQ을 연결하기 위한 환경을 모두 완료 하였다. 이제 실질적으로 코드를 작성해서 RabbitMQ 에서 사용할 Queue를 만들고 토픽에 대하여 큐를 연동하기 위한 Exchange 바인딩 과정을 거칠 것이다. 다음과 같이 RabbitConfiguration.java 파일을 생성한다

```java
@Configuration  
public class RabbitConfiguration {  
  
    public static final String queueName = "default";  
	public static final String topicExchangeName = "test-exchange";  
  
	@Bean  
	Queue defaultQueue() {  
		return new Queue(queueName, false);  
	}  

	@Bean  
	TopicExchange exchange() {  
		return new TopicExchange(topicExchangeName);  
	}  
  
    @Bean  
	Binding bindingDefault(Queue queue, TopicExchange exchange) {  
		return BindingBuilder.bind(queue).to(exchange).with("default.#");  
	}  
  
    @Bean  
	RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {  
		RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);  
		rabbitTemplate.setMessageConverter(Jackson2JsonMessageConverter());  
		return rabbitTemplate;  
	}  
  
	@Bean  
	public Jackson2JsonMessageConverter Jackson2JsonMessageConverter(){  
		return new Jackson2JsonMessageConverter();  
	}  
  
}
```

큐의 이름은 'default'이고 exchange 이름은 'test-exchange' 이다. 'default.#'은 'default.\<any\>' 로 이해하면 되며, 해당 토픽으로 들어오는 메세지는 모두 'default' 큐에 넣겠다는 의미이다. 이것에 대한 자세한 설명은 추후 RabbitMQ 튜토리얼 정리에서 할 예정이다. 자세한 것은 RabbitMQ 튜토리얼에서 보면 아주 좋다. 메세지는 json 포맷으로 오는 것을 가정하고 jackson 라이브러리를 통해서 오브젝트로 변환한다. 

마지막으로 DefaultListener.java 파일을 다음과 같이 작성한다. 
	
```java
@Component  
@RabbitListener(queues = RabbitConfiguration.queueName)  
public class DefaultListener {  
  
	@RabbitHandler  
	public void receiveMessage(LinkedHashMap message, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long tag) {  
		try {  
			System.out.println(message);  
			channel.basicAck(tag, false);  
		} catch (Exception e) {  
			e.printStackTrace();  
		}  
	}  
  
}
```

RabbitListener 어노테이션을 통해서 DefaultListener 클래스는 RabbitMQ의 리스너라는 것을 명시한다. 큐의 이름은 RabbitConfiguration 내에서 정의한 'default' 이다. 해당 어노테이션에서는 큐를 동시다발적으로 처리할 수 있는 옵션이 있는데, 이것은 추후 정리 예정이다. 

RabbitHandler 어노테이션을 메소드에 달아주면, RabbitListener에 명시한 큐의 데이터를 처리할 수 있다. 첫 번째 인자는 들어오는 메세지인데, 현재 LinkedHashMap으로 받아들인다. Jackson 라이브러리가 json 데이터를 오브젝트로 변경해주는데, 이를 표현하는 방식이 LinkedHashMap이다. 클래스로 변환도 가능하지만, 이것은 추가적인 작업이 필요하다. 두 번째 인자로는 채널, 세 번째 인자로는 태그이다. 내부에서 처리하는 코드를 보면, channel.basicAck 라고 메세지를 정상적으로 받았다는 ack를 수동으로 보내는 것을 확인할 수 있다. 이렇게 처리하는 이유는 내부적인 exception 발생 시 메세지를 다시 처리 하기 위한 수단이다. Exception이 발생하여 메세지에 대한 처리가 비정상적으로 끝났다면, 서비스 정상화 후 같은 메세지를 처리할 필요가 있다. 이런 것을 컨트롤하기 위하여 수동적으로 ack 메세지를 보낸다. 

이와 같이 작성 후 Spring Boot 어플리케이션을 실행해보자!

### 테스트 

Amazon MQ 페이지에 접속 후 본인이 만든 브로커를 선택한다. 하단에 'RabbitMQ 웹 콘솔'에 있는 URL에 접속하여 본인이 입력한 아이디 및 패스워드를 입력한다. 'Queues' 탭으로 이동한다.

성공적으로 스프링부트가 구동 됐을 경우 Virtual Host에는 /, Name은 default 로 큐가 생성이 되어 있을 것이다. default 큐의 상세 페이지로 이동한다. Publish Message 란에 다음과 같이 입력 후 'Publish message' 버튼을 눌러본다.

![image](/assets/imgs/Pasted image 20210110232951.png)

위와 같은 과정을 통해서 성공적으로 메세지를 보냈을 경우, 다음과 같이 Spring Boot 어플리케이션 로그가 찍히는 것을 확인할 수 있다. 

![image](/assets/imgs/Pasted image 20210110233047.png)

### 결론 

AWS에서 제공하는 Amazon MQ 제품을 통해 RabbitMQ 브로커 인스턴스를 생성하는 법을 알아보았으며, Spring Boot에서 RabbitMQ 연동을 하고 메세지를 받는 것을 테스트 해보았다. 현재는 메세지를 HashMap으로 받는 법만 정리 하였지만, 다음에는 Class 변환하는 법과 메세지를 전송하는 법에 대하여 정리할 예정이다. 본 글은 시리즈 글로 진행된다. 작성된 코드는 [Github](https://github.com/jookwang-park/spring-boot-rabbitmq-lecture)에서 확인 가능하다.