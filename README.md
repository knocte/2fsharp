# Guide for C# devs to learn F# real FAST

This guide is mostly samples based. It will take you 10minutes of your time
and by understanding it you will already get a hang of 80% of the language.

### Example 1: Basic function declarations and implementation

```
public int GiveMeTheLength(string input)
{
    var result = input.Length;
    return result;
}
```
becomes
```
let GiveMeTheLength(input) =
    let result = input.Length
    result
```

* All things are public by default unless you explicitly specify `private` modifier.
* Keyword `var` becomes `let` (which is also used to define functions/methods, not only to declare variables).
* You don't need the `return` keyword. The last element of the function is the value to be returned.
* No need for braces, it works via 4-space (or 2) indentation like Python.
* No need for semi-colons to denote the end of a line.
* Specifying types is always optional, except in very special cases when the compiler cannot infer them.

If you want to specify the types in the sample above, it would become:

```
let Length(input: string): int =
    let result: int = input.Length
    result
```

### Example 2: Basic keywords and operators

```
using System;

class MainClass
{
    void Main()
    {
        int exitCode = 0;
        if (incomingChar == Environment.NewLine)
            exitCode = 1;
        else if (!(incomingChar == String.Empty))
            exitCode = 2;
        else if (incomingChar != "\t" && incomingChar.Length > 1)
            exitCode = 3;
        else
            exitCode = 4;
        Environment.Exit(exitCode);
    }
}
```
becomes
```
open System

let mutable exitCode: int = 0
if incomingChar = Environment.NewLine then
    exitCode <- 1
elif not (incomingChar = String.Empty) then
    exitCode <- 2
elif (incomingChar <> "\t" && incomingChar.Length > 1) then
    exitCode <- 3
else
    exitCode <- 4
Environment.Exit(exitCode)
```
* The `using` keyword becomes `open`.
* The `if (x) return a; else if (y) return b; else return c;` pattern becomes `if x then a elif y then b else c`, without the need of parenthesis but with new keywords `then` and `elif`.
* Initial assignment (to a readonly constant) operator is `=`. If you need to re-assign a
new value to the same element, then you explicitly mark it as mutable and use the `<-`
operator.
* Thanks to the above, the `=` operator can be a comparison operator too (no need for
doubling it like in C#: `==`).
* Operator `!=` becomes `<>`.
* Operator `!` becomes `not`.
* Operators `&&` and `||` are same in F#.
* No need to enclose entry point of program in Main() function, just write your statements
in a .fsx script or write the statements in the last `.fs` file fed to the F# compiler.

In general, such a simple piece of code like the one in the example can be coded easily
without a mutable variable, just by doing readonly assignments, this way:

```
open System

let exitCode =
    if incomingChar = Environment.NewLine then
        1
    elif not (incomingChar = String.Empty) then
        2
    elif (incomingChar <> "\t" && incomingChar.Length > 1) then
        3
    else
        4

Environment.Exit(exitCode)
```

(This is similar to the use of the `?` operator in C#, but more succinct and easy to read.)


### Example 3: Basic collections

```
var intArray = new int[] { 1, 2, 3 };
var intList = new List<int>({ 4, 5, 6 });
IEnumerable<int> sequenceOfIntegers = intList;
var dictionary = new Dictionary<int,string>() {
    { "One", 1 },
    { "Two", 2 }
}
```
becomes
```
let intArray = [| 1; 2; 3 |]
let intList = [ 4 ; 5 ; 6 ]
let sequenceOfIntegers: seq<int> = intList
let dic: IDictionary<string,int> = dict [ ("One", 1); ("Two", 2) ]
```
* Commas become semicolons when declaring elements of an array/list/dictionary.
* `IEnumerable<T>` becomes `seq<T>` (short for "sequence").
* You use `dict` to initialize an `IDictionary<K,V>` collection, however in F#
you would rather use a `Map<K,V>` because the latter is immutable.


### Example 4: Basic blocks

```
try {
    TrySomething();
} catch (SomeException ex) {
    DoSomethingElse(ex);
} finally {
    MakeSureToCleanup();
}
```
becomes
```
try
    try
        TrySomething()
    with
    | :? SomeException as ex ->
        DoSomethingElse(ex)

finally
    MakeSureToCleanup()
```
The `catch` keyword becomes `with`. However, there are no `try-with-finally`
blocks! We have only `try-with` blocks and `try-finally` blocks. Therefore
the equivalent in F# would need nesting (like it's done in the example above).

You may think this is an F# downside but try-catch-finally blocks are extremely
rare, especially given the `using` construct (for `IDisposable`) in C#:

```
using (var reader = new StreamReader(someFile))
{
    DoStuff(reader);
}
```
which becomes
```
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* No need for nesting a sub-block when using `use`
* Therefore, the resource will be disposed when it goes out of scope (the function ends).


### Example 5: Avoiding nulls and ignoring things

```
void Check(SomeType someParam1, SomeType someParam2)
{
    if (someParam1 != null)
        stringBuilder.Append(someParam1.ToString());

    if (someParam2 != null)
        stringBuilder.Append(String.Empty);
}
```
becomes
```
let Check(someParam1: Option<SomeType>,
          someParam2: Option<SomeType>): unit =

    match someParam1 with
    | Some(someValue) -> // like 'as' in C#, you cast and want the value
        let str = someValue.ToString()
        ignore(stringBuilder.Append(str))
    | None -> ()

    match someParam2 with
    | Some(_) -> // like 'is' in C#, you don't care about the value
        stringBuilder.Append(String.Empty) |> ignore
    | _ -> ()

```
In C# you write null checks everywhere (no safety at compile time). In F#,
you do the null check in a safer way with an `Option<T>` type (similar to
`Nullable<T>` but better) and a match expression (pattern matching).

* When you don't want to return anything, in C# you use `void` which is metadata for specifying absence of a type, but in F#  you need to return a special type called `unit`, which only has one possible value: `()`. That's why generally `()` means doing nothing (as per the above code).
* A `match-with` block is almost like a switch block, but more succint because it includes the casting (to someValue).
* There are three ways of ignoring things:
  * For example, we don't care about the return value of Append(), in C# we just ignore it but in
F# you need to be explicit about ignoring it, using the `ignore()` magic function.
  * The underscore in a match expression: it's like a `default` in a C# `switch`.
  * The underscore in `Some(_)`, when we want to make sure the value is not None, but we don't care
about its contents (like an `is` operator in C#, instead of `as`).
* The pipe operator (like in bash) is `|>` (and it works like in bash). Then `ignore(x)` is the same as `x |> ignore`.



### Example 6: Basic types

This immutable C# class below is much easier to write in F#:

```
public class Foo
{
    public Foo (int bar, string baz)
    {
        this.bar = bar;
        this.baz = baz;
    }

    readonly int bar;
    public int Bar
    {
        get { return bar; }
    }

    readonly string baz;
    public string Baz
    {
        get { return baz; }
    }
}

class Static
{
    Foo CreateFoo()
    {
        return new Foo(42, "forty-two");
    }
}
```

because it's just one line:

```
type Foo = { Bar: int; Baz: string }

module Static =
    let CreateFoo () =
        { Bar = 42; Baz = "forty-two" }
```

So then:
* Classes without behaviour (like the above Foo) are called "Records", they seem similar to structs but they are still reference types and allocated on the heap. They are immutable (once you create them, you cannot change their values underneath).
* Static classes are "modules", like the "Static" type above.
* In F#, there's no need to use the keyword "new" when creating instances of new classes or structs, except if the class being created implements IDisposable.

### Example 7: Order is important, and circular dependencies are the root of all evil

As a C# developer, you know that this code compiles fine:

```
class Foo
{
    static void Bar()
    {
        Baz();
    }

    static void Baz()
    {
    }
}
```

Why wouldn't it? You may think. Sure.
And this also compiles:

```
class Foo1
{
    public static void Bar()
    {
    }

    static void Baz()
    {
        Foo2.Baz();
    }
}

class Foo2
{
    void Bar()
    {
        Foo1.Bar();
    }

    public static void Baz()
    {
    }
}
```

Maybe you understand already where I'm coming from. The last C# snippet compiles fine, because C# allows circular dependencies. However, this last statement is only half-true, because circular dependencies are valid to the C# language, but not valid in terms of .NET assemblies (you cannot reference an assembly A from B, if A already depends on B). The principles of modularity would forbid you to write code like this, just because in the future you would not be able to separate it into two assemblies (you could not place Foo1 in one assembly and Foo2 in a different assembly, because circular dependencies are not valid in .NET).

Then, what you need to learn from this is that F# is a language that, once again, prevents you to shoot yourself in the foot in this way, because circular references in the same assembly are **not** valid. How does it achieve this? By forcing you to declare type A before type B, in case the latter calls the former. Therefore, this equivalent code snippet in F# will fail to compile:

```
module Foo1 =
    let Bar() =
        ()

    let Baz() =
        Foo2.Baz()

module Foo2 =
    let Bar() =
        Foo1.Bar()

    let Baz() =
        ()
```

Why? The error will be:

* Error FS0039: The value, namespace, type or module 'Foo2' is not defined. (Referring to Foo1.Baz implementation.)

It's not fixable unless we simply stop using circular dependencies (despite an existing escape hatch via the `and` keyword which I will not explain in this guide, because it's too advanced); because the way that the F# compiler has to avoid circular dependencies within the same assembly is requiring everything that we depend to, to be declared earlier. This means that if some function A calls function B, then B needs to be declared before A (if they are in the same file, then B needs to be at the top and B at the bottom; if they are in different files, then `B.fs` needs to be listed earlier than `A.fs` in the `.fsproj` file.

Therefore, this smaller snippet equivalent to our very first C# sample, doesn't compile either:

```
module Foo =
    let Bar() =
        Baz()

    let Baz() =
        ()
```

It gives the error:

* Error FS0039: The value or constructor 'Baz' is not defined. (Referring to Foo1.Bar implementation.)

But as we just learned, this is easier to fix; just declare Baz first:

```
module Foo =
    let Baz() =
        ()

    let Bar() =
        Baz()
```

------------------------------------------------------

CONGRATS!! You already know enough to maybe understand 80% of F# code.
Or maybe 80% of simple F# code, which is the code that is being used,
for instance, in most F# scripts: easy code.

I could explain you how to build the equivalent of a classes (with behaviour,
constructors, properties) or structs (value types and stack allocated) in F#,
but most scripts don't even need types, they just need functions, values and calls.
