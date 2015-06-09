# PHP 7

PHP 7 has been slated for release [in November of this year](https://wiki.php.net/rfc/php7timeline) (2015). It comes with a number of new features, changes, and backwards compatibility breakages that are outlined below.


**Features**
* [Combined Comparison Operator](#combined-comparison-operator)
* [Null Coalesce Operator](#null-coalesce-operator)
* [Scalar Type Declarations](#scalar-type-declarations)
* [Return Type Declarations](#return-type-declarations)
* [Unicode Codepoint Escape Syntax](#unicode-codepoint-escape-syntax)
* [Closure `call()` Method](#closure-call-method)
* [Filtered `unserialize()`](#filtered-unserialize)
* `IntlChar` Class
* `session_start()` Options
* Expectations
* [Group `use` Declarations](#group-use-declarations)
* Generator Return Expressions
* [Integer Division with `intdiv()`](#integer-division-with-intdiv)
* `preg_replace_callback_array()` Function


**Changes**
* JSON Extension Replaced with JSOND
* Integer Semantics
* ZPP Failure on Overflow
* Variable Syntax Uniformity
* Non-Object Call Errors
* Exceptions in the Engine
* Fixes to `foreach()`'s Behaviour
* Fixes to `list()`'s Behaviour
* Fixes to Custom Session Handler Return Values
* Removal of PHP 4-Style Constructors
* Removal of date.timezone Warning
* Removal of Alternative PHP Tags
* Removal of Multiple Default Blocks in Switch Statements
* Removal of Dead Server APIs
* Removal ofHex Support in Numerical Strings
* Reclassification and Removal of E_STRICT Notices
* Deprecation of Salt Option for `password_hash()`


**Backward Compatibility Breakages**
* Removal of PHP 4-Style Constructors
* JSON Extension Replaced with JSOND
* Unicode Codepoint Escape Syntax
* Integer Semantics
* ZPP Failure on Overflow
* Uniform Variable Syntax
* Fixes to list()'s Behaviour
* Removal of Alternative PHP Tags
* Removal of Multiple Default Switch Statement Blocks
* Removal of Hex Support in Numerical Strings
* Integer Division with intdiv()
* Fixes to Custom Session Handler Return Values


## Features

### Combined Comparison Operator
The combined comparison operator (or spaceship operator) is a short-hand notation for performing three-way comparisons from two operands. It has an integer return value that can be either:

* positive integer (if the left-hand operand is greater than the right-hand operand)
* 0 (if both operands are equal)
* negative integer (if the right-hand operand is greater than the left-hand operand)

The operator has the same precedence as the equality operators (`==`, `!=`, `===`, `!==`) and has the exact same behaviour as the other loose comparison operators (`<`, `>=`, etc). It is also non-associative like them too, so chaining of the operands (like `1 <=> 2 <=> 3`) is not allowed.

```PHP
// compares strings lexically
var_dump('PHP' <=> 'Node'); // int(1)

// compares numbers by size
var_dump(123 <=> 456); // int(-1)

// compares corresponding array elements with one-another
var_dump(['a', 'b'] <=> ['a', 'b']); // int(0)
```

Objects are not comparable, and so using them as oparands with this operator will result in undefined behaviour.

RFC: [Combined Comparison Operator](https://wiki.php.net/rfc/combined-comparison-operator)

### Null Coalesce Operator
The null coalesce operator (or isset ternary operator) is a short-hand notation for performing `isset()` checks in the ternary operator. This is a common thing to do in applications, and so a new syntax has been introduced for this exact purpose.

```PHP
// old style
$route = isset($_GET['route']) ? $_GET['route'] : 'index';

// new style
$route = $_GET['route'] ?? 'index';
```

RFC: [Null Coalesce Operator](https://wiki.php.net/rfc/isset_ternary)

### Scalar Type Declarations
Scalar type declarations come in two flavours: **coercive** (default) and **strict**. The following types for parameters can now be enforced (either coercively or strictly): strings (`string`), integers (`int`), floating-point numbers (`float`), and booleans (`bool`). They augment the other types introduced in the PHP 5.x versions: class names, interfaces, `array` and `callable`.

```PHP
// Coercive mode
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1)); // int(9)
```

To enable strict mode, a single `declare()` directive must be placed at the top of the file. This means that the strictness of typing for scalars is configured on a per file basis. This directive not only affects the type declarations of parameters, but also of function return types (see [Return Type Declarations](#return-type-declarations)), built-in PHP functions, and functions from loaded extensions.

If the type-check fails, then an `E_RECOVERABLE_ERROR` is produced. The only leniency present in strict typing is the automatic conversion of integers to floats (but not vice-versa) when an integer is provided in a float context.

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
var_dump(add('2', 3)); // Fatal error: Argument 1 passed to add() must be of the type integer, string given...
```

Note that **only** the *invocation context* applies when the type-checking is performed. This means that the strict typing applies only to function/method calls, and not to the function/method definition context. In the above example, the two functions could have been declared in either a strict or coercive file, but so long as they're being called in a strict file, then the strict typing rules will apply.

RFC: [Scalar Type Declarations](https://wiki.php.net/rfc/scalar_type_hints_v5)

### Return Type Declarations
Return type declarations enable you to specify the return type of a function, method, or closure. The following return types are supported: `string`, `int`, `float`, `bool`, `array`, `callable`, `self` (methods only), `parent` (methods only), `Closure`, the name of a class, and the name of an interface.

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

With respect to subtyping, **invariance** has been chosen for return types. This simply means that when a method is either overridden in a subtyped class or implemented as defined in a contract, its return type must match exactly the method it is (re)implementing.

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
    public function test() : B // causes a variance mismatch error
    {
        return new B;
    }
}
```

The overriding method `D::test() : B` causes an `E_COMPILE_ERROR` because covariance is not allowed. In order for this to work, `D::test()` method must have a return type of `A`.

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
        return null; // not good!
    }
}
```

This time, the implemented method causes an `E_RECOVERABLE_ERROR` when executed because `null` is not a valid return type - only an instance of the class `A` can be returned.

RFC: [Return Type Declarations](https://wiki.php.net/rfc/return_types)

### Unicode Codepoint Escape Syntax

This enables you to output a UTF-8 encoded unicode codepoint in either a double-quoted string or a heredoc. Any valid codepoint is accepted, with leading `0`'s being optional.

```PHP
echo "\u{aa}"; // ª
echo "\u{0000aa}"; // ª (same as before but with optional leading 0's)
echo "\u{9999}"; // 香
```

RFC: [Unicode Codepoint Escape Syntax](https://wiki.php.net/rfc/unicode_escape)

### Closure call() method

The new call() method for closures is used as a shorthand way of invoking a closure whilst binding an object scope to it. This creates more perfomant and compact code by removing the need to create an intermediate closure before invoking it.

```PHP
class A {private $x = 1;}

// Pre PHP 7 code
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A');
echo $getX(); // 1

// PHP 7+ code
$getX = function() {return $this->x;};
echo $getX->call(new A); // 1
```

RFC: [Closure::call](https://wiki.php.net/rfc/closure_apply)

### Filtered `unserialize()`

This feature seeks to provide better security when unserializing objects on untrusted data. It prevents possible code injections by enabling the developer to whitelist classes that can be unserialized.

```PHP
// converts all objects into __PHP_Incomplete_Class object
$data = unserialize($foo, ["allowed_classes" => false]);

// converts all objects into __PHP_Incomplete_Class object except those of MyClass and MyClass2
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);

// default behaviour (same as omitting the second argument) that accepts all classes
$data = unserialize($foo, ["allowed_classes" => true]);
```

RFC: [Filtered unserialize()](https://wiki.php.net/rfc/secure_unserialize)

### Group `use` Declarations

This gives the ability to group multiple `use` declarations according to the parent namespace. This seeks to remove code verbosity when importing multiple classes that come under the same namespace.

```PHP
// PHP 5.6 and below
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

// PHP 7+
use parent\child\{
    ClassA,
    ClassB,
    ClassC as C
};
```

RFC: [Group use Declarations](https://wiki.php.net/rfc/group_use_declarations)

### Integer Division with `intdiv()`

The `intdiv()` function has been introduced to handle division where an integer is to be returned.

```PHP
var_dump(intdiv(10, 3)); // int(3)
```

RFC: [intdiv()](https://wiki.php.net/rfc/intdiv)
