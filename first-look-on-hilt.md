---
title: 'First look on Hilt'
date: 2020-06-11T21:04:00.001+02:00
draft: false
aliases: [ "/2020/06/first-look-on-hilt.html" ]
tags : [Android]
author: "Stavro Xhardha"
---

  

[![](https://1.bp.blogspot.com/-IBwldAxS5CI/XuJ3mn1rT8I/AAAAAAAAUDI/0Rljva1Df28Xhw2MnosUUWFPs7ng42kdgCK4BGAsYHg/d/raphael-gritschke-9aZzeHjSnCo-unsplash.jpg)](https://1.bp.blogspot.com/-IBwldAxS5CI/XuJ3mn1rT8I/AAAAAAAAUDI/0Rljva1Df28Xhw2MnosUUWFPs7ng42kdgCK4BGAsYHg/s5184/raphael-gritschke-9aZzeHjSnCo-unsplash.jpg)

onCreate
--------

A new Dependency Injection library called [Hilt](https://dagger.dev/hilt/) was presented from the Google team. It was designed on top of Dagger library and provides a simpler, less boilerplate API to handle dependencies in an Android application. As first try, it was a real game changer. Therefore, we will make a short introduction to it, and then discuss about some opinions.

Why was it build?
-----------------

First of all, Dagger was a little hard to start with, especially for beginners. Second, with the deprecation of Dagger-Android, there was a new library needed to solve DI in Android. It is also less boilerplate than Dagger and makes testing simpler.

The problem with Dagger - Android
---------------------------------

Long story short, Android is hard, and `@ContributesAndroidInjector` made things harder. That, in my own humble opinion was a strong reason, for Dagger-Android to be abandoned. Forgetting to add a dependency there and having a runtime crash as well as trying to fix build issues, brought a lot of headache to those who tried using it.

What changes the game with Hilt?
--------------------------------

If I would try to phrase it, I would say that it treats Android classes like they deserve to be treated, **as normal classes**. While in Dagger-Android, Activities, Fragments, WorkManager were classes, but also mysterious objects, which were very scary to work with.

{{< admonition >}}
This post assumes the reader knows Dagger
{{< /admonition >}}

Quick start on Hilt
===================

Let's just start with modules. Since it is built on top of Dagger, there are some things which remain. Let's suppose we have this example:

```kotlin
@Module  
object MyAppScopeDependenciesModule{  
  @Provides  
  @Singleton  
  fun provideDependency1() : Dep1 = Dep1.builder().build()  
   
  @Provides  
  @Singleton  
  fun provideDependency2() : Dep2 = Dep2.builder().build()  
}
```

And let's create a component just for the sake of the example (App level scope):

```kotlin
@Singleton  
@Component(modules = [MyAppScopeDependenciesModule::class])  
interface MyApplicationComponent{  
  val dependency1: Dep1  
  val dependency2: Dep2  
   
  @Component.Factory  
  interface Factory{  
   fun create(application: Application): MyApplicationComponent  
  }  
}
```

And let's hit the Build button and then let's start importing dependencies:

```kotlin
class MyApplication : Application(){  
   
  @Inject lateinit var dep1: Dep1  
   
  override fun onCreate(){  
    super.onCreate()  
   
    DaggerMyApplicationComponent.factory().create(this)  
  }  
}  
```

The relation between scopes and modules was always mysterious by them who never cared to check what the Dagger's annotation processor generated. Therefore, I must say that this was pretty well spotted by those who built Hilt. In Dagger, scoped modules were connected with scoped components by a stand-alone annotation, which provided nearly 0 information if these two (or more modules) were related.

Now, let's try to build the above example with hilt:

```kotlin
@Module  
@InstallIn(ApplicationComponent::class)  
object MyAppScopeDependenciesModule{  
  @Provides  
  fun provideDependency1() : Dep1 = Dep1.builder().build()  
   
  @Provides  
  fun provideDependency2() : Dep2 = Dep2.builder().build()  
}
```

Before you hit Build button, that's all you need with Hilt.

### Where is my component?

Hilt provides the component for you. No need there for creating a component or a scope. Think of components and scopes as they were merged together. And actually, this is why Hilt is a game changer. [Here](https://dagger.dev/hilt/component-hierarchy.svg)is the component hierarchy needed to be used in Android apps, coming from `dagger.hilt.android.components.*`. Basically, you know your dependencies life length, and now you know where to install it. One last step, let's perform Dependency Injection:

```kotlin
@HiltAndroidApp  
class MyApplication : Application(){  
   
  @Inject lateinit var dep1: Dep1  
   
  override fun onCreate(){  
    super.onCreate()  
    ...  
  }  
}
```

Also, if you want to perform DI in Activities, Fragments, Views, Services or Broadcast receivers, there is no need anymore for `AndroidInjection.inject(this)`. Instead just mark them with `@AndroidEntryPoint` at the top of the class:

```kotlin
@AndroidEntryPoint  
class MainActivity : AppCompatActivity() {  
  // either comming from an ActivityComponent or ApplicationComponent  
  @Inject lateinit var dependency: Dependency  
   
  override fun onCreate() {  
    super.onCreate()  
    ...  
  }  
}
```

{{< admonition >}}
The injection happens in `onCreate()`
{{< /admonition >}}

### Is that the best it can do?

Nope, Hilt has also finally solved the problem of `ViewModel`s instantiate process and in general, having runtime and build time dependencies in the constructor at once. Before Hilt, I used to install [AssistedInject](https://github.com/square/AssistedInject)to manage creating `saveStateHandle` properly and there is a [full tutorial](https://www.coroutinedispatcher.com/2019/08/how-to-produce-savedstatehandle-in-your.html) on how to do that. But let's also do something simple as a short presentation:

```kotlin
class MyViewModel @AssistedInject constructor(  
  private val dep1: Dep1,  
  @Assisted private val saveStateHandle: SaveStateHandle  
){  
   
  @AssistedInject.Factory  
  interface Factory{  
    fun create(saveStateHandle: SaveStateHandle) : MyViewModel  
  }  
}
```

And then install a module for it:

```kotlin
@AssistedModule  
@Module(includes = [AssistedInject_ViewModelModule::class])  
abstract class ViewModelModule
```

And then expose the ViewModel in the `AppComponent`:

```kotlin
@Component(...)  
interface AppComponent{  
  ...  
  val vmFactory: MyViewModel.Factory  
  ...  
}
```

And after that I could have a ViewModel happily ever after in my Fragment:

```kotlin
inline fun  Fragment.viewModelFactory(  
    crossinline provider: (SavedStateHandle) -> T  
) = viewModels {  
    object : AbstractSavedStateViewModelFactory(this, fragment.arguments ?: Bundle()) {  
        override fun  create(key: String, modelClass: Class, handle: SavedStateHandle): T =  
            provider(handle) as T  
    }  
}  
   
class HomeFragment : Fragment() {  
   
    private val myViewModel by viewModelFactory { Application.component().vmFactory.create(it) }  
   
    ...  
 }
```

{{< admonition >}}
For more details on this solution check the link provided above.
{{< /admonition >}}

Having to do all this steps for just a ViewModel was painful, not to mention that `AssistedInject` still had a lot to do (Or I could use Koin, but that is not the topic at the moment).

While with Hilt, it is pretty simple:

```kotlin
class EditorViewModel @ViewModelInject constructor(  
    private val playgroundRepository: PlaygroundRepository,  
    @Assisted private val savedStateHandle: SavedStateHandle  
) : ViewModel() {}
```

After that, nothing else is needed. Just import the ViewModel as you normally do:

```kotlin
//inside Fragment  
private val editorViewModel by viewModels()
```

{{< admonition >}}
Please check Hilt's documentation for correct dependencies to import. The `ViewModel` and `WorkManager` solution come as separated dependencies. For more, check [here](https://developer.android.com/training/dependency-injection/hilt-jetpack)
{{< /admonition >}}

onDestroyView
-------------

Personally, I would be a very happy developer by using Hilt as a Dependency Injection tool for Android. It makes it easy to track dependencies, easy to start with and less boilerplate than Dagger.

However, one of the cons I noticed when using Hilt, was that it adds even more abstraction over your project and you either need to know a little more about code generation or better not use it by heart. Also, forgetting to perform DI as `AndroidInjection.inject(this)` or `DaggerMComponent.builder().build().inject(this)` and annotating the class with `@AndroidEntryPoint`is still tricky, you forget either way. But there is no problem with that since the error would be generated at build time.

Nevertheless, it looks very promising.

onDestroy
---------

I hope I gave a short introduction to get started with Hilt and also some opinions on it. Android is not a simple framework/library to work with and having more and more configurations for every tool that you need to use is always a huge headache. Therefore, I am very glad that Google team introduced Hilt. And for all Dagger fans here, Hilt is a strong argument against all who complain about Daggers complexity.

Stavro Xhardha