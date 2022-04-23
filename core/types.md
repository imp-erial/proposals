# types #

Allows for defining new types within a MPRL file instead of making a library to provide it.

The name of this struct must be a valid type name. It defines the name of the "library" the contained types are defined in. If no name is defined, substruct names MUST be the first defined in scope. All substructs contained inside are new types which are members of this library.

Each substruct is declared as a normal struct, other than the name. Use [meta-programming](meta.md) to control adding extra keys or other functionality.

As usual with importing, if the type's name has not yet been declared in the scope, it can be accessed without the library name.

## Example ##

```mprl
types num {
	number byte  { size: 1 byte  }
	number word  { size: 2 bytes }
	number dword { size: 4 bytes }
	number qword { size: 8 bytes }
}

num.byte { base: $1000 }
byte {}  # also ok, since byte wasn't in scope already
```

## Defining types in a struct ##

Since this is a static struct, you may define it inside any other struct. Structs have scoped types, normally, in order to provide custom types (or more convenient names) for certain situations. Thus, declaring this as substruct adjusts the types available to that struct, rather than the global scope.

## Relative references ##

When a type definition uses `@this`, it refers to the struct using the type (and equally itself). This holds for using `@parent`, as well.

### Example ###

```mprl
types {
	bin pstring {
		# Define a new key: prefix size
		meta keys { psize: number }
		meta defaults { psize: 1  }

		# Define the format
		number size { size: @parent.psize }
		string data { size: @parent{size} }

		# Restrict the size
		size: math("@this.psize + @this{size}")

		# Change the top
		meta top { is: @parent{data} }
	}
}
```
