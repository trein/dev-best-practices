# Python coding standards

## Introduction
We use a slightly modified version of the official [Style Guide for Python Code (PEP8)](http://www.python.org/dev/peps/pep-0008/) and [Google Python Style Guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html) as a basis for our coding standards. This article describes our modifications and additions and also contains a summary of the coding standards. Reading the official style guide is highly recommended before moving on, as this article merely complements it. A code-based summary of formatting rules can be found on `sample_code.md`.

## Required reading
* [Python Best Practices](http://docs.python-guide.org/en/latest/)
* [Python Best Practice Patterns by Vladimir Keleshev](http://stevenloria.com/python-best-practice-patterns-by-vladimir-keleshev-notes/)
* [Style Guide for Python code](http://www.python.org/dev/peps/pep-0008/)
* [Google Python Style Guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html)


## Contributing
This coding standard is a work-in-progress. In order to collaborate with it, create a pull request with your change and add the motivation in the summary of your commit.

## Code Style Rules
### Naming Conventions

- "Internal" means internal to a module or protected or private within a class.
- Prepending a single underscore (`_`) has some support for protecting module variables and functions (not included with import * from). Prepending a double underscore (`__`) to an instance variable or method effectively serves to make the variable or method private to its class (using name mangling).
- Place related classes and top-level functions together in a module. Unlike Java, there is no need to limit yourself to one class per module.
- Use `CapWords` for class names, but `lower_with_under.py` for module names.
- __Names to Avoid:__ Single character names except for counters or iterators dashes (-) in any package/module name `__double_leading_and_trailing_underscore__` names (reserved by Python)

| Type                      | Public                | Internal              |
|---------------------------|-----------------------|-----------------------|
|Packages                   | `lower_with_under`      |                       |
|Modules                    | `lower_with_under`      | `_lower_with_under`     |
|Classes                    | `CapWords`              | `_CapWords`             |
|Exceptions                 | `CapWords`              |                       |
|Functions                  | `lower_with_under()`    | `_lower_with_under()`   |
|Global/Class Constants     | `CAPS_WITH_UNDER`       | `_CAPS_WITH_UNDER`      |
|Global/Class Variables     | `lower_with_under`      | `_lower_with_under`     |
|Instance Variables         | `lower_with_under`      | `_lower_with_under` (protected) or `__lower_with_under` (private) |
|Method Names               | `lower_with_under()`    | `_lower_with_under()` (protected) or `__lower_with_under()` (private) |
|Function/Method Parameters | `lower_with_under`      |                       |
|Local Variables            | `lower_with_under`      |                       |


## Language Rules

### Executable Python files
Python files which are meant to be executed directly, should have a shebang on the very first line.  If your python file is not made for direct execution, it should not have neither a shebang.

The shebang we use is: `#!/usr/bin/env python`

### Main
Even a file meant to be used as a script should be importable and a mere import should not have the side effect of executing the script's main functionality. The main functionality should be in a main() function.
In Python, pydoc as well as unit tests require modules to be importable. Your code should always check `if __name__ == '__main__'` before executing your main program so that the main program is not executed when the module is imported.

```python
def main():
      pass
      
if __name__ == '__main__':
    main()
```

### Imports
__Definition:__
Reusability mechanism for sharing code from one module to another.

__Pros:__
The namespace management convention is simple. The source of each identifier is indicated in a consistent way; `x.Obj` says that object `Obj` is defined in module `x`.

__Cons:__
Module names can still collide. Some module names are inconveniently long.

__Decision:__

* Do not use relative names in imports. Even if the module is in the same package, use the full package name. This helps prevent unintentionally importing a package twice.
* Use `import x` for importing packages and modules. 
* Use `from x import y` where `x` is the package prefix and `y` is the module name with no prefix. 
* Use `from x import y as z` if two modules named `y` are to be imported or if `y` is an inconveniently long name.

For example the module sound.effects.echo may be imported as follows:

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```


### Exceptions
__Definition:__
Exceptions are a means of breaking out of the normal flow of control of a code block to handle errors or other exceptional conditions.

__Pros:__
The control flow of normal operation code is not cluttered by error-handling code. It also allows the control flow to skip multiple frames when a certain condition occurs, e.g., returning from N nested functions in one step instead of having to carry-through error codes.

__Cons:__
May cause the control flow to be confusing. Easy to miss error cases when making library calls.

__Decision:__
Exceptions must follow certain conditions:

* Raise exceptions like this: `raise MyException('Error message')` or `raise MyException`.
* Do not use the two-argument form (`raise MyException, 'Error message'`) or deprecated string-based exceptions (`raise 'Error message'`).
* Modules or packages should define their own domain-specific base exception class, which should inherit from the built-in `Exception` class. The base exception for a module should be called `Error`.

```python
class Error(Exception):
    pass
```


* __Never__ use catch-all except: statements, or catch `Exception` or `StandardError`, unless you are re-raising the exception or in the outermost block in your thread (and printing an error message). Python is very tolerant in this regard and except: will really catch everything including misspelled names, `sys.exit()` calls, `Ctrl+C` interrupts, `unittest` failures and all kinds of other exceptions that you simply don't want to catch.
* Minimize the amount of code in a `try/except` block. The larger the body of the try, the more likely that an exception will be raised by a line of code that you didn't expect to raise an exception. In those cases, the `try/except` block hides a real error.
* Use the finally clause to execute code whether or not an exception is raised in the try block. This is often useful for cleanup, i.e., closing a file.
When capturing an exception, use as rather than a comma. For example:

```python
try:
    raise Error
except Error as error:
    pass
```


### List Comprehensions

__Definition:__
List comprehensions and generator expressions provide a concise and efficient way to create lists and iterators without resorting to the use of `map()`, `filter()`, or `lambda`.

__Pros:__
Simple list comprehensions can be clearer and simpler than other list creation techniques. Generator expressions can be very efficient, since they avoid the creation of a list entirely.

__Cons:__
Complicated list comprehensions or generator expressions can be hard to read.

__Decision:__
Okay to use for simple cases. Each portion must fit on one line: mapping expression, for clause, filter expression. Multiple for clauses or filter expressions are not permitted. Use loops instead when things get more complicated.

```python
# Yes:
  result = []
  for x in range(10):
      for y in range(5):
          if x * y > 10:
              result.append((x, y))

  for x in xrange(5):
      for y in xrange(5):
          if x != y:
              for z in xrange(5):
                  if y != z:
                      yield (x, y, z)

  return ((x, complicated_transform(x))
          for x in long_generator_function(parameter)
          if x is not None)

  squares = [x * x for x in range(10)]

  eat(jelly_bean for jelly_bean in jelly_beans
      if jelly_bean.color == 'black')
# No:
  result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]
  
  return ((x, y, z)
          for x in xrange(5)
          for y in xrange(5)
          if x != y
          for z in xrange(5)
          if y != z)
```

### Default Iterators and Operators

__Definition:__
Container types, like dictionaries and lists, define default iterators and membership test operators (`in` and `not in`).

__Pros:__
The default iterators and operators are simple and efficient. They express the operation directly, without extra method calls. A function that uses default operators is generic. It can be used with any type that supports the operation.

__Cons:__
You can't tell the type of objects by reading the method names (e.g. `has_key()` means a dictionary). This is also an advantage.

__Decision:__
Use default iterators and operators for types that support them, like lists, dictionaries, and files. The built-in types define iterator methods, too. Prefer these methods to methods that return lists, except that you should not mutate a container while iterating over it.

```python
# Yes:
      for key in adict: ...
      if key not in adict: ...
      if obj in alist: ...
      for line in afile: ...
      for k, v in dict.iteritems(): ...

# No:  
      for key in adict.keys(): ...
      if not adict.has_key(key): ...
      for line in afile.readlines(): ...
```

### Conditional Expressions

__Definition:__
Conditional expressions are mechanisms that provide a shorter syntax for if statements. For example: `x = 1 if cond else 2`.

__Pros:__
Shorter and more convenient than an if statement.

__Cons:__
May be harder to read than an if statement. The condition may be difficult to locate if the expression is long.

__Decision:__
Okay to use for one-liners. In other cases prefer to use a complete if statement.

### Default Argument Values

__Definition:__
You can specify values for variables at the end of a function's parameter list, e.g., def foo(a, b=0):. If foo is called with only one argument, b is set to 0. If it is called with two arguments, b has the value of the second argument.

__Pros:__
Often you have a function that uses lots of default values, but—rarely—you want to override the defaults. Default argument values provide an easy way to do this, without having to define lots of functions for the rare exceptions. Also, Python does not support overloaded methods/functions and default arguments are an easy way of "faking" the overloading behavior.

__Cons:__
Default arguments are evaluated once at module load time. This may cause problems if the argument is a mutable object such as a list or a dictionary. If the function modifies the object (e.g., by appending an item to a list), the default value is modified.

__Decision:__
Okay to use with the following caveat: __Do not use mutable objects as default values in the function or method definition__.

```python
# Yes:
def foo(a, b=None):
    if b is None:
        b = []

# No:
def foo(a, b=[]):
    ...

# No:
def foo(a, b=time.time()):     # The time the module was loaded???
    ...

# No:
def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
```

### Properties
__Definition:__
A way to wrap method calls for getting and setting an attribute as a standard attribute access when the computation is lightweight.

__Pros:__
Readability is increased by eliminating explicit get and set method calls for simple attribute access. Allows calculations to be lazy. Considered the Pythonic way to maintain the interface of a class. In terms of performance, allowing properties bypasses needing trivial accessor methods when a direct variable access is reasonable. This also allows accessor methods to be added in the future without breaking the interface.

__Cons:__
Properties are specified after the getter and setter methods are declared, requiring one to notice they are used for properties farther down in the code (except for readonly properties created with the `@property` decorator - see below). Must inherit from `object`. Can hide side-effects much like operator overloading. Can be confusing for subclasses.

__Decision:__
Use properties in new code to access or set data where you would normally have used simple, lightweight accessor or setter methods. Read-only properties should be created with the `@property` decorator.

Inheritance with properties can be non-obvious if the property itself is not overridden. Thus one must make sure that accessor methods are called indirectly to ensure methods overridden in subclasses are called by the property (using the [Template Method DP](http://en.wikipedia.org/wiki/Template_method_pattern)).


```python
# Yes:
import math
class Square(object):
     """A square with two properties: a writable area and a read-only perimeter.

     To use:
     >>> sq = Square(3)
     >>> sq.area
     9
     >>> sq.perimeter
     12
     >>> sq.area = 16
     >>> sq.side
     4
     >>> sq.perimeter
     16
     """

     def __init__(self, side):
         self.side = side

     def __get_area(self):
         """Calculates the 'area' property."""
         return self.side ** 2

     def ___get_area(self):
         """Indirect accessor for 'area' property."""
         return self.__get_area()

     def __set_area(self, area):
         """Sets the 'area' property."""
         self.side = math.sqrt(area)

     def ___set_area(self, area):
         """Indirect setter for 'area' property."""
         self.__set_area(area)

     area = property(___get_area, ___set_area,
                     doc="""Gets or sets the area of the square.""")

     @property
     def perimeter(self):
         return self.side * 4
```

### True/False evaluations

__Definition:__
Python evaluates certain values as false when in a boolean context. A quick "rule of thumb" is that all "empty" values are considered false so 0, None, [], {}, '' all evaluate as false in a boolean context.

__Pros:__
Conditions using Python booleans are easier to read and less error-prone. In most cases, they're also faster.

__Cons:__
May look strange to C/C++ developers.

__Decision:__
Use the "implicit" false if at all possible, e.g., if foo: rather than if foo != []:. There are a few caveats that you should keep in mind though:

Never use == or != to compare singletons like None. Use is or is not.
Beware of writing if x: when you really mean if x is not None:—e.g., when testing whether a variable or argument that defaults to None was set to some other value. The other value might be a value that's false in a boolean context!
Never compare a boolean variable to False using ==. Use if not x: instead. If you need to distinguish False from None then chain the expressions, such as if not x and x is not None:.
For sequences (strings, lists, tuples), use the fact that empty sequences are false, so if not seq: or if seq: is preferable to if len(seq): or if not len(seq):.
When handling integers, implicit false may involve more risk than benefit (i.e., accidentally handling None as 0). You may compare a value which is known to be an integer (and is not the result of len()) against the integer 0.

```python
# Yes:
if not users:
    print 'no users'

if foo == 0:
    self.handle_zero()

if i % 10 == 0:
    self.handle_multiple_of_ten()

# No:
if len(users) == 0:
    print 'no users'

if foo is not None and not foo:
    self.handle_zero()

if not i % 10:
    self.handle_multiple_of_ten()
```
Note that '0' (i.e., 0 as string) evaluates to true.