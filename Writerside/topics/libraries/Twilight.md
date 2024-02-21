
# Twilight ✨

Twilight is an API for developers creating plugins for Spigot or Paper based Minecraft servers. It contains a wide range of utilities and QOL improvements, from inventories, to schedulers and databases.

Twilight is built using **Kotlin**, and is recommended for usage with.

If you have any questions or need any support, head over to the [Flyte Discord](https://discord.flyte.gg)!

## Features

Twilight takes advantage of Kotlin's extension functions to add additional functions to various classes used within the API. Many of these are convenience functions, and some add complete new functionality. To see all of the functions added, view them [inside the code](https://github.com/flytegg/twilight/tree/master/src/main/kotlin/gg/flyte/twilight/extension).

### Scheduler
Bukkit's built in scheduler is tedious at best, so Twilight takes advantage of beautiful Kotlin syntax to make it easier to write, as well as adding a custom TimeUnit to save you calculating ticks.

How to schedule a single task to run on Bukkit's main thread either sync or async:
`
```kotlin
sync {
    println("I am a sync BukkitRunnable")
}

async {
    println("I am an async BukkitRunnable")
}
```

How to schedule a delayed task, with optional custom time unit and async parameters:

```kotlin
// SYNC
delay {
    println("I am a sync BukkitRunnable delayed by 1 tick")
}

delay(10) {
    println("I am a sync BukkitRunnable delayed by 10 ticks")
}

delay(1, TimeUnit.SECONDS) {
    println("I am a sync BukkitRunnable delayed by 1 second")
}

// ASYNC
delay(10, true) {
    println("I am an async BukkitRunnable delayed by 10 ticks")
}

delay(1, TimeUnit.SECONDS, true) {
    println("I am an async BukkitRunnable delayed by 1 second")
}
```

How to schedule a repeating task, with optional custom time unit and async parameters:
```kotlin
// SYNC
repeat(10) {
    println("I am a sync BukkitRunnable running every 10 ticks")
}

repeat(10, TimeUnit.SECONDS) {
    println("I am a sync BukkitRunnable running every 10 seconds")
}

repeat(5, 10) {
    println("I am a sync BukkitRunnable running every 10 ticks, waiting 5 before starting")
}

repeat(5, 10, TimeUnit.SECONDS) {
    println("I am a sync BukkitRunnable running every 10 seconds, waiting 5 before starting")
}

// ASYNC
repeat(10, true) {
    println("I am an async BukkitRunnable running every 10 ticks")
}

repeat(10, TimeUnit.SECONDS, true) {
    println("I am an async BukkitRunnable running every 10 seconds")
}

repeat(5, 10, true) {
    println("I am an async BukkitRunnable running every 10 ticks, waiting 5 before starting")
}

repeat(5, 10, TimeUnit.SECONDS, true) {
    println("I am an async BukkitRunnable running every 10 seconds, waiting 5 before starting")
}
```

> Twilight `repeat` conflicting with Kotlin's `repeat`? As an alternative, you can use `repeatingTask`.

## Databases
### MongoDB
Currently we have support for MongoDB. To configure it, you can take one of two routes:

#### Environment variables
You can use the following Environment variables for your MongoDB:
```env
MONGO_URI="your URI string"
MONGO_DATABASE="your database name"
```

#### Builder
When building your Twilight instance, you can specify your URI and database like so:
```kotlin
val twilight = twilight(plugin) {
    mongo {
        uri = "your URI string"
        database = "your database name"
    }
}
```

From here you can use the following function to get a collection from your database:
```kotlin
MongoDB.collection("my-collection")
```
And use the standard features of the Mongo Sync Driver with your `MongoCollection`.

**OR** you can use some of our custom features, making communicating with a Mongo database infinitely easier. Here's how you do it:

```kt
class Profile(
    @field:Id val id: UUID,
    val name: String
) : MongoSerializable
```

What's happening here? We're declaring what should be used as the key identifier for our class in the database, we can do so by annotating a field with `@field:Id`.

We also implement an interface `MongoSerializable`. This gives us access to a bunch of methods which make our lives really easy when it comes to moving between our class instance and our database.

For example, I could do the following:
```kt
val profile = Profile(UUID.randomUUID(), "Name")

profile.save()
// this returns a CompletableFuture of the UpdateResult
// or we could do profile.saveSync() if we need it to be sync, does not return wrapped by a CompletableFuture

profile.delete()
// this returns a CompletableFuture of the DeleteResult
// same as save, can do profile.deleteSync() for sync, does not return wrapped by a CompletableFuture
```

If we ever want to find and load and instance of our class from the database, we can use some functions from the TwilightMongoCollection:

```kt
val collection = MongoDB.collection<Profile>() // by default this assumes the name of the collection is the plural camel case of the type, f.x. Profile -> profiles, SomeExampleThing -> someExampleThings
// you can specify the name of the collection if you wish it to be different like so
val collection = MongoDB.collection<Profile>("myCollection")
collection.find() // returns a CompletableFuture<MongoIterable<Profile>>
collection.find(BsonFilter) // returns a CompletableFuture<MongoIterable<Profile>>
collection.findById(id) // id must be the same type as the field marked as the id on the class, returns a CompletableFuture<MongoIterable<Profile>>
collection.delete(BsonFilter) // returns a CompletableFuture<DeleteResult>
collection.deleteById(id) // id must be the same type as the field marked as the id on the class, returns a CompletableFuture<DeleteResult>
// all of these have sync versions which follow the same pattern, f.x. collection.findSync(), where the return value is the same as the async version, just not wrapped by a CompletableFuture
```

If we need something that isn't already wrapped by the TwilightMongoCollection, it exposes us the MongoCollection of Documents, which we can get with `collection.documents`.

### SQL (MySQL, Postgres)

#### Getting Started
To get started you need to create an instance of the SQL Wrapper like so
```kotlin
val db = SQLWrapper(url = "your db url", user="user", password="password")
db.connect()
```

#### QueryBuilder
The QueryBuilder class will help you in creating everything from simple queries like SELECT's to even complex JOIN's.
All you need to start is an instance of the query builder, here's an example usage
```kotlin
val queryBuilder = QueryBuilder()

// Example SELECT query
val selectQuery = queryBuilder.select("id", "name").from("person").where("age" gt 18).build()

// Example INSERT query
val insertQuery = queryBuilder.insertInto("person", "id", "name").values(1, "John").build()

// Example UPDATE query
val updateQuery = queryBuilder.update().table("person").set("name", "John Doe").where("id" eq 1).build()

// Example DELETE query
val deleteQuery = queryBuilder.delete().table("person").where("id = 1").build()
```
#### Using objects in your database
If you would like to retrieve and store data as objects within your database there a some methods provided for this


1 - Your object must implement SQLSerializable
2 - You must have a table that fits the structure of your object, you can create by calling `convertToSQLTable()` on your object, then execute the statement like so
```kotlin
// NOTE: convertToSQLTable() takes a optional dialect parameter, at this time the only additional dialect is postgres
val createTable = yourObjectInstace.convertToSQLTable()

if(db.execute(createTable)) {
    // successfully executed
}
```
3 - To insert your object call `toInsertQuery()` like so
```kotlin
val insertToTable = yourObjectInstace.toInsertQuery()

if(db.execute(insertToTable)) {
    // successfully executed
}
```

4 - To retrieve objects from your database you call a select statement like normal but call `toListOfObjects<Type>()` on the returned `Results` class

#### Running queries
Once you have your query using either the QueryBuilder or your own you can run it like so
```kotlin
val result = result.executeQuery(selectQuery)
```
once you have run the query it will return a `Results` class, it can be used like so
```kotlin
result?.let { res ->
  println("MyColumn Value: " + res["my_column"])
}
```
The results class contains a list of all the rows and columns returned by the database.
You can access any of the columns values the same way you would with a map.

### Ternary Operator
There is a basic ternary operator implementation added which can be used like so:
```kotlin
val test = false
println(test then "yes" or "no")
```
This doesn't yet work for evaluating functions either side of the ternary though, we plan to figure this out in the near future.

### UUID ⟷ Name
Twilight can do the heavy lifting and query the Mojang API to find the UUID from name or name from UUID of a player, particularly useful for networks. Twilight will cache responses in an attempt to not break the rate limit imposed by Mojang.

If you have a UUID and you want to get a name, you can call `nameFromUUID`:
```kotlin
NameCacheService.nameFromUUID(UUID.fromString("a008c892-e7e1-48e1-8235-8aa389318b7a"))
```
This will look up your cache to see if we already know the name, otherwise we will check the MongoDB "cache" of key, value pairs, and finally, we'll query Mojang if we still don't know it.

After each step the key, value pair will be stored so the next call is just on the cache.

Similarly, if you have a name and want to get the UUID, you can call `uuidFromName`:
```kotlin
NameCacheService.uuidFromName("stxphen")
```

Currently the only way to configure your MongoDB "cache" for UUIDs and names, is to have an Environment variable called `NAME_CACHE_COLLECTION` with the value being what you want to call the collection. Don't want to use the Mongo cache? Disable `useMongoCache` in the settings.

### Files Extensions

#### File.hash()
You can easily get the hash of a file using this method. The parameter `algorithm` is used to define which should be used for the hash, the default is SHA-256.

If you need to hold a reference to a file at a remote location you can use the `RemoteFile` class provided here. This allows you to easily get the hash of a remote file with `RemoteFile#hash`.

This is particularly useful when used in parallel with our other open-source library, [resource-pack-deploy](https://github.com/flytegg/resource-pack-deploy), so you can get the hash of the latest resource pack file, and use it to reset a client's cached version of your resource pack if there are any differences.

### Symbols
Twilight offers a collection of widely used symbols within the `Symbols` object.

These include, but are not limited to:
- • (BULLET)
- ❤ (HEART)
- ★ (STAR)
- » (SMALL_RIGHT)
- █ (SQUARE)
- ⬤ (CIRCLE)
- ⛏ (PICKAXE)
- ⛀ (COIN)
- ⛁ (COINS)

### Libraries
Twilight is bundled with some useful libraries, some include:
- Bson
- Dotenv
- Mongo Sync Driver
- GSON

#### GSON
We're aiming to provide some standard GSON Type Adapters for ease of use. Currently, we have adapters for the following:
- Location

We have a useful exclusion strategy, allowing you to exclude marked fields of a class from being in serialized JSON. All you need to do is annotate the class field with `@Exclude`!

Make sure to use our `GSON` import rather than making your own/own builder, as the Gson instance has to declare the ExclusionStrategy.
