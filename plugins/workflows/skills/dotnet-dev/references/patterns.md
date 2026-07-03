# .NET / C# Patterns

## Project and file structure

- Follow the solution's existing layout — typically `src/<Project>/` and
  `tests/<Project>.Tests/`. Never invent a parallel structure.
- One public type per file, filename matches the type: `OrderService.cs`
  contains `OrderService`.
- Namespaces mirror folder paths (file-scoped namespaces in modern code:
  `namespace MyApp.Orders;`).
- Naming: `PascalCase` for types/methods/properties, `camelCase` for locals
  and parameters, `_camelCase` for private fields, `IThing` for interfaces,
  `ThingAsync` suffix for async methods.

## Dependency injection

- Constructor injection only — dependencies are visible in the signature and
  the type is honest about what it needs:

```csharp
public class OrderService(IOrderRepository repository, ILogger<OrderService> logger)
    : IOrderService
{
    // primary constructor (C# 12); use a classic ctor in older codebases
}
```

- No service locator (`IServiceProvider.GetService` inside business logic) and
  no `new`-ing dependencies that have interfaces — both hide the dependency
  graph and kill testability.
- Register with the right lifetime: `Scoped` for per-request state
  (DbContext), `Singleton` for stateless services, `Transient` when in doubt
  and the object is cheap. Never inject a `Scoped` into a `Singleton`.

## Async

- Async all the way down. `.Result`, `.Wait()`, and `.GetAwaiter().GetResult()`
  on hot paths are deadlocks waiting for a synchronization context.
- Accept and forward `CancellationToken` on every async public API —
  ASP.NET Core hands you one per request; EF and HttpClient accept it.
- `Task` return for I/O-bound work; don't wrap synchronous work in
  `Task.Run` inside libraries.
- Name async methods with the `Async` suffix; return `Task<T>`, not `void`
  (async void is only for event handlers — exceptions there are uncatchable).

## Error handling

- Throw specific exception types (`ArgumentException`,
  `InvalidOperationException`, domain exceptions) — never `throw new Exception`.
- Catch only what you can handle; let the rest bubble to middleware.
  ASP.NET Core apps centralize the exception → HTTP mapping in one exception
  handler, not try/catch per controller action.
- Preserve stack traces: `throw;`, never `throw ex;`.
- Validate at the boundary (request models, guard clauses at public API
  entry) so internals can assume valid state.

## LINQ and collections

- LINQ for transformation, loops for side effects — a `foreach` that mutates
  is clearer than a `.Select` with side effects.
- Materialize deliberately: know whether you're returning a lazy `IEnumerable`
  or a snapshot (`.ToList()`). Multiple enumeration of a lazy query is a
  hidden N+1.
- Expose the narrowest useful type: `IReadOnlyList<T>` beats `List<T>` on
  public APIs.

## Entity Framework

- Queries stay in the repository/data layer; controllers and services don't
  compose `IQueryable`.
- `AsNoTracking()` for read-only queries — change tracking is significant
  overhead you're paying for updates that never happen.
- Project early (`.Select` into a DTO) instead of loading full entities and
  mapping in memory. `Include` + `Select` together is a smell — projection
  makes the Include redundant.
- Explicit `Include` for navigation properties you need — lazy loading in a
  request loop is the classic N+1. `AsSplitQuery()` when including multiple
  collections (avoids Cartesian explosion).
- Filter and page *before* materializing — `Where` after `ToList()` pulled
  the whole table into memory. `.Any()` for existence, never `.Count() > 0`.
- Bulk changes use `ExecuteUpdateAsync`/`ExecuteDeleteAsync`, not
  fetch-modify-save loops.
- Diagnosing slowness? Turn on SQL logging
  (`Microsoft.EntityFrameworkCore.Database.Command` at Information) and count
  the actual queries before guessing.

## API shape (ASP.NET Core)

- **Detect the existing style first** — controllers (`ControllerBase` /
  `[ApiController]`) vs minimal APIs (`app.MapGet` in `Program.cs`) — and
  never mix the two in one project. Fresh project with no endpoints →
  default to minimal APIs.
- Controllers/endpoints stay thin: bind → validate → call service → map to
  response. Business logic lives in services; never inject data stores
  directly into endpoints.
- DTOs are `sealed record` types (immutable, value equality), named
  `Create{Entity}Request` / `{Entity}Response`, separate from EF entities —
  entities never go on the wire.
- `DateTimeOffset` over `DateTime` on API contracts — preserves the UTC
  offset and serializes to ISO 8601 correctly.
- Typed results with explicit unions —
  `Task<Results<Ok<ProductResponse>, NotFound>>` + `TypedResults.Ok(...)` /
  `TypedResults.NotFound()` — the compiler then infers OpenAPI responses.
- Status codes carry meaning: POST → 201 + `Location` header
  (`CreatedAtAction` / `TypedResults.Created`), DELETE → 204, never
  200-with-error-body.
- Centralize error mapping: `AddProblemDetails()` + `UseExceptionHandler()`
  (RFC 7807), custom exception→status mapping via one `IExceptionHandler` —
  not try/catch per action.
- Enums serialize as strings (`JsonStringEnumConverter`) unless the API
  contract explicitly requires integers.

## Before you hand off

- `dotnet build` — zero errors, no new warnings.
- Every async public API: `Async` suffix, `CancellationToken` accepted and
  forwarded.
- No sync-over-async (`.Result`/`.Wait()`), no service-locator lookups.
- No EF entities on the wire; read-only queries `AsNoTracking()`; no N+1 in
  loops.
