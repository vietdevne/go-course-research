postgres:
	docker run --name some-postgres -e POSTGRES_PASSWORD=123456 -d -p 5432:5432 postgres:15-alpine

createdb:
	docker exec -it some-postgres createdb --username=postgres --owner=postgres simple_bank

dropdb:
	docker exec -it some-postgres dropdb simple_bank -U postgres

migrateup:
	migrate -path db/migration -database "postgres://postgres:123456@localhost:5432/simple_bank?sslmode=disable" -verbose up

migratedown:
	migrate -path db/migration -database "postgres://postgres:123456@localhost:5432/simple_bank?sslmode=disable" -verbose down

sqlc:
	sqlc generate

test:
	go test -v -cover ./...

.PHONY: postgres createdb dropdb migrateup migratedown sqlc test