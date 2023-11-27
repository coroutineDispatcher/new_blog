---
title: "Using Idling Resources in your Espresso Tests"
datePublished: Wed Jun 19 2019 08:44:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5rq83000409jsh7ixf8c2
slug: using-idling-resources-in-your-espresso-tests

---


Using Idling Resources in your Espresso Tests \* { font-family: Georgia, Cambria, "Times New Roman", Times, serif; } html, body { margin: 0; padding: 0; } h1 { font-size: 50px; margin-bottom: 17px; color: #333; } h2 { font-size: 24px; line-height: 1.6; margin: 30px 0 0 0; margin-bottom: 18px; margin-top: 33px; color: #333; } h3 { font-size: 30px; margin: 10px 0 20px 0; color: #333; } header { width: 640px; margin: auto; } section { width: 640px; margin: auto; } section p { margin-bottom: 27px; font-size: 20px; line-height: 1.6; color: #333; } section img { max-width: 640px; } footer { padding: 0 20px; margin: 50px 0; text-align: center; font-size: 12px; } .aspectRatioPlaceholder { max-width: auto !important; max-height: auto !important; } .aspectRatioPlaceholder-fill { padding-bottom: 0 !important; } header, section\[data-field=subtitle\], section\[data-field=description\] { display: none; }

Testing is wow. And UI testing in Android is awesome. Espresso is a UI testing framework for Android. I can say that it was pretty easy and…

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104616747/3fc4da6a-2bb1-4eca-ad7e-c5becf402124.jpeg)

Testing is wow. And UI testing in Android is awesome. [Espresso](https://developer.android.com/training/testing/espresso) is a UI testing framework for Android. I can say that it was pretty easy and you can get used to it in a couple of hours. It’s just like structured English, but in Java/Kotlin.

A simple example from the _documentation_ when using the Espresso framework is this:

```kotlin
@Test
fun greeterSaysHello() {
    onView(withId(R.id.name\_field)).perform(typeText("Steve"))
    onView(withId(R.id.greet\_button)).perform(click())
    onView(withText("Hello Steve!")).check(matches(isDisplayed()))
}
```

> Note: These tests run under the `androidtest` package.

But sometimes (or mostly) we are not working on the main thread. We have lots of code that blocks our UI and force us to run on another thread. Since Espresso test is quick and straightforward it doesn’t really wait until our background execution has started, is running, is being finished or has already finished. Therefore, we might need a extra hand…

**Introducing to the Idling Resources:**

**Dependency:**

From the documentation of the code the Idling Resources are defined like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104618312/acabead9-a792-4089-83fc-230f270e96b8.png)

Now `Espresso` knows how to wait until some heavy work is executing.

Let’s see an example:

First we need a setup for that Idling Resources. Create a class and implement the \`IdlingResource\` interface:

Basically we are going to tell Espresso to hold when you see an incremented value here, and continue if this value has reached 0

For making things easy create this helper class:

Let’s use it now:

```kotlin
fun onHeavyWorkButtonClicked() {
    **EspressoIdlingResource.increment()**
    _viewModelScope_._launch_(**Dispatchers.IO**) {  //heavy work happens here
        _withContext_(**Dispatchers.Main**) { **EspressoIdlingResource.decrement()**
            //heavy work has ended
        }
    }}
```

Just another simple step. Now we need to tell the UI test that we are using the `IdlingResource` on the current view:

And we are good to go. Just test the views as usual! Good luck! 👍👌✔

> Note: I’ve also refactored this case in a simple Android package:

[**stavro96/smoothie**  
\_Simple way to handle the Idling Resource in your Espresso tests - stavro96/smoothie_github.com](https://github.com/stavro96/smoothie "https://github.com/stavro96/smoothie")[](https://github.com/stavro96/smoothie)

**Super helpful resource:**

[**Android testing with Espresso’s Idling Resources and testing fidelity**  
\_Synchronizing your app with Espresso_medium.com](https://medium.com/androiddevelopers/android-testing-with-espressos-idling-resources-and-testing-fidelity-8b8647ed57f4 "https://medium.com/androiddevelopers/android-testing-with-espressos-idling-resources-and-testing-fidelity-8b8647ed57f4")[](https://medium.com/androiddevelopers/android-testing-with-espressos-idling-resources-and-testing-fidelity-8b8647ed57f4)

**Other posts from my blog:**

[**Annotation processor: Say less, mean more.**  
\_I’ve always been curious on what is behind an annotation. As much as they made my angry, believe me they are so fun…\_proandroiddev.com](https://proandroiddev.com/annotation-processor-say-less-mean-more-b0e23dd9a3e2 "https://proandroiddev.com/annotation-processor-say-less-mean-more-b0e23dd9a3e2")[](https://proandroiddev.com/annotation-processor-say-less-mean-more-b0e23dd9a3e2)

[**Retrofit met Coroutines**  
\_Finally the latest version of Retrofit (2.6.0) has got out. While it was already really easy to use and so much fun…\_proandroiddev.com](https://proandroiddev.com/retrofit-met-coroutines-7bbe7e86825a "https://proandroiddev.com/retrofit-met-coroutines-7bbe7e86825a")[](https://proandroiddev.com/retrofit-met-coroutines-7bbe7e86825a)

[**Usage of the ViewModelScope**  
\_Based on my last blog post about easy implementation on Kotlin Coroutines,we were also introduced with the…\_proandroiddev.com](https://proandroiddev.com/usage-of-the-viewmodelscope-f28703467b31 "https://proandroiddev.com/usage-of-the-viewmodelscope-f28703467b31")[](https://proandroiddev.com/usage-of-the-viewmodelscope-f28703467b31)

By [Stavro Xhardha](https://medium.com/@stavro96) on [June 18, 2019](https://medium.com/p/66043182ca63).

[Canonical link](https://medium.com/@stavro96/using-idling-resources-in-your-espresso-tests-66043182ca63)
