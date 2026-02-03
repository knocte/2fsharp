# Guide for TypeScript devs to learn F# real FAST

This guide is mostly samples based. It will take you 15-30minutes of your time and by understanding it you will already get a hang of 80% of the most used elements of the language.

### Example 1: Basic function declarations and implementation

```typescript
function giveMeTheLength(input: string): number {
    const result = input.length;
    return result;
}
```
becomes
```fsharp
let GiveMeTheLength(input) =
    let result = input.Length
    result
```

* You don't need the `function` keyword. Use `let` instead (which is also used to declare variables, not only to define functions/methods).
* You don't need the `return` keyword. The last element of the function is the value to be returned.
* Indentations works via 4-space (or 2) like Python.
* Specifying types is always optional, except in very special cases when the compiler cannot infer them.

If you want to specify the types in the sample above, it would become:

```fsharp
let GiveMeTheLength(input: string): int =
    let result: int = input.Length
    result
```

Note that, like in TypeScript, all things are public by default. (To make things private in F#, you can use the `private` modifier.)


### Example 2: Basic keywords and operators

```typescript
import * as process from 'process';

let exitCode: number = 0;
if (incomingChar === "\n") {
    exitCode = 1;
} else if (!(incomingChar === "")) {
    exitCode = 2;
} else if (incomingChar !== "\t" && incomingChar.length > 1) {
    exitCode = 3;
} else {
    exitCode = 4;
}
process.exit(exitCode);
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
* The `import` keyword becomes `open`.
* The `{` and `}` braces disappear, and the condition's closing `)` followed by `{` becomes `then`
* Initial assignment operator is `=`. If you need to re-assign a new value to the same element, then you explicitly mark it as mutable and use the `<-` operator.
* Thanks to the above, the `=` operator can be a comparison operator too (no need for triple equals like in TypeScript: `===`).
* Operator `!==` becomes `<>`.
* Operator `!` becomes `not` in F#.
* Operators `&&` and `||` remain the same in F#.
* In F# there's no need for a top-level Main() function either, just write your statements
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


### Example 3: Basic collections

```typescript
const intDynamicArray: number[] = [1, 2, 3, 4, 5, 6];
const dictionary: Record<string, number> = {
    "One": 1,
    "Two": 2
};
```
becomes the following (take in account you don't need the type annotations in F# either, we just add them for reference):
```fsharp
let intList: List<int> = [ 4 ; 5 ; 6 ]
let intArray: array<int> = [| 1; 2; 3 |]
let dictionary: IDictionary<string,int> = dict [ ("One", 1); ("Two", 2) ]
```
* In TypeScript there is no concept of fixed-size arrays, so its arrays are like what other languages typically call lists. In F# an array and a list are different things, but both of them have a fixed-size length, due to the fact that all idiomatic collections in F# tend to be immutable. That said, it is easier to append (via the `@` operator) or to prepend (via the `::` operator) elements to lists, but this operation will always generate a new list, instead of mutate it like in TypeScript/JavaScript.
* With regards to syntax, commas become semicolons when declaring elements of an array/list/dictionary.
* Even though you can use `dict` to initialize an `IDictionary<string,int>` collection, in F# you would rather use a `Map<string,int>` because the latter is immutable.

NOTE: if you were to create a collection type whose elements had a type that is not defined yet (it could change at runtime), then you have to use the quote character to denote those generic types. Example: instead of `Map<string,int>`, `Map<'K,'V>`, meaning `'K` is the key's type and `'V` is the value's type.


### Example 4: Basic blocks

```typescript
try {
    trySomething(someParam);
} catch (err: unknown) {
    if (err instanceof SomeError) {
        if (someCondition(err)) {
            doSomethingElse(err);
            throw new OtherError();
        } else {
            throw err;
        }
    } else {
        throw err;
    }
} finally {
    makeSureToCleanup(someParam);
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
* In TypeScript, if you want to only catch one type of error, you need to do a runtime type-check inside the catch block; however in F# you can specify the type to catch via the `:?` syntax.
* The `throw X` becomes `raise X` in F#, but re-throwing becomes the function call `reraise()` (better than throwing the same exception because otherwise you lose stacktrace details related to the first throw).
* However, there are no `try-with-finally` blocks! We have only `try-with` blocks and `try-finally` blocks. Therefore the equivalent in F# would need nesting (like it's done in the example above).

You may think this is an F# downside but try-catch-finally blocks are extremely rare, especially given the `using` construct (for `Disposable` objects) in TypeScript:

```typescript
{
    using reader = new Reader();
    doStuff(reader);
}
```
which in F# becomes
```fsharp
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* No need for nesting a sub-block when using `use`
* Therefore, the resource will be disposed when it goes out of scope (the function ends).


### Example 5: Avoiding nulls (undefined) and ignoring things

```typescript
function check(someParam1: SomeType | null, someParam2: SomeType | null): void {
    if (someParam1 !== null) {
        stringBuilder.append(someParam1.toString());
    }

    if (someParam2 !== null) {
        stringBuilder.append("");
    }
}
```
becomes
```fsharp
let Check(someParam1: Option<SomeType>, someParam2: Option<SomeType>): unit =

    match someParam1 with
    | Some(someValue) ->
        let str = someValue.ToString()
        ignore(stringBuilder.Append(str))
    | None -> ()

    match someParam2 with
    | Some(_) ->
        stringBuilder.Append(String.Empty) |> ignore
    | _ -> ()

```
In TypeScript you write null/undefined checks everywhere (with limited safety at compile time unless you use strict null checks). In F#, you do the null check in a safer way with an `Option<'T>` type and a match expression (pattern matching).

* When you don't want to return anything, in TypeScript you use `void`, but in F# you need to return a special type called `unit`, which only has one possible value: `()`. That's why generally `()` means doing nothing (as per the above code).
* A `match-with` block is almost like a switch block, but more succint because it includes the casting (to `someValue`).
* There are three ways of ignoring things:
  * For example, let's say the `Append()` function returned some value, in TypeScript we just ignore it by not assigning it to a variable, but in F# you need to be explicit about ignoring it, using the `ignore()` magic function.
  * The underscore in a match expression: it's like an `else` clause in a TypeScript `if`.
  * The underscore in `Some(_)`, when we want to make sure the value is not None, but we don't care about its contents.
* The pipe operator (like `|` in bash) in F# is `|>` (and it works like in bash). Therefore `ignore(x)` can be also written as `x |> ignore`.


### Example 6: Basic types

This immutable TypeScript class below is much easier to write in F#:

```typescript
function createFoo(): Foo {
    return new Foo(42, "forty-two");
}

class Foo {
    private readonly _bar: number;
    private readonly _baz: string;

    constructor(bar: number, baz: string) {
        this._bar = bar;
        this._baz = baz;
    }

    get bar(): number {
        return this._bar;
    }

    get baz(): string {
        return this._baz;
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
* Classes without behaviour (like the above Foo) are called "Records", they are reference types and allocated on the heap. They are immutable (once you create them, you cannot change their values underneath).
* You cannot leave functions outside types like in TypeScript, you would need to place them inside "modules" (classes that don't generate instance, aka static classes), like the "FooFactory" type above.


### Example 7: Order is important, and circular dependencies are the root of all evil

As a TypeScript developer, you know that this code compiles fine:

```typescript
function bar(): void {
    if (baz()) {
        return bar();
    }
}

function baz(): boolean {
    return false;
}
```

Why wouldn't it? You may think. Sure.
And this also compiles:

```typescript
class Foo1 {
    private _x: number;
    private _y: number;

    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    bar(): void {
        console.log("bla");
    }

    baz(): void {
        const foo2 = new Foo2(2, 1);
        foo2.baz();
    }
}

class Foo2 {
    private _x: number;
    private _y: number;

    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    bar(): void {
        const foo1 = new Foo1(1, 2);
        foo1.bar();
    }

    baz(): void {
        console.log("bla");
    }
}
```

Maybe you understand already where I'm coming from. The last TypeScript snippet can be executed with no issues, because TypeScript allows circular dependencies. The principles of modularity would forbid you to write code like this, just because in the future you would not be able to separate this code easily into different libraries (you could not place Foo1 in one library and Foo2 in a different library, because you might receive "undefined" errors whose root cause is the use of circular dependencies).

Then, what you need to learn from this is that F# is a language that, once again, prevents you to shoot yourself in the foot in this way, because circular references in the same library are **not** valid. How does it achieve this? By forcing you to declare type A before type B, in case the latter calls the former. Therefore, this equivalent code snippet in F# will fail to compile:

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

It's not fixable unless we simply stop using circular dependencies (despite an existing escape hatch via the `and` keyword, or the `rec` keyword when applied to modules or namespaces, which I will not explain in this guide, because it's too advanced); because the way that the F# compiler has to avoid circular dependencies within the same assembly is requiring everything that we depend to, to be declared earlier. This means that if some function A calls function B, then B needs to be declared before A (if they are in the same file, then B needs to be at the top and A at the bottom; if they are in different files, then `B.fs` needs to be listed earlier than `A.fs` in the `.fsproj` file).

Therefore, this smaller snippet equivalent to our very first TypeScript sample in this section, doesn't compile either:

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

Functions are first-class citizens in TypeScript so passing them as parameters is easy, as can be seen in the following snippet (note there are 4 combinations depending on the signature of the function):

```typescript
function delegateReception1(procedureWithNoReturnValueAndNoArguments: () => void): void {
    procedureWithNoReturnValueAndNoArguments();
}

function proc1(): void {
    console.log("hello");
}

function sendingProcAsDelegate1(): void {
    delegateReception1(proc1);
}


function delegateReception2(procedureWithNoReturnValueAndOneArg: (arg: string) => void): void {
    const bar = "baz";
    procedureWithNoReturnValueAndOneArg(bar);
}

function proc2(foo: string): void {
    console.log("hello 2 " + foo);
}

function sendingProcAsDelegate2(): void {
    delegateReception2(proc2);
}


function delegateReception3(functionWithOneReturnValueAndNoArguments: () => number): void {
    const result = functionWithOneReturnValueAndNoArguments();
    console.log("hello 3 " + result);
}

function func3(): number {
    return 3;
}

function sendingFuncAsDelegate3(): void {
    delegateReception3(func3);
}


function delegateReception4(functionWithOneReturnValueAndOneArg: (arg: number) => number): void {
    const bar = 4.0;
    const result = functionWithOneReturnValueAndOneArg(bar);
}

function func4(foo: number): number {
    console.log("hello 4 " + foo);
    return 4;
}

function sendingFuncAsDelegate4(): void {
    delegateReception4(func4);
}
```

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

As you can see, we denote the function signatures concatenating the types via the `->` symbol, e.g. `TArg1->TResult`. Remember, `void` is `unit` in F#, a dummy real type with only one possible value `()` that makes it moot to distinguish between functions and procedures.


### Example 9: tuples, partial application and currification

Chances are, if you've never done any functional programming, you may be scared about some concepts from it such as "partial application" and "currification", but truth is, they are not so complex concepts, and to explain them properly we need to explain tuples first, and why it's not recommended to abuse them in F# (in fact, you cannot use partial application with tuples! more on this later).

Let's start with a TypeScript snippet with a function that receives a tuple and returns a tuple:

```typescript
const recursive = false;

function receiveAndReturnTuple(someTuple: [string, number]): [string, number] {
    const [str, i] = someTuple;
    const counter = i + 1;
    console.log(str);
    const newTuple: [string, number] = [str, counter];
    if (recursive) {
        receiveAndReturnTuple(newTuple);
    }
    return newTuple;
}
```

As you can see, the tuple is assumed to have two arguments: a string and an integer.

With F#:

```fsharp
let recursive = false

let rec ReceiveTuple(str: string, i: int): string*int =
    let counter = i + 1
    Console.WriteLine(str)
    let newTuple = (str, counter)
    if recursive then
        ReceiveTuple newTuple |> ignore
    newTuple
```

In this case, the F# type that would let you reference this function would be `string*int->string*int`; so this is a new symbol that we're learning now: unlike with other programming languages in which the asterisk character involves pointers, in this case it is just a separator of types in a tuple.

But have you noticed how tuples blend into what seemed to be normal parameters in F#? In fact, along all this guide up until now, all the methods we have written in F# that received more than one parameter, were actually using tuples, even if you might have not noticed. But then, you might think, can you write the above method without using a tuple as a parameter in F# then? Yes you can, just omitting the comma, this way:

```fsharp
let recursive = false

let rec ReceiveNonTuple (str: string) (i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    let newTuple = (str, counter)
    if recursive then
        ReceiveNonTuple str counter |> ignore
    newTuple
```

What's the difference between the functions `ReceiveTuple` and `ReceiveNonTuple`? Both receive the same number of arguments, and with the same types. However, the first one has its parameters as an F# tuple, and the second one has parameters declared in an idiomatic-F# way (in "currified form"). Why is this more idiomatic in F#? Because `ReceiveTuple` cannot be used in partial application scenarios, while `ReceiveNonTuple` can be because it's using a currified style. In this case, the type of the function, instead of being `string*int->string*int`, it is `(string->int)->(string*int)`.

So why `(string->int)->(string*int)` is better than `string*int->string*int`? The latter allows for interoperability with C# (as it's the way that parameters are passed at the CIL level), but the former allows partial application in a very straightforward and non-convoluted way (partial application is also possible with TypeScript and C#, but in a more complex way which makes it not worth it). So without further ado, let's look at a very simple example of partial application: let's suppose we want to create a Multiplication function that receives two integers and returns one integer:

```fsharp
let Multiply (x: int) (y: int): int =
    x * y
```

Now, if we wanted to write a function to double the value of a number without having to repeat any implementation detail from the Multiply function, we could simply write it this way:

```fsharp
let Double (x: int): int =
    Multiply 2
```

What happens when we only pass one argument to the function "Multiply"? Let's look at its original signature: `(int->int)->int`. Currification laws tell us that using parentheses in type expressions is actually not needed (or that placing them elsewhere results in an equivalent expression), which means that we can write it this way as well (`int->int->int`) or this way (`int->(int->int)`). Therefore, passing only one parameter to a function that originally received two parameters, actually results in returning another function. We can probably understand it better this way:

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

This is a too simple example to maybe make you convinced of how powerful and useful partial application is. But it's the foundations of, for example, Dependency Injection in functional programming. You will probably only grasp the flexibility it allows, with time, but at least we can already show you its simplicity in F#.


### Example 10: string interpolation

In TypeScript you might write things like:

```typescript
const aStringToShowToTheUser = `Hello ${name}, I see you are ${age} years old`;
```

This has two problems: in large codebases we might replace the interpolated variable with one that doesn't have implicit conversion to string, or its converted value is unexpected.

In F# instead:

```fsharp
let aStringToShowToTheUser = sprintf "Hello %s, I see you are %i years old" name age
```

Why is this slightly better? Because:
* You need to be explicit about the type of each variable, specifying it inside the string via the letter after the `%` character
* This means that by being explicit we can assume that the developer is aware of the string conversion (in case the type is not string).
* If type's letter doesn't match the type of the element supplied for the same position, then you get a compiler error (instead of a possibly useless string representation of the element at runtime).


### Example 11: asynchronous code

A simple TypeScript snippet with asynchronous code:

```typescript
class Toast {
    public IsYummy(): boolean {
        return true;
    }
}

async function ToastBreadAsync(): Promise<Toast> {
    return new Toast();
}

function ApplyButter(toast: Toast) { /* TODO */ }

function ApplyJam(toast: Toast) { /* TODO */ }

async function MakeToastWithButterAndJamAsync(): Promise<Toast> {
    var toast = await ToastBreadAsync();
    ApplyButter(toast);
    ApplyJam(toast);
    return toast;
}

async function Main() {
    console.log("Hello World!");
    var toast = await MakeToastWithButterAndJamAsync();
    console.log("Bye World!" + toast.IsYummy());
}
```

Becomes in F#:

```fsharp
type Toast() =
    member this.IsYummy() =
        true

let ToastBread(): Async<Toast> =
    async {
        return Toast()
    }

let ApplyButter(toast) =
    () //TODO

let ApplyJam(toast) =
    () //TODO

let MakeToastWithButterAndJam() =
    async {
        let! toast = ToastBread()
        ApplyButter(toast)
        ApplyJam(toast)
        return toast
    }


[<EntryPoint>]
let main(argv) =
    Console.WriteLine("Hello World!")
    let toast = MakeToastWithButterAndJam()
                |> Async.RunSynchronously
    Console.WriteLine ("Bye world!" + (toast.IsYummy().ToString()))
    0 // return an integer exit code
```

The key differences:
* In TypeScript, when you call an asynchronous method, you're given a `Promise<T>` object which represents the job being worked on, and it has already been started. In F#, though, the job is represented by an `Async<'T>` object which hasn't been started yet (you can later decide how to start it; e.g. in this example it's just started with the `Async.RunSynchronously` call).
* The equivalent of `await` in TypeScript, is simply the addition of the `!` character to the let statement in F#.
* In TypeScript, you can convert computation-heavy synchronous methods into asynchronous by using `async` keyword in function definition, in F# you simply wrap them with an `async{}` block (a computation expression).

Moreover, there's a subtle difference: in TypeScript, a promise can either be fulfilled or rejected. However, in F#, an `Async<'T>` object will always return a value of type `'T` once it is run. To achieve something similar in F#, you can return a value of type [`Result<'T,'TFailure>`](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/results) or use [`Async.Catch`](https://learn.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asynccatch) to catch exceptions that are thrown during its execution.

If we change the TypeScript code above slightly to introduce parallelization (supposing we have two toasters):

```typescript
class Ingredients {}

class Toast {
    public constructor(i: Ingredients)
    {
        this.ingredients = i;
    }
    ingredients: Ingredients
}

async function ToastBreadAsync(i: Ingredients): Promise<Toast> {
    return new Toast(i);
}

async function GatherIngredients() {
    return new Ingredients();
}

async function Make2ToastsAsync(i: Ingredients): Promise<Toast[]> {
    var toast1 = ToastBreadAsync(i);
    var toast2 = ToastBreadAsync(i);
    return await Promise.all([toast1, toast2]);
}

async function MakeToastsAsync(): Promise<void> {
    var i = await GatherIngredients();
    await Make2ToastsAsync(i);
}

async function Main(): Promise<void> {
    console.log("Hello World!");
    await MakeToastsAsync();
    console.log("Bye World!");
}
```

Then in F# it becomes:

```fsharp
type Ingredients () = class end
type Toast (i: Ingredients) = class end

let ToastBread(i): Async<Toast> =
    async {
        return Toast(i)
    }

let GatherIngredients () =
    async { return Ingredients() }

let Make2Toasts(i) =
    async {
        let twoJobs: List<Async<Toast>> = [ToastBread(i); ToastBread(i)]
        let! _ = Async.Parallel(twoJobs)
        return ()
    }

let MakeToasts() =
    async {
        let! i = GatherIngredients()
        do! Make2Toasts(i)
    }


[<EntryPoint>]
let main(argv) =
    Console.WriteLine("Hello World!")
    MakeToasts()
        |> Async.RunSynchronously
    Console.WriteLine("Bye world!")
    0 // return an integer exit code
```

As you can see, then:
* The equivalent of `Promise<void>` in F# is `Async<unit>`. To await this kind of jobs, instead of using `let! x = ...` you would just need `do! ...`.
* The equivalent for `Promise.all` is `Async.Parallel`.


### Example 12: write less characters! especially good for readability of F# scripts

Now that you understood the difference between tuples and currified parameters in F#, and how the latter is always preferrable, you may understand that writing so many parentheses was actually only needed to map things in tuples and is, in fact, a powerful inertia from TypeScript devs that are starting to work with F#.

But as you start learning F# more and more, leaving the TS/JS days behind, and writing always parameters in currified form, and writing less types (so that they can be inferred by the compiler), you realize how many less characters you need to type:

* Not so many parentheses because you don't use tuples anymore.
* No need to use semicolons if you just use EOL separators.
* No need to use braces so much as you need them in TS/JS (you only need them in F# when you deal with records).
* No need to use colon character `:` so many times if you let the F# compiler infer types more.
* Using the pipe operator `|>` more to avoid writing many parentheses on the right side of a very long line.
* No need for parentheses in `if` expressions in F# (as opposed to TS/JS, which always needs them).

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

```fsharp
type Ingredients () = class end
type Toast (i: Ingredients) = class end

let ToastBread i: Async<Toast> =
    async {
        return Toast i
    }

let GatherIngredients () =
    async { return Ingredients() }

let Make2Toasts i =
    async {
        let twoJobs: List<Async<Toast>> = [ToastBread i; ToastBread i]
        let! _ = Async.Parallel twoJobs
        return ()
    }

let MakeToasts() =
    async {
        let! i = GatherIngredients()
        do! Make2Toasts i
    }


[<EntryPoint>]
let main argv =
    Console.WriteLine "Hello World!"
    MakeToasts()
        |> Async.RunSynchronously
    Console.WriteLine "Bye world!"
    0 // return an integer exit code
```

------------------------------------------------------

CONGRATS!! You already know enough to maybe understand 80% of F# code.

Or maybe 80% of simple F# code, which is the code that is being used, for instance, in most F# scripts: easy code.

I could explain you how to build the equivalent of classes (with behaviour, constructors, properties) or structs (value types and stack allocated) in F#, but... 1) it's not idiomatic F#; 2) if you're looking for an alternative safer scripting language, most scripts don't even need types, they just need functions, values and calls!

If you're still not convinced about making the switch completely to F#, well, you could dip your toes first by using [Fable](https://fable.io), a transpiler that would allow you to compile your F# code to TypeScript/JavaScript, so that you can still continue using the ecosystem you're proficient with, but with a new tool under your belt.

So, I recommend as an interesting next step to watch these talks:
* [From F# to JavaScript with Fable](https://www.youtube.com/watch?v=5191ytFmG_A)
* [I Tried Replacing JavaScript With F# — Here’s What Happened!](https://www.youtube.com/watch?v=Z1cMBH4mWr4)
