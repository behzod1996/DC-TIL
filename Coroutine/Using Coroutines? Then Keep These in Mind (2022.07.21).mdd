# Using Coroutines? Then Keep These in Mind (2022.07.21)

## **Think of coroutines as light-weight threads**

Coroutines are ***less resource-intensive compared to threads.***
When dealing with a handful of coroutines/threads this difference may not be obvious but when dealing with say 100000 
them you will witness the difference. 
Code that exhausts the available memory of JVM when using threads can be expressed using coroutines without reaching the memory limit. 
For example, the following code launches 100000 coroutines that each wait 3 seconds and then print a message while consuming a reasonable amount of memory.

```kotlin
fun main() = runBlocking {
    repeat(100_000) {
        launch {
            delay(3000L)
            println("done!")
        }
    }
}
```

Writing the above piece of code using threads is as follows:

```kotlin
fun main() {
    repeat(100_000) {
        thread {
            Thread.sleep(1000L)
            println(".")
        }
    }
}
```
Running the above code consumes too much memory and will probably throw an `OutOfMemoryError` exception if your machine does not have a huge amount of 
RAM available.

## **Each coroutine has a scope**

Each coroutine has a scope (which is of type`CoroutineScope`). In other words, you cannot start a coroutine without specifying its scope. The scope determines the lifetime of the coroutine. Also, the scope allows the so-called [structured concurrency](https://kotlinlang.org/docs/coroutines-basics.html#structured-concurrency).

As an example consider Android’s `ViewModel` class. A `ViewModel` has a coroutine scope named `viewModelScope` and its lifecycle is tied to the `ViewModel`. Being tied to the `ViewModel`’s lifecycle means that if you launch coroutines using the `viewModelScope` and the `ViewModel` is destroyed then all those coroutines are canceled automatically and there is no need to be worried about resource-wasting, memory leaks, etc.

## **Each coroutine has a context**

Each coroutine has a context (which is of type `CoroutineContext`). Coroutine context is a set of elements. Think of the elements as the properties of a coroutine like its name, dispatcher, job, etc. Some of the classes/interfaces that implement `CoroutineContext` are:

- `CoroutineName`: User-specified name of coroutine. This name is used when debugging coroutines.
- `CoroutineDispatcher`: Determines the dispatcher which is used to run the coroutine.
- `Job`: Provides a couple of useful operations that can be done on a coroutine like canceling it.

When you create a coroutine you can provide its context. If you do not specify the context of a coroutine, the context will have a default value of `EmptyCoroutineContext`.

## **Each coroutine has a context**

Each coroutine has a context (which is of type `CoroutineContext`). Coroutine context is a set of elements. Think of the elements as the properties of a coroutine like its name, dispatcher, job, etc. Some of the classes/interfaces that implement `CoroutineContext` are:

- `CoroutineName`: User-specified name of coroutine. This name is used when debugging coroutines.
- `CoroutineDispatcher`: Determines the dispatcher which is used to run the coroutine.
- `Job`: Provides a couple of useful operations that can be done on a coroutine like canceling it.

When you create a coroutine you can provide its context. If you do not specify the context of a coroutine, the context will have a default value of `EmptyCoroutineContext`.

```kotlin
fun main() = runBlocking {
    launch(Dispatchers.IO + CoroutineName("TheMostUsefulCoroutine")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}
```
## **A coroutine is not bound to a specific thread**

In general, a coroutine is not bound to any particular thread.**(Coroutine - hech qanday threadga bo’glanmangadir).** 

It may suspend its execution in one thread and resume in another one. This means that there is a pool of threads available and at each point in time a potentially different thread from this pool could be responsible for running the coroutine code.

In order to see this behavior in action, we will use the [Unconfined](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#unconfined-vs-confined-dispatcher) dispatcher. Consider the following code:

```kotlin
fun main() = runBlocking {
    launch(Dispatchers.Unconfined) {
        println("Before delay. Thread name: ${Thread.currentThread().name}")
        delay(500)
        println("After delay. Thread name: ${Thread.currentThread().name}")
    }
}
```

## ****Parent-Child Relationship****

### ****Child coroutines****

When a coroutine is launched in the scope of another coroutine, the new coroutine becomes the child of the parent coroutine.

```kotlin
fun main() = runBlocking {
    launch(CoroutineName("parent")) {
        launch(CoroutineName("child1")) {
            delay(1000)
            println("coroutine1 is done")
        }
        launch(CoroutineName("child2")) {
            delay(2000)
            println("coroutine2 is done")
        }
    }
}
```

In the above code, we launch two coroutines (`child1` and `child2`) in the scope of another coroutine (`parent`). This makes `child1` and `child2` the children of `parent`. When a coroutine becomes the child of another coroutine a couple of things happen. One such thing is that the child coroutine inherits the `CoroutineContext` of its parent.

## **Parent coroutine always waits**

A parent coroutine always waits for the completion of all its children. A parent coroutine does not have to explicitly track all the children it launches, and it does not have to call `join()` on the `Job` of each child to wait for their completion.

```kotlin
fun main() = runBlocking {
    val request = launch(CoroutineName("request")) {
        repeat(3) { i ->
            launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done")
    }
    request.join()
    println("Processing of the request is complete")
}
```

```
Now processing of the request is complete

request: I'm done

Coroutine 0 is done

Coroutine 1 is done

Coroutine 2 is done
```

In the above code, we launch 3 coroutines inside the `request` coroutine. This effectively means that those 3 coroutines become the children of the`request` coroutine. In order to wait for all the children to finish we just call `request.join()`and the `request` coroutine will wait for all its children to finish.

# **Cancellation**

## **Cancellation is cooperative**

A coroutine has to cooperate to be cancellable. 

**(”Coroutine” - bekor qilish qobiliyatiga ega bo’lish uchun, birgalikda hamkorlik qilish kerak.)**

For example, if a coroutine is doing a CPU-intensive task and is not checking for its cancellation it continues its execution even if we call `cancel()` on its `Job`.

```kotlin
fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) {
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

```
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
job: I'm sleeping 3 ...
job: I'm sleeping 4 ...
main: Now I can quit.
```

As can be seen after 1.3 seconds we call `job.cancelAndJoin()` to cancel the coroutine but the coroutine does not respect the cancellation and continues to print `job: I’m sleeping 3 …` and `job: I’m sleeping 4…`

To make the previous code cancellable we should use the `isActive` property which is an extension property available inside the coroutine.

```kotlin
fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5 && isActive) { // <-- HERE!
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

The only change that we made was to replace `while ( i < 5)` with `while(i < 5 && isActive)`. Now if you run the above code the output will be like the following.

## **Canceling a parent coroutine cancels its children**

When a parent coroutine is canceled, all its children are canceled too.

```kotlin
fun main() = runBlocking {
    val job = launch(CoroutineName("parent")) {
        launch {
            delay(2000)
            println("coroutine1: I'm done")
        }
        launch {
            delay(3000)
            println("coroutine2: I'm done")
        }
    }
    println("Cancelling the parent coroutine...")
    job.cancelAndJoin()
    println("Parent coroutine cancelled")
}
```

## **Canceling a child coroutine does not affect its siblings or its parent**

When a child coroutine is canceled, it does not affect the execution of its parent or siblings.

```kotlin
fun main() = runBlocking {
    val job = launch(CoroutineName("parent")) {
        println("Launching two coroutines...")
        val child1Job = launch {
            println("coroutine1: I'm started")
            delay(2000)
            println("coroutine1: I'm done")
        }
        launch {
            println("coroutine2: I'm started")
            delay(3000)
            println("coroutine2: I'm done")
        }
        delay(1000)
        child1Job.cancel()
    }
    job.join()
}
```

In the code above, we launch two child coroutines then after a second we cancel the first one. Canceling the first child coroutine has nothing to do with the second child coroutine and the second child continues its execution.

# **Exceptions**

## **Coroutines throw CancellationException when canceled**

When a coroutine is canceled it throws a `CancellationException`. There is no need to use a `try/catch` to catch this exception (as it’s ignored by the coroutine machinery) unless you want to do something when the coroutine is canceled.

As can be seen, when the coroutine is canceled a `CancellationException` is thrown which is caught by the `try/catch` block.

Removing the `try/catch` block does not cause the program to crash because as stated earlier, the `CancellationException` is ignored by the coroutine machinery.

## **Exception in a child coroutine cancels its siblings and parent**

When a child coroutine throws an exception (other than `CancellationException`) its siblings (if any) and parent coroutine are canceled altogether.

```kotlin
fun main() = runBlocking {
    launch(CoroutineName("parent")) {
        launch {
            println("coroutine1 started")
            delay(500)
            throw RuntimeException()
        }
        launch {
            try {
                println("coroutine2 started")
                delay(1000)
                println("coroutine2 ended")
            } catch (ex: CancellationException) {
                println("coroutine2 canceled")
            }
        }
        delay(2000)
        println("parent coroutine ended")
    }
}
```

In the above code, we launch a coroutine with two children. After 500 milliseconds the first child coroutine throws an exception. This cancels both its sibling and parent coroutines.

```
coroutine1 started
coroutine2 started
coroutine2 canceled
Exception in thread "main" java.lang.RuntimeException
 at ... (omitted for brevity)
```

Based on the above output, the parent and the sibling coroutines did not have the chance to print their respective messages because they got canceled when coroutine1 threw an exception.

It is possible to change this behavior so that when a child coroutine throws an exception, its sibling(s) and parent coroutines are not canceled.

[Source](https://mfallahpour.medium.com/using-coroutines-then-keep-these-in-mind-part-1-of-2-5087337712bd)
