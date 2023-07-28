[[Стандартная библиотека Golang]]

```Go
import (
	"encoding/json"
	"fmt"
)

type User struct{
	Id int
	Name string
}

func main() {
	
	us := User{
		Id: 123,
		Name: "Ivan",
		}
	// принимает значение и возвращает массив байт и ошибку
	bytes, err := json.Marshal(us)
	fmt.Println(string(bytes))
	// -> "{Id: 123, Name: Ivan}"

	// при этом после конвертации в строку получится более красивое значение
	bytes, err := json.MarshalIndent(us, "", "   ")
	fmt.Println(string(bytes))
	// -> "{
	//          Id: 123,
	//          Name: Ivan
	//     }"
}

```
при этом в структуре поля должны начинаться с большой буквы иначе другой пакет(библиотека) не увидит их и будет просто пустой словарь
```go
// -> "{}"
```
Чтобы при конвертации в json выводились нужные нам названия полей их можно прописать в тега структуры
```go
type User struct{
	Id   int    `json:"id"`
	Name string `json:"name"`
}
```