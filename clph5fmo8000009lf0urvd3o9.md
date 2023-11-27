---
title: "Are service locators in Android actually simple?"
datePublished: Mon Dec 02 2019 08:06:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5fmo8000009lf0urvd3o9
slug: are-service-locators-in-android-actually-simple

---


  
This week, I've been playing with service locators in Android. I made [this](https://github.com/coroutineDispatcher/service_locator_demo) repo with 3 different branches. One of them is using Dagger as a DI tool, the other branches are implementation with KodeIn and Koin.  
  
The app is pretty simple, just retrieves some data from a Cat API, saves them to a local Room persistence and then renders them to the screen:  
  
[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104053822/f2fc30f7-b75d-4bbd-a22f-6be6ff12f2d4.jpeg)](https://1.bp.blogspot.com/-QU4jCWTjQCo/XeI9eqyD9qI/AAAAAAAAQjI/IKjH3T1I3VcrOen2PeMKVhXtsWhPc8K_wCLcBGAsYHQ/s1600/ServiceLocator%2BDiagram.jpg)

  
The screen renders just a the list of cats retrieved by the database after the data are fetched from a remote API. So let us list our dependencies:  
  
1 - First I'm using Retrofit, to fetch the data (and the interface which implements the server request methods)  
2- Second I need a database (I always use Room, but in my early beginning of Android development, I remember using Realm and it was awesome too)  
3- Since I'm using coroutines, I always keep a data class with coroutine dispatchers to be used in the ViewModel.  
4- The famous ViewModelFactory (for simplicity I'm not handling the savedStateHandle in this article)  
5-  Picasso, to render the images.  
  
I'm comparing these  tools in verbosity, simplicity and time to set up.  
  
## [Koin](https://insert-koin.io/) 
Koin as a service locator is the easiest to get in with. The docs are pretty simple and it provides a lot of features, making it the perfect tool for Android in particular. However, it has some things I don't really like. Something I like to call the _pre-declaration of the constructor_. Notice this code here:  
  
```kotlin
val myModule = module{  
      ...  
      single {  
            CoroutineDispatchers(  
                Dispatchers.Main,  
                Dispatchers.IO  
            )  
        }  
  
        factory {  
            EntranceRepository(get(), get())  
        }  
  
        viewModel {  
            EntranceFragmentViewModel(get(), get())  
        }  
      ...  
    }
```

These lines are what I love and what I hate about Koin. First of all, I really love that it provides it's own feature about the ViewModel factory. But it has some perks, IMO:  
  
1- First of all when it comes to naming, if you think about it, one keyword is single (so it's a scope), and the other is a factory (doesn't give a message about the scope, but it's a factory), and the other is a viewmodel (definitely not a scope). This makes it a little bit chaotic, mixing the concept of scopes with factory patterns.  
  
2- The second thing is declaration of the EntranceRepository(get(), get()) which is basically definition of a constructor. This is boilerplate IMO. And correct me if I'm wrong, but I assume it uses some kind of reflection (in which I don't know much), in order to invoke constructor in runtime. If you don't provide the EntranceRepository like that, you are most likely to break the app in runtime.  
  
_I'm not handling the part of field injection, because it looks the same in all DI tools. The constructor injection in Android is a little bit more tricky than the other parts._  
  
## [KodeIn](https://kodein.org/Kodein-DI/) 
  
The same thing applies to KodeIn when it comes to speed of set up. It's fantastic. Furthermore, the lazy initialisation of dependencies looks really great to me.  
  
1- However, this is what I found suspicious in KodeIn [docs](https://kodein.org/Kodein-DI/?6.5/android#_view_models):  
  
 _ To use Kodein, you need an Android context. For that, View Models need to implement AndroidViewModel._  
I'm pretty sure not many would agree to do that, and [here](https://medium.com/androiddevelopers/locale-changes-and-the-androidviewmodel-antipattern-84eb677660d9) are some good reasons about it.  
  
Otherwise, you would need to keep your ViewModelFactory pattern, and KodeIn hasn't a pre-built feature for that, you have to do it manually. The pre declaration of constructor problem, does apply for KodeIn too.  
  
2- Another thing I noticed in KodeIn field injection, was the by kodein() delegation which does create a little bit of confusion because the kodein() method is a different extension method for Activities, Fragments etc.  
  
**My own idea of a service locator in Android:**  
Must note that this is only an idea and it's totally immature, but I find compile time safer than runtime. Therefore, annotations are perfectly solving a case of DI in the JVM world. If we forget Dagger for a moment, the next best example would be Spring framework with its' @Autowire injection keyword.  
Anyways, there could be library which could be a mix between Dagger and other service locators. I still imagine something like this:  
  
```kotlin
@Holder  
class MyApplication: Application(){  
   
  override fun onCreate(){  
   ..  
   MyApplicationHolder.init(this) //after compilation  
  }  
    
  @Single  
  fun provideDependency1() : Dependency1{  
    return Dependency1.builder().setItsThings(things).build()   
  }  
    
  @SomeScoped  
  fun provideDependency3(): Dependency3{  
    return Dependency3.builder().setItsThings3(things3).build()  
  }  
    
  @Single  
  @ByFactory  
  fun provideViewModel() : MyViewModel{  
    return MyViewModel(dependency1, dependency2)  
  }  
}  
  
@Single //which would be just a @Inject + @Singleton equivalent  
class Dependency2(){  
}  
  
//ViewModels  
  
@Single //A ViewModel factory would be generated for you by the "framework"  
class MyViewModel(d1: D1, d2: D2) : ViewModel(){  
    
}
```

This is far from explained or implemented, and I am still learning about code generation, but a Dagger without components would be awesome as a service locator (hopefully this is not Dagger1, because I've never seen it 😅) .  
  
**Conclusion:**  
Using KodeIn or Koin in your codebase is pretty reasonable thing to do, if it solves your problem. But this article was more about to state that _they are not as simple as developer state_ they are. Furthermore, I still think that Dagger takes a little more time to implement (and hopefully that's the hardest thing in Dagger, trust me) , but the generation of boilerplate comes from the framework and giving less work to the developer.
  
If you want to know more about service locators, dependency injection and IoC in particular, please refer [here](https://martinfowler.com/articles/injection.html).  
  
Full repository [here](https://github.com/coroutineDispatcher/service_locator_demo).  
  
Stavro Xhardha
