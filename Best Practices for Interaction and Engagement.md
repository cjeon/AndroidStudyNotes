# [Best Practices for Interaction and Engagement](https://developer.android.com/training/best-ux.html)  

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

