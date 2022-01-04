---
layout: post
title: Engine vs Game
description: Discussion on how we want to organize game specific code vs engine utilities # Add post description (optional)
img: game_vs_engine.png # Add image post (optional)
tags: [Game Engine, Programming] # add tag
---

## Problem: Where does the boundary between game and engine begin?

This is a fairly individual question to ask, and the answers can very quite a bit. In most of my previous projects I never bothered to make any real distinction, the purposes of those engines were to make one specific game so a tight coupling between the two wasn't a problem. However, for this project, while I do have a first game somewhat in mind, I plan on making many projects using this code base, so I want to go with an architecture that makes it reusable without being too tied up in endless abstraction. 

Another question to ask is, how much control/responsibility do you want to put on the user (game) side? Should the engine simply be a set of utilities and libraries, or should it manage some internal state and handle some heavy lifting?

I've answered these questions for myself, and I'll be discussing some potential architectures and why I chose what I chose.

## Engine.h

As I've moved from project to project I find myself re-writing many of the same features. I do my best to take existing code when I can and slot it in, but it's not always that easy. The first architecture I considered is a series of lightweight header only libraries, that either got compiled into a single include or linked via a single header that includes the rest of the libraries. 

### Pros



### Cons

- The types of utilities you can provide with this method are realistically constrained to systems that don't manage internal state. If you include a header file with the intention of calling a few helpful methods to make your life easier, in my mind, you don't really expect to have to set up and tear down a bunch of additional structure. So every call to a function should be constrained to that function and any additional internal logic. 
