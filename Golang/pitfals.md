* Chapter 7. Most type conversions are checked at compile time, so if they are invalid, your code won’t compile. (Type conversions between slices and array pointers can fail at runtime and don’t support the comma ok idiom, so be careful when using them!)

TODO
Intefaces and nil 


# Errors
* Error messages should not be capitalized nor should they end with punctuation or a newline. In most cases, you should set the other return values to their zero values when a non-nil error is returned.
* The second situation is a reused err variable. The Go compiler requires every variable to be read at least once. It doesn’t require that every write to a variable is read. If you use an err variable multiple times, you have to read it only once to make the compiler happy. **Use tooling to detect**.
* Sentinel errors are one of the few variables that are declared at the package level. By convention, their names start with Err (with the notable exception of io.EOF). They should be treated as read-only; there’s no way for the Go compiler to enforce this, but it is a programming error to change their value. Sentinel errors are usually used to indicate that you cannot start or continue processing. For example, the standard library includes a package for processing ZIP files, archive/zip. This package defines several sentinel errors, including ErrFormat, which is returned when data that doesn’t represent a ZIP file is passed in. Be sure you need a sentinel error before you define one. Once you define one, it is part of your public API, and you have committed to it being available in all future backward-compatible releases. It’s far better to reuse one of the existing ones in the standard library or to define an error type that includes information about the condition that caused the error to be returned.
* Even when you define your own custom error types, always use error as the return type for the error result.
* When using custom errors, never define a variable to be of the type of your custom error. Either explicitly return nil when no error occurs or define the variable to be of type error.
* Avoiding the init function if possible.
* That means that any package-level variables configured via init should be effectively immutable. While Go doesn’t provide a way to enforce that their value does not change, you should make sure that your code does not change them.
* The go get command downloads modules and updates the go.mod file (two paths are possible - just go get and go get `<specific module>`).
* Go’s semantic versioning supports the concept of pre-releases. Let’s assume that the current version of your module is tagged v1.3.4. You are working on version 1.4.0, which is not quite done, but you want to try importing it into another module. What you should do is append a hyphen (-) to the end of your version tag, followed by an identifier for the pre-release build. In this case, use a tag like v1.4.0-beta1 to indicate beta 1 of version 1.4.0 or v1.4.0-rc2 to indicate release candidate 2.
* Go provides a way for you to indicate that certain versions of a module should be ignored. This is done by adding a retract directive to the go.mod file of your module. It consists of the word retract and the semantic version that should no longer be used.
* A workspace allows you to have multiple modules downloaded to your computer, and references between those modules will automatically resolve to the local source code instead of the code hosted in your repository.
* The go.work file is meant for your local computer only. Do not commit it to source control!

Ok:
```
func GenerateErrorUseErrorVar(flag bool) error {
	var genErr error
	if flag {
		genErr = StatusErr{
			Status: NotFound,
		}
	}
	return genErr
}
```
Bad:
```
func GenerateErrorBroken(flag bool) error {
	var genErr StatusErr
	if flag {
		genErr = StatusErr{
			Status: NotFound,
		}
	}
	return genErr
}
```

* A function in the Go standard library wraps errors, and you’ve already seen it. The fmt.Errorf function has a special verb, %w. Use this to create an error whose formatted string includes the formatted string of another error and which contains the original error as well. The convention is to write : %w at the end of the error format string and to make the error to be wrapped the last parameter passed to fmt.Errorf.
* If you need to handle errors that may wrap zero, one, or multiple errors, use this code as a basis.
```
	var err error
	err = funcThatReturnsAnError()
	switch err := err.(type) {
	case interface {Unwrap() error}:
		// handle single error
		innerErr := err.Unwrap()
		// process innerErr
	case interface {Unwrap() []error}:
		//handle multiple wrapped errors
		innerErrs := err.Unwrap()
		for _, innerErr := range innerErrs {
			// process each innerErr
		}
	default:
		// handle no wrapped error
	}
```
* Use errors.Is when you are looking for a specific instance or specific values. Use errors.As when you are looking for a specific type.
* Sometimes you find yourself wrapping multiple errors with the same message - you can simplify by using defer.

# Modules, Packages, and Imports

* While you can store more than one module in a repository, it is discouraged. Everything within a module is versioned together. Maintaining two modules in one repository requires you to track separate versions for two different modules in a single repository.
* To reduce confusion, do not use uppercase letters within it.
* Sometimes you want to share a function, type, or constant among packages in your module, but you don’t want to make it part of your API. Go supports this via the special **internal** package name.
* When your module is small, keep all your code in a single package. As long as no other modules depend on your module, there is no harm in delaying organization.
* You can group modules into two broad categories: those that are intended as a single application and those that are primarily intended as libraries. If you are sure that your module is intended to be used only as an application, make the root of the project the main package. The code in the main package should be minimal; place all your logic in an internal directory, and the code in the main function will simply invoke code within internal. This way, you can ensure that no one is going to create a module that depends on your application’s implementation.
* If you want your module to be used as a library, the root of your module should have a package name that matches the repository name. This makes sure that the import name matches the package name.
* [Simple Go project layout with modules](https://eli.thegreenplace.net/2019/simple-go-project-layout-with-modules/)
* Package by functionality - not by layer. For example, if you wrote a shopping site in Go, you might place all the code for customer management in one package and all the code for inventory management in another.
* The “golang-standards” GitHub repository claims to be the “standard” module layout. Russ Cox, the development lead for Go, has publicly stated that it is not endorsed by the Go team and that the structure it recommends is in fact an antipattern. Please do not cite this repository as a way to organize your code.

# Go Tooling
* The `go install` command takes an argument, which is the path to the main package in a module’s source code repository, followed by an @ and the version of the tool you want (if you just want to get the latest version, use @latest). It then downloads, compiles, and installs the tool.
* It is a best practice to commit the source code created by go generate to version control. This allows people browsing your source code to see everything that’s invoked, even the generated parts.
* The go build command makes it easy to cross-compile, or create a binary for a different operating system and/or CPU. Here is how to build a binary for Linux on 64-bit Intel CPUs:`GOOS=linux GOARCH=amd64 go build`

# Concurrency in Go
* Each value written to a channel can be read only once. If multiple goroutines are reading from the same channel, a value written to the channel will be read by only one of them.
* By default, channels are unbuffered. Every write to an open, unbuffered channel causes the writing goroutine to pause until another goroutine reads from the same channel. Likewise, a read from an open, unbuffered channel causes the reading goroutine to pause until another goroutine writes to the same channel.
* Go also has buffered channels. These channels buffer a limited number of writes without blocking. If the buffer fills before there are any reads from the channel, a subsequent write to the channel pauses the writing goroutine until the channel is read. Just as writing to a channel with a full buffer blocks, reading from a channel with an empty buffer also blocks.
* Most of the time, you should use unbuffered channels.
* Anytime you are reading from a channel that might be closed, use the comma ok idiom to ensure that the channel is still open.
* The responsibility for closing a channel lies with the goroutine that writes to the channel.



# Appendix

## Errors examples

### Example 1
```
func fileChecker(name string) error {
	f, err := os.Open(name)
	defer f.Close()

	if err != nil {
		return fmt.Errorf("in fileChecker: %w", err)
	}
	return nil
}

func fileChecker2(name string) error {

	err := fileChecker(name)
	if err != nil {
		return fmt.Errorf("in fileChecker2: %w", err)
	}
	return nil
}

func main() {
	err := fileChecker2("not_here.txt")

	if err != nil {
		fmt.Println(err.Error())
		if wrap := errors.Unwrap(err); wrap != nil {
			fmt.Println(wrap)
			if wrap := errors.Unwrap(wrap); wrap != nil {
				fmt.Println(wrap)
			}
		}
		if errors.Is(err, os.ErrNotExist) {
			fmt.Println("That file doesn't exist")
		}
	}
}
```

Prints:
```
in fileChecker2: in fileChecker: open not_here.txt: The system cannot find the file specified.
in fileChecker: open not_here.txt: The system cannot find the file specified.
open not_here.txt: The system cannot find the file specified.
That file doesn't exist
```

