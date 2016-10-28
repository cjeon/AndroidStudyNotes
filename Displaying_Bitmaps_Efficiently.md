# Displaying Bitmaps Efficiently

# Subsampling (scale down) of a bitmap 
* Bitmaps can quickly consume memory budget (Android devices have 16MB of memory per application at minimum.)
* [BitmapFactory](https://developer.android.com/reference/android/graphics/BitmapFactory.html) provides many decoding methods. 
* Setting [inJustDecodeBounds](https://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inJustDecodeBounds) to **true** while decoding will *not* allocate memory but just caculate outWidth, outHeight, outMimeType.
* We can decided to load full image (as-is) or to load only part of it (scaled down). Factors to consider are:
    1. Memory cost of an image.
    2. Available memory.
    3. Dimensions of target image containcer (such as ImageView, etc).
    4. Screen size and density.

(after enabling `inJustDecodeBounds = true`)

``` java
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) >= reqHeight
                && (halfWidth / inSampleSize) >= reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
```
*Why **increase by power of two** instead of **decrease by power of two**?  
Reason: because decoder uses final value by rounding down to nearest power of two.

In order to make use of inSampleSize aquired above, we need to set `inJustDecodeBounds` back to `false` and pass `inSampleSize`.

# Image processing off the UI thread
* Heavy tasks should **not** be done on UI Thread, **obviously**.

## Using AsyncTask

* One way to go is to use **AsyncTask**. Imagine we had a async method called `loadBitmap`. It should do followings:
    1. Load an Image,
    2. Process it,
    3. Set image to imageView.  
 
    So that we can just call `loadBitmap` and everything is done in AsyncTask.
* In main activity, we just need to have this method below. We tell AsyncTask what to load, where to load, and that's it.

``` java
public void loadBitmap(int resId, ImageView imageView) {
    BitmapWorkerTask task = new BitmapWorkerTask(imageView);
    task.execute(resId);
}
```
* `BitmapWorkerTask` is a little more complicated, but it mainly does above 3. (Code from [google](https://developer.android.com/training/displaying-bitmaps/process-bitmap.html))
``` java 
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    private final WeakReference<ImageView> imageViewReference;
    private int data = 0;

    public BitmapWorkerTask(ImageView imageView) {
        // Use a WeakReference to ensure the ImageView can be garbage collected
        imageViewReference = new WeakReference<ImageView>(imageView);
    }

    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        data = params[0];
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
    }

    // Once complete, see if ImageView is still around and set bitmap.
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            if (imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```
* Above code does followings:
    1. `BitmapWorkerTask` is a constructor. Note the **WeakReference**. As comment says, if imageView is about to be gc-ed, then this task should not prevent that.
    2. When `execute()` is called, 4 functions are called in order. 4 functions are: 
        1. `onPreExecute`
        2. `doInBackground`
        3. `onProgressUpdate`
        4. `onPostExecute`
        
        Here, since we have `doInBackground` and `onPostExecute`, these would be called in order.
    3. `doInBackground` will decode image, and return `Bitmap`. 
    4. `onPostExecute` will check if `imageView` is still there, and if it is, will set `bitmap` on `imageView`.


## Concurrency

* In android, some view components are **recycled**, and this can cause problems. Since we made weak references to view components above, if certain view components are recycled, unexpected results may happen.
* We do not want this to happen:
    1. More than one async tasks trying to load different images onto a same view (possible if view is recycled).
* So, we are going to check if more than one async task is going to execute on a view, and prevent this from happening by stopping prior one.
* We should make sure that:
    1. Each view should have a mark to tell if some task is working on it. 
    2. When we attach an async task to a view, we should check if another(or same) async task is working on this view.
    3. If we detect it, we should make sure that only one task runs on it. 
        3-1. If another task is running, cancel it, and run newer one.
        3-2. If same task is already running, do nothing.
* We **previously** loaded an image in these steps:
    1. Give `int resId, ImageView imageView` to `BitmapWorkerTask`.
    2. `BitmapWorkerTask` loades image onto imageview. done.
* We are **now** going to load an image in these steps:
    1. Create a `AsyncDrawable` with `placeholderBitmap, task` and bind it with `imageView`.  
        This `AsyncDrawable` has WeakReference to `task`.
    2. `BitmapWorkerTask` loades image onto imageview.
    3. However, before step 1, we check if the image already has a task. If we find one, we cancel it(it's a different task) or we stop(it's the same task).

* Codes are as follows, also from [google](https://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency)

``` java
static class AsyncDrawable extends BitmapDrawable {
    private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;

    public AsyncDrawable(Resources res, Bitmap bitmap,
            BitmapWorkerTask bitmapWorkerTask) {
        super(res, bitmap);
        bitmapWorkerTaskReference =
            new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
    }

    public BitmapWorkerTask getBitmapWorkerTask() {
        return bitmapWorkerTaskReference.get();
    }
}
```
We are going to attach this to `imageView`. Later, we will use this to make sure no two tasks are running on same view.

``` java
public void loadBitmap(int resId, ImageView imageView) {
    if (cancelPotentialWork(resId, imageView)) {
        final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
        final AsyncDrawable asyncDrawable =
                new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
        imageView.setImageDrawable(asyncDrawable);
        task.execute(resId);
    }
}
```
We are executing a task, however, before that, we call `cancelPotentialWork`. This function is called to make sure that no two tasks are running at the same time on same view.
We will view `cancelPotentialWork` later.  
Above code is almost same as before except `asyncDrawable` parts. As explained above, we are binding them with view to make sure no two tasks are running at the same time.

``` java
public static boolean cancelPotentialWork(int data, ImageView imageView) {
    final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

    if (bitmapWorkerTask != null) {
        final int bitmapData = bitmapWorkerTask.data;
        // If bitmapData is not yet set or it differs from the new data
        if (bitmapData == 0 || bitmapData != data) {
            // Cancel previous task
            bitmapWorkerTask.cancel(true);
        } else {
            // The same work is already in progress
            return false;
        }
    }
    // No task associated with the ImageView, or an existing task was cancelled
    return true;
}
```
Above code checks two things:

1. Are any tasks running on the imageView? (If no, it immediately returns true.)
2. If there are, is it trying to load same data as we are currently trying to? Or is it different? (If same, we don't do anything further. If different, we cancel that outdated task.)  

Here, we used `getBitmapWorkerTask` to retrieve some info about imageView.

``` java
private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
   if (imageView != null) {
       final Drawable drawable = imageView.getDrawable();
       if (drawable instanceof AsyncDrawable) {
           final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
           return asyncDrawable.getBitmapWorkerTask();
       }
    }
    return null;
}
```
It checks if any task is running on it (if there is no drawable, we can continue, if there is and it's a instance of `AsyncDrawable`, we should check further.)

Finally, we just need to modify `onPostExecute` of `BitmapWorkerTask`.

``` java
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (isCancelled()) {
            bitmap = null;
        }

        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            final BitmapWorkerTask bitmapWorkerTask =
                    getBitmapWorkerTask(imageView);
            if (this == bitmapWorkerTask && imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```
Not much change. If it's canceled, do nothing. If this task is the task the `imageView` has been waiting for, do change image.

# Caching Bitmaps
So far so good. However, think of this scenario: 
>You are making a gallery. You need to present a lot of images, and user may scroll down, or sometimes up. System may gc your images, even if user may want to see them again. You have to load images already been loaded before again and again.  

We can avoid this by **caching** images. There are two options. **Memory(RAM) Cache** and **Disk Cache**.

## Memory `LruCache`

**LruCache** keeps recently referenced objects in a strong referenced LinkedHashMap. Deletes least referenced item when cache storage runs out.  
Before setting up LruCache, we should choose size of the cache. For that, we need to consider following factors:

1. How many images do we need to cache
2. How big are images?
3. How memory intensive is your app?

Below is example of LruCache.

``` java
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Get max available VM memory, exceeding this amount will throw an
    // OutOfMemory exception. Stored in kilobytes as LruCache takes an
    // int in its constructor.
    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);

    // Use 1/8th of the available memory for this memory cache.
    final int cacheSize = maxMemory / 8;

    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            // The cache size will be measured in kilobytes rather than
            // number of items.
            return bitmap.getByteCount() / 1024;
        }
    };
    ...
}

public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }
}

public Bitmap getBitmapFromMemCache(String key) {
    return mMemoryCache.get(key);
}
```
Things to note are:

1. `LruCache` takes size in kilobytes.
2. As explained above, `LruCache` works like normal hashmap.

Below are how `LruCache` used in practice.

``` java
public void loadBitmap(int resId, ImageView imageView) {
    final String imageKey = String.valueOf(resId);

    final Bitmap bitmap = getBitmapFromMemCache(imageKey);
    if (bitmap != null) {
        mImageView.setImageBitmap(bitmap);
    } else {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }
}
```
As shown above, `loadBitmap` function first looks for image in cache. if it exists, continue with cached data. Else, it starts `BitmapWorkerTask`.

`BitmapWorkerTask` also needs to be modified. We should add procedure to save loaded images in cache.

``` java
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final Bitmap bitmap = decodeSampledBitmapFromResource(
                getResources(), params[0], 100, 100));
        addBitmapToMemoryCache(String.valueOf(params[0]), bitmap);
        return bitmap;
    }
    ...
}
```

## `DiskLruCache`

As its name suggests, `DiskLruCache` is a cache which acts almost same as `LruCache`, but stores data in disk, rather than memory.

``` java
private DiskLruCache mDiskLruCache;
private final Object mDiskCacheLock = new Object();
private boolean mDiskCacheStarting = true;
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB
private static final String DISK_CACHE_SUBDIR = "thumbnails";

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR);
    new InitDiskCacheTask().execute(cacheDir);
    ...
}

class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) {
            File cacheDir = params[0];
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE);
            mDiskCacheStarting = false; // Finished initialization
            mDiskCacheLock.notifyAll(); // Wake any waiting threads
        }
        return null;
    }
}

class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);

        // Check disk cache in background thread
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);

        if (bitmap == null) { // Not found in disk cache
            // Process as normal
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        }

        // Add final bitmap to caches
        addBitmapToCache(imageKey, bitmap);

        return bitmap;
    }
    ...
}

public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }

    // Also add to disk cache
    synchronized (mDiskCacheLock) {
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) {
            mDiskLruCache.put(key, bitmap);
        }
    }
}

public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) {
        // Wait while disk cache is started from background thread
        while (mDiskCacheStarting) {
            try {
                mDiskCacheLock.wait();
            } catch (InterruptedException e) {}
        }
        if (mDiskLruCache != null) {
            return mDiskLruCache.get(key);
        }
    }
    return null;
}

// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();

    return new File(cachePath + File.separator + uniqueName);
}
```

Points to note are:

1. `get` from cache is locked until cache is Initialized. This is here because initiating disk cache takes more time than initiating memory cache.
2. Memory cache was checked in the UI thread, however, checking disk cache is checked in the background because this can take a long time.

## Handling configuration changes

When configuration changes - such as rotating display - it will destroy all the views you made **and make you lose control of caches you made**.  
You can avoid this tragedy by passing your cache to a fragment and keep fragment by calling `setRetainInstance(true)`

``` java
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    RetainFragment retainFragment =
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());
    mMemoryCache = retainFragment.mRetainedCache;
    if (mMemoryCache == null) {
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            ... // Initialize cache here as usual
        }
        retainFragment.mRetainedCache = mMemoryCache;
    }
    ...
}

class RetainFragment extends Fragment {
    private static final String TAG = "RetainFragment";
    public LruCache<String, Bitmap> mRetainedCache;

    public RetainFragment() {}

    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {
        RetainFrasgment fragment = (RetainFragment) fm.findFragmentByTag(TAG);
        if (fragment == null) {
            fragment = new RetainFragment();
            fm.beginTransaction().add(fragment, TAG).commit();
        }
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
}
```
Here, `retainFragment.mRetainedCache` is Initialized when `mMemoryCache` is `null`.  
`RetainFragment` is created, or loaded by `findOrCreateRetainFragment`. It tries to find fragment by `findFragmentByTag`.

# Bitmap + UI examples
## ViewPager

ViewPager: Swipe left and right to change view. (Like Azar main view.) Sometimes loades images onto view.

``` java
public class ImageDetailActivity extends FragmentActivity {
    public static final String EXTRA_IMAGE = "extra_image";

    private ImagePagerAdapter mAdapter;
    private ViewPager mPager;

    // A static dataset to back the ViewPager adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.image_detail_pager); // Contains just a ViewPager

        mAdapter = new ImagePagerAdapter(getSupportFragmentManager(), imageResIds.length);
        mPager = (ViewPager) findViewById(R.id.pager);
        mPager.setAdapter(mAdapter);
    }

    public static class ImagePagerAdapter extends FragmentStatePagerAdapter {
        private final int mSize;

        public ImagePagerAdapter(FragmentManager fm, int size) {
            super(fm);
            mSize = size;
        }

        @Override
        public int getCount() {
            return mSize;
        }

        @Override
        public Fragment getItem(int position) {
            return ImageDetailFragment.newInstance(position);
        }
    }
}
```

fragment implementation.

``` java
public class ImageDetailFragment extends Fragment {
    private static final String IMAGE_DATA_EXTRA = "resId";
    private int mImageNum;
    private ImageView mImageView;

    static ImageDetailFragment newInstance(int imageNum) {
        final ImageDetailFragment f = new ImageDetailFragment();
        final Bundle args = new Bundle();
        args.putInt(IMAGE_DATA_EXTRA, imageNum);
        f.setArguments(args);
        return f;
    }

    // Empty constructor, required as per Fragment docs
    public ImageDetailFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mImageNum = getArguments() != null ? getArguments().getInt(IMAGE_DATA_EXTRA) : -1;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        // image_detail_fragment.xml contains just an ImageView
        final View v = inflater.inflate(R.layout.image_detail_fragment, container, false);
        mImageView = (ImageView) v.findViewById(R.id.imageView);
        return v;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final int resId = ImageDetailActivity.imageResIds[mImageNum];
        mImageView.setImageResource(resId); // Load image into ImageView
    }
}
```
Here, following improvements can be made.

1. Instead of loading images in UI thread, we can use **async task** discussed before.
2. We can **cache** the images.

## GridView Implementation

I'll not copy-paste the example codes here because we are going to talk about the same things we've been talking about again.

1. Load images in the AsyncTask
2. Cache your images if necessary.
3. (+) For greedview and listview, pay attention to **concurrency**


**what is adapter?**

