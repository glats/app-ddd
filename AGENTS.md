# Project conventions

`CLAUDE.md` is a symlink to this file. Keep technical artifacts in English.

## Architecture

Use [Go With The Domain](https://threedots.tech/go-with-the-domain/) as the conceptual architecture reference.
Keep the domain independent of infrastructure. This project contains no executable or concrete adapter.

- Domain: standard library only.
- Adapter root: repository contracts and domain types only.
- Application: adapter-root contracts, domain and common helpers; no concrete adapter subpackages.
- Transport ports: application and domain.
- Service: assembles application handlers, not infrastructure connections.
- Common helpers: standard library, local trace context and zerolog.

Depguard enforces these boundaries. Do not weaken them to make a check pass.

## Implementation

Repository interfaces belong in `internal/adapter/repository.go`, not domain or application ports.
Methods take `context.Context` first. Commands change state and return only errors; queries return data.
The caller supplies new IDs. Keep each use case with its tests. Use private aggregate fields, constructors,
validation and typed errors. Preserve existing outputs, metric keys and error identity during refactors.

Run `make test`, `make lint` and `make coverage`; `format`, `tidy` and `mock` are also available.
Logger configuration belongs in `internal/common/logger`; entrypoints call it with parsed values and an output writer.
The common logger must not import the environment/config adapter.
Tools use `go tool -modfile=./tools/go.tool.mod`. Regenerate mocks with `make mock`, never by hand.
Do not add module replacements or private registry requirements. Keep required third-party notices.
