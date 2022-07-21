### Less known but useful Kotlin functions

**REMEMBER :**

> *less the code less the bugs*
> 
> 
> *less the code less the maintenance*
> 
> *less the cod less the time consumed*
> 
> *less the code less hairpulling*
> 

## **`find()`**

`find()` is used to find an element in a collection, it takes a predicate returns first element matching the condition, if no element is found, it will be null

For example, you have a list of users and you want to find user with the id:

***Without `find()`***:

```kotlin
fun findUser(id:Int):User?{
 for(i in 0..users.size - 1){
     if(users[i].id == id){
        return users[i]
     }
 }
 return null
}
```

***With `find()`***

```kotlin
val user = users.find{ it == id}
```

## **`filter()`**

`filter()` is used as name suggests filter some elements from a collection based on a condition will return empty list if not element match the condition filtered list otherwise.

for example you want to find users who has hobby of reading from a list of users

***Without `filter()`***

```kotlin
fun getReadingUsers():List<User>{
 val readingUsers = mutableListOf<User>()
 for(i in 0..users.size - 1){
     if(users[i].hobby == "reading"){
        readingUsers.add(users[i])
     }
 }
 return readingUsers
}
```

***with `filter()`***

```kotlin
data class User(
	val name: String,
    val hobby: String
)

fun main() {
    val users = listOf(
        User("Behzod","Reading"),
        User("Sardor","Running"),
        User("Teshaqul","Reading"),
        User("Eshmamat","Running")
    )
    
    val readingUsers = users.filter {
        it.hobby == "Reading"
    }
    
    println("Reading users ${readingUsers}")
    
}
```

## **`filterNot()`**

filter not is just opposite of `filter()` it will filter all elements from a list who does not match the given condition.

for example in a game you want to send price to the users who won so you will find winners

***Without `filterNot()`***

```kotlin
fun getWinners():List<User>{
 val winners = mutableListOf<User>()
 for(i in 0..users.size - 1){
     if(users[i].won == true){
        winners.add(users[i])
     }
 }
 return winners
}

data class User(
	val name: String,
	val teamName: String,
	val won: Boolean
)

fun main() {
	val users = listOf(User())	

}
```
