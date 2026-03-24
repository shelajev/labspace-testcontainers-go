# Run All Tests & Next Steps

You've written both a standalone integration test and a reusable suite. Let's run everything together and review what you've built.

## Run All Tests

```bash
go test -v ./...
```

You'll see output like this:

```text no-run-button no-copy-button
=== RUN   TestCustomerRepository
2024/01/01 12:00:00 Starting container for image: postgres:16-alpine
2024/01/01 12:00:02 Container started
--- PASS: TestCustomerRepository (3.21s)
=== RUN   TestCustomerRepoTestSuite
=== RUN   TestCustomerRepoTestSuite/TestCreateCustomer
=== RUN   TestCustomerRepoTestSuite/TestGetCustomerByEmail
--- PASS: TestCustomerRepoTestSuite (2.98s)
    --- PASS: TestCustomerRepoTestSuite/TestCreateCustomer (0.01s)
    --- PASS: TestCustomerRepoTestSuite/TestGetCustomerByEmail (0.01s)
PASS
ok  	github.com/testcontainers/testcontainers-go-demo/customer	6.201s
```

Two containers started in total — one for `TestCustomerRepository` and one for the suite. All tests pass.

> [!TIP]
> You can run tests with `-count=1` to bypass Go's test result cache and force containers to start fresh every time:
> ```bash
> go test -v -count=1 ./...
> ```

## What You Built

Let's look at the files you created:

```bash
find ~/project -name "*.go" | sort
```

```text no-run-button no-copy-button
~/project/customer/repo.go
~/project/customer/repo_suite_test.go
~/project/customer/repo_test.go
~/project/customer/types.go
~/project/testhelpers/containers.go
```

| File | Role |
|------|------|
| `customer/repo_test.go` | Standalone test — one container, one test, full lifecycle |
| `testhelpers/containers.go` | Reusable helper — encapsulates container setup |
| `customer/repo_suite_test.go` | Suite — one container, multiple tests, shared lifecycle |

## What You Learned

- **Testcontainers for Go** starts real Docker containers from your test code using a clean, declarative API
- **Wait strategies** ensure containers are ready before tests run, eliminating flaky timing-based sleeps
- **`t.Cleanup`** ensures containers are always terminated, even when tests fail
- **Test suites** (`testify/suite`) let you amortize container startup cost across many tests
- **Extracting container setup** into a `testhelpers` package keeps tests readable and promotes reuse

## Next Steps

### Try different PostgreSQL versions

Change `postgres:16-alpine` to `postgres:16-alpine` or `postgres:17-alpine` in your test and rerun. Testcontainers handles the rest — no configuration changes needed.

### Explore other Testcontainers modules

Testcontainers for Go has ready-made modules for many common services. Add one to your project:

```bash no-run-button
go get github.com/testcontainers/testcontainers-go/modules/redis
go get github.com/testcontainers/testcontainers-go/modules/kafka
go get github.com/testcontainers/testcontainers-go/modules/localstack
```

### Use Testcontainers Cloud

For CI environments or teams with limited local Docker resources, [Testcontainers Cloud](https://testcontainers.com/cloud/) runs your containers remotely. Set the `TC_CLOUD_TOKEN` environment variable and your tests work without any code changes.

### Read the docs

- [Testcontainers for Go documentation](https://golang.testcontainers.org)
- [Available Go modules](https://golang.testcontainers.org/modules/)
- [Generic container usage](https://golang.testcontainers.org/features/containers/)

---

You've completed the **Getting Started with Testcontainers for Go** lab!

You now know how to write integration tests that use real databases — the foundation for reliable, production-faithful testing with Go.
