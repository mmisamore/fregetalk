% Pure Functional Programming\
on the JVM with Frege 
% Michael D. Misamore 
% Sept 1, 2015 

# Some History

<img src="church.jpg" style="height: 5em; float:right">
<img src="turing.jpg" style="height: 5em; float:right">

> - In the beginning (1936), Alan Turing invented his "a-machines" for formalizing computations\

> - Around the same time (1936), Alonzo Church completed work on a totally different model for
formalizing computations: the &lambda;-calculus

> - Church and Turing proved that the same computations can be expressed in
either system. 

> - Imperative programming languages use Turing's machines as basis; functional
programming languages use Church's &lambda;-calculus.


# What is Functional Programming? 

> - Declarative ("what the answer is"), not imperative ("steps to get to the
answer"). Order of expressions and declarations doesn't matter 

> - Primary code-reuse pattern is *function composition*

. . .
```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
```

> - Extensive use of higher-order functions (i.e. functions that take other
functions as arguments)

> - The output of any function depends *only* on its input, and not on any other state

> - Program execution consists of evaluating one (typically huge) expression 


# What is Pure Functional Programming?

> - Everything in Functional Programming, plus ...

> - Evaluating expressions does not cause side-effects (e.g.
accessing DB, printing to screen, writing files, launching missiles, etc.)

> - In any context, evaluating an expression more than once always gives the
same answer

> - Variable assignment doesn't exist; all data structures are *immutable* and
*persistent*

> - A totally pure functional program does nothing but <span
style="color:red">generate heat</span>


# Purity - Why it Matters

    // Whoops.java
    public class Whoops {
       private int z = 0;
       private int f() { z++; return z; }

       private void g() {
          int x = f() + f();
          int y = 2*f();
          System.out.println(x + " == " + y);
       }

       public static void main(String[] args) { 
          Whoops w = new Whoops(); w.g(); 
       }
    }

    bash-3.2$ java Whoops
    3 == 6


# Disadvantages of Pure Functional Programming

> - Radically different than imperative and object-oriented programming (e.g. no
variable assignment, no objects, no for- or while-loops)

> - Lazy evaluation can lead to space leaks that are difficult to diagnose

> - Learning to think about everything in terms of composition of higher-order
functions takes time  

> - Producing fast immutable data structures is still an active research area

> - Most algorithms in literature are expressed in imperative terms 


# Advantages of Pure Functional Programming?

> - (Referential transparency) In any context, evaluating a pure expression
*always gives the same answer*: f() + f() == 2*f() is true!

> - This also enables *safe parallelism*: evaluating expressions in parallel
is no different than evaluating them in (any) order

> - We don't have to worry about global state, because there isn't any!

> - All squares are rectangles (since everything is immutable)

> - We also don't have to worry about object lifetimes or null pointers


# What is Frege?

> - A purely functional, strongly and statically typed, *lazily evaluated*
programming language for the JVM.  Modeled on Haskell, which doesn't currently
run on the JVM.

> - Lazy evaluation: a non-strict, *call-by-need* evaluation strategy implemented
using thunks. The special value &bot; = undefined inhabits every type.

. . .
```haskell
f :: Int -> Int -> Int -> (Int,Int)
f x y z = (x, y)
```
Then "f 1 2 3" and "f 1 2 undefined" both give (1, 2)

- Created by Ingo Wechsung in 2011. Development ongoing. Compiler written in
Frege

> - Compiles to Java. Interop w/ Java via foreign function interface


# What is Haskell?

> - A standardized, general-purpose, purely functional, strongly and statically
typed programming language with non-strict semantics

> - Features lazy evaluation and a crazy-powerful type system: global type
inference (Hindley-Milner), higher-order functions, algebraic data types, type
classes, pattern matching, higher-rank polymorphism, higher-kinded types

> - Friendly community: [Reddit](http://www.reddit.com/r/haskell/), 
[Haskell Cafe mailing list](https://mail.haskell.org/mailman/listinfo/haskell-cafe), FreeNode #haskell IRC 

> - Books and Tutorials: [Learn You a Haskell](http://learnyouahaskell.com),
[Real World Haskell](http://book.realworldhaskell.org), [Developing Web
Applications with Haskell and Yesod](http://www.yesodweb.com/book)

> - Libraries and Dev: [Hackage](http://hackage.haskell.org) (~6000 packages as
of 2014), [Stack](https://github.com/commercialhaskell/stack/wiki)

> - Problem: It's addictive 


# Tour of Frege syntax - Basic Functions

```haskell
--- Basic syntax. This is a documentation comment
module examples.Fibonacci where

--- The 'main' function
main :: [String] -> IO () -- type signature
main _ = println (take 10 (map fibonacci [0..]))

--- Here is a function definition
fibonacci :: Int -> Integer -- type signature
fibonacci 0 = 1 -- pattern matches
fibonacci 1 = 1
fibonacci n | n >= 2 = fibonacci (n-1) + fibonacci (n-2) -- guards
fibonacci _ = 0 -- wildcards
```

# Infinite data structures, where clauses

```haskell
--- All the Fibonacci numbers as a list, once and for all
fibs :: [Integer]
fibs = map f [0..] where
   f 0 = 1
   f 1 = 1
   f n | n >= 2 = f (n - 1) + f (n - 2)
   f _ = 0

main :: [String] -> IO ()
main _ = println (take 10 (filter (> 500) (map (+1) fibs)))
```
Output: [611, 988, 1598, 2585, 4182, 6766, 10947, 17712, 28658, 46369]


# List Comprehensions, boolean guards

```haskell
-- List comprehension and boolean guards
main _ = println (take 10 pythagoreanTriples)
pythagoreanTriples :: [(Integer,Integer,Integer)]
pythagoreanTriples = [ (a,b,c) | c <- [1..], a <- [1..c], b <- [1..c], 
                                 a <= b, a^2 + b^2 == c^2 ]
```
Output: [(3, 4, 5), (6, 8, 10), (5, 12, 13), (9, 12, 15), (8, 15, 17), (12, 16,
20), (7, 24, 25), (15, 20, 25), (10, 24, 26), (20, 21, 29)]

> - Observe how declarative this is: we are saying *what* the thing is, not *how*
to construct it

> - We still have to limit the search space for *a* and *b* to something finite


# Basic I/O in Frege

```haskell
-- Basic I/O
main :: [String] -> IO ()
main _ = sneakyIO `seq` do
   println "Hello, I'm Frege. What is your name?"
   name <- stdin.getLine
   println ("Nice to meet you, " ++ name ++ ". Let's hang out!")
   
sneakyIO :: IO ()
sneakyIO = println "I'll try to do IO too!"
```
> - The sneakyIO here is not so sneaky: it *does* get evaluated, but evaluation
doesn't produce side-effects!

> - Anything that could potentially produce IO must have return type *IO a* for some
*a*. This is <span style="color:red">statically enforced</span> by the type checker!


# Function Composition 

> - The function composition operator is called .

. . .
```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
```
Observe that *a*, *b*, and *c* here are type parameters, so . is polymorphic

> - Example: 
```haskell
take :: Int -> [a] -> [a]
sort :: [a] -> [a]
filter :: (a -> Bool) -> [a] -> [a]

someFn :: [Int] -> [Int]
someFn = take 10 . sort . filter (>20)
```

# Partial evaluation and function application

> - In Haskell (hence Frege), *every* function with an argument is unary, i.e. a
-> b -> c is really a -> (b -> c)

. . .
```haskell
map :: (a -> b) -> [a] -> [b]
* :: Int -> Int -> Int
2* :: Int -> Int
($) :: (a -> b) -> a -> b

map (2*) [1,2,3] == [2,4,6]

main _ = println $ map ((2*) . (2*)) [1,2,3]
```
Output: [4, 8, 12]


# Higher-order functions

```haskell
applyUntil :: (a -> Bool) -> (a -> a) -> a -> a
applyUntil p f a = if (p a) then a else applyUntil p f (f a)

hailStone :: Integer -> Integer
hailStone n | even n = n `div` 2
            | otherwise = 3*n + 1

main _ = println (applyUntil (== 1) hailStone 31)

-- or

main _ = println ((takeWhile (/= 1) . iterate hailStone) 31)
```
Output: [31, 94, 47, 142, 71, 214, 107, 322, 161, 484, 242, 121, 364, 182, 91,
274, 137, 412, 206, 103, 310 ...


# Data types in Frege

```haskell
-- Data structures
data Maybe a = Just a | Nothing
data Either a b = Left a | Right b
data BinTree a = Empty | Leaf a | Branch (BinTree a) (BinTree a)
data List a = Empty | Cons a (List a)
data NEList a = Singleton a | Cons a (NEList a)

-- Type synonyms
type MaybeInt = Maybe Int
type PartialMap a b = a -> Maybe b

-- Record types
data MyRecord = MyRecord { a :: String, b :: Int }
main _ = println ( MyRecord { a = "Hello", b = 3 }.a )
```

# Typeclasses (i.e. Interfaces)

```haskell
-- (a :+: b) :*: c == (a :*: c) :+: (b :*: c)
class SemiNearRing a where
   (:+:) :: a -> a -> a
   (:*:) :: a -> a -> a 

instance SemiNearRing Integer where
   x :+: y = x + y
   x :*: y = x * y

instance SemiNearRing [String] where
   xs :+: ys = xs ++ ys
   xs :*: ys = [ xi ++ " " ++ yj | xi <- xs, yj <- ys]

main _ = println $ ["twenty", "thirty"] :*: ["one", "two", "three"]
-- Output: ["twenty one", "twenty two", "twenty three",
--          "thirty one", "thirty two", "thirty three"]
```

# A Method for Dealing with Failure

```haskell
-- Ordinary function composition
(.) :: (b -> c) -> (a -> b) -> (a -> c)

-- Composition with possibility of failure at each step 
(<=<) :: (b -> Maybe c) -> (a -> Maybe b) -> (a -> Maybe c)

-- A transformation that only works on Just values, otherwise Nothing
fmap :: (a -> b) -> Maybe a -> Maybe b

-- A flattening operation
join :: Maybe (Maybe a) -> Maybe a

-- Our definition of <=<
(g <= f) a = join (fmap g (f a :: Maybe b) :: Maybe Maybe c)

-- ... And now we can compose functions that can fail!
return :: a -> Maybe a; return a = Just a
```

# A Method for Dealing with Failure (2)

```haskell
-- Definition of join and return for Maybe a
join (Just (Just a)) = Just a
join _ = Nothing
return a = Just a

-- laws:
-- join is associative: join . join == join . fmap join
-- return is identity: join . fmap return == join . return == id

-- Checking identity laws:
(join . fmap return)(Just a) == join (Just (Just a)) == Just a ==
join (Just (Just a)) = (join . return)(Just a)

(join . fmap return)(Nothing) == join (Just (Nothing)) == Nothing ==
join (Just (Nothing)) = (join . return)(Nothing)

-- Any such thing with join and return satisfying these laws is a Monad
```


# Memoization without State

```haskell
-- Build an immutable Java array of Integers by reusing 
-- values at previously computed indices (memoized array 
-- construction)
fibMemo :: JArray Integer
fibMemo = arrayCache f 100 where
   f 0 array = 1
   f 1 array = 1
   f n array = array.[n-1] + array.[n-2]

main _ = println $ fibMemo `elemAt` 99
-- Output: 354224848179261915075
```

> - Credit: Ingo, via personal correspondence on Github


# Foreign Function Interface to Java

> - Data structures in Frege are immutable with pure operations, but Java data
structures are <span style="color:blue">mutable</span> and operations have
<span style="color:red">side-effects</span>

> - In Frege, each mutable Java data structure is given its own local "state
thread" 

> - Within each state thread we can perform all the mutations we want, and then
freeze to a final immutable result

> - This means that we have to write *bindings* to Java libraries. Frege is
still relatively young: bindings do not yet exist for many popular libraries

> - This is an opportunity to work on some bindings!


# Some inspiration

> "Those who want really reliable software will discover that they must find means
> of avoiding the majority of bugs to start with, and as a result the programming
> process will become cheaper."

. . .

> "[T]he purpose of abstracting is not to be vague, but to create a new semantic level 
> in which one can be absolutely precise." 
 
. . .

> The Humble Programmer, Edsger W. Dijkstra, 1972

. . .

Slogan: "OO programming helps you to manage state. Functional programming helps you 
to eliminate state. Pure functional programming helps you to eliminate
unintended side-effects."
 

# References 

* [Ingo's Introduction to Frege](web.mit.edu/frege-lang_v3.21/Introduction_Frege.pdf)

* [Frege Maven Plug-in](https://github.com/talios/frege-maven-plugin)

* [Frege IDE Plug-in](https://github.com/Frege/eclipse-plugin/wiki/fregIDE-Tutorial)


# Questions?


