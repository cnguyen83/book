## Traits: Defining Shared Behavior

Traits allow us another kind of abstraction: they let us abstract over behavior
that types can have in common. A *trait* tells the Rust compiler about
functionality a particular type has and might share with other types. In
situations where we use generic type parameters, we can use *trait bounds* to
specify, at compile time, that the generic type may be any type that implements
a trait and therefore has the behavior we want to use in that situation.

> Note: *Traits* are similar to a feature often called 'interfaces' in other
> languages, though with some differences.

### Defining a Trait

### Implementing a Trait on a Type

### Trait Bounds

<!-- Return to the example of the `largest` function here and discuss how using
`<` in the function body means we need to use `T: PartialOrd`. Then maybe show
how to implement `PartialOrd` for a struct? Throw a `clone` in to get around
needing to specify `Copy` too, maybe, or just specify `Copy` and then either
getting rid of `Copy` or `clone` could segue into lifetimes? /Carol -->





<!-- Not rearranged yet from here to the end /Carol -->






Here's an example definition of a trait named `Printable` that has a method
named `print`:

Filename: src/lib.rs

```rust
trait Printable {
    fn print(&self);
}
```

Listing 10-7: A `Printable` trait definition with one method, `print`

We declare a trait with the `trait` keyword, then the trait's name. In this
case, our trait will describe types that can be printed. Inside curly braces we
declare a method signature, but instead of providing an implementation, we put
a semicolon after the signature. A trait can have multiple methods in its body,
with the method signatures listed one per line and each line ending in a
semicolon. This has declared the trait `Printable`, defined as "methods that
are able to print values".

<!--Why do we do it like this, with the semicolon instead of
implementation---is that what makes it generic? (above) I'm also still quite
unclear on what a trait is/does -- I added this last line to try to wrap up
what this Printable trait is for, but I'm not confident in that, can you please
check and change? -->

Now we have the trait written we can implement it on a type. We use the `impl`
keyword, and we must also specify the trait name. Listing 10-8 shows an example
implementing the `Printable` trait from Listing 10-7 (that only has the `print`
method) for a `Temperature` enum:

Filename: src/lib.rs

```rust
enum Temperature {
    Celsius(i32),
    Fahrenheit(i32),
}

impl Printable for Temperature {
    fn print(&self) {
        match *self {
            Temperature::Celsius(val) => println!("{}°C", val),
            Temperature::Fahrenheit(val) => println!("{}°F", val),
        }
    }
}
```

Listing 10-8: Implementing the `Printable` trait on a `Temperature` enum

<!-- I think this code needs a little more talking through, point out what's
new and what we're doing. For eg, where does `match` come in? -->

Inside the `impl` block, we specify definitions for the trait's methods in the
context of the specific type. In the same way `impl` lets us define methods,
we've used it to define methods that pertain to our trait.

We can call methods that our trait has defined just like we can call other
methods:

<!--I'm afraid I'm still not clear on what this will actually do, the t.print
command -- just print the value of t? And where is the trait brought into
scope? I see the Temperature enum, but not the Printable trait. Perhaps a
little more explanation after this example would help make this all clearer -->

Filename: src/main.rs

```rust
fn main() {
    let t = Temperature::Celsius(37);

    t.print();
}
```

Note that in order to use a trait's methods, the trait itself must be in scope.
If the definition of `Printable` was in a module rather than within the same
program, it would need to be defined as `pub` and we would need to `use` the
trait in the scope where we wanted to call the `print` method. This is because
it's possible to have two traits that both define a method named `print`, and
is our `Temperature` enum implemented both Rust wouldn't know which `print`
method we inteded with `use`ing it first.

### Trait Bounds

This technique---defining traits with methods and implementing the trait methods
on a particular type---gives Rust more information than just defining methods
on a type directly. We are telling Rust that the type that implements the trait
can be used in places where the code specifies that it needs some type that
implements

<!--I am again finding this hard to follow, this line above is quite
circuitous! Any way to slow it down, map it out? Are we saying:

We are telling Rust that where we use the type that implements the trait in our
code, that type must have the specific functionality defined by the trait---so
if any type that doesn't have that functionality is used the program will not
compile.

? So by applying a trait bound, we are saying: " any type T must be a value
that is printable"? What values would that exclude?
 -->

a trait. To illustrate this, try out Listing 10-6, which has a `print_anything`
function definition similar to the `show_anything` function from Listing 10-3,
but this time it has a *trait bound* on the generic type `T` and uses the
`print` function from the trait. A trait bound constrains the generic type so
that is can only be a type that implements the trait specified, and not just
any type. With the trait bound, we're then allowed to use the trait method
`print` in the function body:

<!-- So when we used this before it failed because we didn't use a trait bound,
is that right? But why did that cause it to fail previously? I'm not clear on
that. What are the specific situations where trait bounds are required? -->

Filename: src/lib.rs

```rust
fn print_anything<T: Printable>(value: T) {
    println!("I have something to print for you!");
    value.print();
}
```

Listing 10-6: A `print_anything` function that uses the trait bound `Printable`
on type `T`

You specify a trait bound within angle brackets in the type name declaration.
After the name of the type that you want to apply the bound to, you add a colon
(`:`) and then give the name of the trait. This function now specifies that it
takes a `value` parameter that can be of any type, as long as that type
implements the trait `Printable`. We need to specify the `Printable` trait in
the type name declaration to be able to call the `print` method that is part of
the trait.

Now we are able to call the `print_anything` function from Listing 10-6 and,
since we implemented the trait `Printable` on `Temperature` in Listing 10-5,
pass it a `Temperature` instance as the `value` parameter:

Filename: src/main.rs

```rust
fn main() {
    let temperature = Temperature::Fahrenheit(98);
    print_anything(temperature);
}
```

If we implement the `Printable` trait on other types, we can use them with the
`print_anything` method too. However, if we try to call `print_anything` with
an `i32`, which does *not* implement the `Printable` trait, we get a
compile-time error that looks like this:

```bash
error[E0277]: the trait bound `{integer}: Printable` is not satisfied
   |
29 | print_anything(3);
   | ^^^^^^^^^^^^^^ trait `{integer}: Printable` not satisfied
   |
   = help: the following implementations were found:
   = help:   <Point as Printable>
   = note: required by `print_anything`
```

Traits are an extremely useful feature of Rust. You'll almost never see generic
functions without an accompanying trait bound. There are many traits in the
standard library, and they're used for many, many different things. For
example, our `Printable` trait is similar tothe standard trait `Display`, which
is in fact how `println!` decides how to format things with `{}`. The `Display`
trait has a `fmt` method that determines how to format something.

<!-- Didn't we discuss the display trait in an earlier chapter, when something
wasn't compiling because we hadn't called it? I can't remember which one, but
it would be useful to cross-ref that here -->

Listing 10-7 shows our original example from Listing 10-3, this time using the
standard library's `Display` trait in the trait bound on the generic type:

Filename: src/lib.rs

```rust
use std::fmt::Display;

fn show_anything<T: Display>(value: T) {
    println!("I have something to show you!");
    println!("It's: {}", value);
}
```

Listing 10-7: The `show_anything` function with trait bounds

Now that this function specifies that `T` can be any type as long as that type
implements the `Display` trait, this code will compile.

<!-- I wonder if it would simplify this section to use this show_anything
example throughout, rather than print_anything, to explain traits and trait
bounds? -->

### Multiple Trait Bounds

Each generic type can have its own trait bound. If we wanted to create a
function with two generics, 'T' and 'U', and give each generic its own trait
bound, `Display` and `Printable` respectively, the signature would look like
this:

```rust,ignore
fn some_function<T: Display, U: Printable>(value: T, other_value: U) {
```

As you might expect!

You can also specify multiple trait bounds on one type, by listing the trait
bounds in a list with a `+` between each trait. For example, here's the
signature of a function that takes a type `T` that implements both `Display`
and `Clone` ( another standard library trait we mentioned in Chapter XX):

```rust,ignore
fn some_function<T: Display + Clone>(value: T) {
```

<!-- does this mean the type must satisfy both bounds, so it's AND rather than
OR? -->

#### Organizing Multiple Trait Bounds with where

When trait bounds start getting complicated, the `where` syntax can help you
make your code a bit cleaner. The `where` syntax allows you to move the trait
bounds to after the function arguments list, so it doesn't clutter up your
function.

<!-- How does that help clean it up? Is this right, it just de-clutters? -->

And in fact, the error we got when we ran the code from Listing 10-3 referred
to it:

```bash
help: consider adding a `where T: std::fmt::Display` bound
```

<!-- So why does where create an error, if it's just for cleaning up code? -->

The definition of `show_anything` in Listing 10-X means the exact same thing as
the definition in Listing 10-7, but we've used `where` to move the trait bound
to the end of the `fn` line:

Filename: src/lib.rs

```rust
use std::fmt::Display;

fn show_anything<T>(value: T) where T: Display {
    println!("I have something to show you!");
    println!("It's: {}", value);
}
```

Listing 10-X: A tidier function with `where`

Instead of the `T: Display` trait bound going inside the angle brackets, it's
moved to the end of the function signature. This can make complex signatures
easier to read.

You can also place a `where` clause and its parts on a new line. Here's the
signature of a function that takes three generic type parameters that each have
the same two bounds:

```rust,ignore
fn some_function<T, U, V>(t: T, u: U, v: V)
    where T: Display + Clone,
          U: Printable + Debug,
          V: Clone + Printable
{
```

This makes is very clear that each generic type has two traits. Generic type
parameters and trait bounds are part of Rust's rich type system.

<!-- To wrap this up, can you summarize what trait bounds bring to Rust -- is
this a safety measure, for example, to prevent a program compiling with
incompatible data? -->

A final important kind of generic interacts with Rust's ownership and
references features, and they're called *lifetimes*.
