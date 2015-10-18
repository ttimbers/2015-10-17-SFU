---
layout: page
title: Programming with Python
subtitle: Command-Line Programs
minutes: 30
---
## Learning Objectives 

 *   Use the values of command-line arguments in a program.
 *   Handle flags and files separately in a command-line program.
 *   Read data from standard input in a program so that it can be used in a pipeline.

The IPython Notebook and other interactive tools are great for prototyping code and exploring data,
but sooner or later we will want to use our program in a pipeline
or run it in a shell script to process thousands of data files.
In order to do that,
we need to make our programs work like other Unix command-line tools.
For example,
we may want a program that reads a dataset
and prints the average inflammation per patient.

This program does exactly what we want - it prints the average inflammation per patient
for a given file.

~~~
$ python readings.py --mean inflammation-01.csv
5.45
5.425
6.1
...
6.4
7.05
5.9
~~~

We might also want to look at the minimum of the first four lines

~~~
$ head -4 inflammation-01.csv | python readings.py --min
~~~

or the maximum inflammations in several files one after another:

~~~
$ python readings.py --max inflammation-*.csv
~~~

Our scripts should do the following:

1. If no filename is given on the command line, read data from standard input.
2. If one or more filenames are given, read data from them and report statistics for each file separately.
3. Use the `--min`, `--mean`, or `--max` flag to determine what statistic to print.

To make this work,
we need to know how to handle command-line arguments in a program,
and how to get at standard input.
We'll tackle these questions in turn below.

## Command-Line Arguments

Using the text editor of your choice,
save the following in a text file called `sys-version.py`:

~~~ 
import sys
print('version is', sys.version)
~~~

The first line imports a library called `sys`,
which is short for "system".
It defines values such as `sys.version`,
which describes which version of Python we are running.
We can run this script from the command line like this:

~~~ 
$ python sys-version.py
~~~


Create another file called `argv-list.py` and save the following text to it.

~~~ 
import sys
print('sys.argv is', sys.argv)
~~~

The strange name `argv` stands for "argument values".
Whenever Python runs a program,
it takes all of the values given on the command line
and puts them in the list `sys.argv`
so that the program can determine what they were.
If we run this program with no arguments:

~~~ 
$ python argv-list.py
~~~

the only thing in the list is the full path to our script,
which is always `sys.argv[0]`.
If we run it with a few arguments, however:

~~~ 
$ python argv-list.py first second third
~~~

then Python adds each of those arguments to that magic list.

With this in hand,
let's build a version of `readings.py` that always prints the per-patient mean of a single data file.
The first step is to write a function that outlines our implementation,
and a placeholder for the function that does the actual work.
By convention this function is usually called `main`,
though we can call it whatever we want:

~~~ 
$ cat readings-01.py
~~~

~~~ 
import sys
import numpy

def main():
    script = sys.argv[0]
    filename = sys.argv[1]
    data = numpy.loadtxt(filename, delimiter=',')
    for m in data.mean(axis=1):
        print(m)
~~~

This function gets the name of the script from `sys.argv[0]`,
because that's where it's always put,
and the name of the file to process from `sys.argv[1]`.
Here's a simple test:

~~~ 
$ python readings-01.py inflammation-01.csv
~~~

There is no output because we have defined a function,
but haven't actually called it.
Let's add a call to `main`:

~~~ 
$ cat readings-02.py
~~~

~~~ 
import sys
import numpy

def main():
    script = sys.argv[0]
    filename = sys.argv[1]
    data = numpy.loadtxt(filename, delimiter=',')
    for m in data.mean(axis=1):
        print(m)

main()
~~~

and run that:

~~~ 
$ python readings-02.py inflammation-01.csv
~~~


## Handling Multiple Files

The next step is to teach our program how to handle multiple files.
Since 60 lines of output per file is a lot to page through,
we'll start by using three smaller files,
each of which has three days of data for two patients:

~~~ 
$ ls small-*.csv
~~~

~~~
$ cat small-01.csv
~~~


~~~ 
$ python readings-02.py small-01.csv
~~~


Using small data files as input also allows us to check our results more easily:
here,
for example,
we can see that our program is calculating the mean correctly for each line,
whereas we were really taking it on faith before.
This is yet another rule of programming:
*test the simple things first*.

We want our program to process each file separately,
so we need a loop that executes once for each filename.
If we specify the files on the command line,
the filenames will be in `sys.argv`,
but we need to be careful:
`sys.argv[0]` will always be the name of our script,
rather than the name of a file.
We also need to handle an unknown number of filenames,
since our program could be run for any number of files.

The solution to both problems is to loop over the contents of `sys.argv[1:]`.
The '1' tells Python to start the slice at location 1,
so the program's name isn't included;
since we've left off the upper bound,
the slice runs to the end of the list,
and includes all the filenames.
Here's our changed program
`readings-03.py`:

~~~ 
$ cat readings-03.py
~~~

~~~ 
import sys
import numpy

def main():
    script = sys.argv[0]
    for filename in sys.argv[1:]:
        data = numpy.loadtxt(filename, delimiter=',')
        for m in data.mean(axis=1):
            print(m)

main()
~~~

and here it is in action:

~~~ 
$ python readings-03.py small-01.csv small-02.csv
~~~



## Handling Command-Line Flags

The next step is to teach our program to pay attention to the `--min`, `--mean`, and `--max` flags.
These always appear before the names of the files,
so we could just do this:

~~~ 
$ cat readings-04.py
~~~

~~~ 
import sys
import numpy

def main():
    script = sys.argv[0]
    action = sys.argv[1]
    filenames = sys.argv[2:]

    for f in filenames:
        data = numpy.loadtxt(f, delimiter=',')

        if action == '--min':
            values = data.min(axis=1)
        elif action == '--mean':
            values = data.mean(axis=1)
        elif action == '--max':
            values = data.max(axis=1)

        for m in values:
            print(m)

main()
~~~

This works:

~~~
$ python readings-04.py --max small-01.csv
~~~

but there are several things wrong with it:

1.  `main` is too large to read comfortably.

2.  If `action` isn't one of the three recognized flags,
    the program loads each file but does nothing with it
    (because none of the branches in the conditional match).
    [Silent failures](reference.html#silence-failure) like this
    are always hard to debug.

This version pulls the processing of each file out of the loop into a function of its own.
It also checks that `action` is one of the allowed flags
before doing any processing,
so that the program fails fast:

~~~ 
$ cat readings-05.py
~~~

~~~ 
import sys
import numpy

def main():
    script = sys.argv[0]
    action = sys.argv[1]
    filenames = sys.argv[2:]
    assert action in ['--min', '--mean', '--max'], \
           'Action is not one of --min, --mean, or --max: ' + action
    for f in filenames:
        process(f, action)

def process(filename, action):
    data = numpy.loadtxt(filename, delimiter=',')

    if action == '--min':
        values = data.min(axis=1)
    elif action == '--mean':
        values = data.mean(axis=1)
    elif action == '--max':
        values = data.max(axis=1)

    for m in values:
        print(m)

main()
~~~

This is four lines longer than its predecessor,
but broken into more digestible chunks of 8 and 12 lines.

Python has a module named [argparse](http://docs.python.org/dev/library/argparse.html)
that helps handle complex command-line flags. We will not cover this module in this lesson
but you can go to Tshepang Lekhonkhobe's [Argparse tutorial](http://docs.python.org/dev/howto/argparse.html)
that is part of Python's Official Documentation.

## Handling Standard Input

The next thing our program has to do is read data from standard input if no filenames are given
so that we can put it in a pipeline,
redirect input to it,
and so on.
Let's experiment in another script called `count-stdin.py`:

~~~ 
$ cat count-stdin.py
~~~

~~~ 
import sys

count = 0
for line in sys.stdin:
    count += 1

print(count, 'lines in standard input')
~~~

This little program reads lines from a special "file" called `sys.stdin`,
which is automatically connected to the program's standard input.
We don't have to open it --- Python and the operating system
take care of that when the program starts up ---
but we can do almost anything with it that we could do to a regular file.
Let's try running it as if it were a regular command-line program:

~~~ 
$ python count-stdin.py < small-01.csv
~~~

~~~ 
2 lines in standard input
~~~

A common mistake is to try to run something that reads from standard input like this:

~~~ 
$ count_stdin.py small-01.csv
~~~

i.e., to forget the `<` character that redirect the file to standard input.
In this case,
there's nothing in standard input,
so the program waits at the start of the loop for someone to type something on the keyboard.
Since there's no way for us to do this,
our program is stuck,
and we have to halt it using the `Interrupt` option from the `Kernel` menu in the Notebook.

We now need to rewrite the program so that it loads data from `sys.stdin` if no filenames are provided.
Luckily,
`numpy.loadtxt` can handle either a filename or an open file as its first parameter,
so we don't actually need to change `process`.
That leaves `main`:

~~~ 
$ cat readings-06.py
~~~

~~~ 
def main():
    script = sys.argv[0]
    action = sys.argv[1]
    filenames = sys.argv[2:]
    assert action in ['--min', '--mean', '--max'], \
           'Action is not one of --min, --mean, or --max: ' + action
    if len(filenames) == 0:
        process(sys.stdin, action)
    else:
        for f in filenames:
            process(f, action)
~~~

Let's try it out:

~~~ 
$ python readings-06.py --mean small-01.csv
~~~


That's better.
In fact,
that's done:
the program now does everything we set out to do.
