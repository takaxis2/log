clinet ↔ api server : DTO사용

api 서버 : controller , service, db

controller ↔ service : DTO 사용

service ↔ db : Entity 사용

  

db에 반영할때만 entity를 사용하고 사용자에게 값을 보내줄때는 DTO로 보내는데,

보통 db에서 데이터를 찾으면 entity가 나오기 때문에 DTO로 변환과정이 필요함.

  

.net에서는 AutoMapper를 사용한다

## **AutoMapper 란?**

- AutoMapper는 객체 전환간에 Property 값을 자동으로 매핑해주는 오픈소스 라이브러리 입니다.
- 보통 DTO -> Entity 객체간 전환에 대표적으로 많이 사용합니다.

  

설치

```C#
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

  

루트경로에(Program.cs랑 같은 위치) AutoMapperProfile.cs 생성

```C#
public class AutoMapperProfile:Profile
{
	public AutoMapperProfile()
	{
		CreateMap<Entity, DTO>();
		CreateMap<Entity, DTO>();
	}
}
```

여러개일 경우에는 그만큼 추가하면 된다

  

program.cs

```C#
builder.Services.AddAutomapper(typeof(Program).Assembly);
```

이 코드가 쓰이는 클래스가 매개변수에 쓰인다.

  

사용

- 생성자로 IMapper _mapper를 주입받는다

```C#
_mapper.Map<DTO>(Entity);
```

- Entity가 매개변수로, DTO가 리턴된다