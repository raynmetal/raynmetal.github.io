---
layout: post
title:  "ToyMaker Game Engine (2023 onwards)"
date:   2026-10-14 14:00:00 +0530
categories: project personal
tags: [ToyMaker, C++, SDL, OpenGL, 3D, Deferred Shading, CMake, Game Engine, Linear Algebra, XPBD, Tools]
image: https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExdnI2Ymd5cXptNTV1cGd5ZWs0NmRudzE2OTZ6OGQ5YTJnamhoY2NvYSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/2oSky4S3RpywDDpsu0/giphy.gif
image_alt: "A set of granular single degree of freedom position constraints being used to model the behaviour of a hinge joint in a triple pendulum in ToyMaker."
excerpt_separator: <!--end excerpt-->
---

[ToyMaker](https://www.github.com/raynmetal/toymaker) is an open source, extensible,
single-threaded 3D game engine written in *C++* on top of [SDL3](https://wiki.libsdl.org/SDL3/FrontPage)
and [OpenGL.](https://www.opengl.org/)

<!--end excerpt-->

In ToyMaker, scene data is described in JSON and managed through a combination of ECS and scene tree APIs. Demo projects using ToyMaker have
been compiled successfully for Windows and Linux.

- ***[ToyMaker GitHub Repository](https://www.github.com/raynmetal/toymaker)***
- ***[ToyMaker Documentation](https://raynmetal.github.io/toymaker/index.html)***
- ***[ToyMaker Trello Board](https://trello.com/b/ALP2KjNp/toymaker-engine)***

![Fill Container 1](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExdGpmMGI1bWF0eHVpanplMGVmcXdyc2cybHpidDB0Z2l1eXQ4MmdxeiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/HLqkqke2YgdZUamEdI/giphy.gif)

![Fill Container 2](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExZnVsNzU3c2QyOWtkamUxemNyOGR3Mml1YzJjMjRlaWFiZnB5ZzF1NiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YzVg8ozl9jqvc6I6c3/giphy.gif)

![Fill Container 3](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExdDAyZmh5djV0NWo3Nm13OGVyaGp2dWFxOW5pNXBoM243NXBnc3NqcCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/oaH2X6j6iVvN1c4tIp/giphy.gif)

`Examples/FillContainer` -- shows spheres spawning at a steady rate filling an invisible
container, with several collision checks and corrections taking place every frame.

![Triple Pendulum](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExdnI2Ymd5cXptNTV1cGd5ZWs0NmRudzE2OTZ6OGQ5YTJnamhoY2NvYSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/2oSky4S3RpywDDpsu0/giphy.gif)

`Examples/TriplePendulum` -- shows set of granular single degree of freedom position
constraints being used to model the behaviour of a hinge joint in a triple pendulum.

![Slope Friction](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExbXZqZnZlZ2ZqNXYzeWsxZTNncDlrZGduOTBoYmNqd2E0NW5ib200cSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/hB2tfGISPc9946Vr0v/giphy.gif)

`Examples/SlopeFriction` -- demos friction forces causing a moving block to slow and come
to a stop sliding down a gentle incline.

![Many Lights](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExdTQ5MXlxMHprZzl3OGRmZnJvc2xuZzJvaGNsMWs3c201dGYycThmMyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/hUsAyQAKHBP7KFUr2I/giphy.gif)

`Examples/ManyLights` -- scene containing 441 point lights and 441 models (2 meshes each)
rendered with instancing for both.

![One One Collision](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExemZlYTg0enp6Y2ZudzVpZHozanV3NHQ5cm0xNXBsaTB0N3RtY3hjZCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/4m0mQ5owt4ez6mrmJ2/giphy.gif)

`Examples/LeftRightCollision` -- shows single one-to-one collision between two spheres.

![Resizable UI](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExbnp2YWRpYjRmNnJpejI2OGZ2OWc5OXk1OXQyamJmdmJvMjM4ZGR2MCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/uMj0ixmBKVoMAX3B2P/giphy.gif)

`Examples/UILayout` -- shows UI elements being dynamically repositioned in response to window resizing.

![Interactive Center Text](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExbDJ1aTl3Y3dzMGZ4NGx5d2YzeHJtNzV6cTIycHJhZG96aTl0YTllOSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/MLEgE7qNc53t5HznIb/giphy.gif)

`Examples/CenterText` -- demo featuring clickable text that spins on interaction.

## Technologies

- ***Programming Languages:*** C++, GLSL
- ***Tools:*** [CMake](https://cmake.org/), [Doxygen](https://www.doxygen.nl/index.html) [Clang](https://clang.llvm.org/get_started.html), [mingw-w64](https://www.mingw-w64.org/), [RenderDoc](https://renderdoc.org/)
- ***Libraries:*** [SDL](https://www.libsdl.org/), [SDL Image](https://github.com/libsdl-org/SDL_image), [SDL TTF](https://github.com/libsdl-org/SDL_ttf), [SDL Mixer](https://github.com/libsdl-org/SDL_mixer), [GLEW](https://github.com/nigels-com/glew), [Nlohmann JSON](https://json.nlohmann.me/), [GLM](https://github.com/g-truc/glm), [Assimp](https://github.com/assimp/assimp)

## History

ToyMaker started out as a personal learning project in 2023, when I began studying OpenGL
and learning graphics programming.  It was undifferentiated from [Game of Ur](https://github.com/raynmetal/game-of-ur) until
2026, when it was moved into its own respository. ToyMaker's commit state history leading
up to the division may be browsed [on this Game of Ur tag.](https://github.com/raynmetal/game-of-ur/tree/toymaker-fork)

## Architecture

### Entity-Component-System (ECS)

*ToyMaker* is built on top of an [ECS system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2core_2ecs__system.html) that exists side-by-side with its scene
system.  Scene data, such as [transforms,](https://raynmetal.github.io/toymaker/structToyMaker_1_1Transform.html)
[object bounds (or colliders),](https://raynmetal.github.io/toymaker/structToyMaker_1_1ObjectBounds.html)
[physics state,](https://raynmetal.github.io/toymaker/structToyMaker_1_1PhysicsState.html)
etc., are stored as simple (or as close as possible) *plain-old-data* types, in *tightly
packed component arrays.*  Each component instance is additionally tagged with the *entity ID*
of its owning object.  The *set of all components* across all arrays marked with a specific
entity ID *describe a single object* in the scene that the ID names.

![An example of ECS in action.]({{- "/assets/images/ECS_Example.png" | relative_url -}})

*ECS systems* (derived from [ToyMaker::System](https://raynmetal.github.io/toymaker/classToyMaker_1_1System_3_01TSystemDerived_00_01std_1_1tuple_3_01TListenedForComponents_8_8_8_01b38038026963ebaf6c6a6fed209b61e7.html))
roughly represent the various ways in which *scene data is mutated* over an application's
lifetime.  Each system tends to access no more than a handful of components for each
entity (e.g., the *render system* mainly deals with *transforms and models,* while the *physics
system* cares primarily about *object bounds and physics state*), and declares which
components these are.  Components named by these systems end up becoming *ECS components.*

Keeping component data in tightly packed arrays, coupled with the fact that any single
system only requires data from a handful arrays at most, leads to significant performance
gains owing to [cache locality.](https://en.wikipedia.org/wiki/Locality_of_reference)  For
convenience, however, the [ECS::Entity class](https://raynmetal.github.io/toymaker/classToyMaker_1_1Entity.html)
provides an interface for accessing object data in a near object-oriented fashion.

### Resource Database

*Static resources,* i.e., reusable resources which aren't likely to change over the runtime
of an application, are *tabulated,* *serialized/deserialized,* and *provided shared access
to* through [ToyMaker::ResourceDatabase.](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2core_2resource__database.html)
Examples of data of this kind include [3D models,](https://raynmetal.github.io/toymaker/classToyMaker_1_1StaticModel.html)
[images/textures,](https://raynmetal.github.io/toymaker/classToyMaker_1_1Texture.html)
[sounds,](https://raynmetal.github.io/toymaker/classToyMaker_1_1Sound.html) and even
[shader programs.](https://raynmetal.github.io/toymaker/classToyMaker_1_1ShaderProgram.html)

![An example scenario involving the storage and use of resources.]({{- "/assets/images/Resource_Database_Example.png" -}})

Each resource may be constructed in a variety of ways; models, for example, may be
[generated procedurally,](https://raynmetal.github.io/toymaker/classToyMaker_1_1StaticModelCuboidDimensions.html)
or loaded into memory [from a file.](https://raynmetal.github.io/toymaker/classToyMaker_1_1StaticModelFromFile.html)
Accordingly, each resource, inheriting from [ToyMaker::Resource,](https://raynmetal.github.io/toymaker/classToyMaker_1_1Resource.html)
may be associated with several [resource constructors.](https://raynmetal.github.io/toymaker/classToyMaker_1_1ResourceConstructor.html)

The [resource database](https://raynmetal.github.io/toymaker/classToyMaker_1_1ResourceDatabase.html)
is a *global object* that shares references to *named* resources already in memory, or
otherwise *constructs* that resource according to its *user-provided description,* where
the description is usually registered with the database at the start of the application.

### Update loop

ToyMaker's *update loop* is broadly split into 3 distinct steps:

1. ***The fixed update:***  This is where logic that *ought to be* independent of the speed
of the underlying hardware goes.  ToyMaker's [Physics system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2physics__system.html)
lives here, mostly.  The size of a single fixed-step is configurable.  How often this
runs relative to the variable update depends on a number of factors, including user
hardware, and the number of simultaneously running applications.

2. ***The variable update:***  This is where logic that is less sensitive to time goes.
Typically, scripts will receive input and set input related parameters from here.  Updates
related to game visuals, that don't depend on a reliable simulation, often also go here.

3. ***The render update:***  This is where [ToyMaker's render system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2render__system.html)
will use scene data and render it to a 2D image to be displayed to the user.  How often
this runs once again depends on user hardware, application configuration, and the current
state of the user's machine.

### Scene System

[ToyMaker's scene system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2scene__system.html)
provides a logically hierarchical representation of the scene.  It is comprised of a number
of scene nodes, inheriting from [ToyMaker::SceneNodeCore](https://raynmetal.github.io/toymaker/classToyMaker_1_1SceneNodeCore.html)
where each node holds a weak reference to its parent, and strong references to its
children.

Usually, each child node will apply its *local* transform after its parent
has applied its *global* transform in order to obtain the node's final position,
orientation, and scale.  The *combination* of the current node's local transform and
its parent's world transform becomes the *current node's world transform.*

From the point of view of the resource database, a scene is merely another type of resource
that can be constructed either through [its description,](https://raynmetal.github.io/toymaker/classToyMaker_1_1SceneFromDescription.html)
or loaded from a [scene file.](https://raynmetal.github.io/toymaker/classToyMaker_1_1SceneFromFile.html)

#### Viewports

The scene may also have [viewport nodes.](https://raynmetal.github.io/toymaker/classToyMaker_1_1ViewportNode.html)
A viewport node, along with its associated camera, provide a rendering target for a
single ECS world.  Multiple viewport nodes can be associated with one ECS world,
potentially showing the state of the world with different perspectives and rendering
configurations.  Alternatively, a viewport node may opt to own its own [ECS world,](https://raynmetal.github.io/toymaker/classToyMaker_1_1ECSWorld.html)
marking the boundary between its parent's world and its own.

A viewport node with its *own ECS world* is useful for a number of things, including making
a UI overlay over a 3D scene, or making a game-in-game style feature. A viewport node that
shares its world with its parent can be used to implement interesting features,
like reflections, FPS scopes with a different FOV from the regular first-person
camera, or a split-screen for local multiplayer.

#### Aspects & Sim Objects

Quite often it is the case that developers would like to define behaviour directly
on top of the object that should exhibit that behaviour -- as opposed to fiddling with
the constraints and indirectness of ECS systems.  For that purpose, ToyMaker provides
a component-based scripting interface, that it calls [sim objects](https://raynmetal.github.io/toymaker/classToyMaker_1_1SimObject.html)
and [aspects.](https://raynmetal.github.io/toymaker/classToyMaker_1_1SimObjectAspect.html)

An aspect is a user-defined script -- similar to a MonoBehaviors from Unity, or scripts
in Godot -- that are hooked into application loop events, and that provide full access
to the components of the scene node that owns it.  It also provides methods for querying
scene nodes relative to the one it's attached to.

A sim object is simply a special subclass of scene node that is capable of hosting one or
more aspects.  It is also the only scene node class eligible to be the root of a scene
described by a scene file or scene description.

### Rendering System

The [render system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2render__system.html)
is responsible for actually displaying the scenes ToyMaker manages.  It does so by
periodically reading scene data and turning it into images through a combination of
various techniques.

The render system is divided into render sets -- a set of target color and
depth-stencil buffers allocated for a single viewport -- and some sequence of render
stages.  Each render stage represents a combination of CPU and GPU work, resulting in
the addition of a specific effect to a target image, or the creation of a certain resource
that is intended to be used with the target image.

![Clip showing intermediate textures generated in the rendering pipeline]({{- "/assets/images/2025-10-ToyMakerDebugTextures.gif" | relative_url -}})

![A flowchart describing the rendering pipeline.]({{- "/assets/images/render_pipeline.png" | relative_url -}})

At present, each render stage is associated with a single OpenGL shader program, and
the only rendering pipeline supported by ToyMaker is a 3D Blinn-Phong deferred shading
pipeline with the following stages:

1. [ToyMaker::GeometryRenderStage](https://raynmetal.github.io/toymaker/classToyMaker_1_1GeometryRenderStage.html)
-- Converts input mesh-material pairs into albedo, specular, and position textures,
collectively called geometry textures.
2. [ToyMaker::LightingRenderStage](https://raynmetal.github.io/toymaker/classToyMaker_1_1LightingRenderStage.html)
-- Uses geometry textures and light data to produce a texture of the lit scene.
3. [ToyMaker::SkyboxRenderStage](https://raynmetal.github.io/toymaker/classToyMaker_1_1SkyboxRenderStage.html)
-- Adds a cubemap background to the lit scene.
4. [ToyMaker::TonemappingRenderStage](https://raynmetal.github.io/toymaker/classToyMaker_1_1TonemappingRenderStage.html)
-- Applies gamma correction and camera exposure post-processing effects to the lit scene.

### Physics & spatial query systems

The [physics system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2physics__system.html)
is responsible for simulating physics for appropriately tagged entities in the scene.  It
does so through an implementation of [XPBD, or Extended Position-Based Dynamics.](https://raynmetal.github.io/blog/technical/2026/07/24/xpbd-in-toymaker.html)

Each entity managed by the physics system possesses [physics state,](https://raynmetal.github.io/toymaker/structToyMaker_1_1PhysicsState.html)
and [object bounds.](https://raynmetal.github.io/toymaker/structToyMaker_1_1ObjectBounds.html)
Every fixed step, the physics system *predicts* where each object should end up next:

1. Using the sum of forces acting on the center of mass of the object, compute the change
to its velocity, and add it to the object’s previous velocity to find its new velocity.

2. Using the sum of torques acting on the object, compute the change to its angular
velocity, and add the change to its previous angular velocity to find its new angular
velocity.

3. Using the velocity of the object, compute and apply the change to the object’s
position in this timestep.

4. Using the angular velocity of the object, compute and apply the change to the object’s
orientation in this timestep.

Then, the physics system *corrects* all violations of applicable physical *constraints:*

1. Get the collision response data for a pair of intersecting objects.

2. Compute the generalized inverse mass for each object.

3. Use collision data to compute a Lagrange multiplier delta.

4. Use the Lagrange multiplier delta to compute a positional correction impulse.

5. Apply the correction impulse (in opposing directions) to both objects.

6. Derive the new velocities of each object.

... where a constraint is anything from a collision, to a joint between two objects, to
an implied spring connecting two objects.

This update is performed one or more times every fixed update, where the fixed update
itself takes place 10s of times each second per the application configuration.  Optionally,
the system can use [signals](https://raynmetal.github.io/toymaker/group__ToyMakerSignals.html)
to report collision events to scripts and other systems.  Collisions are resolved using
[GJK](https://www.youtube.com/watch?v=Qupqu1xe7Io), with [sweep and prune](https://leanrada.com/notes/sweep-and-prune/)
used to minimize the number of times *GJK* is required to run.

[ToyMaker's spatial query system](https://raynmetal.github.io/toymaker/group__ToyMakerSpatialQuerySystem.html)
provides an interface for answering most spatial queries; for example, what is the closest
object to this object?  Do these two objects intersect?  What objects exist within a given
region?  And so on.  It does so through a dynamic [Octree](https://raynmetal.github.io/toymaker/group__ToyMakerSpatialQuerySystem.html)
implementation.

### Inputs

[ToyMaker's input system](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2input__system.html)
translates raw input from the platform running the application into *actions,* which
are abstracted normalized inputs directly consumable by systems and scripts.  The
input configuration informs the application which raw inputs map to which actions.  It
tends too look something like:

```plaintext
   W    ______
 A|S|D        \___.--> Character : Move (2 Axes) --> (Move handlers)
<keyboard>     /
              /
             /
 (( L ))----/
<controller>
```

### Window context

For the time being, the [window context](https://raynmetal.github.io/toymaker/classToyMaker_1_1WindowContext.html)
is a simple wrapper over *SDL* that requests a window from the operating system,
initializes the *OpenGL* context, performs initialization for other libraries ToyMaker
depends on, and reports window level changes to scripts and systems managed by the
ToyMaker engine.

### Signals

This is a two-part lightweight implementation of the [observer pattern,](https://en.wikipedia.org/wiki/Observer_pattern)
enabling the decoupling of signal publishers from its consumers in the engine environment.
An object will either play the role of a [signal publisher,](https://raynmetal.github.io/toymaker/classToyMaker_1_1Signal.html)
broadcasting the fact that some specific event took place, or the role of a
[signal observer,](https://raynmetal.github.io/toymaker/classToyMaker_1_1SignalObserver.html)
which takes some action when such an event occurs.  Any single object often plays both
roles, advertising its signals to other objects, and listening for events from other
objects, and this capability is represented as the [signal tracker base class.](https://raynmetal.github.io/toymaker/classToyMaker_1_1SignalTracker.html)

As to why this is necessary: a script that is responsible for applying damage in a game,
for example, should *not* need to know that the damage should cause a sound to play,
or for some metric to be updated. It should only do what it is intended to do, which
is update health numbers, and leave further processing to systems subscribed to its
signal.

## References

- [ToyMaker Github repo](https://github.com/raynmetal/toymaker) -- Where the game engine
and its sources are hosted.

- [Game of Ur Github repo](https://github.com/raynmetal/game-of-ur) -- A game built on top
of the ToyMaker engine, playable on Windows and Linux.

- [Game and engine development timeline video](https://youtu.be/1ytOqm6NgKM) -- A
development timeline video for the ToyMaker engine and the Game of Ur.

- [Engine documentation](https://raynmetal.github.io/toymaker/index.html) -- The site
holding everything I've written about the ToyMaker engine.

- [ToyMaker Trello board](https://trello.com/b/ALP2KjNp/toymaker-engine) -- Trello board for engine development.  This is going to keep getting updates, albeit much more slowly than before.

- [Learn OpenGL](https://learnopengl.com/) -- One stop shop for practically everything you
need to know for starting graphics programming.

- [Euler angles and gimbal lock](https://www.youtube.com/watch?v=zc8b2Jo7mno) - A video
demonstrating why euler angles are insufficent for representing changing orientations.

- [Model View Projection - J Santell](https://jsantell.com/model-view-projection/) -- High
level overview of what the most commonly used matrices in 3D rendering actually do.

- [The Perspective and Orthographic Projection Matrix](https://www.scratchapixel.com/lessons/3d-basic-rendering/perspective-and-orthographic-projection-matrix/opengl-perspective-projection-matrix.html)
-- Talks about how the perspective projection matrix is actually derived.

- [Visualizing the 4d numbers Quaternions](https://www.youtube.com/watch?v=d4EgbgTm0Bg) --
Very cool video talking about what quaternions are, how their math works, and how they
can be used to specify rotations in 3D.

- [Linear Algebra - Khan Academy](https://www.khanacademy.org/math/linear-algebra) -- Best
linear algebra learning material I've found online.

- [Beginning Game Programming v3.0](https://lazyfoo.net/tutorials/SDL3/index.php) -- This
taught me SDL 3.

- [CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html) -- A
guide to perhaps the most ubiquitous build system for C++.

- [Makefile Tutorial](https://makefiletutorial.com/) -- Used to create recipes for the
creation of specific files that depend on some process being applied to other files.

- [Pro Git](https://git-scm.com/book/en/v2) -- Comprehensive manual for a very popular
versioning tool.

- [Learn C++](https://www.learncpp.com/) -- A series of lessons in the use of C++.

- [C++ Templates and Metaprogramming](https://medium.com/@AlexanderObregon/c-templates-and-metaprogramming-214a7b0db803)
-- Medium article talking about some common C++ template idioms.

- [Modern Template Metaprogramming](https://www.youtube.com/watch?v=Am2is2QCvxY) -- The
first of a series of video talks about the power of C++'s generics.

- [Physics introduction - Godot Engine 4.7 documentation in English](https://docs.godotengine.org/en/stable/tutorials/physics/physics_introduction.html#using-rigidbody2d)
-- An overview of physics data as represented in Godot, useful for learning how popular engines tackle this task.

- [Object/Object Intersections - realtimerendering.com](https://www.realtimerendering.com/intersections.html) -- An extensive collection of
links to intersection test techniques for different pairs of shapes.

- [Implementing GJK (2006) - Casey Muratori](https://www.youtube.com/watch?v=Qupqu1xe7Io) -- Video lecture giving a simple breakdown of how
GJK works, and how it can be implemented.

- [Computing Distance - Erin Catto](https://box2d.org/files/ErinCatto_GJK_GDC2010.pdf) --
Slides that are presumably connected with a talk given at GDC, whose video I could not
find.  Covers everything from GJK's rationale to its implementation in easy non-academic
language, with plenty of diagrams.

- [Game Physics: Contact Generation (EPA) - Allen Chou](https://allenchou.net/2013/12/game-physics-contact-generation-epa/)
-- Article describing the Expanding Polytope Algorithm, or EPA, for finding collision
contact points using the simplex found with GJK.

- [Detailed Rigid Body Simulation with Extended Position Based Dynamics - Matthias Müller, Miles Macklin, Nuttapong Chentanez,
Stefan Jeschke, Tae-Yong Kim](https://matthias-research.github.io/pages/publications/PBDBodies.pdf)
-- The paper on the realtime physics simulation approach, XPBD, talked about in this article.

- [Ten Minute Physics - Matthias Müller](https://matthias-research.github.io/pages/tenMinutePhysics/index.html)
-- Collection of educational physics programming videos, many of them closely connected
with the XPBD paper.

- [Impulse-Based Dynamic Simulation - Brian Mirtich](https://people.eecs.berkeley.edu/~jfc/mirtich/impulse.html)
-- A paper on impulse-based physics solvers as seen in most current physics engines.  The
appendix contains a very useful collection of derivations for various 3D physics formulae.

- [Introduction to Octrees - Eric Nevala](https://gamedev.net/tutorials/programming/general-and-gameplay-programming/introduction-to-octrees-r3529)
-- An introduction to spatial partitioning algorithms in general, with a guide to building
octrees in particular.

- [Advanced Octrees 3: non-static Octrees - David Geier](https://geidav.wordpress.com/2014/11/18/advanced-octrees-3-non-static-octrees/)
-- A very short article which talks about how to dynamically update the region encompassed
by an octree in response to changes in a scene.

- [Sort, sweep, and prune: Collision detection - leanrada](https://leanrada.com/notes/sweep-and-prune/) -- An explanation of Sweep-and-Prune, a popular broad-phase collision detection
algorithm.
