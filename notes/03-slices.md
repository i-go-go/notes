# Урок 3: Массивы и слайсы

## Массив
**Массив** - нумерованная последовательность элементов фиксированной длинны. Массив располагается _последовательно_
в памяти и не меняет своей длинны.

```go
var arr [256]int        // фиксированная длинна
var arr [10][10]string  // может быть многомерным
arr := [10]int{1,2,3,4,5}
arr := [...]{1,2,3}     // длинна будет вычисленна при инициализации
```

Длинна массива - часть типа, т.е. массивы разной длинны имеют разные типы данных.

Операции над массивами:
```go
arr[3] = 1    // индексация
len(arr)      // длинна массива
arr[3:5]      // получение слайса
```

## Слайсы
**Слайсы** - это те же "массивы", но переменной длинны.

Создание слайсов:
```go
var s []int             // не-инициализированный слайс, nil
s := []int{}            // с помощью литерала слайса
s := make([]int, 3)     // с помощью функции make, s == {0,0,0}
s := make([]int, 3, 10) 
```

Мы можем создавать не только многомерные массивы, но и многомерные слайсы. Но они отличаются по работе "под капотом".
Многомерный массив будет представлять из себя отдельный кусок памяти, просто перемещаться по нему мы будем
по n-измерениям. Многомерный слайс работает по иному, для каждой строки создается отдельный слайс, а потом эти
объекты ложатся в другой слайс (слайс слайсов). И в памяти это лежит не последовательно, а как получится.

### Добавление в слайс
Добавление новых элементов в слайс происходит при помощи функции `append`:

```go
s[i] = 1                // работает если i < len(s)
s[len(s) + 10] = 1      // вылетит panic
s = append(s, 1)        // добавление 1 в конец слайса
s = append(s, 1, 2 ,3)  // добавлние 1,2,3 в конец слайса
s = append(s, s2...)    // добавляет содержимое слайса s2 в конец s
var s []int             // s == nil
s = append(s, 1)        // s == {1}, append умеет работать с nil-слайсами
```

### Получение под-слайсов (нарезка)
`s[i:j]` - возвращает под-слайс с i-го элемента включительно, по j-ый не включительно. Длинна нового слайса
будет `j-i`.

```go
s := []int{0,1,2,3,4,5,6,7,8,9}
s2 := s[:]    // копия s (shallow)
s2 = s[3:5]   // []int{3,4}
s2 = s[3:]    // []int{3,4,5,6,7,8,9}
s2 = s[:5]    // []int{0,1,2,3,4}
```

**Важная особенность:** такие слайсы используют общую память. Поэтому слайс `s2` будет использовать ту же область
памяти что и `s`, но будет иметь иные границы.

### Авто-увеличение слайса
Если `len < cap` - увеличивается `len`. 
Если `len = cap` - увеличивается `cap`, выделяется новый кусок памяти и данные копируются. `cap` каждый раз
увеличивается в 2 раза ([пример как это работает](https://play.golang.org/p/g7cjWi_dF9F)).

### Отличия new от make
ToDo: Поискать детальное инфо

[<< Предыдущая](02-strings.md) | [Оглавление](../readme.md) | [Следующая >>](08-goroutines.md)
