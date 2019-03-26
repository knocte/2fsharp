# Guide for C# devs to learn F# real FAST

This guide is mostly samples based. It will take you 10minutes of your time
and by understanding it you will already get a hang of 80% of the language.

### Example 1: Basic function declarations and implementation

```csharp
public int GiveMeTheLength(string input)
{
    var result = input.Length;
    return result;
}
```
becomes
```fsharp
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

```fsharp
let Length(input: string): int =
    let result: int = input.Length
    result
```

### Example 2: Basic keywords and operators

```csharp
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
```fsharp
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

```fsharp
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

```csharp
var intArray = new int[] { 1, 2, 3 };
var intList = new List<int>({ 4, 5, 6 });
IEnumerable<int> sequenceOfIntegers = intList;
var dictionary = new Dictionary<int,string>() {
    { "One", 1 },
    { "Two", 2 }
}
```
becomes
```fsharp
let intArray = [| 1; 2; 3 |]
let intList = [ 4 ; 5 ; 6 ]
let sequenceOfIntegers: seq<int> = intList
let dic: IDictionary<string,int> = dict [ ("One", 1); ("Two", 2) ]
```
* Commas become semicolons when declaring elements of an array/list/dictionary.
* `IEnumerable<T>` becomes `seq<'T>` (short for "sequence").
* You use `dict` to initialize an `IDictionary<K,V>` collection, however in F#
you would rather use a `Map<'K,'V>` because the latter is immutable.

(NOTE: generic types need the quote character (') as a prefix, as you might have noted above.)


### Example 4: Basic blocks

```csharp
try {
    TrySomething(someParam);
} catch (SomeException ex) {
    if (SomeCondition(ex)) {
        DoSomethingElse(ex);
        throw new OtherException();
    } else
        throw;
} finally {
    MakeSureToCleanup(someParam);
}
```
becomes
```fsharp
try
    try
        TrySomething(someParam)
    with
    | :? SomeException as ex ->
        if SomeCondition(ex) then
            DoSomethingElse(ex)
            raise OtherException
        else
            reraise()

finally
    MakeSureToCleanup(someParam)
```
* The `catch` keyword becomes `with`.
* The `throw X;` clause becomes `raise X`, and an empty `throw;` becomes the function call `reraise()`.
* However, there are no `try-with-finally` blocks! We have only `try-with` blocks and `try-finally` blocks. Therefore
the equivalent in F# would need nesting (like it's done in the example above).

You may think this is an F# downside but try-catch-finally blocks are extremely
rare, especially given the `using` construct (for `IDisposable`) in C#:

```csharp
using (var reader = new StreamReader(someFile))
{
    DoStuff(reader);
}
```
which becomes
```fsharp
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* No need for nesting a sub-block when using `use`
* Therefore, the resource will be disposed when it goes out of scope (the function ends).


### Example 5: Avoiding nulls and ignoring things

```csharp
void Check(SomeType someParam1, SomeType someParam2)
{
    if (someParam1 != null)
        stringBuilder.Append(someParam1.ToString());

    if (someParam2 != null)
        stringBuilder.Append(String.Empty);
}
```
becomes
```fsharp
let Check(someParam1: Option<SomeType>, someParam2: Option<SomeType>): unit =

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

```csharp
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

static class FooFactory
{
    static internal Foo CreateFoo()
    {
        return new Foo(42, "forty-two");
    }
}
```

because it's just one line:

```fsharp
type Foo = { Bar: int; Baz: string }

module FooFactory =
    let internal CreateFoo () =
        { Bar = 42; Baz = "forty-two" }
```

So then:
* Classes without behaviour (like the above Foo) are called "Records", they seem similar to structs but they are still reference types and allocated on the heap. They are immutable (once you create them, you cannot change their values underneath).
* Static classes are "modules", like the "FooFactory" type above.
* In F#, there's no need to use the keyword "new" when creating instances of new classes or structs, except if the class being created implements IDisposable.

### Example 7: Order is important, and circular dependencies are the root of all evil

As a C# developer, you know that this code compiles fine:

```csharp
static class Foo
{
    static void Bar()
    {
        if (Baz())
            Bar();
    }

    static bool Baz()
    {
        return false;
    }
}
```

Why wouldn't it? You may think. Sure.
And this also compiles:

```csharp
static class Foo1
{
    public static void Bar()
    {
    }

    static void Baz()
    {
        Foo2.Baz();
    }
}

static class Foo2
{
    static void Bar()
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

```fsharp
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

It's not fixable unless we simply stop using circular dependencies (despite an existing escape hatch via the `and` keyword which I will not explain in this guide, because it's too advanced); because the way that the F# compiler has to avoid circular dependencies within the same assembly is requiring everything that we depend to, to be declared earlier. This means that if some function A calls function B, then B needs to be declared before A (if they are in the same file, then B needs to be at the top and A at the bottom; if they are in different files, then `B.fs` needs to be listed earlier than `A.fs` in the `.fsproj` file).

Therefore, this smaller snippet equivalent to our very first C# sample in this section, doesn't compile either:

```fsharp
module Foo =
    let Bar() =
        if Baz() then
            Bar()

    let Baz(): bool =
        false
```

It gives the error:

* Error FS0039: The value or constructor 'Baz' is not defined. (Referring to Foo.Bar implementation.)
* Error FS0039: The value or constructor 'Bar' is not defined. (Referring to Foo.Bar implementation.)

But as we just learned, this is easier to fix; just declare Baz first. And for a function to be able to call itself (which, in a way, it's also a cyclic dependency), we use the `rec` keyword (which means "recursive"):

```fsharp
module Foo =
    let Baz(): bool =
        false

    let rec Bar() =
        if Baz() then
            Bar()
```


### Example 8: delegates, anonymous methods, functions, oh my!

In the earlier versions of C#, the way to pass functions (and procedures, which are the functions that return `void`) was via delegate types and anonymous functions. See an example with 4 combinations:

```csharp
static class SomeOldCsharpClass
{
    delegate void ProcedureWithNoReturnValueAndNoArguments();

    static void DelegateReception1(ProcedureWithNoReturnValueAndNoArguments dlg)
    {
        dlg.Invoke();
    }

    static void SendingAnonymousMethodAsDelegate1()
    {
        DelegateReception1(delegate ()
        {
            Console.WriteLine("hello 1");
        });
    }

    delegate void ProcedureWithNoReturnValueAndOneArg(string foo);

    static void DelegateReception2(ProcedureWithNoReturnValueAndOneArg dlg)
    {
        string bar = "baz";
        dlg.Invoke(bar);
    }

    static void SendingAnonymousMethodAsDelegate2()
    {
        DelegateReception2(delegate (string foo)
        {
            Console.WriteLine("hello 2 " + foo);
        });
    }

    delegate int FunctionWithOneReturnValueAndNoArguments();

    static void DelegateReception3(FunctionWithOneReturnValueAndNoArguments dlg)
    {
        int result = dlg.Invoke();
    }

    static void SendingAnonymousMethodAsDelegate3()
    {
        DelegateReception3(delegate ()
        {
            Console.WriteLine("hello 3");
            return 3;
        });
    }

    delegate long FunctionWithOneReturnValueAndOneArg(double foo);

    static void DelegateReception4(FunctionWithOneReturnValueAndOneArg dlg)
    {
        double bar = 4.0;
        long result = dlg.Invoke(bar);
    }

    static void SendingAnonymousMethodAsDelegate4()
    {
        DelegateReception4(delegate (double foo)
        {
            Console.WriteLine("hello 4 " + foo);
            return 4;
        });
    }
}
```

That was all fine and well, but then new versions of C# came along, which made it less verbose and a bit more elegant (even thanks to local functions, as you will notice):

```csharp
static class SomeNewCsharpClass
{
    static void SendingAnonymousMethodAsDelegate1()
    {
        void DelegateReception1(Action dlg)
        {
            dlg.Invoke();
        }

        DelegateReception1(() =>
        {
            Console.WriteLine("hello 1");
        });
    }

    static void SendingAnonymousMethodAsDelegate2()
    {
        void DelegateReception2(Action<string> dlg)
        {
            string bar = "baz";
            dlg.Invoke(bar);
        }

        DelegateReception2((string foo) =>
        {
            Console.WriteLine("hello 2 " + foo);
        });
    }

    static void SendingAnonymousMethodAsDelegate3()
    {
        void DelegateReception3(Func<int> dlg)
        {
            int result = dlg.Invoke();
        }

        DelegateReception3(() =>
        {
            Console.WriteLine("hello 3");
            return 3;
        });
    }

    static void SendingAnonymousMethodAsDelegate4()
    {
        void DelegateReception4(Func<double, long> dlg)
        {
            double bar = 4.0;
            long result = dlg.Invoke(bar);
        }

        DelegateReception4((double foo) =>
        {
            Console.WriteLine("hello 4 " + foo);
            return 4;
        });
    }
}
```

The most interesting change is that delegate types using the `delegate` keyword became `System.Action`, `System.Action<TArg1>`, `System.Function<TResult>`, `System.Function<TArg1,TResult>` and so on. Anonymous methods via the same `delegate` keyword became lambdas via the `=>` symbol.

But good news! Functions in F# are a native citizen, so this all looks even simpler in this language (and local functions also work):

```fsharp
module SomeFsharpModule =

    let SendingAnonymousMethodAsDelegate1() =
        let DelegateReception1(dlg: unit->unit) =
            dlg()

        DelegateReception1(fun _ ->
            Console.WriteLine("hello 1")
        )

    let SendingAnonymousMethodAsDelegate2() =
        let DelegateReception2(dlg: string->unit) =
            let bar = "baz"
            dlg(bar)

        DelegateReception2(fun bar ->
            Console.WriteLine("hello 2 " + bar)
        )

    let SendingAnonymousMethodAsDelegate3() =
        let DelegateReception3(dlg: unit->int) =
            let result = dlg()
            ()

        DelegateReception3(fun _ ->
            Console.WriteLine("hello 3")
            3
        )

    let SendingAnonymousMethodAsDelegate4() =
        let DelegateReception4(dlg: double->int64) =
            let bar = 4.0
            let result = dlg(bar)
            ()

        DelegateReception4(fun bar ->
            Console.WriteLine("hello 4 " + bar.ToString())
            int64(4)
        )
```

As you can see, the equivalent of BCL's `System.Function` and `System.Action` become native F# syntax for denoting arguments and return values via the `->` symbol, e.g. `TArg1->TResult`. Remember, `void` is `unit` in F#, a dummy real type with only one possible value `()` that makes it moot to distinguish between functions and actions. Last but not least, C#'s `(...) => { ... }` becomes `fun ... -> ...`.


### Example 9: tuples, partial application and currification

Chances are, if you've never done any functional programming, you may be scared about some concepts from it such as "partial application" and "currification", but truth is, they are not so complex concepts, and to explain them properly we need to explain tuples first, and why it's not recommended to abuse them in F# (in fact, you cannot use partial application with tuples! more on this later).

Let's talk first about a C# snippet which has an `out` parameter:

```csharp
int anInteger;
if (int.TryParse(someString, out anInteger)) {
    DoSomethingWithAnInteger(anInteger);
} else {
    DoSomethingElse();
}
```

If you try to translate the above into F# from which what you've learned so far, you will first wonder: how can I declare a variable without assigning a value to it? Truth to be told, there's really no way to do this in F#. But then after knowing this, you would have the temptation to simply assign any dummy value to an `anInteger` variable, because after all, it would be overwritten by the `TryParse()` call, right? This would not be very elegant, especially because for this to work, we would need to mark the variable as `mutable` (so as to be able to override it with a second value later), which is not idiomatic F#. The real best way to do this in F# is just using what is called an "Active Pattern", which will convert the function call above into a function call that virtually returns two values simultaneously:

```fsharp
match int.TryParse(someString) with
| (true, anInteger) -> DoSomethingWithAnInteger(anInteger)
| (false, _) -> DoSomethingElse()
```

This active pattern above has provided syntax sugar to the F# compiler which converted the `bool TryParse(string,out int)` signature in something like `(bool,int) TryParse(string)`, where `(bool,int)` is a tuple! So, in F#, tuples of any length of elements can be created very easily this way, in any other scenario, without the need to use the `System.Tuple<X,Y,...>` type. In a way, it reminds me of the way this was also improved in the last versions of C# (where you could use tuples in function signatures too without the use of `Tuple` either).

Let's double check on what we mean. This would be with old C#:

```csharp
bool ReceiveTuple(Tuple<string,int> aTuple)
{
    var counter = aTuple.Item2++;
    Console.WriteLine(aTuple.Item1);
    var newTuple = new Tuple<string,int>(aTuple.Item1, counter)
    ReceiveTuple(newTuple);
    return true;
}
```

If we wanted to have a variable that points to this function, its type would need to be `Func<Tuple<string,int>,bool>`.

Then with new C# (under the hood, it compiles to `ValueTuple<X,Y,...>` elements):

```csharp
bool ReceiveTuple((string str, int i) aTuple)
{
    var counter = aTuple.i++;
    Console.WriteLine(aTuple.str);
    var newTuple = (aTuple.str, counter);
    ReceiveTuple(newTuple);
    return true;
}
```

Now to have a variable that points to this new function, its type would become `Func<ValueTuple<string,int>,bool>`.

With F#:

```fsharp
let rec ReceiveTuple(str: string, i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    let newTuple = (str, counter)
    ReceiveTuple (newTuple)
    true
```

In this case, the F# type that would let you reference this function would be `string*int->bool`; so this is a new symbol that we're learning now: unlike with other programming languages in which the asterisk character involves pointers, in this case it is just a separator of types in a tuple.

But have you noticed how tuples blend into what seemed to be normal parameters in F#? In fact, along all this guide up until now, all the methods we have written in F# that received more than one parameter, were actually using tuples, even if you might have not noticed. But then, you might think, can you write the above method without tuples in F# then? Yes you can, just omitting the comma, this way:

```fsharp
let rec ReceiveNonTuple (str: string) (i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    ReceiveNonTuple str counter
    true
```

What's the difference between the functions `ReceiveTuple` and `ReceiveNonTuple`? Both receive the same number of arguments, and with the same types. However, the first one has its parameters as an F# tuple, and the second one has parameters declared in an idiomatic-F# way (in "currified form"). Why is this more idiomatic in F#? Because `ReceiveTuple` cannot be used in partial application scenarios, while `ReceiveNonTuple` can be because it's using a currified style. In this case, the type of the function, instead of being `string*int->bool`, it is `(string->int)->bool`.

So why `(string->int)->bool` is better than `string*int->bool`? The latter allows for interoperability with C# (as it’s the way that parameters are passed at the CIL level), but the former allows partial application in a very straightforward and non-convoluted way (we will see, later, that partial application is also possible with C#, but in such a complex way that its benefits don’t outweigh the drawbacks of its poor readability/maintainability). So without further ado, let’s look at a very simple example of partial application: let’s suppose we want to create a Multiplication function that receives two integers and returns one integer:

```fsharp
let Multiply (x: int) (y: int): int =
    x * y
```

Now, if we wanted to write a function to double the value of a number without having to repeat any implementation detail from the Multiply function, we could simply write it this way:

```fsharp
let Double (x: int): int =
    Multiply 2
```

What happens when we only pass one argument to the function “Multiply”? Let’s look at its original signature: `(int->int)->int`. Currification laws tell us that using parenthesis in type expressions is actually not needed (or that placing them elsewhere results in an equivalent expression), which means that we can write it this way as well (`int->int->int`) or this way (`int->(int->int)`). Therefore, passing only one parameter to a function that originally received two parameters, actually results in returning another function. We can probably understand it better this way:

```fsharp
let Double (x: int): int =
    let doubleFunc = Multiply 2
    let result = doubleFunc x
    result
```

Or even this way (with types redundantly specified):

```fsharp
let Double (x: int): int =
    let doubleFunc: int->int = Multiply 2
    let result: int = doubleFunc x
    result
```

This is a too simple example to maybe make you convinced of how powerful and useful partial application is. But it’s the foundations of, for example, Dependency Injection in functional programming. You will probably only grasp the flexibility it allows, with time, but at least we can already show you its simplicity in F#, at least compared to C#, because this is how you would implement partial application with the latter:

```csharp
static Func<int, Func<int, int>> Multiply()
{
    return (x) => {
        (y) => {
            x * y;
        };
    };
}

static Function<int,int> Double()
{
    return Multiply().Invoke(2);
}

static int Main()
{
    var someInt = 3;
    return Double().Invoke(someInt);
}
```

Can you wrap your head around that? To me, a bit harder to do than reading the F# code.


### Example 10: string interpolation

In the early days of C# you might write things like:

```csharp
var aStringToShowToTheUser = String.Format("Hello {0}, I see you are {1} years old", name, age);
```

This has two problems: in large codebases where there are many variables and maybe many elements to include in a string, it could easily happen that we include a number for a variable not supplied (e.g. `{2}`) or that we provide an element which didn't have implicit conversion to string (or whose conversion was the standard and useless `ToString()` base method of `System.Object`, which I think simply prints the type of the object).

In F# we have an alternative that is much better:

```fsharp
let aStringToShowToTheUser = sprintf("Hello %s, I see you are %i years old" name age)
```

Why is this better? Because:
* If you supply less arguments (or more) than the ones needed to interpolate in the string, you will get a compiler error instead of an exception at runtime (fail faster!).
* You need to specify the type of the element inside the string, via the letter after the `%`, and if it doesn't match the type of the elemnt supplied for the same position, then you get a compiler error (instead of a useless string representation of the element).
* You need less parenthesis (for more info about this, see next section).



### Example 11: write less characters! especially good for readability of F# scripts

Now that you understood the difference between tuples and currified parameters in F#, and how the latter is always preferrable, you may understand that writing so many parenthesis was actually only needed to map things in tuples and is, in fact, a powerful inertia from C# devs that are starting to work with F#.

But as you start learning F# more and more, leaving the C# days behind, and writing always parameters in currified form, and writing less types (so that they can be inferred by the compiler), you realize how many less characters you need to type:

* Not so many parenthesis because you don't use tuples anymore.
* No need to use semicolons if you just use EOL separators.
* No need to use braces so much as you need them in C# (you only need them in F# when you deal with records).
* No need to use colon character `:` so many times if you let the F# compiler infer types more.
* Using the pipe operator `|>` more to avoid writing many parenthesis on the right side of a very long line.
* No need for parenthesis in `if` expressions in F# (as opposed to C#, which always needs them).

With all these in mind, we're now going to re-write again all F# samples of this guide but without all these redundant characters:

```fsharp
open System

let exitCode =
    if incomingChar = Environment.NewLine then
        1
    elif not (incomingChar = String.Empty) then
        2
    elif incomingChar <> "\t" && incomingChar.Length > 1 then
        3
    else
        4

Environment.Exit exitCode
```

```fsharp
let intArray = 
        [| 1
           2
           3 |]
let intList =
        [ 4
          5
          6 ]
let sequenceOfIntegers = intList
let dic = dict [ ("One", 1)
                 ("Two", 2) ]
```

```fsharp
try
    try
        TrySomething someParam
    with
    | :? SomeException as ex ->
        if SomeCondition ex then
            DoSomethingElse ex
            raise OtherException
        else
            reraise()

finally
    MakeSureToCleanup someParam
```

```fsharp
use reader = new StreamReader someFile
DoStuff reader
```

```fsharp
let Check someParam1 someParam2 =

    match someParam1 with
    | Some someValue -> // like 'as' in C#, you cast and want the value
        let str = someValue.ToString()
        stringBuilder.Append str |> ignore
    | None -> ()

    match someParam2 with
    | Some _ -> // like 'is' in C#, you don't care about the value
        stringBuilder.Append String.Empty |> ignore
    | _ -> ()

```

```fsharp
type Foo = 
    { Bar: int
      Baz: string }

module FooFactory =
    let internal CreateFoo () =
        { Bar = 42
          Baz = "forty-two" }
```

```fsharp
module Foo =
    let Baz() =
        false

    let rec Bar() =
        if Baz() then
            Bar()
```

```fsharp
module SomeFsharpModule =

    let SendingAnonymousMethodAsDelegate1() =
        let DelegateReception1 dlg =
            dlg()

        DelegateReception1(fun _ ->
            Console.WriteLine "hello 1"
        )

    let SendingAnonymousMethodAsDelegate2() =
        let DelegateReception2 dlg =
            let bar = "baz"
            dlg bar

        DelegateReception2(fun bar ->
            Console.WriteLine("hello 2 " + bar)
        )

    let SendingAnonymousMethodAsDelegate3() =
        let DelegateReception3 dlg =
            let result = dlg()
            ()

        DelegateReception3(fun _ ->
            Console.WriteLine "hello 3"
            3
        )

    let SendingAnonymousMethodAsDelegate4() =
        let DelegateReception4 dlg =
            let bar = 4.0
            let result = dlg bar
            ()

        DelegateReception4(fun bar ->
            Console.WriteLine("hello 4 " + bar.ToString())
            int64 4
        )
```

```fsharp
match int.TryParse someString with
| true, anInteger -> DoSomethingWithAnInteger anInteger
| false, _ -> DoSomethingElse()
```

```fsharp
let rec ReceiveNonTuple str i =
    let counter = i + 1
    Console.WriteLine str
    ReceiveNonTuple str counter
    true
```

```fsharp
let Multiply x y =
    x * y

let Double x =
    let doubleFunc = Multiply 2
    let result = doubleFunc x
    result
```

```fsharp
let aStringToShowToTheUser = sprintf "Hello %s, I see you are %i years old" name age
```


------------------------------------------------------

CONGRATS!! You already know enough to maybe understand 80% of F# code.

Or maybe 80% of simple F# code, which is the code that is being used, for instance, in most F# scripts: easy code.

I could explain you how to build the equivalent of classes (with behaviour, constructors, properties) or structs (value types and stack allocated) in F#, but... 1) it's not idiomatic F#; 2) if you're looking for an alternative safer scripting language, most scripts don't even need types, they just need functions, values and calls!

Anyway, I recommend as an important next step to watch this talk: https://www.youtube.com/watch?v=HQ887aOZITY
