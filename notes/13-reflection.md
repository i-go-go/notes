# Урок 13: Рефлексия

Рефлексия - это возможность в программе исследовать и изменять свою структуру (в частности переменные) на этапе выполнения.

Некоторые возможности вынесены на уровень языка.  
Например type assertion или type switch:
```go
// type assertion
var r io.Readervar f *os.Filef, ok = r.(*os.File)

type switch
switch v := i.(type) {
	case int: // here v has type int
		i = v + 1
	case string: // here v has type string
		i = v + "1"
	default: // no match; here v has the same type as i
}
```

Пакет `reflect` в Go представляет API для работы с переменными заранее неизвестных типов.  
Почитать: [The Laws of Reflection - The Go Blog](https://blog.golang.org/laws-of-reflection)

## reflect.Value
Значения типа `reflect.Value` представляют собой программную обертку над значением произвольной переменной.
```go
var i int = 42
var s = struct{string; int}{"hello", 42}

iv := reflect.ValueOf(i)    // тип reflect.Value
sv := reflect.ValueOf(&s)   // тип reflect.Value
```

Какие методы есть у `reflect.Value`?
```go
// вернуть тип обертку над типом
value.Type() reflect.Type
// вернуть «базовый» тип
value.Kind() reflect.Kind

// // вернуть обернутое значение как interface{}
value.Interface() interface{}
// вернуть значение как int64
value.Int() int64
// вернуть значение как string
value.String() string

// возможно ли изменить значение ?
value.CanSet() bool
// установить значение типа int64
value.SetInt(int64)
// разыменование указателя или интерфейса
// (переходит по указателю)
value.Elem() reflect.Value 
```

## reflect.Kind и reflect.Type
`reflect.Kind` представляет собой базовый тип для значения. `reflect.Kind` определяет какие методы имеют смысл
для конкретного `reflect.Value`, а какие вызовут панику.

`reflect.Type` представляет собой информацию о конкретном типе: имя, пакет, список методов и т.д. Большинство
методов продублировано в `reflect.Value`.

```go
t.Name() string             // имя типа
t.PkgPath() string          // пакет, в котором определен тип
t.Size() uintptr            // размер в памяти, занимаемый значением
t.Implements(u Type) bool   // реализует ли интерфейс u 
t.MethodByName(string name) reflect.Value // метод по имени
```

### Пример изменения значений
Неправильный способ:
```go
var x float64 = 3.4
v := reflect.ValueOf(x) // ???
v.SetFloat(7.1)         // panic: reflect.Value.SetFloat using unaddressable value
fmt.Println(v.CanSet()) // false
```

Правильный способ:
```go
var x float64 = 3.4
p := reflect.ValueOf(&x)    // адрес переменной x
fmt.Println(p.Type())       // *float64
fmt.Println(p.CanSet())     // false
v := p.Elem()               // переход по указателю
fmt.Println(v.Type())       // float64
fmt.Println(v.CanSet())     // true
v.SetFloat(7.1)
fmt.Println(x)              // 7.1
```

`reflect.Value.Elem()` - переходит по указателю или к базовому объекту интерфейса.

## Работа со структурами
Если `v` это рефлексия значения структуры (`reflect.Value`), то:
```go
v.NumField() int                // возвращает кол-­во полей в структуре
v.Field(i int) reflect.Value    // возвращает рефлексию для отдельного поля
v.FieldByName(s string) reflect.Value // тоже, но по имени поля
```

Если `t` это рефлексия типа структуры `t := v.Type()`, то:
```go
t.NumField() int    // возвращает кол­во полей в структуре
// возвращает рефлексию для конкретного поля
t.Field(i int) reflect.StructField
// тоже, но по имени поля
t.FieldByName(name string) (reflect.StructField, bool) 
```

Свойства `reflect.StructField`:
```go
Name string             // имя поля
Type reflect.Type       // рефлексия типа поля
Tag reflect.StructTag   // описание тэгов конкретного поля
Offset uintptr          // смещение в структуре
...
```

Преобразование структуры в map [Go Playground](https://play.golang.org/p/B7QEHLgNSTG):
```go
func structToMap(iv interface{}) (map[string]interface{}, error) {
	v := reflect.ValueOf(iv)
	if v.Kind() != reflect.Struct {
		return nil, errors.New("not a struct")
	}

	t := v.Type()
	mp := make(map[string]interface{})
	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i) // reflect.StructField
		fv := v.Field(i)    // reflect.Value
		mp[field.Name] = fv.Interface()
	}
	return mp, nil
}
```

Вычитывание map в структуру [Go Playground](https://play.golang.org/p/6bdq9ZkPCY0):
```go
func mapToStruct(mp map[string]interface{}, iv interface{}) (error) {
 	v := reflect.ValueOf(iv)
	if v.Kind() != reflect.Ptr {
		return errors.New("not a pointer to struct")
	}
	v = v.Elem()
	if v.Kind() != reflect.Struct {
		return errors.New("not a pointer to struct")
	}
	t := v.Type()
	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i) // reflect.StructField
		fv := v.Field(i)    // reflect.Value
		if val, ok := mp[field.Name]; ok {
			mfv := reflect.ValueOf(val)
			if mfv.Kind() != fv.Kind() {
				return errors.New("incomatible type for " + field.Name)
			}
			fv.Set(mfv)
		}
	}
	return nil
}
```

## Работа с функциями
Получив рефлексию функции/метода мы можем исследовать количество и типы ее аргументов:
```go
f := fmt.Printf
v := reflect.ValueOf(f)
t := v.Type()

t.NumIn()       // количество аргументов
t.NumOut()      // количество возвращаемых значений
a1t := t.In(0)  // reflect.Type первого аргумента
o1t := t.Out(0) // reflect.Type первого возвращаемого значения
t.IsVariadic()  // принимает ли функция переменное число аргументов ?
```

### Получение списка методов типа
Мы можем получить список методов определенных над типом:
```go
type Int int
func (i Int) Say() string {
	return"42"
}

func main() {
	var obj Int
	v := reflect.ValueOf(obj)
	for i := 0; i < v.NumMethod(); i++ {
		meth := v.Method(i)     // reflect.Value
	}
	sayMethod := v.MethodByName("Say")  // reflect.Value
```

### Вызов функций / методов
В случае неправильного количества / типа аргументов случится паника.
```go
func main() {
	f := fmt.Printf
	v := reflect.ValueOf(f)
	args := []

	reflect.Value{
		reflect.ValueOf("test %d\n"),
		reflect.ValueOf(42),
	}
	ret := v.Call(args) // []reflect.Value
	fmt.Println(ret)
}
```

## Указатели и Unsafe Pointers
В Go указатели на разные типы несовместимы между собой (т.к. сами являются разными типами). Однако
тип `unsafe.Pointer` является исключением. Компилятор Go позволяет делать явное преобразование типа любого указателя
в `unsafe.Pointer` и обратно (а также в `uintptr`) - [Type-Unsafe Pointers](https://go101.org/article/unsafe.html):

```go
import "unsafe"

var b [8]byte	// массив
bp := &b
var sp *St
var up unsafe.Pointer
up = unsafe.Pointer(bp)
sp = (*St)(up)
sp.a = 12345678
fmt.Println(b) // [78 97 188 0 0 0 0 0]
```

[<< Предыдущая](12-os.md) | [Оглавление](../readme.md)
