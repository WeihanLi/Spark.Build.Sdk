# AGENTS.md

## Project Overview

Spark.Build.Sdk is an opinionated .NET 10 MSBuild SDK packaged as `Spark.Build.Sdk`.
The package content lives under `src/Spark.Build.Sdk/Sdk` and is packed as an
MSBuild SDK, not as a runtime library.

Key files:

- `src/Spark.Build.Sdk/Spark.Build.Sdk.csproj` defines NuGet package metadata and
  includes the SDK files in the package.
- `src/Spark.Build.Sdk/Sdk/Sdk.props` imports defaults and the selected profile.
- `src/Spark.Build.Sdk/Sdk/Sdk.targets` imports profile targets when present and
  emits low-importance SDK profile info.
- `src/Spark.Build.Sdk/Sdk/defaults/*.props` contains shared compiler,
  deterministic build, packaging, and security defaults.
- `src/Spark.Build.Sdk/Sdk/profiles/*.props` contains `library`, `sample`, and
  `test` profile behavior.
- `version.json` uses Nerdbank.GitVersioning for package versioning.

## Build and Test Commands

Use the same command shape as CI:

```powershell
dotnet restore
dotnet build --configuration Release --no-restore
dotnet pack src/Spark.Build.Sdk/Spark.Build.Sdk.csproj --configuration Release --no-build
```

The repository currently has no dedicated test project. When changing SDK props
or targets, at minimum run restore, release build, and pack. For behavior changes,
also validate the packed SDK from `artifacts/package/release` in a small sample
project when practical.

## Code Style Guidelines

- Keep MSBuild XML readable and consistent with existing files: two-space
  indentation, blank lines between logical property groups, and one setting per
  element.
- Prefer conditional defaults such as `Condition="'$(Property)' == ''"` so
  consuming projects can override SDK behavior.
- Keep profile-specific behavior in `Sdk/profiles` and shared defaults in
  `Sdk/defaults`.
- Avoid adding custom MSBuild targets unless props alone cannot express the
  behavior.
- Preserve package layout under `Sdk/**`; consumers depend on MSBuild SDK import
  conventions.
- Keep documentation and examples aligned with the current package id and
  version in `global.json`.

## Testing Instructions

For SDK changes, verify both repository build health and consuming-project
behavior:

1. Run `dotnet restore`.
2. Run `dotnet build --configuration Release --no-restore`.
3. Run `dotnet pack src/Spark.Build.Sdk/Spark.Build.Sdk.csproj --configuration Release --no-build`.
4. If a default or profile changed, create or reuse a temporary consuming project
   and confirm the expected MSBuild properties are applied for the relevant
   `SparkBuildProfile` value.

Useful consuming-project checks include `library`, `sample`, and `test` profiles:

```xml
<PropertyGroup>
  <SparkBuildProfile>test</SparkBuildProfile>
</PropertyGroup>
```

## Security Considerations

- Do not weaken the NuGet audit defaults in `Sdk/defaults/security.props`
  without an explicit reason.
- Security audit warnings `NU1900` through `NU1904` are configured as errors;
  keep that behavior intentional and documented if it changes.
- Avoid committing secrets, package feed credentials, API keys, or local NuGet
  source configuration.
- Treat dependency and tool version changes as security-sensitive. Review audit
  output and keep `PrivateAssets="All"` for build-only package references unless
  there is a specific reason to expose them transitively.

## Pull Request and Commit Guidance

- Keep changes narrowly scoped. Separate SDK behavior changes from unrelated
  documentation or formatting edits.
- In PR descriptions, call out which SDK defaults or profiles changed and how a
  consuming project is affected.
- Include the restore/build/pack commands you ran, plus any sample-project
  validation.
- Commit messages should be concise and action-oriented, for example
  `Add test profile defaults` or `Tighten NuGet audit settings`.

## Agent Notes

- Do not modify generated package artifacts unless the task specifically requires
  checking pack output.
- Be careful with existing uncommitted changes. Inspect them before editing the
  same files, and do not revert unrelated work.
- This repo targets .NET 10 tooling. If the local SDK is missing, report that
  clearly instead of downgrading target frameworks or package settings.
