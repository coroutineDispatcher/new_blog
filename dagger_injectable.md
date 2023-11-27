---
title: "A comparison between Flutter's Injectable and Dagger 2 in the JVM world"
date: 2022-04-03T21:04:29+02:00
draft: false
---

![](/images/salzbourg_at_night.png)

I have been playing around with Flutter lately. More specifically I discovered [Injectable](https://pub.dev/packages/injectable); a dependency injection tool for Flutter, also built by google, but extremely similar to [Dagger 2](https://dagger.dev). Since I also feel very comfortable in Dagger 2, I would like to make some comparisons between them.

## Differences

### Generated code

The biggest difference they have is that Flutter Injectable it's a combination of two libraries: [`GetIt`](https://pub.dev/packages/get_it) and `injectable` where `injectable`makes sure to generate the code and `GetIt`is the one that is used internally by the compiler to construct the service locator; while in Dagger 2, dependencies are generated manually. Let's see an example:

&nbsp;

```dart
@injectable  
class ServiceA {}  
  
@injectable  
class ServiceB {  
    ServiceB(ServiceA serviceA);  
}  
```

&nbsp;

This would generate: 

&nbsp;

```dart
final getIt = GetIt.instance;  
  
void $initGetIt(GetIt getIt) {  
 final gh = GetItHelper(getIt, environment);  
  gh.factory<ServiceA>(() => ServiceA());  
  gh.factory<ServiceB>(ServiceA(getIt<ServiceA>()));  
} 
```

&nbsp;

If you check the `factory<T>` methods, these are not normal dart methods, but rather helper methods to generate constructors for given dependencies from `GetIt`package.
To understand what `Injectable` is doing differently think about Dagger 2 generated `Koin` or `KodeIn` code instead of just plain Java code (or Kotlin).
Let's see what Dagger would generate for two dependencies and one provided dependency via module.

&nbsp;

```java
@Generated(
  value = "dagger.internal.codegen.ComponentProcessor",
  comments = "https://google.github.io/dagger"
)
public final class DaggerApplicationComponent implements ApplicationComponent {
  private ApplicationModule applicationModule;

  private ApplicationModule_ProvideDependency1Factory provideDependency1Factory;

  private ApplicationModule_ProvideDependency2Factory provideDependency2Factory;

  private Provider<ProvidedDependency> providedDependency;

  private DaggerApplicationComponent(Builder builder) {
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  // etc etc etc
  ....
}
```

&nbsp;

### Performing DI and field injection.

This is also a difference between the two platforms. That’s because of language limitations or compiler behavior. For example, in Flutter/Dart, if you keep running the app and do not run:

&nbsp;

```bash
flutter packages pub run build_runner build
```

&nbsp;

The app will continue with the old dependencies even though you might have added new dependencies in the graph (which might result in crashes later).
In Dagger 2, if something goes wrong with your dependencies or something goes wrong during the compilation, you will always know. Of course, in Flutter, you can listen for dependency declaration changes in Flutter as well, but the Dagger 2 way seems smoother:

&nbsp;

```bash
flutter packages pub run build_runner watch 
```

&nbsp;

Field injection is basically different because the code generation is different:

&nbsp;

```dart
/// Flutter:
Dependency dependency = getIt<Dependency>();
```

&nbsp;

And in Dagger 2:

&nbsp;

```kotlin
// Kotlin
@Inject
lateinit var dependency: Dependency

override fun onInit(){
    DaggerSomeComponent.builder().build().inject(this) //or something similar
}
```

&nbsp;

### Bridges between written code and compiler/processor

Well, what I mean is the entry point. For Dagger 2 it's components, for `Injectable`, it's an annotation. Let's see again an example:

&nbsp;

```dart
final getIt = GetIt.instance;  
  
@InjectableInit(  
  initializerName: r'$initGetIt', 
)  
void configureDependencies() => $initGetIt(getIt);   // generated after first build
```

```dart
void main() {  
 configureDependencies();  
 runApp(MyApp());  
}  
```

&nbsp;

And in Dagger 2:

&nbsp;

```kotlin
@Component(dependencies = [Dep1::class, Dep2::class])
interface MyComponent{
    fun dep1() : Dep1
    fun dep2() : Dep2

    // or for field injection just: 
    fun inject()
}
```

```kotlin
DaggerMyComponent.builder().module(WhateverModule()).build().dep1()
// or: 
DaggerMyComponent.builder().module(WhateverModule()).build().inject(this)
```

&nbsp;

I'm pretty sure that the more you deep dive on each of them, there will be a lot more differences. Let's see some similarities though:

## Similarities

I would say that `Injectable` and `Dagger 2` are two of the most similar platforms that you can use across different tech stacks. The declaration is something that both platforms are almost the same and do almost the same thing. 

### Declarations.

Let's see how you declare dependencies in `Injectable` and `Dagger 2`.

&nbsp;

```dart
@injectable  
class ServiceA {}  
  
@injectable  
class ServiceB {  
    ServiceB(ServiceA serviceA);  
}
```

&nbsp;

In Dagger 2, with Kotlin it would be:

&nbsp;

```kotlin
class ServiceA @Inject constructor()
class ServiceB @Inject constructor(serviceA: ServiceA)
```

&nbsp;

But what if `ServiceA` was a singleton?

&nbsp;

```dart
@singleton // or @lazySingleton  
class ServiceA {}
```

&nbsp;

And in Dagger 2:

&nbsp;

```kotlin
@Singleton
class ServiceA @Inject constructor()
```

&nbsp;

And what if `ServiceA` needs a builder to be constructed? For that we need a module, correct?

&nbsp;

```dart
@module  
abstract class ServiceModule {  
  ServiceA serviceA get => ServiceA.builder().someConfiguration().anotherConfiguration().build();
}  
```

&nbsp;

```kotlin
@Module
object SomeModule {
  @Provides
  fun provideServiceA() = ServiceA.builder().someConfiguration().anotherConfiguration().build()
}
```

&nbsp;

Small difference here: In Dagger 2 you have to include the module in the respective component, that's not the same case for Injectable. The dependency will be generated by itself.

### Named dependencies

If for some reason you would need 2 instances of the same object but you want to differentiate between them, you can always use the `@Named()` annotation, in both tools:

&nbsp;

```dart
@injectable  
class MyRepo {  
   final Service service;  
    MyRepo(@Named('impl1') this.service)  
}  
```

&nbsp;

```kotlin
class MyRepo @Inject constructor(@Named("impl1") service: ServiceA)
```

&nbsp;

Difference exists in declaration though:

&nbsp;

```dart
@Named("impl1")  
@Injectable(as: Service)  
class ServiceImpl implements Service {}  
  
@Named("impl2")  
@Injectable(as: Service)  
class ServiceImp2 implements Service {}  
```

&nbsp;

For dagger here, it works with a module:

&nbsp;

```kotlin
@Module
abstract class SomeModule {
    @Provides
    @Named("impl1")
    abstract fun bindsImpl1Service(serviceImpl: ServiceImpl) : Service

    @Provides
    @Named("impl2")
    abstract fun bindsImpl1Service(serviceImpl: ServiceImp2) : Service
}
```

&nbsp;

### Factory pattern

This was slightly easier in Injectable. That's because it's built after Dagger and even though inspired by Dagger, this could be done better and actually was done better:

&nbsp;

```dart
@injectable  
class BackendService {  
  BackendService(@factoryParam String url);  
}  
```

&nbsp;

In Dagger 2, before `@AssistedInject` became part of the library, you needed an extra dependency in Gradle files. This is how it looks now:

&nbsp;

```kotlin
class BackendService @AssistedInject constructor(private val client: NetworkClient, @Assisted url: String){
   @AssistedInject.Factort
   interface Factory {
       fun create(url: String) : BackendService
   }
}
```

&nbsp;

These are the features that most remind you of each other when you work with any of them. One thing to note though is that Dagger has a lot more features (and more annotations) but because they are either missing in Injectable or they are not the main features, that part is not covered in this article.

## Closing thoughts

Although it doesn't make sense to come to a conclusion about which tool is better, because the compared tools are used on different platforms, I would say that I still remain an old fan of Dagger 2. Now that hilt is out there, it's even simpler than it was before. But working with Flutter's Injectable was also a surprise. I can't say I know the whole platform really well, and it took me 10 minutes to get used to this dependency injection tool, it's just amazing.