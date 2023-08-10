## WebClient

### Spring WebClient란?

우리가 개발하는 어플리케이션들은 크게 요청자(consumer, subscriber), 제공자(producer, provider) 이렇게 2가지로 나눠볼 수 있다. 요청자는 제공자에게 무언가를 요청할 때 제공자가 공개한 API를 이용한다. 

요청 시 프로그램에서 가장 흔하게 사용하는 것이 HTTP Client이다. Spring WebClient는 웹으로 API를 호출하기 위해 사용되는 Http Client 모듈 중 하나이다. Java에서 가장 많이 사용하는 Http Client는 RestTemplate이다. 그럼 WebClient와 RestTemplate는 어떤 공통점과 차이점이 있을까?

공통점은 둘다 HttpClient 모듈이라는 것이다. 차이점은 통신방법인데, RestTemplate는 Blocking 방식이고, WebClient는 Non-Blocking 방식이다. 

결론은 Spring WebClient를 사용하는 이유는 요청자와 제공자 사이의 통신을 좀 더 효율적인 Non-Blocking 방식으로 하기 위해서이다. 

WebClient는 웹 요청을 수행하기 위한 기본 진입점을 나타내는 인터페이스로 Spring Web Reactive 모듈의 일부로 생성되었으며 이러한 시나리오에서 클래식 RestTemplate를 대체할 것이다. . 

### Spring WebClient의 작동 방식

Spring WebClient의 동작 원리를 이해하기 위해선 RestTemplate의 동작원리를 같이 이해해야 한다. 

- **RestTemplate**
    
    RestTemplate은 Multi-Thread와 Blocking 방식을 사용한다. 
    
    ![다운로드 (2)](https://github.com/isprogrammingfun/TIL/assets/78543382/70059f8e-b346-4b6c-b888-2700a7db11bf)
    
    Thread Pool은 요청자 애플리케이션 구동 시에 미리 만들어둔다.
    
    Request는 먼저 Queue에 쌓이고, 가용한 쓰레드가 있다면 그 쓰레드에 할당되어 처리된다. 즉, 1요청 당 1쓰레드가 할당된다.
    
    각 쓰레드에서는 Blocking 방식으로 처리되어 응답이 올때까지 그 스레드는 다른 요청에 할당 될 수 없다. 
    
    요청을 처리할 쓰레드가 있으면 아무런 문제가 없지만, 쓰레드가 다 차는 경우 이후의 요청은 Queue에 대기하게 된다. 
    
    대부분의 문제는 네트워킹이나 DB와의 통신에서 생기는데, 이런 문제가 여러 쓰레드에서 발생하면 가용한 쓰레드수가 현저하게 줄어들게 되고, 전체 서비스는 매우 느려지게 된다. 
    
- **Spring WebClient**
    
   ![다운로드](https://github.com/isprogrammingfun/TIL/assets/78543382/06d57763-7ec8-442e-9362-85a93592f41b)
    
    Spring WebClient는 single Thread와 Non-Blocking 방식을 사용한다. Core당 1개의 Thread를 이용한다. 
    
    각 요청은 Event Loop내에 Job으로 등록이 된다. Event Loop은 각 Job을 제공자에게 요청한 후, 결과를 기다리지 않고 다른 Job을 처리한다. 
    
    Event Loop는 제공자로부터 callback으로 응답이 오면, 그 결과를 요청자에게 제공한다.
    
    WebClient는 이렇게 이벤트에 반응형으로 동작하도록 설계되었다. 그렇기에 반응성, 탄력성, 가용성, 비동기성을 보장하는 Spring React 프레임워크를 사용하고, React Web 프레임워크인 Spring WebFlux에서 Http Client로 사용된다. 
    
- 성능비교

![다운로드 (3)](https://github.com/isprogrammingfun/TIL/assets/78543382/d6a3d7f4-472f-4b1a-8565-ce708ebf0cf6)

위에 사진은  RestTemplate을 사용하는 Spring Boot1과 WebClient를 사용하는 Spring Boot2의 성능비교 결과이다. 1000명까지는 비슷하지만 동시사용자가 늘수록 RestTemplate은 급격하게 느려지는것을 볼 수 있다.