---
title: "Modularizing your Android app, breaking the monolith (Part 1)"
datePublished: Mon Nov 04 2019 08:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5q16r000508jq3o8w7uiu
slug: modularizing-your-android-app-breaking-the-monolith-part-1

---


  
Inspired by a Martin Fowlers post about [Micro Frontends](https://martinfowler.com/articles/micro-frontends.html), I decided to break my monolithic app into a modular app. I tried to read a little more about breaking monolithic apps in Android, and as far as I got, I felt confident to share my experience with you. This will be some series of blog posts where we actually try to break a simple app into a modularized Android app.  
  
_Note: You should know that I am no expert in this, so if there are false statements or mistakes please feel free to criticize, for the sake of a better development. _  
  
**What do you benefit from this approach:**  

*   Well, people are moving pretty fast nowadays and delivery is required faster and faster. So, in order to achieve this, modularising Android apps is really necessary.
*   You can share features across different apps.
*    Independent teams and less problems per each.
*   Conditional features update.
*   Quicker debugging and fixing.
*   A feature delay doesn't delay the whole app.

As per writing tests, there is not too much difference about being in a monolith or having a modularized Android app. So we will skip tests on this series. Just make sure that each test stands in the right module which corresponds to the chosen feature.  
 Now, there is a small benefit if you are thinking Android specifically:  

*   Significantly reduces APK size. Which means more installs (according to some 🌚)

  
** A small introduction:**  
The app actually is pretty simple. Is built with Architecture Components and has only one Activity. Each fragment has some dependencies but they are Singletons, like database reference, or Picasso and the Retrofit API interface.  
  
So basically, this is my application schema:  
  

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104538281/afd9de15-278c-4533-a76a-4f39d5edb1df.jpeg)](https://1.bp.blogspot.com/-EsLKUeP8Bdk/Xb2Z-XmrmeI/AAAAAAAAQLU/9C7pqG6QoxMns3z1ZDxGQDOIzQYeZ9DEACLcBGAsYHQ/s1600/Multi%2BModule%2BRefactoring%2BDiagram.jpg)

  
All my fragments are pulling dependencies from my Application level. Except from one of them. That fragment is totally unrelated to the other part of the app. It just shows some hardcoded values in the RecyclerView. That really looks like a nice way to start.  
  
_Note: I also have an AlarmManager and 2 WorkManagers both pulling dependencies from the application layer, which will be covered later._  
  
But first things first, let's set up gradle for a multi module project. What I mean is that there is no need to reimplement dependencies over and over again when I add a new Android module for some new feature. Instead, we can just put all of our external libraries and dependencies inside the project level gradle and let all the other modules gradle inherit from it. That would bring better management when library updates occur. A smart thing I found [in this YouTube video](https://www.youtube.com/watch?v=TWLkswxjSr0&t=1902s):  

```groovy
ext {

    compileSDKVersionValue = 29
    minSDKVersionValue = 22
    targetSDKVersionValue = 29

    libraries = [
            cardView              : 'androidx.cardview:cardview:1.0.0',
            androidXLegacySupport : 'androidx.legacy:legacy-support-v4:1.0.0',
            androidXAppCompat     : 'androidx.appcompat:appcompat:1.1.0',
            lifecycleExtension    : 'androidx.lifecycle:lifecycle-extensions:2.1.0',
            viewModelKtx          : 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0-rc01',
            constraintLayout      : 'androidx.constraintlayout:constraintlayout:2.0.0-beta3',
            fragmentNavigation    : 'androidx.navigation:navigation-fragment-ktx:2.2.0-rc01',
            fragmentNavigationKtx : 'androidx.navigation:navigation-ui-ktx:2.2.0-rc01',
            retrofit              : 'com.squareup.retrofit2:retrofit:2.6.1',
            loggingInterceptor    : 'com.squareup.okhttp3:logging-interceptor:4.1.0',
            dagger                : 'com.google.dagger:dagger:2.24',
            coroutinesCore        : 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.2',
            livedataKtx           : 'androidx.lifecycle:lifecycle-livedata-ktx:2.2.0-rc01',
            picasso               : 'com.squareup.picasso:picasso:2.71828',
            materialDialogCore    : 'com.afollestad.material-dialogs:core:2.8.1',
            materialDialogInputs  : 'com.afollestad.material-dialogs:input:3.0.0-rc2',
            room                  : 'androidx.room:room-runtime:2.2.1',
            roomKtx               : 'androidx.room:room-ktx:2.2.1',
            jodaTime              : 'net.danlew:android.joda:2.10.2',
            rocket                : 'com.github.stavro96:Rocket:1.2.0',
            paging                : 'androidx.paging:paging-runtime:2.1.0',
            fragment              : 'androidx.fragment:fragment:1.2.0-rc01',
            fragmentKtx           : 'androidx.fragment:fragment-ktx:1.2.0-rc01',
            workManager           : 'androidx.work:work-runtime-ktx:2.2.0',
            viewModelSavedState   : 'androidx.lifecycle:lifecycle-viewmodel-savedstate:1.0.0-rc01',
            retrofitMoshiConverter: 'com.squareup.retrofit2:converter-moshi:2.4.0',
            moshiCore             : 'com.squareup.moshi:moshi:1.8.0',
            moshiKotlin           : 'com.squareup.moshi:moshi-kotlin:1.6.0',
            smoothie              : 'com.github.stavro96:smoothie:1.0',
            assistedInject        : 'com.squareup.inject:assisted-inject-annotations-dagger2:0.5.0',
            kotlin                : 'org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version',
            coreKtx               : 'androidx.core:core-ktx:1.2.0-beta01',
            airBnBLottie          : 'com.airbnb.android:lottie:3.0.7',
            firebase              : 'com.google.firebase:firebase-core:17.2.1',
            crashLytics           : 'com.crashlytics.sdk.android:crashlytics:2.10.1',
            locationService       : 'com.google.android.gms:play-services-location:17.0.0'
    ]

    annotationProcessors = [
            moshiKapt         : 'com.squareup.moshi:moshi-kotlin-codegen:1.8.0',
            daggerKapt        : 'com.google.dagger:dagger-compiler:2.24',
            assistedInjectKapt: 'com.squareup.inject:assisted-inject-processor-dagger2:0.5.0',
            roomKapt          : 'androidx.room:room-compiler:2.2.1'
    ]

    testImplementations = [
            coroutinesTest        : 'org.jetbrains.kotlinx:kotlinx-coroutines-test:1.3.2',
            jUnit                 : 'junit:junit:4.12',
            mockitoCore           : 'org.mockito:mockito-core:2.28.2',
            jodaTimeTest          : 'joda-time:joda-time:2.10.2',
            mockitoKotlin         : 'com.nhaarman.mockitokotlin2:mockito-kotlin:2.1.0',
            mockitoInline         : 'org.mockito:mockito-inline:2.13.0',
            androidArchCoreTesting: 'android.arch.core:core-testing:1.1.1',
            coroutinesTesting     : 'org.jetbrains.kotlinx:kotlinx-coroutines-test:1.3.2',
            mockWebServer         : 'com.squareup.okhttp3:mockwebserver:4.1.0'
    ]

    androidTestImplementations = [
            espressoCore            : 'androidx.test.espresso:espresso-core:3.2.0',
            espressoContrib         : 'androidx.test.espresso:espresso-contrib:3.2.0',
            espressoIntents         : 'androidx.test.espresso:espresso-intents:3.2.0',
            espressoAccessibility   : 'androidx.test.espresso:espresso-accessibility:3.2.0',
            espressoWeb             : 'androidx.test.espresso:espresso-web:3.2.0',
            espressoIdlingConcurrent: 'androidx.test.espresso.idling:idling-concurrent:3.2.0',
            testRunner              : 'androidx.test:runner:1.2.0',
            testRules               : 'androidx.test:rules:1.2.0',
            androidJUnit            : 'androidx.test.ext:junit:1.1.1',
            fragmentTesting         : 'androidx.fragment:fragment-testing:1.2.0-rc01',
            mockitoAndroid          : 'org.mockito:mockito-android:2.24.5',
            workerTest              : 'androidx.work:work-testing:2.2.0'
    ]

    roomIncrementalAnnotationProcessor = [
            "room.schemaLocation"  : "$projectDir/schemas".toString(),
            "room.incremental"     : "true",
            "room.expandProjection": "true"]
}
```
  
This would keep all your build.gradle files super clean. Now you don't have to require a new dependency and manage the updates because you deal with it only once. And this is how you call them after:  
  
```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.core:core-ktx:1.1.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
    implementation libraries.fragmentNavigationKtx
    implementation libraries.fragmentNavigation
    implementation libraries.viewModelKtx
}
```
  
_Note: You must apply plugin : 'kotlin-kapt' when creating a new module._  
  
Basically, this is all what my new module needs. And now let's break something.  
  
Create a new Android Module and just move all your new features classes over there. Don't forget layouts, strings, dimens, drawable and all resources that are unrelated to other fragments. Get them too. For the current case, there will be no errors because I inherit nothing from any of my app components.  
  
All I have to do now, is just apply this feature to my app/build.gradle level.  
  
_Note: The nav graph should break because moving fragment outside the module would bring to unresolved element, but not to worry, we will fix this right now:_  

```groovy
dependencies {
    ...rest of the dependencies 
    implementation project(':feature_6_module') 
    //the naming is not like that in my original project
}
```
  
Done. The application schema now would look like this:  
  

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104539798/c7accc15-bf3a-4b8a-b450-057db8693dfd.jpeg)](https://1.bp.blogspot.com/-SKJ-0_sgYNU/Xb6p2JsjOZI/AAAAAAAAQL4/A8bjTdwbgnsqqgPhk1QqUqlffFFzYsLfACLcBGAsYHQ/s1600/Multi%2BModule%2BRefactoring%2BDiagram%2B%25281%2529.jpg)

  
Now, my feature\_6\_module is installed as an "external library" to my app module.  
  
**Conclusion**  
  
This was part 1 of breaking a monolithic app into a modularized app in Android. Next part will be all about Dagger and core dependencies.  
  
 Stavro Xhardha
