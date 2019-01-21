# String functions library #
Library name would likely be: `string`

Certain string manipulation functions would likely be very useful as built-ins, particularly against statically defined strings (i.e. not references to external strings).

Most of these functions would likely be unable to accept external data because they're not two-way; however, any that can be implemented in a two-way fashion could be allowed to be used in such a way in a given implementation.

If ever possible for a library to extend basic types, similar to haXe's `using` statement, these functions could be keys representing alternate views of the data, if it's possible to access them in that way.


## split ##
Splits a string on a given substring or multiple substrings. If there is some sort of `regex` type, that should also be acceptable as a substring.

This is a `list` type.

### Usage ###

```rpl
split[STRING, SUBSTRING_1, ...]
```

```rpl
split {
    data: STRING
    on: SUBSTRING
    on: [SUBSTRING_1, ...]
}
```

### Example ###

```rpl
static {
    split: split["a,b.c", ",", "."]
    result: [a, b, c]
}
```


## join ##
Joins a list of strings into a single string with some string in between each. If you wish to have a preceding or trailing copy of the separator, place an empty string at the appropriate point.

If *data* is a single string, it should likely still accept it and simply not add any joiners. This should not convert any numbers into strings.

### Usage ###

```rpl
join[[STRING_1, ...], JOINER]
```

```rpl
join {
    data: [STRING_1, ...]
    with: JOINER
}
```

### Example ###

```rpl
static {
    join: join[[a, b, c], -]
    result: "a-b-c"
}
```


## doc ##
When provided a list, this is shorthand for a `join` with the joiner set as `"$n"`, thus providing a simple way to do multi-line strings where each line represents a real line and there is no need to strip indenting.

When provided a string, it should dedent the string based on the following algorithm:

1. If the first line is all spaces (`/\A\s*$/u`), remove it; otherwise, keep it but do not process it.
2. If the last line is all spaces (`/^\s*\z/u`), keep it as an empty line.
3. Do not process any empty line (`/^$/`).
4. Extract the spacing at the beginning of every line (`/^(?P<sp>\s+)/u`), hereafter called indentation.
5. Keep the shortest `sp` (in terms of number of characters) as `sp`.
6. Remove that spacing from every line in the string (`s/^\k<sp>//gu`).
7. Print a warning if some line has indentation that is not the same as the selected string.

### Usage ###

```rpl
doc[STRING_1, ...]
```

```rpl
doc {
    data: [STRING_1, ...]
}
```

### Examples ###

```rpl
static {
    doc: doc[
        It's raining; it's pouring.
        The old man was snoring.
        He bumped his head on the back of the bed.
        And couldn't get up in the morning.
    ]
    result:
        `It's raining; it's pouring.$n`
        `The old man was snoring.$n`
        `He bumped his head on the back of the bed.$n`
        `And couldn't get up in the morning.`
}
```

```rpl
static {
    doc: doc(``
        A wonderful bird is the pelican,
        His bill will hold more than his belican,
        He can take in his beak
        Enough food for a week
        But I'm damned if I see how the helican!
    ``) # ~ Dixon Lanier Merritt
    result:
        `A wonderful bird is the pelican,$n`
        `His bill will hold more than his belican,$n`
        `He can take in his beak$n`
        `Enough food for a week$n`
        `But I'm damned if I see how the helican!`
}
```
