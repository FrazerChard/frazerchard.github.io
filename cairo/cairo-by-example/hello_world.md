---
layout: page
title:  "Hello World on StarkNet"
permalink: /cairo/cairo-by-example/hello_world.md
toc: false
---

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
