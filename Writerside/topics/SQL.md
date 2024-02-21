# SQL (MySQL, Postgres)

#### Getting started
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