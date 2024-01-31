  

결론

1. 리프레시 토큰은 HTTP ONLY SECURE 쿠키에 저장하자.
2. 액세스 토큰은 프로그램상 자바스크립트 로컬 변수(메모리)에 저장하고, http 헤더에 bearer 토큰으로 담아서 매 요청마다 보내도록 하자.
3. 로컬스토리지는 사용하지 말자. (보안에 매우 취약)
4. 액세스 토큰의 기한을 짧게 하여 탈취 당하더라도 짧은 시간 안에 사용불가로 만든다
    
    (엑세스: 5 ~ 10분, 리프레시 : 30 ~ 60분이 보통)
    

  

이유

1. localstorage : XSS 공격에 취약
2. cookies : XSS공격, js로 외부에서 접근가능, CSRF 공격에 취약

  

  

[https://simsimjae.tistory.com/482](https://simsimjae.tistory.com/482)