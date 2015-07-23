Klawr (Experimental WIP)
================
A set of Unreal Engine 4 plugins that will allow users to write game-play code in C# (and eventually
other CLI languages such as F#), in game projects targeting the Windows platform. If you're looking
for a cross-platform alternative Xamarin has developed a [plugin](https://mono-ue.github.io/) that
provides Mono integration, however a commercial Mono license is not cheap, and it doesn't work in
newer UE versions. The primary aim of this project is to make it possible to do anything that can
currently be done in Blueprints in C# scripts, in order to eliminate the need for complex Blueprint
spaghetti.

The current development focus is on script components (see UKlawrScriptComponent), these are actor
components whose functionality is implemented in a C# class. The C# class attached to a script
component can access any Blueprint accessible property or function exposed by the actor it is
attached to or any of its sibling components (this is still limited at present since only a subset
of all the possible property types is supported). Eventually the properties and methods defined in
a C# script component will also be exposed to Blueprints.

Some time was also spent developing script objects, which allow for "subclassing" in C# of almost
any UObject-derived class, actors included, but that's currently on the back-burner.

NOTE: This project is still highly experimental, and not very useful (though it's getting there).

Check the [Wiki](https://github.com/enlight/klawr/wiki) for additional details.

Prerequisites
========
- Unreal Engine 4.8.0 or later (you'll need the source from GitHub)
- Visual Studio 2013
- .NET Framework 4.5 or later

Overview
======
The core of the CLR hosting code is located in the Klawr.ClrHost libraries, there is a C++ library that bootstraps the CLR, and a couple of C# libraries that handle hosting related tasks and provide some core classes for use by the auto-generated UE4 C# API as well as user scripts. The C++ library is linked against KlawrRuntimePlugin.

**KlawrCodeGeneratorPlugin** generates C# wrappers for UObject subclasses from the reflection information gathered by the **Unreal Header Tool (UHT)** from UFUNCTION and UPROPERTY decorators in the engine/game source. This is the same reflection information that underpins much of the functionality provided by Blueprints.

**KlawrRuntimePlugin** executes user scripts written in C#, scripts have access to the wrapped UE4 API generated by KlawrCodeGeneratorPlugin.

Building
======
This is a two phase process, in the first phase the CLR hosting libraries must be built, in the second phase the plugins themselves.

Source Location
-----------------
The directory hierarchy of this repository matches that of the UE4 source repository, once you check out this project you can just copy the Engine directory into your UE4 source checkout and everything should end up in the right place. However, if you plan on making any changes to the source you'll probably want create a couple of directory junctions instead of copy/pasting files back and forth. To that end I've provided **CreateLinks.bat**, which you can right-click on and **Run as administrator** directly from the checkout root. The batch file will prompt you to enter the path to the UE4 source checkout that you'd like to use to build Klawr, and will then create the necessary directory junctions.

Now everything should be in the right place, from here on any paths are relative to the UE4 source checkout.

Customizing the Unreal Header Tool
----------------------------------
To ensure that the Klawr code generator plugin is built before UHT is executed you'll need to
modify `Engine/Source/Programs/UnrealHeaderTool/UnrealHeaderTool.Target.cs` in the engine source.
A patch file (`UnrealHeaderToolPatch.diff`) with the relevant one-line change is provided in this
repository. However, since it is a one-liner you may find it quicker to make the change by hand,
all you have to do is replace the following line in `UnrealHeaderTool.Target.cs`

``` cpp
AdditionalPlugins.Add("ScriptGeneratorPlugin");
```

with

``` cpp
AdditionalPlugins.Add("KlawrCodeGeneratorPlugin");
```

Libraries
---------
1. Open `Engine\Source\ThirdParty\Klawr\Klawr.ClrHost.sln` in VS2013.
2. Set the `Solution Configuration` to either `Release` or `Debug`.
3. Set the `Solution Platform` to `x64` (this is very important, Win32 builds are not supported yet).
4. Select `Build Solution` to build all the libraries.

Assuming that the build finished with no errors you can move on to phase two.

Plugins
--------
1. Configure UHT to use the Klawr code generator plugin, to do so add/edit the `Plugins` section in
   `Engine\Programs\UnrealHeaderTool\Saved\Config\Windows\Engine.ini` (if the file doesn't exist,
   create it):
    ```
    [Plugins]
    ProgramEnabledPlugins=KlawrCodeGeneratorPlugin
    ScriptExcludedModules=ScriptPlugin
    ScriptExcludedModules=ScriptEditorPlugin
    ScriptExcludedModules=ScriptGeneratorPlugin
    ScriptExcludedModules=KlawrRuntimePlugin
    ScriptSupportedModules=CoreUObject
    ScriptSupportedModules=Engine
    ```
2. Run `GenerateProjectFiles.bat` in your UE4 source checkout.
3. Open `UE4.sln` in VS2013.
4. Set the `Solution Configuration` to `Debug Editor` or `Development Editor`.
5. Set the `Solution Platform` to `Win64` (this is very important, Win32 builds are not supported yet).
6. Select `Build Solution` to build everything.

During the build you may see a bunch of console windows popup briefly, don't panic, this is just
the Klawr code generator plugin building the UE4 C# wrappers assembly. The wrappers assembly can be
rebuilt manually by running `Engine\Intermediate\ProjectFiles\Klawr\Build.bat` from the console.

Using
====
First of all make sure you've got a project open that you don't mind obliterating in case something goes wrong, next enable KlawrRuntimePlugin and KlawrEditorPlugin in UnrealEd, restart as requested.

Now you can [create a script component Blueprint](https://github.com/enlight/klawr/wiki/Creating-a-Script-Component-Blueprint).

License
=====
Klawr is licensed under the MIT license.

Klawr uses the [pugixml](http://pugixml.org/) library to parse XML, it was written by
Arseny Kapoulkine and is licensed under the MIT license.
