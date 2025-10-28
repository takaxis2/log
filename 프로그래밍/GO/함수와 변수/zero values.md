선언만 하고 초기화하지 않은 변수는 zero value로 초기화됩니다. zero value는 변수의 타입에 따라 다음과 같이 나뉩니다.

- 숫자 타입 : 0
- boolean 타입 : false
- string 타입 : ""

```Go
import (
    "fmt"
)

var (
    i int // zero value = 0
    f float64 // zero value = 0
    b bool // zero value = false
    s string // zero value = ""
)

func main() {
    fmt.Printf("int의 zero value: %v\n", i)
    fmt.Printf("float64의 zero value: %v\n", f)
    fmt.Printf("boolean의 zero value: %v\n", b)
    fmt.Printf("string의 zero value: %q\n", s)
}
```