Annotation

|   |   |
|---|---|
|Annotation|Description|
|@Test|테스트용 메소드를 표현하는 어노테이션|
|@BeforeEach|각 테스트 메소드가 시작되기 전에 실행되어야 하는 메소드 표현|
|@AfterEach|각 테스트 메소드가 시작된 후에 실행괴어야하는 메소드 표현|
|@BeforeAll|테스트 시작 전에 실행되어야 하는 메소드 표현(static 처리 필요)|
|@AfterAll|테스트 종료 후에 실행되어야 하는 메소드 표현(static 처리 필요)|

  

JUnit Main Annotaion

@SpringBootTest

- 통합 테스트 용도
- @SpringBootApplication을 찾아가 하위의 모든 Bean을 스캔, 로드
- 그 후 Test용 Application Context를 만들어 Bean을 추가하고, MockBean을 찾아 교체

  

@ExtendWith

- JUnit4에서 @RunWith로 사용되던 어노테이션이 ExtendWith로 변경됨
- @ExtendWith는 메인으로 실행될 Class를 지정할 수 있음
- @SpringbootTest는 기본적으로 @ExtendWith가 추가되어 있음

  

@WebMvcTest(Class명.class)

- ()에 작성된 클래스만 실제로 로드하여 테스트를 진행
- 매게변수를 지정해주지 않으면 @Controller, @RestController, @RestControllerAdvice 등 컨트롤러와 연관된 Bean이 모두 로드됨
- 스프링의 모든 Bean을 로드하는 @SpringBootTest 대신 컨트롤러 관련 코드만 테스트할 경우 사용

  

@Autowired about MockBean

- Controller의 API를 테스트하는 용도인 MockMvc 객체를 주입 받음
- perform() 메소드를 활용하여 컨트롤러의 동작을 확인할 수 있음. andExpect(), andDo(), andReturn() 등의 메소드를 같이 활용

  

@MockBean

- 테스트할 클래스에서 주입받고 있는 객체에 대해 가짜 객체를 생성해주는 어노테이션
- 해당 객체는 실제 행위를 하진 않음
- given() 메소드를 활용하여 가짜 객체의 동작에 대해 정의하여 사용할 수 있음

  

@AutoConfigureMockMvc

- spring.test.mockmvc의 설정을 로드하면서 MockMvc의 의존성을 자동으로 주입
- MockMvc 클래스는 REST API 테스트를 할 수 있는 클래스

  

@Import

- 필요한 Class들을 Configuration으로 만들어 사용할 수 있음
- Configuration Component 클래스도 의존성 설정할 수 있음
- Import된 클래스는 주입으로 사용가능