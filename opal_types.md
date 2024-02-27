# Opal Type System

## Values and Types
In Opal, a **value** represents a unit of data that is operated on by Opal's underlying abstract machine. All runtime operations fundamentally work with values - they are a program's only input and output. They are used to affect the control flow of a program. The concrete data which a value represents may be known by the programmer at compile time, or by a program's end user prior to runtime, or it may be dynamically computed at runtime. In may persist through a program's runtime, or for a statically known portion of it, or for a dynamically determined portion of it, or may be entirely transient. In a less abstract sense, values are the binary patterns found in CPU registers, in memory, or at IO ports.

All values have a type. A value's type describes two things:

1. The way a value's bitwise representation should be interpreted.
2. The operations that can be performed on that value.

Values can only be used in contexts which expect values of their own type, or a type which they are compatible with. In most cases, attempting to use a value in a context which expects a value with an incompatible type produces a **type error** in the **type checking** phase of compilation. This is a fatal error.

Below, Opal's type system is described. It establishes the language's primitive types, the mechanisms by which new types may be established by the programmer, and their relationships to each other. The type system governs the compatibility of values in any particular context.

## Variables
A **variable** is used to name a value, which allows it be to be referred to across the variable's **scope**. All variables have a scope, which establishes both the context where it is extant and where it is legal to refer to. A variable must be *declared* before it can be used. The location of a variable's declaration determines its scope.

In Opal, a variable corresponds to a location in memory at which its value may be loaded from or stored to. The specific memory location that a variable refers to is affected by many factors, including the type of the value it names, or where and how it is declared - it may be absolute and known prior to runtime or it may be relative to a location only known at runtime.

Using variables allows for programmers to interact with values without worrying about their specific storage locations.

The declaration of a variable looks like this.

```rust
let name: Type = init
```

Here, `name` is the name - or **identifier** - of the variable. Its name is used to access the stored value. The type of the stored value is given by `Type`; in Opal, a variable can only refer to values of a type compatible with `Type`. The `=` is an **assignment operator**, which **assigns** the value `init` to `name`: it stores `init` in `name`'s memory location.

Here, `init` is `name`'s **initializer** and is an identifier (just as `name` is) but more generally, the initializer only needs to be an **expression** - of which an identifier is one **kind** of. Expressions come in different forms - they can be arithmetic operations, member accesses on another variable, or a function call. All expressions **evaluate** to a value with a concrete type. When a variable's identifier is used in an expression, it evaluates to its stored value.

## Constants
Like values, **constants** are used to name a value and are used in much the same way. They have scopes. They must be declared before usage. However unlike variables, constants directly correspond to a value - not a location in memory where the value is to be stored. This means that constants are not stored alongside other program data, such as declared variables. Instead, they exist only in the program's code. When a constant is used in an expression, the compiler can directly substitute its value for its name.

This makes constants a compile-time (comptime) construct, unlike variables which are runtime constructs.

The declaration of a constant looks like this. They cannot be reassigned after declaration.

```rust
const name: Type = init
```

Like variable declarations, `name` is the constant's identifier - unlike variables, it is not used to access a stored value. It's used to *substitute* its value (given by *init*) at usage sites. Compared to variables, a constant's initializer is restricted to values that can be known at compile time. This restricts it to the name of other constants, some arithmetic operations performed on other constants, or **literals**.

Literals can be thought of as predefined constants provided by Opal. As such, its illegal for any declaration to use a name which the language has reserved as a literal, similar to keywords. This includes variables, constants, or other type declarations discussed below. All literals are values with primitive types. The literals provided by Opal are discussed below.

## Primitive Types
Opal provides a minimal set of primitive data types. The language ways to create new types, which may be composed of these primitive types, or composed of other user defined types - or both.

### Booleans
A boolean is the most simple data type - it represents the logical idea of truth and falseness. There are exactly two values which have the `bool` type, which can be referred to by the literals `true` and `false`.

### Integers
Integers are whole valued numbers and correspond directly to integers as a mathematical concept. Because there are an infinite number of mathematical integers, yet types have an associated **size** that describes the amount of memory needed to represent its values, the integer types in Opal place restrictions on the range of mathematical integers that they can represent.

Opal has 6 builtin integer types, and each represents a different subset of mathematical integers.

Name  | Min    | Max Value
----- | ------ | ---------
`u8`  | 0      | 255
`i8`  | -128   | 127       
`u16` | 0      | 65535
`i16` | -32768 | 32767
`u32` | 0      | TODO
`i32` | TODO   | TODO

Integer literals are any valid mathematical integer expressed in base-10, or in base-2 if prefixed with `0b`, or in base-16 if prefixed with `0x`. Integer literals can also use underscores to separate digits into easier-to-read groups.

Attempting to assign an integer literal that is not within the range of an integer variable's bounds will result in a compilation error. Effectively, the type of an integer literal is **inferred** by the type of variable its being assigned to.

### Characters
Characters represent a single ASCII character. Character literals a single printable ASCII character surrounded with apostrophes: `'a'`.

### Arrays
An array represents a fixed number of values of compatible types which are stored contiguously in memory. Strictly speaking, an array is a **generic type** - there are any number of array types, which are unique and independent types. They are distinguished by the type of their elements, and the number of elements which they contain. These are called *generic parameters*. An array type is represented like this:

```python
[Type, Size]
```

Because all values have a known type, and because types are checked and validated at compile-time, array types must specify their **generic arguments** - which are `Type`, and `Size` in the context of the above example. `Type` must be a valid type; that is to say any primitive type discussed here, or type declared by the programmer. If `Type` is generic, its generic arguments must also be provided. `Size` must be an unsigned integer constant, as described above; it may be specified via an in-scope `const` identifier, or an integer literal.

Opal provides syntax for array literals. Unlike the above literals which are essentially predefined constants, these literals use special syntax in Opal's grammar.

```rust
let array: [u8, 2] = [5, 6]
```

### References
A reference type represents another variable. That is, the value of a reference type is a memory location, at which another value is located; the value *refers to* or *points to* another value.

A **reference** is the value of a reference type. Because a reference can be used to access the value of another variable, the type of the referred variable dictates what can be done via the reference. For that reason, the type of the referred variable must be encoded in the reference's own type.

So just like arrays, references are generic types. Reference types take one generic parameter, which is the type of the value it refers to. A reference type looks like this, where `Type` is the referenced type.

```haskell
^Type
```

Similar to arrays, generic types must specify their generic argument - so a particular reference can only be used to refer to values compatible with a single type. A `^Type` can only refer to `Type`s.

#### Mutability
In Opal, all variables are **immutable** by default. This means that once they have been assigned, they cannot be reassigned. Trying to reassign an immutable variable will result in a compilation error. Often, it is necessary or convenient to assign a new value to a variable; to **mutate** the variable.

The **mutability** of a variable is intrinsic to it, just as its scope and type are. For this reason, a mutable variable cannot be converted to an immutable variable and an immutable variable cannot be converted to a mutable variable. Mutability is specified in the declaration of a variable by placing the keyword `mut` before its name:

```rust
let mut name: Type = init
```

Because references can be used to access other variables, whether or not the referenced variable is mutable affects which operations can be performed via it. This means that a distinction needs to be made between references to immutable values and references to mutable values. That looks like this:

```rust
let foo: u8 = 1
let mut bar: u8 = 2

#// Reference to immutable integer
let ref: ^u8 = &foo

#// Reference to mutable integer
let mref: ^mut u8 = &bar  #// Note: the mut is part of the type!
```

Here the `&` behind `foo` and `bar` is an operator named **address-of**. So `&foo` is an expression which evaluates to the location of `foo` - in other words, `ref` refers to `foo`.

Note that both `mref` and `ref` are *themselves immutable*. This means that neither one can be updated to refer to other integer variables, regardless of *their* mutability. So care needs to be taken with where `mut` is placed in reference declarations. There are four possibilities.

```rust
let ref: ^u8 = ...          #// Immutable reference to immutable integer.
let mut ref: ^u8 = ...      #// Mutable reference to immutable integer.

let ref: ^mut u8 = ...      #// Immutable reference to mutable integer.
let mut ref: ^mut u8 = ...  #// Mutable reference to mutable integer.
```

### Strings
Strings are a series of characters that are stored contiguously in memory. Opal provides a string literal syntax that is analagous to character literal syntax. In place of apostrophe delimiters, quotation marks are used. Between, any number of characters can be placed.

Because string literals can be any size, and because all types in Opal have a fixed size, it is not possible to use a `str` value directly; it does not have a valid type.

String literals are compiled into Opal binaries as static data. String literal expressions evaluate to values of the special reference type `^str`. Unlike typical references which contain only a location, `^str`s contain both the length of the string and a pointer to the first data member. This is an implementation detail, however this distinguishes Opal from a language like C, which uses NULL terminated literals. This allows the length of a `^str` to be determined in constant O(1) time without any indirection; in C, determining the length of a NULL terminated `char*` is O(n).

Below is a `^str` variable declaration that initializes via a string literal expression.

```rust
let hello_world: ^str = "Hello, World!"
```

### Function Pointers
In Opal, functions have types which are determined by the type and arity of their parameters and return value. Functions come in two flavors: declared functions and anonymous functions. Declared functions have names and are created using the `def` keyword at module-level scope. Anonymous functions are created via expressions using a special syntax that produces values known as **lambdas**.

Below is an example of a declared function which has a single statement in it: an expression statement that produces a lambda value.

```py
def function() {
    (x: u8) -> u8 => { ... }
}
```

The lambda value is never assigned to a variable and so it is inaccessible on account of not having its own name. Variables can be declared which can refer to either type of function. These are called function pointers and their type mirrors that of the function declaration syntax.

```py
def function() {
    let ptr: def(u8) -> u8 = (x: u8) -> u8 => { ... }
}
```

When being used as an initializer, the parameter types and return type of an anonymous function can be ommitted - the Opal compiler will infer them to be the same as in the variable's type annotation. However, this does not mean that lambdas are dynamically typed - they do expect and return values of concrete types and mismatches will result in a type error at comptime. The same type inference can be used when assigning a declared function to a function pointer variable.

```py
def function(x: u8) -> u8 {
    let mut ptr: def(u8) -> u8 = function
    ptr = (x) => { ... }
}
```

Unlike references to other types, the address of a function does not need to be explicitly taken when assigning it to a function pointer.

In Opal, arguments must be passed to declared functions using the name of the parameters they corresponds to unless the parameter is marked as **anonymous** with `anon`. This allows for the arguments to be specified in an arbitrary order at the call site, and it makes source resilient to reordering of parameters in a function declaration.

However in the case of function pointers and anonymous function expressions, named parameters are not used. Two functions which take and return the same types in the same order have the same type, regardless of the name or anonymity of any parameter. This means both functions are compatible with the same function pointer. When a function is called through a pointer, the arguments are passed positionally as if all the parameters were anonymous. Similarly, all parameters are anonymous and must be passed positionally to anonymous functions - its like poetry, it rhymes - and they do not need to be declared as anonymous parameters.