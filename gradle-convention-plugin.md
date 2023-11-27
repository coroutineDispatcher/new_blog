---
title: "Gradle Convention Plugin"
date: 2022-08-01T19:41:08+02:00
draft: false
---

![](/images/tegernsee.jpeg)

As many of us already know, the Google Android team will soon start using a new way of applying plugins in Android/Kotlin modules. It's more an improvement and a try to centralize the plugin usage, rather than just copying or having duplicate code all around the android/kotlin modules in your project. In case your plugin would have some extra configuration, would be far easier if they would be tackled in a single source of truth rather than looping all modules in the root `build.gradle`file (even though not a real problem), or manually placing them in all modules necessary. This new API solves the second-mentioned problem.

&nbsp;

{{< admonition type=info title="">}}
Currently, an example is present in the [NowInAndroid Github Repository](https://github.com/android/nowinandroid).
{{</ admonition >}}

&nbsp;

### Let's describe the problem with a little bit of code.

Now that compose is introduced, the gradle files would need also some extra configurations, like enabling compose in `buildFeatures` or modifying `kotlinCompilerExtensionVersion`. 
But together with compose, I would also needed a few extra other plugins, which might or might not be related to the primary plugin; let's say: `com.android.application`, `org.jlleitschuh.gradle.ktlint`, `dagger.hilt.android.plugin` and `com.google.firebase.crashlytics`. So we would have:

&nbsp;

```kotlin
plygins {
    id("com.android.application")
    id("org.jlleitschuh.gradle.ktlint")
    id("dagger.hilt.android.plugin")
    id("com.google.firebase.crashlytics")
    kotlin("kapt")
}
```

&nbsp;

And of course, some compose configurations as well.

But what if I can make this a one-liner? Still, the use case might be perfect, as for android/kotlin-jvm libraries we would have to pick the respective ones instead of `com.android.application`, but I'm pretty sure the whole picture can be seen.

&nbsp;

All we would have to do is create a new module, and just code in Kotlin, like we normally do:

```kotlin
class AndroidApplicationComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply("com.android.application")
            pluginManager.apply("org.jlleitschuh.gradle.ktlint")
            pluginManager.apply("dagger.hilt.android.plugin")
            pluginManager.apply("com.google.firebase.crashlytics")
            pluginManager.apply("kotlin-kapt")
            val extension = extensions.getByType<BaseAppModuleExtension>()
            configureAndroidCompose(extension)
        }
    }
}
```

&nbsp;

Where `configureAndroidCompose(extension)` is just a function:

&nbsp;

```kotlin
internal fun Project.configureAndroidCompose(
    commonExtension: CommonExtension<*, *, *, *>,
) {
    val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")

    commonExtension.apply {
        buildFeatures {
            compose = true
        }

        composeOptions {
            kotlinCompilerExtensionVersion = libs.findVersion("composeVersion").get().toString()
        }
    }
}
```

&nbsp;

If you noticed `VersionCatalogsExtension`, I must mention that this plugin conventions are introduced along with "Gradle Version Catalogs", which I would like to tackle in another short blog post.
And that's it for this example. The next step would be to register the plugin, so gradle would know what to do and when:

&nbsp;

```kotlin
gradlePlugin {
    plugins {
        register("androidCompose") {
            id = "com.myNewPlugin.androidCompose"
            implementationClass = "AndroidApplicationComposeConventionPlugin"
        }
    }
}
```

&nbsp;

After that, in the `:app` module (for this example) we would just have to put a one-liner, instead of many:

&nbsp;

```kotlin
plygins {
    id("com.myNewPlugin.androidCompose")
}
```

&nbsp;

After that, just hit the "make (or clean + build) project" button, and then Gradle would hopefully take it from there.

&nbsp;

{{< admonition type=warning title="">}}
The API is still not stable. However, it would not hurt if one would turn their groovy gradle files into kotlin dsl for easier processing.
{{</ admonition >}}

&nbsp;

## Conclusion

The new plugin convention API seems quite nice and easy when having to tackle a lot of configurations for multiple android modules in a single project. Since it is new, I also have a lot to learn and understand from this, but I hope it would be just a very basic and early introduction.