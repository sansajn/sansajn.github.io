---
layout: post
title: "Transformation composition in Bullet"
date: 2021-02-28 20:00:00 +0100
tags: [bullet, c++]
comments: true
author: Adam Hlavatovic
---

If you are familiar with *GLM* library then you definitely miss easy to use `transform()` or `rotate()` functions when working with Bullets transformations (`btTransform`).

I've found handy to implement them with Bullet types `btVector3` and `btScalar` and returning `btTransform` which can be further composed with ordinary `*` operator. There is the sample

```c++
// transformation composition sample
#include <iostream>
#include <bullet/LinearMath/btTransform.h>
#include <bullet/LinearMath/btQuaternion.h>

using std::cout;

btTransform translate(btVector3 const & v);
btTransform translate(btScalar x, btScalar y, btScalar z);

//! \param[in] angle angle in radians
btTransform rotate(btScalar angle, btVector3 const & axis);


int main(int argc, char * argv[])
{
	// glm way
	btTransform T = rotate(btRadians(45), btVector3{0, 0, 1}) * translate(1, 1, 0);

	btVector3 pt = T*btVector3{0, 0, 0};  // first translation, then rotation applyed

	cout << "pt=(" << pt.x() << ", " << pt.y() << ", " << pt.z() << ")\n";

	// bullet way
	btQuaternion q{btVector3{0, 0, 1}, btRadians(45)};
	btTransform U{q},
		V{btQuaternion{}, btVector3{1, 1, 0}};
	btVector3 pu = (U*V)*btVector3{0, 0, 0};

	cout << "pu=(" << pu.x() << ", " << pu.y() << ", " << pu.z() << ")\n";

	cout << "done!\n";
	return 0;
}

btTransform translate(btVector3 const & v)
{
	btTransform T;  // uninitialized by default
	T.setIdentity();
	T.setOrigin(v);
	return T;
}

btTransform translate(btScalar x, btScalar y, btScalar z)
{
	return translate(btVector3{x, y, z});
}

btTransform rotate(btScalar angle, btVector3 const & axis)
{
	return btTransform{btQuaternion{axis, angle}};
}
```
