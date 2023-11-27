---
title: "The Great Wall of China was originally created to keep WebView out. It failed miserably."
date: 2020-08-07T21:06:25+02:00
draft: false
author: "Stavro Xhardha"
---

![Photo by halacious @ Unsplash](/images/webview_post.jpg)

## onCreate

Let's face it, getting a `NaN` in Android is more frustrating than a `(Kotlin)NullPointerException`. Well, as a Native Android developer, I actually never touched the web professionally, except playing around with Vue js when people seemed pretty hyped for it. I tried to be a web developer only once in my life, but even then, I cheated, I used KotlinJS.

Anyways, many mobile developers have been in situations when they had to write a `WebViewFragment` and load an URL. End of story. But some of them, might have had more difficult cases, and believe me, it's somehow painful. I also had some pain with `WebView`s lately, so I thought I should share my experience, so we can save time for a lucky developer next time. Let's go through some cases, which they consumed more time in search for solution, rather than in actual development.

## Adding your own cookies in a WebView

There are cases that you need to interact with the server using cookies. And a webpage as well, cannot be loaded without these cookies. So, how do you solve this issue? Well, let's suppose you cached the cookies in`SharedPreferences`(the source of data doesn't really matter):

&nbsp;

```kotlin
val cookiesString = sharedPrefs.getCookies()
```

&nbsp;

In order to modify the default `WebView` cookies, you need a `CookieManager`. I guess the name is pretty descriptive for what this class is for. However, the Android documentation gives this explanation about it: &nbsp;

{{< admonition tip "CookieManager">}}
Manages the cookies used by an application's WebView instances.
{{</ admonition>}}

{{< admonition >}}
For simplicity, let's assume that we have only one pair of cookies
{{</ admonition>}}

### So, let's apply cookies:

```kotlin
val url = MyAwesomeUrl
val cookieManager = CookieManager.getInstance().apply{
    setAcceptThirdPartyCookies(webView, true)
    setCookie(url, cookie) { cookieSet ->
        if(cookieSet)  webView?.loadUrl(url)
    }
}
```

&nbsp;

The most important part that is missed in almost all Stack Overflow answers is this particular callback. Since we have to interact with the internal phone memory to set our preferred cookies, we have to wait for some answer. And this is one of the cases. After that, you can easily load the URL.

### Loading your HTML file from device internal storage or assets

This is another "special" case that I faced (even though not so confusing). Well, I had a couple of HTML files which I had to download from the server. In case they fail to download, had to load a default one, which is supposed to be based in the `assets` folder. So, let's quickly show the code for both of them:

&nbsp;

```kotlin
// loading from internal app storage
val directory = "${MyApp.component().filesDir.absoluteFile}/someDir"
val htmlFile = File( "$directory/my_downloaded_html_file.html").readText()

mWebView.apply{
    settings.javascriptEnabled = true
    loadDataWithBaseUrl("file://$directory", htmlFile,"text/html","base64",null)
}
```

&nbsp;

For `assets` folder, it's exactly the same logic, but instead of the internal app storage, the `requireActivity().assets.open("htmle_file_name.html")` is required.

Uh, but there is more.....

### Executing JavaScript methods

In case you need to execute a Javascript method, which lies somewhere magically inside the `<script>` tag of the rendered HTML file, that's where work begins. Well, in my case, once the WebView was rendered, I had to immediately call a JS function inside. So, what is the case for that?

First, we need to be sure that the `WebView` has finished loading everything we said it to. Therefore, a callback is needed:

&nbsp;

```kotlin
 mWebView.webViewClient = object : WebViewClient() {
    override fun onPageFinished(view: WebView?, url: String?) {
        view?.evaluateJavascript("javascript:doSomething()", null)
    }
}
```

&nbsp;

The lucky part, is that I wasn't expecting for a return value here. But that doesn't mean I didn't have to write a JavaScript interface. I did, but for another case.

### Extracting current page content

Let's consider this problem: Extract the `WebView` content, if it's a JSON, serialize and save it, then proceed to the next fragment. Otherwise, take the user one step back. For our scope, the only important part, would be: Extract the content from the `WebView`. Let's suppose we loaded some url and the content is rendered.

&nbsp;

```kotlin
webView.apply{
    addJavascriptInterface(WebContentJavascriptInterface(requireActivity()), "ContentReader")
}
```

&nbsp;

In this case, the `"ContentReader"`, is just a name to expose the object in JavaScript. So then, I can do:

&nbsp;

```kotlin
// inside the webView context
addJavascriptInterface(WebContentJavascriptInterface(requireActivity()), "ContentReader")
webViewClient = object : WebViewClient() {
                override fun onPageFinished(view: WebView?, url: String?) {
                    loadUrl(
                        "javascript:ContentReader.showWebViewContent" +
                                "('<html>'+document.getElementsByTagName('html')[0].innerHTML+'" +
                                "</html>');"
                    )
                    super.onPageFinished(view, url)
                }
            }
```

&nbsp;

So the method is executed, but we still have no clue where our `showWebViewContent` is. Our JavascriptInterface is still missing:

&nbsp;

```kotlin
inner class WebContentJavascriptInterface(private val context: Context) { // you need a context for this case
        @JavascriptInterface
        fun showWebViewContent(webViewContent: String) {
            val extractedTextFromHtml = HtmlCompat.fromHtml(
                webViewContent,
                HtmlCompat.FROM_HTML_MODE_LEGACY
            )
            viewModel.tryParsing(extractedTextFromHtml)
        }
    }
```

&nbsp;

Notice that our `showWebViewContent` is the same in Kotlin and in our string that is supposed to execute in JavaScript. However, the magic happens with the `@JavascriptInterface` annotation. And that is the most important part to note here. It is precisely this annotation, that allows you to expose whatever you like from your Kotlin code to JavaScript.

## OnDestroy

Working with `WebView`s is not that common in Android development. But when it happens, it gets very abstract for the developer. It somehow feels like switching platforms. The problems faced are not actually pretty hard to solve, but they are so uncommon and there is not too much structured information about them. Therefore, I hope this article helps everybody in the future, who has to deal with similar cases with `WebView`s.

Stavro Xhardha
