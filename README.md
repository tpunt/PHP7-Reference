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
* Fixes to `foreach()`s Behaviour
* Fixes to `list()`s Behaviour
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
