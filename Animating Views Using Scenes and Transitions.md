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
## Create a Scene From a Layout Resource

`res/layout/activity_main.xml` (note the include)

``` java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/master_layout">
    <TextView
        android:id="@+id/title"
        ...
        android:text="Title"/>
    <FrameLayout
        android:id="@+id/scene_root">
        <include layout="@layout/a_scene" />
    </FrameLayout>
</LinearLayout>
```

`res/layout/a_scene.xml` 

``` java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view1"
        android:text="Text Line 1" />
    <TextView
        android:id="@+id/text_view2"
        android:text="Text Line 2" />
</RelativeLayout>
```

`res/layout/another_scene.xml` (reverse order of TextView)

``` java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view2"
        android:text="Text Line 2" />
    <TextView
        android:id="@+id/text_view1"
        android:text="Text Line 1" />
</RelativeLayout>
```

## Generate Scenes from Layouts

``` java
Scene mAScene;
Scene mAnotherScene;

// Create the scene root for the scenes in this app
mSceneRoot = (ViewGroup) findViewById(R.id.scene_root);

// Create the scenes
mAScene = Scene.getSceneForLayout(mSceneRoot, R.layout.a_scene, this);
mAnotherScene =
    Scene.getSceneForLayout(mSceneRoot, R.layout.another_scene, this);
```

## Create a Scene in Your Code
``` java
Scene mScene;

// Obtain the scene root element
mSceneRoot = (ViewGroup) mSomeLayoutElement;

// Obtain the view hierarchy to add as a child of
// the scene root when this scene is entered
mViewHierarchy = (ViewGroup) someOtherLayoutElement;

// Create a scene
mScene = new Scene(mSceneRoot, mViewHierarchy);
```

## Create Scene Actions
To provide custom scene actions, define your actions as Runnable objects and pass them to the Scene.setExitAction() or Scene.setEnterAction() methods. The framework calls the setExitAction() method on the starting scene before running the transition animation and the setEnterAction() method on the ending scene after running the transition animation.

# Applying a Transition
## Create a Transition
1. Use builtins.
2. Create from resource file.
3. Dynamically create transitions.

**Builtin transitions**

| Class | Tag | Attributes | Effect |  
--------|-----|------------|--------|
|AutoTransition | `<autoTransition/>` |	- |	Default transition. Fade out, move and resize, and fade in views, in that order. |  
|Fade |	`<fade/>` |	android:fadingMode="[fade_in, fade_out, fade_in_out]" |	`fade_in` fades in views, `fade_out` fades out views, `fade_in_out` (default) does a fade_out followed by a fade_in.|  
|ChangeBounds|	`<changeBounds/>` |	- |	Moves and resizes views.|  


## Create a transition instance from a resource file
1. Add the res/transition/ directory to your project.
2. Create a new XML resource file inside this directory.
3. Add an XML node for one of the built-in transitions.

in xml,  
`<fade xmlns:android="http://schemas.android.com/apk/res/android" />`

in code,

``` java
Transition mFadeTransition =
        TransitionInflater.from(this).
        inflateTransition(R.transition.fade_transition);
```

## Create a transition instance in your code

``` java
Transition mFadeTransition = new Fade();
```

# Apply a Transition
`TransitionManager.go(mEndingScene, mFadeTransition);`

## Choose Specific Target Views
We can add, or remove part of view hierarchy by calling: `removeTarget()` and `addTarget()`

## Specify Multiple Transitions
We can congregate multiple transitions into one.

``` java
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
    android:transitionOrdering="sequential">
    <fade android:fadingMode="fade_out" />
    <changeBounds />
    <fade android:fadingMode="fade_in" />
</transitionSet>
```

## Apply a Transition Without Scenes
We can use this approach if transition those not include a lot of change in view. For example, small modification within same layout, or transition to nearly identical layout.

1. When the event that triggers the transition occurs, call the TransitionManager.beginDelayedTransition() method providing the parent view of all the views you want to change and the transition to use. The framework stores the current state of the child views and their property values.
2. Make changes to the child views as required by your use case. The framework records the changes you make to the child views and their properties.
3. When the system redraws the user interface according to your changes, the framework animates the changes between the original state and the new state.

`activity_main.xml`

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <EditText
        android:id="@+id/inputText"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    ...
</RelativeLayout>
```

`MainActivity.java`

``` java
private TextView mLabelText;
private Fade mFade;
private ViewGroup mRootView;
...

// Load the layout
this.setContentView(R.layout.activity_main);
...

// Create a new TextView and set some View properties
mLabelText = new TextView();
mLabelText.setText("Label").setId("1");

// Get the root view and create a transition
mRootView = (ViewGroup) findViewById(R.id.mainLayout);
mFade = new Fade(IN);

// Start recording changes to the view hierarchy
TransitionManager.beginDelayedTransition(mRootView, mFade);

// Add the new TextView to the view hierarchy
mRootView.addView(mLabelText);

// When the system redraws the screen to show this update,
// the framework will animate the addition as a fade in
```

## Define Transition Lifecycle Callbacks
Transition lifecycle callbacks are useful, for example, for copying a view property value from the starting view hierarchy to the ending view hierarchy during a scene change.  
Override `TransitionListener.onTransitionEnd()`.

# Creating Custom Transitions
## Extend the Transition Class
Override below.

``` java
public class CustomTransition extends Transition {

    @Override
    public void captureStartValues(TransitionValues values) {}

    @Override
    public void captureEndValues(TransitionValues values) {}

    @Override
    public Animator createAnimator(ViewGroup sceneRoot,
                                   TransitionValues startValues,
                                   TransitionValues endValues) {}
}
```

## Capture View Property Values
Transition animations use the property animation system described in Property Animation.  
However, a property animation usually needs only a small subset of all the view's property values. nstead, the framework invokes callback methods that allow a transition to capture only the property values it needs and store them in the framework.


### Capturing Starting Values
To pass the starting view values to the framework, implement the `captureStartValues(transitionValues)` method.

example:

``` java
public class CustomTransition extends Transition {

    // Define a key for storing a property value in
    // TransitionValues.values with the syntax
    // package_name:transition_class:property_name to avoid collisions
    private static final String PROPNAME_BACKGROUND =
            "com.example.android.customtransition:CustomTransition:background";

    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        // Call the convenience method captureValues
        captureValues(transitionValues);
    }


    // For the view in transitionValues.view, get the values you
    // want and put them in transitionValues.values
    private void captureValues(TransitionValues transitionValues) {
        // Get a reference to the view
        View view = transitionValues.view;
        // Store its background property in the values map
        transitionValues.values.put(PROPNAME_BACKGROUND, view.getBackground());
    }
    ...
}
```

### Capture Ending Values

``` java
@Override
public void captureEndValues(TransitionValues transitionValues) {
    captureValues(transitionValues);
}
```

### Create a Custom Animator
Override `createAnimator()`. Example in [here](https://github.com/googlesamples/android-CustomTransition/blob/master/Application/src/main/java/com/example/android/customtransition/ChangeColor.java) (Too long to attach)

