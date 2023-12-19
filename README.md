# repro-nuget-feed-auth-issue

The `.csproj` in `repro-nuget-feed-auth-issue` has a dependency on the package `dependency-a` published by `repro-nuget-feed-auth-issue-dependency-a`.

## Setup

- Create feed `internal-sample-feed` in your org
- Replace `{your-org}` in `nuget.config`s
- Create repositories for the two projects and add the respective code within them
- Create pipelines based on `pipeline.yml` within the project folders
- Publish `dependency-a` by running its pipeline
- Run pipeline of the remaining project consuming the published dependency

## Problem

Observe that in Azure DevOps Service everything works fine.

Reproduce on Azure DevOps Server 2022.0.1 (AzureDevOpsServer_20231109.3) and observe that the restore within `dotnet pack` fails with (`DotNetCoreCLI` task version is 2.202.0, agent version is 3.227.2, latest .NET 6 SDK installed):

```
error NU1301: Unable to load the service index for source https://pkgs.dev.azure.com/{your-org}/_packaging/internal-sample-feed/nuget/v3/index.json
```

The issue can be fixed by explicitly running a `dotnet restore` upfront.
We consider this as a workaround as several .NET CLI commands perform an implicit restore which is obviously not working on-prem and because it "just works" in Azure DevOps Service/Cloud.
This would mean that we would need to clutter up pipelines with restore steps just before anything (`dotnet build`, `dotnet test`, `dotnet pack` etc.).