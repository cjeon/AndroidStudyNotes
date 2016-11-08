# [Best Practices for Interaction and Engagement](https://developer.android.com/training/best-ux.html)  

# [Back & Up](https://developer.android.com/design/patterns/navigation.html)

Back : navigate to previous screen.  
Up : navigate to parent screen.

# [Ancestral & Temporal ](https://developer.android.com/training/design-navigation/ancestral-temporal.html)

Temporal(back) : Navigate back to previous screen (regardless of the app) 
Ancestral(Up & Home) : Navigate to parent of current screen (regardless of the app flow) 

# Creating Swipe Views with Tabs

> Use `ViewPager` widget to implement swipe views with multiple fragments.

1. Put `ViewPager` in the layout.
2. Create and attach `PagerAdapter` to the `ViewPager`.
3. `PagerAdapter` supplies pages to `ViewPager` when user scrolls through.

## Example `ViewPager` in layout

### 1. Whole screen is swipable view pager.

``` xml
<android.support.v4.view.ViewPager
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/viewPager">
</android.support.v4.view.ViewPager>
```

### 2. Only 150dp is swipable view pager.

``` xml
<android.support.v4.view.ViewPager
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:id="@+id/viewPager">
</android.support.v4.view.ViewPager>
```

## Attaching `PagerAdapter` to `ViewPager`

``` java
// get viewPager by id
ViewPager viewPager = (ViewPager) findViewById(R.id.viewPager);
// get predefined FragmentPagerAdapter
FragmentPagerAdapter fragmentPagerAdapter = new ExampleFragmentPagerAdapter(getSupportFragmentManager());
// set Adapter.
viewPager.setAdapter(fragmentPagerAdapter);
```

## Example implementation of `PagerAdapter`

There are two recommended kinds of adapters. `FragmentPagerAdapter` and `FragmentStatePagerAdapter`. Both use fragments. The difference is `FragmentPagerAdapter` saves fragment data ["in the fragment manager as long as the user can return to the page."](https://developer.android.com/reference/android/support/v4/app/FragmentPagerAdapter.html) while `FragmentStatePagerAdapter` [destroys fragments as the user navigates to other pages, minimizing memory usage.](https://developer.android.com/training/implementing-navigation/lateral.html).

Each PagerAdapters only need two method implementations. `getCount()` to get max count of pages, and `getItem()` to get fragment.

Below are minimal implementations of two different pager adapters. Only difference is `getCount()`.

### 1. `FragmentPagerAdapter`

``` java
public class ExampleFragmentPagerAdapter extends FragmentPagerAdapter {
    public ExampleFragmentPagerAdapter(FragmentManager fragmentManager) {
        super(fragmentManager);
    }

    @Override
    public int getCount() { return 10; }

    @Override
    public Fragment getItem(int position) {
        Fragment fragment = new SwipeViewFragment();
        Bundle args = new Bundle();
        args.putInt(SwipeViewFragment.ARG_OBJECT, position);
        fragment.setArguments(args);
        return fragment;
    }
}
```

### 2. `FragmentStatePagerAdapter`

``` java
public class ExampleFragmentStatePagerAdapter extends FragmentStatePagerAdapter {
    public ExampleFragmentStatePagerAdapter(FragmentManager fragmentManager) { super(fragmentManager); }

    @Override
    public int getCount() {
        return 100;
    }

    @Override
    public Fragment getItem(int position) {
        Fragment fragment = new SwipeViewFragment();
        Bundle args = new Bundle();
        args.putInt(SwipeViewFragment.ARG_OBJECT, position);
        fragment.setArguments(args);
        return fragment;
    }
}
```

## Creating Tabs
> **Note: [Official doc](https://developer.android.com/training/implementing-navigation/lateral.html) introduces deprecated method. Use `TabLayout` instead.**

Use `android.support.design.widget.TabLayout` with above made `PagerAdapter`.

1. Define `TabLayout` & `ViewPager` in the layout.
2. Link `TabLayout` with adapter by `setupWithViewPager()` method.


### 1. layout(minimal)

``` xml
<android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.TabLayout
            android:id="@+id/tabLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />

</android.support.v4.view.ViewPager>
```

### 2. Code(minimal)

``` java
public class SwipeViewWithTabsActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_swipe_view_with_tabs);

        // Get view components and adapter.
        TabLayout tabLayout = (TabLayout) findViewById(R.id.tabLayout);
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewPager);
        ExampleFragmentStatePagerAdapter pagerAdapter = new ExampleFragmentStatePagerAdapter(getSupportFragmentManager());
        
        // Set ViewPager's adpater.
        viewPager.setAdapter(pagerAdapter);
        
        // Link TabLayout with view pager.
        tabLayout.setupWithViewPager(viewPager);
    }
}
```

# [Creating a Navigation Drawer](https://developer.android.com/training/implementing-navigation/nav-drawer.html)

Use `android.support.v4.widget.DrawerLayout`

1. Define a layout. With at least two child. 1. Main content (what users usually see) and 2. Drawer.
2. Attach `OnItemClickListener` to Drawer's listview layout.
3. Done!

1. Layout(minimal)

``` xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
                    ...
    android:id="@+id/drawer_layout">

    <!-- Content -->
    <LinearLayout
        android:id="@+id/content_layout"
                    ...
        android:orientation="vertical">
    </LinearLayout>

    <!-- The navigation drawer -->
    <ListView 
        android:id="@+id/left_drawer"
                    ...
        android:background="@color/Gray"/>
</android.support.v4.widget.DrawerLayout>
```

2. DrawerActivity.java

``` java
public class DrawerActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_drawer);

        // Drawer settings
        DrawerLayout drawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        ListView listView = (ListView) findViewById(R.id.left_drawer);
        String[] names = getResources().getStringArray(R.array.drawer_array);

        listView.setAdapter(new ArrayAdapter<String>(this, R.layout.drawer_item_textview, names));
        DrawerItemClickListener drawerItemClickListener = new DrawerItemClickListener();
                    ...
        listView.setOnItemClickListener( drawerItemClickListener );
    }
}
```

3. drawerItemClickListener

``` java
public class DrawerItemClickListener implements ListView.OnItemClickListener {
    public FragmentManager fragmentManager;
    public ListView listView;
    public DrawerLayout drawerLayout;

    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        selectItem(position);
    }

    /** Swaps fragments in the main content view */
    private void selectItem(int position) {
        Fragment fragment = new SwipeViewFragment();
        Bundle args = new Bundle();
        args.putInt(SwipeViewFragment.ARG_OBJECT, position);
        fragment.setArguments(args);

        // Insert the fragment by replacing any existing fragment
        fragmentManager.beginTransaction()
                .replace(R.id.content_layout, fragment)
                .commit();

        // Highlight the selected item and close the drawer
        listView.setItemChecked(position, true);
        drawerLayout.closeDrawer(listView);
    }
}
```

# [Providing Proper Back Navigation](https://developer.android.com/training/implementing-navigation/temporal.html)

> Back navigation is how users move backward through the history of screens they previously visited. All Android devices provide a Back button for this type of navigation, so **your app should not add a Back button to the UI**. (...) However, there are a few cases in which your app should manually specify the Back behavior in order to provide the best user experience.

Two ways are suggested by the official doc.

1. Start multiple activities: bundle them with `TaskStackBuilder` and launch this by `startActivities()`.
2. Start pending intent: Start an activity, when it destroys, launch pending intent(usually sent to another app). Build the stack with `TaskStackBuilder` and launch this by `send`.

## Simple example of multiple activities:

``` java
// in MainActivity.java
public void openCustomBackButtonActivity(View view) {
    Intent intent = new Intent(this, FragmentStateSwipeViewActivity.class);
    Intent intent2 = new Intent(this, FragmentSwipeViewActivity.class);
    Intent main = new Intent(this, MainActivity.class);
    Intent[] intents = TaskStackBuilder.create(this)
            .addNextIntent(main)
            .addNextIntent(intent)
            .addNextIntent(intent2)
            .getIntents();
    startActivities(intents);

}
```

## Example of pending intent:

``` java
// TODO: Fill here
```

## Fragment Navigations

When using `FragmentManager`, you can do this:

``` java
getSupportFragmentManager().beginTransaction()
                           .add(detailFragment, "detail")
                           // below line is crucial
                           .addToBackStack()
                           .commit();
```

In other cases, implementing `onBackPressed()` with `Stack` would work.

## WebView

Override `onBackPressed` .

``` java
@Override
public void onBackPressed() {
    if (mWebView.canGoBack()) {
        mWebView.goBack();
        return;
    }

    // Otherwise defer to system default behavior.
    super.onBackPressed();
}
```



