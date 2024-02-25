# stdnum #

A library that provides commonly used numerical representations. All of these are derived from `number` so they are all serializable and are all stringifiable in the same way.

Provides pre-sized types similar to those from stdint.h:

* sint8  - Signed integer, 8 bits (1 byte)
* sint16 - Signed integer, 16 bits (2 byte)
* sint32 - Signed integer, 32 bits (4 byte)
* sint64 - Signed integer, 64 bits (8 byte)
* uint8  - Unsigned integer, 8 bits (1 byte)
* uint16 - Unsigned integer, 16 bits (2 byte)
* uint32 - Unsigned integer, 32 bits (4 byte)
* uint64 - Unsigned integer, 64 bits (8 byte)


## varint ##

Aliases: varnum, vlq

Represents variably sized quantities such as:

* [Variable-length quantity](https://en.wikipedia.org/wiki/Variable-length_quantity)
* [LEB128](https://en.wikipedia.org/wiki/LEB128)

### Keys ###

Note that *sign*, inherited from `number`, effectively does nothing in this struct. TODO: It may not be available or it may reflect the existence of a sign bit in *format*.

* *format* - Describes the format of the octets (or if they even are octets).
  - Default value is **cnnnnnnn**
  - As a string, describes the format for every octet.
    + Each non-whitespace character represents a bit, MSB first.
    + The length of the string represents of length of each chunk. Normally 8.
    + The characters can be one of the following, case-insensitive:
      * **n** - Part of the number.
        - If the digits are not in order of MSB to LSB, you can used numbers instead of n.
        - The numbers are only considered for order, their value is otherwise unused.
        - If a number is used multiple times, they're ordered as if those were MSB to LSB.
        - **n** itself is ordered after numbers.
        - If you really need more than 11 ordering markers, you may use `$(e000)` up to `$(f8ff)` (Unicode Private Use Area) as the next ordering after **n**.
      * **c** - Continuation bit.
      * **s** - Sign bit. 0 = positive, 1 = negative.
        - If there is no sign bit, it's assumed to be positive. However, *transform* can still return a negative number. If you want to restrict the possible number space to only positive integers, set *min* to 0.
      * **p** - Sign bit but 0 = negative, 1 = positive.
  - As a specialized struct:
    + *first* (string) - Format for the first octet, defaults to same as *other*.
    + *last* (string) - Format for the last octet, defaults to same as *other*.
    + *other* (string) - Format for any other octet.
* *endian* (string) - **big** (default) if the MSBs are in the first octet, **little** if reversed.
* *transform* - Mathematically transforms the number decoded into the expected number.
  - **signed** - If this key is set to **signed**, this performs the usual [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) transformation.
  - **zigzag** - Interprets it as an alternating sequence of negative and positive numbers such that `0b0 = 0`, `0b1` = -1, `0b10 = 1`, `0b11` = -2, etc. See [here](https://en.wikipedia.org/wiki/Variable-length_quantity#Zigzag_encoding) for more info.
    + In the following fomulas, **n** is the number concatenated from processing *format*, **k** is the total number of bits, and **x** is the final, unpacked value.
    + Zigzag encoding formula: `(x << 1) ^ (x >> k - 1) = n`
    + Zigzag decoding formula: `(n >> 1) ^ (-(n & 1)) = x`
  - As a `math` type, it represents the decoding direction. (packed -> *format* -> *transform* -> unpacked)
    + It's supplied the variable **n** which represents the result of *format* and **k** which represents the total number of bits.
    + Example for making the result [bijective](https://en.wikipedia.org/wiki/Bijective_numeration) using **cnnnnnnn** format: `n + (1 << (k - 7))`

### Pre-defined forms ###

```rpl
varint leb128 {
    format: cnnnnnnn
    endian: little
    transform: signed
}

varint uleb128 {
    format: cnnnnnnn
    endian: little
}
```

### Other examples ###

```rpl
varint UnrealSignedVLQ {
    format {
        first: csnnnnnn
        other: cnnnnnnn
    }
    endian: big
}

varint GoogleProtobuf {
    format: cnnnnnnn
    endian: little
    transform: zigzag
}

varint GitVLQ {
    format: cnnnnnnn
    # TODO: endian
    transform: math("n + (1 << (k - 7))")
}
```


## fraction ##

Represents a lossless floating point number by storing the numerator and denominator as separate bigints.

The basic value is the float value, which may be lossy depending on implementation.

### Keys ###

* *numerator* (number) - The top part of the fraction.
* *denominator* (number) - The bottom part of the fraction.
* *ratio* (list of numbers) - Effectively \[*numerator*, *denominator* - *numerator*\] (reflects as appropriate). If this was not explicitly set, it uses the reduced forms, eg. 5/10 becomes 1:1
  - If this contains more than two numbers, the *numerator* is the first element in the list, and the *denominator* is the sum of the list.
  - Since `math` is a number, it's possible to specify formulas in the various fields. *ratio* may work strangely in this case.


## ieee754 ##

Alias: iec559

Represents [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) standard floating point numbers.

The basic value is the floating point value, ideally represented in memory in the same level of precision as requested.

### Keys ###

* *size* (size) - This, along with the *base* determines the format only if it's an officially supported size. Those sizes are 4, 8, and 16 for both, and also 2 and 32 for base 2.
* *radix* (number) - Official values are 2 or 10.
* TODO?: a precision p;
* TODO?: an exponent range from emin to emax, with emin = 1 − emax for all IEEE 754 formats.
* *precision* (string) - Forces *radix* to 2 and sets size to an official size.
  - Possible values are: **half** (16 bits), **single**  (32 bits), **double** (64 bits), **quadruple** (128 bits), or **octuple** (256 bits).
  - If *radix* is not 2, this key cannot be set and has no value.
* *exponent* (number) - The exponent component of the number. Set it like `exponent: number { size: 8 bits }` to control its size.
* *fraction* (number) - The fraction component of the number. Set its size similarly to *exponent*.
* *combination*, *coefficient* or *significand* TODO
* *figures* (number) - Number of [significant figures](https://en.wikipedia.org/wiki/Significant_figures). Primarily for stringification, but may also be used elsewhere.

Special note that *sign* does not refer to the IEEE 754 sign bit. Rather, it is always **signed**. Additionally, the sign bit is always a size of 1.

### Pre-defined forms ###

TODO: binary16, binary32, binary64, binary128, binary256, decimal32, decimal64, decimal128


## vector ##

TODO: what is this/is it actually useful

* AltiVec/Velocity Engine/VMX: [1](http://mirror.informatimago.com/next/developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/2rt_powerpc_abi/chapter_9_section_2.html#//apple_ref/doc/uid/20001297/BCIGAGJD), [2](https://en.wikipedia.org/wiki/AltiVec)
* [SSE](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions)
