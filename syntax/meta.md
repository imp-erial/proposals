# meta #
The meta-programming type, allowing you to make a new type based on an existing type.

```mprl
meta TYPE NEW_TYPE {
    # keys...
}
```

TYPE must be an existing type and NEW_TYPE must be orthographically valid for a type name and must not be the same as TYPE.

meta types explicitely do not export or import anything where they are in the structure and effectively have no siblings.

There are three suggestions for accessing the data passed to the constructor:

* `@this.data`, this means `data: @this.data` would always be necessary if the data was intended to be passed through. Additionally data is a concept for value types, not all types.
* `@basic`, a new special reference just for this
* `@{}` pointing at an empty substruct, which is normally invalid

Following examples will use `@{}` to stand out.

## Simple example: intlist ##

```mprl
meta list intlist {
    type: number { size: 4 }
    length: @{}
}
```

Then this can be used in something such as:

    intlist(8)

To pull out a list of 8 uint_32s

## Complex example: string strip ##

```mprl
meta string strip {
    {
        bin: @{}[0].bin
        padside: both
        padding: @{SyncEncoding}.padding.bin
    }
    encoding: @{}[0].encoding
    static SyncEncoding {
        padding: @{}[1]
    }
}
```

This structure will strip some character sequence from a string by abusing the padding decoder; i.e. `strip["##abc#", "#"]` is equal to `"abc"`

## Other properties and notes ##

* Should be able to use references to the basic/its keys in unusual places such as in place of a key name, struct type, or struct name. Possibly in other references?
* `@parent` refers to the parent of where the meta struct is defined and not where it's used. `@back` could possibly refer to where it was used.

## Complexities ##

* Unable to deal with multiple types of basics like `x(0)` vs `x[0,1]`.
* Unable to deal with both basic instantiation and proper instantiation in one definition.
* Unable to do conditional substructuring (i.e. decide whether or not a certain substruct should exist).
* Cannot declare the intended types of the input keys.
