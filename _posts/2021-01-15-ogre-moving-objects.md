---
layout: post
title: "OGRE, moving objects and GUI in cube rain scene"
date: 2021-01-15 17:40:00 +0100
categories: ogre guide
comments: true
author: Adam Hlavatovic
---

This post is the second post in my miniseries to OGRE graphics library and today we learn how to handle moving object in a scene and GUI integration.


### Content

- [Cube rain](#cube-rain)
- [Basic concept implementation](#basic-concept-implementation)
- [GUI integration](#gui-integration)
- [Changing number of cubes](#changing-number-of-cubes)


## Cube rain

We are going to create scene with "falling" cubes from top of the scene to its bottom. The idea is to create some cubes at the beginning, let them fall and recreate those that reached the ground so it looks like rain of cubes.

![cube rain scene screenshot](https://raw.githubusercontent.com/sansajn/ogre-guide/master/doc/ogre_cube_rain.png)


## Basic concept implementation

We can start the same way as in case of previous camera sample and deriving `ogre_app` from `ApplicatonContext`, `InputListener` and this time also from `RenderTargetListener`.

`RenderTargetListener` allows us to receive notification before viewport update in which case `preViewportUpdate()` member function is called. Our custom implementation will handle GUI, but we will go through GUI handling later in this article.

Let's start with constructor where we initialize pool of 300 cubes stored in `_cubes` vector this way

```c++
_cubes.resize(_cube_count);
for (cube_object & cube : _cubes)
    cube = new_cube();

_cube_nodes.resize(_cube_count);
```

we also need one scene node instance for each cube in scene stored in `_cube_notes` vector, that is why `_cube_nodes.resize()` is called. 

Let's now focuse on `setup_scene()` member function, where at the beginning we create lights and camera controller pretty much the same way as in camera sample. Then in a for loop we create node for each cube and store it in `_cube_nodes` via iterator whis way

```c++
auto cube_nodes_it = begin(_cube_nodes);

for (cube_object & cube : _cubes) {
	*cube_nodes_it = create_cube_node(scene, cube);
		++cube_nodes_it;
	
	*cube_nodes_it = node;  // save node for later update
	++cube_nodes_it;
}
```

Helper member function `create_cube_node()` implementation is pretty straightforward. It creates cube entity and associate it with a scene node after scaling so not all cubes looks the same, this way

```c++
SceneNode * ogre_app::create_cube_node(SceneManager & scene, cube_object const & cube) {
	Entity * cube_model = scene.createEntity(SceneManager::PT_CUBE);
	cube_model->setMaterialName("cube_color");  // see media/cube.material

	SceneNode * nd = scene.getRootSceneNode()->createChildSceneNode(cube.position);
	Real model_scale = 0.2 * (2.0 / cube_model->getBoundingBox().getSize().x);
	Real cube_scale = model_scale * cube.scale;
	nd->setScale(cube_scale, cube_scale, cube_scale);
	nd->attachObject(cube_model);

	return nd;
}
```

At the end of `setup_scene()` we create xyz axis at `(0,0,0)` position.

That was pretty much the setup and after that we ends up with a bunch of cubes in scene, but these cubes are not yet moving. To create movement we need periodically update every cube's position in a `update()` member function.

Class `ApplicationContext` offers `frameStarted()` function which can be used as basis for our `update()` this way

```c++
bool ogre_app::frameStarted(FrameEvent const & evt) {
	duration<double> dt{evt.timeSinceLastFrame};
	update(dt);
	return ApplicationContext::frameStarted(evt);
}
```

and finally in `update()` implementation we update every cube's position this way

```c++
constexpr Real fall_speed = 3;
constexpr Real fall_off_threshold = -10.0;

auto cube_node_it = begin(_cube_nodes);
for (cube_object & cube : _cubes) {
	cube.position.y -= fall_speed * (2.0 - cube.scale) * dt.count();
	if (cube.position.y < fall_off_threshold)
		cube = new_cube();

	(*cube_node_it)->setPosition(cube.position);  // update scene position
	++cube_node_it;
}
```

where new cube y position is calculated by `y -= fall_speed * (2.0 - cube.scale) * dt` formula where `2.0 - cube.scale` adds little bit variation so not all cubes are falling the same way. 


## GUI integration

We would like to change number of falling cubes in our scene so some GUI integration comes handy and luckylly for us OGRE already integrate an awesome *Dear ImGui* library. All we need to do in `ogre_app` is to override `preViewportUpdate()` member function from `RenderTargetListener` this way

```c++
void ogre_app::preViewportUpdate(RenderTargetViewportEvent const & evt) {
	if (!evt.source->getOverlaysEnabled())
		return;

	ImGuiOverlay::NewFrame();

	update_gui();
}
```

and in `update_gui()` member function we create iteger slider this way

```c++
void ogre_app::update_gui() {
	ImGui::Begin("Info");  // begin window

	ImGui::SliderInt("Number of cubes", &_cube_count, 100, 1500);

	ImGui::End();  // end window

	ImGui::Render();
}
```

where number of cubes is stored as `_cube_count` private member variable. Then we need to handle keyboard and mouse inputs with `ImGuiListener` instance stored as `_imgui_listener` member variable. We can then use `InputListenerChain` class to chain input handling for `ImGuiInputListener` and `CameraMan` instances at the end of `setup()` member function this way

```c++
_imgui_listener = make_unique<ImGuiInputListener>();
_input_listeners = InputListenerChain({_imgui_listener.get(), _cameraman.get()});
```

Keyboard and mouse `InputListener` overrides in `ogre_app` looks this way

```c++
bool ogre_app::keyReleased(KeyboardEvent const & evt) {
	return _input_listeners.keyReleased(evt);
}
```

> **tip**: watch out implementations for `keyPressed()`, `mouseMoved()`, `mousePressed()` and `mouseReleased()` in [`cube_rain.cpp`](https://github.com/sansajn/ogre-guide/blob/f6d3859f8d72c706aeca760ea839858e5a22872a/moving_ojects/cube_rain.cpp#L253) file 


## Changing number of cubes

We heve made small simplification during `update()` description and skipped handling number of cubes option change via slider widget.

There are two scenarios there, we can eigther end up with less cubes in the scene or with more cubes in the scene after using the slider widget. In both cases we resize `_cubes` vector with 

```c++
int prev_cube_count = size(_cubes);
_cubes.resize(_cube_count);
```

and then in case we end up with less cubes we remove additional cube nodes from scene graph whis way

```c++
if (_cube_count < prev_cube_count) {
	for_each(begin(_cube_nodes) + _cube_count, end(_cube_nodes),
		[&root](SceneNode * nd){root.removeChild(nd);});

	_cube_nodes.resize(_cube_count);
}
```

In case we end up with more cubes, we need to add additional cubes to scene graph this way

```c++
else if (_cube_count > prev_cube_count) {
	_cubes.resize(_cube_count);
	_cube_nodes.resize(_cube_count);

	for (int i = 0; i < _cube_count - prev_cube_count; ++i) {
		cube_object & cube = _cubes[prev_cube_count + i];
		cube = new_cube();

		_cube_nodes[prev_cube_count + i] = create_cube_node(*_scene, cube);
	}
}
```
