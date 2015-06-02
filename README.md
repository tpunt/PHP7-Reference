# PHP 7

PHP 7 has been slated for release [in November of this year](https://wiki.php.net/rfc/php7timeline) (2015). It comes with a number of new features, changes, and backwards compatibility breakages that are outlined below.


**Features**
* [Combined Comparison Operator (<=>)](#cco)
* Null Coalesce Operator (??)
* Scalar Type Hints
* Return Type Declarations
* Unicode Codepoint Escape Syntax
* Closure `call()` Method
* Filtered `unserialize()`
* `IntlChar` Class
* `session_start()` Options
* Expectations
* Group `use` Declarations
* Generator Return Expressions
* Integer Division with `intdiv()`
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

### Combined Comparison Operator (<=>)
The combined comparison operator (or spaceship operator) is a short-hand notation for performing three-way comparisons from two operands. It has an integer return value that can be either:

* 1 (if the left-hand operand is greater than the right-hand operand)
* 0 (if both operands are equal)
* -1 (if the right-hand operand is greater than the left-hand operand)

The operator has the same precedence as the equality operators (`==`, `!=`, `===`, `!==`) and has the exact same behaviour as the other loose comparison operators (<, >=, etc). It is also non-associative like them too, so chaining of the operands (like `1 <=> 2 <=> 3`) is not allowed.

```PHP
// compares strings lexically
var_dump('PHP' <=> 'Node'); // int(1)

// compares numbers by size
var_dump(123 <=> 456); // int(-1)

// compares corresponding array elements with one-another
var_dump(['a', 'b'] <=> ['a', 'b']); // int(0)
```

Objects are not comparable, and so using them as oparands with this operator will result in undefined behaviour.

RFC: [Combined Comparison Operator](https://wiki.php.net/rfc/combined-comparison-operator]

### Null Coalesce Operator (??)
The null coalesce operator (or isset ternary operator) is a short-hand notation for performing `isset()` checks in the ternary operator. This is a common thing to do in applications, and so a new syntax has been introduced for this exact purpose.

```PHP
// old style
$route = isset($_GET['route']) ? $_GET['route'] : 'index';

// new style
$route = $_GET['route'] ?? 'index';
```

RFC: [Null Coalesce Operator](https://wiki.php.net/rfc/isset_ternary)
