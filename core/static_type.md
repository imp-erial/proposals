# static.type

Replaces rejected [extern](../syntax/extern.md) proposal.

Instead of introducing a new keyword which complicates parsing, this proposes the use of the `static` builtin type as a namespace for creating static version of other types. The specific types would not have to be registered under the static namespace. Instead, subtyping static will query the types available in the scope and create a static version on demand.

`static.number Two {data: 2}` could be considered syntactic sugar for the following:

```mprl
static {
    number Two {data: 2}
    meta top {
        is: @parent{Two}
    }
}
```

Although the concept of using top-level structs as a type has been deprecated, I still believe there are times when this is useful, though less so for syntactic types like numbers. Although such things could be constrained as keys within a static, this provides another option, and also allows for explicitness about what is static and what isn't (since this could still be used as a prefix for already static types).

That said, I'm mostly writing this down so I don't forget the idea in case I find a better use case for it.
