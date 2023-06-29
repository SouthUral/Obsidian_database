[[Golang]]
## fmt


## strings
Используется для работы со строками
```go
// обрезает символы, переданные вторым аргументом, с обеих сторон строки
strings.Trim(s, cutset string) string
strings.Trim(" hello ", " ") // "hello"

// преобразует все буквы в строке в нижний регистр
strings.ToLower(s string) string
strings.ToLower("пРиВеТ") // "привет"

// озаглавливает первую букву в каждом слове в строке
strings.Title(s string) string
strings.Title("привет, джон") // "Привет, Джон"

// Для замены символов в строке существует функция ReplaceAll(s, old, new string) string 
strings.ReplaceAll("hello world!", "world", "buddy") // hello buddy!
strings.ReplaceAll("one two three", " ", "_") // one_two_three

// Наличие пробелов можно проверить с помощью функции
strings.Contains(firstName, " ")

// проверяет наличие подстроки в строке
strings.Contains("hello", "h") // true

// разбивает строку по Юникод символам или по переданному разделителю
strings.Split("hello", "") // ["h", "e", "l", "l", "o"]

// склеивает строки из слайса с разделителем
strings.Join([]string{"hello", "world!"}, " ") // "hello world!"

// склеивает большой объем строк быстро
sb := strings.Builder{}

sb.WriteString("hello")
sb.WriteString(" ")
sb.WriteString("world")

sb.String() // "hello world"
если работа идет с рунами то нужно использовать такие конструкции
sb.WriteRune(rune)
```

---
## strconv
Библиотека для конвертации чисел в строку и наоборот
Пример:

    import "strconv"

    s := strconv.Itoa(-42) // "-42"

    func IntToString(num int) string {
        return strconv.Itoa(num)
    }

преобразовать строку в число:

    s := strconv.Atoi(string(num_byte))

## math/rand

math.Max() сравнивает числа и возвращает наибольшее
math.Min() сравнивает числа и возвращает наименьшее

Пример:

    unc MinInt(x, y int) int {
        return int(math.Min(float64(x), float64(y)))
    }

функция math.Min() принимает значения типа float64, и для использования функции нужно привести значения к float64

func Shuffle (n int , swap func (i, j int ))
Shuffle псевдорандомизирует порядок элементов, используя источник по умолчанию. n - количество элементов. Shuffle вызывает панику, если n < 0. swap меняет местами элементы с индексами i и j.

    import (
        "fmt"
        "math/rand"
        "strings"
    )

    func main() {
        words := strings.Fields("ink runs from the corners of my mouth")
        rand.Shuffle(len(words), func(i, j int) {
            words[i], words[j] = words[j], words[i]
        })
        fmt.Println(words)
    }

## unicode

Пакет unicode предоставляет данные и функции для проверки некоторых свойств кодовых точек Unicode.

    MaxASCII = '\u007F'      // максимальное значение ASCII.

    func IsASCII(s string) bool {
	for _, r := range s {
		if r > unicode.MaxASCII {
			return false
		}
	}

    return true
    }

    // проверяет, является ли руна латинским симоволом
    unicode.Is(unicode.Latin, rune)
