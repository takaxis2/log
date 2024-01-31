부모를 삭제할 때

- CASCADE
- SET NULL
- SET DEFAULT
- RESTRICT

자식을 입력할 때

- AUTOMATIC
- SET NULL
- SET DEFAULT
- DEPENDENT

  

- ON DELETE
- 부모 테이블의 행이 삭제될 때 자식 테이블의 행에 어떤 일이 발생하는지 정의할 수 있다

|   |   |
|---|---|
|Action|설명|
|CASCADE|부모 삭제 시 자식도 삭제|
|SET NULL|부모 삭제 시 자식은 NULL로 설정|
|SET DEFAULT|부모 삭제 시 자식은 기본 값 설정|
|RESTRICT|자식이 없는 경우만 부모 삭제|

  

- ON INSERT
- 자식 테이블의 행이 입력될 때 부모 테이블의 행과 관련해서 어떻게 할 것인지 정의

  

|   |   |
|---|---|
|Action|설명|
|AUTOMATIC|부모가 없을 때 부모 입력 후 자식 입력|
|SET NULL|부모가 없는 경우, 자식의 FK를 NULL|
|SET DEFAULT|부모가 없는 경우, FK를 기본값으로|
|DEPENDENT|부모의 PK가 있는 경우만 자식 입력|