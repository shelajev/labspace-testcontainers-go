# Testcontainers Go Demo

A customer management service backed by PostgreSQL, used in the
**Getting Started with Testcontainers for Go** lab.

## Structure

```
customer/
  types.go        - Customer struct
  repo.go         - Repository (CreateCustomer, GetCustomerByEmail)
testdata/
  init-db.sql     - Schema creation and seed data
testhelpers/      - Created during the lab
  containers.go
go.mod
```

## Running Tests

After completing the lab:

```bash
go test -v ./...
```
