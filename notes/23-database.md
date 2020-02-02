# Урок 23: Работа с SQL

**Data Definition Language (DDL)** - язык для объявления данных, например операторы create table, create index и другие.
**Data Manipulation Language (DML)** - язык для управления данными, например операторы insert, update, delete и другие.
**Data Query Language (DQL)** - язык для получения данных, чаще всего это оператор select.

**Data Source Name (DSN)** - строка подключения к БД, содержит все необходимые операции. Синтаксис DSN зависит
от используемой базы данных и драйвера.

Примеры для PostgreSQL ([подробнее](https://godoc.org/github.com/lib/pq)):
```
"postgres://user:pass@localhost:5432/mydb?sslmode=verify-full"
"user=user dbname=mydb sslmode=verify-full password=pass"
```

### Подключение к БД (простейший вариант)
Создание нового подключения:
```go
import "database/sql"
import _ "github.com/jackc/pgx/stdlib"

dsn := "..."
db, err := sql.Open("pgx", dsn) // *sql.DB
if err != nil {
	log.Fatalf("failed to load driver: %v", err)
}
// создан пулл соединений
```

Использование подключения:
```go
err := db.PingContext(ctx)
if err != nil {
	return errors.Errorf("failed to connect to db: %v", err)
}
// работаем с db
```

### Пулл соединений
`sql.DB` - представляет собой пулл соединений с базой данных. Новые соединения будут открываться по мере необходимости.
Как следствие он безопасен для конкурентного использования (также как и `http.Client`).

Основные настройки:
```go
// Макс. число открытых соединений от этого процесса
db.SetMaxOpenConns(n int)
// Макс. число открытых неиспользуемых соединений
db.SetMaxIdleConns(n int)
// Макс. время жизни одного подключения
db.SetConnMaxLifetime(d time.Duration)
```

![Database: Connection Pool](../images/db-connection-pool.png)

Более продвинутый пример подключения с пуллом и pgx:
```go
import (
	"context"
	"github.com/jackc/pgx/v4/log/logrusadapter"
	"github.com/jackc/pgx/v4/pgxpool"
	"github.com/pkg/errors"
	"github.com/sirupsen/logrus"
	"net"
	"time"
)

func Postgres(ctx context.Context, logger logrus.FieldLogger, dsn string) (*pgxpool.Pool, error) {
	cfg, err := pgxpool.ParseConfig(dsn)
	if err != nil {
		return nil, errors.Wrap(err, "failed to parse DSN")
	}

	cfg.MaxConns = 8
	cfg.ConnConfig.TLSConfig = nil
	cfg.ConnConfig.PreferSimpleProtocol = true
	cfg.ConnConfig.RuntimeParams["standard_confirming_string"] = "on"
	cfg.ConnConfig.Logger = logrusadapter.NewLogger(logger)
	cfg.ConnConfig.DialFunc = (&net.Dialer{
		KeepAlive: 5 * time.Minute,
		Timeout:   1 * time.Second,
	}).DialContext

	pool, err := pgxpool.ConnectConfig(ctx, cfg)
	if err != nil {
		return nil, errors.Wrap(err, "failed to connect to postgres")
	}

	return pool, nil
}
```

Попробовать: https://github.com/go-pg/pg

[<< Предыдущая](22-grpc.md) | [Оглавление](../readme.md)
