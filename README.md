# ThinkMeta.SolutionFiles

This repository contains build files that are used across all Visual Studio projects by ThinkMeta.

While it is not intended for use in other projects, you are welcome to copy the repository and modify the files to suit your needs.

# Installation

Clone the repo as a submodule in the root directory of the Visual Studio solution. The files will be placed in the `$(SolutionDir)ThinkMeta.SolutionFiles` folder.

# Activation

## For C# and C++ projects:

Create a "Directory.Build.props" file in the root directory with the following content:

```xml
<Project>
    <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles/ThinkMeta.Build.props"/>
</Project>
```

Also create a "Directory.Build.targets" file in the root directory with the following content:

```xml
<Project>
    <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles/ThinkMeta.Build.targets"/>
</Project>
```

## Central nuget package version management (C# only):

Create a "Directory.Packages.props" file in the root directory with the following content:

```xml
<Project>
    <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles/ThinkMeta.Packages.props" />
</Project>
```

# Customization

## C++

### Defaults

Every C++ project gets these settings:

|Area|Setting|
|---|---|
|Configurations|`Debug` and `Release` for `x64`, toolset `v145`, Unicode character set|
|Compiler|`/W4`, warnings as errors (C4324 disabled), conformance mode, latest C++ standard, C17, multi-processor compilation, fiber-safe optimizations|
|Release|function-level linking, intrinsics, frame pointer omission, whole program optimization with link-time code generation, COMDAT folding, unreferenced code removal|
|Output|`build\$(Configuration)\bin\`, intermediate files in `build\$(Configuration)\obj\$(MSBuildProjectName)\`|
|Modules|one BMI directory for the whole solution, `build\$(Configuration)\bmi\`|
|Linker|subsystem `Console`, debug information in all configurations|

### Solution-specific settings

Settings for all C++ projects of a solution can be defined in `$(SolutionDir)SolutionFiles/$(SolutionName).Cpp.props`,
e.g. `SolutionFiles/MySolution.Cpp.props` for `MySolution.slnx`:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <ItemDefinitionGroup>
        <ClCompile>
            <RuntimeLibrary Condition="'$(Configuration)' == 'Debug'">MultiThreadedDebug</RuntimeLibrary>
            <RuntimeLibrary Condition="'$(Configuration)' == 'Release'">MultiThreaded</RuntimeLibrary>
        </ClCompile>
    </ItemDefinitionGroup>
</Project>
```

* It applies to every C++ project built as part of the solution, **including the projects of submodules**. This is the
  place for switches that all libraries of an executable have to agree on, such as the C runtime.
* It is imported after the defaults above, so it can override them. Settings in a project file and the `TM_NoCrt`
  settings take precedence over it.
* It is only used when the solution is built (Visual Studio or `msbuild MySolution.slnx`). A project built on its own has
  no `$(SolutionName)` and gets the defaults only.
* It is imported after `Microsoft.Cpp.props`: item definitions (`ClCompile`, `Link`, ...) and properties evaluated later
  work, properties that Visual Studio evaluates earlier (e.g. `ConfigurationType`, `PlatformToolset`) do not.

### Properties

You can use the following properties to customize the build:

|Property|Description|
|---|---|
|`TM_DisableBuildHooks`|If set to `true`, the provided C++ .props and .targets files will not be used.|
|`TM_NoCrt`|If set to `True`, the project is built without the C runtime (see below).|
|`TM_NoCrtEntryPoint`|Entry point symbol of a `TM_NoCrt` project. Defaults to `DllMain`.|
|`TM_NoCrtLibraries`|Import libraries a `TM_NoCrt` project links besides `kernel32.lib`, separated by semicolons.|
|`TM_NoCrtVerifyImports`|If set to `False`, the import check after linking a `TM_NoCrt` project is skipped.|
|`TM_DumpBin`|Full path of `dumpbin.exe` for that check, in case it is not found next to the toolset.|

### Building without the C runtime

`TM_NoCrt` belongs in the project's `Globals` property group, because it is evaluated while the
build hooks are imported:

```xml
  <PropertyGroup Label="Globals">
    ...
    <TM_NoCrt>True</TM_NoCrt>
  </PropertyGroup>
```

The resulting binary is independent of the toolset and of `/MD` vs `/MT` in its consumers, needs no
VC++ redistributable, and imports only what `TM_NoCrtLibraries` declares. After linking, the build
checks that nothing pulled the runtime back in and fails otherwise.

In exchange the code must live without the CRT: no global objects with a non-trivial constructor
(static initializers never run), no `thread_local`, no `std::atomic::wait`/`notify`, no floating
point, and `operator new`/`delete` as well as `memset`/`memcpy` have to be provided by the project
itself. Header-only parts of the STL are fine. Modules of such a project are written to their own
BMI directory, since their switches differ from the rest of the solution.

## C#

Custom properties can be defined in `$(SolutionDir)SolutionFiles/$(SolutionName).Net.props`.

You can use the following properties to customize the build (either in the above .props file or in a .csproj file):

|Property|Description|
|---|---|
|<i>Any MSBuild property</i>|If an MSBuild property is defined, it will not be overwritten by later .props files (except where required).|
|`TM_BuildPath`|Sets the build path.<br>The output, intermediate and publish path will be set inside this folder.|
|`TM_NoAnalyzers`|Disables code analysis for the project.|
|`TM_IsTestProject`|If set to `true`, the required properties and package references for test projects will be included.

# Text Editor configuration files

The configuration files are copied to `$(SolutionDir)` when the build is started. You should add these copied files to your VCS ignore list (`.gitignore` for Git).

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
    <VCProjectVersion>18.0</VCProjectVersion>
    <Keyword>Win32Proj</Keyword>
    <ProjectGuid>{0218a0a5-a969-44de-879b-49d831137261}</ProjectGuid>
    <RootNamespace>MyProject</RootNamespace>
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

As above, with few modifications:

```xml
  ...
  <PropertyGroup Label="Configuration">
    <ConfigurationType>StaticLibrary</ConfigurationType>
  </PropertyGroup>
  ...
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
  <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles\ThinkMeta.Cpp.StaticLibrary.props" />
  ...
```

### Projects that are also used as submodules

`$(SolutionDir)` is empty when a project is built on its own. A project that other solutions pull in as a submodule
therefore sets a fallback before the first import, relative to the project file:

```xml
  <PropertyGroup>
    <SolutionDir Condition="'$(SolutionDir)' == ''">$(MSBuildThisFileDirectory)..\</SolutionDir>
  </PropertyGroup>
  <Import Project="$(SolutionDir)ThinkMeta.SolutionFiles\ThinkMeta.Cpp.Configurations.props" />
```

Inside a solution `$(SolutionDir)` is the solution's folder, so the submodule project uses the solution's
ThinkMeta.SolutionFiles and its solution-specific settings.

## .csproj (C#)

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>$(TargetFramework)</TargetFramework>
    </PropertyGroup>
</Project>
```
