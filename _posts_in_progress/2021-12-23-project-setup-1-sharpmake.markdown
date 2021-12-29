---
layout: post
title: Sharpmake [Setup]
description: Overview of building and setting up Sharpmake for managing visual studio projects # Add post description (optional)
img: sharpmake.png # Add image post (optional)
tags: [Game Engine, Programming, Build System] # add tag
---

Potential issue - CA1416: Validate platform compatibility (I ran into this running the default bootstrap.bat after cloning the repo, possibly due to my inactivated windows license?)

# Why Sharpmake?

# Cloning the Sharpmake Repo

# Building Sharpmake 

# Creating a basic project in Sharpmake

# Structure of Our Project
* Core engine as DLL
* Testbed application for development purposes to test engine features
* Game Project Code as EXE
* Gameplay Code as lightweight DLL (for hot reloading purposes)
* Editor as EXE (separation of game app and editor allows for live editing)

# Creating the Engine Project

# Creating the TestBed application 
* In the future we will have the editor automatically generate the solution for a new application through a launcher

# Linking the projects

# Next Steps
* Add teaser for next post