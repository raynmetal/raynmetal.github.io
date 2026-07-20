---
layout: post
title: "Adding XPBD Physics to ToyMaker"
date: 2026-7-17 14:00:00 +0530
categories: blog technical
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

The physics engine is the system in the game that attempts to model physical phenomena.  Forces should cause an object's
velocity to change, and the object's velocity should cause its position to change.  Out of this bag of data therefore, to
represent a particular _"physical"_ object in the game at a particular instant in time, we might have:

- The object's _position_ (as a list of 3 floats) and _orientation_ (as a 4 float [quaternion](https://www.youtube.com/watch?v=zjMuIxRvygQ)).

- The _kinematic properties_ of the object, i.e., its _positional_ and _angular velocities_, each as a list of 3 floats.

- The _forces_ acting on the object, represented as the _sum force_ acting on its _center of mass,_ and the _sum torque_ around
_an axis passing through its center of mass._

- The _physical properties_ of the object -- its _mass,_ _rotational inertia,_ _static_ and _dynamic friction coefficients,_ its
_shape,_ and so on.

We might choose to represent this data in a kind of table, like so:

| Name | Type   |   Value    |
|------|--------|------------|
| pos  | `flt3` | 0, 0, 5    |
| ori  | `quat` | 1, 0, 0, 0 |
| vel  | `flt3` | 0, -1, 0   |
| aVel | `flt3` | 0, 0, 0    |
| frce | `flt3` | 0, -10, 0  |
| trq  | `flt3` | 0, 0, 0    |
| mass | `flt`  | 1          |
| sFrc | `flt`  | 0.3        |
| dFrc | `flt`  | 0.2        |
| shp  | `enum` | SPHERE     |
| rdus | `flt`  | 1.0        |

... and more, which is indeed close to how our computer program _"sees"_ this object.

A part of what separates computer games from other kinds of software is that the state of the game, all represented
in tables much like the one above, is advanced several times each second, with or without input from the user.  Those
results are displayed to the user 10s of times each second too, in the form of rapidly switching images, giving the
illusion of movement.

The physics engine performs these updates for the physical state of the simulation, by dividing time into a series of
discrete time steps; for a physics engine running at 60Hz, this would be a timestep of ~16 milliseconds, or 0.016
seconds.  At each step, the engine asks "Given that this object is here right now, where should it be in the next 1/60th
of a second?", which it then answers using formulae derived from [Newtonian mechanics.](https://en.wikipedia.org/wiki/Newton%27s_laws_of_motion)

Here's what that looks like for the values from the table above, in a single time step.

| Name | Type   |   Value       |
|------|--------|---------------|
| vel  | `flt3` | 0, -1.16, 0   |
| pos  | `flt3` | 0, 0, 4.98144 |

Where:

- velocity update:
    1. `newVel = vel + frce * timestep / mass`
    2. `newVel = (0, -1, 0) + (0, -10, 0) * 0.016 / 1.0`
    3. `newVel = (0, -1.16, 0)`

- position update:
    1. `newPos = pos + newVel * timestep`
    2. `newPos = (0, 5.0, 0) + (0, -1.16, 0) * 0.016`
    3. `newPos = (0, 4.98144, 0)`

Now picture such an update happening 60 times a second for this object _and every other object in the game,_ each
represented in much the same way, and you now have a rough idea of what a physics engine does.

Also, while this is a start, things get even hairier when you try to model _interactions between two or more objects_
as opposed to each object in isolation -- but we'll get to that later.

## The first draft of the solver

We've already seen the table representing much of the data that we'll want to associate with an object in order to be able
to physically model that object.  I already had a representation for object shapes in the form of [ToyMaker's ObjectBounds
component,](https://raynmetal.github.io/toymaker/structToyMaker_1_1ObjectBounds.html) so my first task came was to make
a struct for the object's _other_ physical properties.

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

So now that we have a representation for our object's physics state, the next step is to write an _integration function_ to
get our objects moving.  The integration function is called once per timestep, and looks something like this:

1.  For each object:

    1.  Using the sum of forces acting on the center of mass of the object, compute the change to its velocity, and add
    it to the object's previous velocity to find its new velocity.

    2.  Using the sum of torques acting around the object, compute the change to its angular velocity, and add the change
    to its previous angular velocity to find its new angular velocity.

    3.  Using the velocity of the object, compute and apply the change to the object's position in this timestep.

    4.  Using the angular velocity of the object, compute and apply the change to the object's orientation in this timestep.

> **Note**
>
> Integration means something like "accumulate," for those not terribly comfortable with calculus.  In our example
> from the previous section we use a formula to discover, given a force acting on an object's center of mass, the
> _change to that object's velocity for a fixed quantity of time,_ which we then _add_ to the object's previous velocity
> in order to obtain its new one.
>
> We do a similar thing with the object's velocity as well, where we _add_ the _change in position due to its velocity for
> a fixed quantity in time_ to the object's old position to obtain its new position.
>
> The addition of a quantity representing some change, to another quantity representing an accumulation of such changes,
> is what we might call "integration."

In [ToyMaker::PhysicsSystem::integrateForces(),](https://raynmetal.github.io/toymaker/classToyMaker_1_1PhysicsSystem.html#a5288427ffc70e8ed4d1799fb5a931a3f)
that looks something like this:

```c++
for(auto entity: mEntities) {
    auto physics { getComponent<PhysicsState>(entity) };
    auto bounds { getComponent<ObjectBounds>(entity) };

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
}
```

Which is practically identical what we describe in the previous section, but expanded in C++ syntax.  This is what we
get as a result:

![A clip of a text on an implied board, spinning around its center in response to forces generated by user clicks]({{- "/assets/images/forces-0.gif" | relative_url -}})

Here we see pointer clicks applying a force to a text object fixed at its center, causing it to spin.  The applied forces
cause a change to the object's angular velocity, which in turn causes the object's orientation to change around 30 times
each second.

Here's a clip of two balls colliding:

![A clip of two balls moving towards each other at fixed velocities, then intersecting instead of bouncing]({{- "/assets/images/collision-0.gif" | relative_url -}})

... well not really.  There are a couple of problems here:

1. Visually, it's quite clear to us that the two balls are intersecting, and that they _shouldn't._  But try to imagine
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

    The fact that the balls in the demo are intersecting is slightly more difficult to see, in this form.  Even
    if we're able to apply our knowledge of geometry to answer questions about spheres, what about other shapes?

    Turns out that there's a very popular pair of algorithms that helps us answer intersection questions for _all_
    convex shapes: [GJK,](https://en.wikipedia.org/wiki/Gilbert%E2%80%93Johnson%E2%80%93Keerthi_distance_algorithm) to
    determine whether a pair of convex shapes are intersecting, and [EPA,](https://winter.dev/articles/epa-algorithm/)
    to find, for each object, its point of contact, and the distance and direction the object would need to move in order
    to be separated from the other object.

    These two are certainly worth writeups of their own, but they've been covered extensively, so I'll just leave
    links to helpful references at the bottom of this post.

2. Assuming that we _can_ get our hands on information about the contact, how do we modify the motion of the objects such that
they believably comply with physical laws?

## Enter XPBD

Computer simulations that attempt to model physical phenomena use a class of algorithms called _constraint solvers._ The
basic idea here is that, while it is difficult to entirely prevent inaccuracies (or _errors_) in realtime simulations, what
we can more easily do is

1. Discover the error when it has occurred:

    - In our earlier example, an _intersection between two shapes_ occurred (which we know shouldn't happen, making it an
    error), and we were able to determine the _location,_ _direction,_ and _magnitude_ of the intersection.

    - We might model the _extension of a spring_ as an error relative to its _rest length._

    - We could also class the _deformation/change in volume_ of a deformable object as an error relative to its _original
    volume._

    - Angular errors can be computed too.  An _overbent joint_ (as in human-like ragdolls) is an error relative to the angular
    limits of that joint.

2. Then, either:

    1. Compute the forces it would take to effect a correction to the objects implicated in the error in as few steps as
    possible, and to apply them to the objects.

    2. Apply corrections to the objects right away, and derive what forces would have caused the correction _after_ the
    correction has been applied (if required).

The former approach is the one that has been popular in the kind of realtime physics simulations we see in video games for
a long time.  While there's plenty of material talking about this approach, I haven't studied it and so can't say much with
certainty.  From what I understand, [Brian Mirtich's Impulse-based Dynamic Simulation of Rigid Body Systems -- 1996](https://people.eecs.berkeley.edu/~jfc/mirtich/impulse.html)
is the place to start.

[Detailed Rigid Body Simulation with Extended Position Based Dynamics - Matthias Müller, Miles Macklin, Nuttapong Chentanez,
Stefan Jeschke, Tae-Yong Kim](https://matthias-research.github.io/pages/publications/PBDBodies.pdf) takes the latter approach,
applying corrections to bodies in error relative to some constraint, and deriving correction forces post-hoc as per need.  This
paper was the primary basis of my implementation, along with several videos from [Matthias' Ten Minute Physics collection of
videos.](https://matthias-research.github.io/pages/tenMinutePhysics/index.html)


