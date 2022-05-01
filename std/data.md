# To data or not to data #

The data struct has been a thorn in my side particularly through this revamp process and yet here I am, ambivalent to whether or not it's included. Although it will (likely) not make it into the first releases of the new version, I'm keeping it in this folder because it was part of the original system.


## Problems I've had with data ##

Initially, what caused this rewrite, was that when `data` was used by something outside of the struct, it exported the data twice, once into each place. This was problematic for large values. This, inherently, was a problem of encapsulation: `data` structs only cared about `data` structs' references. Forcing it to care about others meant a change to the base struct class.

"But why should I change the base for one class? Why should I force lib authors to make any changes for one struct?"

The best answer, really, was to bake it into the core. *Whenever* a value is going to be exported to two places, the system should be aware of how to pick one; not just in the context of `data`.

As I've been designing the new version, however, I've run into other problems that have caused more scope creep. For instance, originally if you wanted a complex type, you define it in the root then reference it for the type field. But what if this is designed as an included file, and that type isn't used as a type? It would affect the structure of the root, then, so clearly we need something to ensure it never will.


## Living without data ##

A world without `data`, and no equivalent shorthand, looks tedious but not terrible. Essentially, since values are structs now, they can be used in-place as a much as a single key in a `data` struct would be:

```rpl
data Data {
    xlength: [number, 1]
    xstring: [string, @this.xlength]
}
```

Would be the same as:

```rpl
bin Data {
    number xlength { size: 1 }
    string xstring { length: @parent{xlength} }
}
```

The primary difference is in how "count" comes into play. `data` makes this simple, normal RPL does not:

```rpl
# v0.0.9
data Data {
    xcount: [number, 1]
    xnames: [@Name, @this.xcount]
}

format Name {
    xname: [string, 10]
}

# modern
data Data {
    xcount: [number, 1]
    xnames: [string, 10:@this.xcount]
}
```

And the equivalent:

```rpl
bin Data {
    number xcount { size: 1 }
    list xnames {
        length: @parent{xcount}
        type: string { size: 10 }
    }
}
```

Clearly it will continue to get wordier and wordier as the struct becomes more complex, but there is more flexibility and explicitness. Those are nice features for a programming language but less so for a notational language.

To reiterate, the problem is how much you have to type. `data` was intended to be a simple way to represent C-style structs but with some ability for dynamic lists; a very common use case in unpacking things like save data or archive formats.


## Needs ##

* Allows for quickly mocking out structured ctypes.
  - Even though it should be quick and simple to fit these needs, it should be possible to build out complex descriptions without excess complexity.
  - Preferably in a way that doesn't conflate supplying a list as a complex type with supplying the simple description.
* Allows for simple "X length list of Ys [of size Z]" style constructions.
* Allows for referencing some other serializeable structure defined in RPL to be used as a type.
* Allows for quickly overwriting decoding-related keys
* Fits in with the linking system properly (likely by generating structs)
  - Should prefer to never be a required export, but depends on implementation. (e.g. when using something in a way outside the scope of the `data` ideals, this requirement isn't applicable.)
* Able to export the whole structure in an editable structured data form like JSON.


## Alternatives ##

### C-style ###

A gross alternative would be to straight up parse C-style structs, considering that was the original goal. It would be able to be quickly picked up by ROM hackers who know how to program, but many don't, and "custom" syntaxes are just another complexity for such people.

However, if implemented, it would look something like this:

```rpl
data(``
    uint_8 length;
    char[length] string;
``)
```

This is limiting in terms of allowing both size and count, but generalizing ctypes for that (`TYPE_SIZE`) could work. However, this really doesn't allow one to do anything particularly complex, and cannot specify things like endianness or encoding. `char` also implies that it wouldn't be decoded and simply remain as (signed) characters, which is untrue for Imperial.

We could mitigate this by using RPL types instead of ctypes.

```rpl
data(``
    number(1) length;
    string(length) string;
``)
```

But it becomes very unlike C and basically just becomes some dumb limiting syntax.


### type type ###

In the spirit of the list-based construction of xkeys, there could be a `type` which takes a list basic of that form. This allows them to go anywhere, instead of just in `data` and by that eliminates the need for `data` at all.

```rpl
bin {
    length: type[number, 1]
    string: type[string, @this.length]
}
```

The advantage is that this is barely any more typing and it allows for specifying some other structure simply and sanely with no guesswork on the interpreter's part at the key level.

One difference is that this could no longer accept unnamed arguments such as **little** or **big** representing endianness but in the long run that's likely a good thing.

There are two primary options for setting these fields:

1. `type[number { endian: big }, 2]`
2. `type[number, 2, endian: big]`

It is, of course, possible for both to be supported. Indeed, #1 *must* be supported, since it's a language feature. It's mainly just ugly. #2 forces the spec for type to solidfy, there can never be any more positional arguments without breaking backwards compatibility (which is fine, because the recommendation is no more than 2 fixed-position arguments).

Needs a better name than type probably. Does not allow for structured export but does reduce some ambiguity.


### Back to data ###

If we keep `data` but make it so the keys can't accept a plain list describing the format but instead use something like `type`, above, but call it `x` it would solve the problems I have with `data` currently.

Additionally, I would want to remove the x prefix for the keys and completely remove the defaults for stuff baked into the root of the struct. If defaults are needed there can be a parent static. The problem with this is that it would prevent the use of other important keys like *name*, *base*, etc. and it's not enough to just say "don't use those names" because Serializeable may gain more keys.

To be sane, the solution would be to put the structural keys in a `keys` substruct but it's not very notational. Other options would be:

* put the non-structural keys in a substruct (gross)
* use x again (was always gross)
* expand the syntax to allow for some other specialized prefix like `/` or something (how to reference this?)
* require quoted keys for those (not very notational)
  - ref would be `@Struct["key"]` probably
* every key could be a substruct, which wouldn't require any core changes, but may influence whether or not we accept named basic-type struct declarations as substructs
  - accepting those: `x key [number, 2]`
  - otherwise: `x key {:[number, 2]}`
  - referencing: `@Struct{key}`

While the last method seems like it ticks the most boxes, it does break the convention for substruct naming which is unfortunate but not terrible. Aesthetically it's pretty nice, keeping short refs (vs going thru a key of separation) and has a good separation between xkeys and regular ones. This does effectively require every `data` struct to have a name (so the subs aren't exposed to the global scope) which is fine but will be a gotcha.
