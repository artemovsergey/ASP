# Report of Coverage Test

- dotnet tool install -g dotnet-reportgenerator-globaltool
- установить `coverlet.msbuild`
- dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura
- reportgenerator -reports:"coverage.cobertura.xml" -targetdir:"coveragereport" -reporttypes:Html
