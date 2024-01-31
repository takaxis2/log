  

컨트롤러에서 권한확인

```C#
// Authorize를 추가, jwt의 role로 쳐낸다
[HttpGet(Name = ""), Authorize("ADMIN", "USER")]
```

  

program.cs

```C#
// 코드 추가와 필요한 Nuget 패키지 추가

builder.Services.AddAuthentication().AddJwtBearer(options =>{
	options.TokenValidationparameters = new TokenValidationparameters
		{
			// 아래 값들은 상황에 따라서 바뀔 수 있다.
			// 확인 할 값에는 true, 아니며 false
			// 마지막은 발급시에 사용하던 코드와 같은 형식으로 확인
			ValidateIssuerSigningKey = true,
			ValidateAudience = false,
			ValidateIssuer = false,
			IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration.GetSection("Appsettings:Token").Value!))
		};
});

// 없으면 추가
app.UseAuthorization();
```

  

swagger에서 편리하게 jwt 값을 전달해서 사용하는 법.

적용하면 swaggerUI에 Authoraze 버튼이 생긴다.

```C#
builder.Services.AddSwaggerGen(options =>{
	options.AddsecurityDefinition("oauth2", new OpenApiSecurityScheme 
	{
		// jwt는 header의 Authorization에 값이 들어간다(bearer + " " + jwt)+
		In = ParameterLocation.Header, 
		Name = "Authorization",
		Type = SecuritySchemeType.ApiKey
	});
	
	options.OperationFilter<SecurityRequirementsOperationFilter>();
})
```