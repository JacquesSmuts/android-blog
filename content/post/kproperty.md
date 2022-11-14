---
title: "Getting arbitrary properties from unknown classes with KProperty"
author: "Jacques Smuts"
image: "/images/generic_interfaces3.png"
tags: ["android", "kotlin", "kproperty", "kproperty1", "reflection"]
date: 2019-08-01T20:29:26+02:00
draft: false
---
The Kproperty class is powerful and useful. 

<!--more-->

### Accessing a property, normally

You have a class, maybe a `data class` to retain info about films. Our class might look like this:

{{< kotlin >}}

fun main (args: Array<String>) {
//sampleStart
	data class Film(val name: String, val releaseDate: Long, val runtime: Int)
    val hereditary = Film("Hereditary", 1528401600, 127)
    println(hereditary.name)
//sampleEnd

	println("is the best horror film since The Thing")
}
{{< /kotlin >}}

Which is fine for a basic class. However there are a massive amount of different films. And different film types, with different attributes. Horror films have some sort of monster or threat, but drama films don't. Animated films have an animation style, and documentary films have an educational subject matter. A simple solution would be to extend your `Film` class to have multiple `null`able fields to reflect all of the potential properties of a film. That solution would reflect a database table more accurately and can be a perfectly fine solution.

## Dynamic Typing via Interfaces

But perhaps you have a constraint where a better approach would be to follow the SOLID principles and create some sort of [Clean Code implementation of interfaces](https://medium.com/@severinperez/avoiding-interface-pollution-with-the-interface-segregation-principle-5d3859c21013) that a data class can extend. A bunch of little interfaces, like so:

{{< kotlin >}}

//sampleStart
interface Film {
    val name: String
    val releaseDate: Long
    val runtime: Int
}

interface PostApocalyptic: Film {
    val yearsInFuture: Int
    val causeOfSocietalCollapse: String
}

interface Romance: Film {
    val loveTriangles: Int
    val happyEnding: Boolean
}

data class YoungAdultFilm(override val name: String,
                         override val releaseDate: Long,
                         override val runtime: Int,
                         override val yearsInFuture: Int,
                         override val causeOfSocietalCollapse: String,
                         override val loveTriangles: Int,
                         override val happyEnding: Boolean
                         ): Film, PostApocalyptic, Romance
//sampleEnd

fun main (args: Array<String>) {
    val divergent = YoungAdultFilm("Divergent", 1395172800, 127, 50, "fascism", 1, true)
    println(divergent.name)
   	println("is not.")
}
{{< /kotlin >}}

Yes, this is a fairly unusual way of doing it, but sometimes it is required due to circumstances outside of your control. Luckily it still has some advantages in type safety. For example, now we are able to do checks to determine whether or not a given class is a certain type and return the property of our choosing. With the magic of Kotlin's aggressive type inference, this code compiles and is typesafe:

{{< kotlin >}}

interface Film {
    val name: String
    val releaseDate: Long
    val runtime: Int
}

interface PostApocalyptic: Film {
    val yearsInFuture: Int
    val causeOfSocietalCollapse: String
}

interface Romance: Film {
    val loveTriangles: Int
    val happyEnding: Boolean
}

data class YoungAdultFilm(override val name: String,
                         override val releaseDate: Long,
                         override val runtime: Int,
                         override val yearsInFuture: Int,
                         override val causeOfSocietalCollapse: String,
                         override val loveTriangles: Int,
                         override val happyEnding: Boolean
                         ): Film, PostApocalyptic, Romance


fun main (args: Array<String>) {
    val midsommar = YoungAdultFilm("Midsommar", 1395172800, 127, 0, "Society", 1, true)
    val hasLoveTriangles = hasLoveTriangles(midsommar)
    println("${midsommar.name} has love triangles? $hasLoveTriangles")
}
//sampleStart
fun hasLoveTriangles(film: Film): Boolean {
    
    return if (film is Romance) {
        film.loveTriangles > 0
    } else {
        false
    }
}
//sampleEnd

{{< /kotlin >}}

The film might or might not be a Romance. But the moment we have done a check for `film is Romance` then the compiler can infer that the film has the `loveTriangles` property.

Pretty convenient right?

But now for every single property we have to write this out manually. If film is `PostApocalyptic`, then get the `causeOfSocietalCollapse` property. If the film is `Romance`, then get the `happyEnding` property. And so forth for every single property of every interface which extends `Film`.

This all becomes rather tedious. So maybe we should rather write out something Generic, which allows us to check for any arbitrary property. Is this even possible?

## Generics Saves the Day Again

Yes. We can iterate through the properties and find the one which matches a name we specify. This is not an ideal solution because we have to pass in a string. Not very typesafe. And if any variable name changes, that string will no longer match.

However, with Kotlin's `inline` and `reified` keywords, we can do amazing generic work. First you have to have the Kotlin Reflection Library, (as described in my [previous article on Reflection]({{< ref "/post/generic_interface_and_methods" >}})).

Then you have to read about [Reflection and KProperty<>](https://kotlinlang.org/docs/tutorials/kotlin-for-py/member-references-and-reflection.html) and that if you can get access to a property, you can obtain the value of that property by using `kproperty.get(instance)`.

Then putting all that together then we can write out a fairly simple generic function, like so:

{{< kotlin >}}
import kotlin.reflect.KProperty1

interface Film {
    val name: String
    val releaseDate: Long
    val runtime: Int
}

interface PostApocalyptic: Film {
    val yearsInFuture: Int
    val causeOfSocietalCollapse: String
}

interface Romance: Film {
    val loveTriangles: Int
    val happyEnding: Boolean
}

data class YoungAdultFilm(override val name: String,
                         override val releaseDate: Long,
                         override val runtime: Int,
                         override val yearsInFuture: Int,
                         override val causeOfSocietalCollapse: String,
                         override val loveTriangles: Int,
                         override val happyEnding: Boolean
                         ): Film, PostApocalyptic, Romance


//sampleStart
fun main (args: Array<String>) {

    val film = YoungAdultFilm("The End of Evangelion", 869342400, 85, 20, "Instrumentality", 1, true)

    [mark]val hasHappyEnding: Boolean? = getAttribute(film, Romance::happyEnding)[/mark]
    println("${film.name} has a happy ending? $hasHappyEnding")
}

inline fun <T, reified Interface> getAttribute(input: Film, property: KProperty1<Interface, T>): T? {
    if (input is Interface) {
        return property.get(input) as? T
    }
    return null
}
//sampleEnd
{{< /kotlin >}}


Now you can just pass in your `Interface::Property` reference and you'll know that the right property type will be accessed, that the property is definitely linked to the right interface, and that any future name changes will not result in breaking changes without the IDE telling you.

#### To explain the function:

KProperty1 takes as Generic input the interface or class to which it belongs, as well as the required output (`T`). In our case we allow any output, but it must be a property of the given `<Interface>`. In short, we can theoretically get *any attribute* from a given class with casting, but that isn't safe. Instead we use this function, and with the `<Interface, T>` we pass in, it is still typesafe and reliable.

This looks pretty great so far. If you want to keep going you can turn the getAttribute into an Extension Function or infix function on the Film class. Maybe you find this to be a bit more readable:

{{< kotlin >}}
import kotlin.reflect.KProperty1

interface Film {
    val name: String
    val releaseDate: Long
    val runtime: Int
}

interface PostApocalyptic: Film {
    val yearsInFuture: Int
    val causeOfSocietalCollapse: String
}

interface Romance: Film {
    val loveTriangles: Int
    val happyEnding: Boolean
}

data class YoungAdultFilm(override val name: String,
                         override val releaseDate: Long,
                         override val runtime: Int,
                         override val yearsInFuture: Int,
                         override val causeOfSocietalCollapse: String,
                         override val loveTriangles: Int,
                         override val happyEnding: Boolean
                         ): Film, PostApocalyptic, Romance

fun main (args: Array<String>) {

    val film = YoungAdultFilm("The End of Evangelion", 869342400, 85, 20, "Instrumentality", 1, true)

//sampleStart
    val hasHappyEnding = film.getAttribute(Romance::happyEnding) ?: false
//sampleEnd
    
    println("${film.name} has a happy ending? $hasHappyEnding")
}

inline fun <T, reified Interface>Film.getAttribute(property: KProperty1<Interface, T>): T? {
    return getFilmAttribute(this, property)
}

inline fun <T, reified Interface> getFilmAttribute(input: Film, property: KProperty1<Interface, T>): T? {
    if (input is Interface) {
        return property.get(input) as? T
    }
    return null
}

{{< /kotlin >}}

### Win win? 

Well mostly. The disadvantage is that you can't know whether or not any given property is nullable or not. Via this method, everything becomes nullable. To obtain only non-nullable values you have to either do the type-casting manually, or use a default value. You can see me passing `false` into the elvis operator above as a default value.

Furthermore, there's a performance cost.


### Reflection is Slower, But By How Much?

Luckily this one is fairly simple to test. I just access the property several times, and log how long it takes to do that directly or via reflection. The code is here, and you can run it to see the result for yourself:

{{< kotlin >}}
import kotlin.reflect.KProperty1
import kotlin.system.measureTimeMillis

interface Film {
    val name: String
    val releaseDate: Long
    val runtime: Int
}

interface PostApocalyptic: Film {
    val yearsInFuture: Int
    val causeOfSocietalCollapse: String
}

interface Romance: Film {
    val loveTriangles: Int
    val happyEnding: Boolean
}

data class YoungAdultFilm(override val name: String,
                         override val releaseDate: Long,
                         override val runtime: Int,
                         override val yearsInFuture: Int,
                         override val causeOfSocietalCollapse: String,
                         override val loveTriangles: Int,
                         override val happyEnding: Boolean
                         ): Film, PostApocalyptic, Romance

//sampleStart
val iterations = 1000000000
fun main (args: Array<String>) {

    val normalTime = measureTimeMillis(::normalDirectAccessTime)
    println("It takes $normalTime milliseconds to do $iterations direct property access operations")

    val manualCastingTime = measureTimeMillis(::manualCastingAccessTime)
    println("It takes $manualCastingTime milliseconds to do $iterations manual casting property access operations")
    
    val reflectionTime = measureTimeMillis(::reflectionAccessTime)
    println("It takes $reflectionTime milliseconds to do $iterations reflected property access operations")
   
}
//sampleEnd

fun normalDirectAccessTime() {
   
    val film = YoungAdultFilm("The End of Evangelion", 869342400, 85, 20, "Instrumentality", 1, true)

    repeat(iterations) {
        film.happyEnding
    }
}

fun manualCastingAccessTime() {
	val film = YoungAdultFilm("The End of Evangelion", 869342400, 85, 20, "Instrumentality", 1, true)

    repeat(iterations) {
        getHappyEnding(film)
    }
}

fun getHappyEnding(film: Film): Boolean {
	return if (film is Romance) {
		film.happyEnding
	} else {
		false
	}
}


fun reflectionAccessTime() {
   
    val film = YoungAdultFilm("The End of Evangelion", 869342400, 85, 20, "Instrumentality", 1, true)

    repeat(iterations) {
        getAttribute(film, Romance::happyEnding) ?: false
    }
}

inline fun <T, reified Interface> getAttribute(input: Film, property: KProperty1<Interface, *>): T? {
    if (input is Interface) {
        return property.get(input) as? T
    }
    return null
}

{{< /kotlin >}}

In my case, the reflected property access operation takes on average about 2.5 times longer than the direct access. And the manual casting takes just a smidgeon longer than the normal direct access.

That's actually pretty impressive. I was expecting more than 90% decrease in performance for reflection; instead it's only about 60%. This is not ideal if you care about optimizing for billions of operations per second, but acceptable for less intense usecases, such as in user(Android) applications that cater for a single user at a time.

### Conclusion

KProperty is an essential class for doing higher-order programming in Kotlin. You can use it to get further information about any given property, or you can use it to obtain properties that you don't normally have access to, as demonstrated above.




