# Building an OpenGL ES Environment
What is OpenGL ES?
>OpenGL for Embedded Systems (OpenGL ES or GLES) is a subset of the OpenGL computer graphics rendering application programming interface (API) for rendering 2D and 3D computer graphics such as those used by video games, typically hardware-accelerated using a graphics processing unit (GPU).  -Wikipedia

In shorter, rougher words, it is library that we can use to do graphical things. There are quite a few ways to make use of OpenGL.

1. Implement `GLSurfaceView` and `GLSurfaceView.Renderer`. (For full screen)
2. Implement `TextureView`. (For smaller graphics)
3. Build things on `SurfaceView`. (Recommended `for real, do-it-yourself developers` and `requires writing quite a bit of additional code`, according to [official doc](https://developer.android.com/training/graphics/opengl/environment.html))

## Declare OpenGL ES Use in the Manifest

First, we need to declare in `Android.manifest` that we are going to use OpenGL 2.0

``` 
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
```
additionally if we are going to use texture compression,
```
<supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
<supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />
```

## Create an Activity for OpenGL ES Graphics

With activity that uses `OpenGL`, there is an additional component that we can add to our layout. `GLSurfaceView`.
Following is a minimal activity which sets `GLSurfaceView` as ContentView.

``` java
public class OpenGLES20Activity extends Activity {

    private GLSurfaceView mGLView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Create a GLSurfaceView instance and set it
        // as the ContentView for this Activity.
        mGLView = new MyGLSurfaceView(this);
        setContentView(mGLView);
    }
}
```

## Build a GLSurfaceView Object

Below is a minimal implementation of `GLSurfaceView`. It does

1. set OpenGL ES version
2. set GLRenderer.

``` java
class MyGLSurfaceView extends GLSurfaceView {

    private final MyGLRenderer mRenderer;

    public MyGLSurfaceView(Context context){
        super(context);

        // Create an OpenGL ES 2.0 context
        setEGLContextClientVersion(2);

        mRenderer = new MyGLRenderer();

        // Set the Renderer for drawing on the GLSurfaceView
        setRenderer(mRenderer);
    }
}
```
`MyGLRenderer` is defined below.

Optionally, if we are sure that we can take control of `onDrawFrame()`, we save some memory by setting `setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY)`.  


## Build a Renderer Class

GLSurfaceView alone cannot do much things. We must implement renderer to do some real jobs.  
We must implement the below three methods.
* `onSurfaceCreated()` - Called once to set up the view's OpenGL ES environment.
* `onDrawFrame()` - Called for each redraw of the view.
* `onSurfaceChanged()` - Called if the geometry of the view changes, for example when the device's screen orientation changes.

Below are lines of codes which does... *draw black background*.

``` java
public class MyGLRenderer implements GLSurfaceView.Renderer {

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }

    public void onDrawFrame(GL10 unused) {
        // Redraw background color
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
    }

    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }
}
```

Of course, we will do things more interesting further.

# Defining Shapes
We are going to start from shapes. Like triangles, squares etc., but they are more interesting than *black background*, I assume.  
We are going to define

1. Triangle
2. Square

Notes:

1. Coordinate system: It's three dimensional, and center of GLSurfaceView is (0,0,0). Top right is (1,1,0), bottom left is (-1,-1,0).
2. Shapes are drawn in order of declaration. What is defined first is drawn first.

## Triangle 


``` java
public class Triangle {

    private FloatBuffer vertexBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float triangleCoords[] = {   // in counterclockwise order:
             0.0f,  0.622008459f, 0.0f, // top
            -0.5f, -0.311004243f, 0.0f, // bottom left
             0.5f, -0.311004243f, 0.0f  // bottom right
    };

    // Set color with red, green, blue and alpha (opacity) values
    float color[] = { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };

    public Triangle() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
                // (number of coordinate values * 4 bytes per float)
                triangleCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());

        // create a floating point buffer from the ByteBuffer
        vertexBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        vertexBuffer.put(triangleCoords);
        // set the buffer to read the first coordinate
        vertexBuffer.position(0);
    }
}
```

## Square


``` java
public class Square {

    private FloatBuffer vertexBuffer;
    private ShortBuffer drawListBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float squareCoords[] = {
            -0.5f,  0.5f, 0.0f,   // top left
            -0.5f, -0.5f, 0.0f,   // bottom left
             0.5f, -0.5f, 0.0f,   // bottom right
             0.5f,  0.5f, 0.0f }; // top right

    private short drawOrder[] = { 0, 1, 2, 0, 2, 3 }; // order to draw vertices

    public Square() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 4 bytes per float)
                squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
        vertexBuffer.position(0);

        // initialize byte buffer for the draw list
        ByteBuffer dlb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 2 bytes per short)
                drawOrder.length * 2);
        dlb.order(ByteOrder.nativeOrder());
        drawListBuffer = dlb.asShortBuffer();
        drawListBuffer.put(drawOrder);
        drawListBuffer.position(0);
    }
}
```

# Drawing Shapes

## Initialize Shapes

We can Initialize them anywhere, but most commonly it is wise to initialize them in `onSurfaceCreated`.

``` java
public class MyGLRenderer implements GLSurfaceView.Renderer {

    ...
    private Triangle mTriangle;
    private Square   mSquare;

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        ...

        // initialize a triangle
        mTriangle = new Triangle();
        // initialize a square
        mSquare = new Square();
    }
    ...
}
```

## Draw a Shape

>Drawing a defined shape using OpenGL ES 2.0 requires a significant amount of code, because you must provide a lot of details to the graphics rendering pipeline

We need to implement followings:

1. `Vertex Shader` - OpenGL ES graphics code for rendering the vertices of a shape.
2. `Fragment Shader` - OpenGL ES code for rendering the face of a shape with colors or textures.
3. `Program` - An OpenGL ES object that contains the shaders you want to use for drawing one or more shapes.

### Shaders

We need to:

1. Write them in `OpenGLShadingLanguage(GLSL)`.
2. Compile them.
3. Added to `Program`.

Basic Shader code.

``` java
public class Triangle {

    private final String vertexShaderCode =
        "attribute vec4 vPosition;" +
        "void main() {" +
        "  gl_Position = vPosition;" +
        "}";

    private final String fragmentShaderCode =
        "precision mediump float;" +
        "uniform vec4 vColor;" +
        "void main() {" +
        "  gl_FragColor = vColor;" +
        "}";

    ...
}
```
If you want to know more about GLSL, please refer to [official doc](https://www.opengl.org/sdk/docs/).

### Compiling.


We can compile this code by creating a utility method in render class.

``` java
public static int loadShader(int type, String shaderCode){

    // create a vertex shader type (GLES20.GL_VERTEX_SHADER)
    // or a fragment shader type (GLES20.GL_FRAGMENT_SHADER)
    int shader = GLES20.glCreateShader(type);

    // add the source code to the shader and compile it
    GLES20.glShaderSource(shader, shaderCode);
    GLES20.glCompileShader(shader);

    return shader;
}
```
Note: Make sure that you do not compile the shader more than once. Save it somewhere!

### Loading onto program.


``` java
public class Triangle() {
    ...

    private final int mProgram;

    public Triangle() {
        ...

        int vertexShader = MyGLRenderer.loadShader(GLES20.GL_VERTEX_SHADER,
                                        vertexShaderCode);
        int fragmentShader = MyGLRenderer.loadShader(GLES20.GL_FRAGMENT_SHADER,
                                        fragmentShaderCode);

        // create empty OpenGL ES Program
        mProgram = GLES20.glCreateProgram();

        // add the vertex shader to program
        GLES20.glAttachShader(mProgram, vertexShader);

        // add the fragment shader to program
        GLES20.glAttachShader(mProgram, fragmentShader);

        // creates OpenGL ES program executables
        GLES20.glLinkProgram(mProgram);
    }
}
```
Now, we have done

1. Defining shaders, 
2. Compling shaders,
3. Loading shaders onto program.  

It means we can now **draw** them.

### Drawing


``` java
private int mPositionHandle;
private int mColorHandle;

private final int vertexCount = triangleCoords.length / COORDS_PER_VERTEX;
private final int vertexStride = COORDS_PER_VERTEX * 4; // 4 bytes per vertex

public void draw() {
    // Add program to OpenGL ES environment
    GLES20.glUseProgram(mProgram);

    // get handle to vertex shader's vPosition member
    mPositionHandle = GLES20.glGetAttribLocation(mProgram, "vPosition");

    // Enable a handle to the triangle vertices
    GLES20.glEnableVertexAttribArray(mPositionHandle);

    // Prepare the triangle coordinate data
    GLES20.glVertexAttribPointer(mPositionHandle, COORDS_PER_VERTEX,
                                 GLES20.GL_FLOAT, false,
                                 vertexStride, vertexBuffer);

    // get handle to fragment shader's vColor member
    mColorHandle = GLES20.glGetUniformLocation(mProgram, "vColor");

    // Set color for drawing the triangle
    GLES20.glUniform4fv(mColorHandle, 1, color, 0);

    // Draw the triangle
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);

    // Disable vertex array
    GLES20.glDisableVertexAttribArray(mPositionHandle);
}
```

then, we can now implement

``` java
public void onDrawFrame(GL10 unused) {
    ...

    mTriangle.draw();
}
```

to draw triangles.

# Applying Projection and Camera Views
Definitions:
>Projection: adjusts the coordinates of drawn objects based on the width and height of the GLSurfaceView where they are displayed. Without this calculation, objects drawn by OpenGL ES are skewed. 

>Camera View: adjusts the coordinates of drawn objects based on a virtual camera position.

As of my understanding, Projection: modify objects, Camera View: modify eye i.e., camera.  
If we want to change a color of an object, we apply projection. If we want to change size of a object i.e., we get close to the object and want object to look bigger, we change Camera view. (Please fix me if I'm wrong.)

To find out more about transformations, refer to [this blog article](http://www.songho.ca/opengl/gl_transform.html).  
(Note: **lots** of matrices!)

[Official doc](https://developer.android.com/training/graphics/opengl/projection.html#camera-view) says we should apply camera view transformation after manually defined projections.

Here is an example of a transformation.

``` java
// mMVPMatrix is an abbreviation for "Model View Projection Matrix"
private final float[] mMVPMatrix = new float[16];
private final float[] mProjectionMatrix = new float[16];
private final float[] mViewMatrix = new float[16];

@Override
public void onSurfaceChanged(GL10 unused, int width, int height) {
    GLES20.glViewport(0, 0, width, height);

    float ratio = (float) width / height;

    // this projection matrix is applied to object coordinates
    // in the onDrawFrame() method
    Matrix.frustumM(mProjectionMatrix, 0, -ratio, ratio, -1, 1, 3, 7);
}
```
This is a projection which calculates width-to-height ratio. As its name suggests, it may be used during screen orientation changes.

We can also create a camera view.

``` java
@Override
public void onDrawFrame(GL10 unused) {
    ...
    // Set the camera position (View matrix)
    Matrix.setLookAtM(mViewMatrix, 0, 0, 0, -3, 0f, 0f, 0f, 0f, 1.0f, 0.0f);

    // Calculate the projection and view transformation
    Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mViewMatrix, 0);

    // Draw shape
    mTriangle.draw(mMVPMatrix);
}
```

Here, `setLookAtM`'s parameters are as follows:

```
rm	float: returns the result
rmOffset  int: index into rm where the result matrix starts
eyeX	float: eye point X
eyeY	float: eye point Y
eyeZ	float: eye point Z
centerX	float: center of view X
centerY	float: center of view Y
centerZ	float: center of view Z
upX	float: up vector X
upY	float: up vector Y
upZ	float: up vector Z
```
eye: position of camera
center: where to look at
up: upward (sky, if eye is standing right up. Needed for vector calculations)


Step 1. setLookAtM.
Step 2. multiply it with previous projection.
step 3. draw!

### Apply Projection and Camera Transformations


``` java
public class Triangle {

    private final String vertexShaderCode =
        // This matrix member variable provides a hook to manipulate
        // the coordinates of the objects that use this vertex shader
        "uniform mat4 uMVPMatrix;" +
        "attribute vec4 vPosition;" +
        "void main() {" +
        // the matrix must be included as a modifier of gl_Position
        // Note that the uMVPMatrix factor *must be first* in order
        // for the matrix multiplication product to be correct.
        "  gl_Position = uMVPMatrix * vPosition;" +
        "}";

    // Use to access and set the view transformation
    private int mMVPMatrixHandle;

    ...
}
```

then we can draw **triangle** on the SurfaceView!

```
public void draw(float[] mvpMatrix) { // pass in the calculated transformation matrix
    ...

    // get handle to shape's transformation matrix
    mMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");

    // Pass the projection and view transformation to the shader
    GLES20.glUniformMatrix4fv(mMVPMatrixHandle, 1, false, mvpMatrix, 0);

    // Draw the triangle
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);

    // Disable vertex array
    GLES20.glDisableVertexAttribArray(mPositionHandle);
}
```

# Adding Motion
In fact, using the openGL only to draw triangles and squares is not a wise usage of it. We can draw a triangle just by defining custom drawable or canvas. However, openGL can do things other Android graphic frameworks cannot do, or do in complicated way.  
Fully customizable motion is one of them.

## Rotate a Shape
We can rotate a shape by setting `setRotateM` of `Matrix`, like below.

``` java
private float[] mRotationMatrix = new float[16];
public void onDrawFrame(GL10 gl) {
    float[] scratch = new float[16];

    ...

    // Create a rotation transformation for the triangle
    long time = SystemClock.uptimeMillis() % 4000L;
    float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, angle, 0, 0, -1.0f);

    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);

    // Draw triangle
    mTriangle.draw(scratch);
}
```
Then each time `onDrawFrame` is called, `time` will change, which will lead to change of `RotateM`, which will result in rotating object.

# Responding to Touch Events
## Setup a Touch Listener
We can responde to touch event by implementing `onTouchEvent` in `GLSurfaceView` class.  
Below code changes the rotation of triangle according to touch event.

``` java
private final float TOUCH_SCALE_FACTOR = 180.0f / 320;
private float mPreviousX;
private float mPreviousY;

@Override
public boolean onTouchEvent(MotionEvent e) {
    // MotionEvent reports input details from the touch screen
    // and other input controls. In this case, you are only
    // interested in events where the touch position changed.

    float x = e.getX();
    float y = e.getY();

    switch (e.getAction()) {
        case MotionEvent.ACTION_MOVE:

            float dx = x - mPreviousX;
            float dy = y - mPreviousY;

            // reverse direction of rotation above the mid-line
            if (y > getHeight() / 2) {
              dx = dx * -1 ;
            }

            // reverse direction of rotation to left of the mid-line
            if (x < getWidth() / 2) {
              dy = dy * -1 ;
            }

            mRenderer.setAngle(
                    mRenderer.getAngle() +
                    ((dx + dy) * TOUCH_SCALE_FACTOR));
            requestRender();
    }

    mPreviousX = x;
    mPreviousY = y;
    return true;
}
```
## Expose the Rotation Angle
Since we are accessing `mRenderer`'s angle via `setAngle` and `getAngle`, we need to have a way to access it.  
And because render is running on separate thread, we need to make sure the angle is saved and read from memory only, we need to declare this as `volatile`.

``` java
public class MyGLRenderer implements GLSurfaceView.Renderer {
    ...

    public volatile float mAngle;

    public float getAngle() {
        return mAngle;
    }

    public void setAngle(float angle) {
        mAngle = angle;
    }
}
```

## Apply Rotation
Now we can apply rotation by overriding previous `onDrawFrame`.

``` java
public void onDrawFrame(GL10 gl) {
    ...
    float[] scratch = new float[16];

    // Create a rotation for the triangle
    // long time = SystemClock.uptimeMillis() % 4000L;
    // float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, mAngle, 0, 0, -1.0f);

    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);

    // Draw triangle
    mTriangle.draw(scratch);
}
```
