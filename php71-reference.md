# PHP 7.1

PHP 7.1 will be released towards the end of 2016, and with it comes some
interesting features and changes (outlined below).

**[Features](#features)**
* [Nullable Types](#nullable-types)
* [Void Functions](#void-functions)
* [Symmetric Array Destructuring on Assignment](#symmetric-array-destructuring-on-assignment)
* [Class Constant Visibility](#class-constant-visibility)
* [`iterable` Pseudo-Type](#iterable-pseudo-type)
* [Multi Catch Exception Handling](#multi-catch-exception-handling)
* [Support for Keys in `list()`](#support-for-keys-in-list)
* [Support for Negative String Offsets](#support-for-negative-string-offsets)
* [Support for AEAD in ext/openssl](#support-for-aead-in-extopenssl)
* [Convert callables to `Closure`s with `Closure::fromCallable()`](#convert-callables-to-closures-with-closurefromcallable)
* [Asynchronous Signal Handling with `pcntl_async_signals()`](#asynchronous-signal-handling-with-pcntl_async_signals)
* [Additions to ext/curl](#additions-to-extcurl)
  * [Support for HTTP/2 Server Push](#support-for-http2-server-push)
  * [Better Error Retrieval](#better-error-retrieval)

**[Changes](#changes)**
* [Apprise on Arithmetic with Invalid Strings](#apprise-on-arithmetic-with-invalid-strings)
* [Deprecation of ext/mcrypt](#deprecation-of-extmcrypt)
* [Deprecation of `mb_ereg_replace()` Eval Option](#deprecation-of-mb_eregi_replace-eval-option)
* [Warn on Octal Overflow](#warn-on-octal-overflow)
* [Forbid Dynamic Calls to Scope Introspection Functions](#forbid-dynamic-calls-to-scope-introspection-functions)
* [Throw on Passing too few Function Arguments](#throw-on-passing-too-few-function-arguments)
* [Inconsistency Fixes to `$this`](#inconsistency-fixes-to-this)
* [Session ID Generation without Hashing](#session-id-generation-without-hashing)
* [RNG Fixes and Changes](#rng-fixes-and-changes)
  * [Fix `mt_rand()` Implementation](#fix-mt_rand-implementation)
  * [Alias `rand()` to `mt_rand()`](#alias-rand-to-mt_rand)


## Features

### Nullable Types

Types can now be made nullable, enabling for either the specified type or
`null` to be passed to or returned from a function. The syntax chosen is the
same used by Hack, where a question mark is used to prefix a type to denote it
as nullable.

```php
function test(?string $name)
{
    var_dump($name);
}

test('tpunt'); // string(5) "tpunt"
test(null); // NULL
test(); // Uncaught Error: Too few arguments to function test(), 0 passed in...
```

Invoking `test()` without any arguments causes an `Error` exception to be
thrown (see [Throw on Passing too few Function Arguments](#throw-on-passing-too-few-function-arguments)).
This is because nullable types are distinctly different from default
parameters, where they still require an argument to be passed.

With respect to subtyping, the following rules hold true:
 - Parameter types of subclasses may add type nullability
 - Return types of subclasses may remove type nullability

```php
interface ParentClass
{
    function test(string $param) : ?string;
}

interface ChildClass2 extends ParentClass
{
    // parameter type made nullable
    // return type made un-nullable
    function test(?string $param) : string;
}
```

RFC: [Nullable Types](https://wiki.php.net/rfc/nullable_types)

### Void Functions

A `void` return type has been introduced to amalgamate the other return types
introduced into PHP 7. Functions declared with `void` as their return type
can either omit their `return` statement altogether, or use an empty return
statement. `null` is not a valid return value for a void function.

```php
function swap(&$left, &$right) : void
{
    if ($left === $right) {
        return;
    }

    $tmp = $left;
    $left = $right;
    $right = $tmp;
}

$a = 1;
$b = 2;
var_dump(swap($a, $b), $a, $b); // null, int(2), int(1)
```

Attempting to use a void function's return value simply evaluates to `null`,
with no warnings are emitted. The reason for this is because warnings would
implicate the use of generic higher order functions.

RFC: [Void Return Type](https://wiki.php.net/rfc/void_return_type)

### Symmetric Array Destructuring on Assignment

The shorthand array syntax (`[]`) may now be used to destructure arrays for
assignments (including within `foreach`). This pattern matching ability enables
for values to be more easily extracted from arrays.

```php
$pdo = new PDO('mysql:host=...;dbname=...', '...', '...');
$stmt = $pdo->query('SELECT id, val FROM ...');

while (['id' => $id, 'val' => $val] = $stmt->fetch(PDO::FETCH_ASSOC)) {
    // logic here with $id and $val
}
```

RFC: [Square bracket syntax for array destructuring assignment](https://wiki.php.net/rfc/short_list_syntax)

### Class Constant Visibility

Support for specifying the visibility of class constants has been added.

```php
class Something
{
    const PUBLIC_CONST_A = 1;
    public const PUBLIC_CONST_B = 2;
    protected const PROTECTED_CONST = 3;
    private const PRIVATE_CONST = 4;
}
```

RFC: [Class Constant Visibility](https://wiki.php.net/rfc/class_const_visibility)

### `iterable` Pseudo-Type

A new pseudo-type (similar to `callable`) - `iterable` - has been introduced.
It may be used in parameter and return types, where it accepts either arrays or
objects that implement the `Traversable` interface. With respect to subtyping,
parameter types of child classes may narrow a parent's `iterable` type to
either `array` or `object` (provided `object` implements `Traversable`). With
return types, child classes may widen a parent's return type of `array` or
`object` to `iterable`.

This type will typically be used for type checking parameter or return values
that will be used either in a `foreach` loop or `yield from` statement.

```php
function iterator(iterable $iter)
{
    foreach ($iter as $i) {
        //
    }
}
```

A new function, `is_iterable` has also been introduced to match the other `is_`
functions. It effectively performs the following check:
`is_array($var) || $var instanceof Traversable`.

RFC: [Iterable](https://wiki.php.net/rfc/iterable)

### Multi Catch Exception Handling

Multiple exceptions per catch block may now be specified using the pipe
character (`|`). This is useful for when different exceptions from different
class hierarchies are handled the same.

```php
try {
    // some code
} catch (AnException | AnotherException $e) {
    // handle exceptions
}
```

RFC: [Catching Multiple Exception Types](https://wiki.php.net/rfc/multiple-catch)

### Support for Keys in `list()`

The `list()` language construct now enables for keys to be specified inside of
it. This means that it can now destructure any type of arrays (like with the
new [symmetric array destructuring](#symmetric-array-destructuring-on-assignment)
syntax).

```php
$pdo = new PDO('mysql:host=...;dbname=...', '...', '...');
$stmt = $pdo->query('SELECT id, val FROM ...');

while (list('id' => $id, 'val' => $val) = $stmt->fetch(PDO::FETCH_ASSOC)) {
    // logic here with $id and $val
}
```

RFC: [Allow specifying keys in list()](https://wiki.php.net/rfc/list_keys)

### Support for Negative String Offsets

Ubiquituous support for negative string offsets has been added. This affects [a
number of built-in functions](https://wiki.php.net/rfc/negative-string-offsets#in_built-in_functions),
as well as the array dereferencing operator (`[]`).

```php
var_dump("abcdef"[-2]); // string (1)"e"
var_dump(strpos("aabbcc", "b", -3)); // int(3)
```

RFC: [Generalize support of negative string offsets](https://wiki.php.net/rfc/negative-string-offsets)

### Support for AEAD in ext/openssl

Support for AEAD (modes GCM and CCM) have been added by extending the
[openssl_encrypt](http://php.net/openssl_encrypt) and
[openssl_decrypt](http://php.net/openssl_decrypt) functions with additional
parameters.

RFC: [OpenSSL AEAD support](https://wiki.php.net/rfc/openssl_aead)

### Convert callables to `Closure`s with `Closure::fromCallable()`

A new static method has been introduced to the `Closure` class to allow for
callables to be easily converted into `Closure` objects. This is because
`Closure`s are more performant to invoke than other callables, and allow for
out-of-scope methods to be executed in other contexts.

```php
class Test
{
    public function exposeFunction()
    {
        return Closure::fromCallable([$this, 'privateFunction']);
    }

    private function privateFunction($param)
    {
        return $param;
    }
}

$privFunc = (new Test)->exposeFunction();
$privFunc('some value');
```

RFC: [Closure from callable function](https://wiki.php.net/rfc/closurefromcallable)

### Asynchronous Signal Handling with `pcntl_async_signals()`

Prior to PHP 7.1, PCNTL signal handling could be done synchronously via
[`pcntl_signal_dispatch`](http://php.net/pcntl_signal_dispatch) or
asynchronously via [ticks](http://php.net/manual/en/control-structures.declare.php#control-structures.declare.ticks).
Now, a new function - `pcntl_async_signals([bool $async = false])` - has been
introduced to switch asynchronous signal handling either on or off, without
having to use ticks (which introduce a lot of overhead).

```php
pcntl_async_signals(true); // turn on async signals

pcntl_signal(SIGHUP,  function($sig) {
    echo "SIGHUP\n";
});

posix_kill(posix_getpid(), SIGHUP);
```

RFC: [Asynchronous Signal Handling (without TICKs)](https://wiki.php.net/rfc/async_signals)

### Additions to ext/curl

#### Support for HTTP/2 Server Push

Support for server push has been added to the CURL extension (requires version
7.46 and above). This can be leveraged through the `curl_multi_setopt` function
with the new `CURLMOPT_PUSHFUNCTION` constant. The constants `CURL_PUST_OK` and
`CURL_PUSH_DENY` have also been added so that the execution of the server push
callback can either be approved or denied.

RFC: [ext/curl HTTP/2 Server Push Support](https://wiki.php.net/rfc/curl_http2_push)

#### Better Error Retrieval

Three new functions have been introduced to enable for errors related to multi
and share handles to be retrieved.

```php
int curl_multi_errno(resource $mh);
int curl_share_errno(resource $rh);
string curl_share_strerror(int $errno);
```

RFC: [Add curl_multi_errno(), curl_share_errno() and curl_share_strerror()](https://wiki.php.net/rfc/new-curl-error-functions)


## Changes

### Apprise on Arithmetic with Invalid Strings

New `E_WARNING` and `E_NOTICE` errors have been introduced when invalid strings
are coerced using the arithmetic operators (`+` `-` `*` `/` `**` `%` `<<` `>>`
`|` `&` `^`). An `E_NOTICE` is emitted when the string can be partially
converted to a sensical numeric, and an `E_WARNING` is emitted when the string
cannot be converted at all to a sensical numeric.

```php
'1b' + 'something';

// results in a notice for '1b' and a warning for 'something':
// Notice: A non well formed numeric value encountered in %s on line %d
// Warning: A non-numeric value encountered in %s on line %d
```

RFC: [Warn about invalid strings in arithmetic](https://wiki.php.net/rfc/invalid_strings_in_arithmetic)

### Deprecation of ext/mcrypt

The mcrypt extension has been abandonware for quite some time now (nearly a
decade). It was also fairly complex to use, causing many to implement it
incorrectly (and subsequently, insecurely). It has therefore been deprecated in
favour of [OpenSSL](http://php.net/openssl), where it will be removed from the
core and into PECL in PHP 7.2.

RFC: [Deprecate (then Remove) Mcrypt](https://wiki.php.net/rfc/mcrypt-viking-funeral)

### Deprecation of `mb_ereg(i)_replace()` Eval Option

The `e` pattern modifier has been deprecated for the `mb_ereg_replace()` and
`mb_eregi_replace()` functions. This was already done in PHP 5.5 for
`preg_replace()`, but it was overlooked for the `mb_ereg` functions.

RFC: [Deprecate mb_ereg_replace eval option](https://wiki.php.net/rfc/deprecate_mb_ereg_replace_eval_option)

### Warn on Octal Overflow

Previously, 3 octit octals could overflow without warning the programmer. Now,
an `E_WARNING` will be emitted on such overflows (with the previous overflow
behaviour remaining the same).

```php
var_dump("\500");

// Warning: Octal escape sequence overflow \500 is greater than \377 in %s on line %d
// string(1) "@"
```

### Forbid Dynamic Calls to Scope Introspection Functions

Dynamic calls for certain functions have been forbidden. These functions either
inspect or modify another scope, and present with them ambiguous and unreliable
behaviour. The functions are as follows:
```
assert() - with a string as the first argument
compact()
extract()
func_get_args()
func_get_arg()
func_num_args()
get_defined_vars()
mb_parse_str() - with one arg
parse_str() - with one arg
```

For example, the following will not work:
```php
(function () {
    'func_num_args'();
})();

// Warning: Cannot call func_num_args() dynamically in %s on line %d
```

RFC: [https://wiki.php.net/rfc/forbid_dynamic_scope_introspection](Forbid dynamic calls to scope introspection functions)

### Throw on Passing too few Function Arguments

An `Error` exception will now be thrown when a *user-defined function* (not an
*internal function*) is passed too few arguments. This new behaviour aims to
help expose potentially buggy code.

```php
function test($param){}
test(); // Uncaught Error: Too few arguments to function test(), 0 passed in %s on line %d and exactly 1 expected in %s:%d
```

RFC: [Replace "Missing argument" warning with "Too few arguments" exception](https://wiki.php.net/rfc/too_few_args)

### Inconsistency Fixes to `$this`

Whilst `$this` is considered a special variable in PHP, it lacked proper checks
to ensure it wasn't used as a variable name or reassigned. This has now been
rectified to ensure that `$this` cannot be a user-defined variable, reassigned
to a different value, or be globalised.

RFC: [Fix inconsistent behavior of $this variable](https://wiki.php.net/rfc/this_var)

### Session ID Generation without Hashing

Session IDs will no longer be hashed upon generation. This was an unnecessary
operation that slowed down ID generation. With this change brings about the
removal of the following four ini settings:
* [session.entropy_file](http://php.net/manual/en/session.configuration.php#ini.session.entropy-file)
* [session.entropy_length](http://php.net/manual/en/session.configuration.php#ini.session.entropy-length)
* [session.hash_function](http://php.net/manual/en/session.configuration.php#ini.session.hash-function)
* [session.hash_bits_per_character](http://php.net/manual/en/session.configuration.php#ini.session.hash-bits-per-character)

And the addition of the following two ini settings:
* **session.sid_length** - defines the length of the session ID, defaulting to
  32 characters for backwards compatibility)
* **session.sid_bits_per_character** - defines the number of bits to be stored
  per character (i.e. increases the range of characters that can be used in the
  session ID), defaulting to 4 for backwards compatibility

RFC: [Session ID without hashing](https://wiki.php.net/rfc/session-id-without-hashing)

### RNG Fixes and Changes

#### Fix `mt_rand()` Implementation

`mt_rand()` will now default to using the fixed version of the Mersenne Twister
algorithm, with the ability to preserve the old (incorrect) implementation via
an additional optional second parameter to [`mt_srand()`](http://php.net/mt_srand).

RFC: [RNG fixes and changes: Fix mt_rand() implementation](https://wiki.php.net/rfc/rng_fixes#fix_mt_rand_implementation)

#### Alias `rand()` to `mt_rand()`

`rand()` has little use nowadays given how weak its RNG is, and so it will now
be aliased to `mt_rand()` (since dropping the function all together would cause
too much of a BC break). This will improve the quality of output, including no
longer having system-dependent output, but rather seed-dependent output.

RFC: [RNG fixes and changes: Alias rand() to mt_rand()](https://wiki.php.net/rfc/rng_fixes#alias_rand_to_mt_rand)

([The RFC](https://wiki.php.net/rfc/rng_fixes) has two further changes, but
they will have no practical impact to PHP developers.)
