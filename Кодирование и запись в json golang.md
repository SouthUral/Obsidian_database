[[json Go]]

- Для кодирования в json используется функция ==`json.Marshal()`==
- Для кодирования с красивой структурой json используется функция ==`json.MarshalIndent()`==

Пример кода кодирования и записи в файл:
```go
import (
	"encoding/json"
	"os"

	log "github.com/sirupsen/logrus"
)

type Person struct {
	FirstName string `json:"firstname"`
	LastName  string `json:"lastname"`
	Old       string `json:"old"`
}

func main() {
	Persons := make([]Person, 0)

	Persons = append(Persons, Person{
		FirstName: "Tom",
		LastName: "Bambadil",
		Old: "3000",
		})
	Persons = append(Persons, Person{
		FirstName: "Sergey",
		LastName: "Ivanov",
		Old: "28",
		})

	PersonJson, err := json.MarshalIndent(Persons, "", "  ") 
	if err != nil {
		log.Error(err)
		return
	}

	// 0666 это права доступа к файлу (конкертно эти права означают
	// что все могут читать и изменять этот файл)
	err = os.WriteFile("persons.json", PersonJson, 0666)
	if err != nil {
		log.Error(err)
		return
	}
}
```

Результат: 
```json
[
	{
		"firstname": "Tom",
		"lastname": "Bambadil",
		"old": "3000"
	},
	{
		"firstname": "Sergey",
		"lastname": "Ivanov",
		"old": "28"
	}
]
```