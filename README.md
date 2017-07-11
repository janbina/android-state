# Android-State

A utility library for Android to save objects in a `Bundle` without any boilerplate. It uses an annotation processor to wire up all dependencies.

## Download

Download the latest [library](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22android-state%22) and [processor](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22android-state-processor%22) or grab via Gradle:

```groovy
dependencies {
    compile 'com.evernote:android-state:1.1.2'
    // Java only project
    annotationProcessor 'com.evernote:android-state-processor:1.1.2'

    // Kotlin with or without Java
    kapt 'com.evernote:android-state-processor:1.1.2'
}
```

You can read the [JavaDoc here](https://evernote.github.io/android-state/javadoc/).

## Usage

Annotate any field with `@State` and use the `StateSaver` class to save those fields in a Bundle. This works from an `Activity` or `Fragment`, but also from anywhere else in your code. You can save any type which can be saved in a Bundle like the `String`, `Serializable`, and `Parcelable` data structures.

```java
public class MainActivity extends Activity {

    @State
    public int mValue;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        StateSaver.restoreInstanceState(this, savedInstanceState);
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        StateSaver.saveInstanceState(this, outState);
    }
}
```

## Advanced

You can also save state in a `View` class.

```java
public class TestView extends View {

    @State
    public int mState;

    public TestView(Context context) {
        super(context);
    }

    @Override
    protected Parcelable onSaveInstanceState() {
        return StateSaver.saveInstanceState(this, super.onSaveInstanceState());
    }

    @Override
    protected void onRestoreInstanceState(Parcelable state) {
        super.onRestoreInstanceState(StateSaver.restoreInstanceState(this, state));
    }
}
```

It is recommended that saved properties not be `private`. If a property is `private`, then a non-private getter and setter method are required. This is especially useful for Kotlin, because properties are `private` by default and the aforementioned methods are generated by the compiler.

If you **want** your getter and setter to be used rather than the field value being used directly, the field **must** be private.

```kotlin
class DemoPresenter : Presenter<DemoView>() {

    @State
    var counter = 0

    // ...
}

```

Of course, this also works in Java.

```java
public class TitleUpdater {

    @State
    private String mTitle;

    public String getTitle() {
        return mTitle;
    }

    public void setTitle(String title) {
        mTitle = title;
    }
}
```

If you have a private field and don't want to provide a getter or setter method, then you can fallback to reflection. However, this method is not recommended.

```java
public class ImageProcessor {

    @StateReflection
    private byte[] mImageData;

    // ...
}
```

A custom bundler can be useful, if a class doesn't implement the `Parcelable` or `Serializable` interface, which oftentimes happens with third party dependencies.

```java
public class MappingProvider {

    @State(PairBundler.class)
    public Pair<String, Integer> mMapping;

    public static final class PairBundler implements Bundler<Pair<String, Integer>> {
        @Override
        public void put(@NonNull String key, @NonNull Pair<String, Integer> value, @NonNull Bundle bundle) {
            bundle.putString(key + "first", value.first);
            bundle.putInt(key + "second", value.second);
        }

        @Nullable
        @Override
        public Pair<String, Integer> get(@NonNull String key, @NonNull Bundle bundle) {
            if (bundle.containsKey(key + "first")) {
                return new Pair<>(bundle.getString(key + "first"), bundle.getInt(key + "second"));
            } else {
                return null;
            }
        }
    }
}
```

#### ProGuard

This library comes with a ProGuard config. No further steps are required, but all necessary rules can be found [here](library/proguard.cfg).

## [Icepick](https://github.com/frankiesardo/icepick)

This library is based on [Icepick](https://github.com/frankiesardo/icepick), a great library from Frankie Sardo. However, Icepick is missing some features important to us: it [doesn't support properties](https://github.com/frankiesardo/icepick/issues/81) which is a [bummer for Kotlin](https://github.com/frankiesardo/icepick/issues/47). Also, Icepick does not support private fields which may break encapsulation. A tool shouldn't force you into this direction.

Since Icepick is implemented in Clojure, we decided that it's better for us to rewrite the annotation processor in Java. Unfortunately, that makes it hard to push our features into Icepick itself. That's why we decided to fork the project.

There are also alternatives for Kotlin like [IceKick](https://github.com/tinsukE/icekick). We did not want to use two libraries to solve the same problem for two different languages; we wanted to have one solution for all scenarios.

Upgrading to this library from Icepick is easy. The API is the same; only the packages and the class name (i.e. from `Icepick` to `StateSaver`) have changed. If Icepick works well for you, then there's no need to upgrade.

## License

```
Copyright (c) 2017 Evernote Corporation.
All rights reserved. This program and the accompanying materials
are made available under the terms of the Eclipse Public License v1.0
which accompanies this distribution, and is available at

http://www.eclipse.org/legal/epl-v10.html

Files produced by Android-State code generator are not subject to terms
of the Eclipse Public License 1.0 and can be used as set out in the
copyright notice included in the generated files.
```
