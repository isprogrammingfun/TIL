## TCP 3-way/4-way handshake

### 3-way handshake

---

TCP는 장치들 사이에 논리적인 접속을 성립(establish) 하기 위하여 three-way handshake를 사용한다. 

이는 연결하고자 하는 두 장치 간의 논리적 접속을 성립하기 위해 사용하는 연결 확인 방식으로, 데이터를 보내기 전에 연결을 확립하기 위해 패킷 요청을 세 번 교환하는 것을 의미한다. 

![images_april_5_post_4c4f039f-64a0-4088-91f2-2caa4aa64adf_3-way 핸드셰이크](https://user-images.githubusercontent.com/78543382/212471931-f2c0e3cd-7bc2-4e4a-bc59-65e431341346.jpeg)

![다운로드](https://user-images.githubusercontent.com/78543382/212471951-f2912ab1-06f8-4cb9-90d5-42e3b0e37ce4.png)


![다운로드 (1)](https://user-images.githubusercontent.com/78543382/212471969-130883e0-1d85-4794-b66f-ae33cf1fee0b.png)


1. 통신을 하려면 서버에게 허가를 받아야 하므로, 먼저 클라이언트에서 서버로 연결 확립 허가를 받기 위한 요청인 SYN을 보낸다. 이때 SYN은 연결 확인을 위해 보내는 무작위의 숫자 값이다.

> SYN의 값에 무작위 수를 사용하는 이유는, Connection을 맺을 때 사용하는 포트는 유한 범위 내에서 사용하고 시간이 지남에 따라 재사용된다. 따라서 두 통신 호스트가 과거에 사용된 포트 번호 쌍을 사용할 가능성이 존재한다. 서버 측에서 패킷의 SYN을 보고 패킷을 구분하게 되는데 난수가 아닌 순차적인 숫자가 전송된다면 이전의 connection으로부터 오는 패킷으로 인식할 수 있어 이러한 문제 발생 가능성을 줄이기 위해 ISN을 무작위 난수로 사용하는 것이다.
> 
1. 서버는 클라이언트가 보낸 요청을 받은 후에 허가한다는 응답을 회신하기 위해 연결 확립 응답인 ACK를 보낸다. 동시에 서버도 클라이언트에게 데이터 전송 허가를 받기 위해 연결 확립 요청 SYN을 보낸다. ACK는 받은 SYN에 1을 더해 SYN을 잘 받았다는 신호로 보낸다. 
2. 서버의 요청을 받은 클라이언트는 서버로 허가한다는 응답으로 연결 확립 응답 ACK를 보낸다. 

3-way handshake는 연결할 때 주고 받는 확인 작업이었다면,  4-way handshake는 연결을 해제할 때 주고 받는 확인작업이다.

### 4-way handshake

---

![다운로드 (2)](https://user-images.githubusercontent.com/78543382/212471986-d7abd27c-2381-4c9b-b2a9-354bf9503522.png)


1. 클라이언트가 연결을 종료하겠다는 FIN 플래그를 전송한다.
2. 서버는 일단 확인 메시지를 보내고 자신의 통신이 끝날 때 까지 기다리는 TIME_WAIT 상태가 된다

> TIME_WAIT이란?
만약 서버에서 FIN을 전송하기 전에 전송한 패킷이 라우팅 지연이나 패킷 유실로 인한 재전송 등으로 인해 FIN 패킷보다 늦게 도착하는 상황이 발생 할 수 있는데, 클라이언트에서 세션을 종료시킨 후 뒤늦게 도착하는 패킷이 있다면 이 패킷은 Drop되고 데이터는 유실될 것이다. 이러한 현상에 대비하여 클라이언트는 서버로부터 FIN을 수신하더라도 일정시간 (디폴트 240초) 동안 세션을 남겨두고 잉여 패킷을 기다리는 과정을 거친다.
> 
1. 서버가 통신이 끝났으면 연결이 종료되었다고 클라이언트에 FIN 플래그를 전송한다. 
2. 클라이언트는 확인했다는 메시지를 보낸다. 

- 패킷: 컴퓨터 네트워크에서 데이터를 주고받을 때 정해 놓은 규칙. 패킷은 사용자 데이터와 제어정보로 이루어진다. 사용자 데이터는 페이로드라고 하고, 제어 정보는 페이로드를 전달하기 위한 정보이다.  이메일을 예시로 들자면 이는 네트워크 패킷을 사용하며 사용자와 수신자 간의 정보를 주고받는데, 전송되는 각 패킷에는 소스 및 대상, 프로토콜 또는 ID와 같은 정보가 포함된다.
- 포트: 포트는 논리적인 접속 장소로 클라이언트 프로그램이 네트워크 상의 특정 서버 프로그램을 지정하는 방법으로 사용된다. 네트워크 상에서 통신을 할 때 IP를 토대로 해당 서버가 있는 컴퓨터에 접근을 한다. 그런데 대부분의 경우 하나의 컴퓨터에는 여러 개의 서버가 실행될 수 있다. 컴퓨터에 여러 개의 서버가 실행되고 있다면, 어느 서버에 접속해야 하는지 컴퓨터에게 알려주어야 하는데 이때 사용되는 것이 포트 번호이다.

### 참고자료

---

[https://mindnet.tistory.com/entry/네트워크-쉽게-이해하기-22편-TCP-3-WayHandshake-4-WayHandshake](https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-22%ED%8E%B8-TCP-3-WayHandshake-4-WayHandshake) 

[https://seongonion.tistory.com/74](https://seongonion.tistory.com/74) 

[https://kongit.tistory.com/entry/네트워크-패킷Network-Packet이란-정의-패킷-손실](https://kongit.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%ED%8C%A8%ED%82%B7Network-Packet%EC%9D%B4%EB%9E%80-%EC%A0%95%EC%9D%98-%ED%8C%A8%ED%82%B7-%EC%86%90%EC%8B%A4) 

[https://study-recording.tistory.com/13](https://study-recording.tistory.com/13)