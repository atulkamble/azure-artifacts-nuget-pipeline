# azure-artifacts-nuget-pipeline

Here’s a basic but clear example that shows how to use **Azure Artifacts** and **Azure Pipelines** together, especially for building and consuming **.NET (NuGet)** or **npm** packages in CI/CD workflows.

---

## ✅ **1. Azure Artifacts Example**

### A. **Create a NuGet Package and Publish to Azure Artifacts**
You can publish a `.nupkg` file using the `dotnet` CLI:

```bash
# Create the NuGet package
dotnet pack MyLibrary.csproj -o ./nupkgs

# Authenticate to Azure Artifacts
# Replace with your actual token or use Azure DevOps Credential Provider
dotnet nuget add source "https://pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/nuget/v3/index.json" \
  --name "MyFeed" --username "any" --password YOUR_PAT_TOKEN

# Push the package
dotnet nuget push ./nupkgs/MyLibrary.1.0.0.nupkg --source "MyFeed"
```

> You can also do this for npm using `.npmrc` and `npm publish`.

---

## ✅ **2. Azure Pipelines YAML Example (Build + Publish NuGet)**

```yaml
trigger:
- main

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  artifactFeed: 'ORG_NAME/FEED_NAME'  # Replace with your org/feed name

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '7.x.x'
    installationPath: $(Agent.ToolsDirectory)/dotnet

- script: dotnet build MyLibrary.sln -c $(buildConfiguration)
  displayName: 'Build Solution'

- script: dotnet pack MyLibrary/MyLibrary.csproj -c $(buildConfiguration) -o $(Build.ArtifactStagingDirectory)
  displayName: 'Create NuGet Package'

- task: NuGetAuthenticate@1
  displayName: 'Authenticate to Azure Artifacts feed'

- script: dotnet nuget push $(Build.ArtifactStagingDirectory)/*.nupkg --source "https://pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/nuget/v3/index.json" --skip-duplicate
  displayName: 'Push to Azure Artifacts'
```

---

## ✅ **3. Consuming a Package from Azure Artifacts in Another Project**

Update your `nuget.config`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="AzureArtifacts" value="https://pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

Then just use:

```bash
dotnet add package MyLibrary
```

---

## ✅ **If Using npm instead of NuGet:**

### .npmrc file example

```ini
registry=https://registry.npmjs.org/
@your-scope:registry=https://pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/npm/registry/
//pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/npm/registry/:username=AzureDevOps
//pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/npm/registry/:_password=ENCODED_PAT
//pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/npm/registry/:email=you@example.com
//pkgs.dev.azure.com/ORG_NAME/_packaging/FEED_NAME/npm/registry/:always-auth=true
```

> Then you can run `npm publish` or `npm install @your-scope/package-name`

---

Would you like a complete sample GitHub repo or project zip with this setup (NuGet or npm)? Or do you want the **Azure CLI / Bicep deployment integration** as well?
