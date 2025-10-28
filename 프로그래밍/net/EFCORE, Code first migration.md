Db 연결용 패키지

```C#
dotnet add package Microsoft.EntityFrameworkCore.{데이터베이스};
dotnet add package Microsoft.EntityFrameworkCore.Design;
dotnet add package Microsoft.EntityFrameworkCore.SqlServer;

dotnet tool install --global dotnet-ef;
```

dotnet-ef의 경우 이미 설치가 되어있을 수도 있고 버전이 다를 수도 있기에

삭제하고 설치하는 것을 추천

  

Code First Migration

- 데이터 베이스에 테이블을 만들고 연결하는게 아닌 코드를 작성하고 그것을 기반으로 데이터베이스를 구성

  

관계

- OneToOne
- OneToMany
- ManyToOne
- ManyToMany
- JPA랑은 다르게 클래스 멤버로 구별하는것 같다, 어노테이션이 없음

  

사용하려면 컨텍스트 등록이 필요하다, 고로 이런 파일 생성

```C#
using Microsoft.EntityFrameworkCore;

namespace CouponService.Models;

public class CServiceContext : DbContext{

    public CServiceContext(DbContextOptions<CServiceContext> options) : base(options){}

    public DbSet<User>? Users {get; set;}
    public DbSet<Cupon>? Cupons {get; set;}
    public DbSet<UserCupon>? userCupon {get; set;}

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder){
        optionsBuilder.UseMySql(ServerVersion.AutoDetect("server=127.0.0.1; user=root; password=1q2w3e4r!;database=coupon_service"));
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder){
        // modelBuilder.Entity<Cupon>()
        // .HasMany(c => c.userCupons)
        // .WithOne();

        // modelBuilder.Entity<User>()
        // .HasMany(u => u.userCupons)
        // .WithOne();
    }
    
}
```

관계설정을 주석처리된 부분처럼 할 수도 있다.

  

  

program.cs에도 추가, 위의 코드를 바탕으로 만든 추가 코드

```C#
builder.Services.AddDbContext<CServiceContext>(
    options => options.UseMySql(builder.Configuration.GetConnectionString("MySQL"),
        ServerVersion.AutoDetect(builder.Configuration.GetConnectionString("MySQL"))));
```

  

appsettings.json

```C#
"ConnectionStrings": {
    "MySQL" : "server=127.0.0.1; port=3306; user=root; password=1q2w3e4r!;database=coupon_service"
  }
```

DefaultConnection등 여러 설정이 가능하다

  

처음 사용시, 마이그레이션 파일 생성

```C#
dotnet ef migrations add {migration 이름}
```

마이그래이션 반영, 가장 최근 마이그래이션 파일로 db 반영

```C#
dotnet ef database update
```