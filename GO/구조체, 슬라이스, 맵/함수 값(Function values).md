함수도 값이기 때문에 다른 값과 똑같이 사용할 수 있습니다. 함수를 변수에 대입하거나, 다른 함수에 인자로 넘기는 것도 가능합니다.

12번째 줄은 함수 hypot을 변수에 저장하고 있으며,15번째 줄은 함수 hypot을 다른 함수에 인자로 쓰고 있습니다.

  

```Go
import (
    "fmt"
    "math"
)

func compute(fn func(float64, float64) float64) float64 {
    return fn(3, 4)
}

func main() {
    //① 함수를 변수에 할당해, 변수를 함수처럼 씁니다.
    hypot := func(x, y float64) float64 {
        return math.Sqrt(x*x + y*y)
    }
    fmt.Println("①변수를 통해 함수 호출", hypot(5, 12))
    
    //② 함수를 compute함수에 인자로 전달합니다.
    fmt.Println("②함수를 함수에 인자로 전달")
    fmt.Println("compute(hypot):\t\t", compute(hypot))
    fmt.Println("compute(math.Pow):\t", compute(math.Pow))
}
```

```Plain
실행 결과출력 〉①변수를 통해 함수 호출 13
②함수를 함수에 인자로 전달
compute(hypot):		 5
compute(math.Pow):	 81
```