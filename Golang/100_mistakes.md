# Chapter 2
* Avoiding shadowed variables can help prevent mistakes like referencing the wrong variable or confusing readers.
* Avoiding nested levels and keeping the happy path aligned on the left makes building a mental code model easier.
* When initializing variables, remember that init functions have limited error handling and make state handling and testing more complex. In most cases, initializations should be handled as specific functions.
* Forcing the use of getters and setters isn’t idiomatic in Go. Being pragmatic and finding the right balance between efficiency and blindly following certain idioms should be the way to go.
* Abstractions should be discovered, not created. To prevent unnecessary complexity, create an interface when you need it and not when you foresee needing it, or if you can at least prove the abstraction to be a valid one.
* Keeping interfaces on the client side avoids unnecessary abstractions.
* To prevent being restricted in terms of flexibility, a function shouldn’t return interfaces but concrete implementations in most cases. Conversely, a function should accept interfaces whenever possible.
* Only use any if you need to accept or return any possible type, such as json. Marshal. Otherwise, any doesn’t provide meaningful information and can lead to compile-time issues by allowing a caller to call methods with any data type. Relying on generics and type parameters can prevent writing boilerplate code to factor out elements or behaviors. However, do not use type parameters prematurely, but only when you see a concrete need for them. Otherwise, they introduce unnecessary abstractions and complexity.
* Using type embedding can also help avoid boilerplate code; however, ensure that doing so doesn’t lead to visibility issues where some fields should have remained hidden.
* To handle options conveniently and in an API-friendly manner, use the functional options pattern.
* Following a layout such as project-layout can be a good way to start structuring Go projects, especially if you are looking for existing conventions to standardize a new project.
* Naming is a critical piece of application design. Creating packages such as common, util, and shared doesn’t bring much value for the reader. Refactor such packages into meaningful and specific package names.
* To avoid naming collisions between variables and packages, leading to confusion or perhaps even bugs, use unique names for each one. If this isn’t feasible, use an import alias to change the qualifier to differentiate the package name from the variable name, or think of a better name.
* To help clients and maintainers understand your code’s purpose, document exported elements.
* To improve code quality and consistency, use linters and formatters.

# Chapter 3
* When reading existing code, bear in mind that integer literals starting with 0 are octal numbers. Also, to improve readability, make octal integers explicit by prefixing them with 0o.
* Because integer overflows and underflows are handled silently in Go, you can implement your own functions to catch them.
* Making floating-point comparisons within a given delta can ensure that your code is portable.
* When performing addition or subtraction, group the operations with a similar order of magnitude to favor accuracy. Also, perform multiplication and division before addition and subtraction.
* Understanding the difference between slice length and capacity should be part of a Go developer’s core knowledge. The slice length is the number of available elements in the slice, whereas the slice capacity is the number of elements in the backing array.
* When creating a slice, initialize it with a given length or capacity if its length is already known. This reduces the number of allocations and improves performance. The same logic goes for maps, and you need to initialize their size.
* Using copy or the full slice expression is a way to prevent append from creating conflicts if two different functions use slices backed by the same array. However, only a slice copy prevents memory leaks if you want to shrink a large slice.
* To copy one slice to another using the copy built-in function, remember that the number of copied elements corresponds to the minimum between the two slice’s lengths.
* Working with a slice of pointers or structs with pointer fields, you can avoid memory leaks by marking as nil the elements excluded by a slicing operation.
* To prevent common confusions such as when using the encoding/json or the reflect package, you need to understand the difference between nil and empty slices. Both are zero-length, zero-capacity slices, but only a nil slice doesn’t require allocation.
* To check if a slice doesn’t contain any element, check its length. This check works regardless of whether the slice is nil or empty. The same goes for maps.
* To design unambiguous APIs, you shouldn’t distinguish between nil and empty slices.
* A map can always grow in memory, but it never shrinks. Hence, if it leads to some memory issues, you can try different options, such as forcing Go to recreate the map or using pointers.
* To compare types in Go, you can use the == and != operators if two types are comparable: Booleans, numerals, strings, pointers, channels, and structs are composed entirely of comparable types. Otherwise, you can either use reflect.DeepEqual and pay the price of reflection or use custom implementations and libraries.









