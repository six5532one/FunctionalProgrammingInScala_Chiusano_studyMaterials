These are from Chapter 2 of [Functional Programming in Scala](https://www.manning.com/books/functional-programming-in-scala).

The following example of how to write loops functionally (without mutating a loop variable) will be helpful for exercises 1 and 2.
	```
	def factorial(n: Int): Int = {
		def go(n: Int, acc: Int): Int =
			if (n <= 0) acc
			else go(n-1, n*acc)
	go(n, 1)
	}
	```

1
==
Write a recursive function to get the nth Fibonacci number (http://mng.bz/C29s). The first two Fibonacci numbers are 0 and 1. The nth number is always the sum of the previous two—the sequence begins 0, 1, 1, 2, 3, 5. Your definition should use a local tail-recursive function.
```
def fib(n: Int): Int
```


2
==
Implement isSorted, which checks whether an Array[A] is sorted according to a given comparison function:
```
def isSorted[A](as: Array[A], ordered: (A,A) => Boolean): Boolean
```


3
==
Currying converts a function f of two arguments into a function of one argument that partially applies f. There’s only one implementation of currying that compiles. Write this implementation.
```
def curry[A,B,C](f: (A, B) => C): A => (B => C)
```

Clarification of the method signature of `curry`
------------------------------------------------
- param: a function `f` that takes 2 params (`a, `b`) and has a return value of type `C`
- returns: the curried version of the input function; a function `g` that takes 1 param `a` and returns the function `h` that takes 1 param `b` and has retval of type `C` such that `g` has the signature: `g(a: A): (b: B) => C` and `g(a)(b) = f(a,b)`

An annotated console session that demonstrates `curry`:

	Let `sumTo5` be the input `f` to `curry`, where `f` is a 
	function that takes 2 params
	scala> def sumTo5(x: Int, y: Int): Boolean = x + y == 5
	sumTo5: (x: Int, y: Int)Boolean

	scala> sumTo5(7,-2)
	res2: Boolean = true

	Let `sumTo5Curried` be the return value of calling `curry` with argument `sumTo5`,
	where the return value must be a function that takes 1 param (rather than 2).
	scala> def sumTo5Curried = Chapter1And2.curry(sumTo5)
	sumTo5Curried: Int => (Int => Boolean)
	
	The return type of `curry` is a function that accepts 1 param and returns
	another function that accepts 1 param.
	Apply the return value of `curry` (`sumTo5Curried`) to an argument `x`
	and assign the result of that application to `sumTo5PartiallyApplied`.
	`sumTo5PartiallyApplied` is thus a function that accepts 1 param.
	scala> val x = 7
	x: Int = 7
	scala> val sumTo5PartiallyApplied = sumTo5Curried(x)
	sumTo5PartiallyApplied: Int => Boolean = <function1>
	
	Apply `sumTo5PartiallyApplied` to an arg:
	scala> sumTo5PartiallyApplied(-2)
	res4: Boolean = true

	scala> sumTo5PartiallyApplied(-1)
	res5: Boolean = false

	Innvoke the curried version of `sumTo5`:
	scala> sumTo5Curried(7)(-2)
	res6: Boolean = true

	scala> sumTo5Curried(7)(-1)
	res7: Boolean = false

	Currying gives us an easy way to define related functions:
	scala> def ifAddedTo7ThenSumsTo5(b: Int): Boolean = sumTo5Curried(7)(b)
	ifAddedTo7ThenSumsTo5: (b: Int)Boolean

	scala> ifAddedTo7ThenSumsTo5(-2)
	res8: Boolean = true

	scala> ifAddedTo7ThenSumsTo5(-1)
	res9: Boolean = false



4
--
Implement uncurry, which reverses the transformation of curry. Note that since => associates to the right, A => (B => C) can be written as A => B => C.
```
def uncurry[A,B,C](f: A => B => C): (A, B) => C
```

Clarification of what `uncurry` should accomplish:
	Given the curried form of a function, `f`, return the original function that requires two arguments.
	
A console session that demonstrates `uncurry`:
	scala> def sumTo5(x: Int, y: Int): Boolean = x + y == 5
	sumTo5: (x: Int, y: Int)Boolean

	scala> val sumTo5CurriedThenUncurried = Chapter1And2.uncurry( Chapter1And2.curry(sumTo5) )
	sumTo5CurriedThenUncurried: (Int, Int) => Boolean = <function2>



5
--
Implement the higher-order function that composes two functions.
```
def compose[A,B,C](f: B => C, g: A => B): A => C
```

A console session that demonstrates `compose`:

	scala> def printRes(b: Float): Unit = println(s"$b is a float")
	printRes: (b: Float)Unit

	scala> def asFloat(a: Int): Float = a.toFloat
	asFloat: (a: Int)Float

	scala> printRes(asFloat(5))
	5.0 is a float

	scala> Chapter1And2.compose(printRes, asFloat)(5)
	5.0 is a float
