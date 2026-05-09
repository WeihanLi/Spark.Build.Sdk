# Spark.Build.Sdk

A modern, opinionated, configurable .NET 10 MSBuild SDK for OSS projects.

## Features

- Nullable enabled
- Implicit usings enabled
- Deterministic builds
- SourceLink defaults
- NuGet audit enforcement
- Package validation
- Reproducible builds
- CI-friendly defaults
- Library/sample/test profiles

## Usage

Add to `global.json`:

```json
{
  "msbuild-sdks": {
    "Spark.Build.Sdk": "0.1.0"
  }
}
```

Use in project:

```xml
<Project Sdk="Spark.Build.Sdk">

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>

</Project>
```
