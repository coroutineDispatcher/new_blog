---
title: "Usage of SharedFlow"
date: 2020-10-27T14:24:48+01:00
draft: false
tags: [Kotlin, Kotlin Coroutines, Kotlin Flow, SharedFlow, StateFlow, EventBus]
author: "Stavro Xhardha"
---

## onCreate

With the 1.4 Update, kotlin coroutines brought us a lot of features and improvements. One of them was the introduction of `SharedFlow`. Basically, what I would like to call it, is simply a "Single Event `StateFlow`". Remember single event `LiveData` workaround? Well, here you have it, but 100% kotlin, and not a workaround anymore.
If you are more interested in an even more detailed explanation, follow [this issue](https://github.com/Kotlin/kotlinx.coroutines/issues/2034) on Github, brought by Roman Elizarov. 

## onResume

### The problem
Let's suppose we have this scenario:

After a simple calculation on the ViewModel, we want to decide in which screen the user is going to be navigated to. Let's do that by using `StateFlow` (`MutableStateFlow`):

```kotlin
private val _state = MutableStateFlow<State>(State.Idle)
val state: StateFlow<State> get() = _state

fun checkCalculations(){
    if(calculationsSuccessful())
       _state.value = State.MoveForward
    else
      _state.value = State.ShowError
}
```

Until here, nothing new or problematic. Now, let's check what is happening on our fragment:

```kotlin
viewModel.state
    .onEach{ state ->
        when(state){
            MoveForward -> navigateToNextScreen(addToBackstack = true)
            else -> showError()
        }
    }.launchIn(lifecycleScope)
```

Seems also fine here. **But it's not!**

_If the user wants to go back, they will for sure, but guess what happens: They get re-routet to the screen they were already, moving the navigation back and forward,
everytime they press the back button._

## onPause

There is always a posibility for hacks:

```kotlin
viewModel.state
    .onEach{ state ->
        when(state){
            MoveForward -> {
                navigateToNextScreen(addToBackstack = true)
                viewModel.backtoIdleState()
            }
            else -> showError()
        }
    }.launchIn(lifecycleScope)
```

Then, in the `ViewModel`:

```kotlin
fun backtoIdleState(){
    _state.value = Idle
}
```

But not really a fan of these solutions.

## onResume

Let's see how the problem is solved by using `SharedFlow` (and how simple it is). Having the same scenario, but instead of:

```kotlin
private val _state = MutableStateFlow<State>(State.Idle)
val state: StateFlow<State> get() = _state

fun checkCalculations(){
    if(calculationsSuccessful())
       _state.value = State.MoveForward
    else
      _state.value = State.ShowError
}
```

we use:

```kotlin
private val _events = MutableSharedFlow<Event>()
val events = _events.asSharedFlow()

fun checkCalculations(){ 
    viewModelScope.launch(Dispatchers.Default){ //or whichever Dispatcher you need
        if(calculationsSuccessful())
           _events.emit(event)
      else
         _events.emit(event)
    }
}
```

In order to `emit()` from the `SharedFlow`, we need to use a coroutine or a `suspend` function. But that is all that needs to be changed (and in this case, the naming).

But what changes on the observers side? Nothing, absolutely nothing. We don't have to go back to idle manually, and we don't have to change a single line of it, and the bug is fixed. If the user hits back, they will be routed normally to the previous Fragment easily:

```kotlin
viewModel.state
    .onEach{ event ->
        when(event){
            MoveForward -> navigateToNextScreen(addToBackstack = true)
            else -> showError()
        }
    }.launchIn(lifecycleScope)
```

That's because `SharedFlow` is an implementation of `Flow` already.

## onDestroy

`SharedFlow` looks nice and promising, as many of the latest coroutines APIs. You can also use `SharedFlow` as a pure `EventBus`. Even though, this would make your debugging a little harder. For more information and special operations about `SharedFlow`, please visit he official docs [here](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/).