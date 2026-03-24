# Labspace - Getting Started with Testcontainers for Go

Testcontainers lets you use real Docker containers in your tests instead of mocks.
In this Labspace, you'll learn how to test a PostgreSQL-backed Go application using
Testcontainers for Go.

## Learning objectives

- Write integration tests against a real PostgreSQL container started on demand
- Use the `postgres` module's built-in wait strategies for reliable startup
- Reuse a single container across multiple tests with testify test suites

## Launch the Labspace

```bash
docker compose -f oci://olegselajev241/labspace-testcontainers-go up -d
```

Open your browser to http://localhost:3030.

## Contributing

```bash
# On Mac/Linux
CONTENT_PATH=$PWD docker compose up --watch

# On Windows with PowerShell
$Env:CONTENT_PATH = (Get-Location).Path; docker compose up --watch
```

Open the Labspace at http://localhost:3030.
