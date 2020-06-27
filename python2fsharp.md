# Guide for Python devs to learn F# real FAST

This guide is mostly samples based. It will take you 15-30minutes of your time and by understanding it you will already get a hang of 80% of the most used elements of the language.

### Example 1: Basic function declarations and implementation

```python
def give_me_the_length(input):
    result = len(input)
    return result
```
becomes
```fsharp
let GiveMeTheLength(input) =
    let result = input.Length
    result
```

* All things are public by default unless you explicitly specify `private` modifier.
* Keyword `def` becomes `let` (which is also used to declare variables, not only to define functions/methods).
* You don't need the `return` keyword. The last element of the function is the value to be returned.
* Indentations works via 4-space (or 2) like Python.
* Specifying types is always optional, except in very special cases when the compiler cannot infer them.

If you want to specify the types in the sample above, it would become:

```fsharp
let GiveMeTheLength(input: string): int =
    let result: int = input.Length
    result
```

### Example 2: Basic keywords and operators

```python
import sys

exitCode = 0
if (incomingChar == "\n"):
    exitCode = 1
elif not (incomingChar == ""):
    exitCode = 2
elif (incomingChar != "\t" and len(incomingChar) > 1):
    exitCode = 3
else:
    exitCode = 4
exit(exitCode)
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
* The `:` at the end of an `if` statement becomes `then`
* Initial assignment operator is `=`. If you need to re-assign a new value to the same element, then you explicitly mark it as mutable and use the `<-` operator.
* Thanks to the above, the `=` operator can be a comparison operator too (no need for doubling it like in python: `==`).
* Operator `!=` becomes `<>`.
* Operator `not` is also `not` in F#.
* Operators `and` and `or` become `&&` and `||` in F#.
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

```python
intArray = array("i",[1,2,3])
intList = [4,5,6]
dictionary = {
    "One": 1,
    "Two": 2
}
```
becomes the following (take in account you don't need the type annotations in F# either, we just add them for reference):
```fsharp
let intArray: array<int> = [| 1; 2; 3 |]
let intList: List<int> = [ 4 ; 5 ; 6 ]
let dictionary: IDictionary<string,int> = dict [ ("One", 1); ("Two", 2) ]
```
* Commas become semicolons when declaring elements of an array/list/dictionary.
* You use `dict` to initialize an `IDictionary<string,int>` collection, however in F#
you would rather use a `Map<string,int>` because the latter is immutable.

NOTE: if you were to create a collection type whose elements had a type that is not
defined yet (it could change at runtime), then you have to use the quote character
to denote those generic types. Example: instead of `Map<string,int>`, `Map<'K,'V>`,
meaning `'K` is the key's type and `'V` is the value's type.


### Example 4: Basic blocks

```python
try:
    try_something(someParam)
except SomeError as err:
    if some_condition(err):
        do_something_else(err)
        raise OtherError
    else:
        raise
finally:
    make_sure_to_cleanup(someParam)
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
* The `except` keyword becomes `with`.
* The `raise X` is same in F#, but an empty `raise` becomes the function call `reraise()`.
* However, there are no `try-with-finally` blocks! We have only `try-with` blocks and `try-finally` blocks. Therefore
the equivalent in F# would need nesting (like it's done in the example above).

You may think this is an F# downside but try-catch-finally blocks are extremely
rare, especially given the `with` construct (for `Disposable` objects) in Python:

```python
with Reader() as reader:
    do_stuff(reader)
```
which becomes
```fsharp
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* No need for nesting a sub-block when using `use`
* Therefore, the resource will be disposed when it goes out of scope (the function ends).


### Example 5: Avoiding nulls (None) and ignoring things

```python
def check(someParam1, someParam2):
    if someParam1 is not None:
        stringBuilder.Append(someParam1.to_string())

    if someParam2 is not None:
        stringBuilder.Append("")
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
In python you write None checks everywhere (no safety at compile time). In F#,
you do the null check in a safer way with an `Option<T>` type and a match
expression (pattern matching).

* When you don't want to return anything, in python you just don't use the `return` statement, but in F# you need to return a special type called `unit`, which only has one possible value: `()`. That's why generally `()` means doing nothing (as per the above code).
* A `match-with` block is almost like a switch block, but more succint because it includes the casting (to someValue).
* There are three ways of ignoring things:
  * For example, let's say the `Append()` function returned some value, in python we just ignore it by not assigning it to a variable, but in F# you need to be explicit about ignoring it, using the `ignore()` magic function.
  * The underscore in a match expression: it's like an `else` clause in a python `if`.
  * The underscore in `Some(_)`, when we want to make sure the value is not None, but we don't care about its contents.
* The pipe operator (like `|` in bash) is `|>` (and it works like in bash). Then `ignore(x)` is the same as `x |> ignore`.


### Example 6: Basic types

This immutable python class below is much easier to write in F#:

```python
def create_foo():
    return Foo(42, "forty-two")

class Foo(object):

    def __init__(self, bar, baz):
        self._bar = bar
        self._baz = baz

    @property
    def bar(self):
        return self._bar

    @property
    def baz(self):
        return self._baz

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
* You cannot leave functions outside types like in Python, you would need to place them inside "modules" (classes that don't generate instance, aka static classes), like the "FooFactory" type above.


### Example 7: Order is important, and circular dependencies are the root of all evil

As a Python developer, you know that this code compiles fine:

```python
def bar():
    if baz():
        return bar()

def baz():
    return False
```

Why wouldn't it? You may think. Sure.
And this also compiles:

```python
class Foo1(object):
    def __init__(self, x, y):
        self._x = x
        self._y = y

    def bar(self):
        print "bla"

    def baz(self):
        foo2 = Foo2(2,1)
        foo2.baz()
        
class Foo2(object):
    def __init__(self, x, y):
        self._x = x
        self._y = y

    def bar(self):
        foo1 = Foo1(1,2)
        foo1.bar()

    def baz(self):
        print "bla"
```

Maybe you understand already where I'm coming from. The last python snippet can be executed with no issues, because python allows circular dependencies. The principles of modularity would forbid you to write code like this, just because in the future you would not be able to separate this code easily into different libraries (you could not place Foo1 in one library and Foo2 in a different library, because you might receive "undefined" errors whose root cause is the use of circular dependencies).

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

It's not fixable unless we simply stop using circular dependencies (despite an existing escape hatch via the `and` keyword which I will not explain in this guide, because it's too advanced); because the way that the F# compiler has to avoid circular dependencies within the same assembly is requiring everything that we depend to, to be declared earlier. This means that if some function A calls function B, then B needs to be declared before A (if they are in the same file, then B needs to be at the top and A at the bottom; if they are in different files, then `B.fs` needs to be listed earlier than `A.fs` in the `.fsproj` file).

Therefore, this smaller snippet equivalent to our very first python sample in this section, doesn't compile either:

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

Functions are objects in python so passing them as parameters is easy, as can be seen in the following snippet (note there are 4 combinations depending on the signature of the function):

```python
def delegate_reception1(procedure_with_no_return_value_and_no_arguments):
    procedure_with_no_return_value_and_no_arguments()

def proc1():
    print("hello")

def sending_proc_as_delegate1():
    delegate_reception1(proc1)


def delegate_reception2(procedure_with_no_return_value_and_one_arg):
    bar = "baz"
    procedure_with_no_return_value_and_one_arg(bar)

def proc2(foo):
    print("hello 2 " + foo)

def sending_proc_as_delegate2():
    delegate_reception2(proc2)


def delegate_reception3(function_with_one_return_value_and_no_arguments):
    result = function_with_one_return_value_and_no_arguments()
    print("hello 3 " + result)

def func3():
    return 3

def sending_func_as_delegate3():
    delegate_reception3(func3)


def delegate_reception4(function_with_one_return_value_and_one_arg):
    bar = 4.0
    result = function_with_one_return_value_and_one_arg(bar)

def func4(foo):
    print("hello 4 " + foo)
    return 4

def sending_func_as_delegate4():
    delegate_reception4(func4)
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

Let's start with a python snippet with a function that receives a tuple and returns a tuple:

```python
recursive = False

def receive_and_return_tuple(someTuple):
    str, i = someTuple
    counter = i + 1
    print(str)
    newTuple = (str, counter)
    if recursive:
        receive_and_return_tuple(newTuple)
    return newTuple
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

So why `(string->int)->(string*int)` is better than `string*int->bool`? The latter allows for interoperability with C# (as it’s the way that parameters are passed at the CIL level), but the former allows partial application in a very straightforward and non-convoluted way (partial application is also possible with python and C#, but in a more complex way which makes it not worth it). So without further ado, let’s look at a very simple example of partial application: let’s suppose we want to create a Multiplication function that receives two integers and returns one integer:

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

This is a too simple example to maybe make you convinced of how powerful and useful partial application is. But it’s the foundations of, for example, Dependency Injection in functional programming. You will probably only grasp the flexibility it allows, with time, but at least we can already show you its simplicity in F#.


### Example 10: string interpolation

In the early days of python you might write things like:

```python
aStringToShowToTheUser = "Hello {0}, I see you are {1} years old".format(name, age)
```

This has two problems: in large codebases where there are many variables and maybe many elements to include in a string, it could easily happen that we include a number for a variable not supplied (e.g. `{2}`) or that we provide an element which didn't have implicit conversion to string.

In F# we have an alternative that is much better:

```fsharp
let aStringToShowToTheUser = sprintf("Hello %s, I see you are %i years old" name age)
```

Why is this better? Because:
* If you supply less arguments (or more) than the ones needed to interpolate in the string, you will get a compiler error instead of an exception at runtime (fail faster!).
* You need to specify the type of the element inside the string, via the letter after the `%`, and if it doesn't match the type of the element supplied for the same position, then you get a compiler error (instead of a useless string representation of the element).
* You need less parenthesis (for more info about this, see example 12).


### Example 11: asynchronous code

A simple python snippet with asynchronous code:

```python

class Toast(object):
    def is_yummy(self):
        return True
        
async def create_toast():
    return Toast()

async def toast_bread():
    await create_toast()

async def apply_butter(toast):
    // TODO

async def apply_jam(toast):
    // TODO

async def make_toast_with_butter_and_jam():
    toast = await toast_bread()
    apply_butter(toast)
    apply_jam(toast)
    return toast
    
if __name__ == "__main__":
    print("Hello World!")
    toast = asyncio.run(make_toast_with_butter_and_jam())
    print("Bye World!" + toast.is_yummy())
```

Becomes in F#:

```fsharp
type Toast () =
    member this.IsYummy() =
        true

let ToastBread (): Async<Toast> =
    async {
        return Toast()
    }

let ApplyButter toast =
    () //TODO

let ApplyJam toast =
    () //TODO

let MakeToastWithButterAndJam() =
    async {
        let! toast = ToastBread()
        ApplyButter toast
        ApplyJam toast
        return toast
    }


[<EntryPoint>]
let main argv =
    Console.WriteLine "Hello World!"
    let toast = MakeToastWithButterAndJam()
                |> Async.RunSynchronously
    Console.WriteLine ("Bye world!" + (toast.IsYummy().ToString()))
    0 // return an integer exit code
```

The key differences:
* In python, when you call an asynchronous method, you're given an object which represents the job being worked on, and it has already been started. In F#, though, the job is represented by an `Async<'T>`-typed object which hasn't been started yet (you can later decide how to start it; e.g. in this example it's just started with the `Async.RunSynchronously` call).
* The equivalent of `await` in python, is simply the addition of the `!` character to the let statement in F#.

If we change the python code above slightly to introduce parallelization (supposing we have two toasters):

```python

class Ingredients(object):
    def __init__(self):

class Toast(object):
    def __init__(self, ingredients):
        self._ingredients = ingredients

async def create_toast(ingredients):
    return Toast(ingredients)

async def toast_bread(ingredients):
    await create_toast(ingredients)
    
async def make_2_toasts(ingredients):
    toast1 = toast_bread(ingredients)
    toast2 = toast_bread(ingredients)
    return await asyncio.gather(toast1, toast2)
    
async def make_toasts():
    ingredients = await gather_ingredients()
    return await make_2_toasts(ingredients)

async def gather_ingredients():
    return Ingredients()

if __name__ == "__main__":
    print("Hello World!")
    asyncio.run(make_toasts())
    print("Bye World!")
```

Then in F# it becomes:

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

As you can see, then:
* When dealing with async jobs in python that don't return anything, in F# would mean using `unit` as the generic argument to `Async`: `Async<unit>`. To await this kind of jobs, instead of using `let! x = ...` you would just need `do! ...`.
* The equivalent for the `asyncio.gather` function is F#'s `Async.Parallel` function.


### Example 12: write less characters! especially good for readability of F# scripts

Now that you understood the difference between tuples and currified parameters in F#, and how the latter is always preferrable, you may understand that writing so many parenthesis was actually only needed to map things in tuples and is, in fact, a powerful inertia from python devs that are starting to work with F#.

But as you start learning F# more and more, leaving the python days behind, and writing always parameters in currified form, you realize how many less characters you need to type:

* Not so many parenthesis because you don't use tuples anymore.
* No need to use semicolons if you just use EOL separators.
* No need to use colon character `:` so many times if you let the F# compiler infer types more (similar to the lack of types in python).
* Using the pipe operator `|>` more to avoid writing many parenthesis on the right side of a very long line.
* No need for parenthesis in `if` expressions in F# (same as python).

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
    | Some someValue ->
        let str = someValue.ToString()
        stringBuilder.Append str |> ignore
    | None -> ()

    match someParam2 with
    | Some _ ->
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
