---
title: "Functional Programming"
teaching: 0
exercises: 60
questions:
- "What is functional programming?"
- "What is recursion?"
- "How does functional programming manipulate data?"
objectives:
- "Write recursive functions to calculate simple sequences"
- "Use the MapReduce model to process data"
keypoints:
- "Pure functions are functions which have no side effects"
- "Recursion allows us to calculate a value in a sequence from previous values"
- "The MapReduce model is a common way of processing large amounts of data"
---

## The Functional Paradigm

We don't have any new slides or intro video today, since we're continuing from yesterday's Programming Paradigms.

If you want to refresh your memory though, here's the [video](https://youtu.be/YC4ohR5Pf5Q) and [slides](../slides/1.2-Programming-Paradigms.pptx) from yesterday.

## Set up training materials

So let's download the training materials for this material from the GitHub code repository online. Go to [https://github.com/SABS-R3/2020-software-engineering-day2/tree/gh-pages](https://github.com/SABS-R3/2020-software-engineering-day2/tree/gh-pages) in a browser (any will do, although Firefox is already installed on the provided laptops). Select the green `Code` button, and then select `Download ZIP`, and then in Firefox selecting `Save File` at the dialogue prompt. This will download all the files within a single archive file. After it's finished downloading, we need to extract all files from the archive. Find where the file has been downloaded to (on the provided laptops this is `/home/sabsr3/Downloads`, then start a terminal. You can start a terminal by right-clicking on the desktop and selecting `Open in Terminal`. Assuming the file has downloaded to e.g. `/home/sabsr3/Downloads`, type the following within the Terminal shell:

~~~
cd ~
unzip /home/sabsr3/Downloads/2020-software-engineering-day2-gh-pages.zip
~~~
{: .language-bash}

The first `cd ~` command *changes our working directory* to our home directory (on the provisioned laptops, this is `/home/sabsr3`).

The second command uses the unzip program to unpack the archive in your home directory, within a subdirectory called `2020-software-engineering-day2-gh-pages`. This subdirectory name is a little long to easily work with, so we'll rename it to something shorter:

~~~
mv 2020-software-engineering-day2-gh-pages 2020-se-day2
~~~
{: .language-bash}

Change to the `code` directory within that new directory:

~~~
cd 2020-se-day2/code
~~~
{: .language-bash}

> ## Functions in Maths
> > In mathematics, a function is a relation between sets that associates to every element of a first set exactly one element of the second set.
> >
> > -- Wikipedia - [Function (mathematics)](https://en.wikipedia.org/wiki/Function_(mathematics))
{: .callout}

The Functional paradigm is based on mathematical functions.
A program written in a functional style describes a series of operations which are performed on data to produce a desired output, the focus being on **what** rather than **how**.

You will likely encounter functional programming in the future in data analysis code, or if you use frameworks such as Hadoop, or languages like R.
In fact, there's a good [section on functional programming](https://adv-r.hadley.nz/fp.html) in Hadley Wickham's Advanced R book.

Though we are teaching Functional Programming after Object Oriented Programming, it does not build on Object Oriented as Object Oriented built on Procedural Programming.
It belongs to a different branch in the history of paradigms, the Declarative family.

> ## Functional Style
>
> In his introduction to functional programming in Advanced R, Hadley Wickham gives a good summary of the style:
>
> > It’s hard to describe exactly what a functional style is, but generally I think it means decomposing a big problem into smaller pieces, then solving each piece with a function or combination of functions.
> > When using a functional style, you strive to decompose components of the problem into isolated functions that operate independently.
> > Each function taken by itself is simple and straightforward to understand; complexity is handled by composing functions in various ways.
> >
> > -- Hadley Wickham - [Functional Style](https://adv-r.hadley.nz/fp.html)
{: .callout}

## Side Effect and Pure Functions

There's nothing particularly special about the behaviour of functions in Python, almost any valid code can be put inside a function to be called from elsewhere (even defining classes or other functions).
This means that each function can do anything the Python language can do.

In well designed code, a function should only be responsible for one task (see: [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)).
As with most guidelines, the reason we do this is to make it easier to reason about the behaviour of our code.
If a function performs only one task, we can be sure about when it is appropriate to use that function and what will happen when we call it.

Any behaviour which is not part of the single task of a block of code is called a **side effect**, while a function which has no side effects is a **pure function**.
In practice these definitions are a little flexible depending on who is using them and in which context.
The strictest definition of **side effect** includes things like printing to the terminal or saving to a file, so in these cases a function without side effects have no other effect than to return a value.
In some functional languages, a **pure function** must not only have no side effects, but also must return the same value when given the same arguments.

~~~
def increment_x(x):
    return x + 1

print(increment_x(3))
~~~
{: .language-python}

~~~
4
~~~
{: .output}

~~~
def increment_x(x):
    print('Incrementing', x)
    return x + 1

print(increment_x(3))
~~~
{: .language-python}

~~~
Incrementing 3
4
~~~
{: .output}


## Recursion

See: Recursion

Recursion is one of the common strategies used in Functional Programming.
Instead of using loops to **iteratively** apply an operation, we can express a result in terms of previous results.
To do this, the function needs to call itself to get the previous result, this is called **recursion**.

<a href="{{ page.root }}/fig/droste-effect.jpg">
  <img src="{{ page.root }}/fig/droste-effect-small.jpg" alt="Droste Effect" />
</a>

To illustrate recursive functions, we'll use factorials as an example.
The factorial of a positive integer `N` (written `N!`) is the product of all of the positive integers equal to or less than `N`.
For example, `5! = 5 x 4 x 3 x 2 x 1 = 120`.

We can calculate a factorial iteratively:

~~~
def factorial(n):
    product = 1
    for i in range(1, n + 1):
        product = product * i

    return product

for i in range(5):
    print(factorial(i))
~~~
{: .language-python}

~~~
1
1
2
6
24
~~~
{: .output}

But the factorial function has a property which makes it particularly suitable to be defined recursively.

To define a recursive function we need two things: a **recurrence relation** and a **base case**.
A recurrence relation is a process which can be used to derive the value of a sequence, given the previous value in the sequence.
With just a recurrence relation, the function would run forever, continually trying to get the previous value, so we also need a base case.
The base case is a value in the sequence which is known without having to derive it from previous values.

In the case of the factorial function the recurrence relation is: `N! = N * (N-1)!` or equivalently `f(N) = N * f(N - 1)` - the Nth value in the sequence is N times the previous value.
The base case is `0! = 1` - the factorial of zero is one.

So, if we express the factorial function recursively, we get:

~~~
def factorial(n):
    if n == 0:
        return 1

    return n * factorial(n - 1)
~~~
{: .language-python}

There's something a bit dangerous about this implementation though: if we attempt to get the factorial of a negative number, the code will get stuck in an infinite loop.
In practice, Python has a limit to the number of times a function is allowed to recurse, so we'll actually get an error.

To make this safer, we should handle the case where `n < 0` and raise an error.

~~~
def factorial(n):
    if n < 0:
        raise ValueError('Factorial is not defined for values less than 0')
    if n == 0:
        return 1

    return n * factorial(n - 1)
~~~
{: .language-python}

> ## Recursive Fibonacci
>
> Another well known sequence is the Fibonacci sequence: `0, 1, 1, 2, 3, 5, 8, 13, ...` where each value is the sum of the previous two values.
>
> One possible iterative implementation of a function to calculate the Nth Fibonacci number is shown below.
> Also note how tuple packing and unpacking are used to effectively swap two values without using a temporary variable.
>
> ~~~
> def fibonacci(n):
>     # Iterative fibonacci
>     a, b = 0, 1
>
>     for _ in range(n):
>         a, b = b, a + b
>
>     return a
>
> for i in range(8):
>     print(fibonacci(i))
> ~~~
> {: .language-python}
>
> ~~~
> 0
> 1
> 1
> 2
> 3
> 5
> 8
> 13
> ~~~
> {: .output}
>
> Write an equivalent function which uses recursion to calculate the Nth Fibonacci number.
>
> Hint: first think about what the recurrence relation and base case are.
>
> > ## Solution
> >
> > First, we need to decide what the recurrence relation is - in this case it's `f(N) = f(N - 1) + f(N - 2)`.
> > And the base cases `f(0) = 0` and `f(1) = 1`.
> >
> > For the function itself, we can use the same approach as for the factorial function: first handle the base cases, then the recurrence relation:
> >
> > ~~~
> > def fibonacci(n):
> >     if n < 0:
> >         raise ValueError('Fibonacci is not defined for N < 0')
> >     if n == 0:
> >         return 0
> >     if n == 1:
> >         return 1
> >
> >     return fibonacci(n - 1) + fibonacci(n - 2)
> >
> > for i in range(8):
> >     print(fibonacci(i))
> > ~~~
> > {: .language-python}
> >
> > ~~~
> > 0
> > 1
> > 1
> > 2
> > 3
> > 5
> > 8
> > 13
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Lambda Functions

If we build our programs in a functional way, we tend to end up with a lot of small, one line functions which perform very simple operations.
For example, we might have a function which adds one to a number:

~~~
def add_one(x):
    return x + 1

print(add_one(1))
~~~
{: .language-python}

~~~
2
~~~
{: .output}

If we have a lot of these smaller functions which only get used once, it makes more sense to define them where they're used.

**Lambda functions** are small, nameless functions which fulfil this need.
The structure of these functions is not dissimilar to a normal python function definition - we have a keyword `lambda`, a list of parameters, a colon, then the function body.
In Python, the function body is limited to a single expression, which becomes the return value.

~~~
add_one = lambda x: x + 1

print(add_one(1))
~~~
{: .language-python}

~~~
2
~~~
{: .output}

We have assigned the lambda function to a variable, so we can see it more clearly, but we'd normally use it immediately.
Some style guides (we'll come back to these later in the course) consider it bad style to assign a lambda to a variable.
This is because the main point of lambdas in Python is to avoid having to name the function - it's a bit strange to do that and then give it a name.
Because of this, if you find yourself wanting to name a lambda, just use a normal function instead.

Lambda functions also make it slightly harder to do debugging (coming in a few days time) as the debugger and error messages doesn't have a name to show for them.

Anonymous functions, exist in many modern languages, though they may not be called 'lambda functions' and may be more or less complex to use.
For example, see [Lambda Expressions](https://en.cppreference.com/w/cpp/language/lambda) in C++ with precise control over the visibility of variables inside the function scope, and [Function Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions) in JavaScript, which use syntax much more similar to named function definition.

Finally, there's another common use case of lambda functions that we'll come back to later when we see **closures**.
Due to their simplicity, it can be useful to have a lamdba function as the inner function in a closure.

## Map, Filter, Reduce

One of the most important applications of functional programming in recent years is the Map, Filter, Reduce model of data processing, usually refered to as **MapReduce**.
This model is particularly useful for the processing and analysis of **Big Data** using tools such as Spark or Hadoop.

Note that the `map` and `filter` functions in Python use **lazy evaluation**.
This means that values in an iterable collection are not actually calculated until you need them.
We'll explain some of the implications of this a little later, but for now, we'll just use `list()` to convert the results to a normal list.
In these examples we also see the more typical usage of lambda functions.

The `map` function, takes a function and applies it to each value in an **iterable**.
Here, 'iterable' means any object that can be iterated over - for more details see the [Iterable Abstract Base Class documentation](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterable).
The results of each of those applications become the values in the **iterable** that is returned.

~~~
l = [1, 2, 3]

def add_one(x):
    return x + 1

# Returns a <map object> so need to cast to list
print(list(map(add_one, l)))
print(list(map(lambda x: x + 1, l)))
~~~
{: .language-python}

~~~
[2, 3, 4]
[2, 3, 4]
~~~
{: .output}

Like `map`, `filter` takes a function and applies it to each value in an iterable, keeping the value if the result of the function application is `True`.

~~~
l = [1, 2, 3]

def is_gt_one(x):
    return x > 1

# Returns a <filter object> so need to cast to list
print(list(filter(is_gt_one, l)))
print(list(filter(lambda x: x > 1, l)))
~~~
{: .language-python}

~~~
[2, 3]
[2, 3]
~~~
{: .output}

The `reduce` function is different.
This function uses a function which accepts two values to accumulate the values in the iterable.
The simplest uses here are to calculate the sum or product of a sequence.

~~~
from functools import reduce

l = [1, 2, 3]

def add(a, b):
    return a + b

print(reduce(add, l))
print(reduce((lambda a, b: a + b), l))
~~~
{: .language-python}

~~~
6
6
~~~
{: .output}

These are the fundamental components of the MapReduce style, and can be combined to perform much more complex data processing operations.

> ## Sum of Squares
>
> Using the MapReduce model, write a function that calculates the sum of the squares of the values in a list.
> Your function should behave as below:
>
> ~~~
> def sum_of_squares(l):
>     # Your code here
>
> print(sum_of_squares([0]))
> print(sum_of_squares([1]))
> print(sum_of_squares([1, 2, 3]))
> print(sum_of_squares([-1]))
> print(sum_of_squares([-1, -2, -3]))
> ~~~
> {: .language-python}
>
> ~~~
> 0
> 1
> 14
> 1
> 14
> ~~~
> {: .output}
>
> > ## Solution
> >
> > ~~~
> > from functools import reduce
> >
> > def sum_of_squares(l):
> >     squares = map(lambda x: x * x, l)
> >     return reduce(lambda a, b: a + b, squares)
> > ~~~
> > {: .language-python}
> >
> {: .solution}
>
> Now let's assume we're reading in these numbers from an input file, so they arrive as a list of strings.
> Modify your function so that it passes the following tests:
>
> ~~~
> print(sum_of_squares(['1', '2', '3']))
> print(sum_of_squares(['-1', '-2', '-3']))
> ~~~
> {: .language-python}
>
> ~~~
> 14
> 14
> ~~~
> {: .output}
>
> > ## Solution
> >
> > ~~~
> > from functools import reduce
> >
> > def sum_of_squares(l):
> >     integers = map(int, l)
> >     squares = map(lambda x: x * x, integers)
> >     return reduce(lambda a, b: a + b, squares)
> > ~~~
> > {: .language-python}
> >
> {: .solution}
>
> Finally, like comments in Python, we'd like it to be possible for users to comment out numbers in the input file they give to our program.
> Extend your function so that the following tests pass (don't worry about passing the first set of tests with lists of integers):
>
> ~~~
> print(sum_of_squares(['1', '2', '3']))
> print(sum_of_squares(['-1', '-2', '-3']))
> print(sum_of_squares(['1', '2', '#100', '3']))
> ~~~
> {: .language-python}
>
> ~~~
> 14
> 14
> 14
> ~~~
> {: .output}
>
> > ## Solution
> >
> > ~~~
> > from functools import reduce
> >
> > def sum_of_squares(l):
> >     not_comments = filter(lambda x: x[0] != '#', l)
> >     integers = map(int, not_comments)
> >     squares = map(lambda x: x * x, integers)
> >     return reduce(lambda a, b: a + b, squares)
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

> ## Multiprocessing (Optional Advanced Challenge)
>
> **Advanced challenge for if you're finished early.**
>
> One of the benefits of functional programming is that, if we have pure functions, when applying / mapping a function to many values in a collection, each application is completely independent of the others.
> This means that we can take advantage of multiprocessing, without many of the normal problems in synchronisation that this brings.
>
> Read through the Python documentation for the [multiprocessing module](https://docs.python.org/3/library/multiprocessing.html), particularly the `Pool.map` method.
>
> Update one of our examples to make use of multiprocessing.
> How much of a performance improvement do you get?
> Is this as much as you would expect for the number of cores your CPU has?
>
> **Hint:** To time the execution of a Python script we can use the Linux program `time`:
>
> ~~~
> time python3 my_script.py
> ~~~
> {: .language-bash}
>
> Would we get the same benefits from parallel equivalents of the `filter` and `reduce` functions?
> Why, or why not?
>
> {: .language-bash}
{: .challenge}

{% include links.md %}
