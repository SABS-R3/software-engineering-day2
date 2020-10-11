---
title: "Functional Programming in Python"
teaching: 0
exercises: 70
questions:
- "How do we express the MapReduce model in a more Pythonic way?"
- "How can we make our data processing more memory efficient?"
objectives:
- "Use comprehensions as an alternative representation of the MapReduce model"
- "Use lazy evaluation to be more memory efficient when processing large amounts of data"
keypoints:
- "Lazy evaluation does not calculate values until they are needed"
- "Decorators are a way to modify the behaviour of existing functions"
---

## Functional Programming in Python

The Python language has been designed with a set of principles in mind, one of which is: "there's only one way to do it".
Though this is not quite true, there *are* often a few valid ways to do something, there does tend to be one solution which is more **Pythonic** than the rest.

It's not really possible to teach what is Pythonic and what is not.
This tends to come from experience of using the language for long enough to get an intuition about which idioms feel more natural.

> ## The Zen of Python
>
> Some of the principles that guided the design of Python are shared in 'The Zen of Python':
>
> ~~~
> import this
> ~~~
> {: .language-python}
>
> ~~~
> The Zen of Python, by Tim Peters
>
> Beautiful is better than ugly.
> Explicit is better than implicit.
> Simple is better than complex.
> Complex is better than complicated.
> Flat is better than nested.
> Sparse is better than dense.
> Readability counts.
> Special cases aren't special enough to break the rules.
> Although practicality beats purity.
> Errors should never pass silently.
> Unless explicitly silenced.
> In the face of ambiguity, refuse the temptation to guess.
> There should be one-- and preferably only one --obvious way to do it.
> Although that way may not be obvious at first unless you're Dutch.
> Now is better than never.
> Although never is often better than *right* now.
> If the implementation is hard to explain, it's a bad idea.
> If the implementation is easy to explain, it may be a good idea.
> Namespaces are one honking great idea -- let's do more of those!
> ~~~
> {: .output}
>
> These principles are a reasonable set of guidelines when working in any language, not just Python, so it's worth reading and keeping in mind.
> Many of the common problems we see with research software can be avoided by following these guidelines.
>
{: .callout}

## Comprehensions

Comprehensions are a more Pythonic way to structure map and filter operations.
They serve exactly the same purpose, but are more concise and can be easier to structure in more complex cases, such as mapping over a 2d data structure.
Using comprehensions also gives us control over which data structures we end up with, rather than always getting back a `map` or `filter` iterable.

### List Comprehensions

The **list comprehension** is probably the most commonly used comprehension type.
As you might expect from the name, list comprehensions produce a list from some other iterable type.
In effect they are the same as using `map` and/or `filter` and using `list()` to cast the result to a list, as we did previously.

All comprehension types are structured in a similar way, using the syntax for a literal of that type (in this case a list literal) containing what looks like the top of a for loop.
To the left of the `for` we put the equivalent of the map operation we want to use:

~~~
print([i for i in range(5)])
print([2 * i for i in range(5)])
~~~
{: .language-python}

~~~
[0, 1, 2, 3, 4]
[0, 2, 4, 6, 8]
~~~
{: .output}

We can also use list comprehensions to perform the equivalent of a filter operation, by putting the filter condition at the end:

~~~
print([2 * i for i in range(5) if i % 2 == 0])
~~~
{: .language-python}

~~~
[0, 4, 8]
~~~
{: .output}

### Dictionary and Set Comprehensions

Dictionary and set comprehensions are fundamentally the same as list comprehensions but use the dictionary or set literal syntax.

So set comprehensions are:

~~~
print({2 * i for i in range(5)})
~~~
{: .language-python}

~~~
{0, 2, 4, 6, 8}
~~~
{: .output}

While dictionary comprehensions are:

~~~
print({i: 2 * i for i in range(5)})
~~~
{: .language-python}

~~~
{0: 0, 1: 2, 2: 4, 3: 6, 4: 8}
~~~
{: .output}

> ## Why No Tuple Comprehension
>
> Raymond Hettinger, one of the Python core developers, said in 2013:
>
> ~~~
> Generally, lists are for looping; tuples for structs. Lists are homogeneous; tuples heterogeneous. Lists for variable length.
> ~~~
>
{: .callout}

## Generators

### Generator Comprehensions

Generator comprehensions look very similar to list comprehensions, but behave slightly differently.
What happens if we try to use them in the same was as we did list comprehensions?

~~~
print((2 * i for i in range(5)))
~~~
{: .language-python}

~~~
<generator object <genexpr> at 0x7efc21efcdd0>
~~~
{: .output}

Like the `map` and `filter` functions, generator expressions are not evaluated until you iterate over them.

~~~
for i in (2 * i for i in range(5)):
    print(i)
~~~
{: .language-python}

~~~
0
2
4
6
8
~~~
{: .output}

To look at the reasons why you might choose to use a generator comprehension, we'll use some more IPython magics to investigate the performance of the different comprehensions.

If we initialise and fully iterate over each comprehension, the time taken is very similar.

~~~
%%timeit
l = [2*x for x in range(1000000)]
for x in l:
    pass
~~~
{: .language-python}

~~~
66 ms ± 492 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
~~~
{: .output}

~~~
%%timeit
l = (2*x for x in range(1000000))
for x in l:
    pass
~~~
{: .language-python}

~~~
65.4 ms ± 2.3 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
~~~
{: .output}

If we initialise both a list and a generator comprehension and store them, but do not iterate over them, the generator is around 100,000x faster.
This is the result of lazy evaluation - if no values are calculated until we need them, we expect the initialisation to be very fast.

~~~
%%timeit
l = [2*x for x in range(1000000)]
~~~
{: .language-python}

~~~
59.7 ms ± 372 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
~~~
{: .output}

~~~
%%timeit
l = (2*x for x in range(1000000))
~~~
{: .language-python}

~~~
494 ns ± 11.5 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
~~~
{: .output}

If we initialise both versions, but only partially iterate over each, then we can see the benefit of using a generator comprehension over a list comprehension.

~~~
%%timeit
l = [2*x for x in range(1000000)]
for x in l:
    if x > 10:
        break
~~~
{: .language-python}

~~~
63.5 ms ± 1.77 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
~~~
{: .output}

~~~
%%timeit
l = (2*x for x in range(1000000))
for x in l:
    if x > 10:
        break
~~~
{: .language-python}

~~~
988 ns ± 5.76 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
~~~
{: .output}

Since the list comprehension produces an actual list, whereas the generator comprehension doesn't produce any values until they are required, we also see a large difference in memory use between the two:

~~~
%load_ext memory_profiler
%%memit
# Warning - uses about 400MB RAM
for x in [2*x for x in range(int(1e7))]:
    pass
~~~
{: .language-python}

~~~
peak memory: 438.18 MiB, increment: 386.56 MiB
~~~
{: .output}

~~~
%%memit
for x in (2*x for x in range(int(1e7))):
    pass
~~~
{: .language-python}

~~~
peak memory: 51.78 MiB, increment: 0.00 MiB
~~~
{: .output}

In these outputs, 'peak memory' is the maximum amount of memory used by the Jupyter kernel during the task, while 'increment' is the amount of additional memory that running the cell required - this is the number we're interested in.

## Generator Functions

There's one final common place where we see lazy evaluation - a **generator function**.
Generator functions are similar to generator comprehensions, but in using a function.

Instead of using `return` to return a single value from a function, a generator function **yields** multiple values:

~~~
def count_to_n(n):
    for i in range(n):
        yield i + 1

for i in count_to_n(5):
    print(i)
~~~
{: .language-python}

~~~
1
2
3
4
5
~~~
{: .output}

> ## Extending Academic Model
>
> Optional example - doesn't need to be included.
> Counting papers from academics.
>
> ~~~
> class Paper:
>     def __init__(self, title):
>         self.title = title
>
>     def __str__(self):
>         return self.title
>
>     def __repr__(self):
>         return self.title
>
> class Person:
>     def __init__(self, name):
>         self.name = name
>
>     def __str__(self):
>         return self.name
>
>     def __repr__(self):
>         return self.name
>
> class Academic(Person):
>     def __init__(self, name):
>         super().__init__(name)
>         self.papers = []
>         self.staff = []
>
>     def write_paper(self, title):
>         new_paper = Paper(title)
>
>         self.papers.append(new_paper)
>         return new_paper
>
>     def add_staff(self, academic):
>         if academic not in self.staff:
>             self.staff.append(academic)
>
>     @property
>     def all_papers(self):
>         papers = list(self.papers)
>         for staff in self.staff:
>             papers = papers + staff.all_papers
>
>         return papers
>
>
> academics = [Academic(name) for name in ['Alice', 'Bob', 'Carol', 'David']]
> alice = academics[0]
> bob = academics[1]
> alice.add_staff(bob)
>
> alice.write_paper('A science paper')
> bob.write_paper('Another science paper')
>
> print(alice.all_papers)
>
> ~~~
> {: .language-python}
{: .callout}

## Decorators
Function that accepts a function and returns a (different) function

~~~
def decorate_hello(fun):
    def inner(*args, **kwargs):
        print('Hello!')
        return fun(*args, **kwargs)

    return inner

@decorate_hello
def say_world():
    print('World!')

say_world()
~~~
{: .language-python}

~~~
Hello!
World!
~~~
{: .output}

> ## Sum of Squares (Again)
>
> Instead of using the `map` and `filter` functions, write a sum of squares function using comprehensions.
>
> You may find the `sum` function useful - it sums all items in an iterable collection.
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
> > def sum_of_squares(l):
> >     squares = [x * x for x in l]
> >     return sum(squares)
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
> > def sum_of_squares(l):
> >     integers = [int(x) for x in l]
> >     squares = [x * x for x in integers]
> >     return sum(squares)
> > ~~~
> > {: .language-python}
> >
> > Or, using only one comprehension:
> >
> > ~~~
> > def sum_of_squares(l):
> >     return sum([int(x) * int(x) for x in l])
> > ~~~
> > {: .language-python}
> >
> {: .solution}
>
> Finally, like comments in Python, we'd like it to be possible for users to comment out numbers in the input file the give to our program.
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
> > def sum_of_squares(l):
> >     not_comments = [x for x in l if x[0] != '#']
> >     integers = [int(x) for x in not_comments]
> >     squares = [x * x for x in integers]
> >     return sum(squares)
> > ~~~
> > {: .language-python}
> >
> > Or, in one comprehension:
> >
> > ~~~
> > def sum_of_squares(l):
> >     return sum([int(x) * int(x) for x in l if x[0] != '#'])
> > ~~~
> > {: .language-python}
> {: .solution}
>
{: .challenge}

> ## Dictionary of Squares
>
> Define a function `dict_of_squares` which takes an integer and returns a dictionary whose keys `i` are the integers from 1 to N (inclusive) and whose values are lists of square numbers up to `i` squared.
>
> For example:
>
> ~~~
> def dict_of_squares(n):
>     # your code here
>
> print(dict_of_squares(3))
> ~~~
> {: .language-python}
>
> ~~~
> {1: [1], 2: [1, 4], 3: [1, 4, 9]}
> ~~~
> {: .output}
>
> > ## Solution
> > ~~~
> > def dict_of_squares(n):
> >     return {i: [j * j for j in range(1, i + 1)] for i in range(1, n + 1)}
> > ~~~
> > {: .language-python}
> >
> > To improve readability, you may wish to format it differently - such as:
> >
> > ~~~
> > def dict_of_squares(n):
> >     return {
> >         i: [
> >             j * j for j in range(1, i + 1)
> >         ] for i in range(1, n + 1)
> >     }
> > ~~~
> > {: .language-python}
> >
> {: .solution}
>
{: .challenge}

{% include links.md %}
