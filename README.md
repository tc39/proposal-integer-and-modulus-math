# Modulus and Additional Integer Math
Updated July 9, 2023

## Status

Champion: Peter Hoddie (assisted by Dan Ehrenberg)

## Stage: 1

## Motivation
This proposal consists of two related extensions to the `Math` object: additional functions for integer math and true modulus for integers and floats.

This proposal adds functionality that missing from ECMAScript. It does so in way that is ergonomic for developers and allows for efficient implementation by engines across a wide range of hardware. While there is some overlap in functionality with WebAssembly and asm.js, this proposal is independent of those.

### Integer Math
Integer math operations are often more efficient than floating point math. This tends to be true even on CPUs with an FPU.

This proposal introduces additional static methods on `Math` for signed 32-bit integer values.

While ECMA-262 defines mathematical operations in terms of floating point numbers, engines and ECMAScript compilers may implement optimizations to use integers where there is no observable difference in the result.

Engines can infer some situations where integer optimizations are possible, but it is not always practical. For this reason, ES6 added [`Math.imul`](https://tc39.es/ecma262/#sec-math.imul) to allow source text to directly express a 32-bit signed integer multiply operation.

In addition to being more efficient, integer math operators can be more ergonomic than performing a floating point operation and then converting the result to an integer. The integer divide, integer modulus, integer remainder, and integer random are examples.

### Modulus
The `%` operator is often incorrectly referred to as the "modulo operator" but the actual operation is [remainder](https://tc39.es/ecma262/#sec-numeric-types-number-remainder):

Remainder and modulo operations are equivalent for positive inputs, but not negative inputs. [This article](https://rob.conery.io/2018/08/21/mod-and-remainder-are-not-the-same) describes the differences.

Brendan Eich [noted](https://twitter.com/BrendanEich/status/1295366640259874818):

> ...we still need to add mod (as distinct from C-like %) to JS.

This proposal introduces additional static methods on `Math` for the modulus operation on `Number` and signed 32-bit integer values.

The MDN page for [Math Remainder](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Remainder) has a long paragraph that explains the difference between remainder and modulus, a formula for modulus, and a confusing note that begins "In JavaScript, the modulo operation (which doesn't have a dedicated operator)...".

## Use cases

- Engine optimizations
- Compiler optimizations
- Simplified script code for supported integer operations
- Script optimizations - [J5e](https://j5e.dev) (embedded robotics)

## Description

The following operations are from the original proposal:

- `Math.mod(x, y)` – IEEE 754 modulus
- `Math.idiv(x, y)` – Int32 division
- `Math.imod(x, y)` – Int32 modulus
- `Math.imuldiv(x, y, z)` – Int32 multiply and divide with 64-bit intermediate result -  (x * y) / z
- `Math.irem(x, y)` – Int32 remainder (integer equivalent of `%` operator)

For integer operations, input arguments are converted to integer values using [`ToInt32`](https://tc39.es/ecma262/#sec-toint32).

### Special case

32-bit signed integer division has one special case: dividing the value -2147483648 (0x80000000, smallest negative 32-bit integer value) by -1. The result cannot be expressed in a 32-bit signed integer (the largest positive 32-bit signed integer is +2147483647). This impacts the implementations of `Math.idiv`, `Math.imod`, `Math.irem`, and `Math.imuldiv`.

The behavior of this operation varies depending on the CPU architecture. Some architectures generate an exception while others have a defined behavior. The following return values, which matches what some CPU architectures generate, are proposed:

- `Math.idiv` - result is the value of `x` (-2147483648)
- `Math.imod` - result is `0`
- `Math.imuldiv` - result is `-2147483648`
- `Math.irem` - result is `0`

For architectures which already yield the proposed result, this special case adds no overhead; for those that do not, additional runtime checks are required.

### Add `Math.irandom()`

Generating a random integer is awkward in JavaScript. MDN provides two examples to generate random integer on the `Math.random()` page, including a warning about a common mistake. Generating a random floating point value and then converting it to an integer has needless overhead.

 - `Math.irandom()` – Int32 value from 0 to 2147483647 (inclusive)
 - `Math.irandom(x)` – Int32 value between `0` and `x - 1` (inclusive)
 - `Math.irandom(x, y)` – Int32 value between `x` and `y - 1` (inclusive). This matches the behavior of `getRandomInt` example on MDN.

As with `Math.random()` there is **no** attempt here to provide cryptographically secure random numbers.

`Math.irandom` is convenient for accessing a random element from an `Array`. Here `y` will usually be `undefined` as non-integer array indices are converted to strings. On the other hand, `z` is always one of the array elements.

```js
let x = ["one", "two", "three"];
let y = x[Math.random() * x.length];
let z = x[Math.irandom(x.length)];
```

### Remove

The original version of this proposal included one operation which returned multiple values in an object. This is difficult to optimize and rare. Consequently, it is no longer part of this proposal.

- `Math.idivmod(x, y)` – Int32 division with modulus, returns `[division result, modulus result]`

## Comparison

Most languages provide some subset of these integer and modulo operations. This section contains examples from Python and Ruby.

### Python

- `math.remainder` and `math.fmod`

> ...fmod() is generally preferred when working with floats, while Python’s x % y is preferred when working with integers.

But... `%` is defined as "remainder"

- //

> floored quotient... Also referred to as integer division

- `divmod(x, y)`

> [returns] the pair (x // y, x % y)

### Ruby

- `x.divmod(y)` – returns `[quotient, modulus]`
- `x.modulo(y)` – `x - y * (x / y).floor`
- `x.remainder(y)` – `x - y * (x / y).truncate`.

## Implementations

### Polyfill/transpiler implementations

- (none)

### Native implementations

- [XS](https://github.com/Moddable-OpenSource/moddable/blob/a7fc383c87d4d4d90e4d80af86994bcc899dcaf6/xs/sources/xsMath.c#L296-L405) - current implementation

## Q&A

**Q**: Why not use operators instead of static methods?

**A**: This proposal follows the approach established by `Math.imul`. If ECMAScript supports operator overloading in the future, developers may apply operators here.

**Q**: Why does this need to be built-in, instead of being implemented in ECMAScript?

**A**: These static methods allow engines to optimize in ways that are impractical with equivalent functions implemented in ECMAScript.

**Q**: Do these static methods accept `BigInt` arguments?

**A**: No, to be consistent with the other static methods on `Math`, including `Math.imul`. There is no fundamental objection to supporting `BigInt` where it makes sense should that be the committee's preference.
