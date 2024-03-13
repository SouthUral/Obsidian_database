# Описание
Паттерн __Singleton__ относится к порождающим паттернам уровня объекта. Паттерн контролирует создание единственного экземпляра некоторого класса (Типа) и предоставляет доступ к нему. __SIngleton__ гарантирует что экземпляр класса (значение типа) будет создан только один раз, и при попытке создать еще раз экземпляр класса будет возвращен уже созданный объект.

# Источники
[видео с объяснением](https://www.youtube.com/watch?v=WnzfShDnd9c)
[ссылка на репозиторий](https://github.com/AlexanderGrom/go-patterns/tree/master/Creational/Singleton)

# Реализация
```Go
package singleton

import (
	"sync"
)

// Singleton implementation
type Singleton struct {
}

var (
	instance *Singleton
	once sync.Once
)

func GetInstance() *Singleton {
	once.Do(
			func() {
				instance = &Singleton{}
			}
	)
	return instance
}
```