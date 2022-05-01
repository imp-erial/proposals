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

* *data* (list): This is a value struct, so it has the recursive *data* key.
  - Entries in a list are also considered its children, so children reflect this key.
* *length* (number): Represents the number of elements in the list
* *type*: Describes the type of each element in the list.
  - As the literal **any**, the default, this allows all types.
  - As a literal, this is a single type name.
  - As a list of literals, this restricts the length and treats this as a typed tuple.
    + For example, `[string, number, number]` means a list of three elements with those types in that order, such as `["ab", 1, 2]`
  - In the future, one may specify generative grammars to represent more complex patterns of repeating types. However, these are as of yet undefined.
* *next* (list): Another list which is concatenated to *data* such that only the basic value refers to the total list. This allows ranges to be inserted in a flattened manner.

*each* is an alias of *type* intended to be used when referencing, tying into the cloning system. That is, *type* is cloned for each entry in the list, so referencing it causes the referencer to be cloned for each entry in the list.

## List of serializables ##

If *type* is strictly defined as a serializable type, the list is also serializable. Thus it can be placed as a child of a serializable only in that case. It assumes all the elements are serialized sequentially, thus they must have a deterministic *size* when the list is meant to be un/serialized directly.

This should hold true for any linearly ordered types. Entries in a list cannot define *base*.

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

### Semantics ###

There are multiple types of lists or collections in this world, and sometimes it can be important to know what kind you intend, since *list* generically represents all of them. While this could be done via types which inherit from *list* I believe the notions can be split into properties as well.

From theory we have sets, bags, lists, arrays, matrices, vectors, sequences, tuples, queues, stacks. Arrays and matrices can be multi-dimensional, but looking at only single-dimensional options, it's often unnecessary to distinguish between many of these terms. Additionally they can mean different things in different programming languages which can make it too confusing to refer to them by name.

From common programming paradigms, I would also like to suggest foreach type lists. These are lists meant to be run in a map call, essentially; where a function is applied to each element. There are multiple common use cases here, though. It could be like map (a transformation of every element), or a reduction (run a function on the first element (or take the raw first element), then pass that result and the next element into the function, etc) or it could be like a list of ports that are available for you to listen on where you only want one of them (the first one you pull out that works), or possibly other scenarios.

Some other potential types of lists might be argument lists (each element has a position, type, meaning, and name; and they're meant for a specific function probably), the result of a taxonomical descent ([gparent, parent, this] type thing), ...

#### Theoretical definitions ####

* Set: An unordered collection containing unique elements.
* Bag: An unordered collection able to contain duplicates, such that it's typically stored as a hashmap of value -> count.
* List: An ordered collection which is not necessarily finite, referring specifically to how it's stored, as a sequence of disjointed elements pointing to where the next is (linked list). Adding to a list (at the front or back) is Θ(1) while lookup must be Θ(n) time.
* Array: Fixed length (in theory), ordered, single or multi-dimensional collection. Can often refer to having values from a limited set of values, but not always. Lookup must be Θ(1) time.
* Matrix: Similar to arrays, but essentially always two dimensional.
* Vectors: A collection of numbers which each tend to represent a specific meaning, such as direction and speed. It must exist in vector space. The numbers are technically ordered but not typically indexed as the vector is treated as a whole unit. Often interchangeable with "array" in programming, though, and in such a case every element must be the same type.
* Sequences: Can be used as a generic term for all ordered, one-dimensional collections. They are typically finite. In Haskell it's implemented as a finger tree, distinct from its linked lists.
* Tuples: Fixed length, ordered collection which has specific type definitions for each element. Allegorical to \[mathematical] vectors.
* Queues: A collection, typically a list, which is managed as first in first out (FIFO)
* Stacks: A collection, typically a list, which is managed as last in first out (LIFO)

#### As properties ####

* Ordered vs unordered
* Infinite (function which takes an index and produces a value) vs finite (defined values, or the same function within a limited index range)
* Closed vs open - a concept from linguistic sets where a closed set is a grammatical category which cannot have more words added to it (such as pronouns) vs an open one which can have new words added to it (such as nouns). This is relevant to situations involving merging vs equivalence assertion for exported values which may not be complete, such as exported image palettes in multiple exports when the palette itself is shared in the target binary. The exports may not be equivalent (because they only have the colours used in that specific image) but it should not error when reimporting, it should construct the unioned palette from all exports.
* Unique values or not
* Dimensionality? This could be about imposing the type of a list's elements as being lists of a certain length, technically.
* Types for the elements, either one for every element or, for ordered lists, as a index -> type association (via list or function)
* FIFO vs LIFO - only relevant to ordered lists
* How to apply as an argument to a function: permutative sequence, parallel sequence, options (pick one; potentially pick best or pick worst variants?)
* What to care about in the result: all succeeded/failed, some threshold succeeded/failed (e.g. at least one, over 50%, etc), mapping (produce a list of values)
* A little of both: reduction
