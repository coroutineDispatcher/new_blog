---
title: "What I learned from unit testing"
date: 2019-06-20T12:38:00.002+02:00
draft: false
aliases: ["/2019/06/what-i-learned-from-unit-testing.html"]
tags: [Kotlin, Unit testing, Android Development]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-yWGViBViCa8/XQten77OiAI/AAAAAAAAOEU/MXio_kH-aD8hLwfhZSyHgjAxy3Q9IxIgQCLcBGAs/s1600/ildefonso-polo-700782-unsplash.jpg)](https://1.bp.blogspot.com/-yWGViBViCa8/XQten77OiAI/AAAAAAAAOEU/MXio_kH-aD8hLwfhZSyHgjAxy3Q9IxIgQCLcBGAs/s1600/ildefonso-polo-700782-unsplash.jpg)

Testing, testing testing. I was getting inside the "Fear of getting behind" every time I heard that word. So I decided to react quickly. I knew nothing about testing and this is my experience getting my hands dirty with it. Please feel free to correct anywhere I'm wrong. This is a reason why I'm writing blogs.

So what the hell is testing?

_Testing is just a piece of code where you invoke your written production code and check its' behavior._  
So lets get more specific. In android there is 3 kind of tests:

1.  Unit testing:  *tests that validate your app's behavior one class at a time*
2.  Integration testing: _tests that validate either interactions between levels of the stack within a module, or interactions between related modules_
3.  End - to end tests: _tests that validate user journeys spanning multiple modules of your app_

_Definitions from the official [Android documentation](https://developer.android.com/training/testing/fundamentals). _

This article is about the first point , but you **shouldn't** ignore any of them.  The other 2 kinds, also need a real device or an emulator in order to run. Unit testing don't need a real device to run.

**The setup: **

JUnit is a nice testing tool for unit testing. So we need the dependency:

```groovy
testImplementation 'junit:junit:4.12'
```

Unit test on Android run on the _test_ package. Which is the same as your package name but greener. The IDE will show that for you. Don't confuse the _test_ with _androidTest_ because the second one is not for this case but for UI and Integration tests.

So what I'm going to test?

The simplest case is a repository from one of my personal projects.

```kotlin
class SetupRepository(
    private val treasureApi: TreasureApi,
    private val mSharedPreferences: Rocket
) {

    suspend fun makeCountryApiCallAsync(): Response<ArrayList<Country>> =
        treasureApi.getCountriesListAsync(COUNTRIES_API_URL)

    fun saveCountryToSharedPreferences(country: Country) {
        mSharedPreferences.writeString(COUNTRY_SHARED_PREFERENCE_KEY, country.name)
        mSharedPreferences.writeString(CAPITAL_SHARED_PREFERENCES_KEY, country.capitalCity)
    }

    fun isCountryOrCapitalEmpty(): Boolean {
        return mSharedPreferences.readString(COUNTRY_SHARED_PREFERENCE_KEY)!!.isEmpty()
                || mSharedPreferences.readString(CAPITAL_SHARED_PREFERENCES_KEY)!!.isEmpty()
    }
}
```

So what does this repository do? It has a suspend method which calls some data from a public API (using Retrofit), a method which saves my data to SharedPreferences (ignore the rocket naming, it's a refactored SharedPreferences class) and a method which just reads from SharedPreferences and returns true if one of my read values are empty.

Now let's start testing my class. On the _test_ directory create a new Kotlin class. Let it be empty in the begging. Annotate the class with @RunWith(JUnit4::class) and create 2 methods, setUp() and finish():

```kotlin
@RunWith(JUnit4::class)
class SetupRepositoryTest {

    @Before
    fun setUp() {
         print("Testing started")
    }

    @After
    fun finish() {
        print("Testing finished")
    }
}
```

This is the basic setup. But don't run the test yet, because it won't fire. It needs at least one test case. What would be the simplest thing to test here? Looks like it's the saveCountryToSharedPreferences. But I said that unit testing doesn't run on a real device. How in the world would you access the SharedPreferences. I wont. I will introduce you to test doubles:

My definition for Test Doubles is : Objects you need, but you don't even care about who they are. To make it clear, this is the definition of [Test Double by Google](https://testing.googleblog.com/2013/07/testing-on-toilet-know-your-test-doubles.html):

_A test double is an object that can stand in for a real object in a test, similar to how a stunt double stands in for an actor in a movie. These are sometimes all commonly referred to as “mocks”, but it's important to distinguish between the different types of test doubles since they all have different uses. The most common types of test doubles are stubs, mocks, and fakes._

I'm going to stop at _mocks_. You can build your own mock or you can use mocking libraries. Java/Kotlin applications have a library called _mockito_ which is awesome and super easy to  use, without caring to create _mocks_ on your own. Along with that, I will add _mockito-kotlin_ for more syntactic sugar and easy usage for testing my _suspend_ methods.

```groovy
testImplementation 'org.mockito:mockito-core:2.27.0'
testImplementation "com.nhaarman.mockitokotlin2:mockito-kotlin:2.1.0"
```

**Implementation:**

So, I don't care about my Rocket nor my TreasureApi class/interface. But I need them to instantiate the SetupRepository class. Let's mock:

```kotlin
@RunWith(JUnit4::class)
class SetupRepositoryTest {
    private lateinit var setupRepository: SetupRepository
    private lateinit var rocket: Rocket
    private lateinit var treasureApi: TreasureApi

    @Before
    fun setUp() {
        rocket = mock()
        treasureApi = mock()

        setupRepository = SetupRepository(treasureApi, rocket)
    }

    @After
    fun finish() {
        println("Testing finished")
    }
  }
```

Let's start testing that method now. One last thing: Unit testing is based on a triple A rule: Arrange, Act, Assert. Sometimes you might find some codelabs using the Given, When, Then keywords but it's pretty much the same.

Arrange: Let's pretend that something will have a certain behavior.  
Act: Let's call the method.  
Assert: Check if the selected behavior matches your expectations.

So, let's pretend that something will have a certain behavior. Since I want to send a country as parameter inside that method I will create a fake one: val country = Country("Albania", "Tirana", "blablabla.com"). Let's call the method, just like you do in production code: setupRepository.saveCountryToSharedPreferences(country). And now let's check if my fake object has been executed on my mocks (with the same content). For that we will need the help of verify operator from _mockito: _verify(rocket).writeString(COUNTRY_SHARED_PREFERENCE_KEY, country.name). Let's check the full method:

```kotlin
    @Test
    fun `when writing country execution should go fine`() {
        val country = Country("Albania", "Tirana", "no need")
        setupRepository.saveCountryToSharedPreferences(country)

        verify(rocket).writeString(COUNTRY_SHARED_PREFERENCE_KEY, country.name)
        verify(rocket).writeString(CAPITAL_SHARED_PREFERENCES_KEY, country.capitalCity)
    }
```

Run your test with the help of IDE. You will get notified if the test is correct it the IDE shows a green light:

[![](https://1.bp.blogspot.com/-FEhMqPPR30g/XQ-rJFeeKnI/AAAAAAAAOHg/4fGCO_ZicSwrBkH_3yVGH7lq-B84WAFrQCLcBGAs/s1600/Screenshot%2B2019-06-23%2Bat%2B18.38.42.png)](https://1.bp.blogspot.com/-FEhMqPPR30g/XQ-rJFeeKnI/AAAAAAAAOHg/4fGCO_ZicSwrBkH_3yVGH7lq-B84WAFrQCLcBGAs/s1600/Screenshot%2B2019-06-23%2Bat%2B18.38.42.png)

Since you can see my method, the test should go fine, but if the tests fails for some reason you have to check the method first. If you are sure that the method is correct, you have made wrong assertions.

Let's check a little harder implementation:

I want to check if my makeCountryApiCallAsync method returns an error, when my retrofit api returns an error. _But you are not making a real call, how do you know what the server returns?_ I don't. But I don't care for the real response, so I'm gonna use _mockito_ to fake that response for me. So let's pretend that the server brought some error:

```kotlin
`when`(treasureApi.getCountriesListAsync(COUNTRIES_API_URL)).thenReturn(
            Response.error(
                400, ResponseBody.create(
                    MediaType.parse("application/json"),
                    "{\"error_message\":[\"Do you even lift?\"]}"
                )
            )
        )
```

A small detail. since method is a _suspend_ method, you should wrap it in a runBlocking block. After we call the method: val apiResponse = setupRepository.makeCountryApiCallAsync(). After that we need to assert that the response code, is the same as the simulated response code: assertEquals(400, apiResponse.code()). And the full methoud would be:

```kotlin
    @Test
    fun `on api error response method should return response code 400`() = runBlocking {
        `when`(treasureApi.getCountriesListAsync(COUNTRIES_API_URL)).thenReturn(
            Response.error(
                400, ResponseBody.create(
                    MediaType.parse("application/json"),
                    "{\"error_message\":[\"Do you even lift?\"]}"
                )
            )
        )

        val apiResponse = setupRepository.makeCountryApiCallAsync()

        assertEquals(400, apiResponse.code())
    }
```

And pretty much that's it.

_Note: Don't really test only one case. One method can have lot's of cases to simulate._

_A small heads up for naming methods_: Test methods should be as clear as naming production methods. Perhaps wrong named tests, would confuse you more than you might be. Another thing I want to mention is that my naming pattern is not the best approach, but I find it nice like that. Name the method according to this pattern: <tested entity>\_<conditions/state during test>\_<expected result> .

_Small talk about Test Driven Development_: TDD means writing tests before production code. Since I learned testing after I created most of my project, I would skip TDD on this one, but i believe that TDD is the masterpiece of programming and everyone should use it as a technique.

**Conclusion: **  
Unit testing speeds up time of development. Also unit testing helps you catch more then 70% of your bugs in business logic. Please don't forget to learn all kinds of testing, but this one is the one you should start.  There were parts in my development life, where I couldn't simulate the behavior to reveal the bug, and so I ended up rewriting methods, classes or even packages. That's what unit testing taught me.
