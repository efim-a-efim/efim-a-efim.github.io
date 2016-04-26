---
layout: post
category: msbuild
title: "Applying Web.config transforms to all config files"
tags :
 - .net
 - msbuild
 - devops
 - build
 - config
 - transform
---
{% include JB/setup %}

## Situation

You have a .NET Web app or just a .NET application that has config files. You need this config files to be different for developer's machine, CI server and Production. Also you have some text/HTML/JS/CSS files that must be adjusted for development, production and CI.

## Solution

You'll be surprized, but even native MSBuild with Web extensions can do it properly. Brief description follows:

### 1. Create solution configurations

Open your `.sln` in Visual Studio and open Configurations manager (click on your current build configuration dropdowh at the top of the window and select Configuration manager).

Create a new configuration(s), one for every needed config files modification. E.g. if you need separate config transformation logic for CI server and Production, create `CI` and `Production` configurations. You can create empty configurations or clone them from existing ones. I usually clone from `Release` config. Checkbox `Create project configurations` must be checked.

### 2. Create config transformations

If you're not familiar with [Web config transforms](https://msdn.microsoft.com/en-us/library/dd465326(v=vs.110).aspx), go and read the docs! Lots of them over the Internet...

For every config file you have, you should create file(s) named like `<FileName>.<Configuration>.<Extension>`, example: for `App.config` create `App.CI.config` and `App.Production.config`. Put corresponding transformations into them.

### 3. Create replacement files

If you have some plaintext files or other non-XML files to be modified, just create replacement files for all of them like `<FileName>.<Configuration>.<Extension>`. Put all needed content to replacement files. And yes, you'll need to duplicate some parts.

### 4. Add files to projects

Now we need to add our transforms to project files and tell MSBuild that they are dependent upon our config/original files.

Open your project(s) (`.csproj`, `.vbproj`, etc.) in text editor. Find your config file record, it looks like this:

```xml
<Content Include="Web.config">
  <SubType>Designer</SubType>
</Content>
```

You should replace it with this:

```xml
<Content Include="Web.config">
  <SubType>Designer</SubType>
  <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</Content>
<None Include="Web.CI.config">
  <DependentUpon>Web.config</DependentUpon>
  <CopyToOutputDirectory>Never</CopyToOutputDirectory>
  <SubType>Designer</SubType>
</None>
<None Include="Web.Production.config">
  <DependentUpon>Web.config</DependentUpon>
  <CopyToOutputDirectory>Never</CopyToOutputDirectory>
  <SubType>Designer</SubType>
</None>
```

You see, we've set some additional properties. To be short: we made config file to copy to output folder after build and denied copying for transforms.

In `DependentUpon` tag, always use file paths relative to your project directory.

Now you can open your solution in Visual Studio and see how pretty are your config files with folded-in transforms :)

### 5. Run transforms on config files

Here's a sample MSBuild XML that does the following:

1. Run config transforms on all XML configs. It detects App.config and uses special logic for it.
2. Replace needed files.

Place this XML in your project file, preferrably after PropertyGroup definitions in file's start.

```xml
  <UsingTask TaskName="TransformXml" AssemblyFile="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v10.0\Web\Microsoft.Web.Publishing.Tasks.dll" />
  <ItemDefinitionGroup>
    <_FilesToTransform>
      <IsAppConfig>false</IsAppConfig>
    </_FilesToTransform>
  </ItemDefinitionGroup>

  <Target Name="CopyTransformedConfigs" DependsOnTargets="DiscoverFilesToTransform;TransformAllFiles;" AfterTargets="TransformAllFiles;PipelinePreDeployCopyAllFilesToOneFolder" BeforeTargets="OctoPack">
    <Copy 
        SourceFiles="@(_FilesToTransformNotAppConfig->'$(OutDir)%(RelativeDir)%(Filename)%(Extension)')"
        DestinationFiles="@(_FilesToTransformNotAppConfig->'$(PackageTempRootDir)\PackageTmp\%(RelativeDir)%(Filename)%(Extension)')"
        />
    <Copy 
        SourceFiles="@(_FilesToReplace->'$(OutDir)%(DependentUpon)')"
        DestinationFiles="@(_FilesToReplace->'$(PackageTempRootDir)\PackageTmp\%(DependentUpon)')"
        />
  </Target>
  
  <Target Name="TransformAllFiles" DependsOnTargets="DiscoverFilesToTransform;" AfterTargets="Build;_CopyAppConfigFile" BeforeTargets="OctoPack">
    <!-- Now we have the item list _FilesToTransformNotAppConfig and _AppConfigToTransform item lists -->
    <!-- Transform the app.config file -->    
    <ItemGroup>
      <_AppConfigTarget Include="@(AppConfigWithTargetPath->'$(OutDir)%(TargetPath)')" />
    </ItemGroup>
    
    <PropertyGroup>
      <_AppConfigDest>@(_AppConfigTarget->'%(FullPath)')</_AppConfigDest>
    </PropertyGroup>

    <MakeDir Directories="@(_FilesToTransformNotAppConfig->'$(OutDir)%(RelativeDir)')"
             Condition="Exists('%(RelativeDir)%(Filename).$(Configuration)%(Extension)')"/>
    <MakeDir Directories="@(_FilesToReplace->'$(OutDir)%(RelativeDir)')"
             Condition="Exists('%(RelativeDir)%(Filename).$(Configuration)%(Extension)')"/>

    
    <TransformXml Source="@(_AppConfigToTransform->'%(FullPath)')"
                  Transform="%(_Transform)"
                  Destination="$(_AppConfigDest)"
                  Condition=" Exists('%(RelativeDir)%(Filename).$(Configuration)%(Extension)') " />

    
    <TransformXml Source="@(_FilesToTransformNotAppConfig->'%(FullPath)')"
                  Transform="%(_Transform)"
                  Destination="@(_FilesToTransformNotAppConfig->'$(OutDir)%(RelativeDir)%(Filename)%(Extension)')"
                  Condition=" Exists('%(RelativeDir)%(Filename).$(Configuration)%(Extension)') " />

    <Copy 
        SourceFiles="@(_FilesToReplace)"
	DestinationFiles="@(_FilesToReplace->'$(OutDir)%(DependentUpon)')" />
  </Target>
  
  <Target Name="DiscoverFilesToTransform">
    <!-- 
    This will look through items list: None & Content for those
    with Metadata <TransformOnBuild>True</TransformOnBuild>
    -->
    <ItemGroup>
      <_Transforms Include="@(None);@(Content);@(Resource);@(EmbeddedResource)"
                         Condition="$([System.String]::Copy('%(Filename)%(Extension)').EndsWith('.$(Configuration)%(Extension)')) AND ( '%(Extension)'=='.xml' OR '%(Extension)'=='.config' )"/>
     <_FilesToTransform Include="@(_Transforms->'%(DependentUpon)')">
       <_Transform>%(_Transforms.Identity)</_Transform>
     </_FilesToTransform>
     <_FilesToReplace Exclude="@(_Transforms)" Include="@(None);@(Content);@(Resource);@(EmbeddedResource)" Condition="$([System.String]::Copy('%(Filename)%(Extension)').EndsWith('.$(Configuration)%(Extension)'))" />
    </ItemGroup>    

    <PropertyGroup>
      <_AppConfigFullPath>@(AppConfigWithTargetPath->'%(RootDir)%(Directory)%(Filename)%(Extension)')</_AppConfigFullPath>
    </PropertyGroup>

    <!-- Now look to see if any of these are the app.config file -->
    <ItemGroup>
      <_FilesToTransform Condition=" '%(FullPath)'=='$(_AppConfigFullPath)'">
        <IsAppConfig>true</IsAppConfig>
      </_FilesToTransform>
    </ItemGroup>
          
    <ItemGroup>
      <_FilesToTransformNotAppConfig Include="@(_FilesToTransform)"
                                     Condition=" '%(IsAppConfig)'!='true'"/>
      
      <_AppConfigToTransform  Include="@(_FilesToTransform)"
                              Condition=" '%(IsAppConfig)'=='true'"/>
    </ItemGroup>

    <Message Text="FilesToTransform: %(_FilesToTransform.Identity)" Importance="High"/>
    <Message Text="FilesToReplace: %(_FilesToReplace.Identity) %(_FilesTmp.DependentUpon)" Importance="High"/>
  </Target>

  <PropertyGroup>
    <BuildDependsOn>
      $(BuildDependsOn);
      TransformAllFiles
    </BuildDependsOn>
  </PropertyGroup>
```

For Web projects, use the following:

```xml
  <UsingTask TaskName="TransformXml" AssemblyFile="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v10.0\Web\Microsoft.Web.Publishing.Tasks.dll" />
  <Target Name="TransformAllFiles" BeforeTargets="OctoPack;">
    <ItemGroup>
      <_Transforms Include="@(None);@(Content);@(Resource);@(EmbeddedResource)" Condition="$([System.String]::Copy('%(Filename)%(Extension)').EndsWith('.$(Configuration)%(Extension)')) AND ( '%(Extension)'=='.xml' OR '%(Extension)'=='.config' )" />
      <_FilesToTransform Include="@(_Transforms->'%(DependentUpon)')">
        <_Transform>%(_Transforms.Identity)</_Transform>
      </_FilesToTransform>
      <_FilesToReplace Exclude="@(_Transforms)" Include="@(None);@(Content);@(Resource);@(EmbeddedResource)" Condition="$([System.String]::Copy('%(Filename)%(Extension)').EndsWith('.$(Configuration)%(Extension)'))" />
    </ItemGroup>
    <Message Text="FilesToTransform: %(_FilesToTransform.Identity)" Importance="High" />
    <Message Text="FilesToReplace: %(_FilesToReplace.Identity) %(_FilesTmp.DependentUpon)" Importance="High" />

    <Copy SourceFiles="@(_FilesToTransform)" DestinationFiles="@(_FilesToTransform->'%(RelativeDir)%(Filename).Base%(Extension)')"  Condition="Exists('%(RelativeDir)%(Filename).$(Configuration)%(Extension)')" />
    <TransformXml Source="@(_FilesToTransform->'%(RelativeDir)%(Filename).Base%(Extension)')" Transform="%(_Transform)" Destination="@(_FilesToTransform)" Condition=" Exists('%(RelativeDir)%(Filename).$(Configuration)%(Extension)') " />

    <Copy SourceFiles="@(_FilesToReplace)" DestinationFiles="@(_FilesToReplace->'%(DependentUpon)')" />
  </Target>

  <PropertyGroup>
    <BuildDependsOn>
      $(BuildDependsOn);
      TransformAllFiles
    </BuildDependsOn>
  </PropertyGroup>
```

It has a side effect: while building, it replaces your `Web.config` files with transformed ones (though you can find original content in `Web.Base.config`). Originally, transforms run in MSBuild only when you publish the project and this problem doesn't appear. 
