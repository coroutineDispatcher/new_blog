---
title: 'Generate Kotlin Docs using Dokka'
date: 2020-02-17T15:48:00.000+01:00
draft: false
aliases: [ "/2020/02/generate-kotlin-docs-using-dokka.html" ]
tags : [Dokka, Kotlin DSL, Kotlin Documentation, Gradle, Kotlin, Groovy, KDoc, Android, Java Doc]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-CiTF74zHoNI/XkqivknEhQI/AAAAAAAARnA/sIn3D1zxoFglJf-uz6djW4Ry2zhQ5z-oACLcBGAsYHQ/s1600/sergiu-valena-_Drvb_c_72Y-unsplash.jpg)](https://1.bp.blogspot.com/-CiTF74zHoNI/XkqivknEhQI/AAAAAAAARnA/sIn3D1zxoFglJf-uz6djW4Ry2zhQ5z-oACLcBGAsYHQ/s1600/sergiu-valena-_Drvb_c_72Y-unsplash.jpg)

  

Have you ever generated Kotlin docs (Kdocs) for your library/project? I have. There is a tool for this called Dokka and you can find it [here](https://github.com/Kotlin/dokka). It's not too hard to set up.

I personally used Dokka for a [small API](https://github.com/coroutineDispatcher/rocket) i wrote for `SharedPreferences`. Anyways, the steps are pretty basic. One thing you must be careful though, is to know the [syntax of the Kdocs](https://kotlinlang.org/docs/reference/kotlin-doc.html) pretty well (usually, if you know how to generate Javadoc, Kotlin docs don't have much difference).

Let's take a simple example:

```kotlin
/** Reads a String from SharedPreferences  
* @param [key] the key provided to find the stored value  
* @return [String] the data of type String if found if not returns an empty String  
* @throws [java.lang.ClassCastException] if the key is found but is not a String  
*/  
@Throws(java.lang.ClassCastException::class)  
fun readString(key: String, defaultStringValue: String = ""): String =  
sharedPreferences.getString(key, defaultStringValue)  
?: throw java.lang.ClassCastException("The key exists, but its' value not of type String")  

```

All the commented lines will generate later what we call, the Kotlin Docs/Kdocs (or you can yell at people: "Read the bloody docs" 🤣). Being careful to describe exactly what the function does and check all his components, parameters, return values, exceptions, is the key to generating clear documentation for your project.

So, let's set up Dokka first.

Go to `build.gradle` (project level) and add this line:

```kotlin
classpath "org.jetbrains.dokka:dokka-gradle-plugin:0.9.18" //or later version
```

Than, just apply the plugin in the `build.gradle` (level module):

```groovy
apply plugin: 'org.jetbrains.dokka'  
...  
android {  
   ...  
  
    dokka {  
        outputFormat = 'html'  
  
        outputDirectory = "$buildDir/javadoc"  
    }  
}  

```

All is now set up. As you may have notice the format will be html. Feel free to check the docs for other formats (never used any other).

So, let's suppose you have written some comments with the purpose of generating the documentation. After that, just type:

```
./gradlew dokka
```

Wait for some seconds and there you would see some success message, or if you have done something wrong, the CLI will notify you.

If you have successfully generated the docs, you will find the files on the build folder. After that, it's up to you in where to host them (I use github pages). The CSS of the docs are pretty nice and simple. But you can modify that if you want.

[Here](https://coroutinedispatcher.github.io/rocket/) is an example you can see about Kotlin docs.

Stavro Xhardha