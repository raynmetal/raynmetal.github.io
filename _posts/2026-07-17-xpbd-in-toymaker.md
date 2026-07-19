---
layout: post
title: "Adding XPBD Physics to ToyMaker"
date: 2026-7-17 14:00:00 +0530
categories project personal
tags: [C++, SDL, OpenGL, 3D, Game Engine, Physics Engine, Physics, XPBD, Extended Position Based Dynamics, ToyMaker]
image: /assets/images/slope-friction-1.gif
image_alt: "Moving block slowing down and coming to a halt sliding down a gentle incline."
excerpt_separator: <!--end excerpt-->
---

I decided to add physics to my game engine, now named ["ToyMaker"](https://github.com/raynmetal/toymaker).  In the interest
of learning, I decided to implement it from scratch.  While there's still a lot I'd like to improve, I now have a semi-complete,
realtime, single-threaded CPU rigid body solver based on [Extended Position Based Dynamics,](https://matthias-research.github.io/pages/publications/PBDBodies.pdf)
or XPBD for short.

<!--end excerpt-->

## What is a physics engine?

So a game -- specifically a modern realtime 3D computer game -- is, at its heart, a piece of software like any other.  It
holds a bag of data relating to the purpose of the game, and it allows a user to modify that data in strictly controlled
ways, showing the results of those modifications to the user periodically.

The physics engine is the system in the game that attempts to model physical phenomena, Eg., Forces should cause an object's
velocity to change, and the object's velocity should cause its position to change.  Out of this bag of data therefore, to
represent a particular "physical" object in the game at a particular instant in time, we might have:

- The location (as a list of 3 floats) and orientation (as a 4 float [quaternion](https://www.youtube.com/watch?v=zjMuIxRvygQ))
of the object in 3D space.

- The kinematic properties of the object, i.e., its positional and angular velocities, each as a list of 3 floats.

- The forces acting on the object, represented as the sum force acting on its center of mass, and the sum torque around
an axis passing through its center of mass.

- The physical properties of the object -- its mass, rotational inertia, static and dynamic friction coefficients, shape,
and so on.

We might choose to represent this data in a kind of table, like so:

| Name | Type |   Value    |
|------|------|------------|
| pos  | flt3 | 0, 0, 5    |
| ori  | quat | 1, 0, 0, 0 |
| vel  | flt3 | 0, -1, 0   |
| aVel | flt3 | 0, 0, 0    |
| frce | flt3 | 0, -10, 0  |
| trq  | flt3 | 0, 0, 0    |
| mass | flt  | 1          |
| sFrc | flt  | 0.3        |
| dFrc | flt  | 0.2        |
| shp  | enum | SPHERE     |
| rdus | flt  | 1.0        |

... and more, which is indeed close to how our computer program _"sees"_ this object.

A part of what separates computer games from other kinds of software is that the state of the game, all represented
in tables much like the one above, is advanced several times each second, with or without input from the user.  Those
results are displayed to the user 10s of times each second too, in the form of rapidly switching images, giving the
illusion of movement.

The physics engine performs these updates for the physical state of the simulation, by dividing time into a series of
discrete time steps; for a physics engine running at 60Hz, this would be a timestep of ~16 milliseconds, or 0.016
seconds.  At each step, the engine asks "Given that this object is here right now, where should it be in the next 1/60th
of a second?", which it then answers using formulae from [Newtonian mechanics.](https://en.wikipedia.org/wiki/Newton%27s_laws_of_motion)

Here's what that looks like for the values from the table above, in a single time step.

| Name | Type |   Value       |
|------|------|---------------|
| vel  | flt3 | 0, -1.16, 0   |
| pos  | flt3 | 0, 0, 4.98144 |

Where:

- velocity:
    1. `newVel = vel + frce * timestep / mass`
    2. `newVel = (0, -1, 0) + (0, -10, 0) * 0.016 / 1.0`
    3. `newVel = (0, -1.16, 0)`

- position:
    1. `newPos = pos + newVel * timestep`
    2. `newPos = (0, 5.0, 0) + (0, -1.16, 0) * 0.016`
    3. `newPos = (0, 4.98144, 0)`

Now picture such an update happening 60 times a second for this object _and every other object in the game,_ each
represented in much the same way, and you now have a rough idea of what a physics engine does.

Also, while this is a start, things get even hairier when you try to model _interactions between multiple objects_ as
opposed to each object in isolation -- but we can get to that later.

## The first draft of the solver

We've already seen the table representing much of the data that we'll want to associate with an object in order to be able
to physically model that object.  I already had a representation for object shapes in the form of [ToyMaker's ObjectBounds
component,](https://raynmetal.github.io/toymaker/structToyMaker_1_1ObjectBounds.html) so my first task came was to make
a struct for the object's other physical properties.

Here's what that struct looks like, with some omissions for simplicity:

```c++
struct PhysicsState {
    /**
     * @brief The sum of all the forces acting on this object's center of mass,
     * causing it to move through space.
     *
     * Expressed as a vector in the world frame.
     *
     */
    glm::vec3 mForce { 0.f };

    /**
     * @brief Proportional to sum of all forces acting perpendicular to the vector going
     * from the point at which the forces are being applied to this object' center of
     * mass, causing it to rotate about its axis.
     *
     * Expressed as a vector in the world frame.
     *
     */
    glm::vec3 mTorque { 0.f };

    /**
     * @brief The velocity of this object.
     *
     * Expressed as a vector in the world frame.
     *
     */
    glm::vec3 mVelocity { 0.f };

    /**
     * @brief The angular velocity of this object.
     *
     * Expressed as a vector in the world frame.
     *
     */
    glm::vec3 mAngularVelocity { 0.f };

    /**
     * @brief The inverse of this object's resistance to rotational change.
     *
     * Expressed in terms of the object's local frame, and should be derived from
     * the object's mass and shape.
     *
     */
    glm::vec3 mRotationalInertiaInverse { 1.f };

    /**
     * @brief The inverse of this object's mass.
     *
     */
    float mMassInverse { 1.f };

    /**
     * @brief The friction coefficient of the force that prevents relative motion between the
     * surface of two objects when they are stationary.
     *
     */
    float mCoefficientFrictionStatic { 0.f };

    /**
     * @brief The friction coefficient of the force that hinders motion between the surface of
     * two objects when they are moving relative to each other.
     *
     */
    float mCoefficientFrictionDynamic { 0.f };
};
```

You can find a more complete description of the struct [here.](https://raynmetal.github.io/toymaker/structToyMaker_1_1PhysicsState.html#a60b087dacb5a196a2990a88b4710e21e)

So now that we have a representation for our object's physics state, the next step is to write an integration function to
get our objects moving.

> **Note**
>
> Integration here means something like "accumulate," for those not terribly comfortable with calculus.  In our example
> from the previous section we use a formula to discover, given a force acting on an object's center of mass, the
> _change to that object's velocity for a fixed quantity of time,_ which we then _add_ to the object's previous velocity
> in order to obtain its new velocity.
>
> We do a similar thing with the object's velocity as well, where we _add_ the _change in position due to its velocity for
> a fixed quantity in time_ to obtain the object's new position.
>
> The addition, or accumulation, of a quantity representing some change in another quantity representing an accumulation
> of changes is what we might call "integration."

Taken from [ToyMaker::PhysicsSystem::integrateForces(),](https://raynmetal.github.io/toymaker/classToyMaker_1_1PhysicsSystem.html#a5288427ffc70e8ed4d1799fb5a931a3f) that looks something like this:

```c++
auto physics = getComponent<PhysicsState>(entity);
auto bounds = getComponent<ObjectBounds>(entity);

// update positional and rotational velocities based on external forces
physics.mVelocity += timestepSeconds * physics.mForce * physics.mMassInverse;
const glm::quat orientation { bounds.getOrientationWorld() };
const glm::quat toLocal { glm::inverse(bounds.getOrientationWorld()) };
const glm::vec3 rotationalInertiaLocal { physics.getRotationalInertia() };
const glm::vec3 angularVelocityLocal { toLocal * physics.mAngularVelocity };
const glm::vec3 torqueLocal { toLocal * physics.mTorque };
const glm::vec3 deltaAngularVLocal { timestepSeconds
    * physics.mRotationalInertiaInverse * (
        torqueLocal - glm::cross(
            angularVelocityLocal,
            rotationalInertiaLocal * angularVelocityLocal
        )
) };
physics.mAngularVelocity += orientation * deltaAngularVLocal;

// update position and orientation based on velocities
const float deltaAngle {
    timestepSeconds * glm::length(physics.mAngularVelocity)
};
const glm::vec3 axisAngularVelocity { glm::normalize(physics.mAngularVelocity) };
const glm::quat orientationUpdate { glm::angleAxis(deltaAngle, axisAngularVelocity) };
const auto newOrientation { glm::normalize(orientationUpdate * physicsState.mBounds.getOrientationWorld()) };
bounds.setOrientationWorld(newOrientation);
bounds.setPositionWorld(
    physicsState.mBounds.getPositionWorld() + timestepSeconds * physics.mVelocity
);

updateComponent(entity, physics);
updateComponent(entity, bounds);
```

Which is practically identical what we describe in the previous section, but expanded in C++ syntax.  This is what we
get as a result:

![A clip of a text on an implied board, spinning around its center in response to forces generated by user clicks]({{- "/assets/images/forces-0.gif" | relative_url -}})

Here we see pointer clicks applying a force to a text object fixed at its center, causing it to spin.  The applied forces
cause a change to the object's angular velocity, which in turn causes the object's orientation to change around 30 times
each second.

Here's a clip of two balls colliding:

![A clip of two balls moving towards each other at fixed velocities, then intersecting instead of bouncing]({{- "/assets/images/collision-0.gif -}})

... well not really.  There's a couple of problems here:

1) Visually, it's quite clear to us that the two balls are intersecting, and that they _shouldn't._  But try to imagine
what the computer sees.  Or better still, look:

    | Name- | Type |   Value       |
    |-------|------|---------------|
    | pos1  | flt3 | -0.836, 0, 0  |
    | ori1  | quat | 1, 0, 0, 0    |
    | shp1  | enum | SPHERE        |
    | rdus1 | flt  | 1.0           |
    | pos2  | flt3 | 0.836, 0, 0   |
    | ori2  | quat | 1, 0, 0, 0    |
    | shp2  | enum | SPHERE        |
    | rdus2 | flt  | 1.0           |

    Can you tell that the spheres are intersecting now?  Perhaps you can, if you remember a little geometry.  But I
    bet its not nearly so intuitive now.

2. Let's say we can solve (1)

