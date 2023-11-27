---
title: "Gradle Version Catalogs"
date: 2022-11-20T15:00:00+02:00
draft: false
---

![](/images/catalogs.png)

As many of you already might know, Gradle has introduced a new (yet unstable) way to centralize dependency declarations. In huge codebases and in multi-module projects, managing gradle dependencies has always been hard. Another problem that previous solutions had, was that no two projects were using gradle declaration or configurations the same. In Android, you would always need some time to (probably) memorize gradle files where dependencies are stored, or remember the arrays of declaration for versions mixed with the library path.

For example, we could do:

&nbsp;

```groovy
// third_party_dependencies.gradle

someArrayOfDependencies = [...]

// my_main_dependencies.gradle
someOtherArrayOfDependencies = [...]
```

&nbsp;

And then for each of them, just apply the versioning from some `versions.gradle`file. One other way, as it is more common in new Kotlin projects, but not so much on Android, is to keep the versioning in the `gradle.properties` file and then apply the properties from there. Bottom line: ***one does not reach much, as much as one refactors or structures***. Dependencies get bigger and bigger and there is always the chance to track stuff anyways. Not to mention forgetting to update the dependencies now and then.

With gradle version catalogs, now there is a way to read and write dependencies in a centralized way, and then have some easier way to access them because of the syntax that this feature introduces. In my example, I am trying version catalogs with gradle `.kts` files, but I would assume it should be the same behavior with normal `groovy` gradle files.

Before we start, we must note that this feature is unstable so it would not make that much sense to jump to production with it (not to mention it might affect build time). I personally started a new side project using version catalogs thus I am not planning to reach production in a year or so. The IDE would notify you anyways. 

First, we enable gradle version catalogs in `settings.gradle.kts`:

&nbsp;

```kotlin
enableFeaturePreview("VERSION_CATALOGS")

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}
```

&nbsp;

Yup, it's a `.toml` file. Its structure makes it more readable (or not? 🙈) and easy for some nice code generation.

Let's see an easy example:

&nbsp;

```toml
[versions]
coroutines = "1.6.4"

[libraries]
coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "coroutines" }
coroutines-testing = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "coroutines" }
```

&nbsp;

First of all, the naming in brackets (`[versions]`, `[libraries]`) is necessary so the compiler knows which element is what. But what I mostly like about this feature, is actually the `-` you notice in the libraries declaration. That `-` makes all the magic once the code has been compiled and is ready to be used in your gradle files.

```kotlin
implementation(libs.coroutines.core)
testImplementation(libs.coroutines.testing)
```

That's basically it. This way it feels really natural and gradle dependencies are more intuitive. To be fair, I would always go from file to file because of not memorizing the variable names correctly in cases I needed more than one which is copied in my cache. Especially when working with gradle, not many people like it, and to me personally, it ruins my focus a lot.
With `.toml`files though, you have to be very careful about the typos for variables etc, even though the compiler was quite good in giving the exact description and problem. Reading the errors that it gave me, I do not remember trying to google the error as it was already easy to understand and self-explanatory.

Gradle convention is a good way for centralizing dependencies for your Android/JVM project. Of course that while working with it, there are a few issues, especially when building the project on a new dependency introduction. However, it looks promising even though in a very early stage.

If you want to know more in detail about gradle version catalogs, check out this [link here](https://docs.gradle.org/current/userguide/platforms.html).
