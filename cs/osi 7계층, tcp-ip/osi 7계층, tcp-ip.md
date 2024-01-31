-네트워크에서 통신이 일어나는 과정을 7단계로 나눈 것을 말한다.

-통신이 일어나는 과정을 단계적으로 파악할 수 있다.

-단계 별로 나누어져 있기에 오류 발생 시 다른 곳을 건드리지 않고 이상이 생긴 단계만 고치면 된다

-유지 보수에 용이하다

  

계층

1. application layer
    - *애플리케이션 목적에 맞는 통신 방법 제공
    - HTTP, DNS, SMTP, FTP …
2. presentation layer
    - *애플리케이션 간의 통신에서 메세지 포맷 관리
    - 인코딩 ↔ 디코딩
    - 암호화 ↔ 복호화
    - 압축 ↔ 압축 풀기
3. session layer
    - *통신에서 세션을 관리
    - RPC(remote procedure call)
4. transport layer
    - *애플리케이션 간의 통신을 담당
    - 목적지의 애플리케이션으로 데이터 전송
    - 안정적이고 신뢰할 수 잇는 데이터 전송 보장(TCP)
    - 필수 기능만 제공(UDP)
    - 데이터를 목적지로 전송하기 위해서 어떤 식의 통신을 할 것인가를 결정하는 레이어
5. network layer
    - *호스트간의 통신 담당(IP)
    - 목적지 호스트로 데이터 전송
    - 네트워크 간의 최적의 경로 결정
    - 실제로 데이터가 목적지로 찾아가게 하는 기능
    - 전체적인 경로가 관심사이고 각각의 노드(라우터 등) 사이에서 어떻게 데이터를 전송하는 것에 대해서는 관심사가 아님
6. data link layer
    - *직접 연결된 노드 간의 통신 담당
    - MAC 주소 기반 통신(ARP), 장치와 장치 사이의 데이터 통신때 사용
    - ARP : IP주소를 MAC주소로 변환할 때 사용하는 프로토콜
7. physical layer
    - *물리적인 매게체를 통해서(케이블, 무선신호 등) bits 단위로 데이터 전송

  

메세지는 각 계층을 내려갈 때 마다 헤더가 붙고(data link layer에서는 trailer라고 따로 붙고 전송 후에 에러가 없는지 확인하는 용도이다) 올라갈 때마다 빼어낸다

![[Untitled.png]]

  

### 컴퓨터(A) ↔ 라우터 ↔ 컴퓨터에서 통신(B)

![[Untitled 1.png]]

1. A 컴퓨터 : 데이터를 보낼 때, application layer부터 시작해서 physical layer까지 내려오면서 데이터를 포장(가공) 후, hpysical layer를 통해 라우터로 전달
2. 라우터 : 받을 데이터를 network layer까지 올려서 가공 → 목적지 IP 주소 확인
3. 라우터 : network layer까지 계층을 올린 데이터를 다시 physical layer까지 내린다(가공)후 전달
4. B 컴퓨터 : 받은 데이터를 application layer까지 올린다(가공)

  

- 라우터는 통신을 하도록 만들어 주는 장치이기 때문에 보통 3계층만 구현한다

  

  

  

  

TCP/IP

-인터넷에 특화된 네트워크 구조

-4개의 단계로 압축되어 있음

  

  

![[Untitled 2.png]]

  

출처

[https://shlee0882.tistory.com/110](https://shlee0882.tistory.com/110)

[https://www.youtube.com/watch?v=6l7xP7AnB64&t=893s](https://www.youtube.com/watch?v=6l7xP7AnB64&t=893s)