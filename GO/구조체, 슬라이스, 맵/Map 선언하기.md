맵은 key에 value를 지정하는 자료형입니다. 맵을 만들 때에는 `make`함수를 써야 하며, zero value는 `nil`입니다.

key값에 접근할 때에는 `맵 변수[키]`와 같이 접근합니다.

---

맵 리터럴은 구조체 리터럴에 key를 추가한 것과 같고, 맵 리터럴은 `make`함수가 필요 없습니다.

  

```Go
import "fmt"

type Vertex struct {
    Lat, Long float64
}

func main() {
    //① map 사용
    //map[string] 타입 변수 선언
    var mymap map[string]Vertex
    //make()로 맵 생성
    mymap = make(map[string]Vertex)
    mymap["Bell Labs"] = Vertex{
        40.68433, -74.39967,
    }
    fmt.Println("① mymap[\"Bell Labs\"]: ", mymap["Bell Labs"])
    //② map literal 사용
    var mymap_literal = map[string]Vertex{
        "Bell Labs": Vertex{
            40.68433, -74.39967,
        },
        "Google": Vertex{
            37.42202, -122.08408,
        },
    }
    fmt.Println("② mymap_literal[\"Bell Labs\"]", mymap_literal["Bell Labs"])
    
}
```

  

```Plain
실행 결과출력 〉① mymap["Bell Labs"]:  {40.68433 -74.39967}
② mymap_literal["Bell Labs"] {40.68433 -74.39967}
```