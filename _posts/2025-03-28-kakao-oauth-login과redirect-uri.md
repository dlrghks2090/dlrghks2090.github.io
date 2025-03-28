# **kakao oauth login의 redirect-uri에 대한 이해 및 에러해결**
---
나는 REST 아키텍처 구조인 Client-Server 형식으로 프로젝트를 진행중에 있었다.
Client는 JS/React, Server는 Java/SpringBoot를 사용했다.

1. Client - kakao server 에서 code를 받아와 이 코드를 back server로 보낸다.
2. Back Server - client에서 받아온 code를 사용해 accessToken을 받아와 유저정보 저장 및 jwt를 생성하여 클라이언트에 반환
<br>

---
### **문제발생**
크게 위 세단계로 소셜로그인을 처리하게 구성하였는데, KOE320 오류가 발생
<br>

---
### **KOE320 (invalid_grant) 에러의 원인**
원인1. 동일한 인가 코드를 두 번 이상 사용하거나, 이미 만료된 인가 코드를 사용한 경우, 혹은 인가 코드를 찾을 수 없는 경우<br>
원인2. 인가코드요청과 엑세스 토큰 발급요청 시, 리다이렉트 URI를 동일하게 설정해야하는데 아래와 같이 다르게 설정해서 에러 발생
<br>

```java
<에러발생상황 redirect-uri설정>
예시1
인가코드: null
접근토큰: http://localhost:3000
예시2
인가코드: http://localhost:3000
접근토큰: http://localhost:8080
```

```java
<정상 작동 redirect-uri설정>
인가코드: http://localhost:3000
접근토큰: http://localhost:3000
```
<br>

---
### **해결**
나의 경우 위의 예시2처럼 인가코드와 접근토큰을 발급 받을때 사용하는 redirect-uri을 다르게 설정해 발생하는 오류였다.<br>
아무래도 client에서 code를 발급받고, server에서 accessToken을 발급받았기에 이런 오류가 발생했다.
<br>
```java
<에러상황이 난 설정>
(Client)인가코드: http://localhost:3000/user/kakao/callback
(Server)접근토큰: http://localhost:8080/user/kakao/callback
```
<br>
=> 해결책은 같은 redirect uri로 설정한다.<br>
=> 그럼 프론트 or 백 중에 어느것을 변경할까?<br>
=> client에서 code를 발급받을때 redirct uri로 리다이렉트 되는것을 이용해 path='/user/kakao/callback' KakaoAuthHandle을 통해 새로운 페이지로 리다이렉트 시키는 기능을 만들었기 때문에 Server의 ridirect uri를 http://localhost:8080/user/kakao/callback 로 변경하기로 하였다.
<br>

~~~java
<수정된 설정>
(Client)인가코드: http://localhost:3000/user/kakao/callback
(Server)접근토큰: http://localhost:8080/user/kakao/callback
~~~
<br>

---
### **결론**
해당 이슈는 redirect uri에 대한 이해부족과 카카오의 code와 accessToken 발급시 같은 redirect uri를 사용해야된다는 규정을 몰랐기에 발생한 문제였다. <br>
현재 프론트에 지식을 쌓아가고 있어 에러를 파악하고 수정하는데 시간이 조금 걸렸지만 이 문제를 해결하면서 redirect와 kakao redirect uri에 대한 지식을 정리할 수 있었다.
<br>

---
### **문제 해결에 도움이 되었던 링크**
[카카오 로그인 redirect uri 이슈(서비스 프론트엔드 백엔드 도메인이 다른 경우)](https://devtalk.kakao.com/t/redirect-uri/131865/1)<br>
[리다이렉트(Redirect)](https://docs.tosspayments.com/resources/glossary/redirect)