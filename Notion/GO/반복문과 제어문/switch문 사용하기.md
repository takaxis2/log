switch는 위에서부터 아래로 case를 검사하며, break를 하지 않아도 자동으로 case를 종료합니다.

예를 들어 아래 코드에서 i==0이면 f()는 실행되지 않습니다.

`switch i { case 0: case f(): }`

case를 종료하고 싶지 않으면 끝에 `fallthrough`를 추가하면 됩니다.

  

```Go
import (
    "fmt"
)

func no_fallthrough(score int) {
    var grade string
    switch {
    case score > 90:
        grade = "A"
    case score > 70:
        grade = "B"
    case score > 50:
        grade = "C"
    default:
        grade = "F"
    }
    
    fmt.Println("fallthrough를 쓰지 않으면", grade, "란다")
}

func yes_fallthrough(score int) {
    var grade string
    switch {
    case score > 90:
        grade = "A"
        fallthrough
    case score > 70:
        grade = "B"
        fallthrough
    case score > 50:
        grade = "C"
        fallthrough
    default:
        grade = "F"
    }
    
    fmt.Println("fallthrough를 쓰면", grade, "란다")
}
func main() {
    fmt.Println("교수님 제 성적이 어떻게 되나요?")
    
    score := 100
    yes_fallthrough(score)
    no_fallthrough(score)
}
```