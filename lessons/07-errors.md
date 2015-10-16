---
layout: page
title: Programming with Python
subtitle: Errors and Exceptions
minutes: 30
---
## Learning Objectives

 *   To be able to read a traceback, and determine the following relevant pieces of information:
     * The file, function, and line number on which the error occurred
     * The type of the error
     * The error message
 *   To be able to describe the types of situations in which the following errors occur:
     * `SyntaxError` and `IndentationError`
     * `NameError`
     * `IndexError`
     * `FileNotFoundError`


Errors in Python have a very specific form, called a traceback.
Let's examine one:

~~~ 
import errors_01
errors_01.favorite_ice_cream()
~~~


## Syntax Errors

When you forget a colon at the end of a line,
accidentally add one space too many when indenting under an `if` statement,
or forget a parentheses,
you will encounter a syntax error.

If Python doesn't know how to read the program,
it will just give up and inform you with an error.
For example:

~~~ 
def some_function()
    msg = "hello, world!"
    print(msg)
     return msg
~~~


~~~ 
def some_function():
    msg = "hello, world!"
    print(msg)
     return msg
~~~


## Variable Name Errors

Another very common type of error is called a `NameError`,
and occurs when you try to use a variable that does not exist.
For example:

~~~ 
print(a)
~~~

Variable name errors come with some of the most informative error messages,
which are usually of the form "name 'the_variable_name' is not defined".

Why does this error message occur?
That's harder question to answer,
because it depends on what your code is supposed to do.
However,
there are a few very common reasons why you might have an undefined variable.
The first is that you meant to use a string, but forgot to put quotes around it:

~~~ 
print(hello)
~~~


The second is that you just forgot to create the variable before using it.
In the following example,
`count` should have been defined (e.g., with `count = 0`) before the for loop:

~~~ 
for number in range(10):
    count = count + number
print("The count is: " + str(count))
~~~

Finally, the third possibility is that you made a typo when you were writing your code.
Let's say we fixed the error above by adding the line `Count = 0` before the for loop.
Frustratingly, this actually does not fix the error.
Remember that variables are [case-sensitive](reference.html#case-sensitive),
so the variable `count` is different from `Count`. We still get the same error, because we still have not defined `count`:

~~~ 
Count = 0
for number in range(10):
    count = count + number
print("The count is: " + str(count))
~~~

## Item Errors

Next up are errors having to do with containers (like lists and dictionaries) and the items within them.
If you try to access an item in a list or a dictionary that does not exist,
then you will get an error.


~~~ 
letters = ['a', 'b', 'c']
print("Letter #1 is " + letters[0])
print("Letter #2 is " + letters[1])
print("Letter #3 is " + letters[2])
print("Letter #4 is " + letters[3])
~~~

Here,
Python is telling us that there is an `IndexError` in our code, meaning we tried to access a list index that did not exist.

## File Errors

The last type of error we'll cover today are those associated with reading and writing files: `FileNotFoundError`.
If you try to read a file that does not exist,
you will recieve an `FileNotFoundError` telling you so.

~~~ 
file_handle = open('myfile.txt', 'r')
~~~

One reason for receiving this error is that you specified an incorrect path to the file.
For example,
if I am currently in a folder called `myproject`,
and I have a file in `myproject/writing/myfile.txt`,
but I try to just open `myfile.txt`,
this will fail.
The correct path would be `writing/myfile.
xt`. It is also possible (like with `NameError`) that you just made a typo.

Another issue could be that you used the "read" flag instead of the "write" flag.
Python will not give you an error if you try to open a file for writing when the file does not exist.
However,
if you meant to open a file for reading,
but accidentally opened it for writing,
and then try to read from it,
you will get an `UnsupportedOperation` error
telling you that the file was not opened for reading:

~~~ 
file_handle = open('myfile.txt', 'w')
file_handle.read()
~~~

These are the most common errors with files,
though many others exist.
If you get an error that you've never seen before,
searching the Internet for that error type
often reveals common reasons why you might get that error.
