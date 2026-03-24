# Introduction

Welcome to **Getting Started with Testcontainers for Go**!

In this lab, you will:

- Write integration tests for a PostgreSQL-backed Go repository using **Testcontainers for Go**
- Start real PostgreSQL containers on demand — no mocks, no in-memory substitutes
- Reuse a single container across multiple tests using **testify test suites**
- See why real-dependency testing gives you more confidence than fakes

## What is Testcontainers?

Testcontainers is a library that spins up real Docker containers during your tests and tears them down when finished. Instead of a stripped-down in-memory database, you get an actual PostgreSQL instance running the exact same version as production.

This means your tests catch bugs that fakes often miss: SQL dialect differences, constraint violations, concurrent access patterns, and more. And because containers are ephemeral, every test run starts clean.

## The Application

The project already contains a working customer management service. You'll write tests for it.

| File | What it does |
|------|-------------|
| `customer/types.go` | Defines the `Customer` struct (`Id`, `Name`, `Email`) |
| `customer/repo.go` | `Repository` with `CreateCustomer` and `GetCustomerByEmail` using the `pgx` driver |
| `testdata/init-db.sql` | Creates the `customers` table and seeds one test row |
| `go.mod` | Module definition with all dependencies pre-declared |

> [!NOTE]
> Dependencies have been downloaded into your workspace at startup. You can jump straight into writing tests without any `go get` commands.

## Verify Your Environment

Let's confirm Go and Docker are both available:

```bash
go version && docker info --format 'Docker Engine {{.ServerVersion}}'
```

You should see a Go version (`go1.21` or later) and a Docker Engine version. If Docker isn't running, Testcontainers cannot start containers.

Take a quick look at the code you'll be testing:

```bash
cat ~/project/customer/types.go
```

```bash
cat ~/project/customer/repo.go
```

```bash
cat ~/project/testdata/init-db.sql
```

The repository connects to a PostgreSQL database using a connection string. Your tests will provide that connection string — pointing to a container Testcontainers starts on demand.

When you're ready, move to the next section to write your first test.
