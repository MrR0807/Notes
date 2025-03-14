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
* This is so common that the combination is often referred to as a for-select loop. When using a for-select loop, you must include a way to exit the loop.
```
for {
  select {
    case <-done:
      return
    case v := <-ch:
      fmt.Println(v)
  }
}
```
* Having a default case inside a for-select loop is almost always the wrong thing to do. It will be triggered every time through the loop when there’s nothing to read or write for any of the cases. This makes your for loop run constantly, which uses a great deal of CPU.
* Practically, this means that you should never expose channels or mutexes in your API’s types, functions, and methods.
* Anytime a closure uses a variable whose value might change, use a parameter to pass a copy of the variable’s current value into the closure.
* Buffered channels are useful when you know how many goroutines you have launched, want to limit the number of goroutines you will launch, or want to limit the amount of work that is queued up.
* Implementing backpressure:

```
type PressureGauge struct {
	ch chan struct{}
}
func New(limit int) *PressureGauge {
	return &PressureGauge{
		ch: make(chan struct{}, limit),
	}
}
func (pg *PressureGauge) Process(f func()) error {
	select {
	case pg.ch <- struct{}{}:
		f()
		<-pg.ch
		return nil
	default:
		return errors.New("no more capacity")
	}
}
```
* When you need to combine data from multiple concurrent sources, the select keyword is great. However, you need to properly handle closed channels. If one of the cases in a select is reading a closed channel, it will always be successful, returning the zero value. Every time that case is selected, you need to check to make sure that the value is valid and skip the case. If reads are spaced out, your program is going to waste a lot of time reading junk values. While that is bad if it is triggered by a bug, you can use a nil channel to disable a case in a select. When you detect that a channel has been closed, set the channel’s variable to nil. The associated case will no longer run, because the read from the nil channel never returns a value. Here is a for-select loop that reads from two channels until both are closed:
```
for count := 0; count < 2; {
  select {
    case v, ok := <-in:
      if !ok {
	in = nil // the case will never succeed again!
	count++
	continue
       }
	// process the v that was read from in
    case v, ok := <-in2:
      if !ok {
	in2 = nil // the case will never succeed again!
	count++
	continue
    }
    // process the v that was read from in2
  }
}
```
* Time Out Code:
```
func timeLimit[T any](worker func() T, limit time.Duration) (T, error) {
	out := make(chan T, 1)
	ctx, cancel := context.WithTimeout(context.Background(), limit)
	defer cancel()
	go func() {
		out <- worker()
	}()
	select {
	case result := <-out:
		return result, nil
	case <-ctx.Done():
		var zero T
		return zero, errors.New("work timed out")
	}
}
```
* If you are waiting on several goroutines, you need to use a WaitGroup, which is found in the sync package in the standard library.

```
func main() {
	var wg sync.WaitGroup
	wg.Add(3)
	go func() {
		defer wg.Done()
		doThing1()
	}()
	go func() {
		defer wg.Done()
		doThing2()
	}()
	go func() {
		defer wg.Done()
		doThing3()
	}()
	wg.Wait()
}
```
* While WaitGroups are handy, they shouldn’t be your first choice when coordinating goroutines. Use them only when you have something to clean up (like closing a channel they all write to) after all your worker goroutines exit.
* Run Code Exactly Once. The sync package includes a handy type called Once that enables this functionality. As with sync.WaitGroup, you must make sure not to make a copy of an instance of sync.Once, because each copy has its own state to indicate whether it has already been used. In the example, you want to make sure that parser is initialized only once, so you set the value of parser from within a closure that’s passed to the Do method on once. If Parse is called more than once, once.Do will not execute the closure again.

```
var parser SlowComplicatedParser
var once sync.Once
func Parse(dataToParse string) string {
  once.Do(func() {
    parser = initParser()
  })
return parser.Parse(dataToParse)
}
```
* When to Use Mutexes Instead of Channels - the most common case is when your goroutines read or write a shared value, but don’t process the value.
* If you need to squeeze out every last bit of performance and are an expert on writing concurrent code, you’ll be glad that Go includes atomic support. For everyone else, use goroutines and mutexes to manage your concurrency needs.
* Do not use `sync.Map`. Given these limitations, in the rare situations where you need to share a map across multiple goroutines, use a built-in map protected by a sync.RWMutex.

# The Standard Library

* The io.Seeker interface is used for random access to a resource. The valid values for whence are the constants io.SeekStart, io.SeekCurrent, and io.SeekEnd. This should have been made clearer by using a custom type, but in a surprising design oversight, whence is of type int.
* When working with larger data sources, use the Create, NewFile, Open, and OpenFile functions in the os package. They return an *os.File instance, which implements the io.Reader and io.Writer interfaces. You can use an *os.File instance with the Scanner type in the bufio package.
* The time.After function returns a channel that outputs once, while the channel returned by time.Tick returns a new value every time the specified time.Duration elapses. These are used with Go’s concurrency support to enable timeouts or recurring tasks. You can also trigger a single function to run after a specified time.Duration with the time.AfterFunc function. **Don’t use time.Tick outside trivial programs**, because the underlying time.Ticker cannot be shut down (and therefore cannot be garbage collected). Use the time.NewTicker function instead, which returns a *time.Ticker that has the channel to listen to, as well as methods to reset and stop the ticker.
* encoding/json - Struct tags are composed of one or more tag/value pairs, written as tagName:"tagValue" and separated by spaces. Because they are just strings, the compiler cannot validate that they are formatted correctly, but go vet does.
* It's best to use the struct tag to specify the name of the field explicitly, even if the field names are identical.
* To limit the amount of code that cares about what your JSON looks like, define two structs. Use one for converting to and from JSON and the other for data processing. Read in JSON to your JSON-aware type, and then copy it to the other. When you want to write out JSON, do the reverse. This does create some duplication, but it keeps your business logic from depending on wire protocols.
* Structured Logging - use `log/slog`

# The Context
* Go has another convention that the context is explicitly passed through your program as the first parameter of a function.
* Another function, context.TODO, also creates an empty con text.Context. It is intended for temporary use during development. If you aren’t sure where the context is going to come from or how it’s going to be used, use context.TODO to put a placeholder in your code. Production code shouldn’t include context.TODO.
* The value stored in the context can be of any type, but picking the correct key is important. Like the key for a map, the key for a context value must be comparable. Don’t just use a string like "id". If you use string or another predefined or exported type for the type of the key, different packages could create identical keys, resulting in collisions. This causes problems that are hard to debug, such as one package writing data to the context that masks the data written by another package, or reading data from the context that was written by another package.
* The name of the function that creates a context with the value should start with ContextWith. The function that returns the value from the context should have a name that ends with FromContext.
* There are two ways to add a key to context - iota or struct. However, both ways need to ensure that the key is not exported, this way, there is no way for key collision.
* Since contexts with timers can cancel because of a timeout or an explicit call to the cancellation function, the context API provides a way to tell what caused cancellation. The Err method returns nil if the context is still active, or it returns one of two sentinel errors if the context has been canceled: context.Canceled or context.DeadlineExceeded. The first is returned after explicit cancellation, and the second is returned when a timeout triggered cancellation.
* Most of the time, you don’t need to worry about timeouts or cancellation within your own code; it simply doesn’t run for long enough. Whenever you call another HTTP service or the database, you should pass along the context; those libraries properly handle cancellation via the context.
* You should think about handling cancellation for two situations. The first is when you have a function that reads or writes channels by using a select statement - include a case that checks the channel returned by the Done method on the context; The second situation is when you write code that runs long enough that it should be interrupted by a context cancellation. In that case, check the status of the context periodically using context.Cause.

```
func longRunningComputation(ctx context.Context, data string) (string, error) {
	for {
		// do some processing
		// insert this if statement periodically
		// to check if the context has been cancelled
		if err := context.Cause(ctx); err != nil {
			// return a partial value if it makes sense,
			// or a default one if it doesn't
			return "", err
		}
		// do some more processing and loop again
	}
}
```

# Testing
* Every test is written in a file whose name ends with _test.go. If you are writing tests against foo.go, place your tests in a file named foo_test.go.
* Test functions start with the word Test and take in a single parameter of type *testing.T. By convention, this parameter is named t. Test functions do not return any values. The name of the test (apart from starting with the word “Test”) is meant to document what you are testing, so pick something that explains what you are testing. When writing unit tests for individual functions, the convention is to name the unit test Test followed by the name of the function. When testing unexported functions, some people use an underscore between the word Test and the name of the function.
* Also note that you use standard Go code to call the code being tested and to validate that the responses are as expected. When there’s an incorrect result, you report the error with the t.Error method, which works like the fmt.Print function. You’ll see other error-reporting methods in a bit.
* When should you use Fatal/Fatalf and when should you use Error/Errorf? If the failure of a check in a test means that further checks in the same test function will always fail or cause the test to panic, use Fatal or Fatalf. If you are testing several independent items (such as validating fields in a struct), then use Error or Errorf so you can report many problems at once. This makes it easier to fix multiple problems without rerunning your tests over and over.
* Both TestFirst and TestSecond refer to the package-level variable testTime. Assume that it needs to be initialized in order for the tests to run properly. You declare a function called TestMain with a parameter of type *testing.M. If there’s a function named TestMain in a package, go test calls it instead of the test functions. It is the responsibility of the TestMain function to set up any state that’s necessary to make the tests in the package run correctly. Once the state is configured, the TestMain function calls the Run method on *testing.M. This runs the test functions in the package. The Run method returns the exit code; 0 indicates that all tests passed. Finally, the TestMain function must call os.Exit with the exit code returned from Run.

```
var testTime time.Time

func TestMain(m *testing.M) {
	fmt.Println("Set up stuff for tests here")
	testTime = time.Now()
	exitVal := m.Run()
	fmt.Println("Clean up stuff after tests here")
	os.Exit(exitVal)
}
func TestFirst(t *testing.T) {
	fmt.Println("TestFirst uses stuff set up in TestMain", testTime)
}
func TestSecond(t *testing.T) {
	fmt.Println("TestSecond also uses stuff set up in TestMain", testTime)
}
```
* The Cleanup method on *testing.T is used to clean up temporary resources created for a single test. This method has a single parameter, a function with no input parameters or return values. The function runs when the test completes.
* It’s a common (and very good) practice to configure applications with environment variables. To help you test your environment-variable–parsing code, Go provides a helper method on testing.T. Call t.Setenv() to register a value for an environment variable for your test. Behind the scenes, it calls Cleanup to revert the environment variable to its previous state when the test exits
* While it is good to use environment variables to configure your application, it is also good to make sure that most of your code is entirely unaware of them. Be sure to copy the values of environment variables into configuration structs before your program starts its work, in your main function or soon afterward. Rather than writing this code yourself, you should strongly consider using a third-party configuration library, like Viper or envconfig. Also, look at GoDotEnv as a way to store environment variables in .env files for development or continuous integration machines.
* As go test walks your source code tree, it uses the current package directory as the current working directory. If you want to use sample data to test functions in a package, create a subdirectory named testdata to hold your files. Go reserves this directory name as a place to hold test files.
* If you want to test just the public API of your package, Go has a convention for specifying this. You still keep your test source code in the same directory as the production source code, but you use packagename_test for the package name.

```
package pubadder

func AddNumbers(x, y int) int {
	return x + y
}
```

```
package pubadder_test

import (
	"github.com/learning-go-book-2e/ch15/sample_code/pubadder"
	"testing"
)

func TestAddNumbers(t *testing.T) {
	result := pubadder.AddNumbers(2, 3)
	if result != 5 {
		t.Error("incorrect result: expected 5, got", result)
	}
}
```


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

