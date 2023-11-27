---
title: "Setting up Gradle with Kotlin DSL, a simple guide"
date: 2019-12-16T09:53:00.000+01:00
draft: false
aliases: ["/2019/12/setting-up-gradle-with-kotlin-dsl.html"]
tags:
  [Kotlin DSL, Gradle, Kotlin Delegation, Kotlin, Groovy, Migration, Android]
author: "Stavro Xhardha"
---

[![](https://static.zerochan.net/Kaiba.Seto.full.2339940.gif)](https://static.zerochan.net/Kaiba.Seto.full.2339940.gif)

Kotlin is a very a pretty nice adoptive language and user friendly. It really replaced Java from my everyday programming. However, it was not enough. We all know that groovy runs on JVM. So, why do I even need a new language just for my builds? Can't it be Java? So Java is the basic language for the JVM, Kotlin runs on JVM, Groovy runs on JVM and my build system has a separated language from my business logic system. Why?

I was introduced to Gradle with Kotlin accidentally. I never heard of Kotlin DSL in terms of Gradle. I just created a new Spring project and the built file looked kind of strange. After a little Google-ing, everything was clear. Long story short, I removed groovy from my Gradle build tool in my Android project, and replaced it with Kotlin. It shouldn't take more than 15 minutes to do this, but you can struggle with some things in particular.

{{< admonition >}}
Kotlin DSL means Kotlin Domain Specific Language. It's just a notation, the name is self descriptive.
{{< /admonition >}}

Here are some things to know:

Start with the simplest thing ever: rename your settings.gradle to settings.gradle.kts. It should have less than 5 lines of code:

```groovy
include ':app'
rootProject.name='AppName'
```

is just going to be:

```kotlin
include(":app")
rootProject.name = "AppName"
```

Than stick with build.gradle project level. It's shorter (or it has repetitive things). So instead of having a build.gradle just rename the file to build.gradle.kts. There are 2 important things to note here (at least in my case).

Global variables.

If you need a variable which is going to be shared across modules and be kept in project level gradle, the

```kotlin
 ext.kotlin_version = '1.3.61'
```

Should just be :

```kotlin
val kotlinVersion by rootProject.extra { "1.3.61" }
```

And then you can access it by doing this:

```kotlin
implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk7:${rootProject.extra.get("kotlinVersion")}")
```

in your app module.

The custom _Clean Task,_ transforms from:

```groovy
task clean(type: Delete) {
   delete rootProject.buildDir
}
```

to:

```kotlin
tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

Don't forget that Kotlin has no idea what ' ' are. Use " " instead. Also, the classpaths, are just becoming simple Kotlin methods with String parameters. So here is my full build.gradle.kts:

```kotlin
buildscript {
    val kotlinVersion by rootProject.extra { "1.3.61" }

    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath("com.android.tools.build:gradle:3.5.3")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

So now let's jump to build.gradle module app. I could just paste the whole file (which I would do below), but there are some things to note. For example variables. I know that I it sounds a little stupid, but I never bothered writing variables in  groovy. It was better with just copy-pasting the version 🤦‍♂️. With Kotlin, it's a little more natural to write variables.

Now the plugins transform:

```groovy
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'
```

into:

```kotlin
plugins {
    id("com.android.application")
    kotlin("android")
    kotlin("android.extensions")
}
```

Note that when you have kotlin specific plugins you can just use kotlin() method, otherwise stick to id(). The defaultConfig becomes from:

```groovy
compileSdkVersion 29
defaultConfig {
 applicationId "com.sxhardha.someappName"
 minSdkVersion 24
 targetSdkVersion 29
 versionCode 1
 versionName "1.0"
 testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
}
```

to:

```kotlin
compileSdkVersion(29)
defaultConfig {
   applicationId = "com.sxhardha.someappName"
   minSdkVersion(24)
   targetSdkVersion(29)
   versionCode = 1
   versionName = "1.0"
   testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
}
```

The `buildTypes` clause has also a new thing. Instead of:

```groovy
buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
```

it becomes:

```kotlin
buildTypes {
        getByName("release") {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
```

A now a tricky little thing, KotlinOptions. You would need a getTask()  method to access it. So this:

```groovy
kotlinOptions {
        jvmTarget = "1.8"
    }
```

becomes:

```kotlin
tasks {
        withType<KotlinCompile> {
            kotlinOptions.jvmTarget = "1.8"
        }
    }
```

The rest is just the same. I'm just pasting the full build.gradle.kts (module app) whole file in case I forgot to mention something. There is also a nice guide [here](https://guides.gradle.org/migrating-build-logic-from-groovy-to-kotlin/) in case you have more unmentioned trouble around.

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    id("com.android.application")
    kotlin("android")
    kotlin("android.extensions")
}

android {
    compileSdkVersion(29)
    defaultConfig {
        applicationId = "com.sxhardha.someappName"
        minSdkVersion(24)
        targetSdkVersion(29)
        versionCode = 1
        versionName = "1.0"
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        getByName("release") {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }

    tasks {
        withType<KotlinCompile> {
            kotlinOptions.jvmTarget = "1.8"
        }
    }

}

dependencies {
    val navigationVersion = "2.1.0"
    val lifecycleVersion = "2.1.0"
    val fragmentVersion = "1.2.0-rc03"
    val espressoVersion = "3.2.0"

    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk7:${rootProject.extra.get("kotlinVersion")}")
    implementation("androidx.appcompat:appcompat:1.1.0")
    implementation("androidx.core:core-ktx:1.1.0")
    implementation("androidx.legacy:legacy-support-v4:1.0.0")
    implementation("com.google.android.material:material:1.0.0")
    implementation("androidx.constraintlayout:constraintlayout:1.1.3")
    implementation("androidx.navigation:navigation-fragment:$navigationVersion")
    implementation("androidx.navigation:navigation-ui:$navigationVersion")
    implementation("androidx.lifecycle:lifecycle-extensions:$lifecycleVersion")
    implementation("androidx.navigation:navigation-fragment-ktx:$navigationVersion")
    implementation("androidx.navigation:navigation-ui-ktx:$lifecycleVersion")
    implementation("com.google.android.material:material:1.2.0-alpha02")
    implementation("androidx.fragment:fragment:$fragmentVersion")
    implementation("androidx.fragment:fragment-ktx:$fragmentVersion")
    //test
    testImplementation("junit:junit:4.12")
    //android tests
    androidTestImplementation("androidx.test.espresso:espresso-core:$espressoVersion")
    androidTestImplementation("androidx.test.espresso:espresso-contrib:$espressoVersion")
    androidTestImplementation("androidx.test.espresso:espresso-intents:$espressoVersion")
    androidTestImplementation("androidx.test.espresso:espresso-accessibility:$espressoVersion")
    androidTestImplementation("androidx.test.espresso:espresso-web:$espressoVersion")
    androidTestImplementation("androidx.test.espresso.idling:idling-concurrent:$espressoVersion")
    androidTestImplementation("androidx.test:runner:1.2.0")
    androidTestImplementation("androidx.test:rules:1.2.0")
    androidTestImplementation("androidx.test.ext:junit:1.1.1")
    androidTestImplementation("org.mockito:mockito-android:2.24.5")
    debugImplementation("com.squareup.leakcanary:leakcanary-android:2.0")
    debugImplementation("androidx.fragment:fragment-testing:$fragmentVersion")
}
```

**Conclusion**  
Unfortunately, I am still unable to tell the difference of the build speed because my project is still small. However, I might share it later on [my twitter](https://twitter.com/suspendfunction).

Stavro Xhardha
