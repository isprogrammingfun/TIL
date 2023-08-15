## HTTP HTTPS 차이


### HTTP (Hyper Text Transfer Protocol)

---

HTTP란 서버/클라이언트 모델을 따라 데이터를 주고 받기 위한 프로토콜이다. 즉 HTTP는 인터넷에서 하이퍼텍스트를 교환하기 위한 통신 규약으로, 80번 포트를 사용하고 있다. 따라서 HTTP 서버가 80번 포트에서 요청을 기다리고 있으며, 클라이언트는 80번 포트로 요청을 보내게 된다. HTTP는 애플리케이션 레벨의 프로토콜로 TCP/IP위에서 작동한다. HTTP는 상태를 가지고 있지 않는 Stateless 프로토콜이며 Method, Path, Version, Headers. Body 등으로 구성된다. 하지만 HTTP는 암호화가 되지 않은 평문 데이터를 전송하는 프로토콜이였기 때문에, HTTP로 중요 정보를 보낼 때 제 3자가 중간에 패킷을 가로채 수정할 수 있었고, 정보를 조회할 수 있었다. 이런 보안적인 문제로 인해 HTTPS가 나왔다. 

### HTTPS (Hyper Text Transfer Protocol Secure)

---

HTTPS는 SSL (Secure Socket Layer) 인증서를 사용하는 HTTP이다. SSL 인증서는 일반 HTTP 요청 및 응답을 암호화한다. 즉 HTTPS는 HTTP보다 더 보안적으로 안전한 프로토콜이다. HTTPS를 사용한 웹 페이지를 통해 전송되는 모든 데이터는 추가적인 보안 계층이 있다. 이를 TLS 프로토콜이라고 한다. 모든 유형의 데이터는 변경되거나 손상될 수 없는 HTTPS 사이트를 통해 전달되며 제 3자로부터 보호된다. HTTPS 보호 기능은 도메인 이름 앞에 자물쇠 아이콘으로 확인 할 수 있다. HTTPS는 HTTP와 다르게 443포트를 사용한다. 

### HTTPS 암호화

---

HTTPS는 대칭키 암호화 방식과 비대칭키 암호화 방식 모두 사용하고 있다

- 대칭키 암호화
    - 클라이언트와 서버가 동일한 키를 사용해 암호화/복호화를 진행
    - 키가 노출되면 매무 위험하지만 연산 속도가 빠름
- 비대칭키 암호화
    - 1개의 쌍으로 구성된 공개키와 개인키를 암호화/복호화 하는데 사용
    - 단 하나의 공개키를 사용함으로써 모든 수신자와 개별 키를 만들 필요가 없음
    - 키가 노출되어도 비교적 안전하지만 연산 속도가 느림

암호화를 공개키로 하느냐 개인키로 하느냐에 따라 얻는 효과가 다른데, 공개키와 개인키로 암호화한 효과로는

- 공개키 암호화: 공개키로 암호화를 하면 개인키로만 복호화할 수 있다. 개인키는 나만 가지고 있으므로, 나만 볼 수 있다.
- 개인키 암호화: 개인키로 암호화하면 공개키로만 복호화 할 수 있다. 공개키는 모두에게 공개되어 있으므로 내가 인증한 정보임을 알려 신뢰성을 보장 할 수 있다.

### 참고자료

---

[https://www.ascentkorea.com/difference-between-http-and-https/](https://www.ascentkorea.com/difference-between-http-and-https/) 

[https://mangkyu.tistory.com/98](https://mangkyu.tistory.com/98) 

[https://myjamong.tistory.com/293](https://myjamong.tistory.com/293)
