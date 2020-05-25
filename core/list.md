# Lists #

Lists are a core, syntactic type. The basic syntax, for reference, is:

```rpl
# Elements on the same line must be separated by a comma:
[first element, second element, ...]

# But newlines are also an element separator:
[
    first element
    second element
    ...
]
```

However this document is for discussing them as a struct, rather than their syntax.


## Keys ##

* *length* (number): Represents the number of elements in the list
* *type*: Describes the type of each element in the list.
  - As the literal **any**, the default, this allows all types.
  - As a literal, this is a single type name.
  - As a list of literals, this restricts the length and treats this as a typed tuple.
    + For example, `[string, number, number]` means a list of three elements with those types in that order, such as `["ab", 1, 2]`
  - In the future, one may specify generative grammars to represent more complex patterns of repeating types. However, this are as of yet undefined.


## List of serializables ##

If *type* is strictly defined as a serializable type, the list is also serializable. Thus it can be placed as a child of a serializable only in that case. It assumes all the elements are serialized sequentially, thus they must have a deterministic *size* when the list is meant to be un/serialized directly.


## Formatting ##

While lists cannot be used in a string context generically, they can be used in structs which can be stringified, such as in JSON or as an XML element's list of children. Of course it can also be stringified as RPL.

While these options have no business being specified in the structure, they could fall under whatever pretty-printing system is implemented. The options may be more prudent on JSON etc types directly in terms of where they're implemented, but it's important to have some level of standardization between all the methods of stringification.

* Spacing considerations:
  - Before/after opening/ending brackets (individually).
  - Between element and following separator.
  - Between separator and following element.
  - Indentation for new lines.
  - Tabular alignment of elements and/or separators.
    + Within this, justification of elements per datatype.
    + Numbers could be prepended with 0s based on the number with the most digits, if legal.
    + Column headers via comment, if legal.
* What separator or brackets to use, if there are multiple options.
  - Which to use between two elements on the same line vs between two lines.
  - Which separator to show after the final element, if any.
  - Which to show before the first element, if any, if legal...
* How to split elements and when:
  - After exceeding or in order to stay within a certain line length.
  - After a certain number of elements.
  - Given the elements can split internally, whether or not they should.


## Future ##

### As a context ###

This would need a separate prosal with valid, real-life use cases, but the following is an idea of how it could work.

List contexts are a bit similar to `bin` in that one can (sort of) think of `bin` as a list context with single bytes as the type. This allows one to process the contents of a (usually) 1D list into a more structured form.

```rpl
list {
    # Imagine these were pulled from something external!
    data: [1, q, 3, a, b, c]

    number Length1 {}
    list RolledUp1 {
        length: @{Length1}
        type: string
    }

    number Length2 {}
    list RolledUp2 {
        length: @{Length2}
        type: string
    }
}
```
