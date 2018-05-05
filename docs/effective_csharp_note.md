# Effective C# ノート

書籍[Effective C#](http://amzn.asia/fYaaYnj)のノート

## 1. Prefer Implicitly Typed Local Variables

暗黙的に型指定されるローカル変数を好む。(varを使用してローカル変数を宣言する。)

``` C#
var foo = new MyType();
```

理由の1つ目はコードの可読性が上がるため、2つ目は開発者が誤った型を利用するのを防ぐため。  
2つ目の例として、以下のコード。Queryが２回実行されるため、パフォーマンスに問題がある。  
(本来 q はIQueryable<T>になるべき)

``` C#
// 悪い例
public IEnumerable<string> FindCustomersStartingWith1(
string start)
{
    IEnumerable<string> q =
        from c in db.Customers
        select c.ContactName;
    var q2 = q.Where(s => s.StartsWith(start));
    return q2;
}
```

下記のように修正すれば、Queryは一回実行するだけになる。

``` C# hl_lines="5"  
// 良い例
public IEnumerable<string> FindCustomersStartingWith1(
string start)
{
  　var q =
        from c in db.Customers
        select c.ContactName;
    var q2 = q.Where(s => s.StartsWith(start));
    return q2;
}
```

varを使用しないほうが良いケースとして、数値型を利用するケースがある。
数値は型によって計算誤差がでるため、明示的に指定したほうが可読性が上がる。

``` C#
// 悪い例
var total = 100 * GetMagicNumber() / 6;
// 良い例
double total = 100 * GetMagicNumber() / 6;
```

## 2. Prefer readonly to const

C#では2種類の定数定義がある。コンパイル時定数の方が、コンパイル時に値が決定されるので、若干パフォーマンスが良い。  
しかしコンパイル時定数は変更すると、バイナリ互換性が変わるので、参照しているアセンブリはすべて再ビルドが必要になる。  

``` C#
// Compile-time constant:
public const int Millennium = 2000;
 // Runtime constant:
public static readonly int ThisYear = 2004;
```

## 3. Prefer the is or as Operators to Casts

キャストにはisもしくはas演算子を利用する。可読性がよいので。  
as演算子は、キャストに失敗した場合はnullを返す。  
is 演算子はブール値のみを返す。オブジェクトの型の確認のみ行い、キャストは行う必要がない場合に使用する。  

``` C#
// 良い例
object o = Factory.GetObject();
MyType t = o as MyType;
if (t != null)
{
    // work with t, it's a MyType.
}
else
{
    // report the failure.
}
```

``` C#
// 悪い例
object o = Factory.GetObject();
try
{
    MyType t;
    t = (MyType)o;
    // work with T, it's a MyType.
}
catch (InvalidCastException)
{
    // report the conversion failure.
}
```

値型はas演算子は使用できないが、下記にようにすれば使用できる。

``` C#
object o = Factory.GetValue();
var i = o as int?;
if (i != null)
    Console.WriteLine(i.Value);
```

## 4. Replace string.Format() with Interpolated Strings

Formatは順番を間違えやすいのに対して、挿入文字列を使用したほうが可読性が高い。

``` C#
Console.WriteLine($"The value of pi is {Math.PI.ToString("F2")}");
```

``` C#
using System;

public class Example
{
    public static void Main()
    {
        var name = "Horace";
        var age = 34;
        var s1 = $@"He asked, ""Is your name {name}?"", but didn't wait for a reply.
 {name} is {age:D3} year{(age == 1 ? "" : "s")} old.";
       Console.WriteLine(s1);
    }
}
// The example displays the following output:
//       He asked, "Is your name Horace?", but didn't wait for a reply.
//       Horace is 034 years old.
```

## 5. Prefer FormattableString for Culture-Specific Strings

挿入文字列を使用すると、FormattableStringを利用することができる。

``` C#
System.IFormattable second =
  $"It's the {DateTime.Now.Day} of the {DateTime.Now.Month} month";
Console.WriteLine(
  second.ToString(null, new System.Globalization.CultureInfo("en-us"))
  );
```

## 6. Avoid String-ly Typed APIs

文字列にした型情報の受け渡しを避ける。C#6で追加されたnameof 演算子を利用する。
変数や、クラス、メソッド、プロパティなどの名前(識別子)を文字列リテラルとして取得できる。

``` C# hl_lines="5"  
public static void ExceptionMessage(object thisCantBeNull)
{
    if (thisCantBeNull == null)
        throw new
            ArgumentNullException(nameof(thisCantBeNull),
            "We told you this cant be null");
}
```

## 7. Express Callbacks with Delegates

コールバックはデリゲートによって実装する。  
.NET Frameworkライブラリは、Predicate<T>、Action<>、およびFunc<>を使用して、多くの一般的なデリゲートフォームを定義する。  
Predicateは、条件をテストするブール関数。  
Funcはいくつかのパラメータをとり、単一の結果を生成する。  
Actionは任意の数のパラメータを取り、void戻り型を持つ。  

``` C#
// LINQで使用する例
List<int> numbers = Enumerable.Range(1, 200).ToList();
numbers.RemoveAll(n => n % 2 == 0);                // Predicate
numbers.ForEach(item => Console.WriteLine(item));  // Action
```

## 8. Use the Null Conditional Operator for Event Invocations

自分で定義したイベントを実行する際にはNull演算子(?.)を利用する。  
Updatedというイベントハンドラーがある場合は、Updatedイベントは下記のように実行する。

``` C#
// 良い例（C#5.0以下の場合）
public void RaiseUpdates()
{
    counter++;
    var handler = Updated;
    if (handler != null)
        handler(this, counter);
}
```

``` C#
// 悪い例
public void RaiseUpdates()
{
    counter++;
    if (Updated != null)
        // 下記ステップ実行時にUpdatedイベントハンドラが、外部からNullに設定されて、
        // NullReferenceExceptionが発生する可能性がある。
        // （直近でNullチェックをしているので、極まれなケース。。）
        Updated(this, counter);  
}
```

Null演算子(?.)を利用すれば、下記のように簡単に記述できる。  
下記の場合、Updatedイベントハンドラが、外部からNullに設定されていた場合は、Invoke以降の右辺は実行されず、NullReferenceExceptionが発生しない。

``` C#
// 良い例（C#6.0以上の場合）
public void RaiseUpdates()
{
    counter++;
    Updated?.Invoke(this, counter);  
}
```
