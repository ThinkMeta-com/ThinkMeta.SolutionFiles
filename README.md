# ThinkMeta.SolutionFiles

This repository contains the build files that are used across all Visual Studio projects by ThinkMeta.

While it's not intended to be used by other projects, feel free to copy the repo and modify the files to your own needs.

# Installation

Clone the repo as a submodule in the root directory of the Visual Studio solution. The files will be placed in the `\$(SolutionDir)ThinkMeta.SolutionFiles` folder.

# Activation

## For C# and C++ projects:

Create the "Directory.Build.props" file in the root directory with the following content:

```xml
<Project>
    <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles/ThinkMeta.Build.props"/>
</Project>
```

Currently, a "Directory.Build.targets" file is not needed.

## Central nuget package version management (C# only):

Create the "Directory.Packages.props" file in the root directory with the following content:

```xml
<Project>
    <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles/ThinkMeta.Packages.props" />
</Project>
```

# Customization

## C++

Support for a solution-specific .props file is not implemented yet.

You can use the following properties to customize the build:

|Property|Description|
|---|---|
|`TM_DisableBuildHooks`|If set to `true`, the provided C++ .props and .targets files will not be used.|

## C#

Custom properties can be defined in `\$(SolutionDir)SolutionFiles/\$(SolutionName).Net.props`.

You can use the following properties to customize the build:

|Property|Description|
|---|---|
|<i>Any MSBuild property</i>|If an MSBuild property is defined, it will not be overwritten by the later .props files (except where required).|
|`TM_BuildPath`|Sets the build path.<br>The output, intermediate and publish path will be set inside this folder.|
|`TM_NoAnalyzers`|Disables code analysis for the project.|
|`TM_IsTestProject`|If set to `true`, the required properties and package references for test projects will be included.

# Text Editor configuration files

The configuration files are copied to `\$(SolutionDir)` when the build is started. You should add the copied files to the ignore file of your VCS (`.gitignore` for git).

* C++: `.clang-format`
* C#: `.editorconfig`

# Project file structure

## .vcxproj (C++)

### Console application

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <!-- Load the supported configuration (e.g. Win32/x64, Debug/Release, ...) -->
  <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles\ThinkMeta.Cpp.Configurations.props" />
  <PropertyGroup Label="Globals">
    <VCProjectVersion>17.0</VCProjectVersion>
    <Keyword>Win32Proj</Keyword>
    <ProjectGuid>{0218a0a5-a969-44de-879b-49d831137261}</ProjectGuid>
    <RootNamespace>Episode01</RootNamespace>
    <WindowsTargetPlatformVersion>10.0</WindowsTargetPlatformVersion>
  </PropertyGroup>
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
  <ItemGroup>
    <ClCompile Include="MyFile.cpp" />
  </ItemGroup>
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />
  <ImportGroup Label="ExtensionTargets">
  </ImportGroup>
</Project>
```

### Static library

As above, with one additional import:

```xml
  ...
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
  <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles\ThinkMeta.Cpp.StaticLibrary.props" />
  ...
```

## .csproj (C#)

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>$(TargetFramework)</TargetFramework>
    </PropertyGroup>
</Project>
```