if문도 for문과 마찬가지로 괄호()는 필요 없고, 중괄호{}는 필수입니다.

if문을 쓸 때는

①. `if 조건문 {}` : 바로 조건문을 검사②. `if statement; 조건문 {}` : 조건을 검사하기 전에 간단한 statement를 실행

②에 if 뒤 statement에서 선언한 변수는 if문 내에서만 쓸 수 있습니다.

---

코드를 실행하면 10번째 줄에서 에러가 발생합니다.Go언어 if-else문을 쓸 때, if문이 끝나는 줄에 else문을 선언해줘야 합니다. else 문의 위치를 수정해보세요.

  

```Go
import (
    "fmt"
    "math"
)

func sqrt(x float64) string {
    if x < 0 {
        return sqrt(-x) + "i"
    } else { //else문은 if문이 닫히는(}) 줄과 함께 쓰여야 합니다. 한칸 위로 올려서 제출해보세요! 
        return fmt.Sprint(math.Sqrt(x))
    }
}

func pow(x, n, lim float64) float64 {
    if v := math.Pow(x, n); v < lim {
        return v
    }
    // v는 if문 내부에서만 쓸 수 있고, 여기부터는 쓸 수 없습니다.
    
    return lim
}

func main() {
    fmt.Println(sqrt(2), sqrt(-4))
    
    fmt.Println(
        pow(3, 2, 10),
        pow(3, 3, 20),
    )

}
```