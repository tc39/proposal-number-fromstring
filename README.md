# ECMAScript proposal: `{BigInt,Number}.fromString`

## Status

This proposal is at stage 1 of [the TC39 process](https://tc39.github.io/process-document/).

## Background

[The `BigInt` proposal](https://github.com/tc39/proposal-bigint) initially included a static `BigInt.parseInt` method. After [some discussion](https://github.com/tc39/proposal-bigint/issues/86), it was [removed](https://github.com/tc39/proposal-bigint/commit/30b5594c30bdf7973323d61504017933673df283) in favor of a separate proposal to add a static `fromString` method to both `BigInt` and `Number`. This is that proposal.

## Motivation

`{BigInt,Number}.prototype.toString(radix)` enables converting a numeric value into a string representation of that value. For `BigInt`s specifically, there is currently no built-in way to do the inverse, i.e. to turn a string representation of a `BigInt` with a given radix back into a `BigInt`.

For `Number` values, there is [`parseInt(string, radix = 10)`](https://tc39.github.io/ecma262/#sec-parseint-string-radix) and [`Number.parseInt(string, radix = 10)`](https://tc39.github.io/ecma262/#sec-number.parseint), but its behavior is suboptimal:

- It returns `NaN` instead of throwing a `SyntaxError` exception when `string` does not represent a number.
- It returns `NaN` instead of throwing a `RangeError` exception when `radix` is not valid (i.e. `radix !== 0 && radix < 2` or `radix > 36`).
- It accepts radix `0`, treating it as `10` instead, which does not make sense.
- It ignores leading whitespace and trailing non-digit characters.
- It supports hexadecimal integer literal prefixes `0x` and `0X` but lacks support for octal integer literal prefixes `0o` and `0O` or binary integer literal prefixes `0b` and `0B`, which is inconsistent.
- The fact that `parseInt` has some level of support for integer literal prefixes means that it’s not a clear counterpart to `toString`.

## Proposed solution

We propose extending both `BigInt` and `Number` with a new static `fromString(string, radix = 10)` method which acts as the inverse of `{BigInt,Number}.prototype.toString(radix = 10)`. It accepts only strings that can be produced by `{BigInt,Number}.prototype.toString(radix = 10)`, and throws an exception for any other input.

## High-level API

```js
Number.fromString('42');
// → 42
Number.fromString('42', 10);
// → 42

BigInt.fromString('42');
// → 42n
BigInt.fromString('42', 10);
// → 42n
```

## Illustrative examples

The following examples use `Number.fromString`. The semantics for `BigInt.fromString` are identical except it returns a `BigInt` rather than a `Number`.

Unlike `parseInt`, `fromString` intentionally lacks special handling for integer literal prefixes.

```js
Number.parseInt('0xc0ffee');
// → 12648430
Number.parseInt('0o755');
// → 0
Number.parseInt('0b00101010');
// → 0

Number.fromString('0xc0ffee');
// → SyntaxError
Number.fromString('0o755');
// → SyntaxError
Number.fromString('0b00101010');
// → SyntaxError

Number.fromString('C0FFEE', 16);
// → SyntaxError (toString produces lowercase digits)
Number.fromString('c0ffee', 16);
// → 12648430 === 0xc0ffee
Number.fromString('755', 8);
// → 493 === 0o755
Number.fromString('00101010', 2);
// → 42 === 0b00101010
```

Unlike `parseInt`, `fromString` throws a `SyntaxError` exception when `string` does not represent a number.

```js
Number.parseInt('');
// → NaN
Number.parseInt(' \n ');
// → NaN
Number.parseInt('x');
// → NaN

Number.fromString('');
// → SyntaxError
Number.fromString(' \n ');
// → SyntaxError
Number.fromString('x');
// → SyntaxError
```

Unlike `parseInt`, `fromString` throws a `RangeError` exception when `radix < 2` or `radix > 36`.

```js
Number.parseInt('1234', 0);
// → 1234
Number.parseInt('1234', 1);
// → NaN
Number.parseInt('1234', 37);
// → NaN

Number.fromString('1234', 0);
// → RangeError
Number.fromString('1234', 1);
// → RangeError
Number.fromString('1234', 37);
// → RangeError
```

Unlike `parseInt`, `fromString` throws a `TypeError` exception when `string` is not a string.

```js
Number.parseInt(true, 32);
// → 978894

Number.fromString(true, 32);
// → TypeError
```

### FAQ

#### What about legacy octal integers?

`fromString` intentionally lacks special handling for legacy octal integer literals, i.e. those without the explicit `0o` or `0O` prefix such as `010`. In other words, `Number.fromString('010')` throws a `SyntaxError` exception.

#### What about numeric separators?

`fromString` does not need to support [numeric separators](https://github.com/tc39/proposal-numeric-separator), as they cannot occur in `{BigInt,Number}.prototype.toString(radix)` output. `Number.fromString('1_000_000_000')` throws a `SyntaxError` exception.

#### Does `BigInt.fromString(string)` support the `n` suffix?

`BigInt.fromString` does not need to support the `n` suffix used for `BigInt` literals, as it doesn’t occur in `BigInt.prototype.toString(radix)` output. Furthermore, supporting it would introduce an ambiguity for radices where `n` is a valid digit: should `BigInt.fromString('1n', 32)` return `1` or `55`? With the current proposal, `BigInt.fromString('1n', 32)` returns `55`, and `BigInt.fromString('1n')` throws a `SyntaxError` exception.

## Specification

* [Ecmarkup source](https://github.com/mathiasbynens/proposal-number-fromstring/blob/master/spec.html)
* [HTML version](https://mathiasbynens.github.io/proposal-number-fromstring/)

## Implementations

* none yet
