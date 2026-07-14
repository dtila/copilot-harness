---
applyTo: "src/[projectname].Application.Tests/**/*IntegrationTests.cs"
---

Load skills based on your task:

- **`backend-integration-tests`** — for integration test files (`*IntegrationTests.cs`): AppServerFixture, DefaultAppServerFixture, TransactionScopedFixture, PostgreSQL container patterns, outbox verification, happy path + key negative paths.

## Running tests

Always run only the `IsolatedIntegrationTest` category:

```
dotnet test src/[projectname].Application.Tests/[projectname].Application.Tests.csproj -c Debug --filter "Category=IsolatedIntegrationTest"
```

Never run the full test suite without a filter — it includes live-network and NOT_TESTED tests that will fail in isolation.
