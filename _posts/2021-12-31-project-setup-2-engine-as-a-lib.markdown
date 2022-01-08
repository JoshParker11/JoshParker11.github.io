---
layout: post
title: Engine vs Game
description: Discussion on how we want to organize game specific code vs engine utilities # Add post description (optional)
img: game_vs_engine.png # Add image post (optional)
tags: [Game Engine, Programming] # add tag
---

*Disclaimer: I'm intentionally trying to go with the first reasonable solution I think of to solve the current problem (with some thinking ahead and lessons learned from the past), so there's a chance I'll think of a better solution in the future.*  

## Key Thoughts

## Problem: Where does the boundary between game and engine begin?

This is a fairly individual question to ask, and the answers can very quite a bit. In most of my previous projects I never bothered to make any real distinction, the purposes of those engines were to make one specific game so a tight coupling between the two wasn't a problem. However, for this project, while I do have a first game somewhat in mind, I plan on making many projects using this code base, so I want to go with an architecture that makes it reusable without being too tied up in endless abstraction. 

Another question to ask is, how much control/responsibility do you want to put on the user (game) side? Should the engine simply be a set of utilities and libraries, or should it manage some internal state and handle some heavy lifting?

I've answered these questions for myself, and I'll be discussing some potential architectures and why I chose what I chose.

## Engine.h

As I've moved from project to project I find myself re-writing many of the same features. I do my best to take existing code when I can and slot it in, but it's not always that easy. The first architecture I considered is a series of lightweight header only libraries, that either got compiled into a single include or linked via a single header that includes the rest of the libraries. 

### Pros



### Cons

- The types of utilities you can provide with this method are realistically constrained to systems that don't manage internal state. If you include a header file with the intention of calling a few helpful methods to make your life easier, in my mind, you don't really expect to have to set up and tear down a bunch of additional structure. So every call to a function should be constrained to that function and any additional internal logic. 

## Engine.lib / Engine.dll

Range of possible implementations here, ranging from a similar structure as above with minimal helper libraries that are just linked in, up to something that holds all the state for the core systems of the engine. For the latter, the game only registers a few callbacks to the engine to interface, and all the state, initialization, and teardown are done in the lib / dll.

### Pros

### Cons

## Game.exe

The final setup I considered is how I've done basically every other project I've worked on, where the engine and **a** specific game are a single project and the systems tend towards more highly coupled. 

### Pros

### Cons

## The Path I Chose

For this project I fell somewhere towards the upper end of the Engine.lib variation. I want basically all the platform specific stuff to be dealt with by the engine as well as a good deal of setting up the environment and managing resources. I'm still debating over where the exact line between the two is drawn, if I want to just provide a couple callbacks for init, update, render, and a few others, or if I want the game to have full control over the loop and make it necessary for the game to update the engine systems when it feels like it. Currently, I lean towards the former given that the engine is primarily a tool for me to build with, so if I need finer control I can cross that bridge when I get there. However, there may be benefit in giving back a bit more control to the game if I was to ever publish the full project for anyone to use, or some hybrid opt in approach. Who knows? We'll get there when we get there, and it shouldn't be too big a lift to move the main loop to the game side, so I'm not worried at the moment.

## Basic Application Interface Setup

*<code snippet of game calling engine setup here>* 
 
## What's next?

*Teaser of our first system: Logging!* 