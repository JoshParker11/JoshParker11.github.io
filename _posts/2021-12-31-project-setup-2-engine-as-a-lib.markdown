---
layout: post
title: Engine vs Game
description: Discussion on how we want to organize game specific code vs engine utilities # Add post description (optional)
img: game_vs_engine.png # Add image post (optional)
tags: [Game Engine, Programming] # add tag
---

*Disclaimer: I'm intentionally trying to go with the first reasonable solution I think of to solve the current problem (with some thinking ahead and lessons learned from the past), so there's a chance I'll think of a better solution in the future and the code will be different.*  

## Key Thoughts

* How coupled or decoupled your engine is from your game depends on your situation.
* It can range from everything in a single executable, to something as lightweight as including some header only libraries.
* I'm going to build my engine as a static library and link it into the game code, the engine will handle a good portion of the setup and teardown of the environment, and work with the game by providing callbacks the game can plug into.

## Problem: Where does the boundary between game and engine begin?

This is a fairly individual question to ask, and the answers can vary quite a bit. In most of my previous projects I never bothered to make any real distinction, the purposes of those engines were to make one specific game so a tight coupling between the two wasn't a problem. However, for this project, while I do have a first game somewhat in mind, I plan on making many projects using this code base, so I want to go with an architecture that makes it reusable without being too tied up in endless abstraction. 

Another question to ask is, how much control/responsibility do you want to put on the user (game) side? Should the engine simply be a set of utilities and libraries, or should it manage some internal state and handle some heavy lifting?

I've answered these questions for myself, and I'll be discussing some potential architectures and why I chose what I chose.

## Engine.h

As I've moved from project to project I find myself re-writing many of the same features. I do my best to take existing code when I can and slot it in, but it's not always that easy. The first architecture I considered is a series of lightweight header only libraries, that either got compiled into a single include or linked via a single header that includes the rest of the libraries. 

### Pros

- Simplicity! - The ability to drag and drop a single file into any project and instantly have access to useful helper functions and utilities is not nothing. If all the logic is contained in a single file you don't have to worry about additional dependencies, everything "should" work right out of the box. 
- Reusability - The types of utilities I think of when thinking about this setup include things like *math.h* that would include tons of functions for 2D/3D vector math, matrix manipulation, quaternions, and so on. This file could be dropped into any project that required math operations even projects outside the games sphere.

### Cons

- The types of utilities you can provide with this method are realistically constrained to systems that don't manage internal state. If you include a header file with the intention of calling a few helpful methods to make your life easier, in my mind, you don't really expect to have to set up and tear down a bunch of additional structures. So every call to a function should be constrained to that function and any additional internal logic. 

## Engine.lib / Engine.dll

Range of possible implementations here, going from a similar structure as above with minimal helper libraries that are just linked in, up to something that holds all the state for the core systems of the engine. For the latter, the game only registers a few callbacks to the engine to interface, and all the state, initialization, and teardown is dealt with in the lib / dll.

### Pros

- Cleaner lines between game and engine. By splitting the code between two projects, if you maintain discipline, it's easier to keep game code on the game side, and engine code on the engine side, without crossing boundaries for convenience.  
- Reusable code base. If you succeed at the first point, the engine lib will remain game agnostic and therefor can be linked into any potential game and have an immediate jumping off point for developing it. 

### Cons

- Potentially stuck with a fixed interface. If you tend more towards the involved engine.lib, where state, startup, and shutdown types of things are handled engine side, you may be stuck with the interface that you provide in subsequent games, unless you're willing to rewrite a decent chunk of core code. This isn't as much of a concern for me because I expect to be the only real customer of my engine, so I'll make it how I like, and if I really needed to change core code, I was the one who wrote it, so I could change it if I liked. Just something to keep in mind if you ever wanted to distribute your engine.

## Game.exe

The final setup I considered is how I've done basically every other project I've worked on, where the engine and **a** specific game are a single project, and the systems tend towards more highly coupled. 

### Pros

- No wasted time with unnecessary abstractions. If you know the specific problem you're trying to solve (ie. already have a game in mind) then you don't need to concern yourself with making abstractions to allow for a variety of games, and you don't need to build systems that your game doesn't need.

### Cons

- As I mentioned above, with no clear boundary between engine and game, it's easy to tightly couple your systems, potentially making them less flexible to changes in the design of your game, and certainly make it harder to strip out functionality if you wanted to reuse anything. With all the code in one project, it's tempting to solve problems in a brittle way. For instance, if you have an issue with a system not having access to some piece of state that it needs, you could potentially just bake in a pointer to that state so it always has access. However, you're now burdened with the responsibility to manage that pointer or hard reference, and make sure it's up to date. 

## The Path I Chose

For this project I fell somewhere towards the upper end of the Engine.lib variation. I want basically all the platform specific stuff to be dealt with by the engine as well as a good deal of setting up the environment and managing resources. I'm still debating over where the exact line between the two is drawn, if I want to just provide a couple callbacks for init, update, render, and a few others, or if I want the game to have full control over the loop and make it necessary for the game to update the engine systems when it feels like it. Currently, I lean towards the former given that the engine is primarily a tool for me to build with, so if I need finer control I can cross that bridge when I get there. However, there may be benefit in giving back a bit more control to the game if I was to ever publish the full project for anyone to use, or some hybrid opt-in approach. Who knows? We'll get there when we get there, and it shouldn't be too big a lift to move the main loop to the game side, so I'm not worried at the moment.

## Basic Application Interface Setup

We are going to be adding the skeleton to get the game project interacting with the engine. We will be adding 4 files to the engine, and 2 to the game project. Don't worry they're all small files.

First, we're going to add *Application.h* to the engine project. I usually don't like being this generic with naming, this is a **game** engine after all, why not call it *Game.h*. My reasoning is I have a couple non-game projects I'd like to get to at some point and I might as well leverage the work I'm doing here to build them. They are not games, so I figured the generic *application* work better in this case.

*Application.h*
{% highlight c++ %}
#pragma once

struct Game;

struct ApplicationConfig
{
  short m_StartPosX;
  short m_StartPosY;
  short m_StartWidth;
  short m_StartHeight;
  char* m_Name;
};

bool Application_Create( Game* gameInst );
bool Application_Run();
{% endhighlight %}
 
This is the interface I currently have in mind for creating an application, at the moment I'm just passing in a pointer to a Game instance, later, when I add other types of applications I will probably abstract that bit out, or just put it behind #define's. 

ApplicationConfig is just some generic data about all applications, currently it's just some window size info and the name of the application, but this could potentially expand later. We'll come back to Application_Create and Application_Run in a bit, for now, let's define the Game struct.

*ApplicationTypes.h*
{% highlight c++ %}
#include "Application.h"

struct Game
{
  ApplicationConfig m_AppConfig;

  bool( *Initialize )(Game* gameInst);
  bool( *Update )(Game* gameInst, float dt);
  bool( *Render )(Game* gameInst, float dt);
  void( *OnResize )(Game* gameInst, float width, unsigned height);

  // Game-specific game state, created and managed by the game
  void* m_State;
};
{% endhighlight %}

Here is what I have set up for the Game struct, it consists of the config we defined in the previous file, callbacks for the important steps in the game loop (Initialize, Update, and Render), callbacks for other important events (the window being resized in this case), and a pointer to the internal state of the game. I'm not sure if I'll keep this state here or not, it's currently here for convenience. I'm also not sure if I'll keep the event callbacks here, stuff like window resized events are important, but I might leave that entirely up to the game side to deal with once I have the event system in place.

Now we can start getting into the actual engine/game connection code. The third file we're going to add to the engine side is Entry.h, again I dislike generic names like entry point, but I'm truly terrible at naming anything really, so I'll stick with the usual application naming.

*Entry.h*
{% highlight c++ %}
#include "Application.h" // I know this is included in ApplicationTypes.h just being explicit
#include "ApplicationTypes.h"

#include <iostream> // TEMP

// Defined by a specific game
extern b8 CreateGame( Game* outGame );

// Main entry-point into the application
int main( void )
{
  // Request the game instance from the application
  Game gameInst;
  if ( !CreateGame( &gameInst ) )
  {
    std::cout << "Could not create the game" << std::endl;
    return -1;
  }

  // Validate the game instance was set up properly
  if ( !gameInst.Initialize || !gameInst.Update || !gameInst.Render || !gameInst.OnResize )
  {
    std::cout << "The games function pointers must be assigned!" << std::endl;
    return -2;
  }

  // Initilization
  if ( !Application_Create( &gameInst ) )
  {
    std::cout << "Application failed to create" << std::endl;
    return 1;
  }
  
  // Begin Game Loop
  if ( !Application_Run() )
  {
    std::cout << "Application did not shutdown properly" << std::endl;
    return 2;
  }

  return 0;
}
{% endhighlight %}

The first line to note is the extern of the function CreateGame. Since the **main** function is defined on the library side, the actual game project will not have a main function and instead will implement this extern CreateGame function as the entry point into the application. 

Assuming the game has implemented CreateGame, the program will start executing in main. A Game struct will be filled out by the game, we'll validate that all the callbacks are set, then we will call the engine side Application_Create that we defined in Application.h. Assuming that was successful, we will jump into the engine side Application_Run.

We'll finish up the engine side by looking at how Application_Create and Application_Run are implemented in Application.cpp before jumping over to the game code to see how we implement CreateGame.

*Application.cpp* 
{% highlight c++ %}
#include "Application.h"
#include "ApplicationTypes.h"

struct ApplicationState
{
  Game* gameInstance;
  bool m_IsRunning;
  bool m_IsSuspended;
  PlatformState m_Platform;
  short m_Width;
  short m_Height;
  double m_LastFrameTime;
};

static bool g_Initialized = false;
static ApplicationState g_AppState;

bool Application_Create( Game* gameInst )
{
  if ( g_Initialized )
  {
    return false;
  }

  g_AppState.gameInstance = gameInst;

  g_AppState.m_IsRunning = true;
  g_AppState.m_IsSuspended = false;

  // COMING SOON: Platform layer startup
  
  // Run the clients initialization steps
  if ( !g_AppState.gameInstance->Initialize( g_AppState.gameInstance ) )
  {
    return false;
  }

  g_Initialized = true;

  return true;
}

bool Application_Run()
{
  while ( g_AppState.m_IsRunning )
  {
    if ( !g_AppState.m_IsSuspended )
    {
      // Call the games update routine
      if ( !g_AppState.gameInstance->Update( g_AppState.gameInstance, (float)0 ) )
      {
        g_AppState.m_IsRunning = false;
        break;
      }

      // Call the games render routine
      if ( !g_AppState.gameInstance->Render( g_AppState.gameInstance, (float)0 ) )
      {
        g_AppState.m_IsRunning = false;
        break;
      }
    }
  }

  g_AppState.m_IsRunning = false;
  // COMING SOON: Platform shutdown
  return true;
}
{% endhighlight %}

This is the meat and potatoes of the engine/game loop, almost any system that the engine starts up will be done here. We also manage calling the game initialize, update, and render functions. A couple things to note, you'll see me use a similar pattern in most of the systems I write, where I'll create a struct with the state for the system and a static instance of that state that is managed by the system as well as any other static bits of state (if the system is initialized in this case). Also, I'm currently just passing 0 for the delta time parameter of Update and Render since I don't have timing set up yet. 

Now that we've got an understanding of the engine side, let's hop over to the game side and see how we tie everything in. There isn't much to talk about here so this will be quick. We'll start with whatever you want to name the definition file for the callbacks, I'm going with Game.h because I'm very creative...

*Game.h*
{% highlight c++ %}
#include <ApplicationTypes.h>

bool TestBed_Initialize( Game* gameInst );
bool TestBed_Update( Game* gameInst, float dt );
bool TestBed_Render( Game* gameInst, float dt );
void TestBed_OnResize( Game* gameInst, unsigned width, unsigned height );
{% endhighlight %}

Nothing to really talk about here, just the game side definitions that match the function pointers in ApplicationTypes.h. The only other thing to note is the use of <> for the includes since the engine code is being linked in as a library. 

*Game.cpp*
{% highlight c++ %}
#include "Game.h"

#include <Entry.h>

struct GameState
{
  float dt;
};

b8 CreateGame( Game* outGame )
{
  // Application configuration
  outGame->m_AppConfig.m_StartPosX = 100;
  outGame->m_AppConfig.m_StartPosY = 100;
  outGame->m_AppConfig.m_StartWidth = 1280;
  outGame->m_AppConfig.m_StartHeight = 720;
  outGame->m_AppConfig.m_Name = "Engine Testbed";

  outGame->Initialize = TestBed_Initialize;
  outGame->Update = TestBed_Update;
  outGame->Render = TestBed_Render;
  outGame->OnResize = TestBed_OnResize;

  outGame->m_State = new GameState;

  return true;
}

bool TestBed_Initialize( Game* gameInst )
{
  return true;
}
bool TestBed_Update( Game* gameInst, float dt )
{
  return true;
}
bool TestBed_Render( Game* gameInst, float dt )
{
  return true;
}

void TestBed_OnResize( Game* gameInst, unsigned width, unsigned height )
{
}
{% endhighlight %}

There's also not much going on here yet, we implement the externed CreateGame function and fill out the Game struct that we were given. We fill out the configuration info, assign all the callbacks, and set the game specific state. Right now that state just consists of the delta time variable, but will expand when we start working on actual game logic I suspect.

That's pretty much everything I have to talk about regarding Engine/Game interface. Some things may change over time if I find I have different needs or think of a better solution. But the concepts will most likely stay the same.

## What's next?

I want to get away from using standard library functions ASAP, the first on the hit list is replacing **iostream** with our own version of logging. In the next couple posts we will be working towards this:

![Logging](/assets/img/logging.png) 

Platform independent colored logging!