# frazerchard.github.io

Cairo is a programming language created by [Starkware](https://www.cairo-lang.org/), which can be used to develop scalable, layer two blockchain applications.
For complete and accurate information on how to use the language, see the [official documentation](https://www.cairo-lang.org/docs/).

This site is meant to provide some basic examples on some different functionalities of the Cairo language and some of the most common security concerns that are prevalent to this language and ecosystem.

* TOC
{:toc}

## Hello World

```javascript
// Declare this file as a StarkNet contract.
%lang starknet
// Range check will ensure numbers stay within the felt range
%builtins range_check

// Alphabet substituation cipher for each letter.
// a = 01, b = 02, etc.
const hello = 10000805121215;  // 08, 05, 12, 12, 15
const world = 10002315181204;  // 23, 15, 18, 12, 04.

// View decorator exposes the hello_world function for reading
// Returns two constants with the type felt
@view
func hello_world() -> (num_1: felt, num_2: felt) {
    return (hello, world);
}
```
