[лекция про интерфейсы от Даниила Подольского](https://www.youtube.com/watch?v=SZx5yuh3LPU)

Интерфейс в Go — это набор сигнатур методов (то есть список методов без реализации).

Для того чтобы соответствовать интерфейсу нужно реализовать все методы указанные в интерфейсе

>[!warning] Важно!!!
>zero value интерфейса это nil

# Свойства интерфей

Интерфейс геометрической фигуры:
```go
type geometry interface {
	area() float64
	perim() float64
}
```

Реализуем интерфейс в типе «прямоугольник». Реализовать интерфейс = реализовать его методы. Действует «утиный» принцип, как в питоне: если у типа есть перечисленные в интерфейсе методы — значит, он реализовал интерфейс. Явно указывать, что rect реализует geometry, не требуется:

    type rect struct {
        width, height float64
    }

    func (r rect) area() float64 {
        return r.width * r.height
    }

    func (r rect) perim() float64 {
        return 2*r.width + 2*r.height
    }

Аналогично для типа «круг»:

    type circle struct {
        radius float64
    }

    func (c circle) area() float64 {
        return math.Pi * c.radius * c.radius
    }

    func (c circle) perim() float64 {
        return 2 * math.Pi * c.radius
    }

Если у переменной интерфейсный тип, она поддерживает все методы, заданные на интерфейсе. Благодаря этому функция measure() работает с любой фигурой, реализующей интерфейс geometry:

    func measure(g geometry) {
        fmt.Printf("%T: %+v\n", g, g)
        fmt.Println("area:", g.area())
        fmt.Println("perimiter:", g.perim())
    }

Раз типы circle и rect реализуют интерфейс geometry, мы можем передать их экземпляры в функцию measure():

    r := rect{width: 3, height: 4}
    c := circle{radius: 5}

    measure(r)
    // main.rect: {width:3 height:4}
    // area: 12
    // perimiter: 14

    measure(c)
    // main.circle: {radius:5}
    // area: 78.53981633974483
    // perimiter: 31.41592653589793