메소드가 하나도 없는 인터페이스를 empty 인터페이스라고 하며, 이 empty 인터페이스는 어떤 타입이던 저장할 수 있습니다.

  

```Go
interface{}
```

empty 인터페이스는 보통 타입을 알 수 없는 값을 처리할 때 사용합니다. 예를 들어 아래 코드에서 `fmt.Print`는 `interface{}` 타입의 변수들을 처리합니다.

---

  

```Go
import "fmt"

func main() {
    fmt.Println("① empty interface에 대해")
    var i interface{}
    describe(i)
    
    fmt.Println("① i = 42에 대해")
    i = 42
    describe(i)
    fmt.Println("① i = \"hello\"에 대해")
    i = "hello"
    describe(i)
}

func describe(i interface{}) {
    fmt.Printf("인터페이스 i의 (값, 타입) : (%v, %T)\n", i, i)
}
```

```Plain
실행 결과출력 〉① empty interface에 대해
인터페이스 i의 (값, 타입) : (<nil>, <nil>)
① i = 42에 대해
인터페이스 i의 (값, 타입) : (42, int)
① i = "hello"에 대해
인터페이스 i의 (값, 타입) : (hello, string)
```