반복문에서 range문을 쓰면 슬라이스나 맵을 이터레이트할 수 있습니다.

슬라이스에서 range를 쓸 경우, 인덱스, 원소 값이 매 순회마다 리턴됩니다.

이때 인덱스나 원소값이 필요 없다면 변수를 지정하지 않고 _를 쓸 수 있습니다.

  

```Go
import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
    //① 일반적인 range
    fmt.Println("① 일반적인 range")
    for i, v := range pow {
        fmt.Printf("2**%d = %d\n", i, v)
    }
    
    //② 인덱스가 필요없는 경우 _로 비워둘 수 있습니다.
    fmt.Println("② 인덱스가 필요없는 경우")
    for _, v := range pow {
        fmt.Println(v)
    }
}
```

  

```Plain
실행 결과출력 〉① 일반적인 range
2**0 = 1
2**1 = 2
2**2 = 4
2**3 = 8
2**4 = 16
2**5 = 32
2**6 = 64
2**7 = 128
② 인덱스가 필요없는 경우
1
2
4
8
16
32
64
128
```