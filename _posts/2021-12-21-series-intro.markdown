---
layout: post
title: Game Engines [Series Intro]
description: Testing out adding a new post on the site # Add post description (optional)
img: game_engine_cryengine.jpeg # Add image post (optional)
tags: [Game Engine, Programming] # add tag
---

# Why I make Game Engines

I love making games!   
Telling your own story, bringing your imagination to life, creating new worlds for people discover and explore.  
But I also love understanding how things work, breaking them apart to see what lies beneath, and building my own version to affirm my understanding.  
So when I first began learning how to program (with the intention of pursuing a job in games), it's unsurprising that I gravitated towards game engines and understanding how a button press could make a character dance, or how I could see anything on the screen to begin with. The more I've learned over the years the further I want to dig. I've built several game engines in the past, but whenever a new project catches my attention I can't help but throw everything away and start again with the lessons I've learned and a fresh set of ideas. This however, is not conducive to building new games, since by the time I've rebuilt enough of the engine to get something working the spark has usually died out. 

As engines like Unity and Unreal get better and better, I see fewer and fewer people pursuing the rigorous task of building the core technologies themselves. This is entirely understandable given the time investment required to take on such a task and the very real possibility that too few people will see the end result to justify it. However, I would like to share any knowledge that I've gained and any I gain along the way, in the hopes that it will make the process easy enough so as not to dissuade anyone before they've attempted it.

# Who is this series for?

If you are an aspiring game engineer (especially if you want to focus on core technology), and you're looking to join one of the AAA studio's, you're most likely going to be working in a custom engine, so having an understanding of the architecture and underlying systems will serve you well. Additionally, if you are a hobbyist game maker and want more control over the development process, or if you're like me and building things from the ground up floats your boat, then I think you may find this process interesting as well.

I intend on keeping this series somewhere between a high level blog of my experience building this engine, and a full tutorial series. I won't be going over the basics of C or C++, but I will try to link to references whenever I think something warrants more info. Furthermore, I plan on doing a more structured set of tutorials after this project is stable, but during development the tutorials will mostly be ad hoc.

The process of documenting this project is also for my own benefit, as I hope having some type of outlet for my work will keep me motivated to finish something for once!

# Series Overview

The series is going to be broken up into multiple stages, development of the core engine libraries, building development tools to create content for the engine, and eventually building our own game. I have a few bits and pieces of the game figured out but for the most part it's TBD, thankfully the first two stages will provide ample time for brainstorming.

There are a few pillars that will influence certain decisions along the way and help me keep a proper heading when I'm inevitably enticed in another direction:
* Prioritize action over excessive consideration
* Write simple interfaces for systems so an off the shelf version can be used and later swapped out for our own implementation
* Program in C++ with the mindset of a C engineer
* 3D world building and open-world games are the initial use cases, worry about more generic technologies later

These are the main things I currently care about, these aren't ironclad rules but guiding principles to help direct my efforts productively. The first pillar is the most important and the one I'd consider the most immutable. I tend to get stuck considering every possibility before taking action, when just pursuing a line and figuring out what works and doesn't work is far faster and converges to a better solution quicker in most cases.

The second pillar serves the first. For the sake of prioritizing action, starting the project by leaning on some pre-built code is acceptable, but I want to write the systems in a way that I can swap out my own implementation at a later date.

There are a lot of quality of life features that C++ offers over C that I intend on using, but keeping the mindset of a C engineer helps me avoid using those features as a crutch, and to pick more CPU friendly options when I can. I will most likely use STL containers to begin with but move to my own when I get a chance.

Lastly, the parts of the game I have in mind so far are centered around living, breathing worlds with sprawling terrain, so some initial tooling and systems will be made with the purpose of world building as a priority.

I'll list a few of the systems I plan to work on in each stage, this list is not exhaustive, and a single system like graphics encompasses a vast quantity of subsystems.
 
## Stage 0 [Core Engine]

* Project Setup
* Project Specific Defines  
* Platform Layer
    - Windowing
    - File System
    - Networking 
    - Threading
    - Platform Detection 
* Subsystem Management
* Math Library
* Strings and Hashed String Ids
* Parsing
* Engine Configuration
* Async File I/O
* Basic Game Loop
* Logging
* Event Handling
* Input Handling
* Memory Management
* Resource Management
* Unit Testing
* Layers
* Entity Component Systems
* Graphics (This is the largest single system)
* Physics
* Audio
* Threading
* Reflection
* Scene Graph / Culling
* Job System
* Networking
* Profiling
* Scripting
* Animation (Will probably put this off for quite a while)
* UI (Also delaying this)

## Stage 1 [Development Tools]

* Scene Editor
* Terrain Editor
* Asset Converters
* Scripting

## Stage 2 [Make A Game]

* Concept
* Prototyping
* White boxing
* Game Specific Rendering
* Game Specific Subsystems
* Player Mechanics
* Game Camera
* AI
* Creating Assets
* Playtesting
* Shipping

## Stage 3+ [Stretch Goals]

* Porting the engine (and game) to mobile
* Explore supporting WebGL/WebASM
* Replace the renderer with something like Vulkan or DirectX12

### *I hope you'll come along for the ride and I hope you'll find something of value in the process. Let's begin!*