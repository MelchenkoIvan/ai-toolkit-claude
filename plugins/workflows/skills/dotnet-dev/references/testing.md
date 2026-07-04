# .NET Testing Conventions — xUnit + Moq

## Detect the framework first

Read the test project's `.csproj` before writing tests:

- `xunit.v3` package → **xUnit v3** (runs on Microsoft Testing Platform; not
  fully source-compatible with v2). `xunit` → v2. Match what's installed —
  don't mix the packages or upgrade in passing.
- Repo already on NUnit or MSTest? Follow that framework's idioms instead of
  introducing xUnit alongside it — one test framework per solution.
- Same rule for mocking: if the repo uses NSubstitute or FakeItEasy, match
  it; Moq guidance below transfers concept-for-concept.

## Files and naming

- Test project mirrors the source project: `src/MyApp.Orders` →
  `tests/MyApp.Orders.Tests`, with `<TypeName>Tests.cs` per class under test
  (`OrderServiceTests.cs`).
- Test method names state scenario and expectation:
  `MethodName_Condition_ExpectedResult` —
  `CreateOrder_WhenItemOutOfStock_ThrowsOutOfStockException`. Never `Test1`.
- Run with `dotnet test`; keep the suite runnable from a clean checkout.

## Arrange-Act-Assert

Every test has the three phases, visually separated:

```csharp
[Fact]
public async Task CreateOrder_WhenItemOutOfStock_ThrowsOutOfStockException()
{
    // Arrange
    var repository = new Mock<IOrderRepository>();
    repository.Setup(r => r.GetStockAsync(ItemId, It.IsAny<CancellationToken>()))
              .ReturnsAsync(0);
    var sut = new OrderService(repository.Object, NullLogger<OrderService>.Instance);

    // Act
    var act = () => sut.CreateOrderAsync(new OrderRequest(ItemId, Quantity: 1), CancellationToken.None);

    // Assert
    await Assert.ThrowsAsync<OutOfStockException>(act);
}
```

- One logical assertion per test — multiple `Assert` calls are fine when they
  verify one outcome; verifying two behaviors is two tests.
- Name the subject `sut` (system under test) so it's instantly identifiable.

## xUnit specifics

- `[Fact]` for a single case, `[Theory]` + `[InlineData]` for the same
  behavior across inputs — don't copy-paste facts that differ by one value:

```csharp
[Theory]
[InlineData("", false)]
[InlineData("not-an-email", false)]
[InlineData("a@b.com", true)]
public void IsValidEmail_ReturnsExpected(string input, bool expected) =>
    Assert.Equal(expected, EmailValidator.IsValid(input));
```

- Constructor = per-test setup, `IDisposable.Dispose` = teardown; shared
  expensive state goes in `IClassFixture<T>`. No `[SetUp]`-style attributes —
  that's NUnit.
- Async tests return `Task`, never `async void`.

## Moq specifics

- Mock interfaces, not concrete classes — if you need to mock a class, that's
  a design signal to extract an interface.
- `Setup` only what the test needs; over-specified mocks break on refactor.
- Verify *interactions that are the point of the test*
  (`repository.Verify(r => r.SaveAsync(It.IsAny<Order>(), default), Times.Once)`)
  — don't `Verify` every call, that's testing implementation.
- `MockBehavior.Loose` (default) unless the test is specifically about the
  full interaction protocol.
- Don't mock what you don't own (DbContext, HttpClient) — wrap them: test
  repositories against EF Core InMemory/SQLite, test HTTP with a fake
  `HttpMessageHandler`.

## What to cover

Priority order for a class's suite:

1. The primary behavior path (valid input → expected result/side effect).
2. Domain error paths (out of stock, not found, unauthorized → specific
   exceptions or result types).
3. Boundary/argument validation (null, empty, out-of-range at public API).
4. Cancellation for long-running async operations, where implemented.

Test through public APIs; private methods are covered via the public ones
that use them. A suite coupled to internals breaks on every refactor and
verifies nothing about behavior.

## Anti-patterns that fail review

**False confidence (critical):**

- A test with no assertion — calls the code, verifies nothing. Coverage
  rises, correctness is unchecked.
- Missing `await` on an async assertion (`Assert.ThrowsAsync` un-awaited
  passes unconditionally).
- Self-referential assertions (`Assert.Equal(x, x)`) and tautologies that
  cannot fail.
- Empty catch blocks in tests, or asserting only inside a `catch` that may
  never run — use `Assert.Throws<T>` instead.

**Flakiness:**

- `Thread.Sleep` / wall-clock waits — inject `TimeProvider` (or an
  abstraction) and control time; poll with a timeout only in true
  integration tests.
- Direct `DateTime.Now` / unseeded `Random` in the path under test.
- Order-dependent tests via shared mutable state — xUnit runs classes in
  parallel; every test arranges its own world.

**Assertion quality:**

- `Assert.NotNull(result)` alone is trivial — follow the null guard with
  assertions on the values that matter.
- Don't assert one field of a rich result and call it covered; assert the
  fields the behavior actually changes, and what should *not* have changed.
- Exception tests are exempt: a single `Assert.ThrowsAsync<T>` *is* a
  complete assertion.
