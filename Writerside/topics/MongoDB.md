# MongoDB

### MongoDB
Currently we have support for MongoDB. To configure it, you can take one of two routes:

#### Environment variables
You can use the following Environment variables for your MongoDB:
```Generic
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

```Kotlin
class Profile(
    @field:Id val id: UUID,
    val name: String
) : MongoSerializable
```

What's happening here? We're declaring what should be used as the key identifier for our class in the database, we can do so by annotating a field with `@field:Id`.

We also implement an interface `MongoSerializable`. This gives us access to a bunch of methods which make our lives really easy when it comes to moving between our class instance and our database.

For example, I could do the following:
```Kotlin
val profile = Profile(UUID.randomUUID(), "Name")

profile.save()
// this returns a CompletableFuture of the UpdateResult
// or we could do profile.saveSync() if we need it to be sync, does not return wrapped by a CompletableFuture

profile.delete()
// this returns a CompletableFuture of the DeleteResult
// same as save, can do profile.deleteSync() for sync, does not return wrapped by a CompletableFuture
```

If we ever want to find and load and instance of our class from the database, we can use some functions from the TwilightMongoCollection:

```Kotlin
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
