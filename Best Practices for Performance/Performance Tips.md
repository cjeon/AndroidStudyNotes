# [Performance Tips](https://developer.android.com/training/articles/perf-tips.html)
> This document primarily covers micro-optimizations that can improve overall app performance when combined, **but** it's unlikely that these changes will result in dramatic performance effects  

> There are two basic rules for writing efficient code:  
> * Don't do work that you don't need to do.  
> * Don't allocate memory if you can avoid it.

For further reference about java(this document also covers android), see **effective java**.

<!-- TOC -->

- [[Performance Tips](https://developer.android.com/training/articles/perf-tips.html)](#performance-tipshttpsdeveloperandroidcomtrainingarticlesperf-tipshtml)
- [1. Avoid Creating Unnecessary Objects](#1-avoid-creating-unnecessary-objects)
- [2. Prefer Static Over Virtual](#2-prefer-static-over-virtual)
- [3. Use Static Final For Constants](#3-use-static-final-for-constants)
- [4. Avoid Internal Getters/Setters](#4-avoid-internal-getterssetters)
- [5. Use Enhanced For Loop Syntax](#5-use-enhanced-for-loop-syntax)
- [6. Consider Package Instead of Private Access with Private Inner Classes](#6-consider-package-instead-of-private-access-with-private-inner-classes)
- [7. Avoid Using Floating-Point](#7-avoid-using-floating-point)
- [8. Know and Use Libraries.](#8-know-and-use-libraries)
- [9. Use Native Methods Carefully.](#9-use-native-methods-carefully)
- [10. Performance myths](#10-performance-myths)

<!-- /TOC -->

# 1. Avoid Creating Unnecessary Objects  

> Object creation is never free ... allocating memory is always more expensive than not allocating memory.

Examples (Mostly copy-pastes, because I found the original contents are good by themselves without any modifications. I added some explanations in brackets.):

1. If you have a method returning a `String`, and you know that its result will always be appended to a `StringBuffer` anyway, change your signature and implementation so that the function does the append directly, instead of creating a short-lived temporary object.  
[`String` is a object that cannot be modified. Every action modifying a string will internally cast String object to `StringBugger` object. See [this link](http://www.javaworld.com/article/2076072/build-ci-sdlc/stringbuffer-versus-string.html) for help.]

2. When extracting strings from a set of input data, try to return a substring of the original data, instead of creating a copy. You will create a new `String` object, but it will share the char[] with the data. (The trade-off being that if you're only using a small part of the original input, you'll be keeping it all around in memory anyway if you go this route.)

3. An array of `int`s is a much better than an array of `Integer` objects, but this also generalizes to the fact that two parallel arrays of ints are also a lot more efficient than an array of (int,int) objects. The same goes for any combination of primitive types.  
[Two parallel arrays of ints : 2 Arrays, 2n ints (`2 objects and 2n primitives.`), One array of pairs of ints : 1 Array, n Array, 2n ints (`n+1 objects and 2n primitives.`)]

4. If you need to implement a container that stores tuples of (Foo,Bar) objects, try to remember that two parallel Foo[] and Bar[] arrays are generally much better than a single array of custom (Foo,Bar) objects. (The exception to this, of course, is when you're designing an API for other code to access. In those cases, it's usually better to make a small compromise to the speed in order to achieve a good API design. But in your own internal code, you should try and be as efficient as possible.)

> Generally speaking, avoid creating short-term temporary objects if you can. Fewer objects created mean less-frequent garbage collection, which has a direct impact on user experience.

# 2. Prefer Static Over Virtual
> If you don't need to access an object's fields, make your method static. Invocations will be about 15%-20% faster. It's also good practice, because you can tell from the method signature that calling the method can't alter the object's state.  

That is to say, if you have a static method, make it static!

# 3. Use Static Final For Constants
> **Note**: This optimization applies only to primitive types and String constants, not arbitrary reference types. Still, it's good practice to declare constants static final whenever possible.

Use static final for constants. Deep explanation requires lecturn on structure of JAVA and JVM, and I cannot provide them :)

# 4. Avoid Internal Getters/Setters
if you have code like below

``` java
private int count = 10;

private int getCount() { return count; }

private void someMethod () {
    ...
    int myCount = getCount();
    ...
}
```

make it

``` java
private int count = 10;

private void someMethod() {
    ...
    int myCount = count;
    ...
}
```

because 

> Virtual method calls are expensive, much more so than instance field lookups ... Without a JIT, direct field access is about **3x** faster than invoking a trivial getter. With the JIT (where direct field access is as cheap as accessing a local), direct field access is about **7x** faster than invoking a trivial getter.


# 5. Use Enhanced For Loop Syntax
> The enhanced for loop (also sometimes known as "for-each" loop) can be used for collections that implement the Iterable interface and for arrays. With collections, an iterator is allocated to make interface calls to hasNext() and next(). With an ArrayList, a hand-written counted loop is about 3x faster (with or without JIT), but for other collections the enhanced for loop syntax will be exactly equivalent to explicit iterator usage.

``` java
public void zero() {
    int sum = 0;
    for (int i = 0; i < mArray.length; ++i) {
        sum += mArray[i].mSplat;
    }
}

public void one() {
    int sum = 0;
    Foo[] localArray = mArray;
    int len = localArray.length;

    for (int i = 0; i < len; ++i) {
        sum += localArray[i].mSplat;
    }
}

public void two() {
    int sum = 0;
    for (Foo a : mArray) {
        sum += a.mSplat;
    }
}
```

> `zero()` is slowest, because the JIT can't yet optimize away the cost of getting the array length once for every iteration through the loop.

> `one()` is faster. It pulls everything out into local variables, avoiding the lookups. Only the array length offers a performance benefit.

> `two()` is fastest for devices without a JIT, and indistinguishable from one() for devices with a JIT. It uses the enhanced for loop syntax introduced in version 1.5 of the Java programming language.

> Tip: Also see Josh Bloch's Effective Java, item 46.

# 6. Consider Package Instead of Private Access with Private Inner Classes

``` java
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }

    private int mValue;

    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }

    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
```

In order to access `private mValue` and `private doStuff`, compiler generates synthetic codes like below.

``` java
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
```

As we already discussed in #4. 

# 7. Avoid Using Floating-Point

some facts:

1. Floats are 2x slower than integers.
2. Double is 2x bigger (in space) than floats(obviously).
3. Most processors have hardware integer multiply. However, some does not have hardware division.

# 8. Know and Use Libraries.

**Don't Reinvent the Wheel!**

take a loock at `java.lang`, `java.util`, `java.io`.

# 9. Use Native Methods Carefully.

Hard to code, maintain, and reuse. Consider only when significant performance boost is available.

# 10. Performance myths

Invoking methods via a method with exact type rather than interface `is slower`, but `not significantly`. For ex: invoking a method on a `Hashmap map` is faster than invoking a method on a `Map map` (when `map` is actually a `HashMap`), but only about `6%`. 

Also, with JIT, field access costs about the same as local access so you don't have to optimize the field accesses.



