# Write Your First Test

Time to look at the integration test that starts a real PostgreSQL container.

## How It Works

Testcontainers for Go provides a `postgres` module that handles:

1. Pulling the PostgreSQL image (if not already cached)
2. Starting a container with the configuration you specify
3. Running init scripts to set up your schema
4. Waiting until PostgreSQL is ready to accept connections
5. Returning a connection string you can pass to your application

The test starts the container, creates the repository with its connection string, exercises the code, and cleans up automatically.

## The Test File

Open `customer/repo_test.go` in the editor. Here's what it contains:

```go no-run-button
package customer

import (
	"context"
	"path/filepath"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/testcontainers/testcontainers-go"
	"github.com/testcontainers/testcontainers-go/modules/postgres"
)

func TestCustomerRepository(t *testing.T) {
	ctx := context.Background()

	pgContainer, err := postgres.Run(ctx,
		"postgres:16-alpine",
		postgres.WithInitScripts(filepath.Join("..", "testdata", "init-db.sql")),
		postgres.WithDatabase("test-db"),
		postgres.WithUsername("postgres"),
		postgres.WithPassword("postgres"),
		postgres.BasicWaitStrategies(),
	)
	if err != nil {
		t.Fatal(err)
	}

	t.Cleanup(func() {
		if err := testcontainers.TerminateContainer(pgContainer); err != nil {
			t.Fatalf("failed to terminate container: %s", err)
		}
	})

	connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
	assert.NoError(t, err)

	customerRepo, err := NewRepository(ctx, connStr)
	assert.NoError(t, err)

	c, err := customerRepo.CreateCustomer(ctx, Customer{
		Name:  "Henry",
		Email: "henry@gmail.com",
	})
	assert.NoError(t, err)
	assert.NotNil(t, c)

	customer, err := customerRepo.GetCustomerByEmail(ctx, "henry@gmail.com")
	assert.NoError(t, err)
	assert.NotNil(t, customer)
	assert.Equal(t, "Henry", customer.Name)
	assert.Equal(t, "henry@gmail.com", customer.Email)
}
```

## What's Happening Here

Let's break down the key parts:

**Starting the container**

```go no-run-button no-copy-button
pgContainer, err := postgres.Run(ctx,
    "postgres:16-alpine",
    postgres.WithInitScripts(filepath.Join("..", "testdata", "init-db.sql")),
    postgres.WithDatabase("test-db"),
    postgres.WithUsername("postgres"),
    postgres.WithPassword("postgres"),
    postgres.BasicWaitStrategies(),
)
```

`postgres.Run` starts a PostgreSQL container. The image tag is the second argument, giving you explicit control over which PostgreSQL version to use. `WithInitScripts` runs your SQL file against the database once it's ready. `BasicWaitStrategies()` is the built-in wait strategy: it waits for the readiness log line to appear twice (PostgreSQL restarts itself on first boot), then waits for the port to be reachable — making it reliable on macOS, Windows, and Linux.

**Getting the connection string**

```go no-run-button no-copy-button
connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
```

Testcontainers maps the container's port to a random available port on your host. `ConnectionString` returns the correct host/port — you never hard-code it.

**Cleanup**

```go no-run-button no-copy-button
t.Cleanup(func() {
    if err := testcontainers.TerminateContainer(pgContainer); err != nil {
        t.Fatalf("failed to terminate container: %s", err)
    }
})
```

`t.Cleanup` registers a function that runs after the test, whether it passes or fails. `testcontainers.TerminateContainer` stops and removes the container.

## Run This Test

```bash
cd ~/project && go test -v -run TestCustomerRepository ./customer/
```

You'll see Testcontainers pull (or reuse) the image, start the container, run your test, and terminate the container. The output looks something like:

```text no-run-button no-copy-button
=== RUN   TestCustomerRepository
2026/01/01 12:00:00 github.com/testcontainers/testcontainers-go - Starting container...
2026/01/01 12:00:02 github.com/testcontainers/testcontainers-go - Container started: postgres:16-alpine
--- PASS: TestCustomerRepository (3.21s)
PASS
ok  	github.com/testcontainers/testcontainers-go-demo/customer	3.221s
```

> [!TIP]
> The first run is slower because Docker needs to pull the image. Subsequent runs reuse the cached image and are much faster.

> [!NOTE]
> The `GetCustomerByEmail` assertion retrieves `henry@gmail.com`, which was inserted by `CreateCustomer` in the same test. Each test run gets a fresh database — the init script only seeds `john@gmail.com`. In the next section, you'll see how to share a container across multiple tests.

## Try It Yourself

Try changing the PostgreSQL version from `postgres:16-alpine` to `postgres:17-alpine` in the editor and re-run the test. Testcontainers handles the rest — no configuration changes needed.

```bash
cd ~/project && go test -v -run TestCustomerRepository ./customer/
```

Move on to learn how to share a single container across multiple tests.
