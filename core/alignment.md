# alignment

* natural alignment: alignment for a free scalar in memory
* embedded alignment: alignment for a member of a struct
* leading padding: padding between the start of a struct and its first member
* slop: padding between members in a struct
* stride: padding that comes at the end of a struct

## Strategies

* self-aligned: aligned on a boundary that's a multiple of its own size
  * aka "aligned on natural bounds" I think?
  * chars can start on any, shorts on multiples of 2, etc
  * when you have a larger type below a smaller one, align the larger one to its appropriate bound and slop goes between the two members
  * bit fields cannot cross byte boundaries unnecessarily, adding slop bits between as needed
  * struct members are also aligned, preventing subsequent members from crossing its bounds
* aligned to a specific multiple, like 16 bits on Motorola 68020
  * MIPS has something like this, but the alignment is selected by the coder as some power of 2
* word addressing in CDC & Cray machines, don't need alignment
* Power alignment: derived from the alignment rules used by the IBM xlc compiler for the AIX operating system
  * the embedding alignment of the first element in a data structure is equal to the element’s natural alignment
  * for subsequent elements with a natural alignment less than 4, the embedding alignment of each element is equal to its natural alignment
  * for subsequent elements that have a natural alignment greater than 4 bytes, the embedding alignment is 4, unless the element is a vector data type
  * the embedding alignment for vector data types is always 16 bytes
  * the embedding alignment of a composite type (array or data structure) is determined by the largest embedding alignment of its members
  * the total size of a composite type is rounded up to a multiple of its embedding alignment, and is padded with null bytes
* Mac68k alignment: derived from the alignment rules used by the MPW compilers for classic Mac OS
  * the embedding alignment of a char type is 1 byte
  * the embedding alignment of all other types other than vector types is 2 bytes
  * the embedding alignment for vector data types is 16 bytes
  * the total size of a composite data type is rounded up to a multiple of 2 bytes


## Summary

The goal of alignment is about minimizing how many reads it takes to fetch the full data requested. It seems that at the low level, processors can only fetch in windows of certain sizes (BYTE, WORD, DWORD, etc) and those addresses are inherently aligned to a multiple of that size (hence the self-aligned strategy). So if you are requesting a WORD where that is the maximum size for the processor, when the data is misaligned, it requires two fetches instead of one, then must reconstruct the desired data. Ultimately I don't know if all processors are made this way, if there are ones made a different way, or what the actual restrictions are at a transistor level that require this behavior; but it's clear that if these are the limitations we're working with that alignment makes for more efficient fetches.

Another way I've seen this described has to do with generating the high order byte(s) of the address, that if they're all the same the processor only does it once. I have no idea if this is true, and if so, whether or not it's the same reason as above.

Aligned elements may not cross byte or bit boundaries unnecessarily. For instance `[string{size:5}, number{size:4}]` if this is self-aligned in a 32-bit architecture, the first element has to cross the boundary but the second does not, thus in order to prevent the second from crossing the boundary unnecessarily, 3 padding bytes are inserted between the two. These boundaries are about the word sizes a system can process and ends up essentially being a way of thinking about that high order address bytes thing. So ultimately I think we just have to describe what all these sizes are with an offset to account for any header in the ROM.

Finding exceptional cases to data alignment is markedly difficult. Realistically I think all the above strategies are more or less the same.

Padding/slop should be 00s but it's not guaranteed. I wonder if Imperial can handle something more complex than specifying a padding byte (like in strings)? It's possible that it could be directly accessible as a bin or a list of bins (or both).

The configuration may be as simple as having a maximum number of bytes the boundary can contain, then the actual number would be the largest type in the structure up to that amount. Exceptional cases will decide if there's more info...if I can find them.

[gcc](https://gcc.gnu.org/onlinedocs/gcc-5.2.0/gcc/Type-Attributes.html) offers alignment overrides per member.

Maybe a better idea to have alignment as a locator key on serializables? This will need to inspect the following struct's size in order to determine where its boundary will be, but that's fine. It's only if we're keeping the padding, I guess. Since this would change the base, it might be better served as an option under base, then references to base don't have to be weird. Maybe it's just better as `base: math("something & 0xf8")` though that's not notational and makes automatic bases annoying


## Recommendation so far

Add `align` type used in *base* keys to align the address regardless of it being explicitly set or not. It could also be used in other numerical contexts to create aligned pointers which would not allow implying the "<ins>address</ins>". Forms are:

* `align(ADDRESS, ALIGNMENT, ROUNDING)`
* `align(ADDRESS, ALIGNMENT)`
* `align(ALIGNMENT NAME)`
* `align(ADDRESS)`

These correspond to these keys when treated as a struct:

* *address*: this is what the *base* key normally takes here
  * if blank or omitted, it defaults the same as the *base* key would (to `c` in a binary context)
* *alignment*: one of two options:
  * same as what the *aligned* specialized key defined below takes as a basic value
  * an alignment name defined in the parent's *aligned.alignment*'s specialized key form
* *rounding*: specifies whether the address should be rounded down (if possible) or up
  * when set to **down**, attempts to round down and generates a warning if there's a collision (then rounds up)
    * aliases: **floor**
  * when set to **up**, attempts to round up, which can only collided if the subsequent child is unaligned, which would cause an error
    * aliases: **ceil**
  * when unset, attempts to round down then rounds up if it can't; no warning generated

Add *aligned* specialized key to all serializables which can take a basic of **packed** (default), **self**, or a `size` specifiying the size of a row (wrt bounding). It's possible other basics (particularly literals) could be added later. This controls the implied alignment of its children. As a struct it has the keys:

* *alignment*: holds the basic
  * when set to **packed**, there is no alignment
    * aliases: **none**, **pack**, **0**, **1**
  * when set to **self**, it uses the *size* key as the value here ceil'd to the nearest byte.
  * if *base* is explicitly set, this value is ignored (a warning may be generated if it's misaligned).
  * may be set as a specialized key which contains a name -> size mapping, such as `word: 2`.
* *padding* or *slop* or *stride* (bin): stores the padding actually used
  * if unset and this struct is serialized, it defaults to using 00s
* *blockades* (list of math): describes boundaries this data cannot cross at all
  * supplies the variable **x** which is equal to the address being tested.
  * if the address space from *base* to *end* (exclusive) contains some address such that one of these expressions evaluates to **x** (e.g. `x & 0xf800` == `x`) then this basing is erroneous. If the *base* was defaulted, it can be adjusted to start at the nearest upcoming blockade; otherwise, it should raise an exception.
    * honestly I don't like this "== `x`" part and I think the condition should be part of the equation. Possible this could be a list of some new `cond` type or some such which evaluates math embedded in a boolean proposition

The basic should be normalized to the actual size used.

Note that if a parent is aligned, its childrens' defaulted *base*s will be aligned (i.e. the default *base* will become `align(c, ALIGNMENT)`). However if it instead uses the *alignment* specialized key, there is no default alignment, so each base must specify what to use by name. Possible we could implement a *strategy* key which determines the alignments automatically based on the size and/or type of the children (and/or their children). If so, **self** could be moved there, possibly, or be linked.

System-specific libraries should be able to set up the default alignment for the system. This doesn't mean children should be automatically aligned, simply that the names should be defined (byte, word, ..., bank).


## References

* [The Lost Art of Structure Packing](http://www.catb.org/esr/structure-packing/)
* [PowerPC Data Alignment](http://mirror.informatimago.com/next/developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/2rt_powerpc_abi/chapter_9_section_3.html)
* [The Impact of Memory and Architecture on Computer Performance](https://www.math.utah.edu/~beebe/fonts/memperf.pdf)
* Nice graphics for self-alignment: [Effiziente Programmierung in C](https://hps.vi4io.org/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf)
