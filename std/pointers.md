# Pointers #

Pointers are addresses to a space in the ROM (for example) which contains some data you want to get at. It's essentially what you put in the *base* key, but in order to make an MPRL file more dynamic, we can instead extract the address from where it's mentioned in the ROM. For example:

```mprl
static Level01 {
    data Metadata {
        base: $0240dd
        xtiles: [number, 3]
        xmap: [number, 3]
        xwidth: [number, 1]
        # ...
    }

    tilemap Tiles {
        base: @{Metadata}.xtiles  # <-- using xtiles as a pointer
        # ...
    }

    # ...
}
```

But often times with banking, especially in code blocks, the address we want to extract can be split up, or the bank might be fixed, or maybe it came before the rest of the number (despite that rest being little endian).

For those cases, we need a struct type that's more robust than `number`, which we'll call `ptr` (this may be an alias of `pointer`).


## Pointers as a number context ##

Consider `ptr` as being a context in which each entry represents some number of bytes in little endian order. The size of the `ptr` struct is the sum of its entries' sizes and the value is its entries concatenated and read as a little endian number (this could be configurable).

It could be a binary context that can just be read as a number.

There are two potential forms, and it's possible to allow both:

```mprl
# With substructs
ptr {
    base: $00abcd
    number { base: 5, size: 2}
    number { base: 1, size: 1}
}

# With a key
ptr {
    base: $00abcd
    bytes: [5:2, 1:1]
}
```

If we had the data at $00abcd as `50 02 34 55 a1 9d 00` then the numeric value of these `ptr`s would be $02009d

In the case that endianness should be specifiable, it must be the same as `number`'s *endian* key, so that the child `number`s may inherit it.


## Future considerations ##

Eventually it would be ideal to be able to parse assembler code and extract the parts of the pointer from the code as variable elements such as:

```mprl
ptr {
    base: $00abcd
    extract: asm(``
      mov xi, {bank}
      inc a
      mov x, {address}
    ``)
    bytes: [address, bank]
}
```

This could theoretically make use of refstrings, though that could end up quite wordy. In order to support this feature, it's likely that both `asm` and some sort of general-purpose feature extraction system would need to be defined.
