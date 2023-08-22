## 1. Design DB Schema và generate SQL code bằng dbdiagram.io 
### SQL:

```
Table accounts as A {
  id bigserial [pk]
  owner varchar [not null]
  balance bigint [not null]
  currency varchar [not null]
  created_at timestamp [not null, default: `now()`]

  Indexes {
    owner
  }
}

Table entries {
  id bigserial [primary key]
  account_id bigint [ref: > A.id, not null]
  amount bigint [not null]
  created_at timestamp [not null, default: `now()`]

  Indexes {
    account_id
  }
}

Table transfers {
  id bigserial [primary key]
  from_account_id bigint [ref: > A.id, not null]
  to_account_id bigint [ref: > A.id, not null]
  amount bigint [not null]
  created_at timestamp [not null, default: `now()`]

  Indexes {
    from_account_id
    to_account_id
    (from_account_id, to_account_id)
  }
}

```

### Postgres export:
 [Simple bank.sql](./sources/simplebank.sql)

### Design image export:
![design image export](./sources/Simple%20bank.png "design image export")

## 2. Cài đặt và sử dụng Docker + Postgres + PgAmin4

https://hub.docker.com/_/postgres
```
docker pull postgres:15-alpine
```

Khởi chạy container:
```
docker run --name some-postgres -e POSTGRES_PASSWORD=123456 -d -p 5432:5432 postgres:15-alpine
```

Exec psql:
```
docker exec -it some-postgres psql -U postgres
```

PgAdmin: https://www.pgadmin.org/download/

## 3. Database migration in golang
- Cài đặt [golang migrate](https://github.com/golang-migrate/migrate) và [cli](https://github.com/golang-migrate/migrate/tree/master/cmd/migrate)
- `mkdir -p simplebank/db/migration` và `cd simplebank`
- Tạo migration: `migrate create -ext sql -dir db/migration seq init_schema`
  > Kết quả:
  > ➜  simplebank git:(feat/section1) ✗ migrate create -ext sql -dir db/migration seq init_schema
  >{project dir}/simplebank/db/migration/20230822090810_seq.up.sql
  >{project dir}/simplebank/db/migration/20230822090810_seq.down.sql

  => [up](../simplebank/db/migration/20230822090810_seq.up.sql) and [down](../simplebank/db/migration/20230822090810_seq.down.sql)
- Tạo [makefile](../simplebank/Makefile)
- Xoá docker container và chạy các command makefile:
  ```
  make postgres
  make createdb
  ```
- Run migration:
  ```
  migrate -path db/migration -database "postgres://postgres:123456@localhost:5432/simple_bank?sslmode=disable" -verbose up
  ```
  - `postgres:123456`: username and password
  - `localhost:5432`: port
  - `simple_bank`: database name
  - `?sslmode=disable`: Tránh bị lỗi ` SSL is not enabled on the server`

## 4. Generate CRUD Golang code from SQL | Compare db/sql, gorm, sqlx & sqlc

  Có 4 cách để thực hiện CRUD trong golang:
  - DATABASE/SQL
    - Ưu điểm: chạy nhanh, tường minh
    - Nhược điểm: phải viết sql thuần, dễ mắc lỗi, lỗi chỉ được catch trong thời gian chạy
  - GORM
    - Ưu điểm: viết khá nhanh gọn
    - Nhược điểm: Khó kiểm soát những gì mình làm, performance thấp khi lưu lượng truy cập cao
  - SQLX
    - Ưu điểm: chạy nhanh, dễ sử dụng
    - Nhược điểm: Cú pháp vẫn chưa được ngắn gọn, lỗi chỉ được catch trong thời gian chạy (giống với DATABASE/SQL)
  - SQLC
    - Ưu điểm: Chạy nhanh, dễ sử dụng, tự động generate code, catch lỗi trước khi generate code
    - Nhược điểm: chỉ hỗ trợ đầy đủ postgres, mysql vẫn đang thử nghiệm

  => Trong dự án này sẽ sử dụng SQLC

  - https://docs.sqlc.dev/en/latest/index.html
  - Run `sqlc init` trong dir simplebank -> [file yaml](../simplebank/sqlc.yaml)
  - Setting và cấu trúc thư mục trong file trên
  - Đưa command `sqlc generate` vào [make file](../simplebank/Makefile)
  - Tạo các query CRUD của các table ở thư mục [db/query](../simplebank/db/query/)

## 5. Unit test cho CRUD

  [db/sqlc/*_test.go](../simplebank/db/sqlc/)

## 6,7,8. Implement database transaction in golang
[db/sqlc/store.go](../simplebank/db/sqlc/store.go)
[db/sqlc/store_test.go](../simplebank/db/sqlc/store_test.go)

## 9. Transaction ở mức độ cô lập (Isolation Level)
```
# TODO:
https://youtu.be/4EajrPgJAk0?si=LaKfNslaMmyaWnTz
```
