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

My intentions with documenting my adventures this time around are twofold; firstly, as engines like Unity and Unreal get better and better, I see fewer and fewer people pursuing the rigorous task of building the foundation themselves. This is entirely understandable given the time investment required to take on such a task and the very real possibility that too few people will see the end result to justify it. However, I would like to share any knowledge that I've gained and any I gain along the way, in the hopes that it will make the process easy enough so as not to dissuade anyone before they've begun. Also, many AAA game studios still use custom engines, so if you are reading this and intend on working at one eventually, an understanding of the architecture and underlying systems will serve you well. Secondly, I want to document this journey to reinforce my own understanding and as a way to motivate me to see it through to the end. 

# Series Overview

The series is most likely going to be broken up into multiple stages, development of the core engine, building development tools to create content for the engine, and eventually building our own game. I have a few bits and pieces of the game figured out but for the most part it's TBD, thankfully the first two stages will provide ample time for brainstorming.

There are a few pillars that will influence certain decisions along the way and help me keep a proper heading when I'm inevitably enticed in another direction:
* Prioritize action over consideration
* Write simple interfaces for systems so an off the shelf version can be used and later swapped out for our own implementation
* Program in C++ with the mindset of a C engineer
* 3D world building and exploring are the primary concern, worry about a more generic solution later

These are the main things I currently care most about, these aren't ironclad rules but guiding principles to help direct my efforts productively. The first pillar is the most important and the one I'd consider the most immutable. I tend to get stuck considering every possibility before taking action, when just pursuing a line and figuring out what works and doesn't work is far faster and converges to a better solution quicker in most cases.

The second pillar serves the first. For the sake of prioritizing action, starting the project by leaning on some pre-built code is acceptable, but I want to write the systems in a way that I can swap out my own implementation at a later date.

There are a lot of quality of life features that C++ offers over C that I intend on using, but keeping the mindset of a C engineer helps me avoid using those features as a crutch, and to pick more CPU friendly options when I can. I will most likely use STL containers to begin with but move to my own when I get a chance.

Lastly, the parts of the game I have in mind so far are centered around living, breathing worlds with sprawling terrain, so some initial tooling and systems will be made with the purpose of world building as a priority.

I'll list a few of the systems I plan to work on in each stage, this list is not exhaustive, and a single system like graphics encompasses a vast quantity of subsystems.
 
## Stage 0 [Core Engine]

* Project Setup  
* Platform Layer
    - Data Types
    - Data Structures
    - File System
    - Networking 
    - Threading
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
* Windowing
* Graphics (This is the largest single system)
* Physics
* Audio
* Threading
* Reflection
* Scene Graph / Culling
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
* Porting

### *I hope you'll come along for the ride and I hope you'll find something of value in the process. Let's begin!*