* Ruby uses a convention that may seem strange at first: the first characters of a name indicate how broadly the variable is visible. Local variables, method parameters, and method names should all start with a lowercase letter or an underscore (Ruby itself has a couple of methods that start with a capital letter, but in general this isn’t something to do in your own code). Global variables are prefixed with a dollar sign, $, and instance variables begin with an “at” sign, @. Class variables start with two “at” signs, @@. Although we talk about global and class variables here for completeness, you’ll find they are rarely used in Ruby programs. There’s a lot of evidence that global variables make programs harder to maintain. Class variables aren’t as dangerous as global variables, but they are still tricky to use safely—people tend not to use them much because they often use easier ways to get similar functionality. Finally, class names, module names, and other constants must start with an uppercase letter.

#### Safe Navigation 

It’s common to have a chain of method calls on a series of objects. The way the &. operator works is that if the receiver of the message on the left side (in this case data[:name]) is nil, then the message isn’t sent and the nil value is returned without raising an exception. If the receiver isn’t nil, then the message is processed normally.

```
data[:name]&.upcase
```

The safe navigation operator’s powers only last for the one message. If you want to continue with more downstream messages, you need more safe navigation operators.

```
data[:name]&.upcase&.strip&.split
```
