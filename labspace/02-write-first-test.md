# Write Your First Test

Time to write an integration test that starts a real PostgreSQL container.

## How It Works

Testcontainers for Go provides a `postgres` module that handles:

1. Pulling the PostgreSQL image (if not already cached)
2. Starting a container with the configuration you specify
3. Running init scripts to set up your schema
4. Waiting until PostgreSQL is ready to accept connections
5. Returning a connection string you can pass to your application

Your test starts the container, creates the repository with its connection string, exercises the code, and cleans up via `t.Cleanup`.

## Create the Test File

Create `customer/repo_test.go` with the following content:

```go save-as=customer/repo_test.go
package customer

import (
	"context"
	"path/filepath"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"
	"github.com/testcontainers/testcontainers-go"
	"github.com/testcontainers/testcontainers-go/modules/postgres"
	"github.com/testcontainers/testcontainers-go/wait"
)

func TestCustomerRepository(t *testing.T) {
	ctx := context.Background()

	pgContainer, err := postgres.RunContainer(ctx,
		testcontainers.WithImage("postgres:15.3-alpine"),
		postgres.WithInitScripts(filepath.Join("..", "testdata", "init-db.sql")),
		postgres.WithDatabase("test-db"),
		postgres.WithUsername("postgres"),
		postgres.WithPassword("postgres"),
		testcontainers.WithWaitStrategy(
			wait.ForLog("database system is ready to accept connections").
				WithOccurrence(2).
				WithStartupTimeout(5*time.Second)),
	)
	if err != nil {
		t.Fatal(err)
	}

	t.Cleanup(func() {
		if err := pgContainer.Terminate(ctx); err != nil {
			t.Fatalf("failed to terminate pgContainer: %s", err)
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
pgContainer, err := postgres.RunContainer(ctx,
    testcontainers.WithImage("postgres:15.3-alpine"),
    postgres.WithInitScripts(filepath.Join("..", "testdata", "init-db.sql")),
    postgres.WithDatabase("test-db"),
    postgres.WithUsername("postgres"),
    postgres.WithPassword("postgres"),
    testcontainers.WithWaitStrategy(
        wait.ForLog("database system is ready to accept connections").
            WithOccurrence(2).
            WithStartupTimeout(5*time.Second)),
)
```

`postgres.RunContainer` starts a PostgreSQL container. The `WithInitScripts` option runs your SQL file against the database once it's ready. The `WithWaitStrategy` ensures your test doesn't proceed until PostgreSQL is genuinely accepting connections (the log message appears twice — once for each startup stage).

**Getting the connection string**

```go no-run-button no-copy-button
connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
```

Testcontainers maps the container's port to a random available port on your host. `ConnectionString` returns the correct host/port — you never hard-code it.

**Cleanup**

```go no-run-button no-copy-button
t.Cleanup(func() {
    if err := pgContainer.Terminate(ctx); err != nil {
        t.Fatalf("failed to terminate pgContainer: %s", err)
    }
})
```

`t.Cleanup` registers a function that runs after the test, whether it passes or fails. This ensures the container is always stopped and removed.

## Run This Test

Let's run just this one test to see it work:

```bash
cd ~/project && go test -v -run TestCustomerRepository ./customer/
```

You'll see Testcontainers pull (or reuse) the image, start the container, run your test, and terminate the container. The output looks something like:

```text no-run-button no-copy-button
=== RUN   TestCustomerRepository
2024/01/01 12:00:00 github.com/testcontainers/testcontainers-go - Starting container...
2024/01/01 12:00:02 github.com/testcontainers/testcontainers-go - Container started: postgres:15.3-alpine
--- PASS: TestCustomerRepository (3.21s)
PASS
ok  	github.com/testcontainers/testcontainers-go-demo/customer	3.221s
```

> [!TIP]
> The first run is slower because Docker needs to pull the image. Subsequent runs reuse the cached image and are much faster.

> [!NOTE]
> The `GetCustomerByEmail` test retrieves `henry@gmail.com`, which was inserted by `CreateCustomer` in the same test. Each test run gets a fresh database — the init script only seeds `john@gmail.com`. In the next section, you'll see how to handle this cleanly when multiple tests share the same container.

Move on to learn how to share a single container across multiple tests.
