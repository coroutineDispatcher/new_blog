---
title: "Getting started with Jetbrains Exposed, a Kotlin ORM framework"
date: 2022-02-27T15:07:18+01:00
draft: false
author: "Stavro Xhardha"
---

![](/images/exposed.png)

Even though my main focus is still Android, from time to time I try to play around with other stuff from the Kotlin world. Lately, I have been exploring KorGE (it's a game engine for Kotlin) and KTOR. But today's article is about KTOR, and more specifically, about the database layer. Kotlin and what is being built on top of it, is moving so fast, and I want to catch 'em all.

Anyways, today I wanted to write about Jetbrains Exposed, an ORM in Kotlin which can be easily integrated with Mysql, H2, MariaDB, Oracle, PostgreSQL, SQL Server, and SQLite.

## Setup

First of all, I would head to the [documentation](https://github.com/JetBrains/Exposed#supported-databases) page, which IMO it was a very good start. One thing I like about this framework is that it is very intuitive. As a database, I picked MySQL, since it's been my buddy since I was a student, even though it would not matter much whichever database management system you use.

{{< admonition >}}
Just remember that the framework hasn't reached 1.0 yet.
{{< /admonition >}}

### Dependencies

First installing exposed:

```
// gradle.properties
exposedVersion=0.36.1
```

Then just add the dependency in gradle

```kotlin
val exposedVersion: String by project

plugins {
    ...
}

dependencies {
    ...
    implementation("org.jetbrains.exposed:exposed-core:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-dao:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-jdbc:$exposedVersion")
    ...
}
```

### Playing around

To start, it's pretty standard, like most of the technologies when it comes to interacting with databases. Below is a very basic and quick setup:

```kotlin
fun Application.configureDatabase() {
    val database = Database.connect(
        url = "jdbc:mysql://<YOUR CONNECTION STRING>",
        user = "<YOUR DB USER>",
        password = "<YOUR DB PASSWORD FOR THAT USER>",
        driver = "com.mysql.jdbc.Driver",
    )

    transaction {
        addLogger(StdOutSqlLogger) // optional
        SchemaUtils.create(Table1, Table2)
    }
}
```

Except for the connections string, what is also to be noted is `SchemaUtils.create(Table1, Table2)`. Whenever you add a new table, it must not be forgotten to be added here. We will tackle tables in a little. However, I cheated about migrations; I just updated stuff directly from the database (this is a very basic example though).

### Tables

You would think of data classes here, but actually, it works with objects/singletons. That's because elements like `PrimaryKey()`or other objects are internally declared as `inner class`. Every object should extend `Table`class for it to be an actual table:

```kotlin
object Salaries : Table("table_name") {
    val id = varchar("id", 100).uniqueIndex()
    val name = varchar("name", 100)
    val lastName = varchar("last_name", 100)
    val fullName = varchar("full_name", 100)
    val payment = varchar("payment", 100)

    override val primaryKey = PrimaryKey(id)
}
```

What I found not very intuitive was the setting of the `PrimaryKey`. The variable is optional to override and I had to wrap my head around how this is done. Other than that, I believe the code is pretty straightforward. 

### DAO

DAOs (data access objects) are also straightforward, but it might make sense not to repeat ourselves, therefore I created an abstraction for that:

```kotlin
interface DAO<T> {
    fun selectAll(): List<T>
    fun select(id: String): T?
    fun insertOrIgnore(type: T)
    fun delete(id: String): Int
    fun deleteAll(): Int
}
```

Then it's easier to apply this in every DAO. But let's build a DAO for the `Salaries` table that we created above:

```kotlin
class SalariesDAO : DAO<Salary> {

    override fun selectAll(): List<Salary> = transaction { Salaries.selectAll().map { it.toSalary() } }

    override fun select(id: String): Salary? =
        transaction { Salaries.select { Salaries.id eq id }.map { it.toSalary() }.singleOrNull() }

    override fun insertOrIgnore(type: Salary) {
        transaction {
            Salaries.insertIgnore { salary ->
                //fields
            }
        }
    }

    override fun delete(id: String): Int = transaction { Salaries.deleteWhere { Salaries.id eq id } }

    override fun deleteAll(): Int = transaction { Salaries.deleteAll() }
}
```

A few things to note here. According to the project Wiki, every CRUD operation should run in a `transaction` block, which btw it doesn't accept `suspend` inline blocks but just normal functions. So no coroutines to talk about today. The syntax is DSL friendly so if you are very good at SQL already, you know what you are doing in Kotlin as well (well most likely).
The CRUD functions come from the `Table`object that we extended in the table setup above.

From here on, the database layer and the interaction with it, is basically complete. You can integrate this into your current architecture.

## Closing thoughts

What I didn't like about Exposed was that the developers should rely on Wiki instead of KDOC, and I don't know if there is any documentation coming soon. As for the [Wiki](https://github.com/JetBrains/Exposed/wiki) it is quite nice, but I am still not used to it. Working with exposed was easy, straightforward, and intuitive though. For playing around and for not-so-big projects, exposure should be the thing in the future.