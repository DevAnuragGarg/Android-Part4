# Android-Part4

======
Coroutines: A coroutine is an instance of suspendable computation. It is conceptually similar to a thread, in the sense that it takes a block of code to run that works concurrently with the rest of the code. However, a coroutine is not bound to any particular thread. It may suspend its execution in one thread and resume in another one.

Coroutines are software components that create sub routines for coporative multitasking. A coroutine can be introduced as a sequence of well managed sub tasks. You can execute many coroutines in a single thread. A coroutine can switch between threads. A coroutine can suspend from one thread and resume from another thread. 

Coroutines API allows us to write asynchronous codes in a sequential manner.

======
Why coroutines are called lighweight threads?
This works because coroutines can be suspended at suspension points and then resumed at a later point in time. Coroutines allow you to achieve concurrent behavior without switching threads, which results in more efficient code. Therefore, coroutines are often called “lightweight threads”.	

======
Suspending functions are at the center of everything coroutines. A suspending function is simply a function that can be paused and resumed at a later time. They can execute a long running operation and wait for it to complete without blocking. The syntax of a suspending function is similar to that of a regular function except for the addition of the suspend keyword. It can take a parameter and have a return type. However, suspending functions can only be invoked by another suspending function or within a coroutine.
suspend fun backgroundTask(param: Int): Int {
     // long running operation
}

======
In Kotlin Coroutine, whenever a coroutine is suspended, the current stack frame of the function is copied and saved in the memory. When the fucntion resumes after completing its task, the stack frame is copied back from where it was saved and starts running again. If we are going to use a suspending function, we have to mark our calling function with suspended modifier. A suspended function, can be invoked from a coroutine block or from an another suspending function only. A coroutine can invoke both suspending and non suspending functions. 

======
A suspending function doesn't block a thread, but only suspends the coroutine itself. (one thread can have more coroutines) The thread is returned to the pool while the coroutine is waiting, and when the waiting is done, the coroutine resumes on a free thread in the pool.


Suspended functions: withContext, withTimeout, withTimeoutOrNull, join, delay, await, supervisorScope, coroutineScope. 

======
Under the hood, suspend functions are converted by the compiler to another function without the suspend keyword, that takes an addition parameter of type Continuation<T>. The function above for example, will be converted by the compiler to this:
fun backgroundTask(param: Int, callback: Continuation<Int>): Int {
   // long running operation
}
Continuation<T> is an interface that contains two functions that are invoked to resume the coroutine with a return value or with an exception if an error had occurred while the function was suspended.
interface Continuation<in T> {
   val context: CoroutineContext
   fun resume(value: T)
   fun resumeWithException(exception: Throwable)
}

Delay Function: It is represented as delay(), It is a suspend function which delays coroutine for a given time without blocking a thread, and resumes it after a specified time.

======
Coroutine Scopes
A coroutine should run in a scope. The scope is used in order to track the lifecycle of the coroutine. Also it is a way to bind the coroutine(s) to an app specific lifecycle.

Behind the scope there is the concept of structured concurrency that helps us, the developers, to avoid leaking coroutines which can waste the resources (disk, memory, cpu, network).

Structured concurrency is related to three main things:
1) Keep track of the running coroutine
2) Cancel work when it is not longer necessary to run it
3) Notify errors when something bad happens to a coroutine
Every coroutine builder is an extension of CoroutineScope and inherits its coroutine context in order to propagate context elements and cancellation.

======
Scope in Kotlin’s coroutines can be defined as the restrictions within which the Kotlin coroutines are being executed. Scopes help to predict the lifecycle of the coroutines. There are basically 3 scopes in Kotlin coroutines:
Global Scope
LifeCycle Scope
ViewModel Scope

======
Global Scope is one of the ways by which coroutines are launched. When Coroutines are launched within the global scope, they live long as the application does. If the coroutines finish it’s a job, it will be destroyed and will not keep alive until the application dies, but let’s imagine a situation when the coroutines has some work or instruction left to do, and suddenly we end the application, then the coroutines will also die, as the maximum lifetime of the coroutine is equal to the lifetime of the application. Coroutines launched in the global scope will be launched in a separate thread.
GlobalScope.launch {
    Log.d(TAG, Thread.currentThread().name.toString())
}
The main problem with the coroutines launched in the global scope is that when the activity in which coroutines is launched dies, the coroutines will not die with the activity, since the lifetime of coroutines is decided on the basis of application lifetime, not the activity lifetime. Since the coroutine is using the resources of the activity in which it is launched, and now since that activity has died, the resources will not be garbage collected as a coroutine is still referring to that resources. This problem can lead to a memory leak. So using global scope all the time is not always a good idea. 

======
The lifecycle scope is the same as the global scope, but the only difference is that when we use the lifecycle scope, all the coroutines launched within the activity also dies when the activity dies. It is beneficial as our coroutines will not keep running even after our activity dies. In order to implement the lifecycle scope within our project just launch the coroutine in lifecycle scope instead of global scope.
lifecycleScope.launch {
    while (true) {
        delay(1000L)
        Log.d(TAG, "Still Running..")
    }
}

======
ViewModel Scope is also the same as the lifecycle scope, only difference is that the coroutine in this scope will live as long the view model is alive. ViewModel is a class that manages and stores the UI-related data by following the principles of the lifecycle system in android. A ViewModelScope is defined for each ViewModel in your app. Any coroutine launched in this scope is automatically canceled if the ViewModel is cleared. Coroutines are useful here for when you have work that needs to be done only if the ViewModel is active. For example, if you are computing some data for a layout, you should scope the work to the ViewModel so that if the ViewModel is cleared, the work is canceled automatically to avoid consuming resources.
class MyViewModel: ViewModel() {
    init {
        viewModelScope.launch {
            // Coroutine that will be canceled when the ViewModel is cleared.
        }
    }
}

======
Dispatchers: It is known that coroutines are always started in a specific context, and that context describes in which threads the coroutine will be started in. In general, we can start the coroutine using GlobalScope without passing any parameters to it, this is done when we are not specifying the thread in which the coroutine should be launch. This method does not give us much control over it, as our coroutine can be launched in any thread available, due to which it is not possible to predict the thread in which our coroutines have been launched. Dispatchers help coroutines in deciding the thread on which the work has to be done. Dispatchers are passed as the arguments to the GlobalScope by mentioning which type of dispatchers we can use depending on the work that we want the coroutine to do. Libraries like Room, Retrofit are using their own set of dispatchers. Types:
1) Main  Dispatcher
2) IO Dispatcher
3) Default Dispatcher
4) Unconfined Dispatcher

======
Main Dispatcher: It starts the coroutine in the main thread. It is mostly used when we need to perform the UI operations within the coroutine, as UI can only be changed from the main thread(also called the UI thread).
GlobalScope.launch(Dispatchers.Main) {
   Log.i("Inside Global Scope ",Thread.currentThread().name.toString())
}

======
IO Dispatcher: It starts the coroutine in the IO thread, it is used to perform all the data operations such as networking, reading, or writing from the database, reading, or writing to the files eg: Fetching data from the database is an IO operation, which is done on the IO thread.
GlobalScope.launch(Dispatchers.IO) {
    Log.i("Inside IO dispatcher ",Thread.currentThread().name.toString())
}

======
Default Dispatcher: It starts the coroutine in the Default Thread. We should choose this when we are planning to do Complex and long-running calculations, which can block the main thread and freeze the UI eg: Suppose we need to do the 10,000 calculations and we are doing all these calculations on the UI thread ie main thread, and if we wait for the result or 10,000 calculations, till that time our main thread would be blocked, and our UI will be frozen, leading to poor user experience. So in this case we need to use the Default Thread. The default dispatcher that is used when coroutines are launched in GlobalScope is represented by Dispatchers. so launch(Dispatchers.Default) { … } uses the same dispatcher as GlobalScope.launch { … }. It has a pool of threads with a size equal to the number of cores on the machine your code is running on
GlobalScope.launch(Dispatchers.Default) {
    Log.i("Inside Default dispatcher ",Thread.currentThread().name.toString())
}

======
Unconfined Dispatcher: As the name suggests unconfined dispatcher is not confined to any specific thread. It executes the initial continuation of a coroutine in the current call-frame and lets the coroutine resume in whatever thread that is used by the corresponding suspending function, without mandating any specific threading policy.

====== 
Coroutine builders are a way of creating coroutines. Since they are not suspending themselves, they can be called from non-suspending code or any other piece of code. They act as a link between the suspending and non-suspending parts of our code.
The three essential coroutine builders provided by the kotlinx.coroutines library:
1) launch
2) runBlocking
3) async

======
runBlocking: is a coroutine builder that blocks the current thread until all tasks of the coroutine it creates, finish. So, why do we need runBlocking when we are clearly focusing to avoid blocking the main thread?  We typically runBlocking to run tests on suspending functions. While running tests, we want to make sure not to finish the test while we are doing heavy work in test suspend functions.

======
launch: is essentially a Kotlin coroutine builder that is “fire and forget”. This means that launch creates a new coroutine that won’t return any result to the caller. It also allows to start a coroutine in the background.
This coroutine returns an instance of Job, which can be used as a reference to the coroutine, which may be later used to stop that coroutine.
GlobalScope.launch {
    println(doSomething())
}

======
async is a coroutine builder which returns some value to the caller. async can be used to perform an asynchronous task which returns a value. This value in Kotlin terms is a Deferred<T> value.

======
await() is a suspending function that is called upon the async builder to fetch the value of the Deferred object that is returned. The coroutine started by async will be suspended until the result is ready. When the result is ready, it is returned and the coroutine resumes.
GlobalScope.launch {
	val result = async {
		calculateSum()
	}
	println("Sum of a & b is: ${result.await()}")
}

======
We may need to launch more than 1 coroutine concurrently in a suspending function and get some result returned from the function. There are 2 ways to do this:
1) Structured concurrency is a set of language features and best practices introduced for Kotlin Coroutines to avoid leaks and to manage them productively. It gaurantees to complete all the works started by coroutines within the child scope before the return of the suspending function. When error happen, when exception thrown, structured concurrency gaurantees to notify the caller. We can use it to cancel the task we started. If we cancel the child scope, all the works happening inside that scope will be cancelled.
2) Unstructured concurrency is a wrong way of doing it. It does not gaurantee to complete all the tasks of the suspending function, before it returns.

======
CoroutineScope: It is an interface
coroutineScope: It is a suspending function which allows us to create a child scope within a coroutine scope. This coroutine scope gaurantees completion of the tasks when the suspending function returns. 

======
What causes configuration changes: 1) Screen rotation, Keyboard changes, Language changes, Enabling multiwindow modes.

======
What is the difference between ViewModel() and AndroidViewModel() ?
The AndroidViewModel class extends ViewModel class, so it has all the same functionality. The only added functionality for AndroidViewModel is that it is context aware, when initialising AndroidViewModel we have to pass the application context as a parameter. AndroidViewModel is helpful if we require context to get a system service or have a similar requirement(displaying a Toast message).
class MyAnViewModel(application: Application) : AndroidViewModel(application) {}

======
What is "ViewModelProvider" ?
We can not construct a ViewModel instance on our own. We need to use the ViewModelProvider utility provided by Android to create instances of ViewModels.

======
When do we need to create a ViewModelFactory class ?
ViewModelProvider can only instantiate ViewModels with no arg constructors. So, if the ViewModel has constructor parameters(arguments) , ViewModelProvider need a little extra support to create instances of it. We provide that extra support by creating a Factory class and passing its instance to the ViewModelProvider.

======
What is the onCleared() function of a ViewModel?
When a ViewModel object is no longer required, system will call to its onCleared() function to destroy(clear) it. It will be called when the app is put into the background and the app process is killed in order to free up the system's memory. When the user invokes finish() of an activity, its view model will be cleared(). Also when we click on the back button, ViewModel of current activity will be cleared (onCleared() will be invoked)

======
Livedata: A lifecycle aware observable data holder class. In Android there are 3 app components with lifecycle. Activities, Fragments and Services can be used as observers of LiveData object. Livedata only updates observers in an active lifecycle state. This is very useful. When we are using Rx java to avoid errors, we have to carefully write codes to dispose observers when activity, fragment or service becomes inactive. So, livedata automatically updates the UI when app data changes. No need to write code to handle lifecycle manually. Since livedata aware of lifecycle status changes, they clean themselves when their assciated lifecycle is destroyed. So there is no memory leaks or crashes will happen as a result of destroyed activities or fragments. 

======
What is the difference between RxJava and LiveData?
RxJava is not a lifecycle aware component. So, data stream does not go off, when activity, fragment or service becomes inactive. As a result of that, memory leaks or crashes can happen. Therefore, we have to write codes to dispose them manually. But, on the other hand, Android LiveData aware of lifecycle status changes. And, they clean up themselves(stop emitting data) when their associated lifecycle is destroyed.

======
MutableLiveData class is a subclass of LiveData class. In other words, MutableLiveData child class has created by extending the parent LiveData class. A MutableLiveData instance can do everything a LiveData instnce can do and more. Data in a LiveData object are only readable. We cannot update those values. But, in the other hand, a Mutable LiveData object allows us to change(update) its values. So, When we are creating our own live data(mostly in ViewModels), we define them as MutableLiveData. But, when we are getting live data from other libraries such as Room and Retrofit we get them as LiveData. We can easily transfer values between those two formats.

======
