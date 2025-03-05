* Chapter 7. Most type conversions are checked at compile time, so if they are invalid, your code won’t compile. (Type conversions between slices and array pointers can fail at runtime and don’t support the comma ok idiom, so be careful when using them!)




# Errors
* Error messages should not be capitalized nor should they end with punctuation or a newline. In most cases, you should set the other return values to their zero values when a non-nil error is returned.
* 
