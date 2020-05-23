# references #

This document covers the syntax for references, how they are implemented, how they fit into the link system, and special references.

References are a way of tying structures together in meaningful ways so as to replicate and synchronize data or use established structs as a type in certain situations. These are similar to pointers and references in other languages but provide some additional benefits that its imperative language relatives do not.


## Syntax ##

The basic tokens are that at sign (`@`) initiator, the optional brackets (round or square), the initial struct name, then a sequence of subscriptions. There are three subscriptors: list index, key name, and substruct.

A reference is delimited by one of these forms:

* `@(` and `)`
* `@[` and `]`
* `@` and before anything that's not valid in a reference.

Inside of a reference, it must either begin with a struct name or a substruct subscriptor, then it may be followed by any subscriptor. The struct name has the same restrictions as any struct name. The subscriptors have the following forms.

* `[` some number or reference `]`
* a key separator followed by a key name
* `{` substruct name `}`

Putting it together in eBNF:

```ebnf
Reference = "@" ( "(" Body ")" | "[" Body "]" | Body )
Body = ( StructName | SubscriptSubstruct ) Subscriptor*
Subscriptor = SubscriptKey | SubscriptIndex | SubscriptSubstruct

SubscriptKey = ("." | "-" | "─" | "->") KeyName
SubscriptIndex = "[" (Number | Reference) "]"
SubscriptSubstruct = "{" StructName "}"
```

There may be spacing (including newlines) or even comments between each token.

Note that this syntax is written in terms of semantic validity, what is allowed and makes sense, rather than how a parser should interpret it. Syntax parsing should not fail if a reference is, for instance, missing a Body between two brackets. This is a semantic error that should give clearer errors than a grammar parser would.

At this point it may be unclear what some of these constructions actually do. That will be discussed in the following sections.


## What they do and how they work ##

### Referencing common static data ###

One can store data in a `static` struct and have other structs reference it by name. This is similar to `const`s in other languages. This also demonstrates the key subscriptor.

```rpl
static Const {
    namesize: size(10 bytes)
}

string Name1 {
    size: @Const.namesize
}

string Name2 {
    size: @Const.namesize
}
```

Thus both Name1 and Name2 have a size of 10 bytes.

### List index subscriptor ###

```rpl
static Const {
    names: [Lorem, Ipsum]
}

string Name1 {
    name: @Const.names[0]  # Lorem
}
```

### Substruct subscriptor ###

In normal substruct subscription, we start from the initial struct, check all its children for that name, and if that fails, enter any nameless substructs and check those for that name, and continue until it's found or there are no more nameless substructs (in which case the reference fails).

```rpl
static Example {
    static Child1 {
        a: 1
    }

    static {
        static Child2 {
            a: 2
        }
    }
}

static {
    ac1: @Example{Child1}.a  # 1
    ac2: @Example{Child2}.a  # 2
    ac3: @Example{Child3}.a  # exception
}
```

If there is no base, it first checks the substructs of the struct that this reference is inside of, and if none of those match, it then checks the parent's substructs, and so forth. It does not descend into substructs.

```rpl
static Grandparent {
    name: Gertrude

    static Parent {
        name: Walter

        static Me {
            gaga: @{Grandparent}.name      # Gertrude
            papa: @{Parent}.name           # Walter
            sisy: @{Sister}.name           # Jenny
            baby: @{Child}.name            # Kylie
            lil1: @{Grandchild}.name       # exception
            nephew: @{Sister}{Child}.name  # Abe

            static Child {
                name: Kylie

                static Grandchild {
                   name: Billy
                }
            }

            static {
                name: for demonstration purposes

                static Grandchild {
                   name: this doesn't work either
                }
            }
        }

        static Sister {
            name: Jenny

            static Child {
                name: Abe
            }
        }
    }
}
```

In order to descend, one can use this to select a known parent and use further substruct subscriptors to access those, such as in *nephew*, or use relative references described in the [Special references](#special-references) section.

### Referencing dynamic data ###

References can act as a way to tie two or more keys together so that their values must always be equivalent in a way that respects their keys' normalization schemes. For example:

```rpl
number A {
    size: 4
}

bin B {
    size: @A.data
}
```

`bin`'s *size* normalizes what it's provided into a `size` type, converting bare numbers into the equivalent size in bytes. When this description is used for extracting information from a binary file, the first four bytes are read into A, then B.size interprets this number as the number of bytes to read into B. When using this to instead construct a binary blob, some data may be provided to B which would set its *size* to the number of bytes that data takes up as a size type value, say `size(10 bytes)`, then that, in turn, would set the value of A to the number type of that value, `10`.

### Propagating clones ###

References can also use a struct as a type and act as a means of propagating cloning, such as in this example:

```rpl
data Header {
    xcount: [number, 2]
    xindexes: [@Index, @this.xcount]
    # ^ use Index as a type
    # "this" is a relative reference, referring to "Header"
}

data Index {
    xname: [string, 10]
    xsize: [number, 2]
}

bin File {
    # These propagate cloning.
    name: @Index.xname
    size: @Index.xsize
}
```

In this example, Index is used as a type in Headers.xindexes, which causes Index to be cloned the specified number of times. Because there is no one Index for File to refer to, File is instead cloned the same number of times as Index with a 1:1 relationship. That is, for every Index there is a File such that Index0 <- File0, Index1 <- File1, ... IndexN <- FileN, and ordered like Index0...IndexN File0...FileN, since the *base* isn't specified.


## Link structure ##

TODO:


## Special references ##

There are some special, reserved names that can be used in place of the initial struct name or in the key name only if that key subscription is directly following one such reserved name.

### Relative references ###

Relative references use a hierarchical model strictly defined by when curly braces are used. So in `static A { static B { c: [ HERE ] }}` A is the parent of B, and at the location marked "HERE", B is considered **this**, despite "HERE" being contained in a list which is contained in *c*.

* `@this` refers to the struct which contains this reference
* `@parent` is the direct parent struct of `@this`
* `@gparent` (as in grandparent) is equivalent to `@parent.parent`
* `@ggparent` (as in great grandparent) is equivalent to `@parent.parent.parent`
* You may add as many `g`s as you wish to ascend higher.
* These are more meant for internal use:
  - `@姉` refers to `this`'s directly previous sibling, or if none, the parent (possibly the root).
  - `@妹` refers to `this`'s directly following sibling, or if none, the parent's directly following sibling (and so forth).

The following is reserved, without definition:

* `@back`
* `@wback` (as in way back)
* `@wwback` (as in way, way back)
* Any number of `w`s preceding `back`

### Defs ###

Rather than a reserved name exactly, Defs is more like a `static` that is always defined, even if it was not explicitly defined in a description. Its keys are meant to represent arbitrary values defined on the command-line similar to `-D name=definition` in GCC. If a description defines a struct named Defs, it MUST be a `static` and it MUST NOT have any substructs besides `meta` substructs. Any values defined in Defs act as defaults and may be overwritten by the user.
