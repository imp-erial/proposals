# meta #
The meta-programming type, allowing you to:

* Make a new type based on an existing type.
* Define new keys for use in an existing struct.
* Extend management of a struct in tool-specific ways.

```rpl
TYPE NAME {
    # keys ...

    meta PURPOSE {
        # description ...
    }
}
```

TYPE and NAME work the same as defining a normal struct. Although PURPOSE goes where the name usually goes, this is instead a (name-compliant) keyword that has been registered as a meta extension. It could be registered by a tool or by the core itself.

For meta-programming, these registrations are suggested:

* keys
* defaults
* basic
* help

Tool-specific options could include describing the export format for Exchange or styling options for a GUI render of the struct.

## keys ##

This allows for defining the types for new keys. In order to be future-proof, these definitions should override existing keys with a warning while allowing library-internal references to reach the expected type and default despite this collision.

The keys can be defined with a literal of the type name or as a struct with keys pre-defined. This occludes literals with their data key pre-defined but defining it as a string should have the desired effect in that case.

### Example ###

```rpl
number {
    a: 1
    size: 4 bytes

    meta keys {
        a: number
    }
}
```

## defaults ##

This allows for defining defaults for new keys and overriding defaults for existing keys.

### Example ###

```rpl
number ShortByDefault {
    meta defaults {
        size: 2 bytes
    }
}
```

## basic ##

This allows for interpreting how the basic value is interpreted when instantiating this as a key-struct. Unlike most other meta types, defining multiple of these makes sense, as doing so allows for interpretting multiple forms of basic declarations.

It has a key _input_ which describes the structure of the basic as it's declared by an author (e.g. `type("a")`, the string `a`) and the latter describes what keys to put where.

It has a substruct _output_ which describes where to put the data pulled from the basic declaration. Use `@{basic}` to refer to the the input value from this substruct. This way, other places won't need to refer to the basic directly and can just rely on the keys it writes to. 

The system will scan any declared basics in order and pick the first one whose type matches.

### Example ###

```rpl
meta basic {
    input: string
    output {
        something: @{basic}
        length: math("@{basic}.length * 2")
    }
}
```

### Dynamic usage ###

This is a proposed method of handling dynamic usage if it's ever required. I struggle to find a justification for it to exist, however.

Output may also accept a substruct _key_ which allows for dynamic naming of keys. However, existence and type assertions will still take place, so the key name must exist.

```rpl
meta basic {
    input: list { type: [literal, any] }
    output {
        key {
            name: @{basic}[0]
            value: @{basic}[1]
        }
    }
}
```

Similarly, the same can be done for substructs with _substruct_.

```rpl
meta basic {
    input:
    output {
        substruct {
            type: @{basic}[0]
            name: @{basic}[1]
            keys {
                # ...
            }
        }
    }
}
```

## help ##

This allows for defining information about the struct to the author or user detailing what it does, how it's used, what each key means, etc.

For information on substructs, define a `meta help` in that substruct.

### Example ###

```rpl
meta help {
    name: Full name of the struct
    summary: Short description of what it's for
    description: Long description of what it's for, how to use it, etc.
    keys {
        key1: What this key is for, etc.
        key2: note that the type and defaults can
        key3: be retrieved from the struct declaration itself
    }
}
```

## Other properties and notes ##

* This is a proposal to replace syntax/meta without altering the syntax. It covers all the issues it had.
* `@parent` refers to the parent of where the meta struct is defined and not where it's used. `@back` could possibly refer to where it was used.
* Using a meta struct in a struct does not occlude it from the scope. There must be another system to do that, which will be in a separate proposal.
