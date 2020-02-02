# Урок 14: Кодогенерация

###  Где чаще всего используется
* Генерация структур на основе JSON
* Генерация заглушек для интерфейсов (моки для тестов)
* Генерация кода для Protobuf протокола
* Вставка бинарных данных (изображения, шаблоны и пр.) в код Go в виде `[]byte`
* Генерики (Generics)

### Go Generate
Простая программа из стандартного пакета: [Go Generate source](https://golang.org/pkg/cmd/go/internal/generate/).
Она проходит по всем файлам в пакете, ищет комментарии начинающиеся с `//go:generate` (без пробела) и выполняет код
описанный в них как системную команду:
```go
//go:generate echo "Hello World"
package main

import "fmt"

func main() {
	fmt.Println("Run any unix command in go:generate")
}

// > go generate .
// Hello, World
```

Псевдоним команды (alias):
```go
//go:generate -command bye echo "Goodbye Go!"
//go:generate bye

> go generate .
Goodbye Go!
```

Регулярные выражения (regexp):
```
go generate -run enums
```

Добавляет к выводу имя файла в которых была команда:
```
go generate -v
```

Добавляет к выводу команду:
```
go generate -x
```

Вывести список команд к выполнению:
```
go generate -n
```

### Основные принципы работы
1. `go generate` запускается разработчиком программы/пакета, а не пользователем
2. Инструментарий для `go generate` находится у создателя пакета
3. Генерация кода не должна происходить автоматически во время `go build`, `go get`, `go test` и т.п.
4. Генерация кода должна вызываться явно
5. Инструменты генерации кода «невидимы» для пользователей и могут быть недоступны для него
6. `go generate` работает только с go-файлами, как часть тулкита Go
7. Не забывайте добавлять disclaimer: `^// Code generated .* DO NOT EDIT\.$`

Более детально: [Go generate: A Proposal](https://docs.google.com/document/d/1V03LUfjSADDooDMhe-_K59EgpTEm3V8uvQRuNMAEnjg/edit)

## Свой генератор
Мы можем вызывать генератором выполнение любых Go файлов:
```
go generate go run generator.go
```

В таком файле можно, к примеру, создавать новый файл на основе любых данных. А чтобы он не добавлялся в общую сборку
необходимо добавить в начале специальный комментарий ([go build](https://golang.org/pkg/go/build/)):
```go
// +build ignore

package main

// Вторая функция main в пакете, будет игнорироваться
func main() {
	...
}
``` 

## Интересные примеры библиотек
### Stringer
Генерирует имплементацию стандартного интерфейса `Stringer` для значений на основе набора int’ов:
```go
go get golang.org/x/tools/cmd/stringer

func (t T) String() string

//go:generate stringer -type=MessageStatus
type MessageStatus int

const (
	Sent MessageStatus = iota
	Received
	Rejected
)

func main() {
	status := Sent
	fmt.Printf("Message is %s", status) // Message is Sent
}
```

### JSON Enums
Делает практически тоже самое что и `Stringer`, но только для JSON:
```go
go get github.com/campoy/jsonenums

func (t T) MarshalJSON() ([]byte, error)
func (t *T) UnmarshalJSON ([]byte) error

//go:generate jsonenums -type=Status
type Status int

const (
	Pending Status = iota
	Sent
	Received
	Rejected
)
```

### Generics
```
go get github.com/cheekybits/genny

go:generate genny -in=$GOFILE -out=gen-$GOFILE gen "KeyType=string,int ValueType=string,int"
```

Объявляем заглушки по типам:
```go
type KeyType generic.Type
type ValueType generic.Type

// Пишем обычный код:
func SetValueTypeForKeyType(key KeyType, value ValueType) {
	...
}
```

Другие примеры кодогенераторов: [Awesome Go: Generation and Generics](https://github.com/avelino/awesome-go#generation-and-generics)

## ldflags
```go
package main
import "fmt"

var VersionString = "unset"

func main() {
	fmt.Println("Version:", VersionString)
}

go run -ldflags '-X main.VersionString=1.0' main.go
```

[<< Предыдущая](13-reflection.md) | [Оглавление](../readme.md)
