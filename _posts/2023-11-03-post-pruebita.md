---
title: Inversion of Control vs Dependency Injection in Android
layout: post
description: Este es un post de pruebita
tags:
- android
- kotlin
---

In order to reduce the coupling of our code we can take advantage of the inversion of control pattern to separate classes according to their functionality.

**But what is Inversion of Control?**

It is a design pattern in which your code is part of an implementation that runs as part of an external code instead of having to call an external library that runs inside your code. Usually there is a callback defined where you can put the code that you want to use.

This is something common when working with frameworks. In Android it is something that is displayed on the methods of the lifecycle of activities. For example:

``` kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
}
```

Here you can define what will happen on your Activity when it is created and the Android framework is the one that is in charge of executing your code.

It is also applied on some Kotlin functions like forEach, filter or map. For example with the filter function:

```kotlin
fun main() {
    val numbers = listOf(1,2,4,5,6)
    val odd = numbers.filter{it % 2 == 1}
    println(odd)
}
```

The function filter already knows that it will call the function to filter some elements so the implementation of what to filter is left for us.

**And what is dependency injection?**

It is a specific version of the Inversion of Control pattern in which the objects that will be used by a class are passed to it instead of having to create them therefore that class depends on those objects.

With this you can also separate the functionality of creation of those objects to a different class and let the class that uses those objects be only in charge of knowing when to use them.

For example we can have both a class MyCode and a class Dependency:

``` kotlin
fun main() {
    val myCode = MyCode()
    myCode.hello()
}

class MyCode{
    val dependency = Dependency()

    fun main(){
        dependency.sayHello()
    }    
}

class Dependency {    
    fun sayHello() = println("Hello World")    
}
```

At first the class MyCode needs to know how to instantiate the class Dependency to use it on the function hello.

But what if we instead we pass the Dependency class as a parameter of MyCode

```kotlin
fun main() {
    val dependency = Dependency()
    val myCode = MyCode(dependency)
    myCode.hello()
}

class MyCode(val dependency : Dependency){
    fun hello(){
        dependency.sayHello()
    }   
}

class Dependency {   
    fun sayHello() = println("Hello World")   
}
```

Then MyCode can use the class Dependency without having to know how to instantiate it.

It gets even better, now we can create a new class that can be in charge of creating both MyCode class and Dependency class.

```kotlin
fun main() {
    val myCode = DependencyInjector.makeMyCode()
    myCode.hello()
}

class MyCode(val dependency : Dependency){
    fun hello(){
        dependency.sayHello()
    }
}

class Dependency {
    fun sayHello() = println("Hello World")
}

object DependencyInjector {
    fun makeDependency() : Dependency { return Dependency()}
    fun makeMyCode() : MyCode { return MyCode(makeDependency())}
}
```

On Android is common to see this applied on the model layer when the repository pattern needs to use different data sources. This data sources are usually passed as a parameter. For example

```kotlin
class MyRepository(
    LocalDataSource: DataSource,
    RemoteDataSource: DataSource
){
}
```

In Android there are tools that help with reducing the boilerplate of creating functions that create other classes like for example Dagger Hilt and Koin.

It also helps when writing unit tests because then your tests should only take in consideration how the class that you are testing is using the dependencies instead of how the dependencies are created.

**And what is Dependency Inversion?**

It is a object oriented principle which also says that a class that uses another one shouldn't know how to instantiate them and only needs to know when to use it. However, it also says that the classes that one should use as dependencies should be interfaces instead. This helps because then you can even change the implementation of the class that is used as a dependency and the class that uses it won't need to change anything at all. For example:

```kotlin
fun main() {
    val myCode = DependencyInjector.makeMyCode()
    myCode.hello()
}

class MyCode(val dependency : AbstractDependency){
    fun hello(){
        dependency.sayHello()
    }
}

interface AbstractDependency{
    fun sayHello()
}

class Dependency : AbstractDependency{    
    override fun sayHello() = println("Hello World")
}

object DependencyInjector {
    fun makeDependency() : Dependency { return Dependency()}
    fun makeMyCode() : MyCode { return MyCode(makeDependency())}    
}
```

Here we added an interface AbstractDependency that will be passed as a parameter to the class MyCode instead of Dependency.

Then if we want to add a new different dependency we only need to change what is passed on the makeMyCode method from DependencyInjector. There is no need to change MyCode class at all!

```kotlin
fun main() {
    val myCode = DependencyInjector.makeMyCode()
    myCode.hello()
}

class MyCode(val dependency : AbstractDependency){

    fun hello(){
        dependency.sayHello()
    }
    
}

interface AbstractDependency{
    fun sayHello()
}

class Dependency : AbstractDependency{    
    override fun sayHello() = println("Hello World")
}

class NewDependency : AbstractDependency{
    override fun sayHello() = println("Hola Mundo")
}

object DependencyInjector {   
    fun makeDependency() : Dependency { return Dependency()}
 fun makeNewDependency() : NewDependency { return NewDependency()}
    fun makeMyCode() : MyCode { return MyCode(makeNewDependency())}   
}

```

**Conclusion**

Inversion of Control and Dependency Injection are two concepts that are similar but are used different parts of the code of Android. Both help to improve your code and be easier to change in the future. Nowadays there are tools to help implement these concepts without having to write a lot of code but it is always good to know why these concepts are useful and when it will be a good idea to use them.

**References:**

- [https://medium.com/ssense-tech/dependency-injection-vs-dependency-inversion-vs-inversion-of-control-lets-set-the-record-straight-5dc818dc32d1#:~:text=The%20Inversion%20of%20Control%20is,dependencies%20to%20an%20application's%20class.](https://medium.com/ssense-tech/dependency-injection-vs-dependency-inversion-vs-inversion-of-control-lets-set-the-record-straight-5dc818dc32d1#:~:text=The%20Inversion%20of%20Control%20is,dependencies%20to%20an%20application's%20class.)
- [https://stackoverflow.com/a/6551303/7615478](https://stackoverflow.com/a/6551303/7615478)
- [https://www.codeguru.com/csharp/dependency-injection-vs-inversion-control/](https://www.codeguru.com/csharp/dependency-injection-vs-inversion-control/)
