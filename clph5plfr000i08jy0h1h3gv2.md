---
title: "Server side kotlin using Ktor. An authentication example."
datePublished: Thu Jun 03 2021 16:35:09 GMT+0000 (Coordinated Universal Time)
cuid: clph5plfr000i08jy0h1h3gv2
slug: server-side-kotlin-using-ktor-an-authentication-example

---


Let’s be honest, I didn’t enjoy Ktor in the beginning. It seemed pretty confusing, different, immature, and difficult to start with. Watching it grow from away was a little simpler. However, I returned to ktor since I was interested more in server-side Kotlin. Kotlin is doing great, by the way, lots of improvements and new exciting features came up with [1.5.0 release](https://kotlinlang.org/docs/whatsnew15.html), which makes it the perfect time to consider using it not only on Android but also server side, desktop etc.

# Getting started

Before starting with Ktor, please note that I have no huge expertise on the platform, therefore, I might be killing flies with Bazookas. But let’s see if I get some nice feedback for this, which would be also valuable.

The first thing to do is to visit [start.ktor.io](https://start.ktor.io/#) , which is not different from the concept of creating a new spring (boot) project. For anyone not familiar with start.ktor.io, it’s a web form that generates your project according to your dependency requirements and technologies you want to use for each layer. As far as I know, you can do it from the IDE as well, but since it’s not the first project I create for Ktor, this one works pretty well for me. The screenshot below would help you even more:

![](/images/ktor_start.png)

After configuring your project, let it download and then open it from IntelliJ IDEA (probably possible with Android Studio as well, but never tried it). The cool advantage is that a Ktor project *by default* now starts with kts gradle files instead of groovy. Now let’s talk about what we are using for this plain example:

**Graphql as a technology (kgraphql):**

```kotlin
    implementation("com.apurebase:kgraphql:$kgraphql_version")
    implementation("com.apurebase:kgraphql-ktor:$kgraphql_version")
```

**Mongo DB for presistence (kmongo):**

```kotlin
    implementation("org.litote.kmongo:kmongo:$kmongo_version")
```

**Ktor authentication and JWT dependency for the token (since our example will consist in an authentication schema):**


```kotlin
    implementation("io.ktor:ktor-auth:$ktor_version")
    implementation("io.ktor:ktor-auth-jwt:$ktor_version")
```

**Not going much in detail, but Bcrypt for password encryption/decryption:**

```kotlin
implementation("at.favre.lib:bcrypt:$bcrypt_version")
```

**Koin for DI:**

```kotlin
implementation("org.koin:koin-ktor:$koin_version")
```

# Implementation

Before starting don't forget to add a configuration to run the server, as described in the picture below:

![](/images/configuration.png)

The environment variable is MongoDB connection string which is unique for every application/mongo db user.

## Persistence

Let's start with the data layer and model setup first. Basically, we need a `User` to be saved in the database: 

```kotlin
data class User(override val id: String, val email: String, val hashedPassword: ByteArray) : Model
```

Where `Model`, is just an abstract model which has one common property for every future model, like `id`. This also helps not to forget to implement such property for next
future models: 

```kotlin
interface Model {
    val id: String
}
```

However, let’s keep two more models for the user format, one for the graphql mutation input, and one for the response, which would hold token:

```kotlin
data class UserInput(val email: String, val password: String)
data class UserResponse(val token: String, val user: User)
```

With the model setup, we can now move to persistence logic. It would be also awesome to have some abstraction for a `Repository` since the logic to get item(s) by id would be the same for any database property, just like:

```kotlin
interface Repository<T> {
    var mongoCollection: MongoCollection<T>

    fun getById(id: String): T {
        return try {
            mongoCollection.findOne(Model::id eq id) ?: throw Exception("No item with that ID exists")
        } catch (t: Throwable) {
            throw PropertyNotFoundException("Cannot find item")
        }
    }

    fun getAll(): List<T> {
        return try {
            val result = mongoCollection.find()
            result.asIterable().map { it }
        } catch (t: Throwable) {
            throw Exception("Cannot get all items")
        }
    }

    fun delete(id: String): Boolean {
        return try {
            mongoCollection.findOneAndDelete(Model::id eq id) ?: Exception("No item with that Id exists")
            true
        } catch (t: Throwable) {
            throw Exception("Cannot delete item or item not found")
        }
    }

    fun add(entry: T): T {
        return try {
            mongoCollection.insertOne(entry)
            entry
        } catch (t: Throwable) {
            throw Exception("Cannot add item")
        }
    }

    fun update(entry: Model): T{
        return try {
            mongoCollection.updateOne(Model::id eq entry.id, entry)
            mongoCollection.findOne(Model::id eq entry.id) ?: throw PropertyNotFoundException("No item with that id exists")
        }catch (t: Throwable){
            throw Exception("Cannot update item")
        }
    }
}
```

This would help to have a very simple user repository:

```kotlin
class UserRepository(client: MongoClient) : Repository<User> {
    override lateinit var mongoCollection: MongoCollection<User>

    init {
        val database = client.getDatabase("databaseName")
        mongoCollection = database.getCollection<User>("User")
    }

    fun getUserByEmail(email: String? = null): User? {
        return try {
            mongoCollection.findOne(User::email eq email)
        } catch (t: Throwable) {
            throw Exception("Cannot get user with that email")
        }
    }
}
```

The method `getUserByEmail()` is added extra for this `User` databse property since in the generic Repository has nothing to do with an `email` property, but it's helpful to check if the user is already registered/can be logged in.

*Notice the nice Kotlin DSL being used by `KMongo`: `User::email eq email`.*

## Business logic

Let's now implement the data layer methods into an `AuthService` which would make our lives easier and we would also see after this step how easy it is to make queries or mutations after having such service:

```kotlin
class AuthService : KoinComponent {
    private val client: MongoClient by inject()
    private val repository: UserRepository = UserRepository(client)
    private val secret: String = "someHashedValueNobodyIsAbleToGuess"
    private val algorithm: Algorithm = Algorithm.HMAC256(secret)
    private val verifier: JWTVerifier = JWT.require(algorithm).build()

    fun signIn(userInput: UserInput): UserResponse? {
        val user = repository.getUserByEmail(userInput.email) ?: error("No such user by that email")
        if (BCrypt.verifyer()
                .verify(userInput.password.toByteArray(StandardCharsets.UTF_8), user.hashedPassword).verified
        ) {
            val token = signAccessToken(user.id)
            return UserResponse(token, user)
        }
        error("Password is incorrect")
    }

    fun signUp(userInput: UserInput): UserResponse? {
        val hashedPassword = BCrypt.withDefaults().hash(10, userInput.password.toByteArray(StandardCharsets.UTF_8))
        val id = UUID.randomUUID().toString()
        val emailUser = repository.getUserByEmail(userInput.email)

        if (emailUser != null) {
            error("Email already in use")
        }
        val newUser = repository.add(
            User(
                id = id,
                email = userInput.email,
                hashedPassword = hashedPassword
            )
        )

        val token = signAccessToken(newUser.id)
        return UserResponse(token, newUser)
    }

    fun verifyToken(call: ApplicationCall): User? {
        return try {
            val authHeader = call.request.headers["Authorization"] ?: ""
            val token = authHeader.split("Bearer ").last()
            val accessToken = verifier.verify(JWT.decode(token))
            val userId = accessToken.getClaim("userId").asString()
            return User(id = userId, email = "", hashedPassword = byteArrayOf())
        } catch (e: Exception) {
            print(e.message)
            null
        }
    }

    private fun signAccessToken(id: String): String {
        return JWT.create().withIssuer("example")
            .withClaim("userId", id)
            .sign(algorithm)
    }
}
```

JWT needs an algorithm by choice, where you pass it as a setup when you `signAccessToken` for every new `User`. A `verifytoken` also is needed as an extra security check on each call.

{{< admonition title="Note">}}
In case you are not familliar with JWT, [here](https://jwt.io/introduction]) is a quick intro for it. The article assumes the reader already knows the concept.
{{< /admonition >}}

If you notice, `signIn` and `signUp` have nothing different from usual such methods. The only thing worth mentioning is that in case you don't enjoy 3rd party security libraries, it might be fun to write your own security layer for it. However as far as I know, BCrypt is well trusted. 

{{< admonition title="Note">}}
Also note that nothing has to do with Ktor as a technology in particular yet.
{{< /admonition >}}

## Graphql : I don't need a waiter, I'll get my own drink in the bar!

I have been striving a lot to find a very simple example to explain Graphql to a 5-year-old. Needed to go back to when my career started, a few years ago, when I learned about REST with a simple example: The client orders a Pepsi. Then the waiter comes back and either give a Pepsi, or he responds: "Sorry sir, no Pepsi found, we have only Coke or if you like, we can offer you something else". Well, that's REST. 

But you know what? Who needs a waiter, I'll go to the bar myself and take a look at all the articles but I will be able to get one only after I order it. Great, that's graphql in a very basic 5-year-old example. For more technical explanation, please visit [here](https://graphql.org/).

{{< admonition title="Note">}}
We won't check Graphql in details either. Please visit the link above.
{{< /admonition >}}

Let's now implement the `Authentication` schema:

```kotlin
fun SchemaBuilder.authSchema(authService: AuthService) {

    type<User> {
        description = "User details"
        User::hashedPassword.ignore()
    }

    inputType<UserInput> {
        description = "User credentials"
    }

    mutation("signIn") {
        description = "Authenticate an existing user"

        resolver { userInput: UserInput ->
            try {
                authService.signIn(userInput)
            } catch (e: Exception) {
                null
            }
        }
    }

    mutation("signUp") {
        description = "Authenticate a new user"

        resolver { userInput: UserInput ->
            try {
                authService.signUp(userInput)
            } catch (e: Exception) {
                null
            }
        }
    }
}
```

Grphql is more straightforward than REST. You can read the code as a book: We are talking about a `User` `type` in this case which has a description and will never expose its password on responses. Then we have an `inputType` holding a `UserInput` which describes what we are expecting from the requests (practically, the bartender gives the client a menu, and he doesn't expect them to order a Pepsi when there is no Pepsi on the menu. The menu is the documentation which btw in graphql is generated automatically.)

Unfortunately, there are no `queries` for this use case, however, note that queries, from their name, are used to access properties, while mutation(s) is used to modify properties. Some might argue that we are not doing any mutation for the `signIn` resolver, however, we are signing a token for this new login therefore a mutation is more suitable.

A `resolver` is just a function used for the response values, the lambda defined is the input on the request.

{{< admonition title="Note">}}
The above structure is pure Kotlin DSL and it's very nicely achieved by KGraphql.
{{< /admonition >}}

Now that the setup is almost done, let's connect the dots...

# KTOR

If this was long, and now you are thinking that we just got started with KTOR, you are mistaken. Ktor is the simplest thing ever. All it has is just a few lines of code:

```kotlin
// Application.kt
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

@kotlin.jvm.JvmOverloads
fun Application.module() {
   // set up
}
```

But what do we set up? All we have to do is instantiate our dependency injection tool (koin-ktor) and install the GraphQL feature and all is done:

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

@kotlin.jvm.JvmOverloads
fun Application.module() {

    startKoin {
        modules(mainModule) // not important to show the implementation of mainModule
    }

    install(GraphQL) {
        val authService = AuthService()
        playground = true // This adds support for opening the graphql route within the browser
        context { call ->
            authService.verifyToken(call)?.let { +it }
        }
        schema {
            authSchema(authService)
        }
    }
}
```

Oh yea, the context. Being an android developer that was not hard to understand. The context is... Well, the context. I would like to call it a not expirable session, but you can also think of the context as a scope in general, which in this case is the app itself except the authentication process.
With the help of Kotlin DSL from KGraphql the library fits so much to the tech stack.

You may now run the application and visit the address Ktor gives you on the logs (which usually is http://0.0.0.0:8080/) and see the magic yourselves in `http://0.0.0.0:8080/graphql`. Then you can start playing:

![](/images/graphql_view.png)

Isn't Ktor awesome? It's just a function, where you install features, and as far as I know, it's all written in Kotlin on the background. I hope I made a quick and easy introduction to Ktor and an authentication process. This article was written thanks to a Udemy course that I took which can be found [here](https://www.udemy.com/course/kotlin-multiplatform-mobile/).
