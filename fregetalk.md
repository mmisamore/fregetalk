% Pure Functional Programming\
on the JVM with Frege 
% Michael D. Misamore 
% March 2, 2016 

# Some History

<img src="church.jpg" style="height: 5em; float:right">
<img src="turing.jpg" style="height: 5em; float:right">

> - In the beginning (1936), Alan Turing invented his machines for formalizing
>   computations. Operational/mechanical in style.

> - Around the same time (1936), Alonzo Church completed work on a totally
>   different model: the &lambda;-calculus. Mathematical in style.

> - Church and Turing proved that the same computations can be expressed in
either system. 

> - Imperative programming languages (e.g. Fortran, C, Java) use Turing's
machines as basis; functional programming languages (e.g. Lisp, SML, Haskell) use
Church's &lambda;-calculus.

> - Operational vs. Mathematical leads to Imperative vs. Declarative, i.e. *Doing vs.
>   Being*

# What is Functional Programming? 

> - Declarative ("what the answer is"), not imperative ("steps to get to the
answer"). Order of expressions and declarations doesn't matter 

> - Primary code-reuse pattern is *function composition*

. . .
```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
```

> - The output of any function depends *only* on its input and not on any other state

> - Program execution consists of evaluating expressions, not executing statements

> - Examples of imperative/OO languages with some functional features: C#, Java 8, Groovy,
Scala, JavaScript. These are not functional programming languages.


# What is Pure Functional Programming?

> - Everything in Functional Programming, plus ...

> - Evaluating expressions does not cause side-effects (i.e. 
    <span style="color:red">observable interactions with state or the outside
    world</span>) - this is called *purity*, and it's a big deal

> - In any context, evaluating an expression more than once always gives the
same answer

> - Variable (re)assignment doesn't exist; all data structures are *immutable* and
*persistent*

> - Expressions representing I/O actions are specially labeled. Evaluating these
does *not* cause I/O to occur.

> - Order of I/O is separate from order of evaluation. This is another big
>   innovation.


# Purity Matters

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


# Advantages of Pure Functional Programming?

> - (Referential transparency) In any context, evaluating a pure expression
*always gives the same answer*: f() + f() == 2*f() is true!

> - This also enables *safe parallelism*: evaluating expressions in parallel
is no different than evaluating them in (any) order

> - No shared global state- no global state at all!

> - Don't have to worry about object lifetimes or null pointers

> - Unit testing generally easier: one input, one output

> - Encourages clean separation of pure business logic from I/O


# Disadvantages of Pure Functional Programming

> - Radically different than imperative and object-oriented programming (e.g. no
variable assignment, no objects, no for- or while-loops)

> - Most algorithms in literature are expressed in imperative terms (e.g. graph algorithms)

> - Algorithms that ostensibly depend on order of evaluation have to be expressed
    differently (but this often removes accidental complexity)

> - Bindings are required to interface with impure languages (e.g. Java) 

> - Non-strict evaluation strategies, if used, can sometimes result in undesirable space-leaks


# What is Frege?

> - Haskell for the JVM.

> - Haskell: General-purpose pure functional programming language with strong
>   static typing and non-strict evaluation strategy.

> - Lazy evaluation: a non-strict, *call-by-need* strategy for evaluating
>   expressions.

. . .
```haskell
g :: Int -> Int
g x = g (x+1)
f :: Int -> Int -> Int -> (Int,Int)
f x y z = (x, y)
```
Then "f 2 3 4" and "f 2 3 (g 4)" both give (2, 3) even though g 4 diverges!


# What is Frege (cont.d)?

> - Created by Ingo Wechsung in 2011. Development ongoing. Compiler written in
Frege. Transpiles to Java with a builtin lightweight runtime system.

> - Features a crazy-powerful type system: Hindley-Milner type inference, 
higher-order functions, algebraic data types, Haskell-style type classes,
pattern matching, higher-rank polymorphism

> - Documentation: [Frege Homepage](http://github.com/Frege/frege), [Standard
   Library API](http://www.frege-lang.org/doc/fregedoc.html)

> - Interop with Java via a foreign function interface: 
[Frege FFI Example](http://mmhelloworld.github.io/blog/2013/07/10/frege-hello-java/)

> - Syntax is very close to Haskell, so by learning Frege you are learning
Haskell (which is an excellent idea)

> - Problem: Functional programming is addictive!



# Control Your I/O 

> - The runtime (not the program) is responsible for producing I/O when
>   necessary. Expression evaluation, by itself, does not necessarily produce
>   I/O.

. . .
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

> - Expressions that could potentially ask for I/O must have return type *IO a* for some
*a*. This condition is <span style="color:red">statically enforced</span> by the type checker!

# Refactor at Will

> - Since all functions are pure by default, we can use mathematical methods to refactor them.

> - ```haskell
-- Filter a list of as using some lambda p
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = []
filter p (a:as) = if p a then a:(filter p as) else filter p as 
-- e.g. passing a lambda
filter (\x -> x > 10) [1..100] -- returns [11..100]
```

> - filter p . filter q = filter (\\x -> p x && q x)

> - Similarly: filter p . filter q = filter q . filter p (useful when p or q is
   harder to compute than the other). Such tricks lead to *equational reasoning*
   about program structure - optimizations are potentially much more aggressive
   than in imperative languages.

# Terse Data-Processing Pipelines

> - The function composition operator is called .

. . .
```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
```
Observe that *a*, *b*, and *c* here are type parameters, so . is polymorphic

. . .
```haskell
decompress :: InputStream -> [BusinessObject]
transform :: BusinessObject -> BusinessObject
compress :: [BusinessObject] -> BusinessObject

myTransform input = (compress . sortBy (comparing goodness) . 
filter (\x -> makesMoney x) . map transform . decompress) input
```

# Function Composition - Java(ish) version

<pre>
// Java-style signature and implementation for (.)
Function&lt;A,C&gt; dot( Function&lt;B,C&gt; g, Function&lt;A,B&gt; f ) {
   return new Function&lt;A,C&gt;() {
      C apply( A a ){
         return g.apply( f.apply( a ) );
      }
   };
}

// Note how we have to explicitly compose two functions at a time
List&lt;int&gt; someFn( List&lt;int&gt; l ) {
  return dot( take(10), dot(sort, filter) ).apply(l);
}
</pre>

# Concise Syntax for Data Types 

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

# Flexible Interfaces 

```haskell
-- Typeclass for equality
class Eq a where
   == :: a -> a -> a
   /= :: a -> a -> a
   a /= b = not (a == b) -- default implementation for /=

-- Eq instance for pairs (a,b)
instance Eq (Eq a, Eq b) => (a,b) where
   (a,b) == (c,d) = (a == c) && (b == d)

-- Eq instance for functions a -> a. Can one exist?
instance Eq (Eq a) => (a -> a) where
   f == g = undefined -- reduces to halting problem

-- Open-world assumption: can always add new typeclass instances 
-- to existing types after they are defined
```

# Generators/Explicit Streams are Obsolete

Just return infinite data structures instead.

. . .
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


# No Null Pointers 

```haskell
-- Ordinary function composition
(.) :: (b -> c) -> (a -> b) -> (a -> c)

-- Composition with possibility of failure at each step 
(<=<) :: (b -> Maybe c) -> (a -> Maybe b) -> (a -> Maybe c)

-- A transformation that works on values that may be Nothing 
fmap :: (a -> b) -> Maybe a -> Maybe b

-- A flattening operation: Just (Just a) = Just a, otherwise Nothing
join :: Maybe (Maybe a) -> Maybe a

-- Our definition of <=<
(g <=< f) a = join (fmap g (f a :: Maybe b) :: Maybe Maybe c)

-- ... And now we can compose functions that can fail!
return :: a -> Maybe a; return a = Just a
```

# No Null Pointers (cont'd)

```haskell
-- Definition of join and return for Maybe a
join (Just (Just a)) = Just a
join _ = Nothing
return a = Just a

-- Laws:
-- join is associative: join . join == join . fmap join
-- return is identity: join . fmap return == join . return == id

-- Checking identity laws:
(join . fmap return)(Just a) == join (Just (Just a)) == Just a ==
join (Just (Just a)) = (join . return)(Just a)

(join . fmap return)(Nothing) == join (Just (Nothing)) == Nothing ==
join (Just (Nothing)) = (join . return)(Nothing)

-- Any such thing with join and return satisfying these laws is a Monad
```

# Java Interop

> - Data structures in Frege are immutable with pure operations, but Java data
structures are <span style="color:blue">mutable</span> and operations have
<span style="color:red">side-effects</span>

> - In Frege, each mutable Java data structure is given its own local "state
thread" 

> - Within each state thread we can perform all the mutations we want, and then
freeze to a final immutable result

> - This means that we have to write *bindings* to Java libraries. Frege is
still relatively young: bindings do not yet exist for many popular libraries

> - This is an opportunity to work on some bindings! They are relatively easy to
write for intermediate functional programmers 


# Why Consider Frege?

> - [43k lines of Groovy to 8k lines of Haskell (80% drop in SLOC) with 64x performance and fewer bugs](https://www.youtube.com/watch?v=BveDrw9CwEg)

> - Leverage the power of existing Java libraries: call Java from Frege, and
    [Frege from Java](https://github.com/Frege/frege/wiki/Calling-Frege-Code-from-Java)

> - Take advantage of tons of largely compatible pre-existing Haskell code

> - Static type safety: find more bugs at compile time. The opposite of the
    "reflection everywhere" approach 

> - Learn something completely different

> - Null pointers don't exist (drop microphone)


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

* [Ingo Wechsung's Introduction to Frege](http://web.mit.edu/frege-lang_v3.21/Introduction_Frege.pdf)

* [Frege Gradle Example Project](https://github.com/mperry/frege-gradle-example)

* [My simplification of that example](https://github.com/mmisamore/frege-demo)

* [Frege Maven Plug-in](https://github.com/talios/frege-maven-plugin)

* [Frege IDE Plug-in for Eclipse](https://github.com/Frege/eclipse-plugin/wiki/fregIDE-Tutorial)

* [Learn You A Haskell](http://learnyouahaskell.com/)


# Questions?


