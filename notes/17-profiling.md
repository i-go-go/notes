# Урок 17: Профилирование и оптимизация

## Бенчмарки
Бенчмарки пишутся в файлах тестов. Только в качестве параметра передается:
```go
import "testing"

func BenchmarkExample(b *testing.B) {
	for i := 0; i < b.N; i++ {
		SomeMethod()
	} 
}
```

Запуск бенчмарков:
```
go test -bench=.
	-cpu=2					# Кол-во ядер процессора
	-benchmem \				# Добавить замеры памяти
	-cpuprofile=cpu.out \	# Вывод данных по CPU в файл
	-memprofile=mem.out		# Вывод данных по памяти в файл
```

Пример как запускать в параллельном режиме:
```go
func BenchmarkParallel(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			SomeMethod()
		}
	})
}
```

Небольшой совет по переиспользованию:
```go
type A struct {
	I int
}

func (a *A) Reset() {
	*a = A{}
}

func BenchmarkExample(b *testing.B) {
	a := &A{}
	for i := 0; i < b.N; i++ {
		a.Reset()
		SomeMethod(a)
	} 
}
```

Чем больше у нас указателей на heap`е, тем медленнее работает Garbage Collector. 
```go
const numElements = 1000000
var foo = map[string]int{}

func timeGC() {
	t := time.Now()
	runtime.GC()
	fmt.Printf("GC took: %s\n", time.Since(t))
}

func main() {
	for i := 0; i < numElements; i++ {
		foo[strconv.Itoa(i)] = i
	}
	for {
		timeGC()
		time.Sleep(1 * time.Second)
	}
}

// Из-за большого heap сборка мусора может достигать 1 сек
// GC took: 254.319084ms
// GC took: 349.284017ms
// GC took: 619.572034ms

var foo = map[int]int{}
// Заменим тип string на int в мапе
// GC took: 4.496822ms
// GC took: 3.712796ms
// GC took: 2.845791ms
// В цикле мы вообще не используем мапу, но сам факт наличия
// ее в памяти заставляет GC выполнять работу. Int намного
// меньше string занимает места в памяти. Отсюда и снижение
// времени работы GC. Он дольше проверяет большое
// количество ссылок. Такой же эффект не только с ключами,
// но со значениями
```

### HTTP
По умолчание http-клиент не поддерживает режим Keep-Alive для соединения. Для этого необходимо создать свой клиент
и указать у него пустой транспорт:
```go
client = http.Client{
	Timeout: 3 * time.Second,
	Transport: &http.Transport{},
}
client.Get("http://your-api.com/v1/list.json")
```

ToDo: Доразобрать тему

[<< Предыдущая](15-internals-memory.md) | [Оглавление](../readme.md)
