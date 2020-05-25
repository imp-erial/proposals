# alignment

* natural alignment: alignment for a free scalar in memory
* embedded alignment: alignment for a member of a struct
* leading padding: padding between the start of a struct and its first member
* slop: padding between members in a struct
* stride: padding that comes at the end of a struct

## Strategies

* self-aligned: aligned on a boundary that's a multiple of its own size
  - aka "aligned on natural bounds" I think?
  - chars can start on any, shorts on multiples of 2, etc
  - when you have a larger type below a smaller one, align the larger one to its appropriate bound and slop goes between the two members
  - bit fields cannot cross byte boundaries unnecessarily, adding slop bits between as needed
  - struct members are also aligned, preventing subsequent members from crossing its bounds
* aligned to a specific multiple, like 16 bits on Motorola 68020
  - MIPS has something like this, but the alignment is selected by the coder as some power of 2
* word addressing in CDC & Cray machines, don't need alignment
* Power alignment: derived from the alignment rules used by the IBM xlc compiler for the AIX operating system
  - the embedding alignment of the first element in a data structure is equal to the element’s natural alignment
  - for subsequent elements with a natural alignment less than 4, the embedding alignment of each element is equal to its natural alignment
  - for subsequent elements that have a natural alignment greater than 4 bytes, the embedding alignment is 4, unless the element is a vector data type
  - the embedding alignment for vector data types is always 16 bytes
  - the embedding alignment of a composite type (array or data structure) is determined by the largest embedding alignment of its members
  - the total size of a composite type is rounded up to a multiple of its embedding alignment, and is padded with null bytes
* Mac68k alignment: derived from the alignment rules used by the MPW compilers for classic Mac OS
  - the embedding alignment of a char type is 1 byte
  - the embedding alignment of all other types other than vector types is 2 bytes
  - the embedding alignment for vector data types is 16 bytes
  - the total size of a composite data type is rounded up to a multiple of 2 bytes


## Summary

The goal of alignment seems to primarily be about have some number of LSBs be 0. Not totally sure where the performance benefits come in on this, but seems to be something with how it goes over the bus.

Aligned elements may not cross byte or bit boundaries unnecessarily. For instance `[string{size:5}, number{size:4}]` if this is self-aligned in a 32-bit architecture, the first element has to cross the boundary but the second does not, thus in order to prevent the second from crossing the boundary unnecessarily, 3 padding bytes are inserted between the two. Though this example is about self-aligned systems, the existence of a boundary seems to be critical in all strategies. Most of the strategy, then, seems to be about inferring this value. Likely, then, most of the configuration for alignment should be around how to find it. There can also be other boundaries, like banks.

Finding exceptional cases to data alignment is markedly difficult.

Padding/slop should be 00s but it's not guaranteed. I wonder if Imperial can handle something more complex than specifying a padding byte (like in strings)? It's possible that it could be directly accessible as a bin or a list of bins (or both).

The configuration may be as simple has having a maximum number of bytes the boundary can contain, then the actual number would be the largest type in the structure up to that amount. Exceptional cases will decide if there's more info...if I can find them.

[gcc](https://gcc.gnu.org/onlinedocs/gcc-5.2.0/gcc/Type-Attributes.html) offers alignment overrides per member.

Maybe a better idea to have alignment as a locator key on serializables? This will need to inspect the following struct's size in order to determine where its boundary will be, but that's fine. It's only if we're keeping the padding, I guess. Since this would change the base, it might be better served as an option under base, then references to base don't have to be weird. Maybe it's just better as `base: math("something & 0xf8")` though that's not notational and makes automatic bases annoying


## Recommendation so far

Add *aligned* specialized key to all serializables which can take a basic of **packed** (default), **self**, or a `size` specifiying the size of a row (wrt bounding). It's possible other basics (particularly literals) could be added later. As a struct it has the keys:

* *alignment*: holds the basic
  - when set to **packed**, there is no alignment
    + aliases: **none**, **pack**, **0**, **1**
  - when set to **self**, it uses the *size* key as the value here ceil'd to the nearest byte.
  - if *base* is explicitly set, this value is ignored (a warning may be generated if it's misaligned).
* *padding* or *slop* or *stride* (bin): stores the padding actually used
  - if unset and this struct is serialized, it defaults to using 00s
* *blockades* (list of math): describes boundaries this data cannot cross at all
  - supplies the variable **x** which is equal to the address being tested.
  - if the address space from *base* to *end* (exclusive) contains some address such that one of these expressions evaluates to **x** (e.g. `x & 0xf800` == `x`) then this basing is erroneous. If the *base* was defaulted, it can be adjusted to start at the nearest upcoming blockade; otherwise, it should raise an exception.

The basic should be normalized to the actual size used.


## References

* [The Lost Art of Structure Packing](http://www.catb.org/esr/structure-packing/)
* [PowerPC Data Alignment](http://mirror.informatimago.com/next/developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/2rt_powerpc_abi/chapter_9_section_3.html)
* [The Impact of Memory and Architecture on Computer Performance](https://www.math.utah.edu/~beebe/fonts/memperf.pdf)
* Nice graphics for self-alignment: [Effiziente Programmierung in C](https://hps.vi4io.org/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf)
