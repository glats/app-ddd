# ADR 0001: Self-contained DDD application core

Status: implemented

## Context

The project needs a small, testable domain-focused core that can accept different transports and storage
implementations without making business rules depend on infrastructure.

## Decisions

1. Keep a single Go module with domain types, commands, queries and repository contracts.
2. Put repository interfaces in the contracts-only adapter root. Concrete adapters are separate subpackages.
3. Assemble handlers in `service.NewApp`; add executable entrypoints only when a transport is needed.
4. Keep commands and queries separate. Commands return errors; queries return data without side effects.
5. Provide local helper implementations for handler observation and trace contexts. Keep their public contracts
   stable and verify results, errors, context forwarding, logs and metrics through characterization tests.
6. Enforce package dependencies with depguard; generate mocks and demonstrate assembly in a Go example test.

## Consequences

The core is not a standalone server. Storage and transport can be added without changing domain rules, and
repository-contract changes do not require releasing a separate entity-domain module. Helpers remain deliberately
small; metric backends and distributed tracing systems are not required.

## Reference

[Go With The Domain](https://threedots.tech/go-with-the-domain/) is the conceptual architecture reference.
