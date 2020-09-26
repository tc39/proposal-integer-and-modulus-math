# Modulus and Additional Integer Math
Updated September 1, 2020

## Status

Champion: Peter Hoddie (assisted by Dan Ehrenberg)

## Stage: 0

## Motivation
This proposal consists of two related extensions to the `Math` object: integer math and true modulus.

### Integer Math
Integer math operations are often more efficient than floating point math. This tends to be true even in hardware with an FPU.

While ECMA-262 defines mathematical operations in terms of floating point numbers, some engines (XS) and ECMAScript compilers (Emscripten) implement optimizations to use integers where the result is unobservable.

Engines can infer some situations where integer optimizations are possible, but it is not always practical. For this reason, ES6 added [`Math.imul`](https://tc39.es/ecma262/#sec-math.imul) to allow source text to directly express a 32-bit signed integer multiply operation.

This proposal introduces additional static methods on `Math` for signed 32-bit integer values.

### Modulus
The `%` operator is often incorrectly referred to as the "modulo operator" but the actual operation is [remainder](https://tc39.es/ecma262/#sec-numeric-types-number-remainder):

Remainder and modulo operations are equivalent for positive inputs, but not negative values. [This article](https://rob.conery.io/2018/08/21/mod-and-remainder-are-not-the-same) describes the differences.

Brendan Eich [recently noted](https://twitter.com/BrendanEich/status/1295366640259874818):

> ...we still need to add mod (as distinct from C-like %) to JS.

This proposal introduces additional static methods on `Math` for the modulus operation on `Number` and signed 32-bit integer values.

## Use cases

- Engine optimizations (targets without an FPU, in particular)
- Compiler optimizations
- Application optimizations - [J5e](https://j5e.dev) (embedded robotics)

## Description

- `Math.mod(x, y)` – IEEE 754 modulus
- `Math.idiv(x, y)` – Int32 division
- `Math.imod(x, y)` – Int32 modulus
- `Math.idivmod(x, y)` – Int32 division with modulus, returns `[division result, modulus result]`
- `Math.imuldiv(x, y, z)` – Int32 multiply and divide with 64-bit intermediate result -  `(x * y) / z`
- `Math.irem(x, y)` – Int32 remainder

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

- [XS](https://github.com/Moddable-OpenSource/moddable/blob/e9b1ea4a4f09782b9af3526063968657483bfbdd/xs/sources/xsMath.c#L296-L382) - initial implementation

## Q&A

**Q**: Why not use operators instead of static methods?

**A**: This proposal follows the approach established by `Math.imul`. If ECMAScript supports operator overloading in the future, developers may apply operators here.

**Q**: Why does this need to be built-in, instead of being implemented in ECMAScript?

**A**: These static methods allow engines to optimize in ways that are impractical with equivalent functions implemented in ECMAScript.

**Q**: Do these static methods accept `BigInt` arguments?

**A**: No, to be consistent with the other static methods on `Math`, including `Math.imul`. There is no fundamental objection to supporting `BigInt` where it makes sense should that be the committee's preference.
