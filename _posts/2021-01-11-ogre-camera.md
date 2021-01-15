---
layout: post
title: "OGRE, using camera"
date: 2021-01-13 16:31:00 +0100
categories: ogre guide
comments: true
author: Adam Hlavatovic
---

This post is the second post in my miniseries to OGRE graphics library and today we learn how to work with camera. Full source code listing for today camera sample can be found in [ogre-guide](https://github.com/sansajn/ogre-guide) repository.

> previous posts in OGRE miniseries: [OGRE starter guide]({% post_url 2020-12-01-ogre-starter %})

### Content

- [Implementation](#implementation)
- [Building](#building)


## Implementation

OGRE implements camera controler as [`CameraMan`][CameraMan] in `OgreBites` namespace.

Let's start with our starter sample from previous post, we first define `go()` member function and `CameraMan` instance member variable. `CameraMan` requires handling keyboard, mouse input and `frameRendered` event from [`InputListener`][InputListener]. Our `ogre_app` implementation can looks like this

```c++
class ogre_app
	: public ApplicationContext, public InputListener
{
public:
	void go();

private:
	void setup() override;

	// user input
	bool keyPressed(KeyboardEvent const & evt) override;
	bool keyReleased(KeyboardEvent const & evt) override;
	bool mouseMoved(MouseMotionEvent const & evt) override;
	bool mousePressed(MouseButtonEvent const & evt) override;
	bool mouseReleased(MouseButtonEvent const & evt) override;
	void frameRendered(Ogre::FrameEvent const & evt) override;

	unique_ptr<CameraMan> _cameraman;
};
```

With `go()` member function `main()` becomes

```c++
int main(int argc, char * argv[]) {
	ogre_app app;
	app.go();
	cout << "done\n";
	return 0;
}
```

and `go()` member function implementation looks

```c++
void ogre_app::go() {
	initApp();

	if (getRoot()->getRenderSystem())
		getRoot()->startRendering();

	closeApp();
}
```

User input handling functions (from `InputListener`) just forward event object to the corresponding `CameraMan` member function this way

```c++
bool ogre_app::keyPressed(KeyboardEvent const & evt) {
	return _cameraman->keyPressed(evt);
}
```

> **note**: check out [`main.cpp`]() to see `keyReleased()`, `mouseMoved()`, `mousePressed()` and `mouseReleased()` implementations

`frameRendered()` is implemented the same way, just forward [`FrameEvent`][FrameEvent] object to corresponding member function

```c++
void ogre_app::frameRendered(FrameEvent const & evt) {
	_cameraman->frameRendered(evt);
}
```

In `ogre_app::setup()` member function we create *floor* graphics object with

```c++
MeshManager::getSingleton().createPlane("floor", ResourceGroupManager::DEFAULT_RESOURCE_GROUP_NAME,
	Plane(Vector3::UNIT_Y, -1), 250, 250, 25, 25, true, 1, 15, 15, Vector3::UNIT_Z);

Entity * floor = scene->createEntity("Floor", "floor");
floor->setMaterialName("Examples/Rockwall");
floor->setCastShadows(false);
root_nd->attachObject(floor);
```

and OGRE head graphics object with

```c++
Vector3 const head_pos = {0, 30, 0};
SceneNode * head_nd = root_nd->createChildSceneNode();
head_nd->setPosition(head_pos);
Entity * head = scene->createEntity("ogrehead.mesh");
head_nd->attachObject(head);
```

then set camera to look at OGRE head this way

```c++
SceneNode * cam_nd = root_nd->createChildSceneNode();
cam_nd->setPosition(0, 30, 140);
cam_nd->lookAt(head_pos/2 + Vector3{0, 0, -1}, Node::TS_PARENT);
```

Initialize `_cameraman` member variable and set camera to free look mode with

```c++
_cameraman = make_unique<CameraMan>(cam_nd);
_cameraman->setStyle(CS_FREELOOK);
```

and at the end of `setup()` grab mouse with 

```c++
setWindowGrab();
```

function calll.


## Building

Building the project is easy, just download sample repository with

```bash
git clone https://github.com/sansajn/ogre-guide.git
```

command, go to `camera/` directory and build camera sample with

```c++
cd strarter-guide/camera
scons
```

commands. On successfull build `main` binary should be created so run `./main` command to run the sample. You should be now able to move with mouse and see that camera view is changing this way 

![ogre_camera_head.webp](https://github.com/sansajn/ogre-guide/raw/master/doc/camera_head.webp "moving with camera in the sample window")

We are done!


[CameraMan]: https://ogrecave.github.io/ogre/api/latest/class_ogre_bites_1_1_camera_man.html
[InputListener]: https://ogrecave.github.io/ogre/api/latest/struct_ogre_bites_1_1_input_listener.html
[FrameEvent]: https://ogrecave.github.io/ogre/api/latest/struct_ogre_1_1_frame_event.html