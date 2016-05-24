Advanced Scala Training
---------------------

GENERAL TIPS
============

- in build.sbt its possible to define initial Scala commands to run when enterring the console. This is useful for importing things e.g.: initialCommands in console := "import com.typesafe.training.scalatrain._"

- in the console typing :pas will enter paste mode for pasting blocks of code
- :res will reset the repl, forgetting all existing vals and defs
- :javap -p com.full.package.to.Class will show compiled Java code for a class

- To find a particular method definition in the scala docs, use the alphabet selector in the top left of the page and then search for the method.

- The elements in a Tuple can be swapped using the .swap method.

- Its possible to create classes with arguments that don't use the val keyword. These however will not be available on the object after instantiation. Its probably better to always use val.

- class constructors can be made private so that nothing can instantiate it EXCEPT a companion object. External code is then forced to use the companion object to work with the class.

Functions
=========

- A function is as in mathematics.
- A pure function has no side effects and guarantees the same out put for a given set of inputs.
- High order functions are those that take other functions as arguments, or that return a function. E.g. map, flatMap, filter etc.

Useful Higher Order Functions
===========================

- map, flatMap, filter...
- forall - test some predicate holds for all elements in a collection
- exists - test some predicate holds for at least one element in a collection
- sliding - Create a list of windowed views on some collection of data. Window can have arbitrary length and arbitrary sliding gaps
- foldLeft, foldRight - creates a single value from some collection of values according to some function. Iterates either from the left or the right of the collections. Each time the function is applied it will take the intermediate result of the previous application, as well as the current element of the collection. foldLeft/foldRight expect an initial seed value for the first iteration
- groupBy - organise a collection into a Map where the key is some value chosen from the elements in the collection

For Expressions
===============

- To apply a for expression to an object, the object must have map (generators e.g. 1 <- 1 to 3), flatMap (multiple generators) and filterWith (guards/if logic) methods defined.
- Can also use definitions with =

Inheritance and Traits
======================

- the override key word must be used when overriding already implemented vals/defs. It can be omitted when overriding abstracts but its best practice to use it.
- the sealed key word means that a trait can only be overridden from within the same source file.
- traits that come last in the extension list have higher priority in terms of overriding.

Pattern Matching
================

- Can match on the value of a variable using back ticks, otherwise it will be treat as a wild card. E.g.:

x match {
  case `y` => true
}

- Pipes can be used for OR pattern matching logic
- @ can be used to assign the result of a match to a variable. This is useful in more complex patterns. E.g.

case class X(number: Int, str: String)
X(1, "one") match {
  case x @ X(_, "one") => s"${x.number}, ${x.str}"
}

- Pattern matching can be used to unapply/deconstruct a case class. E.g.:
val X(num, str) = X(1, "one")

- It can be very useful to match over lists when different operations should be applied depending on its size. E.g.

val times = Seq(1, 2, 3, 4, 5)

// Ensure Seq has increasing Int values
times.sliding(2).forall {
  case t1 +: t2 +: _ => t1 < t2
  case _ => true
}

Recursion
=========

- Tail recursiveness is when the recursive call is the last thing to happen in the function, without its results being handled
- Scala can compile tail recursive functions to while loops for efficiency using @tailrec

Variance
========

Comes into play when you're defining classes with generic types. If the concrete type supplied on instantiation has sub/super classes, these relationships will not by default be supported by the compiler when dealing with the container class (it is invariant by default.) E.g.

class A
class B extends A

val qb: Queue[B] = new Queue(new B)
val qa: Queue[A] = qb // does not compile

Generic types can be defined as covariant (follows the same hierarchy of the concrete type) or contravariant (follows the opposite hierarchy of the concrete type) as appropriate to support these relationships. E.g. for covarience:

class Queue[+A] (val elements: Seq[A])

Any [A] or [B extends A] can now be added to the elements Seq.

If it is desired for a method on Queue[B] to return a Queue[A] where [B extends A], i.e. return a more generic version of Queue that may contain mixed subtypes of A, then this can be achieved using lower bounds. E.g:

// B >: A == B is a superclass of A
def enqueue[B >: A](b: B): Queue[B] = new Queue(elements :+ b)
