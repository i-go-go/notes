# Урок 1: Инструменты разработчика

## Шпаргалка по инструментам
[![][pdf]Go Tooling Cheatsheet](../extra/go_tooling_cheatsheet.pdf)

## gofmt и go fmt
Это стандартная утилита из пакета go, которая проверяет соответствие стилю и может автоматически его исправлять:
```
// Выведет список файлов в которых gofmt увидел
// необходимость изменнеий
> gofmt -l .

// Выведет инфо об изменениях в виде patch
> gofmt -d main.go

// Производит форматирование в режиме simplification,
// т.е. упрощает код
> gofmt -s main.go

// Произвести форматирование в файле/файлах (переписать)
> gofmt -w main.go
> gofmt -w -s .

// Рефакторинг при помощи утилиты (заменяем название метода)
> gofmt -r 'Greet > Great' main.go
```

Утилита `go fmt` эквивалентна утилите с ключами `gofmt -w -l`

## go vet
Утилита [go vet](https://golang.org/cmd/vet/) — часть стандартного пакета и рекомендована к использованию командой Go.
Проверяет ряд типичных ошибок, например:
* неправильное использование `Printf` и аналогичных функций
* некорректные build теги
* сравнение function и nil

```
> go vet foo.go         // Анализ файла foo.go
> go vet .              // Анализ всех файлов в папке
> go vet ./...          // Анализ включая поддиректории

// Отключение некоторых анализаторов
> go vet -composites=false foo.go

// Отключение vet проверок при запуске тестов
> go test -vet=off ./...
```

## golint
Golint разработан командой Go и проверяет код на основе документов [Effective Go](https://golang.org/doc/effective_go.html)
и [CodeReviewComments](https://golang.org/wiki/CodeReviewComments) :

```
> golint foo.go         // Анализ файла foo.go
> golint .              // Анализ всех файлов в папке
> golint ./...          // Анализ включая поддиректории
```

## Мета-линтеры
Представляет собой набор из различных линтеров, которые можно гибко настраивать по своему усмотрению. Рекомендуется
применять для полноценной проверки проектов. Более детальная информация и примеры на [официальном сайте](https://golangci.com/).

## go build
Билдинг под различные ОС и архитектуры:
```
// Варианты GOOS: darwin linux windows
// Варианты GOARCH: 386 amd64
export GOOS=linux
export GOARCH=amd64
go build -mod vendor -o bin/myapp-$(GOOS)-$(GOARCH)
```

При билде мы при помощи включаем флага `-mod vendor`  чтение библиотек из локальной папки _vendor_. А при помощи
флага `-o` мы говорим куда положить скомпилированный бинарник.

Пример _Makefile_ со сборкой под все платформы и архитектуры:
```makefile
BINARY=honeydoc
PLATFORMS=darwin linux windows
ARCHITECTURES=386 amd64

build:
	$(foreach GOOS, $(PLATFORMS),\
    $(foreach GOARCH, $(ARCHITECTURES),\
    $(shell export GOOS=$(GOOS); export GOARCH=$(GOARCH); go build -mod vendor -o bin/$(BINARY)-$(GOOS)-$(GOARCH))))
```

`go build -gcflags="-m -m" .` - покажет какие оптимизации производит компилятор, что уходит в heap и т.д.

`go build -ldflags="-X main.version=1.2.3"` - говорит линковщику выставить при компиляции кастомные данные.
К примеру, установить в переменную `version` значение `1.2.3`. Или можно указать текущий коммит при билде:
`git rev-parse --short HEAD`

## go install
Компилирует программу через `go build` и после устанавливает софтину в `$GOPATH/bin/`. Для того чтобы команда
работала обязательно должна быть задана переменная окружения `GOBIN`.

## go list
Позволяет получить информацию об используемых пакетах:
```
// Возвращает список импортов в виде слайса
> go list -f '{{ .Imports }}
[fmt log net/http strconv]

// Возвращает импорты по одному в строку (Гошный шаблон)
> go list -f '{{ join .Imports "\n" }}'
fmt
log
net/http
strconv

// Вывести все данные в формате JSON в less
> go list -json | less

// Вывести информацию о пакете из текущей папки и подпапок
> go list ./...
``` 

## go test
Тестирование кода
```
// Запускает тесты для пакета и всех его зависимостей
> go test all

// Запуск тестов с выводм оценки покрытия кода
// для текущего пакета и всех подпакетов
> go test -cover ./...

// Выведет в браузере информацию о покрытии кода
> gotest -coverprofile=/tmp/profile.out
> go tool cover -html=/tmp/profile.out
```

## benchmarks
Тестирование производительности вашего кода:
```
// Запуск тестов с бенчмарком
> go test -bench=.

// Запуск с тестированием памяти (кол-во аллокаций)
> go test -bench=. -benchmem

// Запуск бенчмарков на 1, 4 и 8 ядрах
> go test -bench=. -cpu=1,4,8
```

ToDo: Законспектировать детальнее [Бенчмарки в Go / Хабр](https://habr.com/ru/post/268585/)

## stress
Утилита для многократного запуска тестов. Например, в тестах есть «плавающая» ошибка, которая воспроизводится крайне
редко. Эта утилита отловит все фейлы тестов ([GoDoc: stress](https://godoc.org/golang.org/x/tools/cmd/stress)):
```
// Компилируем тесты
> go test -c -o=/tmp/test.out

// Запускаем стресс-тест
> stress -p=<кол-во потоков> /tmp/test.out
```

[Оглавление](../readme.md) | [Следующая >>](02-strings.md)

[pdf]: ../images/pdf.png ""
