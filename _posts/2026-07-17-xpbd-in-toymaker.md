---
layout: post
title: "Adding XPBD-based Physics to ToyMaker"
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
ways, periodically showing the results of those modifications to the user.

The physics engine is the system in the game that attempts to model _physical phenomena,_ i.e.,  forces should cause a
change in an object's velocity, and the object's velocity should cause a change in its position, etc.,.  Out of the bag
of data that represents the game, then, in order to represent a particular _"physical"_ object, we might have:

-  The object's _position_ (as a list of 3 floats) and _orientation_ (as a 4 float [quaternion](https://www.youtube.com/watch?v=zjMuIxRvygQ)).

-  The _kinematic properties_ of the object, i.e., its _positional_ and _angular velocities_, each as a list of 3 floats.

-  The _forces_ acting on the object, represented as the _sum force_ acting on its _center of mass,_ and the _sum torque_ around
_an axis passing through its center of mass._

-  The _physical properties_ of the object -- its _mass,_ _rotational inertia,_ _static_ and _dynamic friction coefficients,_ its
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

A part of what separates computer games from other kinds of software is that the _state of the game_ -- all represented
in tables much like the one above -- is _advanced several times each second,_ with or without input from the user.  Those
states are _displayed_ to the user 10s of times each second too, in the form of rapidly changing images, giving the illusion
of movement.

The physics engine performs these updates for the _physical state_ of the simulation by dividing time into a series of
discrete time steps; for a physics engine running at _60Hz,_ this would be a timestep of _~16 milliseconds,_ or _0.016
seconds._  At each step, the engine asks _"Given that this object is here right now, where should it be in the next 1/60th
of a second?",_ which it then answers using formulae derived from [Newtonian mechanics.](https://en.wikipedia.org/wiki/Newton%27s_laws_of_motion)

Here's what that looks like for some of the values from the table above, in a single time step.

| Name | Type   |   Value       |
|------|--------|---------------|
| vel  | `flt3` | 0, -1.16, 0   |
| pos  | `flt3` | 0, 0, 4.98144 |

Where:

- the velocity update:
    1.  `newVel = vel + frce * timestep / mass`
    2.  `newVel = (0, -1, 0) + (0, -10, 0) * 0.016 / 1.0`
    3.  `newVel = (0, -1.16, 0)`

- the position update:
    1.  `newPos = pos + newVel * timestep`
    2.  `newPos = (0, 5.0, 0) + (0, -1.16, 0) * 0.016`
    3.  `newPos = (0, 4.98144, 0)`

Picture such an update happening 60 times a second for this object _and every other object in the game,_ each
represented in much the same way, and you now have a rough idea of what a physics engine does.

## The physics update

### Physics state

We've already seen the table representing much of the data that we'll want to associate with an object in order to be able
to physically model that object.  We already have a representation for object shapes in the form of [ToyMaker's ObjectBounds
component,](https://raynmetal.github.io/toymaker/structToyMaker_1_1ObjectBounds.html) so our first task is to make a struct
for the object's _other_ physical properties.

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

### The integration function (also called "prediction step")

So now that we have a representation for our object's physics state, our next step is to write an _integration function_ to
get our objects moving.  The integration function is called once per timestep, and looks something like this:

1.  For each object:

    1.  Using the sum of forces acting on the center of mass of the object, compute the change to its velocity, and add
    it to the object's previous velocity to find its new velocity.

    2.  Using the sum of torques acting on the object, compute the change to its angular velocity, and add the change
    to its previous angular velocity to find its new angular velocity.

    3.  Using the velocity of the object, compute and apply the change to the object's position in this timestep.

    4.  Using the angular velocity of the object, compute and apply the change to the object's orientation in this timestep.

>  **Note**
>
>  Integration means something like _"accumulation,"_ for those not terribly comfortable with calculus terms.  In our
>  example from the previous section we used a formula to discover, given a force acting on an object's center of mass, the
>  _change to that object's velocity for a fixed quantity of time,_ which we then _added_ to the object's previous velocity
>  in order to obtain its new velocity.
>
>  We did a similar thing with the object's velocity as well, where we _added_ the _change in position due to its velocity for
>  a fixed quantity in time_ to the object's old position to obtain its new position.
>
>  The addition of a quantity representing some _change,_ to another quantity representing an _accumulation of such changes,_
>  is what we might call _"integration."_

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

1.  Visually, it's quite clear to us that the two balls are intersecting, and that they _shouldn't._  But try to imagine
what the computer sees.  Or better still, look:

    | Name  | Type   |   Value       |
    |-------|--------|---------------|
    | pos1  | `flt3` | -0.836, 0, 0  |
    | ori1  | `quat` | 1, 0, 0, 0    |
    | shp1  | `enum` | SPHERE        |
    | rdus1 | `flt`  | 1.0           |
    | pos2  | `flt3` | 0.836, 0, 0   |
    | ori2  | `quat` | 1, 0, 0, 0    |
    | shp2  | `enum` | SPHERE        |
    | rdus2 | `flt`  | 1.0           |

    The fact that the balls in the demo are intersecting is slightly more difficult to see in this form.  Even
    if we're able to apply our knowledge of geometry to answer questions about spheres, what about other shapes?

    Turns out that there's a very popular pair of algorithms that helps us answer intersection questions for _all_
    convex shapes: [GJK,](https://en.wikipedia.org/wiki/Gilbert%E2%80%93Johnson%E2%80%93Keerthi_distance_algorithm) to
    determine whether a pair of convex shapes is intersecting, and [EPA,](https://winter.dev/articles/epa-algorithm/)
    to find, for each object in the pair, its _point of contact,_ and the _distance_ and _direction_ the object would
    need to move in order to be separated from the other object.

    >  **Note**
    >
    >  Concave shapes are quite often represented as groups of convex shapes in realtime physics engines.  That means that
    >  this pair of algorithms can help us represent and use _any_ sensible 3D shape we can imagine in our physics engine.

    These two are certainly worth writeups of their own, but they've already been covered extensively by other authors, so
    I'll just leave links to helpful references for them at the bottom of this post.

2.  Assuming that we _can_ get our hands on information about the contact, how do we modify the motion of the objects such that
they believably comply with physical laws?

## XPBD

### What are constraint solvers?

Computer simulations that attempt to model physical phenomena use a class of algorithms called _constraint solvers._ The
basic idea here is that, while it is difficult to entirely prevent inaccuracies (or _errors_) in realtime simulations, what
we can more easily do is

1.  Discover the error when it has occurred:

    -  In our earlier example, an _intersection between two shapes_ occurred (which we know shouldn't happen, making it an
    error), and we were able to determine the _location,_ _direction,_ and _magnitude_ of the intersection.

    -  We might model the _extension of a spring_ as an error relative to its _rest length._

    -  We could also class the _deformation/change in volume_ of a deformable object as an error relative to its _original
    volume._

    -  Angular errors can be computed this way too.  An _overbent_ joint (as in in human-like ragdolls) is an error relative
    to the _angular limits_ of that joint.

2.  Then, either:

    1.  Compute the forces it would take to effect a correction to the objects involved in the error in as few steps as
    possible, then apply them to the objects, _resolving the error during our usual physics integration step._

    2.  _Apply corrections to the objects right away,_ and derive what forces would have caused the correction _after_ the
    correction has been applied (if required).

The former approach is the one that has been popular in the kind of realtime physics simulations we see in video games for
a long time.  While there's plenty of material talking about this approach, I haven't studied it and so can't say much.
From what I understand, [Brian Mirtich's Impulse-based Dynamic Simulation of Rigid Body Systems -- 1996](https://people.eecs.berkeley.edu/~jfc/mirtich/impulse.html)
is the place to start.  This category of solvers are called Impulse-based solvers.

[Detailed Rigid Body Simulation with Extended Position Based Dynamics - Matthias Müller, Miles Macklin, Nuttapong Chentanez,
Stefan Jeschke, Tae-Yong Kim](https://matthias-research.github.io/pages/publications/PBDBodies.pdf) takes the latter approach,
_immediately_ applying corrections to bodies in error relative to some constraint, and deriving correction forces post-hoc per
need.  This paper was the primary basis of my implementation, along with several videos from [Matthias' Ten Minute Physics
collection of videos.](https://matthias-research.github.io/pages/tenMinutePhysics/index.html)

### Collision constraints

#### Collision response data

Let's go back to our example scene.  We have two shapes -- in this case, spheres -- colliding.  The spheres begin to intersect one
another while moving towards each other.  We have a way to _detect_ and _quantify_ their overlap in [GJK](https://en.wikipedia.org/wiki/Gilbert%E2%80%93Johnson%E2%80%93Keerthi_distance_algorithm)
and [EPA.](https://winter.dev/articles/epa-algorithm/)

![A diagram of two overlapping spheres with labels for their contact information]({{- "/assets/images/diagram-sphere-collision.svg" | relative_url -}})

We call the data just generated the _collision response data_ corresponding to a pair of overlapping objects.  In English, this
is what the labele mean:

-  _**contact point:**_  For each object, a point on the surface of its parent shape that has the deepest penetration relative
to the other colliding shape.

-  _**contact normal:**_  For each object, the direction in which the parent shape would have to move in order to no longer be
overlapping with the other colliding shape.

-  _**penetration depth:**_  For both objects, the shortest distance either shape would need to move along the contact normal to
no longer be overlapping with the other.

As C++ structs taken from [ToyMaker::Contact:](https://raynmetal.github.io/toymaker/structToyMaker_1_1Collision.html)

```c++
/**
 * @brief Object representing contact information between a pair of convex shapes from the
 * perspective of _one_ of those shapes.
 *
 * All 3D points and vectors are given relative to the world.  Additional processing of
 * this data is necessary to convert them to local-space data.
 *
 */
struct Contact {
    /**
     * @brief The worldspace point on the surface of this shape that made contact with the other shape
     *
     */
    glm::vec3 mPoint;

    /**
     * @brief Worldspace normal pointing inward from the surface of this
     * shape such that moving in this direction by the penetration depth
     * will remove the overlap.
     */
    glm::vec3 mNormal;

    /**
     * @brief The length by which to move this object in the direction of
     * the contact normal to separate the colliding objects
     *
     */
    float mPenetrationDepth;
};

/**
 * @brief Data representing everything about a collision.
 *
 * This data includes:
 *
 * - Contact information relative to each object, referred to as A and
 * B, depending on the order of parameters passed to the collision evaluator
 *
 */
struct Collision {

    /**
     * @brief Contact information relative to the _first_ collision shape
     *
     */
    Contact mContactA;

    /**
     * @brief Contact information relative to the _second_ collision shape
     *
     */
    Contact mContactB;
};
```

#### Computing Gauss-Seidel update terms

Per [the paper,](matthias-research.github.io/pages/publications/PBDBodies.pdf) our collision constraint is a _positional_
constraint, meaning that the points participating in the constraint -- here, the contact points -- must be moved through
space in order to comply with the constraint again.  In XPBD, this occurs through a step it calls a _projected Gauss-Seidel
update._ 

For a general position constraint involving 2 participants, we compute a scalar value the paper calls the _Lagrange multiplier
update,_ whose formulae are rendered in c++ syntax below:

```c++
// before the first position solve:
float lambda = 0.f;

// ...

// for every other solve, for all active constraints
const float deltaLambda = (
    (
        -c - alpha / (timestep * timestep) * lambda
    ) / (
        w_1 + w_2 + alpha / (timestep * timestep)
    )
);
lambda += deltaLambda
```

The general method to determine the equation for the update for _any_ positional physical constraint, given a way to measure
error, is described briefly in [Matthias' video of the XPBD technique.](https://youtu.be/jrociOAYqxA?si=ECXFiLRQ1iT9jasx&t=547)
In the context of our collision correction:

-  We can conveniently ignore compliance related terms, labelled `alpha` in the paper, since our collision related constraints
will _always_ have a compliance of 0.

-  We treat the _penetration depth_ of the collision as the _error_ in the collision constraint. In the paper, this is the
`c` term in equation _(4)_.

-  For each object, the _negative_ of the _contact normal_ gives us the direction of the greatest _increase_ in error, which
I'll call the correction direction.  The paper labels this term `n` in equations _(2),_ _(3),_ _(10),_ etc.

-  We can compute the generalized inverse mass for each object as below.  In the paper these are terms `w_1` and `w_2` in
equations _(2)_ and _(3)._

    ```c++
    float computeGeneralizedInverseMassPositional(
        const ObjectBounds& object,
        const PhysicsState& physics,
        const glm::vec3& correctionPoint,
        const glm::vec3& correctionGradient
    ) {
        const glm::vec3 position { object.getPositionWorld() };
        const glm::vec3 correctionOffset { correctionPoint - position };
        const glm::vec3 correctionRotational { glm::cross(correctionOffset, correctionGradient) };
        const float generalizedInverseMass { physics.mMassInverse + (squareDistance(correctionRotational)?
            computeGeneralizedInverseMassRotational(
                object, physics, correctionRotational
            ) : 0.f)
        };
        return generalizedInverseMass;
    }

    float computeGeneralizedInverseMassRotational(
        const ObjectBounds& object,
        const PhysicsState& physics,
        const glm::vec3& correction
    ) {
        const glm::quat orientation { object.getOrientationWorld() };
        const glm::vec3 correctionLocal { glm::inverse(orientation) * correction };
        return glm::dot(correctionLocal, physics.mRotationalInertiaInverse * correctionLocal);
    }
    ```

    Here, `correctionPoint` refers to the point on the surface of the object participating in the constraint.  In our
    collision constraint, this would be the object's contact point with the other object.

    `correctionGradient` is a vector along `n` representing the offset from the constrained point's current position that
    it would need to move in order to increase error `c` by exactly _one_ unit.

    For our collision constraint, this is a vector of magnitude one, and therefore the same as direction vector `n`.

-  The _Lagrange multiplier delta_ -- `deltaLambda` in equations _(4)_ and _(5)_ of the paper, and implicitly in eqns.,
_(6)_ onwards -- is a quantity representing the _impulse_ that must be applied to the colliding objects to perform the
correction.

    ```c++
    const float lagrangeDeltaCollision {
        -mPenetrationDepth / (
            generalizedInverseA + generalizedInverseB
        )
    };
    ```

    >  **Note:**
    >
    >  In equations _(10)_ and onwards, the paper uses `lambda` when I _think_ what is meant is `deltaLambda`.  For
    >  collision constraints (and presumably other stiff constraints), `lambda` gives overly large values for contact
    >  forces, while `deltaLambda` produces more reasonable ones.
    >
    >  This becomes especially important when it comes to computing frictional forces and restitution corrections.

-  We take the _Lagrange multiplier delta_ scalar and _correction direction_ vector together to get the _positional
correction impulse._

    ```c++
    const glm::vec3 positionalImpulse { lagrangeDeltaCollision * mContactNormal };
    ```

#### Applying the positional correction

The positional impulse is applied _as-is_ to the center of mass of the object.  The rotational component of the
correction is computed by its _cross product_ with the relative offset of the contact point from the object's center
of mass. Overall, the update looks something like this:

```c++
ObjectBounds applyImpulseObject(
    ObjectBounds object,
    const PhysicsState& physics,
    const glm::vec3& impulsePositional,
    const glm::vec3& impulsePoint
) {
    // apply positional corrections
    const glm::vec3 position { object.getPositionWorld() };
    const glm::vec3 positionNew { position + impulsePositional * physics.mMassInverse };
    object.setPositionWorld(positionNew);

    // apply rotational corrections
    const glm::vec3 impulseRotation { glm::cross(
        impulsePoint - position, impulsePositional
    ) };
    object = applyImpulseObject(object, physics, impulseRotation);
    return object;
}

ObjectBounds applyImpulseObject(
    ObjectBounds object,
    const PhysicsState& physics,
    const glm::vec3& impulseRotational
) {
    const glm::quat orientation { object.getOrientationWorld() };
    const glm::vec3 impulseRotationalLocal { glm::inverse(orientation) * impulseRotational };
    const glm::vec3 deltaOrientation { orientation * (physics.mRotationalInertiaInverse * impulseRotationalLocal) };
    const glm::quat orientationNew { glm::normalize(
        orientation + .5f * glm::quat(0.f, deltaOrientation.x, deltaOrientation.y, deltaOrientation.z) * orientation
    ) };
    object.setOrientationWorld(orientationNew);
    return object;
}
```

>  **Note**
>
>  While not covered here, separating the rotational part of the correction out from the positional part
>  allows us to reuse the rotational correction function for angular constraints.

#### Modifications to the physics update

Now that we know how to compute values for terms participating in the Gauss-Seidel update, we can talk about the changes
we need to make in our physics update.

The first change relates to our timestep.  Traditionally, constraint solvers would iterate through all active constraints
several times until the total error across constraints had been minimized (or until the solver ran out of iterations).  The
paper advises against this.  Instead, it recommends subdividing the physics timestep into smaller _substeps._  According to
the paper, this improves the accuracy of the simulation and its time to convergence.

The other change will of course be the addition of the position constraint solve itself.

Accordingly, we get:

1.  For each `substep`, where `substep = timestep / numSubsteps`:

    1.  For each object:

        1.  Integrate forces.

        2.  Apply collision (position) constraints.

Where the collision constraint algorithm is:

1.  Get the collision response data for a pair of intersecting objects.

2.  Compute the generalized inverse mass for each object.

3.  Use collision data to compute a _Lagrange multiplier delta._

4.  Use the Lagrange multiplier delta to compute a _positional correction impulse._

5.  Apply the correction impulse (in opposing directions) to both objects.

Overall, this gives us an update that looks like this:

```c++
void PhysicsSystem::onSimulationStep(uint32_t timestepMillis) {
    // subdivide our physics timestep (and convert to seconds)
    const float substepInterval { (static_cast<float>(timestepMillis) / static_cast<float>(mSubsteps)) / static_cast<float>(1e3) };

    // collect all potential collisions
    collectPotentialCollisions(substepInterval);

    // refresh lagrange multipliers for ongoing collisions
    for(auto& constraint: mCollisionConstraints) {
        collisionConstraint.second.resetLagrange();
    }

    for(auto substep { 0 }; substep < mSubsteps; ++substep) {
        integrateForces(substepInterval);

        // this updates the collision response data stored along with
        // each collision constraint
        detectCollisions();

        applyPositionConstraints(substepInterval);
    }
}

void PhysicsSystem::applyPositionConstraints(float substepInterval) {
    // apply non-collision constraints (not shown here)

    // ...
    // ...
    // ...

    // solve collision constraints
    for(auto& collisionPairConstraint: mCollisionConstraints) {
        const auto& collisionEntities { collisionPairConstraint.first };
        auto& collisionConstraint { collisionPairConstraint.second };

        const auto entityA { collisionPairConstraint.first.first };
        const auto entityB { collisionPairConstraint.first.second };

        auto objectA { getComponent<ObjectBounds>(entityA) };
        auto objectB { getComponent<ObjectBounds>(entityB) };
        auto physicsA { getComponent<PhysicsState>(entityA) };
        auto physicsB { getComponent<PhysicsState>(entityB) };

        collisionConstraint.applyConstraintPosition(objectA, physicsA, objectB, physicsB);
    }
}

void CollisionConstraint::applyConstraintPosition(
    ObjectBounds& objectA, PhysicsState& physicsA,
    ObjectBounds& objectB, PhysicsState& physicsB,
) {
    const glm::vec3 positionA { objectA.getPositionWorld() };
    const glm::vec3 positionB { objectB.getPositionWorld() };
    const glm::quat orientationA { objectA.getOrientationWorld() };
    const glm::quat orientationB { objectB.getOrientationWorld() };

    // compute generalized inverse masses for A and B -- these will
    // be recomputed each substep, so there's no point in storing them
    const float generalizedInverseA { computeGeneralizedInverseMassPositional(
        objectA,
        physicsA,
        mLastPointContactA,
        mContactNormal
    ) };
    const float generalizedInverseB { computeGeneralizedInverseMassPositional(
        objectB,
        physicsB,
        mLastPointContactB,
        mContactNormal
    ) };

    // compute correction value
    const float alphaDerivative2 {
        getCompliance() / (substepSeconds * substepSeconds)
    };
    const float lagrangeCollision { getLagrange().at(0) };
    const float lagrangeDeltaCollision {
        -(
            mPenetrationDepth + alphaDerivative2 * lagrangeCollision
        ) / (
            generalizedInverseA + generalizedInverseB + alphaDerivative2
        )
    };
    applyLagrangeDelta(lagrangeDeltaCollision, 0);
    const glm::vec3 positionalImpulse {
        lagrangeDeltaCollision * mContactNormal
    };

    // apply corrections
    objectA = applyImpulseObject(
        objectA,
        physicsA,
        positionalImpulse,
        mLastPointContactA
    );
    objectB = applyImpulseObject(
        objectB,
        physicsB,
        -positionalImpulse,
        mLastPointContactB
    );
}
```

>  **Note**
>
>  The [actual implementation](https://raynmetal.github.io/toymaker/classToyMaker_1_1PhysicsSystem.html#ae781b2269ab42929939430c5878b4692)
>  contains a lot of unrelated architectural cruft and tech debt, so I've omitted that here.

Which finally allows our spheres to actually _collide_ with each other!

![A clip of two balls moving towards each other at fixed velocities, then bouncing off one another]({{- "/assets/images/collision-1.gif" | relative_url -}})

