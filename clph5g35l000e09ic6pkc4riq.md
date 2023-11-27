---
title: "Stateful MVVM across process death using SharedFlow"
datePublished: Sun Dec 13 2020 17:15:51 GMT+0000 (Coordinated Universal Time)
cuid: clph5g35l000e09ic6pkc4riq
slug: stateful-mvvm-across-process-death-using-sharedflow

---


## onCreate

Lately, I have been playing and exploring `SharedFlow`. A pretty nice and very helpful design for situations like [this](https://coroutinedispatcher.com/posts/shared-flow/). But as I was building a `ViewModel` this week, I ran into an architecture design dilema.

## onPause

Disclamer: The solution given below, might not be the best out there, so please feel free to give your thoughts and critics for this.

## onResume

Let's consider this quick example:

```kotlin
class SomeViewModel @ViewModelInject constructor(
   private val dependency1: Dependency1,
   private val dependency2 : Dependency2,
   @Assisted private val savedStateHandle: SavedStateHandle
) : ViewModel() {
  private val _state : MutableSharedFlow<State> = MutableSharedFlow()
  val state : SharedFlow<State> = _state.asSharedFlow()

  sealed class State {
      data class Success(val data: List<Whatever>) : State()
      data class Fail(val message: String?) : State()
  }
}
```

This is how I normally manage it. But what if I want to save my data to survive process death? Adding a new extra variable to observe the `savedStateHandle` is error prone and actually makes things way worse, introducing a lot of bugs. But I really wanted to use `SharedFlow`, because of its' flexibility regarding single events which eventually make live easier. Also, with the help of Hilt, there is not much to be done when it comes to creating a `ViewModel` anymore, so the `savedStateHandle` comes for free.

Therefore, I was thinking: How about, store the whole state in `saveStateHandle`? It would restore immediately what we had before process death. What I needed to do though, was kick out 2 state `SharedFlow` variables:

```kotlin
class SomeViewModel @ViewModelInject constructor(
   private val dependency1: Dependency1,
   private val dependency2 : Dependency2,
   @Assisted private val savedStateHandle: SavedStateHandle
) : ViewModel() {

  val state = savedStateHandle.getLiveData<State>(Key)

  sealed class State {
      data class Success(val data: List<Whatever>) : State()
      data class Fail(val message: String?) : State()
  }
}
```

Ok, so are we back to `LiveData` again? Well, not really. As I hope that in the future, Google team might introduce a direct `Flow` extention from `savedStateHandle`, I was planning to convert it to a `Flow` first, with the help of `androidx.lifecycle:lifecycle-livedata-ktx:2.2.0`. So it becomes:

```kotlin
val state = savedStateHandle.getLiveData<State>(Key).asFlow()
```

With a small modification, since I need a hot `SharedFlow` instead:

```kotlin
val state = savedStateHandle.getLiveData<State>(Key).asFlow().shareIn(viewModelScope, SharingStarted.Lazily)
```

Sometimes, depending on `LiveData` for such implementation, might sound a little hacky, but it seems that's the only option to rely to (atm) in order to receive a hot `sharedFlow` which also survives process death.

One small detail that must not be missed is the fact that the `State` in this case **must** implement `Parcelable`. Whith the help of `kotlin-android` it can be easily tackled:

```kotlin
 sealed class State : Parcelable {
        @Parcelize
        data class Success(val data: List<Whatever>) : State()

        @Parcelize
        data class Fail(val message: String?) : State()
    }
```

All we would need to do now, is just keep setting the value of `savedStateHandle` according to our use case:

```kotlin
class SomeViewModel @ViewModelInject constructor(
   private val dependency1: Dependency1,
   private val dependency2 : Dependency2,
   @Assisted private val savedStateHandle: SavedStateHandle
) : ViewModel() {

  val state = savedStateHandle.getLiveData<State>(Key).asFlow().shareIn(viewModelScope, SharingStarted.Lazily)

  sealed class State : Parcelable {
        @Parcelize
        data class Success(val data: List<Whatever>) : State()

        @Parcelize
        data class Fail(val message: String?) : State()
    }

    fun doSomething(){
       val response =  dependency1.request()
       if(response.isSuccess) savedStateHandle.set(Key, State.Success(response.data))
       else savedStateHandle.set(Key, State.Fail(response.errorMessage))
    }

    companion object {
        const val Key = "saved_state_key"
    }
}
```

## onDestroy

Handling process death is still debatable in terms of: "Do we always need it?". In my opinion, it has its' importance, but sometimes it is too much. However, when choosing not to ignore this topic, it really brings a little more time to think and complexity regarding the design, so I was hoping I brought something which would make it simpler in case your project is already fully relied on `Flow`, `coroutines` API, and perhaps one of you might already made a decision to kick `LiveData` out.
