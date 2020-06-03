# How Rust language secures your code

## Abstract

In this article, I will explain how Rust avoids various common bugs.
As the programming industry matures, more and more focus is being put on the security of the code.
I believe that it makes Rust and this article even more relevant as many of the bugs Rust avoids tend to lead to security vulnerabilities.

## Introduction

Rust seems to enjoy a great increase in popularity these days.
Recently, it even won Stack Overflow's "most loved" language for **the fifth time in a row**!
I [wrote](https://twitter.com/kixunil/status/1265980976305405952) a few words on why I believe it's the case on Twitter.
Now I'd like to dive deeper into the topic.

This article consists of two parts.
The first part explains why mistakes happen,
the second part shows how Rust prevents them.

## About mistakes

### A little bit about human psychology

Obviously, I'm no psychologist, so some things here might be less accurate.
That being said, I think that one doesn't have to be a psychologist to understand some key concepts presented here:

1. Programmers are human beings
2. Human beings are fallible

If you believe that humans are infallible then you're probably too young for this article and you should stop reading it. Seriously.
It has been shown many times that even the most talented, skilled programmers make mistakes.
Some of those mistakes even look stupid.
By why do the mistakes happen?
What in the human brain causes them?

Look at people around you.
Many of them do various kinds of mistakes that you might consider stupid.
Maybe they mismanage their finances, maybe procrastinate, maybe they are addicted to drugs, which may include social media.
Sometimes they just forget (important) things.
Or they are simply confused about things around them.
I guess many people would consider these mistakes "stupid" or "obvious".
Yet people do them.

I believe that the most common causes of mistakes are:

* Forgetting important facts, often caused by distractions in the environment.
* Not having enough time to deeply analyze every decision (sometimes "not having enough time" is imaginary feeling)
* Brain short-circuiting to do an activity that releases more dopamine short-term even if it's harmful/unoptimal long-term

Maybe worth explaining what I consider **imagined** lack of time.
Suppose you need to finish a project in 6 months.
It seems crazy soon, so you intentionally cut corners at the beginning.
By seemingly avoiding perfectionism, you experience terrible bugs five months later.
These bugs are absolutely unacceptable to be in the final product.
Solving the bugs takes you two weeks.
Taking the time to design the system to prevent it would take you one day.
Of course, it's difficult to predict the future this way.
Based on my experience, it seems that people tend to underestimate the impact of future problems more than overestimate it.

### Root causes of common programming bugs

Let's look at some examples of common bugs and think what could've caused them:

* Basic memory leaks - usually just forgetting to call `free()`.
  Sometimes it could be an intentional leak with the intent to "fix it later" then forgetting
  but I don't remember ever experiencing such a thing.
  And finally, if you use C++, it can be that writing `*` is twelve times easier than writing `unique_ptr<>`.
  Yes, when I was using C++ many years ago, I solved many leaks by religiously using smart pointers.
  That one time I couldn't be bothered with a smart pointer in a very short simple function was the single time I forgot to `free()`.
* Complicated memory leaks - sometimes reference cycles, but pretty often just unclear design and the blurry distinction between which part of the code is responsible for managing the memory.
  In the case of multithreaded code, this can become even more complex.
* Use-after-free - mostly forgetting about an object being freed, in complicated cases the (imagined) lack of time for deep analysis.
* Null pointer exception - most likely forgotten null check, but unclear/wrong documentation about whether a function can accept or return `NULL` could be another reason.
  Documentation problems are usually due to time pressure or the human brain avoiding a boring task.
* Passing a wrong type to a function - most often just forgetting, sometimes bad documentation, sometimes missing documentation.
* Bad error handling - most often just forgetting, bad documentation could be also the case.
  I've also seen an error stemming from inconsistent error handling practices in the codebase.
* Incorrect handling of edge cases - again, mostly forgetting and bad documentation.
* Off-by-one errors - forgetting the proper boundaries
* Various strange errors stemming from refactoring or copy-pasting - forgetting, sometimes imagined lack of time (copy-pasting is simple, but will almost always bite you)

I guess I could list more if I wanted but I believe this should be enough to understand the topic.

## How Rust avoids bugs

### The principles

First, let's think about how we could avoid the three root causes of bugs.
The most obvious thing people do, if they don't want to forget a fact is writing it down.
The ancient technology of writing is still relevant these days.
So one could write down every fact about a program.

This would definitely help, but it makes the next root cause even more problematic.
Writing down everything is time-consuming and then you also have to check the program against all facts you've written.
But people already invented technologies for handling long tasks.
In the case of writing, it's compression - more specifically abbreviations.
Instead of writing "This argument must not be `NULL`", one could just write "nonnull" and define what the abbreviation means in another file.
The second technology is automation.
If all the abbreviations are consistent, then a machine can "understand" them and check their validity for us.
For instance, it can warn you if you assign `NULL` into parameter declared as `nonnull`, or if you unconditionally assign a nullable variable into `nonnull` variable.
Many other checks are possible: accesses to uninitialized memory, wrong data types, calling `free()`...

Lastly, we need to hack the human brain to stop short-circuiting and do instant gratification things.
A great book called Atomic Habits says that you should make good things easy to do and bad things hard to do.
It should be hard to forget to write down the facts about the program.
It should be hard to ignore warnings...
One could conceivably hard-code the warnings as hard errors into the compiler.
Then if someone wanted to lower the quality of code below a certain level, the recompilation of the compiler would be required.
This should be sufficiently discouraging.
If there are special cases which are dangerous, but sometimes make sense, they could require some extra effort.
E.g. writing a bit more words, maybe special words that are meant as warnings.

I hope you see the ideas as reasonable solutions to those problems.
Even if not absolutely perfect, they should help to avoid many bugs.

Rust implements all of these ideas, as described below.

### Examples

Let's look at the examples above and see how they are prevented in Rust:

* Basic memory leaks - Rust tracks the usage of memory during compilation and automatically inserts deallocations at proper places.
  Some people call this "compile-time garbage collection".
  I like that term a bit, but I also worry that it could cause people to assign it some properties of run-time garbage collection.
  The most important difference is that it's deterministic.
  The exact algorithm is known and you can tell when stuff deallocates from looking at the code.
  (Admittedly, sometimes you need to look harder, but still, no other information is required.)
  Then obviously, since the decisions are made during compile time, it doesn't introduce run-time overhead.
  Neither in terms of increased CPU nor memory usage.
  And the best part is this also solves resource leaks!
* Complicated memory leaks - Rust forces you into having much cleaner memory management.
  This is not a silver bullet, but it helps a lot.
  While it can't prevent all memory leaks, it frees up your time,
  so you have more time to solve complicated memory leaks because you didn't spend it on simple memory leaks.
* Use-after-free - Rust tracks the lifetime of all objects and detects these kinds of problems.
  This sometimes requires additional annotations.
  Sometimes?
  Great, right? It optimizes the common cases so that you often don't actually need to write any abbreviations.
  The ultimate compression.
* Null pointer exception - safe Rust "doesn't have" `NULL` pointers.
  You can achieve all the objectives you would need `NULL` pointers for by using enums (AKA sum types, AKA algebraic data types).
  It may sound confusing, but in simple terms, Rust forces you to mark all "nullable" types (the opposite of the example above) as `Option<>`
  and then forces you to check all the accesses.
  Of course, you can check access once and then use the value multiple times if it's guaranteed it didn't change.
  As an additional feature, you can have optional integers too - not just pointers. In such a case, it obviously adds one-byte memory overhead
  but that's something you'd otherwise have to do by hand anyway.
* Passing a wrong type to a function - Rust tracks all the types in compile time.
  To not turn your code into a "type soup", it lets you omit the types in the cases when it can figure them out for itself.
  This is still done at compile time.
  No runtime overhead introduced.
* Bad error handling - Error handling is done very similarly to "`NULL` checks" The difference is that in the case of "missing value",
  you can have additional information about the error.
  You are forced to do error checks.
  You are forced to access values correctly.
* Incorrect handling of edge cases - The powerful type system allows us to encode many other aspects of the program.
  As an example, it happened to me that, when writing some networking code, I forgot to handle the case that a client could
  connect to the server and then immediately disconnect, without sending any data.
  This manifested as a type error, which I then handled properly and avoided such a stupid mistake.
* Off-by-one errors - unfortunately, Rust doesn't magically prevent all off-by-one errors as that'd be insanely difficult.
  Interestingly, it makes them a bit less likely by consistently using the same indexing mechanism
  and providing ways to avoid indexing completely.
  On top of that, it makes these errors have much less severe consequences by safely killing the program in the case of out-of-bounds accesses.
  And yes, you can avoid those bound checks if your code is correct and you compile with optimizations.
  I even made [a library](https://crates.io/crates/dont_panic) that could help you check if those bound checks were actually removed.
* Various strange errors stemming from refactoring or copy-pasting - the strong type system can easily detect problems stemming from messed up code.
  It even happens sometimes that when I get lost in the code and my thoughts, I will just attempt to compile the code and resolve the situation solely from errors.
  Yes, I know I should be more careful and split the refactoring into smaller chunks.
  I'm just a human and Rust is a great tool for humans. :)

That was the theory, let's look at some snippets:

```rust
let foo = 42.to_string(); // to string allocates
// There's no free, but foo will be freed!
```

Ok, that was easy, let's make an intentional reference cycle.
This is the shortest code that I could come up with:

```rust
use std::cell::Cell;
use std::rc::Rc;

enum Foo {
    Something(Rc<Cell<Foo>>),
    Nothing,
}

let foo = Rc::new(Cell::new(Foo::Nothing));
foo.set(Foo::Something(foo.clone()));
```

What? So long? What's that `Rc` and `Cell`?
As you can see, making a reference cycle is not trivial.
Compare with Python for instance:

```python
foo = list()
foo.append(foo)
```

While it's not guaranteed that a reference cycle will be clearly visible,
there's a good chance that you will clearly see `Rc<Cell<Type>>` with `Type` pointing to itself.
This kind of code smell could be analyzed and the leak detected.

Even if not perfect, it still helps.

Let's try use after free:

```rust
let foo = 42.to_string();
drop(foo);
println!("foo: {}", foo);
```

```
error[E0382]: borrow of moved value: `foo`

let foo = 42.to_string();
    --- move occurs because `foo` has type `std::string::String`, which does not implement the `Copy` trait
drop(foo);
     --- value moved here
println!("foo: {}", foo);
                    ^^^ value borrowed here after move
```

There is some funny terminology but you probably do understand that Rust tells us the exact place where we freed the value and where it's used afterward.
This works reasonably well even in more complicated cases.

What about "null pointers"?
Here's how we assign a null pointer pointing to 32-bit unsigned integer:

```rust
let ptr: Option<&u32> = None; // We have to specify the type, because Rust can't infer it from just `None`
```

And here's how we can't dereference it unsafely:

```rust
let foo = *ptr * 2;
```

```
type `std::option::Option<&u32>` cannot be dereferenced
```

Pretty straightforward, right? Here's how we do it safely:

```rust
if let Some(ptr) = ptr {
	let foo = *ptr * 2;
	// we checked it once, we can reuse it many times!
	println!("ptr is: {}", ptr);
}
```

I guess wrong types should be obvious to anyone who tried statically typed language:

```rust
let mut x = 42;
x = "foo";
```

Throws a type error.
However, what may surprise you is that this:

```
let mu x = 42u8;
x = 65535u16;
```

also fails to compile!
No silent overflows in conversions!
This is important because there actually were security vulnerabilities caused by silent conversions.

Showing a snippet for edge cases isn't easy but maybe I have something.
In most cases, a program is executed with at least one argument - its name.
In most cases.
Not all cases.

So if you attempt to acces the zeroth argument without checking its existance:

```rust
let program_name = std::env::args().next();
println!("{}", program_name);
```

```
std::option::Option<std::string::String>` doesn't implement `std::fmt::Display
```

Yep, `Option` again.

I don't think there's anything interesting in crashing the program apart from the fact that this:

```rust
let foo = [0, 1];
println!("{}", foo[2]);
```

Does **not** result in a Segmentation fault, but in a nice message about index ou of bounds.

I have no clue how to show you a snippet from refactoring.

## Conclusion

As you can see, Rust developers made a significant effort towards making various bugs less likely to happen.
This article is quite long, yet I barely scratched the surface.
I was considering adding more information but I was worried that it'd overwhelm the readers.
Hopefully, it makes a point already.

There's [a crowdfunding campaign](https://t.ln-ask.me) running for an online Rust workshop that I plan to create in case you're interested in deeper dive into the topic.
If it's funded by the deadline the resulting content will contain complete how-to covering most important things you need to know to be efficient at programming in Rust.
