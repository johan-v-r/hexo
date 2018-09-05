---
title: .NET Core Global Tools Configuration
date: 2018-09-05 17:16:55
tags:
---

## TL;DR: 

[Nate McMaster's post](https://natemcmaster.com/blog/2018/05/12/dotnet-global-tools/) provides a really nice and detailed explanation on how to configure your `csproj` to build your app as a tool.  
 
```xml
<PropertyGroup>
  <PackAsTool>true</PackAsTool>
  <OutputType>Exe</OutputType>
  <TargetFramework>netcoreapp2.1</TargetFramework>
</PropertyGroup>
```

Build with `dotnet pack` (creates NuGet package to be published)  

> Nice to test by setting up your own [NuGet.Server](https://docs.microsoft.com/en-us/nuget/hosting-packages/nuget-server)

___
## Configuration
The above is quite simple and works as expected. However, some projects might require more configuration.  

I wanted to package an existing console app as a .NET global tool, whilst still keeping the current console apps (multi-target) capabilities. So my original configuration was the standard multi-target specification:  

```xml
<PropertyGroup>
  <TargetFrameworks>netcoreapp2.1;net472</TargetFrameworks>
</PropertyGroup>
```

After adding the `<PackAsTool>true</PackAsTool>` to this configuration the NuGet package failed to build, stating that _...Microsoft.NET.PackTool.targets(32,5): error NETSDK1054: only supports .NET Core..._  

I really didn't want to create a new project, so I eventually got the NuGet package building by splitting out the `PackAsTool` into a specific .NET Core 2.1 `PropertyGroup`:  
``` xml
<PropertyGroup>
  <TargetFrameworks>netcoreapp2.1;net472</TargetFrameworks>
</PropertyGroup>

<PropertyGroup Condition="'$(TargetFramework)' == 'netcoreapp2.1'">
  <PackAsTool>true</PackAsTool>
</PropertyGroup>
```

The package now contained the console in their respective framework's `lib` folder, as well as the `netcoreapp2.1` in the `tools` folder. This package was hosted successfully, but **could NOT be installed**.  

The final result was to build separate NuGet packages. This also meant giving them different names. So I just used a random made-up condition (`GlobalTool`) and kept the original `PropertyGroup` with its default name, and specifying my tool with an explicit `netcoreapp.1` framework a new `PackageId`:  
``` xml
<PropertyGroup Condition="'$(GlobalTool)' != true">
  <TargetFrameworks>netcoreapp2.1;net472</TargetFrameworks>
</PropertyGroup>

<PropertyGroup Condition="'$(GlobalTool)' == true">
  <PackAsTool>true</PackAsTool>
  <TargetFramework>netcoreapp2.1</TargetFramework>
  <PackageId>Foo-Tool</PackageId>
</PropertyGroup>
```

> Note the use of `Condition="'$(GlobalTool)' != true"` instead of `Condition="'$(GlobalTool)' == false"` made my default build & pack use that `PropertyGroup`.  

It's also possible to add (override) any other [NuGet metadata properties](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#nuget-metadata-properties) (i.e. Description) in these sections.

Now my pack process looks like this:  
```
dotnet pack  
dotnet pack /p:GlobalTool=true
```  

This results in 2 NuGet packages, which are both uploaded and can be installed independently. You can then install the global tool with `dotnet tool install -g foo-tool` 

 Because the [`ToolCommandName`](https://natemcmaster.com/blog/2018/05/12/dotnet-global-tools/#package-authoring-and-sdk) defaults to the name of the assembly name, you are still able to use it with `foo`.  

___
 ## Conclusion  
 I was pleasantly surprised with the flexibility of the `csproj` configuration. Making this possible took some digging and experimentation, but good exercise to step outside of the standard tooling.  
 
 It's still very early for the .NET Core global tools, but still awesome and can see this being widely adopted soon.