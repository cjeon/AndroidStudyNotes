# [Best Practices for Interaction and Engagement](https://developer.android.com/training/best-ux.html)  

index:

1. [Back & UP](#back--up)
2. [Ancestral & Temporal](#ancestral--temporal)
3. [Creating Swipe Views with Tabs](#creating-swipe-views-with-tabs)
4. [Creating a Navigation Drawer](#creating-a-navigation-drawer)
5. [Providing Proper Back Navigation](#providing-proper-back-navigation)
6. [Building a Notification](#building-a-notification)
7. [Preserving Navigation when Starting an Activity](#preserving-navigation-when-starting-an-activity)
8. [Using Big View Styles](#using-big-view-styles)
9. [Displaying Progress in a Notification](#displaying-progress-in-a-notification)

# [Back & Up](https://developer.android.com/design/patterns/navigation.html)

Back : navigate to previous screen.  
Up : navigate to parent screen.

# [Ancestral & Temporal](https://developer.android.com/training/design-navigation/ancestral-temporal.html)

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

# [Notifying the User](https://developer.android.com/training/notify-user/index.html)

# [Building a Notification](https://developer.android.com/training/notify-user/build-notification.html)

Build notification using `Notification.Builder` or `NotificationCompat.Builder`. Mandatory options are : 1. `setSmallIcon()`, 2. `setContentTitle()`, 3. `setContentText()`. You can find other options at [official doc](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html).

Implementation:

1. Build a notification
2. Add pending intent to launch when user tabs notification.
3. Register notification to the notification manager.

``` java
public void sendNotification(View view) {
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
            .setSmallIcon(R.drawable.ic_error_outline_white_24dp)
            .setContentTitle("Notification Title")
            .setContentText("Notification text")
            .setTicker("Note !")
            .setVibrate(new long[] {1000, 1000});

    Intent main = new Intent(this, MainActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, main, PendingIntent.FLAG_UPDATE_CURRENT);
    builder.setContentIntent(pendingIntent);
    int notificationId = 1;
    NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
    notificationManager.notify(notificationId, builder.build());
} 
```

# [Preserving Navigation when Starting an Activity](https://developer.android.com/training/notify-user/navigation.html)

When user clicks an alarm sent by an application, user will see one of the two kinds of activity: 

1. Regular Activity: This is an activity that user can regularly see during app navigation.
2. Special Activity: This is an activity that user can only see through clicking the alarm, i.e. it has no regular entry. + It's very hard (or impossible) for user to come back to this activity.

Below are ways to provide appropriate context for both situations.

## Regular activity

1. Define activity hierarchy in the manifest.

Here, I'm going to re-use the `UpActivity` we defined above (because it already has well defined structure.)

AndroidManifest.xml

``` xml
<activity android:name=".Activity.MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
   ...
<activity
    android:name=".Activity.UpActivity"
    android:label="Up">
    <meta-dat
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".Activity.MainActivity" />
</activity>
```

Then

1. Use TaskStackBuilder to build a stack of tasks you want user to go through, and build a pending intent.
2. Build a notification (done before)
3. Add notification to NotificationManager w/ pending intent.

``` java
public void sendNotificationWithRegularContext(View view) {
    // 1. Build a stack of tasks -> get pending intent
    Intent up = new Intent(this, UpActivity.class);
    TaskStackBuilder taskStackBuilder = TaskStackBuilder.create(this);
    taskStackBuilder.addNextIntentWithParentStack(up);
    PendingIntent pendingIntent = taskStackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);

    // 2. Build a notification.
    int notificationId = 1;
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
            ... (skip notification details like text, vibration.)
    builder.setContentIntent(pendingIntent);

    // 3. Add notification to NotificationManager.
    NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
    notificationManager.notify(notificationId, builder.build());
}
```

## [Set Up a Special Activity PendingIntent](https://developer.android.com/training/notify-user/navigation.html#ExtendedNotification)

Little modification from above regular notification would suffice.

In android manifest, edit `android:taskAffinity` and `android:excludeFromRecents` like below

``` xml
<activity
    android:name=".Activity.SpecialActivity"
    android:taskAffinity=""
    android:excludeFromRecents="true"/>
```

[`android:taskAffinity`](https://developer.android.com/guide/topics/manifest/activity-element.html#aff): 
> The task that the activity has an affinity for. Activities with the same affinity conceptually belong to the same task (to the same "application" from the user's perspective). The affinity of a task is determined by the affinity of its root activity.
The affinity determines two things — the task that the activity is re-parented to (see the allowTaskReparenting attribute) and the task that will house the activity when it is launched with the FLAG_ACTIVITY_NEW_TASK flag.  
By default, all activities in an application have the same affinity. You can set this attribute to group them differently, and even place activities defined in different applications within the same task. **To specify that the activity does not have an affinity for any task, set it to an empty string**.  
If this attribute is not set, the activity inherits the affinity set for the application (see the <application> element's taskAffinity attribute). The name of the default affinity for an application is the package name set by the <manifest> element.

[`android:excludeFromRecents`](https://developer.android.com/guide/topics/manifest/activity-element.html#exclude)
> Whether or not the task initiated by this activity should be excluded from the list of recently used applications, the overview screen. That is, when this activity is the root activity of a new task, this attribute determines whether the task should not appear in the list of recent apps. Set "true" if the task should be excluded from the list; set "false" if it should be included. The default value is "false".

# [Updating Notifications](https://developer.android.com/training/notify-user/managing.html)

Updating notifications: It's simple. Issue a new notification w/ old notification's ID.

Removing notifications: call `NotificationManager.cancel()` or `NotificationManager.cancelAll()`. 

*Note: All notifications are set to be persistent, i.e., it does not go away when users clicks it. (wonder why google made it this way) To make it go away when user clicks notification, set `setAutoCancel(true)`.

# [Using Big View Styles](https://developer.android.com/training/notify-user/expanded.html)

There are many styles provided by Google. ex: BigPictureStyle, BigTextStyle .. etc. Refer to [official doc]() for more info. Here, we are going to test one of those styles, `Big View Style`. 

1. Build a notification as before.
2. Set style of a notification. Use either predefined one, or you can make your own.
3. Attach necessary appendments to the notification. For example, if your new notification requires 2 extra buttons, attach two more intents to the notification.
4. Notify as before.

``` java
    public void sendBigViewNotification(View view) {

        // Like before, we build a notification. (skipped 10+ lines)
        builder.setContentIntent(pendingIntent);

        // We need two more buttons, so make two more intents.
        // Dismiss
        Intent dismissIntent = new Intent(this, NotificationActivity.class)
                .setAction(NotificationActivity.DISMISS);
        PendingIntent dismissPendingIntent = PendingIntent.getActivity(this, 0, dismissIntent, PendingIntent.FLAG_UPDATE_CURRENT);

        // Snooze
        PendingIntent snoozePendingIntent = PendingIntent.getActivity(this, 0, new Intent(), PendingIntent.FLAG_UPDATE_CURRENT);

        // Then set style of the notification. Add icons, too.
        builder.setStyle(new NotificationCompat.BigTextStyle().bigText("BIG VIEW MESSAGE"))
                .addAction(R.drawable.ic_delete_forever_black_24dp, "dismiss", dismissPendingIntent)
                .addAction(R.drawable.ic_done_black_24dp, "snooze", snoozePendingIntent);

        // Notify as before.
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        notificationManager.notify(1, builder.build());
    }
```

# [Displaying Progress in a Notification](https://developer.android.com/training/notify-user/display-progress.html)

Use `setProgress` function. Takes 3 argument. 

|Parameters||
--|--
max|int
progress|int
indeterminate|boolean

Set indeterminate to false if we do not know when the task will be completed. (covered right next) Below example shows how to setup a notification when we [know|can notify] the progress.

1. Make normal Notification builder.
2. Set its progress to 0
3. Increment its progress.
4. When progress is full, notify user with "progress done!"

Note: Updating should be done in the background thread. RXjava can come in handy.

``` java
public void getFixedDurationProgressNotification(View view) {
    // Make normal notification builder w/ setProgress(1,0,false)
    timedBuilder = new NotificationCompat.Builder(this);
    timedBuilder.setContentTitle("Progress Bar")
            .setContentTitle("Progressing...")
            .setSmallIcon(R.drawable.ic_schedule_black_24dp)
            .setProgress(1,0,false);
    notificationManager.notify(id, timedBuilder.build());

    Observable.interval(50, TimeUnit.MILLISECONDS, Schedulers.io())
            .takeWhile(num -> num <= 60)
            .subscribe(
                    // Increment progress
                    percentage -> {
                        timedBuilder.setProgress(60, percentage.intValue(), false);
                        notificationManager.notify(id, timedBuilder.build()); },
                    ignore -> {},
                    // When done, notify user.
                    () -> {
                        timedBuilder.setProgress(1, 1, false)
                                .setContentTitle("Done!")
                                .setContentText("Done!")
                                .setAutoCancel(true);
                        notificationManager.notify(id, timedBuilder.build()); }
            );
}
```

# [Supporting Swipe-to-Refresh](https://developer.android.com/training/swipe/index.html)

Vertical swipe to refresh contents: often used in SNS(social media) timelines.

## [Adding Swipe-to-Refresh To Your App](https://developer.android.com/training/swipe/add-swipe-interface.html)

3 Methods to support refresh will be covered below.

1. Adding, and responding to swipe-to-refresh gesture.
2. Adding, and responding to refresh menu item.
3. Adding, and responding to refresh menu button. (*Google doesn't recommend this)

Google recommends developer implement both gesture and menu item, because some users may not be able to swipe the screen. However, Google does not recomment the #3 method, because

> If you display the action as a button, users may assume that the refresh button action is different from the swipe-to-refresh action.

I do not buy this, so I also implemented the third way. (Clicking menu **BOTHERS** user!)

### 1. Adding, and responding to swipe-to-refresh gesture.

1. Use `android.support.v4.widget.SwipeRefreshLayout`.
2. Implement `SwipeRefreshLayout.setOnRefreshListener`.

Simple SwipeRefreshLayout w/ one ListView.

``` xml
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
        ...
    android:paddingTop="@dimen/activity_vertical_margin">

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</android.support.v4.widget.SwipeRefreshLayout>
```

Java code for gesture response.

`onCreate`

``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_swipe_refresh);

        ... // some listView adapter settings.

    swipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.activity_swipe_refresh);
    swipeRefreshLayout.setOnRefreshListener(this::addListViewItem);
}
```

`addListViewItem` (Note the `swipeRefreshLayout.setRefreshing(false);`. If you don't set this false, the refreshing wheel will go on forever.)

```java
public void addListViewItem() {
    listItems.add(number);
    listAdapter.notifyDataSetChanged();
    number += 1;
    swipeRefreshLayout.setRefreshing(false);
}
```

### 2. Adding, and responding to refresh menu item. & 3. Adding, and responding to refresh menu button.

Both are very similar except few lines, so I'll just introduce the two together.

1. Define custom menu
2. Inflate the menu by overriding `onCreateOptionsMenu`.
3. Respond to the menu click by overrding `onOptionsItemSelected`.

#### 1. `menu_refresh.xml`

``` xml
<?xml version="1.0" encoding="utf-8"?>
<menu
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <!-- Button -->
    <item
        android:id="@+id/button_refresh"
        android:icon="@drawable/ic_refresh_black_24dp"
        android:title="Refresh"
        app:showAsAction="ifRoom" />
    <!-- Menu item -->
    <item
        android:id="@+id/menu_refresh"
        app:showAsAction="never"
        android:title="Refresh"/>
</menu>
```
Button needs icon because it will be displayed as button if possible(because `app:showAsAction="ifRoom"`), but menu item does not need one (because `app:showAsAction="never"`). 

#### 2. `onCreateOptionsMenu` Menu inflation.

``` java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater menuInflater = getMenuInflater();
    menuInflater.inflate(R.menu.menu_refresh, menu);
    return super.onCreateOptionsMenu(menu);
}
```

#### 3. `onOptionsItemSelected` responding.

``` java
@Override
public boolean onOptionsItemSelected(MenuItem menuItem) {
    switch (menuItem.getItemId()) {
        case R.id.menu_refresh:
        case R.id.button_refresh:
            addListViewItem();
            return true;
    }
    return true;
}
```

