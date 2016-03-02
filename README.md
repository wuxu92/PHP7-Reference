# PHP 7

> 来自英文版：[https://github.com/tpunt/PHP7-Reference](https://github.com/tpunt/PHP7-Reference)

PHP 7 在 [2015年十二月3日]
(http://php.net/archive/2015.php#id2015-12-03-1) 发布了. 它带来了大量的新特性，改变和一些向后兼容损坏（backwards compatibility breakages），这些改变概述如下：


**[性能](#性能)**

**[特性](#特性)**
* [组合比较操作符](#组合比较操作符)
* [空指针合并操作符](#空引用合并操作符)
* [标量类型声明](#标量类型声明)
* [返回值类型声明](#返回值类型声明)
* [匿名类](#匿名类)
* [Unicode编码点转移语法](#unicode编码点转义语法)
* [闭包 `call()` 方法](#closurecall)
* [带过滤的 `unserialize()`](#带过滤的-unserialize)
* [`IntlChar` 类](#intlchar-类)
* [期望](#预期)
* [组和 `use` 声明](#group-use-declarations)
* [生成器返回表达式](#生成器返回表达式)
* [生成器委托](#生成器委托)
* [整形除使用 `intdiv()`](#使用-intdiv-做整数除法)
* [`session_start()` 参数](#session_start-选项)
* [`preg_replace_callback_array()` 函数](#preg_replace_callback_array-函数)
* [CSPRNG 函数](#csprng密码学安全伪随机数构建器函数)
* [支持 `define()` 定义数组常量](#支持-define-定义数组常量)
* [新增的反射类](#反射增强)

**[改动](#改变**
* [放松保留字的限制](#放松保留字的限制)
* [同一变量格式](#统一变量语法)
* [引擎中的异常](#引擎中的异常)
* [可抛出的接口](#可抛出的接口)
* [整形语义](#整数语义)
* [JSON 扩展替代 JSOND](#jsond替代了json扩展)
* [ZPP Failure on Overflow](#zpp-溢出时失败)
* [修复 `foreach()`的行为](#修正-foreach-的行为)
* [修改 `list()`的行为](#修改-list-的行为)
* [改变被0整除的语义](#改变除数为0的语义)
* [修复自定义Session Handler返回值](#修复自定义session-handler的返回值)
* [标记PHP 4风格的构造函数过期](#返回只用php-4风格的构造函数)
* [移除 date.timezone 警告](#移除datetimezone的警告)
* [移除可替换PHP标签](#去除可替代的php标签)
* [移除Switch语句中的多默认块](#移除switch语句中的多默认块)
* [移除重复名字参数的重定义](#移除参数列表中重名参数的重定义)
* [移除Dead Server APIs](#移除已经dead的服务器api)
* [移除数字字符串中的十六进制支持](#移除数字字符串的十六进制支持)
* [移除已标记过期的函数](#移除过期的功能)
* [再分级与移除 E_STRICT 通知](#reclassification-and-removal-of-e_strict-notices)
* [反对使用`password_hash()`的盐选项](#为password_hash函数设置盐的选项过期)
* [无效的八进制字面量错误](#无效的八进制字面量的错误)
* [`substr()` 返回值改动](#substr返回值修改)

**[FAQ](#faq)**
 * [PHP 6发生了什么?](#php-6发生了什么)

## 性能
毫无争议，PHP 7带来的最大的变化就是极大的性能提升，这得益于对Zend引擎的重构，新的引擎使用了更加紧凑的数据结构和更少的堆分配/回收操作。

在实际项目使用中，性能的提升效果不是固定的，虽然很多应用在使用PHP 7之后活的了超过100%的性能加速，同时还只使用更少的内存消耗！

重构后的代码基础也为将来的优化（比如JIT编译）提供了更多的可能，所以，可以预计在接下来的PHP版本仍然会看到不错的性能提升。

PHP 7性能对比图标：


 - [Turbocharging the Web with PHP 7](https://www.zend.com/en/resources/php7_infographic)
 - [Benchmarks from Rasmus's Sydney Talk](http://talks.php.net/oz15#/drupalbench)

## 特性

### 组合比较操作符
组合比较符（也称太空船操作符）用于对两个操作数进行三路比较的简化符号，它返回一个整数，其值为以下之一：

* 一个正整数（当左操作数大于右操作数时）
* 0（当两个操作数相等时）
* 一个负整数（当右操作数大于左操作数时）

该操作符和等式运算符（`==`，`!=`，`===`，`!==`）等有相同的优先级，并且和其它松比较操作符（`<`, `>=`等）有相同的行为，同样组合比较付也是非组合的，不允许对操作数进行链式调用（像 `1 <=> 2 <=> 3`）。

```PHP
// compares strings lexically
var_dump('PHP' <=> 'Node'); // int(1)

// compares numbers by size
var_dump(123 <=> 456); // int(-1)

// compares corresponding array elements with one-another
var_dump(['a', 'b'] <=> ['a', 'b']); // int(0)
```

对象是不可比较的，所以对对象使用组合比较符会导致未定义的行为。

RFC: [Combined Comparison Operator](https://wiki.php.net/rfc/combined-comparison-operator)

### 空引用合并操作符
空引用合并操作符（或者称isset三元操作符）是`isset()`检测的三元操作符的简化符号。空引用检测在应用中是经常需要的操作，为了这个目的引入了这个新的语法

```PHP
// PHP 7 之前
$route = isset($_GET['route']) ? $_GET['route'] : 'index';

// PHP 7+
$route = $_GET['route'] ?? 'index';
```

> 这有点类似于其他语言的 `?:` 合并操作符


RFC: [Null Coalesce Operator](https://wiki.php.net/rfc/isset_ternary)

### 标量类型声明
标量类型声明有两种模式： **强制**（默认）和**严格**。下面类型的参数现在能够强制执行（无论是强制模式还是严格模式）：字符串（`string`），整数(`int`），浮点数（`float`），以及布尔值（`bool`）。它们扩充了PHP 5中引入的其他类型：类名（class names），接口(interfaces)，数组(`array`)和回调(`callable`)类型。

```PHP
// 强制模式
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1)); // int(9)
```

要使用严格模式，一个 `declare()`声明指令必须放在文件的顶部，这意味着声明严格标量是基于文件可配置的。这个指令不仅影响参数的类型声明，也影响到函数的返回值声明（参考[返回值类型声明](返回值类型声明)，内置的PHP函数以及扩展中加载的函数）。

如果类型检查失败了，一个 `TypeError`异常会（见[引擎中的异常](#引擎中的异常)）抛出，在严格模式下，唯一的例外是，当整数当作一个浮点数提供到上下文时，整数到浮点数的自动类型转换（反过来不允许）。

```PHP
declare(strict_types=1);

function multiply(float $x, float $y)
{
    return $x * $y;
}

function add(int $x, int $y)
{
    return $x + $y;
}

var_dump(multiply(2, 3.5)); // float(7)
var_dump(add('2', 3)); // Fatal error: Uncaught TypeError: Argument 1 passed to add() must be of the type integer, string given...
```

注意，**只有**在*调用上下文*执行类型检查时才会适用这些规则，这意味着严格类型检查只在函数/方法调用时使用，而在它们的定义处不使用。在上面的例子中，两个函数可以定义在严格或者强制模式的文件中，但是，只要是它们在声明了严格模式的文件中被调用，就会使用严格类型规则。

**BC Breaks（Backward compatibility breaks：打破向后兼容）**
 - 现在 `int`, `string`, `float`, and `bool` 不再允许作为类名出现。


RFC: [Scalar Type Declarations](https://wiki.php.net/rfc/scalar_type_hints_v5)

### 返回值类型声明
返回类型声明允许为函数，方法或者闭包指定返回值的类型了，支持下列作为返回值类型： `string`,
`int`, `float`, `bool`, `array`, `callable`, `self` (仅方法), `parent`
(仅方法), `Closure`, 类名和接口名。

```PHP
function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
/* Output
Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)
*/
```

在子类化的情况下，返回值类型必须是不变的。这意味着一个方法被子类重写或者被实现时，它的返回值类型必须和父类中的类型完全匹配。

```PHP
class A {}
class B extends A {}

class C
{
    public function test() : A
    {
        return new A;
    }
}

class D extends C
{
    // overriding method C::test() : A
    public function test() : B // Fatal error due to variance mismatch
    {
        return new B;
    }
}
```

覆盖的方法 `D::test() : B`导致了一个 `E_COMPILE_ERROR`，因为返回值类型不允许修改，`D::test()`方法必须使用返回值类型为 `A`。

```PHP
class A {}

interface SomeInterface
{
    public function test() : A;
}

class B implements SomeInterface
{
    public function test() : A // all good!
    {
        return null; // Fatal error: Uncaught TypeError: Return value of B::test() must be an instance of A, null returned...
    }
}
```

这一次，实现的方法在执行时抛出了 `TypeError` 异常（参见 [引擎中的异常](#引擎中的异常)），这是因为 `null` 不是一个正确的返回类型的值，只有类A的实例作为返回值才是允许的。

RFC: [Return Type Declarations](https://wiki.php.net/rfc/return_types)

### 匿名类

在简单的，没有对象实例要创建的时候，匿名类是非常有用的。

```PHP
// PHP 7 之前
class Logger
{
    public function log($msg)
    {
        echo $msg;
    }
}

$util->setLogger(new Logger());

// PHP 7+
$util->setLogger(new class {
    public function log($msg)
    {
        echo $msg;
    }
});
```

同样可以传递参数到匿名类的构造函数，继承其它类或者实现某些接口，还可以像普通类一样使用 traits：

```PHP
class SomeClass {}
interface SomeInterface {}
trait SomeTrait {}

var_dump(new class(10) extends SomeClass implements SomeInterface {
    private $num;

    public function __construct($num)
    {
        $this->num = $num;
    }

    use SomeTrait;
});

/** Output:
object(class@anonymous)#1 (1) {
  ["Command line code0x104c5b612":"class@anonymous":private]=>
  int(10)
}
*/
```

在一个类中嵌套的匿名类没有对外部类的私有或者受保护的字段和方法的访问权限。为了使用外部类的 protected属性或者方法，匿名类可以通过继承外部类实现。为了使用外部类的私有的或者受保护的属性或方法，需要将外部类作为匿名类的构造函数的参数：

```PHP
<?php

class Outer
{
    private $prop = 1;
    protected $prop2 = 2;

    protected function func1()
    {
        return 3;
    }

    public function func2()
    {
        return new class($this->prop) extends Outer {
            private $prop3;

            public function __construct($prop)
            {
                $this->prop3 = $prop;
            }

            public function func3()
            {
                return $this->prop2 + $this->prop3 + $this->func1();
            }
        };
    }
}

echo (new Outer)->func2()->func3(); // 6
```

RFC: [Anonymous Classes](https://wiki.php.net/rfc/anonymous_classes)

### Unicode编码点转义语法

这可以打印一个使用双引号或者heredoc包围的UTF-8编码的Unicode编码点（codepoint）。可以接受任何有效的codepoint，并且先导的 `0`是可以省略的。


```PHP
echo "\u{aa}"; // ª
echo "\u{0000aa}"; // ª (same as before but with optional leading 0's)
echo "\u{9999}"; // 香
```

RFC: [Unicode Codepoint Escape Syntax](https://wiki.php.net/rfc/unicode_escape)

### Closure::call()

闭包的新的 `call()`方法是用于简单干练地绑定一个方法到对象上闭包并调用它。新的方法有更好的性能并且省去了在调用前创建一个中间闭包的代码，使代码更加紧凑。


```PHP
class A {private $x = 1;}

// PHP 7 之前
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // intermediate closure
echo $getX(); // 1

// PHP 7+
$getX = function() {return $this->x;};
echo $getX->call(new A); // 1
```

RFC: [Closure::call](https://wiki.php.net/rfc/closure_apply)

### 带过滤的 `unserialize()`

这个特性旨在为反序列化不可信数据时提供更好的安全性。它通过允许开发者以白名单的形式来规定可以反序列化的类，以此防止潜在的代码注入。

```PHP
// converts all objects into __PHP_Incomplete_Class object
$data = unserialize($foo, ["allowed_classes" => false]);

// converts all objects into __PHP_Incomplete_Class object except those of MyClass and MyClass2
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);

// default behaviour (same as omitting the second argument) that accepts all classes
$data = unserialize($foo, ["allowed_classes" => true]);
```

RFC: [Filtered unserialize()](https://wiki.php.net/rfc/secure_unserialize)

### `IntlChar` 类

新的 `IntChar` 类型旨在暴露更多的ICU功能，这个类自身定义了许多静态方法用于操作多字符集的unicode字符。

```PHP
printf('%x', IntlChar::CODEPOINT_MAX); // 10ffff
echo IntlChar::charName('@'); // COMMERCIAL AT
var_dump(IntlChar::ispunct('!')); // bool(true)
```

为了使用这个类，必须安装 `Intl`扩展。

**BC Breaks**
 - Classes in the global namespace must not be called `IntlChar`.

RFC: [IntlChar class](https://wiki.php.net/rfc/intl.char)

### 预期

预期是向后兼容并增强之前的 `assert()`函数。它使得在生产环境中启用断言的成本为零，并且提供当断言失败时抛出特定异常的能力。

`assert()`函数的原型如下

```
void assert (mixed $expression [, mixed $message]);
```
在旧的API中，如果 `$expression`是一个字符串，那么它会被计算，如果第一个参数falsy，那么断言失败。第二个参数可以是一个字符串（导致触发一个 AssertionError）或者是一个包含错误消息的自定义异常对象。

```PHP
ini_set('assert.exception', 1);

class CustomError extends AssertionError {}

assert(false, new CustomError('Some error message'));
```

这个特性与 PHP.ini 中的两个设置有关

 - zend.assertions = 1
 - assert.exception = 0

**zend.assertions** 有三个值:
 - **1** = 生产并执行代码 (开发模式)
 - **0** = 生成代码并在执行时跳过
 - **-1** = 不生产代码 (零开销, 生产模式)

**assert.exception** 意味着断言失败时抛出一个异常，为了与旧的 `assert()`函数保持兼容，此选项默认是关闭的。

RFC: [Expectations](https://wiki.php.net/rfc/expectations)

### Group `use` Declarations

现在的`use`声明允许根据父命令空间一次导入多个值，这一特性是为了减少引入同一命名空间下的类、函数或者常量时造成的代码冗余。

```PHP
// PHP 7 之前
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```

RFC: [Group use Declarations](https://wiki.php.net/rfc/group_use_declarations)

### 生成器返回表达式

这个特性基于PHP 5.5引入的生成器功能，该功能允许在一个生成器（generator）内使用一个`return`语句，以此使返回的最后一个表达式返回（不允许返回引用）。这个值可以使用新的 `Generator::getReturn()`方法获取，不过需要在生成器已经返回之后使用（即generator returned之后）。

```PHP
// IIFE syntax now possible - see the Uniform Variable Syntax subsection in the Changes section
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}
// 此时generator已经关闭

echo $gen->getReturn(), PHP_EOL;

// output:
// 1
// 2
// 3
```

能够明确地返回生成器的最终值是一个很便利的功能，因为generator返回最终的值（也许来自某种形式的协同计算）能够被执行生成器的客户端专门处理。这和客户端代码需要首先检查最终值是否已经产生然后在使用最终的值相比，更加简单了

RFC: [Generator Return Expressions](https://wiki.php.net/rfc/generator-return-expressions)

### 生成器委托

生成器现在能自动委托给另一个生成器、可遍历对象或者数组，直接使用 `yield from`，而不需要在最外层的生成器书写模版了。

生成器委托基于生成器能返回表达式的能力。使用新的语法 `yield from <expr>`, <expr>还可以是可遍历对象或数组。 <expr>将会被遍历，之后在返回调用的生成器继续执行。这个特性允许 `yield`表达式被切分更细，从而促进代码更加干净，有更好的可复用性。

```PHP
function gen()
{
    yield 1;
    yield 2;

    return yield from gen2();
}

function gen2()
{
    yield 3;

    return 4;
}

$gen = gen();

foreach ($gen as $val)
{
    echo $val, PHP_EOL;
}

echo $gen->getReturn();

// output
// 1
// 2
// 3
// 4
```

RFC: [Generator Delegation](https://wiki.php.net/rfc/generator-delegation)

### 使用 `intdiv()` 做整数除法

引入函数`intdiv()`用于处理需要返回一个整数的整数除法操作。

```PHP
var_dump(intdiv(10, 3)); // int(3)
```

**BC Breaks**
 - 全局命令空间下的函数不能命名为 `intdiv`.

RFC: [intdiv()](https://wiki.php.net/rfc/intdiv)

### `session_start()` 选项

这个特性允许传入一个选项的数组到 `session_start()`函数，这用于设置基于会话的 php.ini 选项：

```PHP
session_start(['cache_limiter' => 'private']); // sets the session.cache_limiter option to private
```

这个特性也引入了一个新的php.ini设置（`session.lazy_write`）,默认情况下设置为 true，意味着session数据只在发生变化时才写入。

RFC: [Introduce session_start() Options](https://wiki.php.net/rfc/session-lock-ini)

### `preg_replace_callback_array()` 函数

使用新的 `preg_replace_callback()`函数能写出更加干净的代码。在PHP 7之前，每一个正则表达式都需要执行一次的回调函数（`preg_replace_callback()`函数的第二个参数）常常需要写很多的分支判断。

现在能用关联数组注册回调了，使用正则表达式作为键，使对应的回调作为值注册到正则表达式。

函数签名如下:

```
string preg_replace_callback_array(array $regexesAndCallbacks, string $input);
```

```PHP
$tokenStream = []; // [tokenName, lexeme] pairs

$input = <<<'end'
$a = 3; // variable initialisation
end;

// PHP 7 之前
preg_replace_callback(
    [
        '~\$[a-z_][a-z\d_]*~i',
        '~=~',
        '~[\d]+~',
        '~;~',
        '~//.*~'
    ],
    function ($match) use (&$tokenStream) {
        if (strpos($match[0], '$') === 0) {
            $tokenStream[] = ['T_VARIABLE', $match[0]];
        } elseif (strpos($match[0], '=') === 0) {
            $tokenStream[] = ['T_ASSIGN', $match[0]];
        } elseif (ctype_digit($match[0])) {
            $tokenStream[] = ['T_NUM', $match[0]];
        } elseif (strpos($match[0], ';') === 0) {
            $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
        } elseif (strpos($match[0], '//') === 0) {
            $tokenStream[] = ['T_COMMENT', $match[0]];
        }
    },
    $input
);

// PHP 7+
preg_replace_callback_array(
    [
        '~\$[a-z_][a-z\d_]*~i' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_VARIABLE', $match[0]];
        },
        '~=~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_ASSIGN', $match[0]];
        },
        '~[\d]+~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_NUM', $match[0]];
        },
        '~;~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
        },
        '~//.*~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_COMMENT', $match[0]];
        }
    ],
    $input
);
```

**BC Breaks**
 - Functions in the global namespace must not be called `preg_replace_callback_array`.

RFC: [Add preg_replace_callback_array Function](https://wiki.php.net/rfc/preg_replace_callback_array)

### CSPRNG（密码学安全伪随机数构建器）函数

这一特性引入了两个新的函数，用于生成安全加密的整数和字符串。它们暴露出简单的API，并且它们是平台独立的。

函数签名:
```
string random_bytes(int length);
int random_int(int min, int max);
```

如果不能找到可用的随机性来源，两个函数都会抛出一个 `Error` 异常。

**BC Breaks**
 - 全局命名空间下的函数不能命名为 `random_int` 或者 `random_bytes`.

RFC: [Easy User-land CSPRNG](https://wiki.php.net/rfc/easy_userland_csprng)

### 支持 `define()` 定义数组常量

在PHP 5.6 引入了使用 `const`关键字定义数组常量，在PHP 7中，可以使用 `define()`函数定义数组常量了。

```PHP
define('ALLOWED_IMAGE_EXTENSIONS', ['jpg', 'jpeg', 'gif', 'png']);
```

RFC: no RFC available

### 反射增强

PHP 7引入了两个新的反射类，第一个是 `ReflectionGenerator`，用于generator的内省（introspection）。

```PHP
class ReflectionGenerator
{
    public __construct(Generator $gen)
    public array getTrace($options = DEBUG_BACKTRACE_PROVIDE_OBJECT)
    public int getExecutingLine(void)
    public string getExecutingFile(void)
    public ReflectionFunctionAbstract getFunction(void)
    public Object getThis(void)
    public Generator getExecutingGenerator(void)
}
```

第二个是 `ReflectionType`，更好的支持常量和返回值类型声明的特性。

```PHP
class ReflectionType
{
    public bool allowsNull(void)
    public bool isBuiltin(void)
    public string __toString(void)
}
```

同时，`ReflectionParameter`引入了两个新的方法：

```PHP
class ReflectionParameter
{
    // ...
    public bool hasType(void)
    public ReflectionType getType(void)
}
```

`ReflectionFunctionAbstract`也引入了两个新方法:

```PHP
class ReflectionFunctionAbstract
{
    // ...
    public bool hasReturnType(void)
    public ReflectionType getReturnType(void)
}
```

**BC Breaks**
 - Classes in the global namespace must not be called `ReflectionGenerator` or
   `ReflectionType`.

RFC: no RFC available

## 改变

### 放松保留字的限制

现在，全局保留字允许作为类，接口和traits的属性，常量和方法名。这减少了当新的关键字被引入和避免API命名限制时导致的对向后兼容的破坏。

这个改变在使用连贯接口创建内部DSLs时很有用。

```PHP
// 'new', 'private', and 'for' were previously unusable
Project::new('Project Name')->private()->for('purpose here')->with('username here');
```

唯一的限制是，`class`关键字仍然不能用作常量名，因为这样可能和类名解析语法（`ClassName::ckass`）冲突。

RFC: [Context Sensitive Lexer](https://wiki.php.net/rfc/context_sensitive_lexer)

### 统一变量语法

这一改变为PHP变量操作符带来更强的正交性，它允许很多以前不允许的新的操作符组合用法，并因此引入了相比旧的方法更加简洁的代码。

```PHP
// nesting ::
$foo::$bar::$baz // access the property $baz of the $foo::$bar property

// nesting ()
foo()() // invoke the return of foo()

// operators on expressions enclosed in ()
(function () {})() // IIFE(Immediately-Invoked Function Expression) syntax from JS
```
这种随意组合变量操作符的能力来自于转变直接变量，属性和方法应用的求值语义。新的行为更加符合直觉并且允许从左到右的求值顺序。

```PHP
                        // old meaning            // new meaning
$$foo['bar']['baz']     ${$foo['bar']['baz']}     ($$foo)['bar']['baz']
$foo->$bar['baz']       $foo->{$bar['baz']}       ($foo->$bar)['baz']
$foo->$bar['baz']()     $foo->{$bar['baz']}()     ($foo->$bar)['baz']()
Foo::$bar['baz']()      Foo::{$bar['baz']}()      (Foo::$bar)['baz']()
```

**BC Breaks**
 - 依赖于旧的求值顺序的代码需要使用大括号重写来明确求值的顺序（见上面示例的中间一列）。这样才能同时向前兼容 PHP 7.x 和向后兼容PHP 5.x

RFC: [Uniform Variable Syntax](https://wiki.php.net/rfc/uniform_variable_syntax)

### 引擎中的异常

引擎中的异常将许多致命和可恢复的致命错误转换为普通异常。这允许应用通过自定义错误处理程序实现优雅降级。同时像`finally`子句和析构函数这样的 cleanup-driven 特性将能被执行。通过使用应用错误的异常，将执行堆栈跟踪并产生更多的调试信息。

```PHP
function sum(float ...$numbers) : float
{
    return array_sum($numbers);
}

try {
    $total = sum(3, 4, null);
} catch (TypeError $typeErr) {
    // handle type error here
}
```

新的异常层级结构如下：

```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
```

查看 [可抛出的接口](#可抛出的接口) 节了解更多关于新的异常结构的信息。

**BC Breaks**
 - 用于处理可恢复致命错误的用户错误处理程序将不会有效，因为异常不再被抛出。
 - `eval()`执行代码的解析错误现在作为异常抛出，这要求它们使用 `try...catch`块包围。

RFC: [Exceptions in the Engine](https://wiki.php.net/rfc/engine_exceptions_for_php7)

### 可抛出的接口

因为引入了[引擎中的异常](#引擎中的异常)，这一改变影响了PHP的异常层次结构。这不是简单地将致命的和可会度的致命错误替换为已经存在的`Exception`类层次结构，而是实现了一个新的一个新的层次结构以防止PHP 5.x的代码中的全部捕捉（catch-all：`catch (Exception $e)`）子句会捕捉到这种新的异常。

新的异常层次如下：

```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
```

这里的 `Throwable`接口被 `Exception` 和 `Error`积累实现，接口的定义如下：

```
interface Throwable
{
    final public string getMessage ( void )
    final public mixed getCode ( void )
    final public string getFile ( void )
    final public int getLine ( void )
    final public array getTrace ( void )
    final public string getTraceAsString ( void )
    public string __toString ( void )
}
```

`Throwable` 不能被用户定义的类实现，一个用户定义的异常类应该继承之前的PHP版本中就已经存在的异常类。

RFC: [Throwable Interface](https://wiki.php.net/rfc/throwable-interface)

### 整数语义

为了使更加符合直觉和平台独立，一些基于整数的行为的语言发生了变化。下面是这些改变的列表：

 - 将 `NAN` 和 `INF` 转换为整数将总是返回0。
 - 位移操作现在禁止使用负数。(将会返回一个 bool(false)并触发一个 E_WARNING）
 - 按位左移的位数超过了整数的位数将总是返回0。
 - 按位右移的位数超过了整数的位数将总是返回0或者-1(视整数符号而定)

**BC Breaks**
 - 任何依赖于上面的旧的语义的代码将不能工作。

RFC: [Integer Semantics](https://wiki.php.net/rfc/integer_semantics)

### JSOND替代了JSON扩展

旧的的JSON扩展的许可不再是自由的（non-free），在基于Linux的发布版本中导致了很多问题。该扩展现在已经被JSOND替代，新的JSOND有一定的[性能提升](https://github.com/bukka/php-jsond-bench/blob/master/reports/0001/summary.md)，同时导致一些向后兼容问题。

**BC Breaks**
 - 数据不能以小数点结束 (比如 `34.` 必须修改为 `34.0` 或者 `34`)
 - 指数 `e` 不能紧跟在小数点后面(比如 `3.e3` 必须修改为 `3.0e3` 或者 `3e3`)

RFC: [Replace current json extension with jsond](https://wiki.php.net/rfc/jsond)

### ZPP 溢出时失败

当讲一个浮点数作为参数传递给一个期待整数的函数时，会进行浮点数到整数的强制转换。如果浮点数的值过大，将会被默认（silently）进行截断（这可能导致符号和精度的损失）。这可能导致一些很难发现的bug。这个改动就是为了当发生从浮点数到整数的隐式类型转换时提醒开发者，如果转换失败会返回`null`并触发一个`E_WARNING`。

**BC Breaks**
 - 那些在旧版本PHP能工作的代码现在可能会触发 E_WARNING异常并并导致失败，如果类型转换的结果直接被传递给其他函数的话（因为现在的版本在转换失败时返回null）。

RFC: [ZPP Failure on Overflow](https://wiki.php.net/rfc/zpp_fail_on_overflow)

### 修正 `foreach()` 的行为

PHP的　`foreach()`循环有很多奇怪的边缘条件，这些都是实现驱动的，这导致在迭代数组的拷贝或者引用时，如果使用迭代修改操作如`current()` 和 `reset()`，或者修改正在迭代的数组，会有很多未定义的和不一致的行为。

这一修改会消除边界条件的未定义行为，并且使得语义更加可预测和直观。

`foreach()` 按值遍历数组

```PHP
$array = [1,2,3];
$array2 = &$array;

foreach($array as $val) {
    unset($array[1]); // modify array being iterated over
    echo "{$val} - ", current($array), PHP_EOL;
}

// PHP 7 之前的输出
1 - 3
3 -

// PHP 7+ 的输出
1 - 1
2 - 1
3 - 1
```

当按值迭代语法使用时，被迭代的数组不是被实时修改的，`current()`现在有了明确定义的行为，它将一直指向数组的开始处。

`foreach()` 按引用编译数组和对象，与按值遍历对象

```PHP
$array = [1,2,3];

foreach($array as &$val) {
    echo "{$val} - ", current($array), PHP_EOL;
}

//PHP 7 之前的结果
1 - 2
2 - 3
3 -

// PHP 7+ 结果
1 - 1
2 - 1
3 - 1
```
`current()`函数不再被 `foreach()`在数组的迭代所影响。 并且嵌套的按引用传递的 `foreach()` 迭代能彼此独立工作：

```PHP
$array = [1,2,3];

foreach($array as &$val) {
    echo $val, PHP_EOL;

    foreach ($array as &$val2) {
        unset($array[1]);
        echo $val, PHP_EOL;
    }
}

// PHP 7 之前的结果
1
1
1

// PHP 7+ 的结果
1
1
1
3
3
3
```

**BC Breaks**
 - 任何依赖于旧的（怪异的和未文档化的）语义的嗲吗将不再能正常工作。

RFC: [Fix "foreach" behavior](https://wiki.php.net/rfc/php7_foreach)

### 修改 `list()` 的行为

旧的文档中，`list()`函数不支持strings，然而在少数的情况下，字符串仍可以使用：

```PHP
// array dereferencing
$str[0] = 'ab';
list($a, $b) = $str[0];
echo $a; // a
echo $b; // b

// object dereferencing
$obj = new StdClass();
$obj->prop = 'ab';
list($a, $b) = $obj->prop;
echo $a; // a
echo $b; // b

// function return
function func()
{
    return 'ab';
}

list($a, $b) = func();
var_dump($a, $b);
echo $a; // a
echo $b; // b
```

现在上面所列出来的情况下都**不允许**对stirng使用 `list()`。

并且空的 `list()`现在会导致一个致命错误，对变量的赋值修改为从左到右了：

```PHP
$a = [1, 2];
list($a, $b) = $a;

// OLD: $a = 1, $b = 2
// NEW: $a = 1, $b = null + "Undefined index 1"

$b = [1, 2];
list($a, $b) = $b;

// OLD: $a = null + "Undefined index 0", $b = 2
// NEW: $a = 1, $b = 2
```

**BC Breaks**
 - 使 `list()` 等于任何非直接字符串不再允许。在上面的例子中 `$a` 和 `$b` 的值都将是`null`。
 - 调用`list()`而没有任何的变量将导致一个致命的错误。
 - 任何依赖于旧的从右至左赋值的代码将不再正常工作。

RFC: [Fix list() behavior inconsistency](https://wiki.php.net/rfc/fix_list_behavior_inconsistency)

RFC: [Abstract syntax tree](https://wiki.php.net/rfc/abstract_syntax_tree)

### 改变除数为0的语义

在PHP 7之前，当除数是0时，不管是对除法（/）还是取余（%）运算符，都会触发一个 E_WARNING 同时返回false。在很多情况下，对于一个算术运算符返回一个布尔值是没有意义的（荒谬的），所以在PHP 7中该行为被改正过来了。

除数为0的行为将返回一个浮点数，即 +INF，-INF或者NAN中的一个。取余操作符的 E_WARNING 被移除（和新的`intdiv()`函数看齐），而是抛出一个 `DivisionByZeroError` 异常。此外，当一个有效的整形参数被提供而导致一个不正确的结果（由整数溢出导致）时，`intdiv()`函数也会抛出一个 `ArithmeticError`异常。

```PHP
var_dump(3/0); // float(INF) + E_WARNING
var_dump(0/0); // float(NAN) + E_WARNING

var_dump(0%0); // DivisionByZeroError

intdiv(PHP_INT_MIN, -1); // ArithmeticError
```

**BC Breaks**
 - 除法操作将不会再返回false（在算术运算中，这可能被转换为0）
 - 在除数为0时，取余操作现在可能抛出一个异常而不是返回`false`。

RFC: No RFC available

### 修复自定义session handler的返回值

当实现一个自定义的session处理程序时，`SessionHandlerInterface`的预测函数期待一个`true`或者`false`作为返回值，但它的行为并不符合预期。由于之前实现中的一个错误，只有一个`-1`返回值被认为是false，这意味着即使用于代表失败的布尔值`false`也会认为是成功了。

```PHP
<?php

class FileSessionHandler implements SessionHandlerInterface
{
    private $savePath;

    function open($savePath, $sessionName)
    {
        return false; // always fail
    }

    function close(){return true;}

    function read($id){}

    function write($id, $data){}

    function destroy($id){}

    function gc($maxlifetime){}
}

session_set_save_handler(new FileSessionHandler());

session_start(); // doesn't cause an error in pre PHP 7 code
```

现在，上面的代码会因为一个致命的错误而失败。当`-1`作为返回值时让然会失败，虽然`0`和`true`仍然被认为是成功。而返回其他的任何值都会导致失败并触发一个 E_WARNING。

**BC Breaks**
 - 如果返回`false`的话，现在会实际上失败。
 - 如果布尔值， 0， -1之外的任何值作为返回值，将会导致失败并触发一个警告信息。

RFC: [Fix handling of custom session handler return values](https://wiki.php.net/rfc/session.user.return-value)

### 返回只用PHP 4风格的构造函数

PHP 4风格的构造函数在PHP 5中被保留，和新的`__construct()`一起工作。现在PHP 4 风格的构造函数被弃用了，在创建对象是只有一个方法：`__construct()`被调用。这是因为判断PHP 4风格的构造函数是够被调用的条件导致了额外的认知开销，可能使一些没有经验的开发者感到困惑（confusing）

例如，如果一个类定义在命名空间类，或者`__construct`方法存在，那么一个PHP 4风格的构造函数被识别为一个普通方法。如果它被定义在`__construct()`方法之前，那么会触发一个 `E_STRICT` 消息，但仍然会被当作一个普通方法。命名空间中的类不识别和类同名的函数作为构造函数，而PHP 7中即使不是在命名空间中定义的类也不会将与函数同名的函数识别为构造函数，而只将 `__construct()`作为构造函数。

在 PHP 7中，如果类没有在命名空间下，并且没有定义`__construct()` 方法，那么PHP 4风格的构造函数被用作构造函数，但是会触发一个 `E_DEPRECATED`。在PHP 8中，PHP 4风格的构造器将总是被识别为普通方法，并且`E_DEPRECATED`消息将消失。

**BC Breaks**
 - 自定义的错误处理函数可能被 `E_DEPRECATED`警告所影响。为了修复这个问题，简单的讲构造器名更新为 `__construct()`即可。

RFC: [Remove PHP 4 Constructors](https://wiki.php.net/rfc/remove_php4_constructors)

### 移除date.timezone的警告

如果默认时区没有设置之前调用了任何与日期和事件相关的函数时，会触发一个警告消息。简单的做法是将INI设置的`date.timezone`为一个有效的时区，但是这要求用户有一个php.ini文件并事先配置它。因为这指示一个触发警告的的设置，并且默认总是设置为 UTC，所以这个警告信息将会被移除。

RFC: [Remove the date.timezone warning](https://wiki.php.net/rfc/date.timezone_warning_removal)

### 去除可替代的PHP标签

可替代的PHP标签 `<%` (and `<%=`), `%>`, `<script language="php">`, 和
`</script>` 被移除了。

**BC Breaks**
 - 那些依赖于上面这些可替换标签的代码需要更新为普通的或者短的开闭标签，这可以手动修改或者使用[自动脚本](https://gist.github.com/nikic/74769d74dad8b9ef221b)转换。

RFC: [Remove alternative PHP tags](https://wiki.php.net/rfc/remove_alternative_php_tags)

### 移除Switch语句中的多默认块

在以前，switch语句中可以有多个`defaul`t块（只有最后一个`default`块被执行），这种没用的功能现在被移除了，多个`default`块将会导致一个致命的错误。

**BC Breaks**
 - 任何在switch语句中使用了多个`default`块的代码现在将导致一个致命错误

RFC: [Make defining multiple default cases in a switch a syntax error](https://wiki.php.net/rfc/switch.default.multiple)

### 移除参数列表中重名参数的重定义

在以前的版本的函数定义中，可以有重复名字的参数，这个能力现在被移除了，如果存在重名的参数现在将导致一个致命错误。

```PHP
function foo($version, $version)
{
    return $version;
}

echo foo(5, 7);

// Pre PHP 7 result
7

// PHP 7+ result
Fatal error: Redefinition of parameter $version in /redefinition-of-parameters.php
```

**BC Breaks**
 - 有重名参数的函数现在将导致一个错误。

### 移除已经Dead的服务器API

下列SAPIs已经从核心中移除，大部分已经从PECL中移除了：

- sapi/aolserver
- sapi/apache
- sapi/apache_hooks
- sapi/apache2filter
- sapi/caudium
- sapi/continuity
- sapi/isapi
- sapi/milter
- sapi/nsapi
- sapi/phttpd
- sapi/pi3web
- sapi/roxen
- sapi/thttpd
- sapi/tux
- sapi/webjames
- ext/mssql
- ext/mysql
- ext/sybase_ct
- ext/ereg

RFC: [Removal of dead or not yet PHP7 ported SAPIs and extensions](https://wiki.php.net/rfc/removal_of_dead_sapis_and_exts)

### 移除数字字符串的十六进制支持

一个十六进制数字的字符串不再认为是数字的（numerical）。

```PHP
var_dump(is_numeric('0x123'));
var_dump('0x123' == '291');
echo '0x123' + '0x123';

// PHP 7 之前
bool(true)
bool(true)
582

// PHP 7+
bool(false)
bool(false)
0
```

这么做的原因是为了推进在语言层面对处理十六进制字符串的一致性。例如，下面指定的转换并不会视为十六进制数字符串：

```PHP
var_dump((int) '0x123'); // int(0)
```

作为替代，字符串的十六进制字符应该用`filter_var()`函数来验证和转换：

```PHP
var_dump(filter_var('0x123', FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX)); // int(291)
```

**BC Breaks**
 - 这一改变影响了 `is_numeric()` 函数和很多操作符，包括 `==`, `+`, `-`, `*`, `/`, `%`, `**`, `++`, 和 `--`

RFC: [Remove hex support in numeric strings](https://wiki.php.net/rfc/remove_hex_support_in_numeric_strings)

### 移除过期的功能


所有标记为过期的功能都被移除了，最值得一提的包括：

 - 原生mysql扩展(ext/mysql)
 - ereg扩展 (ext/ereg)
 - Assigning `new` by reference
 - 在不合适的`$this`上下文调用一个非静态方法。 (例如从类外调用 `Foo::bar()`，而 `bar()` 不是一个静态方法)

**BC Breaks**
 - 任何在PHP 5中运行有已过期警告的代码都将不能工作。(you were warned!)

RFC: [Remove deprecated functionality in PHP 7](https://wiki.php.net/rfc/remove_deprecated_functionality_in_php7)

### Reclassification and Removal of E_STRICT Notices

E_STRICT消息总是灰色领域的意思。 这一改变溢出了这一错误分类。 无论是说移除 `E_STRICT`消息，或者说是将它改为 `E_DEPRECATED`,或是修改为 E_NOTICE了，或者是提升为一个E_WARNING了。

**BC Breaks**
 - 因为E_STRICT是最低的严重错误类别，任何错误提升为`E_WARNING`都可能导致破坏自定义的错误处理程序。

RFC: [Reclassify E_STRICT notices](https://wiki.php.net/rfc/reclassify_e_strict)

###  为`password_hash()`函数设置盐的选项过期

随着在PHP 5.5中引入了新的密码哈希API，许多人开始实现它并自己生产盐。不幸的是，生成的这些盐许多都是来自密码学不安全的函数，比如 `mt_rand()`，这使得这些盐远比默认生成的盐要弱！（是的，使用这个新的API来哈希密码默认会使用盐）。为了防止开发正生成不安全的盐，生成盐的选项在PHP 7中被deprecated了。

RFC: no RFC available

### 无效的八进制字面量的错误

无效的八进制字面量现在会导致一个解析错误而不是静默被截断而被忽略。

```PHP
echo 0678; // Parse error:  Invalid numeric literal in...
```

**BC Breaks**
 - 任何无效的八进制字面量会导致解析错误。

RFC: no RFC available

### `substr()`返回值修改

当进行截断的开始位置等于字符的长度时，`substr()`现在将返回一个空的字符串而不是`false`。

```PHP
var_dump(substr('a', 1));

// PHP 7 之前
bool(false)

// PHP 7+
string(0) ""
```

然而，在其他情况`substr()`仍然可能返回`false`。


**BC Breaks**
 - 那些严格检查返回值为`bool(false)`的代码现在可能是语义无效的了。


RFC: no RFC available

## FAQ

### PHP 6发生了什么

PHP 6 was the major PHP version that never came to light. It was supposed to
feature full support for Unicode in the core, but this effort was too ambitious
with too many complications arising. The predominant reasons why version 6 was
skipped for this new major version are as follows:
 - **To prevent confusion**. Many resources were written about PHP 6 and much
   of the community knew what was featured in it. PHP 7 is a completely
different beast with entirely different focuses (specifically on performance)
and entirely different feature sets. Thus, a version has been skipped to
prevent any confusion or misconceptions surrounding what PHP 7 is.
 - **To let sleeping dogs lie**. PHP 6 was seen as a failure and a large amount
   of PHP 6 code still remains in the PHP repository. It was therefore seen as
best to move past version 6 and start afresh on the next major version, version
7.

RFC: [Name of Next Release of PHP](https://wiki.php.net/rfc/php6)
