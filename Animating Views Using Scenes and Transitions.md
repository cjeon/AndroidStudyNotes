**Good news: We are out of OpenGL !!** (at least it is for me)

anyways,

# Animating Views Using Scenes and Transitions
Animation on views, when used appropriately, gives user a nice UX. For this, Android Framework provides `transitions framework`.
>Transition: Animated changes between two view hierarchies.  

Main topic here are transitions (and no OpenGL).

# The Transitions Framework
The framework has the following features:

1. **Group-level animations**  
Applies one or more animation effects to all of the views in a view hierarchy.
2. **Transition-based animation**  
Runs animations based on the changes between starting and ending view property values.
3. **Built-in animations**  
Includes predefined animations for common effects such as fade out or movement.
4. **Resource file support**  
Loads view hierarchies and built-in animations from layout resource files.
5. **Lifecycle callbacks**  
Defines callbacks that provide finer control over the animation and hierarchy change process.

## Overview
[Transition Example 1](https://developer.android.com/images/transitions/transition_sample_video.mp4)  
![alt text](https://developer.android.com/images/transitions/transitions_diagram.png)

To make transition, we need to do the following:
1. Create `scenes` for two view hierarchies.
2. Create `transition` for each animations.
3. Use `TransitionManager` to start transition.

## Scenes
A scene stores the state of a view hierarchy, including all its views and their property values.  
We can create scenes in two ways.
1. Create from layout file.
2. Create during runtime.

We always need a ending scene, but not starting scene. If not specified, TransitionMananger will automatically make a starting scene.  
> In most cases, you do not create a starting scene explicitly. -Official Doc

We can also define actions on scenes.

## Transitions
* Transitions are like effects. With starting scene and ending scene, we can make some fancy visual animations. 
* There are builtin transitions, too.
* Transitions also have lifecycle.

## Limitations
From official doc,
* Animations applied to a SurfaceView may not appear correctly. SurfaceView instances are updated from a non-UI thread, so the updates may be out of sync with the animations of other views.
* Some specific transition types may not produce the desired animation effect when applied to a TextureView.
* Classes that extend AdapterView, such as ListView, manage their child views in ways that are incompatible with the transitions framework. If you try to animate a view based on AdapterView, the device display may hang.
* If you try to resize a TextView with an animation, the text will pop to a new location before the object has completely resized. To avoid this problem, do not animate the resizing of views that contain text.

# Creating a scene




