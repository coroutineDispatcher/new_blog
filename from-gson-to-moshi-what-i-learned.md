---
title: "From Gson to Moshi, what I learned"
date: 2019-10-28T10:11:00.000+01:00
draft: false
aliases: ["/2019/10/from-gson-to-moshi-what-i-learned.html"]
tags: [Java, POJO, Gson, Kotlin, Json, Moshi, Android, Json Parsing]
author: "Stavro Xhardha"
---

[![](https://static.zerochan.net/Blue-Eyes.White.Dragon.full.2600091.gif)](https://static.zerochan.net/Blue-Eyes.White.Dragon.full.2600091.gif)

There is no doubt that people are getting away from GSON and I agree with [those reasons too](https://github.com/uber/shared-docs/blob/master/Moshi.md). The only advantage GSON has over other parsing libraries is that it takes a really short amount of time to set up. Furthermore, the most important thing is that Moshi is embracing Kotlin support.

First let's implement the dependency:

```kotlin
implementation("com.squareup.moshi:moshi:1.8.0")
```

It's not a struggle to migrate to Moshi. It's really Gson look-a-like. The only thing to do is annotate the object with @field:Json instead of `@SerializedName` (which is Gsons way for JS representation):

```kotlin
data class User( //GSON way
  @SerializedName("name")
  val name: String,
  @SerializedName("user_name")
  val userName: String,
  @SerializedName("last_name")
  val lastName: String,
  @SerializedName("email")
  val email: String
)

data class User( //Moshi way
  @field:Json(name = "name")
  val name: String,
  @field:Json(name = "user_name")
  val userName: String,
  @field:Json(name = "last_name")
  val lastName: String,
  @field:Json(name = "email")
  val email: String
)
```

Apparently, in order to solve a problem we are done, but we haven't unleashed the full power yet. Remember, this way we haven't still yet implemented the Kotlin support. With this, Moshi comes with some new dependency to add and an annotation processor for generating the adapters. Refer to the [docs](https://github.com/square/moshi) for more:

```kotlin
implementation("com.squareup.retrofit2:converter-moshi:2.4.0") //needed for retrofit integration when parsing
implementation("com.squareup.moshi:moshi:1.8.0") //core library
implementation("com.squareup.moshi:moshi-kotlin:1.6.0") //kotlin support
kapt("com.squareup.moshi:moshi-kotlin-codegen:1.8.0") // annotation processor, should have apply plugin: 'kotlin-kapt' above
```

**Default values**:
In Java, we have the transient keyword in order to use optional values for Moshi, while in Kotlin this is achieved though a `@Transient` annotation:

```kotlin
@Entity(tableName = "some_table_name")
@JsonClass(generateAdapter = true)
data class SomeEntity(
 @ColumnInfo(name = "some_id")
 @PrimaryKey(autoGenerate = true)
 @Transient
 //I need this field for my Room as an entity but definitely nothing is comming from the server. Mandatory to have a default value for Moshi
 val id: Int = 0,

    @Json(name = "audio")
    @ColumnInfo(name = "audio_url")
    val audioUrl: String,

    @Json(name = "text")
    @ColumnInfo(name = "text")
    val text: String,

)
```

If you notice more, the @field:Json is now just a @Json. And we have annotated the class with @JsonClass which helps the class to be encoded as JSON format

**Conclusion**
If you skipped the reasons why migrating from Gson to Moshi, I'm giving my own reason
\- Speed (I immediately noticed that even though in debug mode).
\- Kotlin support.
\- Proguard rules: If you choose only the Java version of Moshi, you won't need any pro-guard rules for release builds.

**Another option.**
If you find reasons not to like Moshi, I suggest take a look at [Kotlinx Serialization](https://github.com/Kotlin/kotlinx.serialization). IMO, it's a little too early to use it, but it surely looks promising.

Stavro Xhardha
