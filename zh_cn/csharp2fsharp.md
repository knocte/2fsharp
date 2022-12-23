## C#开发者F#速学指南

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

### 示例 4: 基本块
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
* ` throw X;`子句变成`raise X `，空的`throw; `变成函数调用`reraise() `。
* 但是，没有`try-with-finally `块!我们只有`try-with`块和`try-finally`块。因此F#中的等效代码需要嵌套(就像上面的例子一样)。

你可能认为这是F#的一个缺点，但`try-catch-finally`块是极其罕见的，特别是考虑到C#中的`using` 结构(用于`IDisposable`):

```csharp
using (var reader = new StreamReader(someFile))
{
    DoStuff(reader);
}
```
就变成了
```fsharp
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* 使用 `use`时不需要嵌套子块
* 因此，资源将在超出作用域(函数结束)时被释放。

注：最新C#版本也不再需要嵌套字块

### 示例 5: 避免空值以及忽略返回值
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
    | Some(someValue) -> // 就像C#中的 'as',转换并得到想要的值
        let str = someValue.ToString()
        ignore(stringBuilder.Append(str))
    | None -> ()

    match someParam2 with
    | Some(_) -> // 就像C#中的 'is',你不关心它的值
        stringBuilder.Append(String.Empty) |> ignore
    | _ -> ()

```
在C#中，你到处都写空检查(在编译时不安全)。在F#中，您使用`Option<T>`类型(类似于`Nullable<T>` 但更好)和匹配表达式(模式匹配)以更安全的方式进行空检查。

* 当你不想返回任何东西时，在C#中你使用`void` ，这是用于指定类型的缺失的元数据，但在F#中你需要返回一个名为`unit`的特殊类型，它只有一个可能的值:`()`。这就是为什么`()`通常意味着什么都不做(如上面的代码所示)。
* `match-with` 块几乎类似于一个`switch`块，但更简洁，因为它包括强制转换(到someValue)。
* 忽略返回值有三种方式:
  * 例如，我们不关心Append()的返回值，在C#中我们直接忽略它，但在F#中你需要显式地忽略它，使用`ignore()` 语法糖。
  * match表达式中的下划线:它就像C#`switch`中的 `default`。
  * `Some(_)`中的下划线，当我们想要确保值不是NULL，但我们又不关心它的内容(就像C#中的`is` 操作符，而不是 `as`)。
* 管道操作符是`|>`(它的工作方式与bash中的`|`类似)。`ignore(x)`与 `x |> ignore`相似。
```F#
//match-with结构
match variable  with
| Some(Value1) ->
	DoSomething
| Some(Value2) ->
	DoSomething
| None -> ()
```

### 示例 6: 基本类型

下面这个不可变的C#类用F#编写更容易:

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

因为它只有一行:
```fsharp
type Foo = { Bar: int; Baz: string }

module FooFactory =
    let internal CreateFoo () =
        { Bar = 42; Baz = "forty-two" }
```

那么：
* 没有行为的类(如上面的Foo)被称为“Records”，它们看起来类似于struct，但它们仍然是引用类型，并且分配在堆上。它们是不可变的(一旦创建了它们，就不能在下面更改它们的值)。
* 静态类是“模块”，就像上面的“FooFactory”类型。
* 在F#中，创建新类或结构的实例时不需要使用关键字“new”，除非创建的类实现了IDisposable。


### 示例 7: 顺序很重要，循环依赖是万恶之源

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

为什么不呢？你可能会认为没什么问题。确定。

下面这段代码也可以通过编译:

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

也许你已经明白我的出发点了。最后一个C#代码片段编译得很好，因为C#允许循环依赖。然而，最后这句话只对了一半，因为循环依赖对C#语言有效，但对.Net程序集无效(你不能从B引用程序集A，如果A已经依赖于B)。模块化的原则将禁止你写这样的代码，因为在未来你不能把它分成两个程序集(你不能把Foo1放在一个程序集，而把Foo2放在另一个程序集)。因为循环依赖在. Net中无效)。

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
为什么?将抛出错误:
```

* 错误FS0039: 没有定义值、命名空间、类型或模块'Foo2'(参考Foo1.Baz实现)。

除非我们停止使用循环依赖，它才可能被修复(尽管已经有了escape hatch通过`and`关键字或 `rec` 关键字应用于模块或名称空间;我不会在本指南中解释它的用法，因为它太高级了);因为F#编译器避免在同一个程序集中循环依赖的方法要求我们所依赖的所有东西都要早点声明。这意味着如果某个函数A调用函数B，那么B需要在A之前声明(如果它们在同一个文件中，那么B需要在顶部，A在底部;如果它们在不同的文件中，则`.fsproj`文件中`B.fs`需要列在`A.fs`之前)。

```fsharp
module Foo =
    let Bar() =
        if Baz() then
            Bar()

    let Baz(): bool =
        false
```

它抛出错误:

* 错误FS0039: 没有定义值或构造函数 'Baz' (参考Foo.Bar实现)。

但正如我们刚刚学到的，这个更容易解决;先定义`Baz`。对于一个能够调用自己的函数(在某种程度上，它也是一个循环依赖)，我们使用`rec`关键字(它的意思是“递归”):
```fsharp
module Foo =
    let Baz(): bool =
        false

    let rec Bar() =
        if Baz() then
            Bar()
```


### 示例 8: 委托、匿名方法、函数、我太难了！

在C#的早期版本中，传递函数(和过程，即返回`void`的函数)的方式是通过委托类型和匿名函数。请看一个有4种组合的例子:

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

一切都很好，后来新版本C#的出现使得它不那么冗长、更优雅(感谢局部函数，你会注意到它的):

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

最有趣的变化是使用`delegate` 关键字的委托类型变成了 `System.Action`, `System.Action<TArg1>`, `System.Function<TResult>`, `System.Function<TArg1,TResult>` 等等. 通过 `delegate` 关键字声明的匿名方法通过 `=>` 符号变成了Lambda表达式。

但好消息是F#中的函数是本地公民，所以这一切在这种语言中看起来更简单(局部函数也可以工作):

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

如你所见，相当于BCL的 `System.Function` 和`System.Action` 变成原生F#语法，用于表示参数并通过 `->` 返回值，例如`TArg1->TResult`。 记住， `void`在F#中是 `unit`, 只有一个可能值 `()`的虚拟实类型，这使得区分函数和操作变得毫无意义。 最后，C#中的`(...) => { ... }` 变成`fun ... -> ...`。


### 示例 9: 元组、部分应用与柯里化

如果你从来没有做过函数式编程，你很有可能会害怕其中的一些概念，比如“部分应用程序”和“柯里化”，但事实是，它们并不是那么复杂的概念，为了正确地解释它们，我们需要首先解释元组，以及为什么不建议在F#中滥用它们(事实上，你不能对元组使用部分应用程序!稍后再详细介绍)。

让我们首先讨论一个带有 `out`参数的C#代码片段:

```csharp
int anInteger;
if (int.TryParse(someString, out anInteger)) {
    DoSomethingWithAnInteger(anInteger);
} else {
    DoSomethingElse();
}
```

如果你试着根据你所学到的知识把上面的内容转换成F#，你首先会想：我怎么能声明一个变量而不给它赋值呢？说实话，在F#中真的没有办法做到这一点。但是在知道了这一点之后，你可能会倾向于简单地将任何虚拟值赋给`anInteger` 变量，毕竟最终它将被 `TryParse()` 调用覆盖，对吗？这不是很优雅，特别是因为要这样做的话，我们需要将变量标记为`mutable`(以便以后能够用第二个值重写它)，但这不是F#的惯用法。在F#中最好的方法是使用所谓的“Active模式”，它会将上面的函数调用转换为一个同时返回两个值的函数调用:

```fsharp
match int.TryParse(someString) with
| (true, anInteger) -> DoSomethingWithAnInteger(anInteger)
| (false, _) -> DoSomethingElse()
```

上面的Active模式为F#编译器提供了语法糖，它将`bool TryParse(string,out int)` 签名转换为`(bool,int) TryParse(string)`，其中`(bool,int)` 是一个元组！因此，在F#中，任何元素长度的元组都可以很容易地以这种方式在任何其他场景中创建，而不需要使用 `System.Tuple<X,Y,...>` 类型。在某种程度上，这让我想起了在C#的最后一个版本中改进的方式(在那里你也可以在函数签名中使用元组而不使用`Tuple` )。

让我们再次检查一下我们的意图。这是旧版本的C#写法:

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

如果我们想要有一个指向这个函数的变量，需要`Func<Tuple<string,int>,bool>`类型。

使用新版本C# (在底层它被编译为 `ValueTuple<X,Y,...>` 元素):

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

现在需要一个指向这个函数的变量，需要 `Func<ValueTuple<string,int>,bool>`类型。

使用F#:

```fsharp
let rec ReceiveTuple(str: string, i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    let newTuple = (str, counter)
    ReceiveTuple (newTuple)
    true
```

在这个示例中，允许你引用这个函数的F#类型是`string*int->bool`;这是我们现在学习的一个新符号：不像其他编程语言中星号字符代表指针，在当前示例中，它只是元组中类型的分隔符。

但是你有没有注意到元组是如何像正常的参数一样融入F#中的？事实上，到目前为止，我们在F#中编写的所有接收多个形参的方法实际上都在使用元组，即使你可能没有注意到。所以，你可能会想，是否能在F#中在不使用元组的情况下编写上述方法吗?是的，你可以，只是省略逗号，像这样:

```fsharp
let rec ReceiveNonTuple (str: string) (i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    ReceiveNonTuple str counter
    true
```

函数`ReceiveTuple` 和`ReceiveNonTuple`之间有什么区别？两者都接收相同数量的参数，并且具有相同的类型。但是，第一个函数的参数是F#元组，第二个函数的参数是以惯用的F#方式声明(“柯里化形式”)。为什么这在F#中更常用？因为`ReceiveTuple`不能在部分应用场景中使用，而`ReceiveNonTuple`可以，因为它使用了柯里化风格。在这种情况下，函数的类型不是`string*int->bool`，而是`(string->int)->bool`。

那么为什么`(string->int)->bool` 比' `string*int->bool`更好呢？后者允许与C#的互操作性(因为它是在CIL级别传递参数的方式)，但前者允许以非常直接和不复杂的方式进行部分应用程序(稍后我们将看到也可以使用C#实现部分应用程序，但是以如此复杂的方式，其好处并不超过其糟糕的可读性/可维护性的缺点)。废话不多说，让我们看一个非常简单的部分应用程序的例子：假设我们想要创建一个乘法函数，它接收两个整数并返回一个整数:

```fsharp
let Multiply (x: int) (y: int): int =
    x * y
```

现在，如果我们想写一个函数来使一个数字的值翻倍，而不需要重复Multiply函数的任何实现细节，我们可以简单地这样写:

```fsharp
let Double (x: int): int =
    Multiply 2
```

当我们只向函数“Multiply”传递一个参数时会发生什么？让我们看看它的原始签名:`(int->int)->int`。柯里化定律告诉我们，在类型表达式中使用括号实际上是不需要的(或者将它们放在其他地方会得到一个等效的表达式)，这意味着我们也可以这样写(`int->int->int`)或这样写 (`int->(int->int)`)。因此，只将一个形参传递给最初接收到两个形参的函数，实际上会返回另一个函数。我们可以这样更好地理解:

```fsharp
let Double (x: int): int =
    let doubleFunc = Multiply 2
    let result = doubleFunc x
    result
```

或者甚至是这样(使用冗余指定的类型):

```fsharp
let Double (x: int): int =
    let doubleFunc: int->int = Multiply 2
    let result: int = doubleFunc x
    result
```

这个例子太简单了，可能无法让您相信部分应用程序是多么强大和有用。但它是，例如函数式编程中的依赖注入的基础。随着时间的推移，你可能只会掌握它所允许的灵活性，但至少我们已经可以在F#中向你展示它简单的一面，至少与C#相比，因为下面你使用C#实现部分应用程序的方式：

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

你能理解吗？对我来说，这比阅读F#代码要难一些。

关于部分应用程序及其实用性的最后一个注意事项：你使用F#的时间越长，你就越会意识到部分应用程序实际上是进行依赖注入/控制反转(DI / IoC)的一种简化方式，关于这一点的更多信息请参阅文章“[部分应用是依赖注入](https://github.com/amerina/NetCoreGrowthGuide/blob/main/TranslationArticle/部分应用是依赖注入.md)”。


### 示例 10: 字符串插值

在早期C#版本中你可能这样写：

```csharp
var aStringToShowToTheUser = String.Format("Hello {0}, I see you are {1} years old", name, age);
```

这有两个问题：在大型代码库中，可能在字符串中包含许多元素，有许多变量，很容易发生的情况是，我们包含了一个没有提供的变量的数字(例如'{2} ')，或者我们提供了一个没有隐式转换为字符串的元素(或者其转换是标准且无用的`System.Object`的`ToString()` 方法，它仅仅输出对象类型)。

在F#中，我们有一个更好的替代方案:

```fsharp
let aStringToShowToTheUser = sprintf "Hello %s, I see you are %i years old" name age
```

为什么这样更好呢？因为:
* 如果您提供的参数比在字符串中插入所需的参数少(或多)，则将得到编译器错误而不是运行时异常(失败更快!)
* 您需要通过`%`后面的字母指定字符串中元素的类型，如果它与提供给相同位置的元素类型不匹配，则会得到一个编译器错误(而不是无用的元素字符串表示)。
* 你需要更少的括号(有关这方面的更多信息，请参阅本指南中关于写更少字符的最后一个示例)。


### 示例 11: 异步代码

一个简单的C#异步代码片段：

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

在F#中变成:

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

关键区别：
* 在C#中，当你调用一个异步方法时，你会得到一个`Task<T>`对象，它表示正在执行的作业，并且它已经启动了。但是，在F#中，作业是由尚未启动的`Async<T>`对象表示的(您可以稍后决定如何启动它;例如，在这个例子中，它通过`Async.RunSynchronously`调用)。
* 与C#中`await`等价的是添加` !`字符到F#中的let语句中。
* 在C#中，你可以将计算量大的同步方法转换为异步方法，方法是将它们包装在一个`Task.Run() `调用中，而在F#中，你只需用一个`async{} `块(一个计算表达式)来包装它们。

如果我们稍微改变一下上面的C#代码，引入非泛型Task对象和并行化(假设我们有两个烤面包机):

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

在F#中变成:

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

如你所见，那么:
* 在F#中处理非泛型任务的等效方法是使用`unit`作为`Async`的泛型参数:`Async<unit>`。在等待这类工作时，不要用`let! x = ...` 你只需要`do! ...`。
* `Task.WhenAll` 在F#中是`Async.Parallel`.


### 示例 12: 少写字符！尤其有利于F#脚本的可读性

现在你已经理解了F#中元组和柯里化参数之间的区别，以及后者为什么总是更可取，你可能会明白写这么多圆括号实际上只是为了映射元组中的东西，事实上，这是C#开发人员开始使用F#的一个强大的惯性。

但是当你开始越来越多地学习F#，把C#抛在脑后，总是以柯里化的形式写参数，写更少的类型(这样编译器就可以推断它们)，你会意识到你需要输入的字符少了多少:

* 没有那么多圆括号，因为你不再使用元组了。
* 如果只使用EOL分隔符，则不需要使用分号。
* 不需要像C#中那样使用大括号(你只在F#中处理records时需要它们)。
* 如果你让F#编译器推断更多类型，就不需要多次使用冒号`:`。
* 更多地使用管道操作符 `|>` ，以避免在非常长的行右侧写入许多括号。
* F#中的`if` 表达式不需要括号(与C#相反，C#总是需要括号)。

考虑到所有这些，我们现在要重新编写本指南的所有F#示例，但没有所有这些多余的字符:

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

恭喜! !你已经掌握了足够的知识，可以理解80%的F#代码。

或者是80%的简单F#代码，也就是在大多数F#脚本中使用的代码：简单代码。

我可以向你解释如何在F#中构建等价的类(使用行为，构造函数，属性)或结构体(值类型和堆栈分配)，但是…

1)它不是惯用的F#;

2)如果你正在寻找另一种更安全的脚本语言，大多数脚本甚至不需要类型，他们只需要函数，值和调用!

不管怎样，我建议大家接下来看这个演讲： https://www.youtube.com/watch?v=HQ887aOZITY , 或者看看这篇文章：[C#有F#没有的5个特性](https://github.com/amerina/NetCoreGrowthGuide/blob/main/TranslationArticle/C%23有F%23没有的5个特性.md), 如果过了几天你还在挣扎着转换，也许可以不时地使用这个工具: https://github.com/willsam100/FShaper

