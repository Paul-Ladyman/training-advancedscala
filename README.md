Advanced Scala Training
---------------------

***General Tips***

- in build.sbt its possible to define initial Scala commands to run when entering the console. This is useful for importing things e.g.: initialCommands in console := "import com.typesafe.training.scalatrain.\_"

- in the console typing :pas will enter paste mode for pasting blocks of code
- :res will reset the repl, forgetting all existing vals and defs
- :javap -p com.full.package.to.Class will show compiled Java code for a class

- To find a particular method definition in the scala docs, use the alphabet selector in the top left of the page and then search for the method.

- The elements in a Tuple can be swapped using the .swap method.

- Its possible to create classes with arguments that don't use the val keyword. These however will not be available on the object after instantiation. Its probably better to always use val.

- class constructors can be made private so that nothing can instantiate it EXCEPT a companion object. External code is then forced to use the companion object to work with the class.

- package objects defined using `pacakge object objectName` are special objects who's members are directly available to any code within the same package. According to best practice they should be defined in `package.scala` (intelliJ will do this for you)

- its possible to create a `case object myObject` which is like a `case class` but a singleton without arguments - it can be used in the same way e.g. pattern matching.

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
        case x @ X(\_, "one") => s"${x.number}, ${x.str}"
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

      // Lower bound - B is a superclass of A
      def enqueue[B >: A](b: B): Queue[B] = new Queue(elements :+ b)

Lower and upper bounds can be used independently on any methods

      // Upper bound - A is a subclass of Ordered[A]
      def isIncreasing[A <: Ordered[A]](vals: scala.collection.immutable.Seq[A]): Boolean

Path Dependent types
===================

Nested classes are unique to the instance of the outer class. E.g.

      class MyClass {
        class InnerClass
      }

      val myClass1 = new myClass1
      val innerClass1 = new myClass1.InnerClass
      val myClass2 = new myClass2
      val innerClass2 = new myClass2.InnerClass

      innerClass1.isInstanceOf(innerClass2.type)
      // false

They do however have a common super type: `MyClass#InnerClass`

Implicits
=========

When the compiler reaches a type error, before giving up, it will look for implicit conversions _in scope_. These are typically defs whose signatures match the function that produced the error. The name of the def therefore does not matter, but should be descriptive of what's going on. E.g.

      implicit def stringToInt(s: String): Int = Integer parseInt s

These should be used _sparingly_ as they obfuscate code. Certainly it is a bad idea to convert base types as above.

When multiple implicits are available a priority is applied:
* Local identifiers
* Members of an enclosing scope
* Imported identifiers
* Most specific member from the implicit scope

If multiple implicits are defined on the same level an exception will be raised

Best practice is to put custom implicit conversions in companion objects, this why no specific import is required.

Its also possible to define implicit classes - this is useful to extend particular library interfaces. E.g:

      implicit class IntReverse(val n: Int) extends AnyVal {
        def reverse: Int =
        n.toString.reverse.toInt
      }

Its also best practice to define these classes in companion objects. The class approach also requires that the method be explicitly called on the object so removes some of the obfuscation whilst maintaining the benefits of implicits. Implicit classes must only have one argument and cannot define any vals - though they may define any number of defs.

Arguments to functions can also be implicit - there can only be one and it must be the final argument list:

      implicit val y: Int = 2

      def multiply(x: Int)(implicit y: Int) =
        x * y

Type classes
============

More advanced uses of implicits includes Type Classes. Type Classes provide similar functionality to inheritance I.e. a single interface which can be used as the type of an argument to a method to which specific implementations can be passed. The advantage being less coupling and more flexibility, the disadvantage being more obfuscation. E.g.

      trait Equal[A] {
        def equal(a: A, b: A): Boolean
      }


      object Equal {
        implicit val intEqual = new Equal[Int] {
          override def equal(a: Int, b: Int): Boolean = a == b
        }

        implicit val doubleEqual = new Equal[Double] {
          override def equal(a: Double, b: Double): Boolean = a == b
        }

        implicit val stringEqual = new Equal[String] {
          override def equal(a: String, b: String): Boolean = a != b
        }

        implicit class EqualOps[A](private val v: A) extends AnyVal {
          def === (that: A)(implicit equality: Equal[A]): Boolean = {
            equality.equal(v, that)
          }
        }
      }

      1 === 1 shouldBe true

The trait defines the interface. The implicit class implicitly provides the === method and the implicit vals provide the actual implementations of the interface available for use.

DSLs
====

***By Name Parameters***

Regular method arguments are evaluated at call-site i.e. they are evaluated and then passed to the method. It is possible to defer evaluation to some place within the method itself using by-name parameters:

      def debug(msg: => String): Unit =
        if (isDebugEnabled()) println(msg)

Here `msg` is not actually evaluated unless `isDebugEnabled() == true`. ***Caution*** by-name parameters are evaluated *every* time they are referenced. Recursive calls to methods with by-name parameters might look as if the parameters are being referenced multiple times, but since the recursive call is to the same method the argument is again passed by-name - so not evaluated!
