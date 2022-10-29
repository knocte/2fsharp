# C#开发者F#速学指南

本指南主要基于示例。它将花费你15-30分钟的时间，通过理解它，你将掌握这门语言中80%最常用的元素。


### 示例 1: 基本函数声明和实现

```csharp
public int GiveMeTheLength(string input)
{
    // this is a 1-line comment
    var result = input.Length;
    /* this is a multi-line comment */
    return result;
}
```
变成
```fsharp
let GiveMeTheLength(input) =
    // this is a 1-line comment
    let result = input.Length
    (* this is a multi-line comment *)
    result
```

* 默认情况下，所有东西都是公共的，除非你显式地指定`private`修饰符。
* 关键字`var`变成` let`(它也被用来定义函数/方法，而不仅仅是声明变量)。
* 你不需要` return`关键字。函数的最后一个元素是要返回的值。
* 不需要大括号，它像Python一样通过4个空格(或2)缩进工作。
* 不需要用分号来表示一行的结束。
* 指定类型总是可选的，除非在编译器无法推断它们的非常特殊的情况下。
* 注释与C#中的注释相同，不同的是多行注释使用括号而不是斜杠。

如果你想在上面的例子中指定类型，它会变成:

```fsharp
let GiveMeTheLength(input: string): int =
    let result: int = input.Length
    result
```


### 示例 2: 基本关键字和运算符

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
变成
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
*  `using` 关键字变成 `open`.
* `if (x) foo();  else if (y) bar();  else baz();`模式变成`if x then foo() elif y then bar() else baz()`，不需要在条件中插入括号，但增加了新的关键字`then`和`elif`。
* 初始赋值(给只读常量)操作符是`=`。如果需要将新值重新赋给相同的元素，则显式地将其标记为mutable并使用`<-`操作符。
* 由于上述原因，`=`操作符也可以作为比较操作符(不需要像C#那样将其加倍`==`)
* 操作符`!=` 变成`<>`。
* 操作符`!` 变成 `not`。
* 操作符`&&` 和`||` 在F#是一样的.
* 不需要在Main()函数中包含程序入口点， 只需在.fsx脚本或最后一个.fs文件中编写语句发送给F#编译器即可。

一般来说，像示例中这样简单的代码可以很容易地编码，而不需要可变变量，只需要执行只读赋值，如下所示:

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

(这类似于C#中的 `?` 操作符，但更简洁和易读。)


### 示例 3: 基本集合

```csharp
var intArray = new int[] { 1, 2, 3 };
var intList = new List<int>({ 4, 5, 6 });
IEnumerable<int> sequenceOfIntegers = intList;
var dictionary = new Dictionary<string,int>() {
    { "One", 1 },
    { "Two", 2 }
}
```
变成如下代码(实际上你不需要在F#中作类型声明，我们只是添加它们作为参考):
```fsharp
let intArray: array<int> = [| 1; 2; 3 |]
let intList: List<int> = [ 4 ; 5 ; 6 ]
let sequenceOfIntegers: seq<int> = intList
let dictionary: IDictionary<string,int> = dict [ ("One", 1); ("Two", 2) ]
```
* 在声明数组/列表/字典的元素时，逗号变成分号。
* `IEnumerable<T>` 变成 `seq<'T>` ("sequence"的缩写).
* 使用 `dict` 初始化 `IDictionary<K,V>` 集合, 然而在 F# 中你宁可使用 `Map<'K,'V>` 因为后者是不可变的。

(注意:泛型需要引号字符(')作为前缀，如上所述。)


### Example 4: 基本区块

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
变成
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
* `catch` 关键字变成 `with`.
* `throw X;` 语句变成 `raise X`, 一个空的 `throw;` 变成函数调用 `reraise()`。
* 然而并没有 `try-with-finally` 块！我们只有 `try-with` 块和 `try-finally` 块。因此，在F#中等价的代码需要嵌套(就像上面的例子中所做的那样)。

你可能认为这是F#的缺点，但是try-catch-finally块非常罕见，特别是考虑到C#中的`using`结构(用于`IDisposable`):

```csharp
using (var reader = new StreamReader(someFile))
{
    DoStuff(reader);
}
```
这就变成了
```fsharp
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* 使用`use`时不需要嵌套子块
* 因此，资源将在它超出作用域(函数结束)时被释放。

注：最新C#版本也不再需要嵌套字块

### Example 5: 避免使用Null和忽略某些事情

```csharp
void Check(SomeType someParam1, SomeType someParam2)
{
    if (someParam1 != null)
        stringBuilder.Append(someParam1.ToString());

    if (someParam2 != null)
        stringBuilder.Append(String.Empty);
}
```
变成
```fsharp
let Check(someParam1: Option<SomeType>, someParam2: Option<SomeType>): unit =

    match someParam1 with
    | Some(someValue) -> // 就像C#中的'as'一样，你需要强制转换并获取值
        let str = someValue.ToString()
        ignore(stringBuilder.Append(str))
    | None -> ()

    match someParam2 with
    | Some(_) -> // 就像C#中的'is'一样，你不需要关心它的值
        stringBuilder.Append(String.Empty) |> ignore
    | _ -> ()

```
在C#中你到处写Null检查(在编译时没有安全性)。在F#中，使用`Option<T>`类型(类似于`Nullable<T>`，但更好)和匹配表达式(模式匹配)以更安全的方式进行Null检查。

* 当你不想返回任何东西时，在C#中你使用`void`，它是指定类型缺失的元数据，但在F#中你需要返回一个名为`unit`的特殊类型，它只有一个可能的值:`()`。这就是为什么`()`通常意味着什么都不做(如上面的代码所示)。
* `match-with`语句块类似于switch语句块，但更简洁，因为它包含了转换(到someValue)。
* 有三种忽略事物的方式:
  * 例如，我们不关心Append()的返回值，在C#中我们直接忽略它，但在F#中你需要使用`ignore()`魔法函数明确地忽略它。
  * 匹配表达式中的下划线:它就像C# `switch`中的`default`。
  * `Some(_)`中的下划线，当我们想确保值不是None，但我们不关心它的内容(就像C#中的`is`操作符，而不是`as`)。
* 管道操作符(bash中的`|`)是`|>`(它的工作方式与bash类似)。那么`ignore(x)`等同于`x |> ignore`。

```F#
//match-with结构
match variable  with
| Some(Value1) ->
	DoSomething
| Some(Value2) ->
	DoSomething
| None -> ()
```

### Example 6: 基本类型

下面这个不可变的C#类用F#更容易写:

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

因为只有一行代码:

```fsharp
type Foo = { Bar: int; Baz: string }

module FooFactory =
    let internal CreateFoo () =
        { Bar = 42; Baz = "forty-two" }
```

那么：
* 没有行为的类(如上面的Foo)被称为“记录”，它们看起来类似于结构体，但它们仍然是引用类型，并在堆上分配。它们是不可变的(一旦你创建了它们，你就不能改变它们下面的值)。
* 静态类是“模块”，就像上面的“FooFactory”类型。
* 在F#中，创建新类或结构体的实例时不需要使用关键字“new”，除非正在创建的类实现了IDisposable。


### Example 7: 秩序很重要，循环依赖是万恶之源

作为一个C#开发人员，你知道这段代码可以通过编译:

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

为什么不呢?你可能这样认为。确定？

这也可以编译:

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

也许你已经明白我的立场了。最后一个C#代码片段可以通过编译，因为C#允许循环依赖。然而，最后这句话只是半真半假，因为循环依赖对C#语言是有效的，但对. NET程序集来说是无效的(如果A已经依赖B，则不能从B引用程序集A)。模块化原则将禁止您编写这样的代码，因为在未来您将无法将它分成两个程序集(您不能将Foo1放在一个程序集中，而Foo2放在不同的程序集中。因为循环依赖在.NET中是无效的)。

然后，你需要从这里学到的是，F#是一种避免你以这种方式搬起石头砸自己的脚的语言，因为在同一个程序集中的循环引用也是**无效**的。它是如何实现的呢?通过强制你在类型B之前声明类型A，以防后者调用前者。因此，在F#中等价的代码片段将无法编译:

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

为什么?错误将是:

* 错误FS0039: 值, 命名空间, 类型或模块'Foo2'没有定义。(指Foo1.Baz的实现。)

除非我们停止使用循环依赖，否则它是不可修复的(尽管存在通过`and`关键字或`rec`关键字应用于模块或命名空间通过Hack逃脱;我不会在本指南中解释它，因为它太高级了);因为F#编译器避免在同一个程序集中循环依赖的方式是我们依赖的所有东西都要提前声明。这意味着，如果某个函数A调用了函数B，那么B需要在A之前声明(如果它们在同一个文件中，那么B需要在顶部，A在底部;如果它们在不同的文件中，那么`B.fs`需要在`A.fs`之前列出。在`.fsproj`文件`. Fs `中)。

因此，这一小段代码相当于本节中的第一个C#示例，但也不能编译:

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

A final note about partial application and its usefulness: the longer you use F# the more you will realize that partial application is actually a simplified way of doing Dependency Injection / Inversion of Control (DI / IoC), more info about this in the article "Partial Application is Dependency Injection" (https://blog.ploeh.dk/2017/01/30/partial-application-is-dependency-injection/).


### Example 10: string interpolation

In the early days of C# you might write things like:

```csharp
var aStringToShowToTheUser = String.Format("Hello {0}, I see you are {1} years old", name, age);
```

This has two problems: in large codebases where there are many variables and maybe many elements to include in a string, it could easily happen that we include a number for a variable not supplied (e.g. `{2}`) or that we provide an element which didn't have implicit conversion to string (or whose conversion was the standard and useless `ToString()` base method of `System.Object`, which I think simply prints the type of the object).

In F# we have an alternative that is much better:

```fsharp
let aStringToShowToTheUser = sprintf "Hello %s, I see you are %i years old" name age
```

Why is this better? Because:
* If you supply less arguments (or more) than the ones needed to interpolate in the string, you will get a compiler error instead of an exception at runtime (fail faster!).
* You need to specify the type of the element inside the string, via the letter after the `%`, and if it doesn't match the type of the element supplied for the same position, then you get a compiler error (instead of a useless string representation of the element).
* You need less parenthesis (for more info about this, see the last example in this guide about writing less characters).


### Example 11: asynchronous code

A simple C# snippet with asynchronous code:

```csharp
    static class MainClass
    {
        public class Toast {
            public bool IsYummy()
            {
                return true;
            }
        }

        static async Task<Toast> ToastBreadAsync()
        {
            var task = Task.Run(() =>
                new Toast()
            );
            return await task;
        }

        static void ApplyButter(Toast toast) { /* TODO */ }

        static void ApplyJam(Toast toast) { /* TODO */ }

        static async Task<Toast> MakeToastWithButterAndJamAsync()
        {
            var toast = await ToastBreadAsync();
            ApplyButter(toast);
            ApplyJam(toast);
            return toast;
        }

        public static async Task Main(string[] args)
        {
            Console.WriteLine("Hello World!");
            var toast = await MakeToastWithButterAndJamAsync();
            Console.WriteLine("Bye World!" + toast.IsYummy());
        }
    }
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
* In C#, when you call an asynchronous method, you're given a `Task<T>` object which represents the job being worked on, and it has already been started. In F#, though, the job is represented by an `Async<'T>` object which hasn't been started yet (you can later decide how to start it; e.g. in this example it's just started with the `Async.RunSynchronously` call).
* The equivalent of `await` in C#, is simply the addition of the `!` character to the let statement in F#.
* In C#, you can convert computation-heavy synchronous methods into asynchronous by wrapping them in a `Task.Run()` call, in F# you simply wrap them with an `async{}` block (a computation expression).

If we change the C# code above slightly to introduce non-generic Task objects and parallelization (supposing we have two toasters):

```csharp
public class Ingredients
{
}

public class Toast {
    public Toast(Ingredients i)
    {
    }
}

static async Task<Toast> ToastBreadAsync(Ingredients i)
{
    var task = Task.Run(() =>
        new Toast(i)
    );
    return await task;
}

static async Task<Toast[]> Make2ToastsAsync(Ingredients i)
{
    var toast1 = ToastBreadAsync(i);
    var toast2 = ToastBreadAsync(i);
    return await Task.WhenAll(toast1, toast2);
}

static async Task MakeToastsAsync()
{
    var i = await GatherIngredients();
    await Make2ToastsAsync(i);
}

public static async Task<Ingredients> GatherIngredients()
{
    return await Task.FromResult(new Ingredients());
}

public static async Task Main(string[] args)
{
    Console.WriteLine("Hello World!");
    await MakeToastsAsync();
    Console.WriteLine("Bye World!");
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
* The equivalent of dealing with non-generic Tasks in C#, in F# would mean using `unit` as the generic argument to `Async`: `Async<unit>`. To await this kind of jobs, instead of using `let! x = ...` you would just need `do! ...`.
* The equivalent for `Task.WhenAll` is `Async.Parallel`.


### Example 12: write less characters! especially good for readability of F# scripts

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

Anyway, I recommend as an important next step to watch this talk: https://www.youtube.com/watch?v=HQ887aOZITY , and/or have a look at this article: https://www.compositional-it.com/news-blog/5-features-that-c-has-that-f-doesnt-have/, and if after some days you're still struggling to make the switch, maybe use this tool from time to time: https://github.com/willsam100/FShaper

