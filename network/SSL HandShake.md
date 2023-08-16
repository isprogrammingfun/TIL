## SSL HandShake

### SSL이란?

---

- 기존 OSI 계층에서 보안 계층이라는 독자적인 프로토콜 계층을 만들어, 응용계층과 전송계층 사이에 위치한다.
- CA (Certificate Authority)라는 인증기간에서 발급해준다.
- 브라우저에는 CA 기관들의 목록과 공개키가 내장되어 있다.
- 웹사이트가 신뢰할 수 있는 곳이라는 인증서다.
- 컴퓨터 네트워크 상에서 보안 통신을 제공하기 위해 설계된 프로토콜이다. (클라이언트와 서버 간의 통신을 보증한다)

### SSL HandShake

---

목적: 데이터를 실제로 암호화 하기 위함 대칭키를 전달하는 것 

![img](https://user-images.githubusercontent.com/78543382/213914241-dcbd7796-5d9d-4ddf-9747-faf06235860b.png)


1. TCP 3 way HandShake를 우선적으로 거친다
2. **[Client to Server]** 컴퓨터가 자신의 버전, 암호 알고리즘 목록, SSL 정보, 사용 가능한 압축 방식등을 “Client Hello” 메시지에 담아 서버로 전송한다.
    - Client Hello: 클라이언트가 서버에 연결을 시도하며 전송하는 패킷이다. 자신이 사용 가능한 Cipher Suite 목록, Session ID, SSL 프로토콜 버전, Random Byte등을 전달한다.
        - Random Bytes: 클라이언트가 보안 난수 생성기를 사용하여 랜덤 바이트를 생성 해야한다. 난수 생성을 위한 엔트로피 소스는 운영체제 및 클라이언트 소프트웨어 구현에 따라 다르다.
        - Session ID: 과거에 클라이언트와 서버 간에 TLS/SSL 세션이 설정된 경우 이 식별자는 이 세션의 암호화 매개변수를 업데이트하는 데 사용할 수 있는 이전 세션 ID 값을 가질 수 있다. 새로운 세션에는 비어있다.
        - Cipher Suite: 인증서 검정, 데이터 암호화 프로토콜, Hash 방식 등의 정보를 담고 있는 존재로, 선택된 Cipher Suite의 알고리즘에 따라 데이터를 암호화하게 된다. 클라이언트는 선호하는 순서대로 지원하는 Cipher Suite 목록도 보낸다.
    
 ![다운로드](https://user-images.githubusercontent.com/78543382/213914272-75170a8a-a65a-4f6a-90bb-e1b703e42aeb.png)
    
3. **[Server to Client]** 클라이언트가 제공한 암호 알고리즘 목록, 압축 방식 목록 중에서 하나씩 선택을 한 뒤 세션 ID, CA에서 사인한 서버의 공개 인증서를 “ServerHello”에 담아 전송한다. 
    - ServerHello: 클라이언트가 보내 온 ClientHello 패킷을 받아  Cipher Suite 중 하나를 선택한 다음 클라이언트에게 이를 알린다. 또한 자신의 SSL 프로토콜 버전 등도 같이 보낸다.
        - Random Bytes: 클라이언트와 독립적으로 서버에서 생성된 임의의 숫자
        - Version: 서버의 TLS/SSL 버전을 알려준다
        - Cipher Suite: 클라이언트가 선호하는 Cipher Suite 중 선택한 것을 알려준다.
    
![다운로드 (1)](https://user-images.githubusercontent.com/78543382/213914293-a6f74472-1dba-466a-9727-3c4cbe6b66cc.png)

    
4.  **[Server to Client]** 서버가 자신의 SSL 인증서를 클라이언트에게 보낸다. 이후 클라이언트는 SSL 인증서를 검증하는 과정을 거쳐 서버의 공개키를 획득한다. (Certificate)
    - Certificate: 서버에서 클라이언트로 보내는 SSL 인증서 내부에는 Server가 발행한 공개 키(개인키는 따로 서버가 소유)가 들어있다. 클라이언트는 서버가 보낸 CA의 개인 키로 암호화된 이 SSL 인증서를 이미 모두에게 공개된 CA의 공개키를 사용하여 복호화 한다. 복호화에 성공하면 이 인증서는 CA가 서명한 것이 맞으므로 진짜임을 검증할 수 있다. 인증서 확인 단계는 TLS/SSL 프로토콜의 일부가 아니다.
        - encrypted: 실제로 첫 번째 인증서와 연결된 디지털 서명이다.
        - algorithmIdentifier: SHA-256 해시 값이 상위 인증서 Authority G2에 있는 인증기관의 RSA(비공개) 키를 사용하여 서명되었음을 의미한다. 그러면 브라우저는 Google Internet Authority G2 인증서에 포함된 해당 공개 키를 사용하여 서명을 확인한다.
    
 ![다운로드 (3)](https://user-images.githubusercontent.com/78543382/213914312-63ae483d-5ca9-4769-b112-603e3175a87c.png)
    
5. **[Server to Client] (선택)** SSL 인증서에 서버의 공개키가 없는 경우, 서버가 직접 전달한다.  키 교환 알고리즘이 공개키 방식이 아니라면 인증서 안에 공개키가 들어있지 않다. 이 경우 서버에서 키 교환에 필요한 다른 값을 서버에서 직접 전달한다. DH 알고리즘은 공개된 특정 값을 통해 대칭키를 계산하기 때문에, 이 값dmf Server Key Exchange 과정에서 전달한다. 
6. **[Server to Client]** Server Hello Done: 서버의 작업이 종료
7. **[Client to Server]** 클라이언트가 대칭키(데이터를 실제로 암호화 할 키)를 생성하여, SSL인증서에 있던 서버의 공개키로 암호화하여 전달. 여기서 전달된 대칭 키가 SSL Handshake의 목적이자 가장 중요한 수단인 데이터를 실제로 암호화할 대칭키다. 이 키를 통해 클라이언트와 서버가 실제로 교환하고자 하는 데이터를 암호화 한다. 
8. **[Client to Server] [Server to Client]** 클라이언트와 서버가 서로 교환할 정보를 패킷으로 전달한다. 서버는 클라이언트가 보낸 대칭 키를 서버의 개인키로 복호화 해 대칭키를 얻는다. 

### 참고자료

---

[https://steady-coding.tistory.com/512](https://steady-coding.tistory.com/512)

[https://seongonion.tistory.com/76](https://seongonion.tistory.com/76)

[https://velog.io/@ann0905/HTTPS-2.-HTTPS와-SSL-Handshake](https://velog.io/@ann0905/HTTPS-2.-HTTPS%EC%99%80-SSL-Handshake)

[https://chinggin.tistory.com/565](https://chinggin.tistory.com/565)