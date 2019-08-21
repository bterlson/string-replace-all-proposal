# `String.prototype.replaceAll` proposal

## Status

Champion: Mathias Bynens (Google, @mathiasbynens).

This proposal is at stage 2 of [the TC39 process](https://tc39.github.io/process-document/).

## Motivation

Currently there is no way to replace all instances of a substring in a string without use of a global regexp.
`String.prototype.replace` only affects the first occurrence when used with a string argument. There is a lot of evidence that developers are trying to do this in JS — see the [StackOverflow question](https://stackoverflow.com/q/1144783/96656) with thousands of votes.

Currently the most common way of achieving this is to use a global regexp.

```js
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.replace(/\+/g, ' ');
```

This approach has the downside of requiring special RegExp characters to be escaped — note the escaped `'+'`.

An alternate solution is to combine `String#split` with `Array#join`:

```js
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.split('+').join(' ');
```

This approach avoids any escaping but comes with the overhead of splitting the string into an array of parts only to glue it back together.

## Proposed solution

We propose the addition of a new method to the String prototype - `replaceAll`. This would give developers a straight-forward way to accomplish this common, basic operation.

```js
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.replaceAll('+', ' ');
```

It also removes the need to escape special regexp characters (note the unescaped `'+'`).

## High-level API

The proposed signature is the same as the existing `String.prototype.replace` method:

```
String.prototype.replaceAll(searchValue, replaceValue)
```

Per the current TC39 consensus, `String.prototype.replaceAll` behaves identically to `String.prototype.replace` in all cases, **except** for the case where `searchValue` is a string.

In that case, `String.prototype.replace` only replaces a single occurrence of the `searchValue`, whereas `String.prototype.replaceAll` replaces *all* occurrences of the `searchValue` (as if `.split(searchValue).join(replaceValue)` or a global & properly-escaped regular expression had been used).

Notably, `String.prototype.replaceAll` behaves just like `String.prototype.replace` if `searchValue` is a regular expression, [including if it's a non-global regular expression](https://github.com/tc39/proposal-string-replaceall/issues/16).

## Comparison to other languages

* Java has [`replace`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#replace-java.lang.CharSequence-java.lang.CharSequence-), accepting a `CharSequence` and replacing all occurrences. Java also has [`replaceAll`](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#replaceAll-java.lang.String-java.lang.String-) which accepts a regex as the search term (requiring the user to escape special regex characters), again replacing all occurrences by default.
* Python [`replace`](https://www.tutorialspoint.com/python/string_replace.htm) replaces all occurrences, but accepts an optional param to limit the number of replacements.
* PHP has [`str_replace`](http://php.net/manual/en/function.str-replace.php) which has an optional limit parameter like python.
* Ruby has [`gsub`](https://ruby-doc.org/core/String.html#method-i-gsub), accepting a regexp or string, but also accepts a callback block and a hash of match -> replacement pairs.

## FAQ

### What are the main benefits?

A simplified API for this common use-case that does not require RegExp knowledge. A way to global-replace strings without having to escape RegExp syntax characters. Possibly improved optimization potential on the VM side.

### What about adding a `limit` parameter to `replace` instead?

A: This is an awkward interface — because the default limit is 1, the user would have to know how many occurrences already exist, or use something like Infinity.

### What happens if `searchValue` is the empty string?

`String.prototype.replaceAll` follows the precedent set by `String.prototype.replace`, and returns the input string with the replacement value spliced in between every UCS-2/UTF-16 code unit.

```js
'x'.replace('', '_');
// → '_x'
'xxx'.replace(/(?:)/g, '_');
// → '_x_x_x_'
'xxx'.replaceAll('', '_');
// → '_x_x_x_'
```

## TC39 meeting notes

- [November 2017](https://tc39.github.io/tc39-notes/2017-11_nov-28.html#10ih-stringprototypereplaceall-for-stage-1)
- [March 2019](https://github.com/rwaldron/tc39-notes/blob/master/meetings/2019-03/mar-26.md#stringprototypereplaceall-for-stage-2)
- [July 2019](https://github.com/tc39/tc39-notes/blob/master/meetings/2019-07/july-24.md#stringprototypereplaceall)

## Specification

- [Ecmarkup source](https://github.com/tc39/proposal-string-replaceall/blob/master/spec.html)
- [HTML version](https://tc39.github.io/proposal-string-replaceall/)

## Implementations

- none yet
