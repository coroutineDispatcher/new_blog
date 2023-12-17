---
title: "My first trip with Kotlin Multiplatform (Mobile)"
seoTitle: "My first trip with Kotlin Multiplatform (Mobile)"
seoDescription: "This article will guide you through my journey and what you need to know before you start with Kotlin Multiplatform."
datePublished: Sun Dec 17 2023 18:45:52 GMT+0000 (Coordinated Universal Time)
cuid: clq9u86j1000108jyf4rn4zke
slug: my-first-trip-with-kotlin-multiplatform-mobile
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702839291211/e58823a5-b1fe-444d-9328-94cda5897e81.webp
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1702838643650/251c2b20-1ab4-414a-9bee-18e78b63ef6e.webp
tags: swift, ios, android, kotlin, getting-started, kotlin-multiplatform, kmm

---

I have to be honest; I never liked it. The main reason was that I was not comfortable with compatibility issues when adding/removing dependencies. It seemed at the time that it was consuming more time than I predicted trying to figure out what was going on. The error messages were not a good helper either. However, now that I heard it reached a stable stage, I thought I should give it a try.

One of my favorite apps is [Pocket, from Mozilla](https://www.mozilla.org/en-US/firefox/pocket/) team, so I thought I should create a mock of that one, but in multiplatform.

*Note: The project simply reflects my journey with Kotlin Multiplatform. I do not claim that this repository serves as a good example at this point. It's not a bad example either, but it currently lacks a proper navigation technique and is somewhat hacky.*

The project, along with its readme and details is found [here](https://github.com/coroutineDispatcher/belt).

## What I wanted to achieve

I wanted to test how closely the Kotlin Multiplatform mission holds true. In other words, all the logic can be in Kotlin, and the rest is negotiable—you can either go native or try Compose Multiplatform. As an Android developer, I was in the perfect conditions to test such a concept, so I chose to write as much as I could with Kotlin and Compose and then see how far I could go. If you haven't figured it out by now, my project would be on Kotlin Multiplatform Mobile. I have already tried KM Desktop, and it seems pretty decent, considering that there aren't many modern solutions to build desktop apps anyway.

### Fondamental concept

Before you understand everything else in Kotlin Multiplatform, you need to be familliar with the `expected` and `actual` keywords. I am assuming the reader already knows them. If not, head over [here](https://kotlinlang.org/docs/multiplatform-expect-actual.html).

### First glance

In the beginning, this is the directory structure if you chose Kotlin Multiplatform with Jetbrain Compose:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702837428830/2383a371-ecc0-4f95-917a-fca3a49e35e4.png align="center")

Then you just get an `App` composable which is called by `composableApp/iOSMain/kotlin` (not to be confused with `iosApp` ) that stands alone in parallell with `composableApp`. Through a composable called `MainViewController` that wraps your `App` , your iOS app starts. You don't have to care about how the `MainViewController` is built internally unless you want to learn more about iOS in specific.

The same for Android. Your `MainActivity` calls `App` composable in it's `onCreate`.

```kotlin
// iOS kotlin
fun MainViewController() = ComposeUIViewController { App() }
```

```kotlin
// android
class MainActivity : ComponentActivity() { 
...
    override fun onCreate(...) { 
        setContent { 
            App()
         }
    }
}
```

Once you understood that, you can freely work in the `commonMain` directory in pure Kotlin.

### Architecture

I kept it classical Android. MVVM with use cases, one repository and a couple of datasources, unidirectional and using kotlin flows. Worked like charm.

### But what about viewmodels?

Yes. I created my own viewmodel, pretending to copy the minimal behaviour of a real Android viewmodel, without the deep `onRetainInstanceState` that it goes with. My viewmodels would just dispose themselves once the composable would dispose.

```kotlin
interface ViewModel<T : ScreenState> {
    val state: StateFlow<T>
    val viewModelScope: CoroutineScope
    fun dispose()
}
```

To make sure every actual implementation of a `ViewModel` had a state assigned, I put a generic class `T` that was the child of every `ScreenState` sealed interface.

The `state` is provided as typically done in Android, by a `StateFlow<T>` , the `viewModelScope` for launching coroutines and the `dispose()` function to cancel the scope once the `ViewModel` is no longer needed. I copied the `dispose` name concept from Flutter though.

But, one limitation to this behaviour: It's always important to remember to dispose the viewmodel once my composable has been disposed. This at the moment is pretty manual if a project would grow to much, and needs improvement:

```kotlin
// in a composable function
val viewModel = remember { BeltAppDI.mainViewModel() }

DisposableEffect(Unit) {
    onDispose { viewModel.dispose() }
}
```

And that's how I solved the ViewModel part.

*Note: You don't have to use* `ViewModel` *s at all. There are plenty of ways to architect a KMM app. That's just my way of doing things.*

### How do I do dependency injection?

Again, It's just me. There is nothing wrong with libraries like Koin for instance. They are doing it pretty well. But I thought because the project was not that big, and it was just a project to get your hands dirty with, why bother? Therefore, I just did manual DI with pure Kotlin, as you may have already noticed when I am calling the `viewmodel` in the composable function.

```kotlin
object BeltAppDI {
    private val realm by lazy {
        Realm.open(RealmConfiguration.create(schema = setOf(LinkProperty::class, Tag::class)))
    }
    private val httpClient by lazy { HttpClient() }
    private val linkDatasource by lazy { LinkDatasource(httpClient, realm) }
    private val tagsDatasource by lazy { TagsDatasource(realm) }
    private val linkRepository by lazy { LinksRepository(linkDatasource, tagsDatasource) }
    private val toggleFavouriteItemUseCase by lazy { ToggleFavouriteItemUseCase(linkRepository) }
    private val observeLinkPropertiesUseCase by lazy { ObserveLinkPropertiesUseCase(linkRepository) }
    private val isValidUrlUseCase by lazy { IsValidUrlUseCase(linkRepository) }
    private val addUrlToDatabaseUseCase by lazy { AddUrlToDatabaseUseCase(linkRepository) }
    private val deleteItemUseCase by lazy { DeleteItemUseCase(linkRepository) }
    private val getTagsUseCase by lazy { GetFilteredTagsUseCase(linkRepository) }
    private val updateTagForLinkPropertyUseCase by lazy {
        UpdateTagForLinkPropertyUseCase(
            linkRepository
        )
    }
    private val getLinkPropertyUseCase by lazy { GetLinkPropertyUseCase(linkRepository) }
    private val deleteTagUseCase by lazy { DeleteTagUseCase(linkRepository) }

    fun mainViewModel() = MainViewModel(
        observeLinkPropertiesUseCase,
        toggleFavouriteItemUseCase,
        deleteItemUseCase,
        getTagsUseCase
    )

    fun tagsViewModel(linkProperty: LinkProperty) = TagsViewModel(
        getTagsUseCase,
        updateTagForLinkPropertyUseCase,
        linkProperty,
        getLinkPropertyUseCase,
        deleteTagUseCase
    )

    fun newLinkViewModel() = NewLinkPropertyViewModel(
        addUrlToDatabaseUseCase,
        isValidUrlUseCase
    )
}
```

### Where are my Java libraries?

If you are already an Android Developer like me, you would think that since you know Kotlin, you can have the same level of access in the classical libraries as well. Careful here though. You do not depend on Java libraries in a Kotlin Multiplatform project. That's reasonable of course, because your project needs to compile also natively for iOS. In this project, I noticed this pretty fast because my datasrouce depended on a html parser. The most famous one is Jsoup, but luckily for me, someone had [already built a parser like that for Kotlin Multiplatform](https://github.com/MohamedRejeb/Ksoup).

### Persistence

I used Realm for kotlin. It was pretty straightforward. Another option would have been SQL Delight, but apparently it was causing a conflict with my HTML parser above. I was also pretty content to find that I could directly sync the Realm db with Mongo DB on the server. At the moment I did not do it, but in case I want to publish the app to a wider area of users (now it's just on Github), I can add online sync and it would work out of the box. You can find more information [here](https://medium.com/realm/getting-started-with-kotlin-multiplatform-mobile-kmm-with-flexible-sync-a6b7cbdd56f3).

### Navigation

I confes. I used a hack. You should never use hacks for production apps. Now the thing is that the navigators in Kotlin multiplatform are all third parties (Jetbrains team says that they are working on it), some of them affect the architecture and some of them are not that straightforward. In the future, I might want to integrate [Voyager](https://voyager.adriel.cafe/). That's at least the quickest one to integrate so far. But what did I do? Again, since my project was three to four screens in total, I thought I could just build my own stack (in Kotlin) and add or remove stuff with internal logic. But we have one problem though: *The back behaviour*. Unfortunately you either have to go natively or just stick to a navigation library. I chose the first option.

Below, is my pretty-androidish `BackStackHandler`.

```kotlin
class BackStackHandler(val initialScreen: Navigation) {
    private val backStack = mutableListOf<Navigation>()
    private val _currentNav = MutableStateFlow<Navigation?>(current())
    val navigation = _currentNav.asStateFlow()

    init {
        add(initialScreen)
    }

    fun add(navigation: Navigation) {
        backStack.add(navigation)
        update()
    }

    fun popBackStack() {
        if (current() != null) {
            backStack.removeLast()
        }
        update()
    }

    private fun update() {
        _currentNav.tryEmit(current())
    }

    private fun current() = backStack.lastOrNull()

    fun clearAndFinish() {
        backStack.clear()
        update()
    }

    fun popToLast() {
        backStack.firstOrNull()?.let { lastItem ->
            backStack.removeAll { it != lastItem }
            update()
        }
    }

    fun initialState(): Boolean = current() == initialScreen
}
```

This state was observed by my root/main composable and then making the decision about which part of the app the screen needed to be.

```kotlin
when (val state = navigationState.value) {
    Navigation.MainScreen -> MainScreen(backStackHandler)
    is Navigation.TagsScreen -> TagsScreen(state.linkToModify, backStackHandler)
    Navigation.AddNewLinkScreen -> AddNewLinkScreen(backStackHandler)
    null -> onAppExit()
}
```

I have left `null` on purpose. That's just means that the app should exit. Till now, I tried my best not to go native. Now things change. `onAppExit` is a native lambda function, so I need platform specific API for handling it. Until now, we still don't have to do anything with `expected` and `actual` declarations. I failed to mention above, but Kotlin Multiplatform already has tones of iOS APIs already in Kotlin, so you don't have to code in swift, and the `exit` function is one case:

```kotlin
fun MainViewController() = ComposeUIViewController { App { exit(-1) } }
```

For Android, I just had to finish the Activity:

```kotlin

setContent {
    App {
        finishAffinity()
    }
}
```

### Platform specific code

Platform specific code for my app is written for only two use cases:

* The app needs to share the saved link/website in the contacts/social media/bluetooth etc.
    
* The app needs to handle back press/ back gesture.
    

```kotlin
expect class LinkManager {
    fun shareLink(url: String)
    fun openLink(url: String)
}

expect val linkManager: LinkManager
```

And then in iOS:

```kotlin
actual class LinkManager {
    actual fun shareLink(url: String) {
        val activityViewController = UIActivityViewController(listOf(url), null)
        UIApplication.sharedApplication.keyWindow?.rootViewController?.presentViewController(
            activityViewController,
            animated = true,
            completion = null
        )
    }

    actual fun openLink(url: String) {
        val safariViewController = SFSafariViewController(uRL = NSURL(string = url))
        UIApplication.sharedApplication.keyWindow?.rootViewController?.presentViewController(
            safariViewController,
            animated = true,
            null
        )
    }
}

actual val linkManager = LinkManager()
```

Last, in Android:

```kotlin
actual class LinkManager(private val activityContext: Context) {
    actual fun shareLink(url: String) {
        val intent = Intent(Intent.ACTION_SEND)
        intent.type = "text/plain"
        intent.putExtra(Intent.EXTRA_TEXT, url)
        intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
        activityContext.startActivity(intent)
    }

    actual fun openLink(url: String) {
        val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
        intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
        activityContext.startActivity(intent)
    }
}

actual val linkManager: LinkManager by lazy {
    LinkManager(checkNotNull(MainActivity.instance?.baseContext))
}
```

*Yes. I will improve providing an Android context better in the future. This way is really dangerous for memory leaks in case I forget to nullify the activity instance in the \`MainActivity\`.*

#### Back button and back gestures

This is where the hack begins. I built an event bus and just intercepted the Android back button/ back gestures, iOS back gesture independently. [For Android](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture), you need to have a specific version of the activity library. For iOS, I had no idea how to solve it. However some swift code seemed to do the trick (and if you know a better way, please feel free to sugest it).

```swift
func makeUIViewController(context: Context) -> UIViewController {
    let viewController = MainViewControllerKt.MainViewController()
    let swipeBackGesture = UISwipeGestureRecognizer(target: viewController, action: #selector(viewController.handleSwipe(_:)))
    swipeBackGesture.direction = .right
    viewController.view.addGestureRecognizer(swipeBackGesture)
    return viewController
}
```

```swift
extension UIViewController {
    @objc func handleSwipe(_ gesture: UISwipeGestureRecognizer) {
        if gesture.direction == .right {
            BackGestureListenerKt.backGesture()
        }
    }
}
```

For Android

```kotlin
private val backCallback = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        OnBackInvokedCallback { EventBus.send(EventBus.Event.OnBackPressed) }
    } else {
        null
    }
```

Of course I didn't forget about the lifecycle:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        backCallback?.let {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                onBackInvokedDispatcher.registerOnBackInvokedCallback(
                    OnBackInvokedDispatcher.PRIORITY_DEFAULT,
                    backCallback
                )
            }
        }

        onBackPressedDispatcher.addCallback {
            EventBus.send(EventBus.Event.OnBackPressed)
        }

        setContent {
            App {
                finishAffinity()
            }
        }
    }

override fun onDestroy() {
        super.onDestroy()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            backCallback?.let {
                onBackInvokedDispatcher.unregisterOnBackInvokedCallback(it)
            }
        }
    }
```

And then once the back gesture was detected, I would just call my Kotlin Event Bus to notify the app to pop the backstack through a `SharedFlow<T>`:

```kotlin
LaunchedEffect(eventBus) {
        eventBus.events.collectLatest { event ->
            when (event) {
                EventBus.Event.OnBackPressed -> {
                    if (platform() == iOS && backStackHandler.initialState()) return@collectLatest
                    backStackHandler.popBackStack()
                }
            }
        }
    }
```

The condition there is on purpose, I don't want to exit the iOS app when the back gesture has happened. That's not a normal behaviour in iOS but in Android it is.

*One could say, that integrating the library could have been easier and I agree. I can do that in the future hopefully.*

### Resources

Resources are pretty straightforward, you just have to be careful not to misspell them:

```kotlin
Icon(painter = painterResource("tag.png"), "Tag")
```

But if you want to load a network image for instance, according to my research, the most popular one was [Kamel-Media](https://github.com/Kamel-Media/Kamel):

```kotlin
Box(
                    modifier = Modifier.weight(1f).padding(8.dp).clip(RoundedCornerShape(16.dp))
                ) {
                    when (val resource = asyncPainterResource(item.image.orEmpty())) {
                        is Resource.Loading -> {
                            Text("Loading...")
                        }

                        is Resource.Success -> {
                            Image(
                                painter = resource.value,
                                contentDescription = "link image",
                                modifier = Modifier
                                    .fillMaxWidth()
                                    .fillMaxHeight()
                            )
                        }

                        is Resource.Failure -> {
                            // TODO Failure image
                            println(resource.exception)
                        }
                    }
                }
            }
```

It's pretty cool because you don't have to load a thumbnail until your image is processed. You can just create a new composable yourself.

### Testing

For testing also, remember: You are not on JVM anymore. But Kotlin already has APIs for testing. Take a look [here](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-run-tests.html) to get started. For me, I need to know more about UI testing in KMM, but as for unit tests, I just wrote a test for my `BackStackHandler` with the help of Turbine for flows:

```kotlin
class BackStackHandlerTest {
    private val sut = BackStackHandler(initialScreen = Navigation.MainScreen)

    @Test
    fun `MainScreen is the initial state`() = runTest {
        sut.navigation.test {
            val defaultScreen = awaitItem()
            println(defaultScreen)
            assertTrue { defaultScreen == Navigation.MainScreen }
        }
    }

    @Test
    fun `adding a new screen is reflected on the back stack`() = runTest {
        sut.navigation.test {
            awaitItem() // Default main screen or initial screen
            sut.add(Navigation.AddNewLinkScreen)
            val newAddedScreen = awaitItem()
            assertTrue { newAddedScreen == Navigation.AddNewLinkScreen }
        }
    }

    @Test
    fun `popping the backstack takes me to the previous screen`() = runTest {
        sut.navigation.test {
            awaitItem()
            sut.add(Navigation.TagsScreen(LinkProperty()))
            awaitItem()
            sut.add(Navigation.AddNewLinkScreen)
            awaitItem()
            sut.popBackStack()
            val correctScreen = awaitItem()
            assertTrue { correctScreen is Navigation.TagsScreen }
        }
    }

    @Test
    fun `popping the backstack twice takes me to the previous screen`() = runTest {
        sut.navigation.test {
            awaitItem()
            sut.add(Navigation.TagsScreen(LinkProperty()))
            awaitItem()
            sut.add(Navigation.AddNewLinkScreen)
            awaitItem()
            sut.popBackStack()
            awaitItem()
            sut.popBackStack()
            val correctScreen = awaitItem()
            assertTrue { correctScreen is Navigation.MainScreen }
        }
    }

    @Test
    fun `no matter how many items there are in the backstack pop to last should clear all except the initial`() =
        runTest {
            sut.navigation.test {
                awaitItem()
                sut.add(Navigation.TagsScreen(LinkProperty()))
                awaitItem()
                sut.add(Navigation.AddNewLinkScreen)
                awaitItem()
                sut.add(Navigation.TagsScreen(LinkProperty()))
                awaitItem()
                sut.add(Navigation.AddNewLinkScreen)
                awaitItem()

                sut.popToLast()

                val correctScreen = awaitItem()
                assertTrue { correctScreen is Navigation.MainScreen }
            }
        }

    @Test
    fun `if the screen is in the initial state the backstack handler should reflect that`() {
        sut.popToLast()

        assertTrue { sut.initialState() }
    }

    @Test
    fun `if nothing is in the backstack the app should finish ie the state is null`() = runTest {
        sut.navigation.test {
            awaitItem()
            sut.clearAndFinish()
            val latestState = awaitItem()
            assertNull(latestState)
            assertFalse { sut.initialState() }
        }
    }
}
```

If you don't know how `Turbine` works, take a quick look [here, in my other article](https://dispatchersdotplayground.hashnode.dev/simplify-testing-kotlin-flows-with-turbine).

If I am correct, `mockk` or `mockito` should not work on Kotlin Multiplatform either. Even better. Build your own mocks/fakes.

## Build time

For Android, it was pretty fast. For iOS of course you have to be a little bit more patient while the project compiles.

## Conclusion

Kotlin Multiplatform seems promising, but it's challenging at the same time. Not only do you have to start thinking in Multiplatform, but, in my opinion, you also need to know more details for the specific platforms. For example, you have to be ready to write Swift or JavaScript if your project includes the web. Since I have been in Android for 5-6 years and never professionally worked on any other technologies (although I don't mind trying new technologies, as I have done in my private time), writing that little iOS code consumed a disproportionate amount of time. However, it is also my opinion that Kotlin Multiplatform does a lot of good in the developer mindset and in software engineering in general because it teaches you not to be too tied to your technology but to strive towards being more platform-agnostic.