## Generic Data Types

Using generics where we usually place types, like in function signatures or
structs, lets us create definitions that we can use for many different data
types.

### Using Generic Data Types in Function Definitions

For instance, Listing 10-4 shows two functions. The first function is the one
we extracted in Listing 10-3 that finds the largest `i32` in a slice. The
second function finds the largest `char` in a slice:

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let result = largest(&numbers);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let chars = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&chars);
    println!("The largest char is {}", result);
#    assert_eq!(result, 'y');
}
```

<figcaption>

Listing 10-4: Two functions that differ only in the types in their signatures

</figcaption>
</figure>

Here, the functions `largest` and `largest_char` have the exact same body, so
it would be nice if we could turn these two functions into one and get rid of
the duplication. Luckily, we can do that by introducing a generic type
parameter!

To parameterize the types in the signature of the one function we're going to
define, we need to create a name for the type parameter, just like how we give
names for the value parameters to a function. We're going to choose the name
`T`. Any identifier can be used as a type parameter name, but we're choosing
`T` because Rust's type naming convention is CamelCase, and because type
parameter names tend to be short, often just one letter. Short for "type", `T`
is the traditional default choice.

When we use a parameter in the body of the function, we have to declare the
parameter in the signature. Similarly, when we use a type parameter name in a
function signature, we have to declare the type parameter name before then.
Type name declarations go in angle brackets between the name of the function
and the parameter list.

So the function signature of the generic `largest` function we're going to
define will look like this:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

We would read this as: the function `largest` is generic over some type `T`. It
has one parameter named `list`, and the type of `list` is a slice of values of
type `T`. The `largest` function will return a value of the same type `T`.

Listing 10-5 shows one `largest` function definition using the generic data
type in its signature, and shows how we'll be able to call `largest` with
either a slice of `i32` values or `char` values. Note that this code won't
compile yet!

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let result = largest(&numbers);
    println!("The largest number is {}", result);

    let chars = vec!['y', 'm', 'a', 'q'];

    let result = largest(&chars);
    println!("The largest char is {}", result);
}
```

<figcaption>

Listing 10-5: A definition of the `largest` function that uses generic type
parameters but doesn't compile yet

</figcaption>
</figure>

If you try to compile this code, you'll get this error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 -->
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
 -->
  |
5 |         if item > largest {
  |            ^^^^
```

The note mentions `std::cmp::PartialOrd`, which is a trait defined by the
standard library that the operator `>` uses. What this error is saying is that
the body of `largest` won't work for any type `T`. Because we want to compare
values of type `T` in the body, we can only use types that know how to be
ordered, which is what the trait `std::cmp::PartialOrd` defines. We're going to
come back to traits and how to specify that a generic type has a particular
trait in the next section. Let's set this example aside for a moment and
explore other places we can use generic type parameters first.

### Using Generic Data Types in Struct Definitions

We can define structs to use a generic type parameter in one or more of the
struct's fields with the `<>` syntax too. Listing 10-6 shows the definition and
use of a `Point` struct that can hold `x` and `y` coordinate values of any type:

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<figcaption>

Listing 10-6: A `Point` struct that holds `x` and `y` values of type `T`

</figcaption>
</figure>

The syntax is similar to using generics in function definitions. First, we have
to declare the name of the type parameter within angle brackets just after the
name of the struct. Then we can use the generic type in the struct definition
where we would specify concrete data types.

Note that because we've only used one generic type in the definition of
`Point`, what we're saying is that the `Point` struct is generic over some type
`T`, and the fields `x` and `y` are *both* that same type, whatever it ends up
being. If we try to create an instance of a `Point` that has values of
different types as in Listing 10-7, our code won't compile:

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<figcaption>

Listing 10-7: The fields `x` and `y` must be the same type

</figcaption>
</figure>

If we try to compile this, we'll get the following error:

```text
error[E0308]: mismatched types
 -->
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integral variable, found
  floating-point variable
  |
  = note: expected type `{integer}`
  = note:    found type `{float}`
```

When we assigned the integer value 5 to `x`, the compiler then knows for this
instance of `Point` that the generic type `T` will be an integer. Then when we
specified 4.0 for `y`, which is defined to have the same type as `x`, we get a
type mismatch error.

If we wanted to define a `Point` struct where `x` and `y` could have different
types but still have those types be generic, we can use multiple generic type
parameters. In listing 10-8, we've changed the definition of `Point` to be
generic over types `T` and `U`. The field `x` is of type `T`, and the field `y`
is of type `U`:

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<figcaption>

Listing 10-8: A `Point` generic over two types so that `x` and `y` may be
values of different types

</figcaption>
</figure>

Now all of our instances of `Point` are allowed!

### Using Generic Data Types in Enum Definitions

<!-- Possibly show an enum other than `Option<T>` as well as `Option<T>`,
possibly just show `Option<T>` /Carol -->

### Using Generic Data Types in Method Definitions

<!-- Show implementing a method on a *struct* that has a generic *field*, then
show a method on that struct where the method takes an argument that's also
generic and how that affects where the type parameters need to be declared
/Carol -->

### Performance of Code Using Generics

<!-- Move the monomorphization discussion here but make sure the references to
previous material make sense /Carol -->




<!-- Not rearranged yet from here to the end /Carol -->




We've mentioned generics in previous chapters, but we never dug into what
exactly they are or how to use them. A generic type is like a placeholder type:
in places where we specify a type, like function signatures or structs, we can
instead use a *generic* as a stand-in vlaue that represent an abstract set
rather than a concrete value.

### Syntax of Generics

A generic type is recognizable by its syntax: any time you see angle brackets,
`<>`, you're dealing with generics, like in Chapter 8 where we discussed
vectors with the type `Vec<i32>`. The type defined by the standard library for
vectors is `Vec<T>`. That `T` is a *type parameter*, and serves a similar
function as parameters to functions: you fill in the parameter with a concrete
type (i32, here), and that determines how the overall type works. In the same
way that a function like `foo(x: i32)` can be called with a specific value such
as `foo(5)`, a `Vec<T>` can be created with a specific type, like `Vec<i32>`.

### Enum Definitions Using Generics

Let's dive into generic data types in more detail.

<!-- What are we writing here? Why are we jumping to generics, can you set this
up a little? -->

We'll use our highest number in two lists examples, and write a program like
the one in Listing 10-X that removes duplication, but this time we'll extract a
generic data type instead of extracting a function.

<!--What are we using the option enum for, exactly? -->

We'll use the `Option<T>` enum that we first saw in Chapter 6. First, let's
solve our problem with a `Some` variant that can only hold an `i32`, shown in
Listing 10-4. We'll call this enum `OptionalNumber`:

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust
enum OptionalNumber {
    Some(i32),
    None,
}

fn main() {
    let number = OptionalNumber::Some(5);
    let no_number = OptionalNumber::None;
}
```

<figcaption>

Listing 10-4: Using an enum to find the largest number in two lists

</figcaption>
</figure>

<!-- Can you talk this through briefly, show how it works so they can see the
difference when we write the new function? Also, is that right, this is for 2
lists? -->

This works just fine for `i32`s, but for no other type. If we also wanted to
store `f64`s, we'd have to duplicate code to define a separate `Option` enum
type for that type, and any type we wanted the `Some` variants to be able to
hold. Listing 10-5 shows the enum `OptionalFloatingPointNumber` that can store
floating points:

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust
enum OptionalFloatingPointNumber {
    Some(f64),
    None,
}

fn main() {
    let number = OptionalFloatingPointNumber::Some(5.0);
    let no_number = OptionalFloatingPointNumber::None;
}
```

<figcaption>

Listing 10-5:

</figcaption>
</figure>

With what we currently know how to do in Rust, we would have to write a unique
type for every single kind of value we want `Some` or `None` to be able to
hold. The idea of "an optional value" is a more abstract concept than one
specific type, but we want our program to work for any type.

#### Removing Duplication by Extracting a Generic Data Type

Let's see how to get from two duplicated types to one generic type we can use
in our program. Here are the definitions of our two enums side-by-side:

```text
enum OptionalNumber {   enum OptionalFloatingPointNumber {
    Some(i32),              Some(f64),
    None,                   None,
}                       }
```

The two enums have different names, and a different data type in the variant
`Some`, but are otherwise the same.

To parameterize these enums we need to create a name for the parameters, just
like how we parameterize arguments to a function. In this case, we choose the
name `T`---any identifier can be used here, but we choose `T` because Rust's
default style is CamelCase,, and because type parameters they tend to be short,
often just one letter. Short for "type", `T` is the traditional default choice.

Let's replace the specific types in our `Some` variant definitions with `T`:

```text
enum OptionalNumber {   enum OptionalFloatingPointNumber {
    Some(T),                Some(T),
    None,                   None,
}                       }
```

The content of the enum definitions are now the same, but there's a problem:
we've *used* `T`, but not defined it. This would be similar to using an
argument in the body of a function without declaring it in the signature. We
need to tell Rust that we've introduced a generic parameter. We use the angle
bracket syntax to introduce generic parameters, like so:

```text
enum OptionalNumber<T> {   enum OptionalFloatingPointNumber<T> {
    Some(T),                Some(T),
    None,                   None,
}                       }
```

The `<>`s after the enum name indicate a list of type parameters, just like
`()` after a function name indicates a list of value parameters.

Now the only difference between our two `enum`s is in the name, and since we've
made them generic and no longer specific to integers or floating point numbers,
we can give them the same name, as in Listing 10-X.

```text
enum Option<T> {	enum Option<T> {
    Some(T),            Some(T),
    None,               None,
}                   }
```

Listing 10-X: The Option enum defition

Now they're identical! We've made our type fully generic. This definition is
also how `Option` is defined in the standard library. In plain English, we
might read this definition as: "`Option` is an `enum` with one type parameter,
`T`. It has two variants: `Some`, with a value with type `T`; and `None`, with
no value."

We can now use the same `Option` type whether we're holding an `i32` or an
`f64`:

```rust
let integer = Option::Some(5);
let float = Option::Some(5.0);
```

> Note: Here we've left in the `Option::` namespace for consistency with the
> previous examples, but actually since `use Option::*` is in the prelude, the
> namespace is not needed. We can instead use `Option` like this:
>
> ```rust
> let integer = Some(5);
> let float = Some(5.0);
> ```

When you recognize situations with almost-duplicate types like this in your
code, you can follow this process to reduce duplication and accept any type
using generics.

### Monomorphization at Compile Time

Understanding this refactoring process is also useful in understanding how
generics work behind the scenes: the compiler does the exact opposite of this
process when compiling your code. *Monomorphization* means taking code with
generic type parameters and generating code that is specific for each concrete
type used with the generic code.

<!-- I think I'm following, so monomorphization is the process that turns the
generic into a specific value type once indication of the type is given, is
that right? I'm not 100%, but maybe something like:

Monomorphization is the process that transforms the generic type into a
specific type, once a value is given to indicate what type the generic should be
? -->

Consider this code that uses the standard library's `Option` enum:

```rust
let integer = Some(5);
let float = Some(5.0);
```

When Rust compiles this code, it will perform monomorphization. The compiler
will read the values that have been passed to `Option` and see that we have two
kinds of `Option<T>`: one is `i32`, and one is `f64`. As such, it will expand
the generic definition of `Option<T>` into `Option_i32` and `Option_f64`,
thereby replacing the generic definition with the specific ones.

The monomorpohized version expanded out looks like the duplicated code we
started with at the beginning of this section:

Filename: src/main.rs

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

We can write the non-duplicated code using generics, and Rust will compile that
into code that specifies the type in each instance, meaning we pay no runtime
cost for using generics; it's just like we duplicated each particular
definition. This process is what makes Rust's generics extremely efficient at
runtime.


### Generic Functions and Methods

The `<>` syntax can also be used to introduce generic types in function or
method definitions. Place angle brackets for type parameters after the function
or method name and before the argument list in parentheses, like so:

```rust
fn generic_function<T>(value: T) {
    // code goes here
}
```

To refactor duplicated function definitions with generics, you use the same
process as we do for enums and structs. Consider these two side-by-side
function signatures that differ in the type of `value`:

```text
fn takes_integer(value: i32) {          fn takes_float(value: f64) {
    // code goes here                       // code goes here
}                                       }
```

We'll add a type parameter list that declares the generic type `T` after the
function names, then use `T` where the specific `i32` and `f64` types were:

```text
fn takes_integer<T>(value: T) {       fn takes_float<T>(value: T) {
    // code goes here                     // code goes here
}                                     }
```

At this point, only the names differ, so we could unify the two functions into
one:

```rust,ignore
fn takes<T>(value: T) {
    // code goes here
}
```

Two functions reduced into one! There's one problem though. The function
*definitions* work, but if we try to use `value` in code in the function body,
we'll get an error. For example, the function definition in Listing 10-8 tries
to print out `value` in its body:

Filename: src/lib.rs

```rust,ignore
fn show_anything<T>(value: T) {
    println!("I have something to show you!");
    println!("It's: {}", value);
}
```

Listing 10-8: A `show_anything` function definition that does not yet compile

Compiling this definition results in this error:

```bash
	error[E0277]: the trait bound `T: std::fmt::Display` is not satisfied
 --> <anon>:3:37
  |
3 |     println!("It's: {}", value);
  |                          ^^^^^ trait `T: std::fmt::Display` not satisfied
  |
  = help: consider adding a `where T: std::fmt::Display` bound
  = note: required by `std::fmt::Display::fmt`

error: aborting due to previous error(s)
```

This error mentions something we haven't learned about yet: traits. In the next
section, we'll learn how to make this compile.

<!--I feel like we should give some indication, even a small one, as to why
this didn't work before moving on---is this because our code implies some
condition our value doesn't meet? Otherwise, maybe we should introduce function
generics after discussing traits? It feels a little misleading -->
