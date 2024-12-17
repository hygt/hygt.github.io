---
layout: post
title: "The missing findM and collectFirstM methods in cats.Foldable"
author: Henry
category: Scala
tags:
  - Scala
  - Cats
  - Scalaz
  - collectFirstM
  - findM
  - foldM
  - short-circuit
---

### Update (2024)

Shortly after I wrote this article, both methods were added to the `cats.Foldable` type class.
See the [Scaladoc](https://typelevel.org/cats/api/cats/Foldable.html).

### Take me to the code

You can jump straight to [the solution](#the-solution).

## The problem

Imagine you have a collection of items and want the first one that satisfies a predicate. The Scala standard
library provides [find](https://www.scala-lang.org/api/2.12.6/scala/collection/immutable/List.html#find(p:A=%3EBoolean):Option[A]).

Additionally, if you rather want to get the first item for which a function is defined, there's
[collectFirst](https://www.scala-lang.org/api/2.12.6/scala/collection/immutable/List.html#collectFirst[B](pf:PartialFunction[A,B]):Option[B]).

Now, dealing with effectful types (Scala futures and Monix tasks in my case), I was expecting Cats to
provide similar methods in a monadic context. Sadly, they don't exist. If you're using Scalaz, it seems
that you're at least partially covered: `findM` is provided.

## Setup

Let's define two different types for illustration purposes:

```scala
case class Foo(value: Int) // input
case class Bar(value: Int) // output
```

A predicate in a future context:

```scala
def isEven(foo: Foo): Future[Boolean] = Future.successful {
  foo.value % 2 == 0
}
```

And a partial function (already lifted into an `Option`), also in a future:

```scala
def transformIfEven(foo: Foo): Future[Option[Bar]] =
  isEven(foo).map { even =>
    if (even) Some(Bar(foo.value))
    else None
  }
```

### Imports

If you don't want to "blanket import" Cats implicits, you'll need these:

```scala
import cats.Monad
import cats.data.EitherT
import cats.instances.future._
import cats.instances.list._ // maybe stream too
import cats.syntax.flatMap._
import cats.syntax.foldable._
import cats.syntax.functor._
```

Also don't forget the `ExecutionContext` in order to evaluate futures.

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
```

## Goal

We want to be able to do that:

```scala
val list = List(Foo(1), Foo(2), Foo(3), Foo(4), Foo(5))

for {
  foo <- findM(list, isEven)
  bar <- collectFirstM(list, transformIfEven)
} {
  println(s"First `Foo` that is even: $foo")
  println(s"First `Bar` that was transformed: $bar")
}
```

Which should output:

```
First `Foo` that is even: Some(Foo(2))
First `Bar` that was transformed: Some(Bar(2))
```

I'll keep the input a plain `List` throughout the article for clarity, but this could of course be
generalized to any `Foldable` instance.

## Using foldLeft

A naïve way to proceed is to fold through the entire list and use the accumulator to store the first
result, and then carry it over until the end, ignoring the remaining elements.

```scala
def findM[G[_], A](list: List[A], p: A => G[Boolean])
  (implicit G: Monad[G]): G[Option[A]] = {
  list.foldLeft[G[Option[A]]](G.pure(None)) { (acc, elem) =>
    acc.flatMap {
      case found @ Some(_) => G.pure(found)
      case None => p(elem).map {
        case true  => Some(elem)
        case false => None
      }
    }
  }
}

def collectFirstM[G[_], A, B](list: List[A], pf: A => G[Option[B]])
  (implicit G: Monad[G]): G[Option[B]] = {
  list.foldLeft[G[Option[B]]](G.pure(None)) { (acc, elem) =>
    acc.flatMap {
      case found @ Some(_) => G.pure(found)
      case None => pf(elem)
    }
  }
}
```

This works but iterates through the whole list even after a result is found. Surely, there must be a better
way to do it. Or in other words, I want to short-circuit the traversal as soon as I find that result.

## Foldable#existsM

My first intuition was too look for the monadic counterpart of `exists`. After all, this is just a specific
use of `findM`. And indeed, the `cats.Foldable` Scaladoc is telling me exactly what I wanted to hear:

> Check whether at least one element satisfies the effectful predicate.
> If there are no elements, the result is `false`.  `existsM` short-circuits,
> i.e. once a `true` result is encountered, no further effects are produced.

Let's have a look at the implementation:

```scala
def existsM[G[_], A](fa: F[A])(p: A => G[Boolean])
  (implicit G: Monad[G]): G[Boolean] = {
  G.tailRecM(Foldable.Source.fromFoldable(fa)(self)) {
    src => src.uncons match {
      case Some((a, src)) => G.map(p(a))(bb => if (bb) Right(true) else Left(src.value))
      case None => G.pure(Right(false))
    }
  }
}
```

Here, `fa` is our collection. Don't be distracted by the `Foldable.Source` wrapper.
While there's no specific implementation for `List`, it would probably look like this:

```scala
def existsM[G[_], A](fa: List[A])(p: A => G[Boolean])
  (implicit G: Monad[G]): G[Boolean] = {
  G.tailRecM(fa) {
    list => list match {
      case head :: tail => G.map(p(a))(bb => if (bb) Right(true) else Left(tail))
      case Nil => G.pure(Right(false))
    }
  }
}
```

What's important to notice is the use of `FlatMap#tailRecM`. I'm not going to dive into the motivation or
the inner workings of this method, since it's pretty tangential to my point, and also because you'll easily
find explanations in the [official Cats documentation](https://typelevel.org/cats/faq.html#tailrecm)
or for instance in [Eugene Yokota's](http://eed3si9n.com/herding-cats/tail-recursive-monads.html) series
of articles *herding cats*.

Its signature and the associated Scaladoc is telling us everything we need:

```scala
/**
   * Keeps calling `f` until a `scala.util.Right[B]` is returned. (...)
   */
def tailRecM[A, B](a: A)(f: A => F[Either[A, B]]): F[B]
```

### The solution

We don't need to look any further, here is the better way that I was looking for:

```scala
def findM[G[_], A](list: List[A], p: A => G[Boolean])
  (implicit G: Monad[G]): G[Option[A]] = {
  list.tailRecM[G, Option[A]] {
    case head :: tail => p(head).map {
      case true  => Right(Some(head))
      case false => Left(tail)
    }
    case Nil => G.pure(Right(None))
  }
}

def collectFirstM[G[_], A, B](list: List[A], pf: A => G[Option[B]])
  (implicit G: Monad[G]): G[Option[B]] = {
  list.tailRecM[G, Option[B]] {
    case head :: tail => pf(head).map {
      case found @ Some(_) => Right(found)
      case None => Left(tail)
    }
    case Nil => G.pure(Right(None))
  }
}
```

## Foldable#foldM

After searching the Cats Gitter channel, I found out that a more powerful and generic way would be to use `foldM`.
While a bit scarce on details, the Scaladoc mentions short-circuiting ability. Unfortunately its
implementation isn't of much help, and old messages on Gitter were pointing to the `Foldable` test suite
for inspiration.

There are several examples, but the common idea is to perform a monadic fold within a *fail-fast* data type
and take advantage of the failure to signal that we're done. Given that we do need to store the result
somewhere, a logical choice is to use `Either`. There's a good example in
[the test suite](https://github.com/typelevel/cats/blob/c35bc68cbdf0e722005a853ab7a884daf065add3/tests/shared/src/test/scala/cats/tests/FoldableSuite.scala#L660):

```scala
implicit val F = foldableStreamWithDefaultImpl
val ns = Stream.continually(1)
val res = F.foldLeftM[Either[Int, ?], Int, Int](ns, 0) { (sum, n) =>
  if (sum >= 100000) Left(sum) else Right(sum + n)
}
assert(res == Left(100000))
```

In plain words, we sum through an infinite stream of `1`s, carry the accumulator in a `Right` until
we've reached 100,000 and yield a `Left` that holds the final tally.

Our case is a bit simpler as we don't need to accumulate anything. However, since we're dealing with
effectful types, I ended up resorting to `EitherT`.

```scala
def findM[G[_], A](list: List[A], p: A => G[Boolean])
  (implicit G: Monad[G]): G[Option[A]] = {
  val fold = list.foldM[EitherT[G, Option[A], ?], Option[A]](None) { (_, elem) =>
    EitherT.right(p(elem)).flatMap {
      case false => EitherT.rightT[G, Option[A]](None)
      case true  => EitherT.leftT[G, Option[A]](Some(elem))
    }
  }
  fold.value.map {
    case Left(some) => some
    case _ => None
  }
}

def collectFirstM[G[_], A, B](list: List[A], pf: A => G[Option[B]])
  (implicit G: Monad[G]): G[Option[B]] = {
  val fold = list.foldM[EitherT[G, Option[B], ?], Option[B]](None) { (_, elem) =>
    EitherT.right(pf(elem)).flatMap {
      case None => EitherT.rightT[G, Option[B]](None)
      case found @ Some(_) => EitherT.leftT[G, Option[B]](found)
    }
  }
  fold.value.map {
    case Left(some) => some
    case _ => None
  }
}
```

I find this a lot clunkier and I'll stick to calling `tailRecM` directly in our production code. But
maybe I'm missing a simpler way to do it.

## Testing

Following Cats test suite, we can test the short-circuiting in a similar manner:

```scala
def bomb[A]: A = sys.error("boom")

val stream: Stream[Foo] = Foo(1) #:: Foo(2) #:: Foo(3) #:: Foo(4) #:: bomb[Stream[Foo]]
```

You'll have to replace `List[A]` with `Stream[A]` in both methods, or make them generic and accept any
collection `F[A]` given a `Foldable` type class instance. I'll leave the exercise to the reader. 😃

## Comments

Feel free to [contact me](/about/).
