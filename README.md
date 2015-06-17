# PHP 7

PHP 7 has been slated for release [in November of this year](https://wiki.php.net/rfc/php7timeline) (2015). It comes with a number of new features, changes, and backwards compatibility breakages that are outlined below.

**[Performance](#performance)**

**[Features](#features)**
* [Combined Comparison Operator](#combined-comparison-operator)
* [Null Coalesce Operator](#null-coalesce-operator)
* [Scalar Type Declarations](#scalar-type-declarations)
* [Return Type Declarations](#return-type-declarations)
* [Unicode Codepoint Escape Syntax](#unicode-codepoint-escape-syntax)
* [Closure `call()` Method](#closure-call-method)
* [Filtered `unserialize()`](#filtered-unserialize)
* [`IntlChar` Class](#intlchar-class)
* [`session_start()` Options](#session_start-options)
* [Expectations](#expectations)
* [Group `use` Declarations](#group-use-declarations)
* [Generator Return Expressions](#generator-return-expressions)
* [Generator Delegation](#generator-delegation)
* [Integer Division with `intdiv()`](#integer-division-with-intdiv)
* [`preg_replace_callback_array()` Function](#preg_replace_callback_array-function)

**[Changes](#changes)**
* [JSON Extension Replaced with JSOND](#json-extension-replaced-with-jsond)
* [Integer Semantics](#integer-semantics)
* [ZPP Failure on Overflow](#zpp-failure-on-overflow)
* [Uniform Variable Syntax](#uniform-variable-syntax)
* [Exceptions in the Engine](#exceptions-in-the-engine)
* [Throwable Interface](#throwable-interface)
* [Fixes to `foreach()`'s Behaviour](#fixes-to-foreachs-behaviour)
* [Fixes to `list()`'s Behaviour](#fixes-to-lists-behaviour)
* [Fixes to Custom Session Handler Return Values](#fixes-to-custom-session-handler-return-values)
* [Deprecation of PHP 4-Style Constructors](#deprecation-of-php-4-style-constructors)
* [Removal of date.timezone Warning](#removal-of-date.timezone-warning)
* [Removal of Alternative PHP Tags](#removal-of-alternative-php-tags)
* Removal of Multiple Default Blocks in Switch Statements
* Removal of Dead Server APIs
* Removal of Hex Support in Numerical Strings
* Reclassification and Removal of E_STRICT Notices
* Deprecation of Salt Option for `password_hash()`

**[FAQ](#faq)**
 * What happened to PHP 6?

## Performance

Unarguably the greatest part about PHP 7 is the incredible performance boosts it provides to applications.

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
// Pre PHP 7 code
$route = isset($_GET['route']) ? $_GET['route'] : 'index';

// PHP 7+ code
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

### Closure call() Method

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

### `IntlChar` Class

The new `IntlChar` class seeks to expose additional ICU functionality. The class itself defines a number of static methods and constants that can be used to manipulate unicode characters.

```PHP
printf('%x', IntlChar::CODEPOINT_MAX); // 10ffff
echo IntlChar::charName('@'); // COMMERCIAL AT
var_dump(IntlChar::ispunct('!')); bool(true)
```

In order to use this class, you will need the `Intl` extension installed.

RFC: [IntlChar class](https://wiki.php.net/rfc/intl.char)

### `session_start()` Options

This feature gives the ability to pass in an array of options to the `session_start()` function. This is used to set session-based php.ini options:

```PHP
session_start(['cache_limiter' => 'private']); // sets the session.cache_limiter option to private
```

This feature also introduces a new php.ini setting (`session.lazy_write`) that is, by default, set to true and means that session data is only rewritten if it changes.

RFC: [Introduce session_start() Options](https://wiki.php.net/rfc/session-lock-ini)

### Expectations

Expectations are an enhancement to the older `assert()` function. They enable for zero-cost assertions in production code, and provide the ability to throw custom exceptions on error.

The `assert()` function's prototype is as follows:
```
void assert (mixed $expression [, mixed $message]);
```

As with the old API, if `$expression` is a string, then it will be evaluated. If the first argument is falsy, then the assertion fails. The second argument can either be a plain string (causing an AssertionException to be triggered), or a custom exception object containing an error message.

```PHP
ini_set("assert.exception", 1);

class CustomError extends AssertionException {}

assert(false, new CustomError("Some error message"));
```

With this feature comes two PHP.ini settings (along with their default values):
 - zend.assertions = 1
 - assert.exception = 0

**zend.assertions** has three values:
 - **1** = generate and execute code (development mode)
 - **0** = generate code and jump around at it at runtime
 - **-1** = don't generate any code (zero-cost, production mode)

**assert.exception** means that an exception is thrown when an assertion fails. This is switched off by default to remain compatible with the old `assert()` function.

RFC: [Expectations](https://wiki.php.net/rfc/expectations)

### Group `use` Declarations

This gives the ability to group multiple `use` declarations according to the parent namespace. This seeks to remove code verbosity when importing multiple classes that come under the same namespace.

```PHP
// Pre PHP 7 code
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

// PHP 7+ code
use parent\child\{
    ClassA,
    ClassB,
    ClassC as C
};
```

RFC: [Group use Declarations](https://wiki.php.net/rfc/group_use_declarations)

### Generator Return Expressions

This feature builds upon the generator functionality introduced into PHP 5.5. It enables for a `return` statement to be used within a generator to enable for a final *expression* to be returned (return by reference is not allowed). This value can be fetched using the new `Generator::getReturn()` method, which may only be used once the generator has finishing yielding values.

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

echo $gen->getReturn(), PHP_EOL;

// output:
// 1
// 2
// 3
```

Being able to explicitly return a final value from a generator is a handy ability to have. This is because it enables for a final value to be returned by a generator (from perhaps some form of coroutine computation) that can be specifically handled by the client code executing the generator. This is far simpler than forcing the client code to firstly check whether the final value has been yielded, and then if so, to handle that value specifically.

RFC: [Generator Return Expressions](https://wiki.php.net/rfc/generator-return-expressions)

### Generator Delegation

Generator delegation builds upon the ability of being able to return expressions from generators. It does this by using an new syntax of `yield from <expr>`, where <expr> can be any `Traversable` object or array. This <expr> will be advanced until no longer valid, and then execution will continue in the calling generator. This feature enables `yield` statemets to be broken down into smaller operations, thereby promoting cleaner code that has greater reusability.

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

### Integer Division with `intdiv()`

The `intdiv()` function has been introduced to handle division where an integer is to be returned.

```PHP
var_dump(intdiv(10, 3)); // int(3)
```

RFC: [intdiv()](https://wiki.php.net/rfc/intdiv)

### `preg_replace_callback_array()` Function

This new function addition enables for code to be written more cleanly when using the `preg_replace_callback()` function. Prior to PHP 7, callbacks that needed to be executed per regular expression match required the callback function (second parameter of `preg_replace_callback()`) to be polluted with lots of branching (a hacky method at best). But now, callbacks can be registered on a per-regular expression basis using an associative array with the regular expression as the keys and the callbacks as the values.

```PHP
$tokenStream = []; // [tokenName, lexeme] pairs

$input = <<<'end'
$a = 3; // variable initialisation
end;

// Pre PHP 7 code
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
        } else {
            // skip whitespace
        }
    },
    $input
);

// PHP 7+ code
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

RFC: [Add preg_replace_callback_array Function](https://wiki.php.net/rfc/preg_replace_callback_array)

## Changes

### JSON Extension Replaced with JSOND

The licensing of the old JSON extension was regarded as non-free, causing issues for many Linux-based distributions. The extension has since been replaced with JSOND and comes with some [performance gains](https://github.com/bukka/php-jsond-bench/blob/master/reports/0001/summary.md) and backward compatibility breakages.

**BC Breaks**
 - A number *must not* end in a decimal point (i.e. `34.` must be changed to either `34.0` or just `34`)
 - The `e` exponent *must not* immediately follow the decimal point (i.e. `3.e3` must be changed to either `3.0e3` or just `3e3`)

RFC: [Replace current json extension with jsond](https://wiki.php.net/rfc/jsond)

### Integer Semantics

The semantics for some integer-based behaviour has changed in an effort to make them more intuitive and platform-independent. Here is a list of those changes:
 - Casting `NAN` and `INF` to an integer will always result in 0
 - Bitwise shifting by a negative number of bits is now disallowed (causes a bool(false) return an emits an E_WARNING)
 - Left bitwise shifts by a number of bits beyond the bit width of an integer will always result in 0
 - Right bitwise shifts by a number of bits beyond the bit width of an integer will always result in 0 or -1 (sign dependent)

**BC Breaks**
 - Any reliance on the old semantics for the above will no longer work

RFC: [Integer Semantics](https://wiki.php.net/rfc/integer_semantics)

### ZPP Failure on Overflow

Coercion between floats to integers can occur when a float is passed to an internal function expecting an integer. If the float is too large to represent as an integer, then the value will be silently truncated (which may result in a loss of magnitude and sign). This can introduce hard-to-find bugs. This change therefore seeks to notify the developer when an implicit conversion from a float to an integer has occurred and failed by returning `null` and emitting an E_WARNING.

**BC Breaks**
 - Code that once silently worked will now emit an E_WARNING and may fail if the result of the function invocation is directly passed to another function (since `null` will now be passed).

RFC: [ZPP Failure on Overflow](https://wiki.php.net/rfc/zpp_fail_on_overflow)

### Uniform Variable Syntax

This change brings far greater orthogonality to the variable operators in PHP. It enables for a number of new combinations of operators that were previously disallowed, and so introduces new ways to achieve old operations in terser code.

```PHP
// nesting ::
$foo::$bar::$baz // access the property $baz of the $foo::$bar property

// nesting ()
foo()() // invoke the return of foo()

// operators on expressions enclosed in ()
(function () {})() // IIFE syntax from JS

// operator support on dereferencable scalars
'string'->toUpper(); // call the toUpper() method on the string 'string'
```

The ability to arbitrarily combine variable operators came from reversing the evaluation semantics of indirect variable, property, and method references. The new behaviour is more intuitive, always following a left-to-right evaluation order:

```PHP
                        // old meaning            // new meaning
$$foo['bar']['baz']     ${$foo['bar']['baz']}     ($$foo)['bar']['baz']
$foo->$bar['baz']       $foo->{$bar['baz']}       ($foo->$bar)['baz']
$foo->$bar['baz']()     $foo->{$bar['baz']}()     ($foo->$bar)['baz']()
Foo::$bar['baz']()      Foo::{$bar['baz']}()      (Foo::$bar)['baz']()
```

**BC Breaks**
 - Code that relied upon the old evaluation order must be rewritten to explicity use that evaluation order with curly braces (see middle column of the above). This will make the code both forwards compatible with PHP 7.x and backwards compatible with PHP 5.x

RFC: [Uniform Variable Syntax](https://wiki.php.net/rfc/uniform_variable_syntax)

### Exceptions in the Engine

Exceptions in the engine converts many fatal and recoverable fatal errors into exceptions. This enables for graceful degradation of applications through custom error handling procedures. It also means that cleanup-driven features such as the `finally` clause and object destructors will now be executed. Furthermore, by using exceptions for application errors, stack traces will be produced for additional debugging information.

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

The new exception hierarchy is as follows:
```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
```

See the [Throwable Interface](#throwable-interface) subsection in the Changes section for more information on this new exception hierarchy.

**BC Breaks**
 - Custom error handlers used for handling (and typically ignoring) recoverable fatal errors will not longer work since exceptions will now be thrown
 - Parse errors occuring in `eval()`ed code will now become exceptions, requiring them to be wrapped in a try...catch block

RFC: [Exceptions in the Engine](https://wiki.php.net/rfc/engine_exceptions_for_php7)

### Throwable Interface

This change affects PHP's exception hierarchy due to the introduction of [exceptions in the engine](#exceptions-in-the-engine). Rather than placing fatal and recoverable fatal errors under the pre-existing `Exception` class hierarchy, [it was decided](https://wiki.php.net/rfc/engine_exceptions_for_php7#doodle__form__introduce_and_use_baseexception) to implement a new hierarchy of exceptions to prevent PHP 5.x code from catching these new exceptions with catch-all (`catch (Exception $e)`) clauses.

The new exception hierarchy is as follows:
```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
```

The `Throwable` interface is implemented by both `Exception` and `Error` base class hierarchies and defines the following contract:
```
final public string getMessage ( void )
final public mixed getCode ( void )
final public string getFile ( void )
final public int getLine ( void )
final public array getTrace ( void )
final public string getTraceAsString ( void )
public string __toString ( void )
```

It cannot be implemented by user-defined classes - instead, a custom exception class should extend one of the pre-existing exceptions classes in PHP.

RFC: [Throwable Interface](https://wiki.php.net/rfc/throwable-interface)

### Fixes to `foreach()`'s Behaviour

PHP's `foreach()` loop had a number of strange edge-cases to it. These were all implementation-driven and caused a lot of undefined and inconsistent behaviour when iterating between copies and references of an array, when using iterator manipulators like `current()` and `reset()`, when modifying the array currently being iterated, and so on.

This change eliminates the undefined behaviour of these edge-cases and makes the semantics more predictable and intuitive.

`foreach()` by value on arrays
```PHP
$array = [1,2,3];
$array2 = &$array;

foreach($array as $val) {
    unset($array[1]); // modify array being iterated over
    echo "{$val} - ", current($array), PHP_EOL;
}

// Pre PHP 7 result
1 - 3
3 - 

// PHP 7+ result
1 - 1
2 - 1
3 - 1
```

When by-value semantics are used, the array being iterated over is now not modified in-place. `current()` also now has defined behaviour, where it will always begin at the start of the array.

`foreach()` by reference on arrays and objects and by value on objects
```PHP
$array = [1,2,3];

foreach($array as &$val) {
    echo "{$val} - ", current($array), PHP_EOL;
}

// Pre PHP 7 result
1 - 2
2 - 3
3 - 

// PHP 7+ result
1 - 1
2 - 1
3 - 1
```

The `current()` function is no longer affected by `foreach()`'s iteration on the array. Also, nested `foreach()`'s using by-reference semantics work independently from each other now:
```PHP
$array = [1,2,3];

foreach($array as &$val) {
    echo $val, PHP_EOL;

    foreach ($array as &$val2) {
        unset($array[1]);
        echo $val, PHP_EOL;
    }
}

// Pre PHP 7 result
1
1
1

// PHP 7+ result
1
1
1
3
3
3
```

**BC Breaks**
 - Any reliance on the old (quirky and undocumented) semantics will no longer work.

RFC: [Fix "foreach" behavior](https://wiki.php.net/rfc/php7_foreach)

### Fixes to `list()`'s Behaviour

The `list()` function was documented as not supporting strings, however in few cases strings could have been used:
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

This has now been changed making string usage with `list()` forbidden in all cases.

**BC Breaks**
 - Making `list()` equal to any non-direct string value is no longer possible. `null` will now be the value for the variable `$a` and `$b` in the above examples.

RFC: [Fix list() behavior inconsistency](https://wiki.php.net/rfc/fix_list_behavior_inconsistency)

### Fixes to Custom Session Handler Return Values

When implementing custom session handlers, predicate functions from the `SessionHandlerInterface` that expect a `true` or `false` return value did not behave as expected. Due to an error in the previous implementation, only a `-1` return value was considered false - meaning that any even if the boolean `false` was used to denote a failure, it was taken as a success:
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

session_start(); // should cause an error but doesn't
```

Now, the above will fail with a fatal error. Having a `-1` return value will also continue to fail, whilst `0` and `true` will continue to mean success. Any other value returned will now cause a failure and emit an E_WARNING.

**BC Breaks**
 - If boolean `false` is returned, it will actually fail now
 - If anything other than a boolean, `0`, or `-1` is returned, it will fail and cause a warning to be emitted

RFC: [Fix handling of custom session handler return values](https://wiki.php.net/rfc/session.user.return-value)

### Deprecation of PHP 4-Style Constructors

PHP 4 constructors were preserved in PHP 5 alongside the new `__construct()`. Now, PHP 4-style constructors are being deprecated in favour of having only a single method (`__construct()`) to be invoked on object creation. This is because the conditions upon whether the PHP 4-style constructor was invoked caused additional cognitive overhead to developers that could also be confusing to the inexperienced.

For example, if the class is defined within a namespace or if an `__construct()` method existed, then a PHP 4-style constructor was recognised as a plain method. If it was defined above an `__construct()` method, then an E_STRICT notice would be emitted, but still recognised as a plain method.

Now in PHP 7, if the class is not in a namespace and there is no `__construct()` method present, the PHP 4-style constructor will be used as a constructor but an E_DEPRECATED will be emitted. In PHP 8, the PHP 4-style constructor will always be recognised as a plain method and the E_DEPRECATED notice will disappear.

**BC Breaks**
 - Custom error handlers may be affected by the raising of E_DEPRECATED warnings. To fix this, simply update the class constructor name to `__construct`.

RFC: [Remove PHP 4 Constructors](https://wiki.php.net/rfc/remove_php4_constructors)

### Removal of date.timezone Warning

When any date- or time-based functions were invoked and a default timezone had not been set, a warning was emitted. The fix was to simply set the `date.timezone` INI setting to a valid timezone, but this forced users to have a php.ini file and to configure it beforehand. Since this was the only setting that had a warning attached to it, and it defaulted to UTC anyway, the warning has now been removed.

RFC: [Remove the date.timezone warning](https://wiki.php.net/rfc/date.timezone_warning_removal)

### Removal of Alternative PHP Tags

The alternative PHP tags `<%` (and `<%=`), `%>`, `<script language="php">`, and `</script>` have now been removed.

**BC Breaks**
 - If your code relied upon these alternative tags, then they need to be updated to either the normal or short opening and closing tags. This can either be done manually or automated with [this porting script](https://gist.github.com/nikic/74769d74dad8b9ef221b).

RFC: [Remove alternative PHP tags](https://wiki.php.net/rfc/remove_alternative_php_tags)

## FAQ
