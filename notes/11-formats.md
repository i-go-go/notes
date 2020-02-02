# Урок 11: Форматирование данных

### Кодировка quoted-printable (QP)
Старый формат, используется при отправке Email. Главная проблема - избыточность (~300%) + проблема null-байта (/x00):

```
Content-Disposition: inline
Content-Transfer-Encoding: quoted-printable
Content-Type: text/plain; charset="UTF-8"

=D0=91=D0=BE=D0=BB=D1=8C=D1=88=D0=B8=D0=BD=D1=81=D1=82=D0=B2= =D0=BA==D0=BE=D0=BD=D1=84=D0=B5=D1=80=D0=B5=D0=BD=D1=86=D0
=B8==D1=81=D1=82==D0=B0=D1=80=D1=82=D1=83=D0=B5=D1=82 =D1=80=D0=B0=D0=BD=D0=BE =D1=83=D1=82==D1=80=D0=BE=D0=BC, =D0=BA
=D0=BE=D0=B3=D0=B4=D0=B0 =D1=83 =C2=AB=D1=81=D0==BE=D0=B2=C2=BB =0D=0A=D0=B5=D1=89=D1=91 =D1=81=D0=BB=D0=B8=D0=BF=D0=B0
=D1==8E=D1=82=D1=81=D1=8F =D0=B3=D0=BB=D0=B0=D0=B7=D0=B0=0D=0A=0D=0A=D0=9C=D1==8B =D1=83=D1=81=D1=82=D0=B0=D0=BB=D0
```

### Кодировка Base64
Более распространенная кодировка. При кодировании 3 символов используется 4 байта. В частности не редко можно встретить
в конце закодированной строки один или два символа «равно» (=). Это своего рода символ-заглушка чтобы сделать
закодированный блок символов равным 3.

```
TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5IGhpcyByZWFzb24sIGJ1dCBieSB0aGlzIHNpbmd1bGFyIHBhc3Npb24gZnJvbSBvdGhlciBhbmlt
YWxzLCB3aGljaCBpcyBhIGx1c3Qgb2YgdGhlIG1pbmQsIHRoYXQgYnkgYSBwZXJzZXZlcmFuY2Ugb2YgZGVsaWdodCBpbiB0aGUgY29udGludWVkIGFuZCBp
bmRlZmF0aWdhYmxlIGdlbmVyYXRpb24gb2Yga25vd2xlZGdlLCBleGNlZWRzIHRoZSBzaG9ydCB2ZWhlbWVuY2Ugb2YgYW55IGNhcm5hbCBwbGVhc3VyZS4=
```

Недостатки:
* Избыточность (примерно 33%)
* Проблемы с поточным преобразованием (если текст больше оперативной памяти, то мы не поместимся) 

В Go можно поточно кодировать данные:
```go
package main

import (
	"encoding/base64"
	"os"
)

func main() {
	input := []byte("foo\x00bar")
	encoder := base64.NewEncoder(base64.StdEncoding, os.Stdout)
	encoder.Write(input)
	// Must close the encoder when finished to flush any
	// partial blocks. If you comment out the following line,
	// the last partial block "r" won't be encoded.
	encoder.Close()
}
```

Здесь критически важным является `encoder.Close()`, поскольку именно эта часть закрывает кратность строки символами «равно».

## XML (SAX)
Что делать, если к нам приходит сверхогромный XML файл? Использовать Simple API XML (SAX).

```go
// Полный пример: https://play.golang.org/p/cuSIsVyZpD-
for {
	token, _ := decoder.Token()
	switch se := token.(type) {
	case xml.StartElement:
		fmt.Printf("Start element: %v Attr %s\n", se.Name.Local, se.Attr)
		inFullName = se.Name.Local == "FullName"
	case xml.EndElement:
		fmt.Printf("End element: %v\n", se.Name.Local)
	case xml.CharData:
		fmt.Printf("Data element: %v\n", string(se))
		if inFullName {
			names = append(names, string(se))
		}
	default:
		fmt.Printf("Unhandled element: %T", se)
	}
}
```

## EasyJSON
Библиотека от Маил Ру для очень быстрой работы с JSON. Она, как и ProtoBuf, использует внутри кодогенерацию вместо рефлексии.
[mailru/easyjson: Fast JSON serializer for golang](https://github.com/mailru/easyjson) 

## MessagePack
**MsgPack** - бинарный формат для передачи данных (см. [MessagePack: It’s like JSON. but fast and small.](https://msgpack.org/)).
Пример библиотеки для работы с данным форматом: [vmihailenco/msgpack](https://github.com/vmihailenco/msgpack), но есть и другие.

Преимущества по сравнению с ProtoBuf:
* Проще в использовании (как JSON)
* В нем может быть гибкая схема

## ProtoBuf
**ProtoBuf** - бинарный протокол от Google для передачи данных (см. [Google Protocol Buffers](https://developers.google.com/protocol-buffers/)).
Он позволяет делать более компактной передачу данных. Часто используется вместе с gRPC.

[<< Предыдущая](10-io.md) | [Оглавление](../readme.md)
