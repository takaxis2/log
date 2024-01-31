for문은 세미콜론을 기준으로 세 부분으로 나뉘며, 각각 비워두는 것도 가능합니다.타 언어와는 달리 괄호()가 없고, 중괄호{}가 필수입니다.

`init statement ; condition expression ; post statement {}`

Go언어에 반복문은 for문이 유일합니다. while문을 써야 할 경우에는 다음과 같이 for문을 써서 흉내 냅니다.

`for 조건 {}`

마지막 줄에 `for {}` 를 추가해 무한루프를 실행해 보세요.

  

```Go
import "fmt"

func main() {
    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }
    fmt.Println(sum)
    
    // 세미콜론 없이, C의 while과 비슷하게 쓸 수 있습니다.
    sum = 1
    for sum < 1000 {
        sum += sum
    }
    fmt.Println(sum)
    
    // for{} 로 무한 루프를 만들어보세요!
    
}
```