[[Работа с БД в Golang]]

## Подключение к MongoDB

для работы с монго нужно импортировать следующие библиотеки
```go
package main

import (
	"log"
	"context"
	"fmt"

	"go.mongodb.org/mongo-driver/bson" 
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)
```

настройка подключения (используется `import` из верхнего примера)
```go
// установка настроек клиента
clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")

// подключение к MongoDB
client, err := mongo.Connect(context.TODO(), clientOptions)
if err != nil {
	log.Fatal(err)
}

// проверка подключения
err = client.Ping(context.TODO(), nil)
if err != nil {
	log.Fatal(err)
}

fmt.Println("Connected to MongoDB!")
```

после успешного соединения можно обратиться к коллекции, для этого нужно обратиться с базе и далее к коллекции. Пример:
```go
collection := client.Database("test").Collection("trainers")
```

Если соединение нужно закрыть можно использовать функцию `client.Disconnect()`
```go
err = client.Disconnect(context.TODO())

if err != nil {
	log.Fatal(err)
}

fmt.Println("Connection to MongoDB closed.")
```

## Использование BSON объектов

JSON документы в mongoDB хранятся в двоичном формате - ==BSON==. В отличие от других БД в которых JSON данные хранятся в виде строк и чисел, кодировка BSON добавляет новые типы, такие как **int**, **long**, **date**, **float**, **decimal128**

Это значительно упрощает обработку, сортировку и сравнение данных приложениями. 
Драйвер Go имеет два семейства типов для предоставления данных BSON: 
 - Типы D, семейство D состоит из четырех типов:
	 - **D** : документ BSON. Этот тип следует использовать, когда порядок имеет значение;
	 - **M** : Неупорядоченный словарь(map). Он такой же, как **D** , только не сохраняет порядок;
	 - **A** : массив BSON;
	 - **E** : одиночный элемент внутри **D**.
 - Типы RAW. Используются для проверки среза байтов. Одиночные элементы из RAW можно извлечь с помощью метода `Lookup()` 

## Использование CRUD методов

После успешного соединения с базой можно добавлять и изменять данные в коллекции.
Тип **Collection** содержит в себе методы позволяющие отправлять запросы в базу данных.

### Вставка/создание документов
В коллекцию монго можно вставлять структуры Go.

#### Одиночная вставка

```go

type Message struct {
	id int
	mess string
}

newMes := Message{"id": 1, "mess": "new mess"}

insertResult, err := collection.InsertOne(context.TODO(), newMes)
if err != nil {
	log.Fatal
}

fmt.Println("Inserted a single document: ", insertResult.InsertedID)
```

#### Вставка нескольких документов

```Go

mes1 := Message{"id": 2, "mess": "new mess 2"}
mes2 := Message{"id": 3, "mess": "new mess 3"}
someMessages := []interface{}{mes1, mes2}

insertManyResult, err := collection.InsertMany(context.TODO(), someMessages)
if err != nil {
	log.Fatal(err)
}

fmt.Println("Inserted multiple documents: ", insertManyResult.InsertedIDs)
```

### Обновление документов

Метод `collection.UpdateOne()` позволяет обновить один документ. Требуется создать фильтр для поиска документа в базе данных и документ для операции обновления. Их можно создать используя типы `bson.D`:

```go
filter := bson.D{{"id", 1}}

update := bson.D{
	{"$inc", bson.D{
		{"id", 5}
	}}
}

// код найдет документ, в котором поле id = 1 и увеличит его на 5
updateResult, err := collection.UpdateOne(context.TODO(), filter, update) 
if err != nil {
	log.Fatal(err) 
	}

fmt.Printf("Matched %v documents and updated %v documents.\n", updateResult.MatchedCount, updateResult.ModifiedCount)
```

### Поиск документов

Чтобы найти документ, нужен фильтр и указатель на переменную, в которую может быть декодирован результат.

#### Поиск одного документа

Чтобы найти один документ, можно использовать `collection.FindOne` - этот метод возвращает одно значение, которое можно декодировать в переменную.

```Go
var result Message

err = collection.FindOne(context.TODO(), filter).Decode(&result)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Found a single document: %+v\n", result)
```

#### Поиск нескольких документов

`collection.Find()` - для поиска нескольких документов. Метод возвращает **Cursor** 

>[!info] Cursor
> Это поток документов, с помощью которого можно перебирать и декодировать по одному документу за раз. ==Когда документы в **Cursor** закончатся, нужно его закрыть==. Также **Cursor** можно тонко настроить используя пакет **options**.

```Go
// установлен лимит на выдачу двух документов
options := options.Find()
options.SetLimit(2)
filter := bson.M{}

cur, err := collection.Find(context.TODO(), filter, options) 
if err != nil { 
	log.Fatal(err) 
}

var results []*Message

for cur.Next(context.TODO()) {
	var elem Message
	err := cur.Decode(&elem)
	if err != nil {
		log.Fatal(err)
	}
	results = append(results, &elem)
}

if err := cur.Err(); err != nil {
	log.Fatal(err)
}

cur.Close(context.TODO())

fmt.Printf("Found multiple documents (array of pointers): %+v\n", results)
```

### Удаление

Можно удалить документы используя
 - `collection.DeleteOne()`
 - `collection.DeleteMany()`
 - `collection.Drop()` для удаления всей коллекции