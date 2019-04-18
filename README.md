# dotnet-core-target-manifest-bug
Repo for dotnet-core target manifest bug https://github.com/dotnet/core/issues/2595

The main idea is to not include any Swashbuckle.* dependencies in the published output folder. Those dependencies will be installed in the target system in order to make the deployment smaller. According to [Runtime package store](<https://docs.microsoft.com/en-us/dotnet/core/deploying/runtime-store>) docs this can be done by publishing using a target manifest file.

The current repo solution, specifies the target manifests in the project file. The `artifact.xml` was generated using:

```powershell
dotnet store --manifest project.csproj --framework netcoreapp2.2 --runtime win7-x64 --skip-optimization
```

You can find this command in `create-store.cmd`. Although for some reason the generated target manifest file does not include any Swashbuckle.*. For other projections which includes Swashbuckle it works.

In any case I have included the `artifact.xml` in the solution project.

```xml
<StoreArtifacts>
  <Package Id="Swashbuckle.AspNetCore" Version="4.0.1" />
  <Package Id="Swashbuckle.AspNetCore.Swagger" Version="4.0.1" />
  <Package Id="Swashbuckle.AspNetCore.SwaggerGen" Version="4.0.1" />
  <Package Id="Swashbuckle.AspNetCore.SwaggerUI" Version="4.0.1" />
</StoreArtifacts>

```

`dotnet store` command will output the store artifacts by default here: `C:\Users\admin\.dotnet\store\x64\netcoreapp2.2` which need to be copied into `C:\Program Files\dotnet\store\x64\netcoreapp2.0`

1. ###### 

2. Run the `dotnet publish` command as following:

```powershell
dotnet publish -c Release /p:PublishProfile=Properties\PublishProfiles\FolderProfile.pubxml
```

3. execute \bin\Release\netcoreapp2.2\win7-x64\publish\TargetManifestBug.exe

###### Expected result

The application should start successfully.

###### Actual result

```
Error:
  An assembly specified in the application dependencies manifest (TargetManifestBug.deps.json) was not found:
    package: 'Swashbuckle.AspNetCore.Swagger', version: '4.0.1'
    path: 'lib/netstandard2.0/Swashbuckle.AspNetCore.Swagger.dll'
  This assembly was expected to be in the local runtime store as the application was published using the following target manifest files:
    artifact.xml

```

