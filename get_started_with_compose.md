---
title: "First experience with Jetpack Compose"
date: 2020-09-11T18:23:50+02:00
draft: false
author: "Stavro Xhardha"
tags : [Jetpack Compose, Compose Navigation, Android, AndroidX, ViewModels, Kotlin, Composable, Android UI, Kotlin, Lambdas]
---

![Photo by Nils Nedel @ Unsplash](/images/jetpack_post.jpg)

# onCreate
It's been a couple of weeks since Jetpack Compose reached in alpha state. So, I thought I should start giving it a try.
Getting started with it is not that hard. There are a lot of options, but I will render what worked for me:

- [Joe Birch's](https://joebirch.co) blog.
- Stack Overflow
- A quick read through the Google Codelabs
- Jetpack sample apps from Google's [repository](https://github.com/android/compose-samples) (Mostly looking at JetNews)

{{< admonition tip "Note">}}
This posts code example is not a best practice example. Mostly can be used as a "getting started with" guideline, as the title suggests.
{{</ admonition >}}

# onResume

## First, what is Jetpack Compose and how is it used?

Jetpack Compose is Android's new UI toolkit inspired by Flutter. With it, you can build Android apps without the need to construct a single XML layout code. Instead, everything is done in Kotlin. However, Activities and Fragments and a lot of android dependencies will still be there (even though, not in this article).

### Quick Start

To get started with JetPack Compose, a couple of Gradle dependencies need to be imported, as well as some configuration blocks need to be added in the `android` block.

{{< admonition type=failure title="Note" >}}
- Jetpack Compose doesn't work **without** Android Studio Canary 4.2+
- Jetpack Compose works only with Kotlin
{{</ admonition >}}

This is how your configuration should look:

```groovy
//App level build.gradle
android{
...
    compileOptions {
           sourceCompatibility JavaVersion.VERSION_1_8
           targetCompatibility JavaVersion.VERSION_1_8
       }
       kotlinOptions {
          jvmTarget = '1.8'
          useIR = true
       }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion compose_version
        kotlinCompilerVersion '1.4.0'
    }
}
```

And this is some of the dependencies you might need:

```groovy
//App level build.gradle
implementation "androidx.compose.ui:ui:$compose_version"
implementation "androidx.compose.material:material:$compose_version"
implementation "androidx.ui:ui-tooling:$compose_version"
```
However, if these are not sufficient for your app, you can check the list of all dependencies [here](https://developer.android.com/jetpack/compose/setup).

{{< admonition tip "Note">}}
Alternatively, the easiest path is to just go to `File -> New -> Compose Application`.
{{</ admonition>}}

{{< admonition info "Info">}}
Compose is also interoperable with existing Android applications.
{{</ admonition>}}

### The entry point to a Compose app

As in single activity apps, the Activity is the entry point for a Compose app as well. However, instead of calling `setContentView(R.layout.activity_main)` you have to construct the `setContent` method:

```kotlin
class MainActivity : AppCompatActivity() {
...
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val injector = (application as CatsApplication).injector
        setContent {
            CatsTheme {
                Surface(color = MaterialTheme.colors.background) {
                    CatsAppScafold(...)
                }
            }
        }
    }
}
```
Notice the `CatsTheme` block. Usually, that's the level where the theme is supposed to be set. But what is actually that component? Well, in Compose, everything is a function, most precisely, a composable function. 

### @Composable annotation
`@Composable` annotation is the way to make a function composable. That way, Android will know how to use it, and what to draw on the screen. If a Composable component requires a composable function, a normal method (without `@Composable` annotation) is not correctly constructed.

Now, let's check what is inside our `CatsAppScafold`:

Our entire app, is in this block:

```kotlin
@ExperimentalCoroutinesApi
@Composable
fun CatsAppScaffold(
    navigationViewModel: NavigationViewModel,
    networkClient: NetworkClient
) {
    val currentState = navigationViewModel.state.collectAsState()
    
    Scaffold(
        backgroundColor = Color.White,
        topBar = TopAppBar(title = { Text(text = "Cats <3") }),
        bodyContent = {
            Crossfade(
                current = currentState
            ) { screenState ->
                when (val screenStateValue = screenState.value) {
                    NavigationViewModel.NavigationState.Home -> {
                        HomeScreen(
                            navigateTo = navigationViewModel::navigateTo,
                            networkClient = networkClient
                        )
                    }
                    is NavigationViewModel.NavigationState.BreedDetails -> {
                        WebPageScreen(screenStateValue.urlToRender)
                    }
                }
            }
        }
    )
}
```

Let's go step by step:

A new method that might be noticed, is the `collectAsState()` extension. Since we have declared a `NavigationViewModel` in the activity scope for Navigation, a state is needed to be observed to check the screen status at any moment the app is running.

{{< admonition tip "Note">}}
In this case, what is being used for a state is a `StateFlow<T>` component from the coroutines API.
{{</ admonition >}}

This is how the `NavigationViewModel` looks:

```kotlin
@ExperimentalCoroutinesApi
class NavigationViewModel : ViewModel() {
    private val _state: MutableStateFlow<NavigationState> = MutableStateFlow(NavigationState.Home)
    val state: StateFlow<NavigationState> = _state

    sealed class NavigationState {
        object Home : NavigationState()
        data class BreedDetails(val urlToRender: String) : NavigationState()
    }

    fun navigateTo(state: NavigationState) {
        _state.value = state
    }

    fun goBack() {
        _state.value = NavigationState.Home
    }
}
```

With `collectAsState()`, you can register a listener for a composable function. There are other alternatives too:

```kotlin
LiveData.observeAsState() // For those who use LiveData
Observable.subscribeAsState() // For those who use Rx
```

{{< admonition tip "Note">}}
This approach is followed because the app has only 2 screens. The full example code repository link will be at the bottom of this post
{{</ admonition >}}

#### Let's go through the short blocks as well:

The `Scaffold` block is just a composable function that helps for Material Design elements construction. One of its properties, `topBar`, gives an end to the pain of toolbar manipulation. Using the `TopAppBar` for the toolbar, can make your life easier.
The `bodyContent` block, is the part where our views are going to be constructed.

But what's with the `Crossfade` composable?
`Crossfade` is a component used to perform a cross-fade animation when switching between two views. For this approach, we use it's `current` property to pass our latest state, and here is where the magic of navigation happens. I hope the block inside, is self-explanatory.

### ViewModels
The problem with the above navigation approach is that when you create `viewModels`, in lower scoped composables, they are attached to the only Activity alive in this case, which might be total overkill if one has a lot of screens and `ViewModel`s. Example:

```kotlin
@ExperimentalCoroutinesApi
@Composable
fun HomeScreen(
    navigateTo: (NavigationViewModel.NavigationState) -> Unit,
    networkClient: NetworkClient
) {
    val homeViewModel: HomeViewModel = viewModel(null, object : ViewModelProvider.Factory {
        override fun <T : ViewModel?> create(modelClass: Class<T>): T =
            HomeViewModel(networkClient) as T
    })

    val state = homeViewModel.state.collectAsState()

    when (val statesValue = state.value) {
        HomeViewModel.State.Idle -> homeViewModel.getCatBreeds()
        HomeViewModel.State.Loading -> CenterLoadingIndicator()
        is HomeViewModel.State.Success -> CatBreedsList(
            breedsList = statesValue.breedsList,
            itemClickAction = navigateTo
        )
        is HomeViewModel.State.Error -> ErrorView(retryAction = {
             homeViewModel.getCatBreeds() 
             }
        )
    }
}
```
We need this `ViewModel` only when the HomeScreen is alive but AndroidX `ViewModel`s are associated to the scope of Fragment/Activity once created, in this case, `MainActivity`, and only when `MainActivity` is destroyed, their `onCleared()` method is called. 

### Challenges compared to (getting started with) Flutter

It would not hurt if the developer trying Compose, has already tried Flutter. Flutter uses the same logic as Compose (only the API changes a little), but since Compose is less mature than Flutter, mastering Compose after knowing a little Flutter would be a piece of cake.

Jetpack Compose is not that intuitive as Flutter. A very good example, would be this block:

```kotlin
LazyColumnFor(breedsList) { breed ->
        RippleIndication()
        Card(...){...}
    }
```

To make the `Card` element do a ripple effect when you click it, you have to call the `RippleIndication()` constructor, and below it the element having the ripple effect (or vice versa?). In Flutter it would definitely be something like this:

```kotlin
RippleIndication(
    child: Card(...)
)
```
Which IMO, it's far more intuitive.

#### Snackbars
`Snackbar`s: Believe it or not, I still don't know how you dismiss a `SnackBar` in Jetpack Compose. Let me know when there is a non-hacky way (or any function parameter that I missed reading).

#### mages

In my own humble opinion, missing a feature such as loading an Image from a certain URL, without 3rd party dependency, is a problem. If we still compare with Flutter: A `NetworkImage()` is already there. While in Compose, for the moment, you are either supposed to write some logic on your own or just use `dev.chrisbanes.accompanist:accompanist-coil:0.2.1` (just a wrapper on top of Coil). And all that, for a `CoilImage(data = breed.imageUrl)`.

## More notes

### WebViews
Using WebViews in Jetpack Compose, is same as using the `WebView` class in the current status. However, there is a nice `AndroidView` wrapper for every tool that is either not ready yet, or not planning to be released a new one, ever. It took me some time to figure out how to do it, so I'll just paste the code here:

```kotlin
@SuppressLint("SetJavaScriptEnabled")
@Composable
fun WebPageScreen(urlToRender: String) {
    AndroidView(viewBlock = ::WebView) { webView ->
        with(webView){
            settings.javaScriptEnabled = true
            webViewClient = WebViewClient()
            loadUrl(urlToRender)
        }
    }
}
```

{{< admonition tip "Info">}}
[The sample app can be found in this repository.](https://github.com/coroutineDispatcher/cats_compose_sample)
{{</ admonition >}}

# OnDestroy
As a conclusion, I would say that Jetpack Compose looks very promising for native Android, and Kotlin gives awesome flexibility to use it. Android suffers a lot, as well as the developer who works on it, therefore Compose it's an awesome solution. However, if they want to promote Compose to terms like Kotlin Multi-Platform, or Server Driven UI, they are so much behind. As far as I have heard, SwiftUI is far ahead.

