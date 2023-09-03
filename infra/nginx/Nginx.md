## Nginx

### Nginx의 내부 동작 매커니즘

**Master Process, Worker Process**

Nginx는 어떤 구조를 가지고 있어서 많은 동시 커넥션을 유지할 수 있을까? 비결은 바로 적게 생성되는 프로세스 수에 있다.

Nginx에는 Master Process와 Worker Process라는 것이 있다.

![image1](https://github.com/isprogrammingfun/TIL/assets/78543382/c7de9438-d73f-4600-818e-94bb7523be2b)

Master Process : Nginx.conf 라는 설정파일에 작성한 내용을 읽고, 설정에 알맞게 Worker Process를 생성하는 프로세스

Worker Process : 실제로 일을 하는 프로세스

Nginx는 하나의 Master Process와 다수의 Worker Process로 구성되어 실행됩니다. Master Process는 설정 파일을 읽고, 유효성을 검사한다. 그리고 Worker Process를 관리한다. 모든 요청은 Worker Process에서 처리한다. Nginx는 이벤트 기반 모델을 사용하고, Worker Process 사이에 요청을 효율적으로 분배하기 위해 OS에 의존적인 메커니즘을 사용한다. Worker Process의 개수는 설정 파일에서 정의되며, 정의된 프로세스 개수와 사용 가능한 CPU 코어 숫자에 맞게 자동으로 조정된다.

Worker 프로세스가 만들어질 때 각자 지정된 Listen Socket을 지정받는다. 그리고 그 소캣에 새로운 클라이언트로 부터 받은 요청이 들어오면 커넥션을 생성하고, 그 요청을 처리한다. 그리고 연결된 커넥션은 HTTP 프로토콜에 지정된 keep-alive 시간만큼 끊기지않고 유지된다. 그런데 커넥션이 생성되었다고 해서 Worker 프로세스가 해당 커넥션 하나만 담당하지는 않는다.

**Event 기반 모델 (Event Driven)**

Event ? → Worker 프로세스는 생성된 커넥션에 현재 아무런 요청이 없는 상태일 때 새로운 커넥션이 들어오면 또 연결을 가능하게 한다. 또는 이미 연결된 다른 커넥션이 요청이 시도한 경우, 해당 요청에 대해 처리를 한다. 이렇게 커넥션의 생성 및 제거, 그리고 요청을 처리하는 행위들 모두 Event 라고 부른다. 

발생한 Event는 비동기적으로 처리한다. 이런 Event 들은 OS 커널이 큐(Queue) 형태로 Worker 프로세스에 전달해준다. 이런 이벤트들은 큐에 담긴 상태에서 Worker 프로세스가 처리하기 전까지는 비동기 방식이 되게 한다. 즉, 한 이벤트가 완전히 처리되고 난 후에 큐의 그 다음 이벤트를 꺼내서 그 다음것을 순차적으로 처리하는 방식이다. 그렇기 때문에 한 이벤트가 처리되기 전까지 다른 이벤트들은 동시에 처리되는 것이 아니라, 계속 대기하고 있어야 한다. 이때 Worker 프로세스는 하나의 쓰레드로 이벤트를 순차적으로 꺼내서 처리해나가기 때문에 쉬지 않고 계속해서 일을 한다.

간단히 정리해보자면, Nginx는 멀티 프로세스 싱글 스레드 방식으로 동작하고, 비동기 이벤트 방식으로 동작해서 더 적은 메모리로 운영할 수 있다. 그리고 아파치 서버와 같이 요청이 들어올 때 마다 프로세스와 스레드를 생성하지 않고, 고정된 개수의 프로세스로 처리하기 때문에 프로세스와 스레드에 대한 생성 및 제거 비용이 들지 않는다. 이러한 이유로 요청이 많아져도 아파치 서버에 비해 적절한 메모리 및 CPU 사용을 유지할 수 있다. 

### Nginx의 Thread Pool : 비동기 방식의 해결방안

위에서 말했던 이벤트를 하나씩 순차적으로 처리하면 문제점이 발생한다. 만약 이벤트 처리 시간이 길어져 단일 Thread를 점유하고 있으면, 다른 Event 처리도 늦어지는 Blocking 현상이 발생하는 것이다. 쉽게 말하면 요청을 비동기 방식으로 차근차근 처리해나가면 처리 시간이 오래걸릴 수 있다. 문제가 발생할 수 있는 더 다양한 케이스는 다음과 같다.

- 이벤트 큐 중에서 시간이 오래 걸리는 작업을 수행해야 할 이벤트 ( 디스트에 읽고 쓰는 작업 같은 경우 ) 가 존재할 때
- 동기 방식으로 DB에서 응답을 받는 경우
- Mutex, 또는 라이브러리 함수를 호출하는 경우

위와 같은 요청들을 처리하는 동안 Worker Process는 다른 작업을 수행할 수 없으므로, 이벤트를 처리할 수 없기에 성능이 매우 저하된다는 점이 발생한다. 그래서 Nginx는 Thread Pool 개념을 도입했다.

![image2](https://github.com/isprogrammingfun/TIL/assets/78543382/e6b9c532-e390-4172-a7a0-1831ca90154d)

Worker Process가 요청을 큐에 넣으면, 각 요청들을 Thread Pool에 있는 Thread 들이 골고루 수행하고 수행 결과를 다시 Worker Process에 전달해주는 방식이다. 

예를들어 디스크에 읽고쓰는 작업을 해야한다면 그 뒤에 있는 이벤트는 요청을 처리하는 긴 시간동안 블로킹(blocking) 되는 상태이다. Nginx 는 이런 상황을 방지하기 위해 그렇게 시간이 오래걸리는 작업을 따로 수행하는 쓰레드 풀을 만듭니다. 그리고 worker 프로세스는 지금 처리할 요청이 시간이 오래걸릴 것 같다 싶으면 해당 쓰레드 풀에게 그 이벤트를 위임하고, 큐 안에 있는 다른 이벤트를 처리한다. Thread pool 개념을 도입했다고 해서 디폴트로는 적용되지 않는다. 1.7.11 버전 이상에서부터는 별도의 설정이 필요하다.

### Nginx의 특징

**Context Switching**

![image3](https://github.com/isprogrammingfun/TIL/assets/78543382/781f103a-0047-43eb-bcc8-0f6b75b696d4)

보통 Nginx는 코어 개수만큼의 Worker Process를 만든다. 그리고 Worker Process가 하나의 코어만을 이용하도록 할당한다. CPU 에선 코어가 담당하는 프로세스를 변경하는 컨텍스트 스위칭을 거치지 않아도 되고, 그만큼 CPU 부하가 감소한다. 

**동적인 설정 변경**

![image4](https://github.com/isprogrammingfun/TIL/assets/78543382/3873dd61-bc82-4f36-a1b4-9a2cb821ba53)
이렇게 수많은 동시 커넥션 양을 처리하는데에 있어 프로세스를 적게 만들다보니 가볍기까지 하다. 프로세스를 적게 만드는 이 구조는 Nginx 의 설정을 동적으로 서버를 중단하지 않고도 바꾸는 것을 가능하게 한다. 보통 서버 프로그램들은 설정을 하고 새로운 설정을 적용하기 위해 재부팅 해야 하는 경우가 많다. 그러나 Nginx는 Event Driven 이라는 특정 덕분에 서버가 돌고 있는 도중에도 동적으로 설정 변경이 가능하다. 이것을 service reload 라고 한다. 

개발자가 nginx.conf 라는 설정파일에서 Nginx에 대한 설정을 변경하고, Nginx에 대한 설정을 적용하면 즉, service reload를 하면 Master 프로세스는 그 설정에 맞는 Worker Process를 따로 생성한다.

그리고 기존에 있는 Worker Process가 더 이상 커넥션을 생성하지 않도록 합니다. 즉, 새롭게 이벤트를 할당하지 않는다. 기존 Worker Process (초록색) 가 모든 커넥션을 처리하고 나면, 새로운 Worker Process (노란색)로만 이벤트를 전달하고 기존의 Worker Process를 종료한다.

**로드밸런싱**

위에서 설명한 동적 설정 변경은 언제 사용할까? 아주 대표적인 경우로, Nginx가 여러 동시 커넥션을 관리하는 도중에 뒷단에 서버가 추가되는 경우가 있다. 그때는 Nginx가 로드 밸런서의 역할을 담당하게 된다. 로드 밸런서는 요청을 여러 서버로 분산하는 작업을 수행한다. 

### Nginx의 기타 기능

**SSL 터미네이션 기능 (Reverse Proxy일 때)**

SSL 터미네이션이란 Nginx가 클라이언트와는 https 통신을하고, 서버와는 http 통신을 하는것을 말한다. 이 구조를 만들어서 서버가 복구화 과정을 감당하지 않을 수 있도록 하는것이다. 비즈니스 로직 처리에 리소스를 사용할 수 있도록 비용을 줄여준다.

**캐싱**

Nginx는 http 프로토콜을 사용해서 전달하는 컨텐츠를 캐싱할 수 있다. 

### Nginx와 Apache의 강점

**Nginx**

아파치 서버도 성능 개선을 많이 하는 중이나, 동시 커넥션 관리에선 Nginx 가 크게 앞서는 중 이다. Nginx 는 동시 커넥션 수가 많아져도 메모리 사용률이 낮고 일정한 비율이 나오는 반면, 아파치는 많이 사용한다.

**Apache**

지금껏 오랜기간 버그가 발생되지 않도록 수많은 개선이 이루어졌기 때문에 다양한 OS 에서 안정적이라는 특징을 지닌다. 또한 아파치는 모듈을 추가해서 확장하기 쉽다는 특징도 지닌다. 모듈의 종류도 아파치 서버가 Nginx 보다 훨씬 많다.

### 참고자료

[https://velog.io/@msung99/Nginx-1995년-역사부터-뜯어보는-Nginx-등장배경부터-내부-메커니즘까지](https://velog.io/@msung99/Nginx-1995%EB%85%84-%EC%97%AD%EC%82%AC%EB%B6%80%ED%84%B0-%EB%9C%AF%EC%96%B4%EB%B3%B4%EB%8A%94-Nginx-%EB%93%B1%EC%9E%A5%EB%B0%B0%EA%B2%BD%EB%B6%80%ED%84%B0-%EB%82%B4%EB%B6%80-%EB%A9%94%EC%BB%A4%EB%8B%88%EC%A6%98%EA%B9%8C%EC%A7%80)

[https://velog.io/@devjooj/Server-Ngnix-왜-사용할까](https://velog.io/@devjooj/Server-Ngnix-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C)

https://icarus8050.tistory.com/57

https://sihyung92.oopy.io/server/nginx_feat_apache

https://sihyung92.oopy.io/server/nginx_feat_apache