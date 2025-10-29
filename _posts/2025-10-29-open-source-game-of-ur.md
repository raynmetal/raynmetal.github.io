---
layout: post
title:  "Open Source Game of Ur (2023-2025)"
date:   2025-10-29 14:00:00 +0530
categories: project personal
tags: [Game of Ur, C++, SDL, OpenGL, 3D, Deferred Shading, MinGW, CMake, Game Engine, Linear Algebra]
image: /assets/images/board_route.png
image_alt: "The route over which players compete in The Royal Game of Ur."
---

I adapted a version of the Royal Game of Ur using my custom 3D game engine, ToyMaker.  Both the engine and the game are programmed in C++, and make prodigious use of the SDL, Nlohmann JSON and GLM libraries, and the OpenGL graphics API.

![A small clip from the game showing a player's piece overtaking a couple of opponent pieces.]({{- "/assets/images/2025-10-UrMovingStormbirdOvertakingOpponents.gif" | relative_url -}})

![A small clip from the game showing a player's piece being launched and displacing an opponent's piece.]({{- "/assets/images/2025-10-UrLaunchingRoosterDisplacingOpponent.gif" | relative_url -}})

![A small clip from the game showing a player's piece moving along the board to displace an opponent piece.]({{- "/assets/images/2025-10-UrMovingStormbirdDisplacingOpponent.gif" | relative_url -}})

You can download and play the game from [the releases page of the project repo.](https://github.com/raynmetal/game-of-ur/releases)

Game of Ur is a competitive, two-player board game. The player who moves all 5 of their pieces to the end of the course first, wins the game. The variant implemented in this adaptation is based on a paper by Irving Finkel. See the [game design document](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2game-of-ur_201__design__doc.html) for more information.

## Technologies

- ***Programming Languages:*** C++, GLSL

- ***Tools:*** [VSCode](https://code.visualstudio.com/), [CMake](https://cmake.org/), [Doxygen](https://www.doxygen.nl/index.html), [MSYS2](https://www.msys2.org/), [MinGW-w64](https://www.mingw-w64.org/)

- ***Libraries:*** [SDL](https://www.libsdl.org/), [SDL Image](https://github.com/libsdl-org/SDL_image), [SDL TTF](https://github.com/libsdl-org/SDL_ttf), [GLEW](https://github.com/nigels-com/glew), [Nlohmann JSON](https://json.nlohmann.me/), [GLM](https://github.com/g-truc/glm), [Assimp](https://github.com/assimp/assimp)

## Game Architecture

The game scripts are divided into two parts, namely:

- ***[Game Data Model and Controller:](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2game-of-ur_202__data__model__and__controller.html)*** The part of the program responsible for accurately representing the state of the game, as well as validating and enabling updates to that state.

- ***[The Visual and Interaction Layer:](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2game-of-ur_202__data__model__and__controller.html)*** The part of the program responsible for visually representing the state of the game to the user, as well as translating user input to data usable by the game controller.

![Game code architecture for the Royal Game of Ur.]({{- "/assets/images/Game_of_Ur_Architecture.png" | relative_url -}})

The (more or less exact) workings of the game are described in the [project's game design document.](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2game-of-ur_201__design__doc.html)

### Game Model

All the data representing the state of the game and the rules for how the game is played is represented in the game's "data model."  It is a collection of classes, where each class encodes some game related concept (such as [dice](https://raynmetal.github.io/game-of-ur/classDice.html), [game board](https://raynmetal.github.io/game-of-ur/classBoard.html), and [game pieces](https://raynmetal.github.io/game-of-ur/classPiece.html)).

Any single instance of the game is encapsulated in and interfaced with through an orchestrator class, namely the [Game of Ur Model class](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html).  The model is responsible for (correctly) updating the state of the game (through methods such as [roll dice](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html#a810e9d666bd6b27dee986739498fdc01) and [move piece](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html#a4f211f484a692c859d9402f873d52501)), testing actions for validity (eg., [can launch piece](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html#a240c8112d899bcdb7dedbf8c0dc4282f), [can advance one turn](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html#aebf6536a6f15a4932ac60390ad933476)), and report the current state of the game (with methods like [get current player](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html#aad2ebb5fc392c0551b65564624f237f5) and [get all possible moves](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html#afc9176d51fda2b69b988504db4d06cf8)).

### Game Controller

The game controller is a [Sim Object](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1SimObject.html) which maintains its own instance of the [Game of Ur Model](https://raynmetal.github.io/game-of-ur/classGameOfUrModel.html), reports game-related state changes to other objects in [the scene](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2scene__system.html) using [signals](https://raynmetal.github.io/game-of-ur/group__ToyMakerSignals.html), and, through the [player controls](https://raynmetal.github.io/game-of-ur/classUrPlayerControls.html) it instantiates, provides an interface for other objects using the engine to advance the state of the game.

### Interactive and Visual Layer

![Game visuals with labels corresponding to the list below, figure 1, courtesy aankajhoka@gmail.com.]({{- "/assets/images/ui_w_labels_1.png" | relative_url -}})

![Game visuals with labels corresponding to the list below, figure 2, courtesy aankajhoka@gmail.com.]({{- "/assets/images/ui_w_labels_2.png" | relative_url -}})

The visual aspect of the game, which you see on every game screen, is the collection of [UI objects](https://raynmetal.github.io/game-of-ur/group__UrGameUIComponent.html), [3D scene objects](https://raynmetal.github.io/game-of-ur/classBoardLocations.html), and [interaction logic](https://raynmetal.github.io/game-of-ur/classQueryClick.html) which come together to represent and enable interaction with the current state of the game.  The numbers on the diagram above correspond to the descriptions below.

1. The game phase display.  Displays whose turn it is, current game phase, and action presently required.
2. The dice display.  Has a button to roll dice, displays dice state and last turn's roll.
3. The common counters.  No. of counters the winning player can earn.
4. Player panels.  Per player: no. of counters held, and the state of their pieces.  Launchable pieces can be clicked and added to the board.
5. End turn.  Ends current turn, or begins next phase of play.
6. The board.  Divided into
    1. The battlefield, where pieces belonging to either player may be placed.
    2. Black's region.
    3. White's region.
    4. Rosette houses.  Special tiles which allow player to win or lose counters, whose occupant gains knock-out immunity.
7. A game piece.  Clicking on it after a valid roll causes it to move forward.

## Engine Architecture

The engine provides the foundation for all real-time updates and resource management in the application.  It provides a single threaded update loop (through [simulation step](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1SceneSystem.html#a4f869efc4f3ac52dd8638c0717eb991b), [render step](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1SceneSystem.html#a41c08066b8f1ec2cf7dd42519743510e), and [variable step](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1SceneSystem.html#a718fdf56c9d2f93d29a5066124d93194) callbacks), a [native observer pattern implementation](https://raynmetal.github.io/game-of-ur/group__ToyMakerSignals.html), a [scene representation](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1SceneSystem.html) of the world, a [component-first game scripting interface](https://raynmetal.github.io/game-of-ur/group__ToyMakerSimSystem.html), [input abstraction](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2input__system.html), and a [an extensible entity-component-system implementation](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2core_2ecs__system.html).

### ECS

![An example of ECS in action.]({{- "/assets/images/ECS_Example.png" | relative_url -}})

The [engine's ECS implementation](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2core_2ecs__system.html) is the heart of the engine.  It tightly packs related data into [component arrays](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1ComponentArray.html), and provides application loop and lifecycle updates through the [system base class](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1BaseSystem.html).

### Resource Database

![An example scenario involving the storage and use of resources.]({{- "/assets/images/Resource_Database_Example.png" -}})

[The resource database](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2core_2resource__database.html) dynamically loads static resources from file (or as otherwise specified), destroys them when they are no longer needed, and in general prevents the duplication of memory-heavy objects.

### Scene System

[The scene system](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2scene__system.html) provides a spatially/logically hierarchical representation of objects in a scene, in a manner that is familiar to game developers.

[Sim objects and sim object aspects](https://raynmetal.github.io/game-of-ur/group__ToyMakerSimSystem.html) use the scene system to provide a component-first game scripting interface.  Aspects, as implemented here, correspond quite closely to [Godot script nodes](https://docs.godotengine.org/en/stable/getting_started/step_by_step/scripting_first_script.html#creating-a-new-script) and [Unity MonoBehaviours](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.html).

### Render System

[The render system](https://raynmetal.github.io/game-of-ur/md_docs_2systems_2toymaker-engine_2render__system.html) provides a set of classes that represent steps in a rendering pipeline, as well as a (hard-coded) means to glue it all together.  There is much work to be done to make it more flexible.

![Clip showing intermediate textures generated in the rendering pipeline]({{- "/assets/images/2025-10-ToyMakerDebugTextures.gif" | relative_url -}})

For the time being, the render system implements deferred shading using the Blinn-Phong model for lighting.  The diagram below shows, broadly, how it is structured.

![A flowchart describing the rendering pipeline.]({{- "/assets/images/render_pipeline.png" | relative_url -}})

The scene system's [viewport nodes](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1ViewportNode.html) has a special relationship with the render system.  A viewport node which declares its own [ECS world](https://raynmetal.github.io/game-of-ur/classToyMaker_1_1ECSWorld.html#details) also gets its own instance of the render system.  Every viewport node, regardless of whether or not it owns its own ECS world, owns a target texture, which it updates via the render system per its own configuration.  It tells the render system about how it wants the render to take place, eg., through its texture dimensions and scaling mode properties.

## Related Links

- [Github repo](https://github.com/raynmetal/game-of-ur) -- Where the project and its sources are hosted.  The [releases page](https://github.com/raynmetal/game-of-ur/releases) has the latest Windows build of the game.

- [Game trailer](https://youtu.be/Il71rep51Es) -- A simple gameplay trailer for the Game of Ur.

- [Game and engine development timeline video](https://youtu.be/1ytOqm6NgKM) -- A development timeline video for the ToyMaker engine and the Game of Ur.

- [Documentation](https://raynmetal.github.io/game-of-ur/index.html) -- The site holding everything I've written about (the technical aspects of) the game and the engine.

- [Trello board](https://trello.com/b/LrMfzABA/game-of-ur) -- This is what I've been using to plan development.  I don't plan to do any more work on the project for the time being, but if I do, you'll see it here.

- [Working resources](https://drive.google.com/drive/folders/143lF6BHIolnmm7V8QTZKX0tK-dHCJNz_?usp=drive_link) -- Various recordings, editable 3D models and image files, other fun stuff.  I plan to add scans of my notebooks later on.  Some standouts:

  - [Productivity tracker](https://docs.google.com/spreadsheets/d/15Dyrpi9u48xEYmLvD7jOsMmbripf5tBvuZjaVpBG_Fc/edit?usp=sharing) -- Contains logs of every bit of work I did (or didn't do), and charts derived from them.

  - [References](https://docs.google.com/spreadsheets/d/1QiejopFQt6F_FUrjuMU8kIgUcxQwN-gkdOuSZ4BECYw/edit?usp=drive_link) -- Links to various websites and resources I found interesting or useful during development.
