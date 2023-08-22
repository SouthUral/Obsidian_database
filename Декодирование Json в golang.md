[[json Go]]
### Чтение и декодирование  json файла

код для декодирования такой структуры:
```json
{
	"dbname": "test_name",
	"host": "localhost",
	"port": "5798",
	"login": "test_user",
	"password": "test_password"
}
```

код:
```go
import (
	"encoding/json"
	"fmt"
	"os"
	
	log "github.com/sirupsen/logrus"
)

type testStruct struct {
	DbName   string `json:"dbname"`
	Host     string `json:"host"`
	Port     string `json:"port"`
	Login    string `json:"login"`
	Password string `json:"password"`
}

func main() {

	newVal := testStruct{}
	
	// чтение файла целиком
	data, err := os.ReadFile("config.json")
	if err != nil {
		log.Error(err)
	}
	
	err = json.Unmarshal(data, &newVal)
	if err != nil {
		log.Error(err)
	}
	
	fmt.Println(newVal)
}

```

Если нужно декодировать массив Json с подобной структурой:
```json
[
	{
		"dbname": "test_name",
		"host": "localhost",
		"port": "5798",
		"login": "test_user",
		"password": "test_password"
	},
	{
		"dbname": "name2",
		"host": "localhost",
		"port": "5791",
		"login": "user2",
		"password": "passuser2"
	}
]
```

Код:
```go
type sliceTest []testStruct

type testStruct struct {
	DbName string `json:"dbname"`
	Host string `json:"host"`
	Port string `json:"port"`
	Login string `json:"login"`
	Password string `json:"password"`
}

  

func main() {
	
	newVal := make(sliceTest, 0)
	
	data, err := os.ReadFile("config.json")
	if err != nil {
		log.Error(err)
	}
	
	err = json.Unmarshal(data, &newVal)
	if err != nil {
		log.Error(err)
	}
	
	fmt.Println(newVal)
}
```