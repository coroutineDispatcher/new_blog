---
title: "Let's get beyond null safety"
datePublished: Thu Aug 15 2019 13:25:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5poff000g09ic7suh1hqt
slug: lets-get-beyond-null-safety

---


_Inspired by myself and perhaps some other people, who coded Java style in Kotlin. I have seen tones of articles which (mostly) highlight Kotlins null safety and nothing more. So that's it? If it was only for that I swear I would be still using Java with some null checks. Therefore, this article suggest what to use best in Kotlin as well as droping some everyday Java habits. _  
_What this article is **not**: A comparison between Java and Kotlin.  _  
**Variables:**  
There is a lot of new things obviously but sharing what is important for me in variables is the lateinit and lazy implementation.

```kotlin
private lateinit var expectedUrl: String

@FistRunningMethod //doesn't exists IRL
fun start(){
  expectedUrl = "www.google.com"
  apiInterface.call(expectedUrl)
}

@SecondRunningMethod //doesn't exists IRL
fun start(){
  expectedUrl = "www.amazon.com"
  apiInterface.call(expectedUrl)
}
```

In case you never initialize the variable and use it, Kotlin will throw an exception indicating that the variable has never been initialized:

```kotlin
private lateinit var expectedUrl: String

@FistRunningMethod //doesn't exists IRL
fun start(){
  apiInterface.call(expectedUrl) //will throw exception
}
```

Let's jump into something fun, the lazy initialization, which basically throws us into one of the super powers of Kotlin: [_Delegated Properties_](https://kotlinlang.org/docs/reference/delegated-properties.html). Delegated Properties is just a way of programming in Kotlin which must (or at least has to) be used. Think of delegation as the operator which gives "authority" to a variable to use interfaces and implementations. What does this bring into benefit? Of course, you don't have to implement interfaces all the time in your classes to perform observation, or give values to a map etc. The simplest example is the lazy()function, which takes a lambda and returns a Lazy<T> which is the delegate that "magically" remembers your variable value but the memory is still not allocated. How great is that? I use lazy wherever I can:

```kotlin
private val namesAdapter by lazy {
        NamesAdapter() //value given, not stored
    }

fun startRecyclerView(){
 myRecyclerView.adapter = namesAdapter // this is where the variable comes in life.
}
```

I suggest checking Delegation not only for this case, but even your personal use cases, it's pretty powerful. Delegation is achieved using the keyword by.

**Functions:**

I personally believe methods is the most innovative way in Kotlin. Honestly, one could write books only with Kotlin methods and it would be a thick one. Therefore, a single article wouldn't be enough for methods, but I would state what is more important:

**Extension functions:**  
Extension functions is a way to apply methods to already made classes as part of them. For example, if your whole project needs a method which returns the number of characters in a Stringtimes 2 (don't judge my examples 😂)  you can create a method as part of the String class without creating a whole new class which extends String.

```kotlin
fun String.getDoubleLength(): Int = this.length * 2

//later on
val doubleLengthOfName = "Ben".getDoubleLength() //6 is asigned to doubleLengthOfName
```

After that, you are free to apply this method in every possible String inside you module.

**Lambdas, inline, noiline, crossinline functions: **

Many programming languages including Java, Python or Javascript take functions as parameters. So does Kotlin. But since Kotlin translates to Java, it might have some possible problems on the process. Since lambdas are not such new to programming by now, i wont stop at them but I suggest using them the best way that solves your problem.

Let's jump to inline functions with lambda implementation:

```kotlin
fun executeMyLambda(myLambda: () -> Unit) {
    myLambda() // method gets called
}

fun myOtherMethod() {
    print("Hello")
    executeMyLambda {
        print("My name is Ben")
    }
    print("Goodbye")
}
```

In kotlin perspective, there is nothing wrong with that. But Java disagrees:

```java
public void executeMyLambda(Function myLambda) {
    myLambda.invoke();
}

public void myOtherMethod() {
    System.out.print("Hello");
    executeMyLambda(new Function() {
        @Override
        public void invoke() {
            System.out.print("My name is Ben");
        }
    });
    System.out.print("Goodbye");
}
```

There is an instance of the Function() every time I use my executeMyLambda() method. So what does the inline keyword bring? It precisely prevents that from happening:

```kotlin
inline fun executeMyLambda(myLambda: () -> Unit) {
    myLambda() // method gets called
}

fun myOtherMethod() {
    print("Hello")
    executeMyLambda {
        print("My name is Ben")
    }
    print("Goodbye")
}
```

And in the Java part:

```java
public void myOtherMethod() {
    System.out.print("Hello");
    System.out.print("My name is Ben");
    System.out.print("Goodbye");
}
```

That's a really powerful tool. While sometimes you might need the oposite of the above case, to prevent inline function for happening, so you may provide noinline keyword before the lambda and you are good to go.

On the other hand, there might be some small problems during implementing those cases. For example, when you use the return statement inside the lambda which will break our execution of the higher method. So to prevent this we need to use the crossinline keyword before lambda. After that, even deliberately, you can't make a return statement inside it:

```kotlin
inline fun executeMyLambda(crossinline myLambda: () -> Unit) {
    myLambda() // method gets called
}

fun myOtherMethod() {
    print("Hello")
    executeMyLambda {
        print("My name is Ben")
        //cannot write return statement here
    }
    print("Goodbye")
}
```

**[Reified, a small bonus for syntactic sugar and rutime generics control.](https://kotlinlang.org/docs/reference/inline-functions.html#reified-type-parameters)**

Reified gives the ability to have control over generic types at runtime. I'm going to stick to the example from the official doc, with some small modifications.

```kotlin
fun <T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) { //you cannot do this. The T type is assigned after compilation therefore we can't make that check at runtime
        p = p.parent
    }
    return p as T?
}

//rewriting that method above
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? { // we are "forced" to use  a class type
    var p = parent
    while (p != null && !clazz.isInstance(p)) { //and check the right way
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}

//And also calling that method is a little ugly:
treeNode.findParentOf(AClassExtendingTreeNode::class.java)
```

Let's see how reified gives a hand to us:

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}

//calling the method:
treeNode.findParentOfType<MyTreeNode>()
```

So we lost a parameter, and now we can check in runtime which is the paret type of the given generic.

**Default parameters: **  
You are free to provide default parameters on Kotlin  for functions that not always will require some parameter:

```kotlin
    @GET
    suspend fun getLatestNewsAsync(
        @Url baseUrl: String,
        @Query("q") searchKeyWord: String,
        @Query("apiKey") apiKey: String,
        @Query("page") pageNumber: Int,
        @Query("pageSize") pageSize: Int = 20
    ): Response<NewsResponse>

 fun myMethod(){
   api.getLatestNewsAsync("data", "blablabla", 10)
 }

//somewhere below

 fun myMethod(){
   api.getLatestNewsAsync("data", "blablabla", 10, 30)
 }
```

Notice we are not providing a value for the pageSize on the first case, but we are providing another one in the next case.  
 **Infix and Operator Overloading**  
Kotlin methods marked as infix basically allow you to create your own keyword which on background is a method. It has strict rules, but it's a cool feature. For example, if you need a third instance of the object that holds only one String type variable inside, which is formed by concatinating names of 2 other users, you can do this:

```kotlin
data class User(var name: String?)

fun concatinateUserNames() {
    val user1 = User("Ben")
    val user2 = User("Jim")

    var concatinatedUserNames = user1 plus user2 //this plus operator will be marked with an error
}

fun User.plus(user: User): User {
    val newUser = User("")
    newUser.name = this.name + user.name
    return newUser
}
```

So how this is solved? You can directly apply the infix keyword on that plus method:

```kotlin
infix fun User.plus(user: User): User {
    val newUser = User("")
    newUser.name = this.name + user.name
    return newUser
}

//after that, the error on this line will leave:
var concatinatedUserNames = user1 plus user2
```

But what if we want to go further than that? The plus keyword looks nice, but not so elegant, and since we know we can concatenate two strings with the + operator, why not do it with these objects?

```kotlin
var concatinatedUserNames = user1 + user2 //runtime error
```

So all we need to do is just add the operator keyword to that plus() method and the error will leave:

```kotlin
operator fun User.plus(user: User): User {
    val newUser = User("")
    newUser.name = this.name + user.name
    return newUser
}

//this line will not be wrong anymore
var concatinatedUserNames = user1 + user2
```

Notice we must apply the [right spelling for this to happen](https://kotlinlang.org/docs/reference/operator-overloading.html).

**Scope functions: **  
_Note: Cases provided below, are not the only ones that can be applied to, please refer to the [docs](https://kotlinlang.org/docs/reference/scope-functions.html) for more info._

Kotlin has a sweet way of avoiding boilerplate or builders. It has some operators which can be applied to objects directly. Notice that operations are methods by themselves, but they apply to objects. I am taking the same example for all operations below:

So instead of doing:

```kotlin
val calendar = Calendar.getInstance()
calendar.set(Calendar.DAY_OF_MONTH, 10)
calendar.set(Calendar.MONTH, 1)
calendar.set(Calendar.YEAR, 1996)
... other value setting
```

We can do:

```kotlin
val calendarWithLet = Calendar.getInstance().let { calendar->
            calendar.set(Calendar.DAY_OF_MONTH, 10)
            calendar.set(Calendar.MONTH, 1)
            calendar.set(Calendar.YEAR, 1996)
        }
```

I stared with let operator since I find it more useful. The lambda method inside it is of type Calendar and the method returns the lambda result, in our case Unit. Current case does not fit well with let but the message is given.

**run**

What run does, is "creating a small class environment" inside our other classes. So basically the lambda that run returns, is the context of the Calendar.getInstance() , while the return type in this case is also Unit.

```kotlin
val calendarWithRun = Calendar.getInstance().run {
            set(Calendar.DAY_OF_MONTH, 10) //can also be this.set(Calendar.DAY_OF_MONTH, 10)
            set(Calendar.MONTH, 1)
            set(Calendar.YEAR, 1996)
        }
```

**with**  
with operator is very similar with run operator, but the only difference is that you provide the context as a parameter and not after the instantiation. Example:

```kotlin
val calendar = with(Calendar.getInstance()) {
            //we are inside just like in run
            set(Calendar.DAY_OF_MONTH, 10)
            set(Calendar.MONTH, 1)
            set(Calendar.YEAR, 1996)
        }
```

**apply**  
Apply perhaps is the best solution for our Calendar instantiation. In order to store the calendar value in a variable, apply comes a lot in handy if I want to reuse that instance. The lambda here returns the same as it does in run, the context, with a small difference: apply method returns a value which can be stored:

```kotlin
val calendarWithApply = Calendar.getInstance().apply {
            set(Calendar.DAY_OF_MONTH, 10)
            set(Calendar.MONTH, 1)
            set(Calendar.YEAR, 1996)
        }
```

And last but not least, the also operator. This little guy here, comes in handy when we already have some data initialized and we want to apply extra operations on them:

```kotlin
 val calendarWithApply = Calendar.getInstance().apply {
            set(Calendar.DAY_OF_MONTH, 10)
            set(Calendar.MONTH, 1)
            set(Calendar.YEAR, 1996)
        }.also {
            print(it.timeInMillis)
        }

//you can also do:

calendarwithApply.also{
  print(it.timeInMillis)
}
```

Our also here, has a lambda of type Calendar! which returns Unit, but the method here returns the Calendar type, making also and apply a perfect match.

**Data classes**  
Data classes are the sweet thing. So this is how you create a model in java:

```java
public class User {
    private String username;
    private String password;
    private String firstName;
    private String lastName;
    private Long registeredAt;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Long getRegisteredAt() {
        return registeredAt;
    }

    public void setRegisteredAt(Long registeredAt) {
        this.registeredAt = registeredAt;
    }
}
```

And this is all we have to do in Kotlin:

```kotlin
data class User(
    val userName: String,
    val password: String,
    val firstName: String,
    val LastName: String,
    val registeredAt: Long
)
```

Getters and setters included.

**Conclusion**

These were some of the many things you can do to think in Kotlin style instead of Java. This will help you clear a lot of boilerplate code as well as removing Java style of programming.  Even though Kotlin does translate to Java and is very similar, they are not the same thing. Therefore, applying these rules will come in handy in your career as a Kotlin developer.
