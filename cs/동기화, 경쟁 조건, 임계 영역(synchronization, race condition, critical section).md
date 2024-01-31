race condition(경쟁 조건)

- 여러 프로세스/쓰레드가 동시에 같은 데이터(자원)를 조작할 때 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황

  

synchronization(동기화)

- 여러 프로세스/쓰레드를 동시에 실행해도 공유 데이터의 일관성을 유지하는 것

  

critical section(임계영역)

- 공유 데이터의 일관성을 보장하기 위해 하나의 프로세스/쓰레드만 진입해서 실행 가능하게 한 영역(교착 상태가 발생할 수도 있다)

```C#
do{
	entry section // 임계 영역 진입 조건 확인
		critical section
	exit section // 나머지 조치, 동기화?
} while(true)
```

critical section 문제의 해결책 조건

- mutual exclusion(상호 배제) : 한번에 하나의 프로세스/쓰레드만 임계 영역에서 실행가능
- progress(진행) : 만일 임계 영역이 비어있고 진입해야 하는 프/쓰가 있다면 그 중 하나는 임계 영역 안에서 실행 될 수 있어야 한다
- bounded waiting(한정된 대기) : 어떤 프/쓰가 임계 영역에 진입하지 못하고 무한정으로 대기하면 안된다

  

C# 동기화

- lock 키워드
- Monitor 클래스