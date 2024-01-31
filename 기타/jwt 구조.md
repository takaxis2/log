`Header`, `Payload`, `Signature`의 3부분으로 이루어지고, `Json`형태인 각 부분은 `Base64`로 인코딩 되어 표현. 또한 각각의 부분을 이어 주기 위해, 구분자(.)를 사용하여 구분

  

### 1. HEADER

토큰의 헤더는 `typ` 과 `alg` 두 가지 정보로 구성됩니다. `alg` 는, 헤더를 암호화 하는 것이 아니고, `Signature` 를 해싱하기 위한 알고리즘을 지정

- `typ` : 토큰의 타입을 지정 ex) JWT
- `alg` : 알고리즘 방식을 지정하며, 서명(Signature) 및 토큰 검증에 사용

### 2. PAYLOAD

토큰의 페이로드에는 토큰에서 사용할 정보들의 조각들인 클레임(Claim) 이 담겨 있습니다. 클레임은 총 3가지로 나누어지며 `JSON` 형태로 다수의 정보를 넣을 수 있다

**2-1. 등록된 클레임(Registered Claim)**

등록된 클레임은 토큰 정보를 표현하기 위해 이미 정해진 종류의 데이터들로, 모두 선택적으로 작성이 가능하며, 사용할 것을 권장. 또한 `JWT` 를 간결하게 하기 위해 key는 모두 길이 3의 `String` 이다. 여기서 `subject` 로는 unique 한 값을 사용하는데, 사용자 이메일을 주로 사용.

- iss: 토큰 발급자(issuer)
- sub: 토큰 제목(subject)
- aud: 토큰 대상자(audience)
- exp: 토큰 만료 시간(expiration), NumericDate 형식으로 되어 있어야 함 ex) 1480849147370
- nbf: 토큰 활성 날짜(not before), 이 날이 지나기 전의 토큰은 활성화되지 않음
- iat: 토큰 발급 시간(issued at), 토큰 발급 이후의 경과 시간을 알 수 있음
- jti: JWT 토큰 식별자(JWT ID), 중복 방지를 위해 사용하며, 일회용 토큰(Access Token) 등에 사용

**2-2. 공개 클레임(Public Claim)**

공개 클레임은 사용자 정의 클레임으로, 공개용 정보를 위해 사용됩니다. 충돌 방지를 위해 URI 포맷을 사용

예시

```Plain
{"https://yunmin.tistory.com":true}
```

**2-3. 비공개 클레임(Private Claim)**

비공개 클레임도 공개 클레임과 마찬가지로 사용자 정의 클레임이다. 서버와 클라이언트 사이에 임의로 지정한 정보를 저장.

예시

```Plain
{"token_type":access}
```

### 3. Signature

`Signature` 는 토큰을 인코딩하거나 유효성 검증을 할 때 사용하는 고유한 암호화 코드. 위에서 만든 `Header` 와 `Payload` 의 값을 각각 `Base64` 로 인코딩하고, 인코딩한 값을 비밀 키를 이용해 `Header` 에서 정의한 알고리즘으로 해싱을 하고 이 값을 다시 `Base64` 로 인코딩하여 생성.