# Plan: [Feature name]

> Copy this file next to `spec.md`. The plan answers *how*; the spec answers *what*.
> Optional (1–10 people): top with a `Status:` line (Draft → Active → Shipped → Superseded); add `Owner:` only past ~5 people — below that, `git blame` covers it.

## Technical decisions

- Stack: [e.g. .NET 8, Dapper, MS SQL Server]
- Pattern: [e.g. repository + handler, no MediatR]
- Logging: [e.g. Serilog with `IBaseHandler<TSelf>` contract]

## File structure

```
src/
  Domain/
    ...
  Infrastructure/
    Repositories/XRepository.cs
  Application/
    Handlers/XHandler.cs
tests/
  Integration/
    XHandlerTests.cs
```

## Tasks (in execution order)

1. [First task — usually a failing test or a DDL change]
2. ...
3. ...

## Constraints

- Do NOT [explicit "don't do this" — covers known agent failure modes]
- Maintain consistency with [existing pattern to follow]

## References

- [Spec](./spec.md)
- ADR-NNN: [linked decision]
- `src/.../ExistingPattern.cs` — pattern to follow
