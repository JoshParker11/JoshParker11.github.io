---
layout: post
title: Premake [Setup]
description: Initial setup and configuration of our premake build system and Core Engine project # Add post description (optional)
img: premake.png # Add image post (optional)
tags: [Game Engine, Programming] # add tag
---

# Initial Project Setup
I was originally going to use **Sharpmake** as the build system for the project, it is very powerful, fast, and easy for anyone who is familiar with programming since the configuration is done in C#. However, I'm currently running into issues with the most recent distribution and I didn't want to spend any more time debugging instead of working on the project. If the scope of the build system starts getting out of hand I'll switch over and work through the issues, but for the sake of getting up and running ASAP we're going to use **premake5** for the time being, a more than capable build configuration manager, and my preferred system outside **Sharpmake**. 

## Downloading Premake Binaries and License
Follow this link: https://github.com/premake/premake-core/releases

Click the version you want to download, I'm using **Premake 5.0-beta1**
![Premake Download](/assets/img/premake_setup/exe_download.png)

This will download a zip file containing the executable we need. 
![Exe Unzip](/assets/img/premake_setup/exe_archive.png)

**Make sure to grab the LICENSE as well to give proper credit**

License currently at: https://github.com/premake/premake-core/blob/master/LICENSE.txt

Press the Raw button and save the file as LICENSE.txt
![LICENSE Download](/assets/img/premake_setup/license.png)



## Setting up the project
I have a specific way that I like to set up project, so I will explain using the file structure that I plan on using for this project, if you structure your projects differently no worries just slot things in where you need them and adjust file paths accordingly.

Basic structure for the project:

![Project Structure](/assets/img/premake_setup/proj_tree.png)

I've moved the un-zipped premake5.exe under a directory called Vendor/bin/premake/ along with the LICENSE.txt

I've also create a new file called **premake5.lua** *Premake looks for this file by name specifically so make sure it's correct.*

**premake5.lua** is where we'll hold all our project configuration for now (this might get broken up into multiple files eventually).

## Making the Engine project

You can look through the wiki on the github page or a slightly nicer version here: https://premake.github.io/docs/

I won't be going into extreme depth as to how everything works, so feel free to pursue the docs and search for concepts that I didn't cover. I would specifically look at (https://premake.github.io/docs/Tokens/#value-tokens) to see some predefined values.

### Some initial tags of note

* **workspace** - Name of the solution that is generated
* **project** - Name of a particular project (everything written under this will be a part of that project until another is specified or another variable is declared)
* **location** - Name of folder to emit the main artifact to (.vcxproj, ...) *relative to the script file*
* **files** - Relative paths to the files that should be included in the project
* **targetdir** - Output path of build artifacts (.lib, .pdb, ...)
* **objdir** - Output path of intermediate build artifacts (.obj, .log, ...)

Here is some initial configuration that will create a solution called Engine with a project called Engine within it:

{% highlight lua %}
workspace "Engine"
	architecture "x64"

	configurations
	{
		"Debug",
		"Release",
		"Ship"
	}

outputdir = "%{cfg.buildcfg}-%{cfg.system}-%{cfg.architecture}"

project "Engine"
	location "Engine"
	kind "StaticLib"
	language "C++"
	cppdialect "C++17"
	staticruntime "off"

	targetdir ("bin/" .. outputdir .. "/%{prj.name}")	
	objdir ("bin-obj/" .. outputdir .. "/%{prj.name}")

	files
	{
		"%{prj.name}/src/**.h",
		"%{prj.name}/src/**.cpp"
	}	
{% endhighlight %}

A couple of things to note; Since this is written in **lua** you can declare variables and use them like you would any other script, for us this is currently *outputdir* since we will use this format in multiple places it's nice to pull that out into a variable for easy manipulation later and less code duplication. I've also set up the core Engine project as a "StaticLib", this means it will be built as a .lib which is the current architecture I have planned for the project. The core engine libraries will be statically linked to the game/editor projects, this may change in the future but for now is what I'm going with.

By running: 
![Run Manual](/assets/img/premake_setup/run_premake_manual.png) 
*or the path to whatever directory you have premake5.exe and whatever version of vs you're building for*

You should be able to see two new generated files, **Engine.sln** in the same directory as the script file, and **Engine/Engine.vcxproj**

## Potential Errors and Fixes?

## Making the Testbed project

We will be setting up two initial test projects, the first is **Engine** that we configured above, the second is **Testbed**. Since we are building the core engine as a static library that can't be executed on its own, we need an application that we can test features with while in development.

{% highlight lua %}
project "Testbed"
	location "Testbed"
	kind "ConsoleApp"
	language "C++"
	cppdialect "C++17"
	staticruntime "off"

	targetdir ("bin/" .. outputdir .. "/%{prj.name}")	
	objdir ("bin-obj/" .. outputdir .. "/%{prj.name}")

	files
	{
		"%{prj.name}/src/**.h",
		"%{prj.name}/src/**.cpp"
	}

	includedirs 
	{
		"Engine/src"
	}

	links
	{
		"Engine"
	}
{% endhighlight %}

The setup is basically the same as the previous project, a few key differences are; one *kind* is now "ConsoleApp" so it will be set up as an **.exe** and we added two additional sections *includedirs* and *links*. *includedirs* is equivalent to manually adding a path to **Additional Include Directories** in the Property page of Visual Studio. *links* sets up the dependency that Testbed has on the Engine.lib.

## Making a bat file to run the steps for us

One quality of life thing I like doing is to make a .bat file that runs the build command for me so I can just double-click and have everything setup instead of always opening a terminal and typing the command. 

In the root directory of the project create a file called build.bat (or whatever suits you) and add the following lines:

{% highlight bat %}
call Vendor\bin\premake\premake5.exe vs2019
PAUSE
{% endhighlight %}

Now you can just double-click the build.bat script and it will generate your solution/projects. I will probably extend this later to take a couple parameters when further fine-tuning is required. If you want the script to launch VS for you, or to have a version of it that does, you can simply add:

{% highlight bat %}
call Vendor\bin\premake\premake5.exe vs2019
Engine.sln
PAUSE
{% endhighlight %}

If you run this now, a new solution should open up in vs2019, mine looks like:

![Solution Launched](/assets/img/solution_launched.png) 

The .cpp and .h files are empty and just for the purposes of showing how the script collects files in the respective /src/ directories.

This concludes the initial setup of our project! 

We will be adding more to **premake5.lua** as the project grows (specifically managing some flags and defines set in debug vs release vs ship) but this will get us off the ground and finally onto coding! 

## Next time!
Next, we will be setting up the communication layer between the game project and the core engine, as well as taking a pass at some basic logging functionality so we can see what's going on. See you then!
