---
title: "Dagger Multibindings saved my time"
datePublished: Thu Aug 08 2019 12:03:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5g0lw000209lf6t3a53ue
slug: dagger-multibindings-saved-my-time

---


While the Android architecture made development simpler, we should always be aware of the Lifecycle and with that, the right way to provide ViewModels. Creating one ViewModel factory for each Fragment and ViewModel you have is really expensive, bad architecture, and too much unnecessary code.  Not to mention the fact that you should care about scopes, dependencies, components modules when using Dagger for DI.

You can have a simple "generic" ViewModel factory for all your app as a Singleton, with the help of Kotlin Reflection and Daggers Multibinding.

```kotlin
@ApplicationScope //this is my singleton
class MySingletonViewModelFactory @Inject constructor(
    private val creators: Map<Class<out ViewModel>, @JvmSuppressWildcards Provider<ViewModel>> //dagger multibinding
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        val creator = creators[modelClass] ?: creators.entries.firstOrNull {
            modelClass.isAssignableFrom(it.key) //which means: does it extend a ViewModel or what?
        }?.value ?: throw IllegalArgumentException("unknown model class $modelClass")
        try {
            @Suppress("UNCHECKED_CAST")
            return creator.get() as T
        } catch (e: Exception) {
            throw RuntimeException(e)
        }
    }
}
```

What is provided here is a class, which has a Map in the constructor and extending the ViewModelProvider.Factory. On the only method it has, we are just going to provide our own instance of the ViewModel we need, which is the key of the map, and when that dependency is available we are going to require its particular dependencies which are "stored" in the value of that map. We can place as many ViewModels as we need, and this will be achieved by Dagger Multibinding. So it's just a Map<SomeViewModels, AndSomeViewModeldependencies>. The dependencies cannot just appear directly, so with the help of Provider<T> they arrive on the moment the ViewModel has just started. The @JvmSuppressWildcards is just making sure that the explicit Provider<ViewModel> will be on the Java generated files.

To put what we need on that map, Dagger has a nice way of doing it. First, you need to create an annotation which will pass the key to the map:

```kotlin
@Target(
    AnnotationTarget.FUNCTION,
    AnnotationTarget.PROPERTY_GETTER,
    AnnotationTarget.PROPERTY_SETTER
)
@Retention(AnnotationRetention.RUNTIME)
@MapKey
annotation class ViewModelKey(val value: KClass<out ViewModel>)
```

For example: To access HomeViewModel dependencies, we need a key, which happens to be the class itself. After that, what is needed is just to "put" those ViewModels in the map:

```kotlin
@Suppress("unused")
@Module
abstract class ViewModelModule {
    @Binds //provides the "generic" ViewModel factory created
    abstract fun bindsViewModelFactory(factory: MySingletonViewModelFactory): ViewModelProvider.Factory

    @Binds
    @IntoMap //puts our viewmodels into the map
    @ViewModelKey(GalleryViewModel::class)
    abstract fun bindsGalleryViewModel(viewModel: GalleryViewModel): ViewModel

    @Binds
    @IntoMap
    @ViewModelKey(HomeViewModel::class)
    abstract fun bindsHomeViewModel(viewModel: HomeViewModel): ViewModel

    @Binds
    @IntoMap
    @ViewModelKey(MapViewModel::class)
    abstract fun bindsMapViewModel(viewModel: MapViewModel): ViewModel

    @Binds
    @IntoMap
    @ViewModelKey(NamesViewModel::class)
    abstract fun bindsNamesViewModel(viewModel: NamesViewModel): ViewModel

    //....Lot of other ViewModels

    }
```

Remember, we are in @Singleton scope (although in the example using a custom annotation, personal preference). So, where are the values (dependencies) for each key provided? They just appear when the ViewModel is instantiated. And that's amazing about how Dagger solves DI problem in the Java/Kotlin/Android world:

```kotlin
class HomeViewModel @Inject constructor(
    private val homeRepository: HomeRepository
) : ViewModel() {
}
```

And when injecting a ViewModel factory:

```kotlin
class HomeFragment : BaseFragment() {

    @Inject
    lateinit var factory: ViewModelProvider.Factory
    private val homeViewModel by viewModels<HomeViewModel> { factory }


..... other stuff
}
```

Remember, all this work is not done just for a Repository, each ViewModel implementation might have different dependencies.

**Conclusion:**

Having tons of ViewModels in your Android app might make it too hard and expensive to produce them. When using Dagger Multibindings (if you are using dagger at all) it's a lot easier and prevents from writing bad code. Furthermore, if you want to add another Fragment and ViewModel, it's just like a game.
