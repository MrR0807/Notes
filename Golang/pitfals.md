* Chapter 7. Most type conversions are checked at compile time, so if they are invalid, your code won’t compile. (Type conversions between slices and array pointers can fail at runtime and don’t support the comma ok idiom, so be careful when using them!)

TODO
Intefaces and nil 


# Errors
* Error messages should not be capitalized nor should they end with punctuation or a newline. In most cases, you should set the other return values to their zero values when a non-nil error is returned.
* The second situation is a reused err variable. The Go compiler requires every variable to be read at least once. It doesn’t require that every write to a variable is read. If you use an err variable multiple times, you have to read it only once to make the compiler happy. **Use tooling to detect**.
* Sentinel errors are one of the few variables that are declared at the package level. By convention, their names start with Err (with the notable exception of io.EOF). They should be treated as read-only; there’s no way for the Go compiler to enforce this, but it is a programming error to change their value. Sentinel errors are usually used to indicate that you cannot start or continue processing. For example, the standard library includes a package for processing ZIP files, archive/zip. This package defines several sentinel errors, including ErrFormat, which is returned when data that doesn’t represent a ZIP file is passed in. Be sure you need a sentinel error before you define one. Once you define one, it is part of your public API, and you have committed to it being available in all future backward-compatible releases. It’s far better to reuse one of the existing ones in the standard library or to define an error type that includes information about the condition that caused the error to be returned.
* Even when you define your own custom error types, always use error as the return type for the error result.
* When using custom errors, never define a variable to be of the type of your custom error. Either explicitly return nil when no error occurs or define the variable to be of type error.

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

