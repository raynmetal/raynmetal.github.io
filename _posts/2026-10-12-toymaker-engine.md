---
layout: post
title:  "ToyMaker Game Engine (2023 onwards)"
date:   2026-10-11 14:00:00 +0530
categories: project personal
tags: [ToyMaker, C++, SDL, OpenGL, 3D, Deferred Shading, CMake, Game Engine, Linear Algebra, XPBD, Tools]
image: /assets/images/board_route.png
image_alt: "The route over which players compete in The Royal Game of Ur."
excerpt_separator: <!--end excerpt-->
---

[ToyMaker](https://www.github.com/raynmetal/toymaker) is an open source, extensible, single-threaded 3D game engine written in C++ on top of [SDL3](https://wiki.libsdl.org/SDL3/FrontPage) and [OpenGL.](https://www.opengl.org/)

<!--end excerpt-->

In ToyMaker, scene data is described in JSON and managed through a combination of ECS and scene tree APIs. Demo projects using ToyMaker have been compiled successfully for Windows and Linux.

- ***[ToyMaker GitHub Repository](https://www.github.com/raynmetal/toymaker)***
- ***[ToyMaker Documentation](https://raynmetal.github.io/toymaker/index.html)***
- ***[ToyMaker Trello Board](https://trello.com/b/ALP2KjNp/toymaker-engine)***

## Technologies

- ***Programming Languages:*** C++, GLSL
- ***Tools:*** [CMake](https://cmake.org/), [Doxygen](https://www.doxygen.nl/index.html) [Clang](https://clang.llvm.org/get_started.html), [mingw-w64](https://www.mingw-w64.org/), [RenderDoc](https://renderdoc.org/)
- ***Libraries:*** [SDL](https://www.libsdl.org/), [SDL Image](https://github.com/libsdl-org/SDL_image), [SDL TTF](https://github.com/libsdl-org/SDL_ttf), [SDL Mixer](https://github.com/libsdl-org/SDL_mixer), [GLEW](https://github.com/nigels-com/glew), [Nlohmann JSON](https://json.nlohmann.me/), [GLM](https://github.com/g-truc/glm), [Assimp](https://github.com/assimp/assimp)

## History

ToyMaker started out as a personal learning project in 2023, when I began
studying OpenGL and learning graphics programming.  It was undifferentiated from
[Game of Ur]() until 2026, when it was moved into its own respository.
ToyMaker's commit state history leading up to the division may be browsed
[on this Game of Ur tag,]() marking the separation of ToyMaker from Game of
Ur.

## Architecture

blah

### Entity-Component-System (ECS)

blah

### Resource Database

blah

### Update loop

blah

### Scene System

blah

### Rendering

blah

### Physics

blah

### Inputs

blah

### Window context

blah

### Signals

blah

## Anatomy of an update

blah

### Fixed update

blah

### Variable update

blah

### Input piping

blah

## References

blah

