<!--This lifetime section was a bit of a struggle -- I had to read it through
and come back to the start to wrap my head around it; I think we could rethink
the ordering of the sections within it to make it clearer. I'd like to know up
front what a lifetime does/is for and how it's made, for example, perhaps
starting with:

## Lifetime Syntax
### Lifetime Annotations in Function Signatures

Can you have a look at the structure, perhaps do a little re-ordering here?
You'll see throughout where I've mentioned that it would be useful to hear a
particular section earlier.

-->

## Constraining Abstract Scopes with Lifetimes

<!-- Not quite happy with the section heading yet /Carol -->

Generic type parameters let us abstract over types, and traits let us abstract
over behavior. *Lifetimes* allow us to be generic over *scopes*. A lifetime is
the scope that an object is valid for, and ensure that we get no dangling
references in our programs.

<!-- What's the difference between scope and lifetime, then? Have we been
referring to lifetimes as scopes, before now? -->

Yes, it's a bit unusual, and will be different to tools you've used in other
programming languages. Lifetimes are, in some ways, Rust's most distinctive
feature.

Lifetimes are a big topic that can't be covered in entirety in this chapter, so
will just talk about the very basics of lifetimes here to get you familiar with
the concepts. Chapter 20 will contain more advanced information about
everything lifetimes can do.

### Lifetimes Prevent Dangling References

When we talked about references in Chapter 4, we left out an important detail:
every reference in Rust has a *lifetime*, which is the scope for which that
reference is valid. Most of the time lifetimes are implicit, but just like we
can choose to annotate types everywhere, we can choose to annotate lifetimes.

Lifetime annotation parameters tell Rust what it needs to know to be able to
analyze a function without knowing about all possible calling code. Lifetime
parameters specify generic lifetimes that will apply to any specific lifetimes
the function gets called with.

<!--Ah! This is very useful to know, I had an aha moment here, can we explain
this much earlier on? (below) So if a lifetime parameter can accept a reference
with any lifetime, how does giving the lifetime parameter help Rust analyze
it?-->

Lifetime annotations do not change how long any of the references involved
live. In the same way that functions can accept any type when the signature
specifies a generic type parameter, functions can accept references with any
lifetime when the signature specifies a generic lifetime parameter.

The main aim of lifetimes is to prevent dangling references, which will cause a
program to error. Consider the program in listing 10-8, with an outer scope and
an inner scope. The outer scope declares a variable named `r` with no initial
value, and the inner scope declares a variable named `x` with the initial value
of 5. Inside the inner scope, we attempt to set the value of `r` as a reference
to `x`. Then the inner scope ends and we attempt to print out the value in `r`:

```rust,ignore
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

Listing 10-8: An attempt to use a reference whose value has gone out of scope

When you compile this code, you'll get an error:

```text
	error: `x` does not live long enough
  --> <anon>:6:10
   |
6  |     r = &x;
   |          ^ does not live long enough
7  | }
   | - borrowed value only lives until here
...
10 | }
   | - borrowed value needs to live until here
```

The variable `x` doesn't "live long enough." Why not? Well, `x` is going to go
out of scope when we hit the closing curly brace on line 7, ending the inner
scope. But `r` is valid for the outer scope; its scope is larger and we say
that it "lives longer." If Rust allowed this code to work, `r` would be
referencing memory that was deallocated when `x` went out of scope. So how does
Rust determine that this code should not be allowed?

#### The Borrow Checker

The part of the compiler called the *borrow checker* compares scopes to
determine that all borrows are valid. Here's the same example from Listing 10-8
with some annotations:

```rust,ignore
{
    let r;         // -------+-- 'a
                   //        |
    {              //        |
        let x = 5; // -+-----+-- 'b
        r = &x;    //  |     |
    }              // -+     |
                   //        |
    println!("r: {}", r); // |
                   //        |
                   // -------+
}
```

Listing 10-9:

<!-- Just checking I'm reading this right: the inside block is the b lifetime,
correct? I want to leave a note for production, make sure we can make that
clear -->

We've annotated the lifetime of `r` with `'a` and the lifetime of `x` with
`'b`. As you can see, the inner `'b` block is much smaller than the outer `'a`
lifetime block. At compile time, Rust compares the size of the two lifetimes
and sees that `r` has a lifetime of `'a`, but that it refers to an object with
a lifetime of `'b`. The program is rejected because the lifetime `'b` is
shorter than the lifetime of `'a`-- the subject of the reference does not live
as long as the reference.

Let's look at an example does not try to make a dangling reference, and so runs
normally:

```rust
{
    let x = 5;            // -----+-- 'b
                          //      |
    let r = &x;           // --+--+-- 'a
                          //   |  |
    println!("r: {}", r); //   |  |
                          // --+  |
                          // -----+
}
```

Listing 10-10: A working lifetime reference

Here, `x` has lifetime `'b`, which in this case is larger than `'a`. This means
that `r` can reference `x`: Rust knows that the reference in `r` will always be
valid while `x` is valid.

Note that we didn't have to annotate the code with the lifetimes; Rust figured
them out for us. This is often the case, though there are situations when Rust
can't figure out the lifetimes and annotations are required. We'll look at one
situation now.

### Lifetime Annotation Syntax

<!-- It would be really useful here to have the note about lifetime annotations
not changing the lifetime, and let the reader know what they actually do -->

Lifetime annotations have a slightly unusual syntax, recognized by an
apostrophe `'`:

```rust,ignore
&i32 // a reference
&'a i32 // a reference with an explicit lifetime
```

Here `'a` is a *lifetime*: the `a` is the name, and the single apostrophe
indicates that this name is allocated to a lifetime. Unlike variables and other
Rust objects, lifetime names need to be declared *before* they're used.

<!-- Why do lifetime declarations need to come before their usage? And what is
the lifetime actually doing, here, how does it change/affect the reference
exactly? Do we need to give `a` a value for the lifetime to apply? I think we
need a little more info. -->

Here's a function signature with lifetime declarations and annotations:

```rust,ignore
fn some_function<'a>(argument: &'a i32) {
```

As you can see, lifetime annotations also go inside the generic angle brackets
after the function name.

<!-- so what is this annotation saying, exactly? That the lifetime only applies
to i32 types? Perhaps a real example the reader can run, here, would be
helpful? -->

We can even write functions that take both a lifetime declaration and a generic
type declaration:

```rust,ignore
fn some_function<'a, T>(argument: &'a T) {
```

This function takes one argument, a reference to some type, `T`, and the
reference has the lifetime `'a`. In the same way that we parameterize functions
that take generic types, we parameterize references with lifetimes.

### Lifetime Annotations in Struct Definitions

<!-- Why can't Rust figure out the lifetime itself here, what's different about
it? Can you lay that out? -->

When you have a struct with a field that holds a reference you need to annotate
the lifetimes yourself. It should look something like this:

```rust
struct Ref<'a> {
    x: &'a i32,
}
```

Again, you declare the lifetime name in the angle brackets where generic type
parameters are declared, because lifetimes are a form of generics.

In the examples in Listing 10-9 and 10-10, `'a` and `'b` were concrete
lifetimes: we knew exactly how long `r` and `x` would live. However, when we
write a function, it's unlikely that beforehand we'll know every single
argument it could be called with and how long they will be valid for. We have
be explicit about what we expect the lifetime of the argument to be (we'll
learn about how to know what you expect the lifetime to be in a bit).

<!--where in the code do we explain what we expect the lifetime to be, can you
point that out? How does annotating it change how the struct works/is used?-->

This is similar to writing a function that has an argument of a generic type:
we don't know all the types the arguments might be when the function gets
called. Lifetimes are the same idea, but being generic over the scope of a
reference, rather than a type.

### Lifetime Annotations in Function Signatures

When annotating lifetimes in functions, the annotations go on the function
signature, and not in any of the code in the function body. This is because
Rust is able analyze the code within the function without any help, but when a
function has references to or from code outside that function, the lifetimes of
the arguments or return values will potentially be different each time the
function is called. This would be incredibly costly and often impossible for
Rust to figure out. In this case, we need to annotate the lifetimes ourselves.

<!-- What is it that Rust needs to know, below? -->


To understand lifetime annotations in context, let's write a function that will
return the longest of two string slices. We want to be able to call this
function by passing it two string slices, and we want to get back a string
slice. The code in Listing 10-9 should print `The longest string is abcd` once
we've implemented the `longest` function:

Filename: src/main.rs

```rust
fn main() {
    let a = String::from("abcd");
    let b = "xyz";

    let c = longest(a.as_str(), b);
    println!("The longest string is {}", c);
}
```

Listing 10-9: A `main` function to find the longest string

Note that we want the function to take string slices (which are, as you
remember, references) because we don't want the `longest` function to take
ownership of its arguments, and we want the function to be able to accept
slices of a `String` (like `a`) is as well as string literals (`b`).

<!-- why is `a` a slice and `b` a literal? You mean "a" from the string "abcd"? -->

Refer back to the "String Slices as Arguments" section of Chapter 4 for more
discussion about why these are the arguments we want.

If we try an implementation of the `longest` function without annotating the
lifetime, it won't compile:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Instead we get the following error that specifies lifetimes as a cause:

```text
error[E0106]: missing lifetime specifier
   |
1  | fn longest(x: &str, y: &str) -> &str {
   |                                 ^ expected lifetime parameter
   |
   = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
```

The help text is telling us that the return type needs a generic lifetime
parameter on it because Rust can't tell if the reference being returned refers
to `x` or `y`. Actually, we don't know either, since in the `if` block in the
body of this function returns a reference to `x` and the `else` block returns a
reference to `y`!

To specify the lifetime parameters in this case we need to have the same
lifetime for all of the input parameters and the return type:

Filename: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

This will compile and will produce the result we want with the `main` function
in Listing 10-9.

The function signature now says that for some lifetime `'a` it will get two
arguments, both of which are string slices that live at least as long as the
lifetime `'a`. The function will return a string slice that also will last at
least as long as the lifetime `'a`. This is the contract we are telling Rust we
want it to enforce.

By specifying the lifetime parameters in this function signature, we are not
changing the lifetimes of any values passed in or returned, but we are saying
that any values that do not adhere to this contract should be rejected by the
borrow checker. This function does not know (or need to know) exactly how long
`x` and `y` will live, but only needs to knows that there is some scope that
can be substituted for `'a` that will satisfy this signature.

<!-- Would this restrict the function? These seem like strict parameters -->

### HEADING

<!-- I think we could use a heading here, seems like we wrapped up that example
(and gave quite a lot of info) -->

The exact way to specify lifetime parameters depends on what your function is
doing. For example, if we wanted our function to always returned the first
argument, rather than the longest string slice, we wouldn't need to specify a
lifetime on `y`. This code compiles:

Filename: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

We specify a lifetime for `a` and but for `y`. The lifetime parameter for the
return type needs to match the lifetime parameter of one of the arguments. If
the reference returned does *not* refer to one of the arguments, the only other
possibility is that it refers to a value created within this function, which
would be a dangling reference since the value will go out of scope at the end
of the function. Consider this attempted implementation of `longest`:

Filename: src/main.rs

```rust,ignore
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Even though we've specified a lifetime for the return type, this function fails
to compile, with the following error message:

```text
error: `result` does not live long enough
  |
3 |     result.as_str()
  |     ^^^^^^ does not live long enough
4 | }
  | - borrowed value only lives until here
  |
note: borrowed value must be valid for the lifetime 'a as defined on the block at 1:44...
  |
1 | fn longest<'a>(x: &str, y: &str) -> &'a str {
  |                                             ^
```

The problem is that `result` will go out of scope and get cleaned up at the end
of the `longest` function, and we're trying to return a reference to `result`
from the function. There's no way we can specify lifetime parameters that would
change the dangling reference, and Rust won't let us create a dangling
reference. In this case, the best fix would be to return an owned data type
rather than a reference so that the calling function is then responsible for
cleaning up the value.

Ultimately, lifetime syntax is about connecting the lifetimes of various
arguments and return values of functions. Once they're connected, Rust has
enough information to allow memory-safe operations and disallow operations that
would create dangling pointers or otherwise violate memory safety.

### Lifetime Elision

Now we know that every reference has a lifetime, and we need to provide the
lifetimes for functions that use references as arguments or return values.
However, in Chapter 4 we had a function in the "String Slices" section, shown
again in Listing 10-X, that compiled without lifetime annotations:

Filename: src/lib.rs

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Listing 10-X

The reason for this is historical: in early versions of pre-1.0 Rust, this
would not have compiled. Every reference needed an explicit lifetime. At that
time, the function signature would have been written like this:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

After writing a lot of Rust code, we found that some patterns developed when it
came to lifetimes. The Rust team

<!-- by pattern, do you mean you found that rust tended to calculate lifetimes without needing to be told? I'm not clear what patterns we mean. -->

noticed that the vast majority of code followed the pattern, and forcing
developers to use explicit lifetime syntax on every reference didn't lead to a
great developer experience.

The Rust team introduced *lifetime elision rules* to Rust's analysis of
references so that lifetime annotations weren't required as often. The elisions
rules don't provide full inference: Rust doesn't try to guess what you meant in
places where there could be ambiguity. The rules are a basic set of particular
cases, and if your code fits one of those cases, you don't need to write the
lifetimes explicitly. There are three main rules that these cases must comply
with.

Lifetimes on function arguments are called *input lifetimes*, and lifetimes on
return values are called *output lifetimes*. One rule relates to how Rust
infers input lifetimes.

A case doesn't require explicit annotations if:

<!-- does the case have to comply with all 3 rules, or just 1? -->

1. Each argument that is a reference gets its own lifetime parameter. In other
   words, a function with one argument gets one lifetime parameter: `fn
   foo<'a>(x: &'a i32)`, a function with two arguments gets two separate
   lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`, and so on.

And two rules relate to output lifetimes:

2. If there is exactly one input lifetime parameter, that lifetime is assigned
   to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.
3. If there are multiple input lifetime parameters, but one of them is `&self`
   or `&mut self`, then the lifetime of `self` is assigned to all
   output lifetime parameters. This makes writing methods much nicer.

If none of these three rules apply to the functions, then you must explicitly
annotate input and output lifetimes. Because these rules do apply in the
`first_word` function, we didn't have to specify any lifetimes.

<!-- which rules apply, all three? Does that need a little explanation? -->

These rules will cover the vast majority of cases, allowing you to write a lot
of code without needing to specify explicit lifetimes.

<!-- I think it would be useful to mention ealier on that most cases don't
require lifetime annotations, perhaps in the opening syntax section -->

### Lifetime Annotations in Method Definitions

<!-- Is this different to the reference lifetime annotations, or just a
finalized explanation? -->

Now that we've gone over the lifetime elision rules, defining methods on
structs that hold references will make more sense. The lifetime name needs to
be declared after the `impl` keyword and then used after the struct's name,
since the lifetime is part of the struct's type. The lifetimes can be elided in
any methods where the output type's lifetime is the same as that of the
struct's, because of the third elision rule. In Listing 10-X we have a struct
called `App` that holds a reference to another struct, `Config`, defined
elsewhere.

Filename: src/lib.rs

```rust
struct App<'a> {
    name: String,
    config: &'a Config,
}

impl<'a> App<'a> {
    fn append_to_name(&mut self, suffix: &str) -> &str {
        self.name.push_str(suffix);
        self.name.as_str()
    }
}
```

The `append_to_name` method does not need lifetime annotations even though the
method has a reference as an argument and is returning a reference; the
lifetime of the return value will be the lifetime of `self`.

### The Static Lifetime

There is *one* final special lifetime we need to discuss: `'static`. The
`'static` lifetime is the entire duration of the program. All string literals
have the `'static` lifetime:

```rust
let s: &'static str = "I have a static lifetime.";
```

The text of this string is stored directly in the binary of your program and
the binary of your program is always available. Therefore, the lifetime of all
string literals is `'static`.

<!-- How would you add a static lifetime (below)? -->

You may see suggestions to use the `'static` lifetime in error message help
text, but before adding it, think about whether the reference you have is one
that actually lives the entire lifetime of your program or not (or even if you
want it to live that long, if it could). Most of the time, the problem in the
code is an attempt to create a dangling reference or a mismatch of the
available lifetimes, and the solution is fixing those problems, not specifying
the `'static` lifetime.

## Summary

We've covered the basics of Rust's system of generics. Generics are the core to
building good abstractions, and can be used in a number of ways. There's more
to learn about them, particularly lifetimes, and we'll cover that in later
chapters as we go through. Let's move on to I/O functionality.
