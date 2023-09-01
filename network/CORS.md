## CORS (Cross-Origin Resource Sharing)

### CORS란?


추가 HTTP 헤더를 사용하여, 한 출처에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제. 웹 애플리케이션은 리소스가 자신의 출처(도메인, 프로토콜, 포트)와 다를 때 교차 출처 HTTP 요청을 실행한다. 즉, 도메인, 프로토콜, 포트 번호 중 하나라도 다를 경우 출처가 다른 교차 출처라고 판단되며, 브라우저에는 보안 때문에 Cross-Origin HTTP 요청을 제한 한다. 권한을 부여 받기 위한 Cross-Origin 요청은 서버에서 허가를 받아야 하는데, HTTP-header를 통해 받을 수 있다. 

### 출처?


![image1](https://user-images.githubusercontent.com/78543382/216807498-b16387db-d0bc-489d-9032-edfd69b42fec.png)

Protocol + Host + Port 3가지가 같으면 동일 출처

### CORS 필요한 이유


출처가 다른 두 개의 어플리케이션이 마음대로 소통하는 환경은 위험함. 개발자 도구만 열더라도 각종 통신 정보를 쉽게 열람할 수 있기 때문에 다른 사이트로 본 사이트를 모방할 수 있고, 이를 통해 사용자의 정보 또한 탈취하기 쉬워진다.

![image2](https://user-images.githubusercontent.com/78543382/216807521-fd730719-d5a8-4276-94ea-af7e399532d9.png)

evil.com 페이지를 열었을 때 script가 실행되어 bank.com/account 를 실행하는 상황이다. 두 개의 어플리케이션이 마음대로 소통이 가능하다면, 이때 실제로 내 은행 계좌를 삭제하는 상황이 발생하는 것이다. 

### 동일 출처 정책(Same-Origin Policy)


- 같은 출처에서만 리소스를 공유할 수 있는 규칙을 가진 정책
- 다른 출처에서 얻는 이미지를 담는 img, 외부 주소를 담는 link 같은 여러 태그들은 허용한다.

![image3](https://user-images.githubusercontent.com/78543382/216807567-26d4f0c6-86f2-4817-9303-0bad933891db.png)


Postman 같은 다른 서버에서 API를 호출할 때는 잘 동작하지만, 브라우저에서 API를 호출할 때 CORS Policy 오류가 발생하는 경우가 있는데, 이는 브라우저가 동일 출처 정책을 지키고 있기에 다른 출처의 리소스 접근을 금지하고 있기 때문이다. **브라우저에만 오류가 발생하는 이유는 CORS가 브라우저의 구현 스펙에 포함되는 정책이기 때문에 브라우저를 통하지 않고 서버간 통신을 할 때는 적용되지 않기 때문이다.**  

### CORS 동작원리

**<기본흐름>**

- 웹 애플리케이션이 다른 출처의 리소스에 접근할 때, HTTP header에 요청을 보낸다. HTTP프로토콜을 사용하여 요청을 보내며, 요청 헤더 중 origin 필드에 요청을 보내는 출처를 담아서 보낸다. 이후 서버가 이 요청에 대한 응답을 할 때, 응답 헤더의 access-control-allow-origin 이라는 값에 “리소스 접근이 허용된 출처”를 내려주고, 응답을 받은 브라우저는 자신이 보낸 origin과 서버가 보내준 응답인 access-control-allow-origin을 비교하고 유요한 응답인지 확인한다. 만약 유효하지 않으면 그 응답을 사용하지 않는다.

1. **Preflight Request**
    - 일반적으로 웹 어플리케이션을 개발할 때 가장 많이 마주치는 시나리오로 이 상황에서 브라우저는 요청을 한번에 보내지 않고 예비 요청과 본 요청으로 나누어서 서버로 전송한다.
    - 본 요청을 보내기 전에 보내는 예비 요청을 Preflight라고 하며, 예비 요청에는 OPTIONS 메서드가 사용된다.  OPTIONS 메서드는 Preflight을 사용해 실제 요청이 전송하기에 안전한지 확인하는 용으로 사용한다. 다른 출처 요청이 유저 데이터에 영향을 줄 수 있기 때문에 미리 전송하여 체크하는 것이다.
        - 요청 헤더
            - origin: 어디서 요청을 했는지 서버에 알려주는 주소
            - access-control-request-method: 실제 요청이 보낼 HTTP 메서드
            - access-control-request-headers: 실제 요청에 포함된 header
        - 응답 헤더
            - access-control-allow-origin: 서버가 허용하는 출처
            - access-control-allow-methods: 서버가 허용하는 HTTP 메서드 리스트
            - access-control-allow-headers: 서버가 허용하는 header 리스트
            - access-control-max-age: preflight 요청의 응답을 캐시에 저장하는 시간

 

![image4](https://user-images.githubusercontent.com/78543382/216807589-993d3644-a978-4332-b0c4-85dbd83f24b7.png)


Preflight 요청은 OPTIONS를 사용해 자신의 주소를 보낸다. 또한 origin, access-control-request-method, access-control-request-headers를 같이 보낸다. 정상적인 응답으로 access-control-allow-origin, access-control-allow-method, access-control-allow-headers, access-control-max-age를 응답받는다. 정상 요청과 응답이 가능하다는 Preflight 덕에 실제 요청을 한다. 그리고 정상적인 응답을 받는다. 

1. **Simple Request**
    - 예비 요청을 보내지 않고 서버에 본 요청을 보낸 후, 서버가 응답 헤더에 access-control-allow-origin 값을 보내면 그때 브라우저가 cors 정책 위반 여부를 검사하는 방식이다.
    - GET, HEAD, POST 요청만 가능하다.
    - Accept, Accept-Language, Contet-Language, Content-Type과 같은 [CORS 안전 리스트 헤더](https://developer.mozilla.org/en-US/docs/Glossary/CORS-safelisted_request_header) 혹은 User-Agent 헤더만 사용 가능하다.
    - Contet-Type 헤더는 application/x-www-form-urlencoded, multipart/form-data and text/plain만 가능하다.
    - XMLHttpRequest 객체를 사용하여 요청하면, 요청에서 사용된 XMLHttpRequest.upload에 의해 반환되는 객체에 어떠한 이벤트 리스너도 등록되지 않는다.
    
  ![image5](https://user-images.githubusercontent.com/78543382/216807594-4ea11a49-5347-4bae-8734-7f690afac593.png)
    

브라우저는 다른 출처에 자신의 주소를 origin에 담아서 요청을 보내면 서버는 요청을 확인하고 다른 출처 주소에 접근이 가능하다는 access-control-allow-origin에 해당 주소를 담아서 결과를 리턴한다. access-control-allow-origin의 값은 하나의 출처가 될 수도 있고 “*”를 사용해 어떤 출처도 허용하도록 할 수 있다. 서버가 이 헤더에 응답하지 않거나, 헤더 값이 요청의 출처와 일치하지 않는 도메인인 경우, 브라우저는 응답을 차단한다.

1. Credentialed Request
- 인증된 요청을 사용하는 방법으로 CORS 기본적인 방식 보다는 다른 출처 간 통신에서 조금 더 보안을 강화하고 싶을 때 사용한다.
- 기본적으로 CORS 정책은 다른 출처 요청에 인증정보 포함을 허용하지 않는다. 요청에 인증을 포함하는 플래그가 있거나 access-control-allow-credentials를 true로 설정 한다면 요청할 수 있다.

![image6](https://user-images.githubusercontent.com/78543382/216807599-9680def1-86c1-46b6-83f4-8fa6c60d2a32.png)

서버 응답에 access-control-allow-credentials 가 true로 설정되지 않았거나 access-control-allow-origin 헤더에 있는 값이 허용된 출처가 아니라면 오류가 발생한다.
### 참고 자료


[https://hymndev.tistory.com/78](https://hymndev.tistory.com/78)

[https://escapefromcoding.tistory.com/724](https://escapefromcoding.tistory.com/724)