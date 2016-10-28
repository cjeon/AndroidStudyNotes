# Adding animations
Here we cover not scholastic, techninal knowledge, but rather real-world implementation of widely used animations.

We are going to see

1. Crossfading Views
2. View pager (already covered)
3. Card flip animations
4. Zooming a View
5. Animating layout changes.

# Crossfading Views
[example](https://developer.android.com/training/animation/anim_crossfade.mp4)

Basically we are going to set alpha of a **fading in view** 0(invisible) to 1(visible), and **fading out view** 1(visible) to 0(invisible).

Three steps.

1. Create member variables for the views that you want to crossfade. You need these references later when modifying the views during the animation.
2. For the view that is being faded in, set its visibility to GONE. This prevents the view from taking up layout space and omits it from layout calculations, speeding up processing.
3. Cache the config_shortAnimTime system property in a member variable. This property defines a standard "short" duration for the animation. This duration is ideal for subtle animations or animations that occur very frequently. config_longAnimTime and config_mediumAnimTime are also available if you wish to use them.

In below code, `mContentView` is what is coming in, and `mLoadingView` is what is going away.

``` java
public class CrossfadeActivity extends Activity {

    private View mContentView;
    private View mLoadingView;
    private int mShortAnimationDuration;

    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crossfade);

        mContentView = findViewById(R.id.content);
        mLoadingView = findViewById(R.id.loading_spinner);

        // Initially hide the content view.
        mContentView.setVisibility(View.GONE);

        // Retrieve and cache the system's default "short" animation time.
        mShortAnimationDuration = getResources().getInteger(
                android.R.integer.config_shortAnimTime);
    }
```

``` java
private View mContentView;
private View mLoadingView;
private int mShortAnimationDuration;

...

private void crossfade() {

    // Set the content view to 0% opacity but visible, so that it is visible
    // (but fully transparent) during the animation.
    mContentView.setAlpha(0f);
    mContentView.setVisibility(View.VISIBLE);

    // Animate the content view to 100% opacity, and clear any animation
    // listener set on the view.
    mContentView.animate()
            .alpha(1f)
            .setDuration(mShortAnimationDuration)
            .setListener(null);

    // Animate the loading view to 0% opacity. After the animation ends,
    // set its visibility to GONE as an optimization step (it won't
    // participate in layout passes, etc.)
    mLoadingView.animate()
            .alpha(0f)
            .setDuration(mShortAnimationDuration)
            .setListener(new AnimatorListenerAdapter() {
                @Override
                public void onAnimationEnd(Animator animation) {
                    mLoadingView.setVisibility(View.GONE);
                }
            });
}
```
# Displaying Card Flip Animations
[Example](https://developer.android.com/training/animation/anim_card_flip.mp4)

## Create the Animators

We need to make two animations. 

1. Card coming in from left(`card_flip_left_in`)
2. Card going away to left(`card_flip_left_out`)
3. Card coming in from right(`card_flip_right_in`)
4. Card going away to right(`card_flip_right_out`)

codes:  
`card_flip_left_in.xml`

``` xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Before rotating, immediately set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:duration="0" />

    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="-180"
        android:valueTo="0"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 1. -->
    <objectAnimator
        android:valueFrom="0.0"
        android:valueTo="1.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```
It's invisible (because `alpha = 0`) until the halfway (because until the card is turned half way this is the backside of a card) and it becomes visible when the card has turned more than 90 degrees.

Other codes are similar.
`card_flip_right_out.xml`

``` xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="-180"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

## Create the Views
We need two layouts for front, and back of a card. Text view is like below,

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#a6c"
    android:padding="16dp"
    android:gravity="bottom">

    <TextView android:id="@android:id/text1"
        style="?android:textAppearanceLarge"
        android:textStyle="bold"
        android:textColor="#fff"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/card_back_title" />

    <TextView style="?android:textAppearanceSmall"
        android:textAllCaps="true"
        android:textColor="#80ffffff"
        android:textStyle="bold"
        android:lineSpacingMultiplier="1.2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/card_back_description" />

</LinearLayout>
```

and image view is like below

``` xml
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:src="@drawable/image1"
    android:scaleType="centerCrop"
    android:contentDescription="@string/description_image_1" />
```

### Create the Fragment
Then we are going to put them in two separate fragments.

``` java
public class CardFlipActivity extends Activity {
    ...
    /**
     * A fragment representing the front of the card.
     */
    public class CardFrontFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_card_front, container, false);
        }
    }

    /**
     * A fragment representing the back of the card.
     */
    public class CardBackFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_card_back, container, false);
        }
    }
}
```

## Animate the Card Flip
Since we have fragments in our codes, it is good idea to define a container in our layout.

``` xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Load front card fragment onto the layout

``` java
public class CardFlipActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_activity_card_flip);

        if (savedInstanceState == null) {
            getFragmentManager()
                    .beginTransaction()
                    .add(R.id.container, new CardFrontFragment())
                    .commit();
        }
    }
    ...
}
```

and lastly, animate.

Below code does followings:
1. Sets the custom animations that you created earlier for the fragment transitions.
2. Replaces the currently displayed fragment with a new fragment and animates this event with the custom animations that you created.
3. Adds the previously displayed fragment to the fragment back stack so when the user presses the Back button, the card flips back over.

``` java
private void flipCard() {
    if (mShowingBack) {
        getFragmentManager().popBackStack();
        return;
    }

    // Flip to the back.

    mShowingBack = true;

    // Create and commit a new fragment transaction that adds the fragment for
    // the back of the card, uses custom animations, and is part of the fragment
    // manager's back stack.

    getFragmentManager()
            .beginTransaction()

            // Replace the default fragment animations with animator resources
            // representing rotations when switching to the back of the card, as
            // well as animator resources representing rotations when flipping
            // back to the front (e.g. when the system Back button is pressed).
            .setCustomAnimations(
                    R.animator.card_flip_right_in,
                    R.animator.card_flip_right_out,
                    R.animator.card_flip_left_in,
                    R.animator.card_flip_left_out)

            // Replace any fragments currently in the container view with a
            // fragment representing the next page (indicated by the
            // just-incremented currentPage variable).
            .replace(R.id.container, new CardBackFragment())

            // Add this transaction to the back stack, allowing users to press
            // Back to get to the front of the card.
            .addToBackStack(null)

            // Commit the transaction.
            .commit();
}
```

`setCustomAnimations` takes `enter, exit, popEnter, popExit` as parameters.  
[Documentation](https://developer.android.com/reference/android/app/FragmentTransaction.html#setCustomAnimations(int,%20int,%20int,%20int)) says: `Set specific animation resources to run for the fragments that are entering and exiting in this transaction. The popEnter and popExit animations will be played for enter/exit operations specifically when popping the back stack.`

# Zooming a View
[Example](https://developer.android.com/training/animation/anim_zoom.mp4)

Code too long to attach see [here](https://developer.android.com/training/animation/zoom.html)

1. Load high-resolution images 
2. When thumbnail on click, calculate size and position of thumbnail and final size and position of the high-res image.
3. Animate to zoom in image. Take step backwards to zoom out.

Code explained:

1. `if (mCurrentAnimator != null)`: no repetition!
2. `new Rect()` parts and `offset` parts: Initial size is this and final size is this.
3. `if ((float) finalBounds.width() / finalBounds.height() > (float) startBounds.width() / startBounds.height())`: Do [center crop](http://i.stack.imgur.com/DGeyI.png). There's no guarantee that the initial ration and final ratio are always same.
4. `AnimatorSet set = new AnimatorSet();`: move object from `startBounds.left and .top` to `finalBounds.left and .top`.

# Animating Layout Changes
## Create the Layout

``` xml
<LinearLayout android:id="@+id/container"
    android:animateLayoutChanges="true"
    ...
/>
```

## Add, Update, or Remove Items from the Layout
 
 ``` java
 private ViewGroup mContainerView;
...
private void addItem() {
    View newView;
    ...
    mContainerView.addView(newView, 0);
}
```