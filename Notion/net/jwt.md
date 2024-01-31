준비물

bcrypt : 암호화에 사용

```C#
dotnet add package BCrypt.Net-Next
```

사용법

```C#
BCrypt.Net.BCrypt.HashPassword(/* param */); // 값을 암호화, salt나 형식이 더 있음

BCrypt.Net.BCrypt.Verify(/*string*/, /*hash*/); // 매개변수로 전해진 값이 같은것인지 확인
```

  

토큰 생성

cli로 생성 가능

```C#
dotnet user-jwts create // jwt 생성, id와 jwt토큰을 준다

dotnet user-jwts print {id} --show-all // 아이디에 해당하는 jwt를 풀어서 보여준다

dotnet user-jwts key // 암호화에 쓰는 키를 보여준다
dotnet user-secrets list 
```

  

  

program.cs

```C#
builder.Services.AddAuthentication().AddJwtBearer();
```

appsetings.json

```C#
"Authentication": {
    "DefaultScheme" : "JwtBearer",
    "Schemes": {
      "JwtBearer": {
        "Audiences": [ "http://localhost:5000", "https://localhost:5001" ],
        "ClaimsIssuer": "user-jwt-here"
      }
    }
  }
```

  

또는

```C#
private string CreateToken(User user){
	// jwt body에 들어갈 것 들
	List<Claim> claims = new List<Claim>{
		new Claim(ClaimTypes.Name, user.UserName),
		new Claim(ClaimTypes.Role, "ADMIN"),
		// 필요한 필드 추가
	};

	// appsettings.json에서 jwt 암호화에 쓸 키 값  가져오기
	var key = SymmetricSecurityKey(Encoding.UTF8.GetBytes(
		_configuration.GetSection("Appsettings:Token").Value!));

	// jwt 키 값 암호화
	var cred = new SigningCredentials(key, SecurityAlgorithms.HmacSha512Signature);

	// jwt 생성
	var token = JwtSecurityToken(
			claims: claims,
			expires: DateTime.Now.AddDays(1),
			signingCredentials: cred
		);
	var jwt = new JwtSecurityTokenHandler().writeToken(token);

	return jwt;
}

// 필요한 Nuget 패키지를 다 설치해야한다
```