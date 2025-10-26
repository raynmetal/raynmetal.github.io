---
layout: post
title:  "Two Years of Writing a Game Engine (C++, SDL, OpenGL)"
date:   2025-10-26 12:00:00 +0530
categories: blog technical
tags: [Game of Ur, C++, SDL, OpenGL, 3D, Deferred Shading, MinGW, CMake, Game Engine, Linear Algebra]
---

I've been working on my own game and game engine for the better part of the last 2 years.  I finished work on the engine essentials in June this year, and in the last couple of months wrote a simple (not original) game on top of it, to showcase the engine in action.

I also logged and categorized all the (mostly related) work that I did on a spreadsheet, and made a few fun charts out of them.  If you've ever wondered how long it takes to go from not knowing the first thing about game engines to having made one, I think you should find it interesting.

## Links to the project and related pages

- [Game trailer](https://youtu.be/Il71rep51Es) -- A simple gameplay trailer for the Game of Ur.

- [Game and engine development timeline video](https://youtu.be/1ytOqm6NgKM) -- A development timeline video for the ToyMaker engine and the Game of Ur.

- [Github repo](https://github.com/raynmetal/game-of-ur) -- Where the project and its sources are hosted.  The [releases page](https://github.com/raynmetal/game-of-ur/releases) has the latest Windows build of the game.

- [Documentation](https://raynmetal.github.io/game-of-ur/index.html) -- The site holding everything I've written about (the technical aspects of) the game and the engine.

- [Trello board](https://trello.com/b/LrMfzABA/game-of-ur) -- This is what I've been using to plan development.  I don't plan to do any more work on the project for the time being, but if I do, you'll see it here.

- [Working resources](https://drive.google.com/drive/folders/143lF6BHIolnmm7V8QTZKX0tK-dHCJNz_?usp=drive_link) -- Various recordings, editable 3D models and image files, other fun stuff.  I plan to add scans of my notebooks later on.  Some standouts:

  - [Productivity tracker](https://docs.google.com/spreadsheets/d/15Dyrpi9u48xEYmLvD7jOsMmbripf5tBvuZjaVpBG_Fc/edit?usp=sharing) -- Contains logs of every bit of work I did (or didn't do), and charts derived from them.

  - [References](https://docs.google.com/spreadsheets/d/1QiejopFQt6F_FUrjuMU8kIgUcxQwN-gkdOuSZ4BECYw/edit?usp=drive_link) -- Links to various websites and resources I found interesting or useful during development.

## Notes on the project

### The Engine

The core of ToyMaker engine is my implementation of ECS.  It has a few template and interface classes for writing ECS component structs and system classes.  

One layer above it is a scene system.  The scene system provides a familiar hierarchical tree representation of a scene.  It contains application loop methods callable in order to advance the state of the game as a whole.  It also runs procedures for initializing and cleaning up the active scene tree and related ECS instances.

Built on top of *that* is something I'm calling a SimSystem.  The SimSystem allows "Aspects" to be attached to a scene node.  An Aspect is in principle the same as Unity's MonoBehaviour or Unreal's ActorComponent class.  It's just a class for holding data and behaviour associated with a single node, a familiar interface for implementing game or application functionality.

### Game of Ur

Here's a link to the [game design document](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2game-of-ur_201__design__doc.html) I made for this adaptation. The game implementation itself is organized into 3 loosely defined layers:

- The Game of Ur **data model** is responsible for representing the state of the game, and providing functions to advance it while ensuring validity.

- The **control** layer is responsible for connecting the data model with objects defined on the engine.  It uses signals to broadcast changes in the state of the game, and holds signal handlers for receiving game actions.

- The **visual** layer is responsible for handling human inputs and communicating the current state of the gamehttps://www..

## A rough timeline

The exact things I worked on at any particular point are recorded in [my productivity tracker.](https://docs.google.com/spreadsheets/d/15Dyrpi9u48xEYmLvD7jOsMmbripf5tBvuZjaVpBG_Fc/edit?usp=sharing)  Roughly, though, this is the order in which I did things:

### 2023

1. July - September -- I studied C++, linear algebra, and basic OpenGL.

2. October -- I learned SDL.  I had no idea what it was for before.  Had only a dim idea after.

3. November - 2024 February -- I muscled through the 3D graphics programming tutorials on [learnopengl.com](https://www.learnopengl.com).

### 2024

1. March - August -- I worked on ToyMaker engine's rendering pipeline.

2. August - September -- Wrote my ECS implementation, the scene system, and the input system.

3. September - 2025 January -- Rewrote the scene system, wrote the SimSystem, implemented scene loading and input config loading.

### 2025

1. February -- Rewrote ECS to support instantiation, implemented viewports.

2. March - May -- Implemented simple raycasts, text rendering, skybox rendering.

3. June - August -- Wrote my Game of Ur adaptation.

4. September -- Quick round of documentation.
