# app-ddd

A small Go application core organized around domain rules, commands, queries and explicit repository contracts.
There is no executable, transport or concrete persistence implementation. Add those at the application boundary
without coupling business rules to infrastructure.

## Architecture

```text
internal/
├── domain/user/             aggregate, validation and typed errors
├── adapter/repository.go    repository interfaces only
├── app/
│   ├── app.go               commands and queries exposed by the application
│   ├── command/             state changes returning error
│   ├── query/               read operations returning data
│   └── mock/                generated test doubles
├── service/app.go           dependency assembly
├── port/                    Job and Listener contracts, testable example
└── common/                  logger configuration, handler observation and trace helpers
```

Domain code uses only the standard library. Repository interfaces live in `internal/adapter/repository.go`;
application code may import that contracts-only package, never concrete adapter subpackages. Only the entrypoint
assembles infrastructure. `task lint` enforces these boundaries with depguard.

The conceptual architecture reference is [Go With The Domain](https://threedots.tech/go-with-the-domain/).
Apply its domain-focused, pragmatic approach without adding patterns that the application does not need.

Configure process logging through `internal/common/logger.Configure(level, pretty, output)` from the entrypoint.
The helper selects JSON or readable text and applies the same minimum level to zerolog and slog.

## Run the example

Go 1.25.8 or newer is required. The example wires an in-memory repository into a job:

```sh
go test -run Example_helloJob -v ./internal/port
```

## Repository contract

Implement `adapter.UserRepository` in `internal/adapter/<technology>` and pass it to `service.NewApp`.
Add a depguard rule for that implementation's dependencies.

- Every method receives `context.Context` first and preserves errors.
- Missing records return `user.NotFoundError`; duplicate IDs/emails return `user.AlreadyExistsError`.
- `ListUsers` sorts by name and then ID without mutating the repository's slice.
- Commands accept caller-provided IDs and return only `error`; queries do not modify state.

Use `var _ adapter.UserRepository = (*Repository)(nil)` to check an implementation at compile time.
For a new entrypoint, add `cmd/<name>` and a transport under `internal/port` without changing the domain.

## Development

```sh
make test
make lint
make coverage
```

Other tasks: `tidy`, `format`, `mock`. Make delegates to Task; tools are pinned in `tools/go.tool.mod` and run
through `go tool -modfile=./tools/go.tool.mod`. Do not edit generated mocks manually. Coverage target: over 80%
overall and 100% on command/query handlers. See [ADR 0001](docs/adr/0001-self-contained-frame.md) for decisions.
