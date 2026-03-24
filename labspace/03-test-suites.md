# Reuse Containers with Test Suites

In the previous section, `TestCustomerRepository` starts a container and tears it down for a single test. That works well for isolated tests, but when you have **many tests**, spinning up a container per test adds significant overhead.

The solution: start one container, run all tests against it, tear it down at the end. Testify's **test suites** make this clean and structured.

## The Container Helper

Open `testhelpers/containers.go` in the editor. This is a shared helper that any test file can use:

```go no-run-button
package testhelpers

import (
	"context"
	"path/filepath"

	"github.com/testcontainers/testcontainers-go/modules/postgres"
)

type PostgresContainer struct {
	*postgres.PostgresContainer
	ConnectionString string
}

func CreatePostgresContainer(ctx context.Context) (*PostgresContainer, error) {
	pgContainer, err := postgres.Run(ctx,
		"postgres:16-alpine",
		postgres.WithInitScripts(filepath.Join("..", "testdata", "init-db.sql")),
		postgres.WithDatabase("test-db"),
		postgres.WithUsername("postgres"),
		postgres.WithPassword("postgres"),
		postgres.BasicWaitStrategies(),
	)
	if err != nil {
		return nil, err
	}

	connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
	if err != nil {
		return nil, err
	}

	return &PostgresContainer{
		PostgresContainer: pgContainer,
		ConnectionString:  connStr,
	}, nil
}
```

`PostgresContainer` wraps the postgres module's container and eagerly resolves the connection string, so callers don't need to call `ConnectionString` themselves.

## The Test Suite

Now open `customer/repo_suite_test.go`. This uses testify's `Suite` to start the container once for all tests:

```go no-run-button
package customer

import (
	"context"
	"log"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/suite"
	"github.com/testcontainers/testcontainers-go"
	"github.com/testcontainers/testcontainers-go-demo/testhelpers"
)

type CustomerRepoTestSuite struct {
	suite.Suite
	pgContainer *testhelpers.PostgresContainer
	repository  *Repository
	ctx         context.Context
}

func (suite *CustomerRepoTestSuite) SetupSuite() {
	suite.ctx = context.Background()

	pgContainer, err := testhelpers.CreatePostgresContainer(suite.ctx)
	if err != nil {
		log.Fatal(err)
	}
	suite.pgContainer = pgContainer

	repository, err := NewRepository(suite.ctx, suite.pgContainer.ConnectionString)
	if err != nil {
		log.Fatal(err)
	}
	suite.repository = repository
}

func (suite *CustomerRepoTestSuite) TearDownSuite() {
	if err := testcontainers.TerminateContainer(suite.pgContainer.PostgresContainer); err != nil {
		log.Fatalf("error terminating postgres container: %s", err)
	}
}

func (suite *CustomerRepoTestSuite) TestCreateCustomer() {
	t := suite.T()

	customer, err := suite.repository.CreateCustomer(suite.ctx, Customer{
		Name:  "Henry",
		Email: "henry@gmail.com",
	})
	assert.NoError(t, err)
	assert.NotNil(t, customer.Id)
}

func (suite *CustomerRepoTestSuite) TestGetCustomerByEmail() {
	t := suite.T()

	customer, err := suite.repository.GetCustomerByEmail(suite.ctx, "john@gmail.com")
	assert.NoError(t, err)
	assert.NotNil(t, customer)
	assert.Equal(t, "John", customer.Name)
	assert.Equal(t, "john@gmail.com", customer.Email)
}

func TestCustomerRepoTestSuite(t *testing.T) {
	suite.Run(t, new(CustomerRepoTestSuite))
}
```

## How the Suite Works

**`SetupSuite`** runs once before all tests in the suite. It starts the container and creates a single `Repository` instance shared by all tests.

**`TearDownSuite`** runs once after all tests complete (pass or fail). `testcontainers.TerminateContainer` stops and removes the container.

**Individual tests** (`TestCreateCustomer`, `TestGetCustomerByEmail`) run against the shared container. Notice that `TestGetCustomerByEmail` queries for `john@gmail.com` — the row seeded by `init-db.sql`. It doesn't depend on `TestCreateCustomer` having run first.

> [!WARNING]
> Tests in a suite share state via the database. If `TestCreateCustomer` inserts `henry@gmail.com` and another test tries to insert the same email, you'll get a constraint violation. Design your test data carefully, or use `SetupTest` to reset data before each individual test.

## Run the Suite

```bash
go test -v -run TestCustomerRepoTestSuite ./customer/
```

You should see both suite tests pass with only one container start:

```text no-run-button no-copy-button
=== RUN   TestCustomerRepoTestSuite
=== RUN   TestCustomerRepoTestSuite/TestCreateCustomer
=== RUN   TestCustomerRepoTestSuite/TestGetCustomerByEmail
--- PASS: TestCustomerRepoTestSuite (2.98s)
    --- PASS: TestCustomerRepoTestSuite/TestCreateCustomer (0.01s)
    --- PASS: TestCustomerRepoTestSuite/TestGetCustomerByEmail (0.01s)
PASS
ok  	github.com/testcontainers/testcontainers-go-demo/customer	2.983s
```

Notice the total time (~3s) is similar to the single test in the previous section — but now covers two tests. As you add more tests to the suite, the container startup cost remains constant.

Move on to run everything together and explore what comes next.
