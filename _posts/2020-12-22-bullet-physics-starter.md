---
layout: post
title: "bullet physics starter"
date: 2020-12-22 16:00:00 +0100
categories: physics c++ bullet
comments: true
author: Adam Hlavatovic
---

In the following text we present [*Bullet*][Bullet] physics starter with an object falling towards ground under the gravity force sample. 

### content

- [setup](#setup)
- [build & run](#build-&-run)
- [sample description](#sample-description)


## setup

First we need to install *Bullet* library, under *Ubuntu 20.04 LTS* it can be found in the repository as `libbullet-dev` package. We will also need `g++`, `scons` and `pkg-config` packages for easy building workflow.

> use `sudo apt install libbullet-dev g++ scons pkg-config` command to install all required dependencies

Then clone [bullet-physics-starter](https://github.com/sansajn/bullet-physics-starter) repository with

```bash
git clone https://github.com/sansajn/bullet-physics-starter.git
```

command.


## build & run

Building is super easy, just run

```bash
scons
```

command from sample directory which creates `hello` executable, then run `./hello` to run the simulation. 

There are two objects in the simulation falling body and ground. Simulation itself is divided into multiple short burst of times called steps. In every step simulation will print position for both objects (ground is not moving so its position is `0,-56,0` during whole simulation).

You should see, something like this

```console
$ ./hello
world pos object 1 = 2, 10, 0
world pos object 0 = 0, -56, 0
world pos object 1 = 2, 9.99722, 0
world pos object 0 = 0, -56, 0
...
```

> **tip**: see [scons-starter](https://github.com/sansajn/scons-starter) to get familiar with *SCons* building tool


## sample description

Heart of the physics simulation in *Bullet* is world, world is a place where objects (bodies) can interact with each other and world itself through forces. *Bullet* workflow is simple, create world, fill it with some bodies, setup forces and run simulation for some amount of time. 

Our sample [hello][Sample] works the same way, it creates world represented by [`btDiscreteDynamicsWorld`][World] object type this way

```c++
btDiscreteDynamicsWorld world{&dispatcher, &pair_cache, &solver, &config};
```

> do not bother with `dispatcher`, `pair_cache`, `solver` and `config` variables for now

Then sample creates two bodies represented with [`btRigidBody`][Body] object types. One body for falling object and one for ground. In *Bullet* each body must have shape so bodies can interact with each other. In the sample ground is represented as a square [`btBoxShape`][Box] of side equals to 100 units and on `(0, -56, 0)` position this way

```c++
btBoxShape ground_shape{btVector3{50, 50, 50}};
btDefaultMotionState ground_motion{translate(btVector3{0, -56, 0})};
btRigidBody ground_body{0, &ground_motion, &ground_shape};
```

Ground body `ground_body` is then added into the world with `addRigidBody()` member function this way

```c++
world.addRigidBody(&ground_body);
```

Then sample creates falling object body on `(2, 10, 0)` position with a unit sphere shape [`btSphereShape`][Sphere] this way

```c++
btSphereShape sphere_shape{1};
btDefaultMotionState sphere_motion{translate(btVector3{2, 10, 0})};

btScalar mass = 1;
btVector3 local_inertia = {0, 0, 0};
sphere_shape.calculateLocalInertia(mass, local_inertia);

btRigidBody sphere_body{mass, &sphere_motion, &sphere_shape, local_inertia};
```

this time `sphere_body` has also mass so its position will change during simulation based on applied forces (gravity in our case). One note there, body's motion state is stored in [`btDefaultMotionState`][MotionState] object type (see `sphere_motion` variable) rather than in [`btRigidBody`][Body] objects.

> initial body position is `(2, 10, 0)` with a radius equal to 1 and so it will hit the ground on `(2, -6, 0)` position with a center on `(2, -5, 0)`

Finally, at the end of the sample 150 step simulation is running, where each step represent 1/60s this way

```c++
for (int i = 0; i < 150; ++i) {
   world.stepSimulation(1.f / 60, 10);

   for (int j = world.getNumCollisionObjects() - 1; j >= 0; --j) {
      btTransform trans = get_object_transform(world.getCollisionObjectArray()[j]);
      cout << "world pos object " << j << " = " 
         << trans.getOrigin().getX() << ", " 
         << trans.getOrigin().getY() << ", " 
         << trans.getOrigin().getZ() << "\n";
   }
}
```

From the sample output we can see that the falling body ends on `(2.00006, -5.00001, 9.15244e-05)` position as expected. Pretty cool, what do you think?

> **advanced tip**: watch out other bullet samples from [`bullet3/examples`](https://github.com/bulletphysics/bullet3/tree/master/examples) directory


[Bullet]: https://bulletphysics.org
[World]: https://pybullet.org/Bullet/BulletFull/classbtDiscreteDynamicsWorld.html
[Body]: https://pybullet.org/Bullet/BulletFull/classbtRigidBody.html
[Box]: https://pybullet.org/Bullet/BulletFull/classbtBoxShape.html
[Sphere]: https://pybullet.org/Bullet/BulletFull/classbtSphereShape.html
[MotionState]: https://pybullet.org/Bullet/BulletFull/structbtDefaultMotionState.html
[Sample]: https://github.com/sansajn/bullet-physics-starter/blob/master/hello.cpp
[Book]: https://www.packtpub.com/game-development/learning-game-physics-bullet-physics-and-opengl
