# Урок 20: Низкоуровневые протоколы TCP, UDP, DNS

## Контекст
По сути это некая сущность, которая характеризует конкретный запрос. Обычно он служит для передачи данных в горутину
или же для ее остановки. Является частью стандартной библиотеки:
```go
import "context"

// Дефолтный пустой контекст
func Background() Context

// Пустой контекст, но с пометкой что его необходимо реализовать в будущем
// Например если еще не решили какой именно контекст будет
func TODO() Context

// Служит для ручной отмены контекста через вызов функции
// Например, если есть пулл коннектов, и необходимо при завершении одного отменить все остальные
func WithCancel(parent Context) (Context, CancelFunc)

// Передается к какому моменту времени контекст перестает существовать
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)

// Передается через какое время контекст перестает существовать (под капотом WithDeadline)
func WithTimeout(parent Context, timeout time.Time) (Context, CancelFunc)

func WithValue(parent Context, key interface{}, value interface{}) Context
```

Контексты вкладываются друг в друга (как матрёшка). 
Попробуем создать простой пример для демонстрации работы с контекстами:
```go
import "math/rand"

func main() {
	wg := &sync.WaitGroup()
	ctx := context.Background()
    // Устанавливаем в контексте лимит на 2 секунды
    // Второй возвращаемый аргумент - cancelFunc
    // При помощи него можно прервать вручную
	ctx, _ = context.WithTimeout(ctx, time.Second * 2)
	wg.Add(1)
	go dealLongWithCtx(wg, ctx)
	wg.Wait()
}

func dealLongWithCtx(wg *sync.WaitGroup, ctx context.Context) {
	defer wg.Done()
    // Генерируем рандомное время работы
    // Устанавливаем seed (дефолтный одинаковый в math/rand)
	r := rand.New(rand.NewSource(time.Now.UnixNano()))
	randTime := time.Duration(r.Intn(4000)) * time.Millisecond
	fmt.Printf("Duration: %s\n", randTime)
	timer := time.NewTimer(randTime)

	// Читаем несколько каналов. Если работа занимает более
	// 2 секунд (timeout), то отработает ctx.Done
	// Иначе отработает функция по таймеру
	select {
	case <-ctx.Done():
		fmt.Println("Rejected by timeout")
	case <-timer.C:
		fmt.Println("Done!")
	}
}

// Run 1
// Duration: 3.078s
// Rejected by timeout

// Run 2
// Duration: 1.829s
// Done!
```

## Протокол UDP
* Доставка пакета не гарантируется
* Порядок сообщений не гарантируется
* Соединение не устанавливается (работает с датаграммами)
* Как следствие, он быстрый
* Подходит для потокового аудио/видео, статистики, игр и т.д.

## Протокол TCP
* Доставка пакетов гарантируется (или получим ошибку)
* Порядок пакетов гарантируется
* Соединение устанавливается
* Есть Overhead, поэтому медленнее UDP
* Подходит для http, электронной почты и т.д. и не подходит для realtime функционала.

## Протокол DNS
* Служит для получения IP адреса для доменного имени (и не только)
* Работает как поверх UDP, так и поверх TCP
* Имеет рекурсивную природу
* Имеет механизмы кэширования
* При высоких нагрузках можно использовать `/etc/hosts`

## Пакет net в Go
Для работы с сетью в языке используется пакет `net` и его подпакеты.

### Dialer
Особый тип, задача которого установка соединений. Имеет интерфейс:
```go
func (d *Dialer) Dial(network, address string) (Conn, error)
func (d *Dialer) DialContext(ctx context.Context, network, address string) (Conn, error)
func (d *Dialer) DialTimeout(network, address string, timeout time.Duration) (Conn, error)
```

### Conn
Является абстракцией над поточным сетевым соединением. Имплементирует `Reader`, `Writer`, `Closer`. Это потокобезопасный
тип, т.е. позволяет использовать его одновременно в нескольких горутинах.

## Типичные сетевые проблемы
1. Потеря пакетов
2. Недоступность
3. Тайм-ауты
4. Медленные соединения

Некоторые рекомендации по решению этих проблем:
* Не забываем добавлять таймауты на соединения
* Мониторинг всего и вся (подробнее в лекции по микросервисам)
* Использование Nginx для борьбы с медленными соединениями. В целом Go справляется с ними, но Nginx является более надежным средством.
* Использование инструментов для отладки сетевых проблем

### Инструменты для отладки
* **tcpdump и wireshark** - снифферы для анализа пакетов и извлечения из них полезной информации
* **lsof**
* **netstat**

Пример получения данных при помощи tcpdump:
```
tcpdump src 5.61.37.54 -vv
```

[<< Предыдущая](17-profiling.md) | [Оглавление](../readme.md) | [Следующая >>](21-http.md)