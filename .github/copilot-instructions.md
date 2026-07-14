# GitHub Copilot Instructions (Minimal Router)


## Running tests

Always filter to the `IsolatedIntegrationTest` category when running integration tests:

```
dotnet test src/[projectname].Application.Tests/[projectname].Application.Tests.csproj -c Debug --filter "Category=IsolatedIntegrationTest"
```
