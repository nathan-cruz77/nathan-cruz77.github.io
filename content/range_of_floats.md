Slug: range-of-floats
Title: Building a range of floats
Date: 17-07-2016 00:28
Modified: 19-07-2016 22:37
Category: Python
Lang: en

A few weeks ago I had to implement a few AI algorithms for a college project. I
ended up making some code that might be useful to other people, so I decided to
share it.

## A range of floats
One of the subtasks was to generate all of the numbers in a given interval:
 * [0, 5) with a step of 1 should result in: [0, 1, 2, 3, 4]
 * [0, 0.5) with a step of 0.1 should result in: [0, 0.1, 0.3, 0.4]

Most people familiar with python would say we could simply use
[`range`](https://docs.python.org/3/library/functions.html?highlight=range#func-range).
This very useful function is usually what I'd go with but when I tried it I got this:
```python
>>> list(range(0, 5, 1))
[0, 1, 2, 3, 4]
>>>list(range(0, 0.5, 0.1))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'float' object cannot be interpreted as an integer
```

`range` clearly solves the first case but, why not the second? I thought I was
doing something wrong like changing the argument order or something. So I went to
look for help in the [docs](https://docs.python.org/3/library/stdtypes.html?highlight=range#range).
I got a little disapointed when I found out it doesn't support floats. As I really
needed it I decided to make my own `float_range`.

My first attempt was:
```python
def float_range(start, stop=None, step=1):
    if stop is None:
        stop, start = start, 0

    while(start < stop):
        yield start
        start += step
```
I felt quite smart with this solution unil I found an issue which I believe to be
the reason for floats to not being supported by range.
```python
>>> list(float_range(0, 5, 1))
[0, 1, 2, 3, 4]
>>> list(float_range(5))
[0, 1, 2, 3, 4]
>>> list(float_range(0, 0.5, 0.1))
[0, 0.1, 0.2, 0.30000000000000004, 0.4]
```
Things were good to be true. This problem of floating point arithmetics is well
known and it's related to the difficulty of representing "broken" numbers in
binary. For more information take a look [here](http://stackoverflow.com/a/588014).

There's a builtin function in python that helped me solving this problem.
[`round`](https://docs.python.org/3/library/functions.html?highlight=round#round),
as the name suggests, rounds the number to the closest integer. Despite this
feature being quite amazing it's not the best part of `round`. There's a second
optional parameter to specify how many digits to be taken into consideration when
rounding.

Having found this out I tried it a second time:
```python
def float_range(start, stop=None, step=1):
    if stop is None:
        stop, start = start, 0

    while(start < stop):
        yield round(start, 1)
        start += step
```
This version is much better but still fails when we need dynamic precision.
```python
>>> list(float_range(0, 0.5, 0.1))
[0, 0.1, 0.2, 0.3, 0.4]
>>> list(float_range(0, 0.05, 0.01))
[0, 0.0, 0.0, 0.0, 0.0]
```
This problem would repeat itself as long as I kept the precision constant. I then
decided the best solution would be to determine the step precision at runtime. I
went ahead and wrote a helper function to do so.
```python
def precision(number):
    try:
        res = len(str(number).split('.')[1])
    except IndexError:
        res = 1

    return res
```
We transform the number into a string, split it in `.` and the precision will be
the length of the second string. If splitting the string doesn't result in two
substrings, we're dealing with an integer and the precision is 1.

With this helper function we can enhance the earlier solution:
```python
def float_range(start, stop=None, step=1):
    if stop is None:
        stop, start = start, 0

    while(start < stop):
        yield round(start, precision(step))
        start += step
```
This last version finally works as expected.
```python
>>> list(float_range(0, 5, 1))
[0, 1, 2, 3, 4]
>>> list(float_range(0, 0.5, 0.1))
[0, 0.1, 0.2, 0.3, 0.4]
>>> list(float_range(0, 0.5, 0.01))
[0, 0.01, 0.02, 0.03, 0.04, 0.05, 0.06, 0.07, 0.08, 0.09, 0.1, 0.11, 0.12, 0.13, 0.14, 0.15, 0.16, 0.17, 0.18, 0.19, 0.2, 0.21, 0.22, 0.23, 0.24, 0.25, 0.26, 0.27, 0.28, 0.29, 0.3, 0.31, 0.32, 0.33, 0.34, 0.35, 0.36, 0.37, 0.38, 0.39, 0.4, 0.41, 0.42, 0.43, 0.44, 0.45, 0.46, 0.47, 0.48, 0.49]
```
