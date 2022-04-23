# extern #
A simple hint to the processor that this struct is not directly a part of the overall structure.

```mprl
extern TYPE NAME {
    # ...
}
```

This allows one to include structs that would be used for types without the worry of them affecting the unpacking if they're never managed by another struct.

Specifically, this means it won't connect with other packable structs via implicit base/end.

## Complications ##

What does this mean when used on substructs? Can it be?
