---
layout: post
title:  hello, world
categories: [Project]
---

Old build instructions seem to be vague/old, requiring manual acquisition of any dependencies, filling in all the gaps manually during CMake generation, 

Today, we'll be doing things the easier way:
- Using vcpkg to automatically handle (nearly) all include/library dependencies
- Using CMake integration directly from Visual Studio, without needing to generate solpution/project files 

# Setup

## Git
- Download and install [Git](https://git-scm.com/download/), which we'll use to grab vcpkg, ES source code, and various other things later.

## Visual Studio Community 2019

- Download and run the [Visual Studio Installer](https://visualstudio.microsoft.com/vs/community/).
- On the Workloads page, tick "Desktop Development with C++", which should select all the required components (MSVC build tools, CMake, etc.), then click "Install".

_Note: You could probably get away with a more minimal install with only build tools and a few extra things, then do everything via command line, but I'm unsure of exactly what is required. Good luck if you want to attempt it!_

## Libraries (via vcpkg)

- Use Git to download and install vcpkg, following the [Getting Started](https://vcpkg.io/en/getting-started.html) guide.
- Should be straightforward, but here's the example commands for installing to `c:\src\vcpkg`:
````
cd c:\src
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
.\bootstrap-vgpkg.bat
````
- While still in the `vcpkg` directory, use `vcpkg install` download and compile (nearly) all required dependency packages for EmulationStation:

````
vcpkg install curl freeimage freetype sdl2 rapidjson
````
- Once everything is installed, use this command to make all packages visible to VS2019/CMake:
````
vcpkg integrate install
````

## Libraries (libvlc)

_`//TODO`_
 
# Syncing Project

Two ways we can do this, depending on whether you want to keep everything within VS2019. There's always other options as well, such as [Github Desktop](https://desktop.github.com/).

If you just want to build the latest code for Windows, then you can clone the default master repo - `https://github.com/RetroPie/EmulationStation.git`

However, if you want to make changes and contribute, you'll want to set up your own fork/branch and use that link for the clone. Read the [Getting Started with ES Development](https://retropie.org.uk/forum/topic/10683/getting-started-with-es-development/36) guide on the RetroPie Forums that explains the Git workflow.

## 1) Git (Command Line)

_`//TODO`_

## 2) VS2019 (Git Integration)

_`//TODO`_

# Loading/Configuring Project

Because of VS2019's integrated support for CMake, we don't need to generate an external solution file. Open VS2019, and on the welcome screen, select:

- "Open a Local Folder" > `c:\dev\emulationstation`

After loading the directory, it should automatically find CMakeLists.txt and attempt to generate the project. Check the Output window at the bottom for progress. 

Should see some errors related to `FREETYPE` - This is because we haven't configured CMake yet to properly target x86. Open the CMake Settings window from the top menu bar:

- "Project" > "CMake Settings"

Change the following settings to set up a single configuration, then save the file (Ctrl+S). I'll be picking 'Debug' for this example, but go straight for 'Release' if you want:

- **Config Name:** "Debug" _(or any name that makes sense)_
- **Configuration Type:** "Debug"
- **Toolset:** "msvc_x86"

~ image here plz ~

Any time that CMake files are edited and saved, it should automatically regenerate (if not, select "Project" > "Generate Cache"). Next error we come across should be `VLC_VERSION` - As it's not included with vcpkg and not listed in the CMake variables, we need to add it manually.

Go back to the CMake Settings page, and click on the "Edit JSON" link in the top-right corner to open the raw view, which should look something like this:

````
{
  "configurations": [
    {
      "name": "Debug",
      "generator": "Ninja",
      "configurationType": "Debug",
      "inheritEnvironments": [ "msvc_x86" ],
      "buildRoot": "${projectDir}\\out\\build\\${name}",
      "installRoot": "${projectDir}\\out\\install\\${name}",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": ""
    }
  ]
}
````

Underneath the last `ctestCommandArgs` field, we need to add a list of additional variables:
- `VLC_VERSION` (the downloaded libvlc version, but can be any number > 1.0.0)
- `VLC_INCLUDE_DIR` (path to the libvlc 'include' folder)
- `VLC_LIBRARIES` (path to the two non-x64 .lib folders, separated by a semicolon)

Example shown below for you to copy/paste, but **remember to add the extra comma at the end of the `ctestCommandArgs` line!**

````
    ...
      "ctestCommandArgs": "",
      "variables": [
        {
          "name": "VLC_VERSION",
          "value": "2.2.0",
          "type": "STRING"
        },
        {
          "name": "VLC_INCLUDE_DIR",
          "value": "C:\\src\\lib\\libvlc-2.2.2\\include",
          "type": "PATH"
        },
        {
          "name": "VLC_LIBRARIES",
          "value": "C:\\src\\lib\\libvlc-2.2.2\\lib\\msvc\\libvlc.lib;C:\\src\\lib\\libvlc-2.2.2\\lib\\msvc\\libvlccore.lib",
          "type": "FILEPATH"
        }
      ]
    ...
````

Save again with Ctrl+S, watch CMake in the output window, and it should be successful with no errors - "`CMake generation finished.`"

# Building/Running Project

_`//TODO`_