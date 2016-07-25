Why Functional Programming?
===========================

Problem: Mutable state is bad
-----------------------------
Stateful systems are:
* harder to reason about because you have to keep track of changes to state
Example: Your program is in the middle of modifying a data structure in place when it throws an exception. It's hard to write code to catch the exception, undo the partial changes already made to the data structure, and rethrow the exception. If the exception handler doesn't revert the changes to the partially-modified data structure, the program will continue executing with corrupt data.
* harder to test
* harder to parallelize (need synchronization mechanisms like Mutexes)


How does FP solve this problem?
-------------------------------
FP does away with the concept of mutability; instead of mutating state, you compose functions to obtain the values you need. 

To use this approach, we must construct our programs using mostly pure functions.


What is a pure function?
-------------------------
A function that always returns the same result for the same input and 
does nothing other than return a result.

In contrast, a function that does something other than return a result has side effects.

Examples of side effects:
* modify a variable (i.e. setting a field on an object)
* modify a data structure in place (i.e. setting the right subtree of a binary tree with `oldTree.right = r` modifies it in place whereas `t = new Tree(oldTree.root, oldTree.left, r)` returns a tree with the desired properties without modifying `oldTree`)
* i/o: printing to the console, reading user input, reading from or writing to a file
* throwing an exception

In summary, these are characteristics of a pure function:
* Its result is always the same for the same supplied parameters.
* Its result is not dependent on anything that isn’t supplied as a parameter.
* It does not alter any of the parameters that were supplied.
* It does not alter anything outside its scope.

Mathematical functions are pure functions.
Example:
```
f(x) = mx + b
```


Referential Transparency and the Substitution Model
----------------------------------------------------
*Referential transparency* is a property of expressions that allow us to evaluate programs using the *substitution model*.

An expression is referentially transparent if, in any program, it can be replaced by its result without changing the meaning of the program.

Is the following function referentially transparent?
```
def buyCoffee(cc: CreditCard): Coffee = {
	val cup = new Coffee()
	cc.charge(cup.price)
	cup
}
```
If the function `buyCoffee` were referentially transparent, we could substitute any call with its result `new Coffee()` and the program would behave the same.
If we substitute
`f(buyCoffee(aliceCreditCard))` with `f(new Coffee())` for any `f` in the program, the program would not behave the same way because `buyCoffee` charges credit cards in addition to returning a cup of coffee. So `buyCoffee` is not referentially transparent.

When expressions are referentially transparent, we can use the substitution model to reason about the program.



Why does FP impose the constraint of using pure functions?
-----------------------------------------------------------
* FP relies on function composition, rather than state mutation, to compute the values in our program. Hence, in FP, functions should be reusable and composable. Side effects or external interference make functions less reusable. Eliminating side effects results in more reusable functions, which can be composed in more flexible ways.

* Side effects make it harder to reason about the correctness of a program. For example:
Your program is in the middle of modifying a data structure in place when it throws an exception. It's hard to write code to catch the exception, undo the partial changes already made to the data structure, and rethrow the exception. If the exception handler doesn't revert the changes to the partially-modified data structure, the program will continue executing with corrupt data, making it harder to debug.

In FP, we aim to identify side effects and limit them.





Intro to Functional Programming w Scala
========================================
In the following code, the `main` method of `MyModule` is an impure function ("procedure") because calling it results in a side effect (printing to console). The methods that `main` invokes are pure functions.

We want to minimize the number of procedures in our programs by defining as many functions as possible by combining pure functions. Higher-order functions allow us to define pure functions using function composition.


Higher-order functions
-----------------------
Higher-order functions treat functions like other values, i.e.
- passing them as arguments to other functions
- returning them as the return value of a function
- assigning them to a variable
- storing them in data structures

```
object MyModule {
	def abs(n: Int): Int = {
	    if (n < 0) -n
	    else n
	  }

	  def factorial(n: Int): Int = {
	    def go(n: Int, acc: Int): Int =
	      if (n <= 0) acc
	      else go(n-1, n*acc)
	    go(n, 1)
	  }

	  /**
	  Motivating higher-order functions:
	  instead of defining distinct functions to format
	  the result of calling `formatAbs` and
	  `formatFactorial` on an input, we can define a single
	  function that's responsible for formatting and parameterize
	  the computation that it's formatting the result for.
	  */

	  private def formatAbs(x: Int) = {
	    val msg = "The abs val of %d is %d"
	    msg.format(x, abs(x))
	  }

	  private def formatFactorial(n: Int) = {
	    val msg = "The factorial of %d is %d."
	    msg.format(n, factorial(n))
	  }

	  /**
	  This higher-order function can replace the two formatting
	  methods above.
	  */
	  private def formatResult(name: String, n: Int, f: Int => Int) = {
	    val msg = "The %s of %d is %d."
	    msg.format(name, n, f(n))
	  }

	  def main(args: Array[String]): Unit = {
	    // Without higher-order functions
	    println(formatAbs(-42))
	    println(formatFactorial(7))
	    // Using a higher-order function instead
	    println(formatResult("absolute value", -42, abs))
	    println(formatResult("factorial", 7, factorial))
	  }
}
```


Polymorphism: Defining functions that can be applied to arguments of any type
------------------------------------------------------------------------------
`findFirst` returns the lowest array index of an element that matches the `key` argument and -1 if none of the elements in `ss` match `key`:
```
def findFirst(ss: Array[String], key: String): Int = {
	@annotation.tailrec
	def loop(n: Int): Int =
		if (n > ss.length) -1
		else if (ss(n) == key) n
		else loop(n + 1)
	loop(0)
}
```
We can redefine `findFirst` to search within arrays of any type by making two changes:
1) use a type parameter (`A`)
2) Instead of using the `==` operator to identify the desired element in `as`, use an argument `p`. `p` is a function that takes an argument of type `A` (an element in `as`) and returns a value of Boolean type.

```
def findFirst[A](as: Array[A], p: A => Boolean): Int = {
	@annotation.tailrec
	def loop(n: Int): Int =
		if (n >= as.length) -1
		else if (p(as(n))) n
		else loop(n + 1)
	loop(0)
}
```


Anonymous functions as an alternative to named functions
----------------------------------------------------------
When calling a higher-order function, it's often easier to pass anonymous functions (rather than named ones) as its argument.

Invoking `findFirst` with a named function:
```
	def equals9(x: Int) = x == 9
	findFirst(Array(7, 9, 13), equals9)
```
Invoking `findFirst` with an anonymous function:
```
	findFirst(Array(7, 9, 13), (x: Int) => x == 9)
```

Syntax for anonymous functions:
- left-hand-side: comma-delimited list of parameter names and types
- right-hand-side: body of function
i.e.
```
(x: Int) => x == 9
```
or
```
(x: Double, b: Double) => 5 * x + b
```


Partial application
--------------------
`partial1` is a higher-order function for performing partial application. The name comes from the fact that its input function is being applied to some but not all of the arguments it requires. It takes a parameter `f` (a function that needs both `A` and `B` to produce `C`) and partially applies it to a value `a` of type `A`. As a result, it returns another higher-order function that only needs `B` to produce `C`.
```
def partial1[A,B,C](a: A, f: (A,B) => C): B => C =
	(b: B) => f(a, b)
```

Currying
---------
Currying is the process of decomposing a function of multiple arguments into a chained sequence of functions of one argument. It accomplishes this using partial applications.

Let's walk through the process of currying a function `f(x1, x2, ...., xn)` that requires n arguments.

The result of partial application of `f` to `x1` is `g` (a function that requires params x2, ...., xn):
```
f(x1) = g
```

The result of partial application of `g` to `x2` is `h` (a function that requires params x3, ...., xn):
```
g(x2) = h
f(x1)(x2) = h
```

We can partially apply `h` to `x3`, then partially apply the resulting function to `x4`, and so on until we have a function that only requires param `xn`:
```
f(x1)(x2)(x3)(x4)....(xn) = f(x1, x2, ...., xn)
```

Currying `f` allows us to compose its components to form the following functions:
```
f(x1),
f(x1)(x2),
f(x1)(x2)(x3),
....
f(x1)(x2)(x3)(x4)....(xn)
```
This results in more flexibility in how we can reuse different components of curried `f`.


Example:
```
def line(m: Double, y_intercept: Double, x: Double): Double = m * x + y_intercept
def curriedLine(m: Double)(y_intercept: Double)(x: Double) = line(m, y_intercept, x)
def horizontalLine(y_intercept)(x: Double) = curriedLine(0)(y_intercept)(x)
```
