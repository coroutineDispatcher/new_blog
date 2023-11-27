---
title: "Getting hands dirty with KotlinJs"
date: 2019-07-23T15:46:00.000+02:00
draft: false
aliases: ["/2019/07/getting-hands-dirty-with-kotlinjs.html"]
tags: [Html, Web development, KotlinJs, Javascript, Kotlin, Kotlinxhtml]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-WW-WcLldzXQ/XTcDGOIP3yI/AAAAAAAAOhQ/D4VIHRwfXZoMeFIlzyDvBEWQ-y17SjW_gCK4BGAYYCw/s1600/alice-achterhof-FwF_fKj5tBo-unsplash.jpg)](http://1.bp.blogspot.com/-WW-WcLldzXQ/XTcDGOIP3yI/AAAAAAAAOhQ/D4VIHRwfXZoMeFIlzyDvBEWQ-y17SjW_gCK4BGAYYCw/s1600/alice-achterhof-FwF_fKj5tBo-unsplash.jpg)

Although web development hasn't been my thing since I started programming, I was really attracted the way Kotlin does solve things. So I did give a try to KotlinJS.

**This is how to set up a KotlinJs page in less than an hour. **  
_Note: Chose to use the simplest set up without gradle or maven. You are free to chose what you like._  
First of all, you start with a new project in your IntelliJ and choose Kotlin -> Javascript.

[![](https://1.bp.blogspot.com/-Rblt4Wqz0AQ/XTcF1ilAr_I/AAAAAAAAOhY/gmHS0bDzg0UVzEEJSGhV2WBkrBbDPhikgCLcBGAs/s1600/Screenshot_3.png)](https://1.bp.blogspot.com/-Rblt4Wqz0AQ/XTcF1ilAr_I/AAAAAAAAOhY/gmHS0bDzg0UVzEEJSGhV2WBkrBbDPhikgCLcBGAs/s1600/Screenshot_3.png)

After that, you can name your project and start. The IDE will generate an empty folder for you.

So to start of, you have lots of options here, for example, you may program the html and css on your own, and than connect the dots with kotlin. Or, you can fully type in Kotlin (as I chose to do it) with this nice library called [Kotlinx Html](https://github.com/Kotlin/kotlinx.html).  
Since I'm not using gradle nor maven, I should import it as a JAR (which is not that cool, but for a demo project, no problem) 👎

**On with the show. **  
So what the app will do? The user inputs a name, and he gets back a [Chuck Norris](http://www.icndb.com/api/) joke with the name inside it.

[![](https://1.bp.blogspot.com/-ui27nHBSjbE/XTcJd_LR06I/AAAAAAAAOhk/vHRe0qRsXKMUOpJPoWDUTCFHKBIs9VQ0gCLcBGAs/s1600/Screenshot_4.png)](https://1.bp.blogspot.com/-ui27nHBSjbE/XTcJd_LR06I/AAAAAAAAOhk/vHRe0qRsXKMUOpJPoWDUTCFHKBIs9VQ0gCLcBGAs/s1600/Screenshot_4.png)

And after that:

[![](https://1.bp.blogspot.com/-h8afQiOZxZc/XTcJrfUn-FI/AAAAAAAAOho/ypgZbBFL2QsL8YpZunUmqzCHX1A5cLj3ACLcBGAs/s1600/Screenshot_5.png)](https://1.bp.blogspot.com/-h8afQiOZxZc/XTcJrfUn-FI/AAAAAAAAOho/ypgZbBFL2QsL8YpZunUmqzCHX1A5cLj3ACLcBGAs/s1600/Screenshot_5.png)

**The code:**

I found a nice implementation using the MVP pattern (not sure if it is the right one on the web) [here](https://www.raywenderlich.com/201669-web-app-with-kotlin-js-getting-started#toc-anchor-002). So I set up the interfaces first.

```kotlin
interface JokeContract {
    interface View {
        fun showJoke(jokeValue: String)
        fun showLoader()
        fun hideLoader()
    }

    interface Presenter {
        fun attach(view: View)
        fun loadJokes(input: String)
    }
}
```

After that I create the View implementing class and the Presenter implementer class:

```kotlin
class JokesPresenter : JokeContract.Presenter {
    private lateinit var view: JokeContract.View

    override fun attach(view: JokeContract.View) {
        this.view = view
    }

    override fun loadJokes(input: String) {
        view.showLoader()
        getAsync("https://api.icndb.com/jokes/random?firstName=$input&escape=javascript") {
            val joke = JSON.parse<RandomJokeResponse>(it)

            view.hideLoader()

            view.showJoke(joke.value.joke)
        }
    }

    private fun getAsync(url: String, callback: (String) -> Unit) {
        val xmlHttp = XMLHttpRequest()
        xmlHttp.open("GET", url)
        xmlHttp.onload = {
            if (xmlHttp.status == 200.toShort()) {
                println(xmlHttp.responseText)
                callback.invoke(xmlHttp.responseText)
            }
        }
        xmlHttp.send()
    }
}
```

Notice the API call is made with XMLHttpRequest, inside the getAsync method. That's the object responsible for API calls. When response successful, we parse the response to our RandomJokeResponse object and let the view handle the rest.

```kotlin
class JokesPage(private val presenter: JokeContract.Presenter) : JokeContract.View {
    val root = document.getElementById("root")
    private var loaderdiv: HTMLElement = document.create.div {
        p {
            +"Loading..."
        }
    }

    init {

        val div = document.create.div {
            h3 {
                +"Place your name and get a random joke"
            }

            form {
                id = "form"
                input {
                    id = "name"
                    type = InputType.text
                }
                input {
                    type = InputType.submit
                }

                onSubmitFunction = { event ->
                    val element = document.getElementById("name") as HTMLInputElement
                    if (element.value.isNotEmpty())
                        presenter.loadJokes(element.value)
                    else
                        window.alert("Please place some name")
                    event.preventDefault()
                }
            }
        }
        root!!.appendChild(div)
    }

    override fun showJoke(jokeValue: String) {
        val text = document.create.div {
            h3 {
                +jokeValue
            }
        }
        root!!.appendChild(text)
    }

    override fun showLoader() {
        root!!.appendChild(loaderdiv)
    }

    override fun hideLoader() {
        root!!.removeChild(loaderdiv)
    }
}
```

The HTML tags are not constructed as normally thanks to KotlinxHTML, but they are written in Kotlin(DSL).

**But the browser doesn't speak Kotlin! The connecting dot.**

```kotlin
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Name Joke</title>
</head>
<body>
<div id="root"></div>
<script type="text/javascript" src="out/production/RandomJoke/lib/kotlin.js"></script>
<script type="text/javascript" src="out/production/RandomJoke/lib/kotlinx-html-js.js"></script>
<script type="text/javascript" src="out/production/RandomJoke/RandomJoke.js"></script>
</body>
</html>
```

It certainly needs an HTML file, with some small configuration. On the scripting tags I connect the generated files inside the out directory and libraries that I might include. At that's all sufficient for the browser. Now you may continue to type KOTLIN.

After that, you are free to create a class whatever the name is, and write the main method, so that the View page will be constructed. Since I'm a spring fan, I add the \`Application\` suffix:

```kotlin
fun main() {
    val jokesPresenter = JokesPresenter()
    val jokesPage = JokesPage(jokesPresenter)
    jokesPresenter.attach(jokesPage)
}
```

Done! You may check the page here:

[Name Norris Joke KotlinJS](https://coroutinedispatcher.github.io/name_joke/index.html)

And full project on:

[Name Norris Joke KotlinJS Repository](https://github.com/coroutinedispatcher/name_joke)

**Conclusion**  
Kotlin is the language that brought so innovative ways of solving problems. It's growing and improving every day. You may deal nearly everywhere with it: Android, Server-Side, Frontend and even IOS with Kotlin Multiplatform.

@import url('https://cdn.rawgit.com/lonekorean/gist-syntax-themes/848d6580/stylesheets/monokai.css'); @import url('https://fonts.googleapis.com/css?family=Open+Sans'); body { margin: 20px; font: 16px 'Open Sans', sans-serif; } Good luck playing with it.
