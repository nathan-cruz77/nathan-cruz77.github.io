Slug: measuring-execution-time
Title: Measuring execution time
Date: 21-07-2016 01:39
Modified: 25-12-2016 14:01
Category: Programming
Tags: Python, Optimization, Shell, Profiling, Bash, Linux
Lang: en

On my last [post](/fibonacci-with-cache) I talked about how to use
cache to improve the performance of a recursive implementation of fibonacci. All
charts were built with [Gnuplot](http://www.gnuplot.info/).

Since I wanted a significant amount of data for the charts, I could not generate
them manually. I had to automate.

I had already used the `time` tool, a utility offered by
[`bash`](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29), but had a hard
time formatting it's output, which has the following form:
```shell
bla@laptop:~$ time python3 fibonacci.py 25
real	0m0.064s
user	0m0.056s
sys 	0m0.008s
```
After countless internet searches I found out that there's a program unrelated
to `bash` also called `time` that measures execution and accepts format
operators. It is installed on `/usr/bin` (at least on my
[Debian](https://www.debian.org/) system) and, despite this directory being on my
`PATH`, it is obfuscated by bash's `time`.
In order to call it I had to use the file path. By my own preference I used it's
absolute path (`/usr/bin/time`).

This is how `/usr/bin/time` output looks like:
```shell
bla@laptop:~$ /usr/bin/time python3 fibonacci.py 25
0.06user 0.00system 0:00.06elapsed 95%CPU (0avgtext+0avgdata 7724maxresident)k
0inputs+0outputs (0major+865minor)pagefaults 0swaps
```
As I was only interested in the total time (_elapsed_ in the above snippet),
I was able to use a quite simple format string to retrieve only this field.
```shell
bla@laptop:~$ /usr/bin/time -f "%E" python3 fibonacci.py 25
0:00.06
```
To remove the minutes portion (what precedes `:`) and have an easily parseable
output I used [`cut`](https://pt.wikipedia.org/wiki/Cut_%28Unix%29).
In this specific case I simply had to specify the field delimiter (`:`) and
the field I wanted (`2`).
```shell
bla@laptop:~$ /usr/bin/time -f "%E" python3 fibonacci.py 25 | cut -d: -f2
0:00.07
```
That was not as expected. What is happening here is that the pipe (`|`) is
redirecting the [`stdout`](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout))
of the command on it's left to the
[`stdin`](https://pt.wikipedia.org/wiki/Fluxos_padr%C3%A3o#Entrada_padr.C3.A3o_.28stdin.29)
of the command on it's right, in this case `cut`. but `/usr/bin/time` doesn't print to `stdout` but to
[`stderr`](https://pt.wikipedia.org/wiki/Fluxos_padr%C3%A3o#Erro_padr.C3.A3o_.28stderr.29)!
A way around this issue is to redirect `stderr` to `stdout`. Thankfully `bash` lets
us acomplish it easily. The standard output (`stdout`) is represented by `1` and
standard error (`stderr`) by `2`. To redirect `stderr` to `stdout` we can use `2>&1`.
The other way around is also possible (send `stdout` to `stderr`) using `1>&2`.
Using the first form we get the expected result.
```shell
bla@laptop:~$ /usr/bin/time -f "%E" python3 fibonacci.py 25 2>&1 | cut -d: -f2
00.05
```

To test `fibonacci.py` with multiple numbers we can simply run it inside a loop and
send the output to a file.
```shell
bla@laptop:~$ for n in {1..5}
> do
> /usr/bin/time -f "%E" python3 fibonacci.py $n 2>&1 | cut -d: -f2 > tempos.txt
> done
00.02
00.02
00.02
00.02
00.02
```

Thi is how I generated last post's graphs. Since then I've been asking myself if
this is the most pythonic solution. I believe it's not. It's excessively complex.
I've been digging into code so that I don't need to deal with all this shell.

## Measuring time with python
The task of measuring execution time of pieces of code is usually part of the
_profiling_ process, which aims to identify performance bottlenecks. As this process
is widely used, so are its subtasks. Measuring execution time is no exception.

### Using `time.time()` directly
As most programming language, python provides a module to deal with time in its
standard library. The function
[`time`](https://docs.python.org/3/library/time.html?highlight=time.time#time.time),
which belongs to the module with the same name, returns the amount of seconds elapsed
since [_epoch_](https://en.wikipedia.org/wiki/Unix_time) (January 1, 1970). This
popular time representation is also known as _Unix time_.

With this function it's possible to easily measure the execution time of any code
block. For example:
```python
import time

def slow_func():
    time.sleep(10)

start_time = time.time()
slow_func()
end_time = time.time()

total_time = end_time - start_time

print('{:.3f}s'.format(total_time))
```
This code can already be used inside a loop to get all of the measurements in the
desired format in a much cleaner way than we did before but, can we do better?

### Building a _decorator_
Yes! Using a decorator we can make this code much more reusable and pythonic!
```python
import time
import functools


def timer(func):

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()

        print('{} ran in: {:.3f}s'.format(func.__name__, end - start))
        return result

    return wrapper

def _fibonacci(n):
    if n < 2:
        return n
    return _fibonacci(n - 1) + _fibonacci(n - 2)

@timer
def fibonacci(n):
    return _fibonacci(n)
```
Assuming the code above is in the file `fibonacci_with_decorator.py` we can use
in the interactive shell like so:
```python
>>> from fibonacci_with_decorator import fibonacci
>>> fibonacci(3)
fibonacci ran in: 0.000s
2
>>> fibonacci(25)
fibonacci ran in: 0.053s
75025
```
We had to add a _wrapper_ to the fibonacci function due to its recursive nature.
If the decorator were applied to the recursive function directly the execution time
for every call would be shown.

This method allows us to measure execution time of multiple code blocks running it
once. Besides, performance data can be used in logs, allowing the application
to be analysed without having to stop it.

### Using a context manager
I had completely overlooked this approach until I read a friend's article about
[speeding up MongoDB queries with tornado](https://rafaelcapucho.github.io/2016/09/speeding-up-your-mongodb-queries-up-to-30-times-with-tornado/).

You've probably already seen a context manager, they are very common when handling
files. A context manager is any object that implements the context manager's interface
(methods `__enter__` and `__exit__`). The objects returned by `open` are the most
common example.

Using these objects in a _`with`_ block we implicitly make `__enter__` be called
when entering the block and `__exit__` when exiting it. For files we usually have
something like:
```python
with open('some_file_path') as f:
    pass # read from f

# f is no longer open
```
When we exit the block the file is automatically closed (i.e. `f.close()`), so
we don't have to do so.

As any object that implements the context manager interface can be used with
_`with`_, we can make a timer with it. We just have to start the timer on `__enter__`
and stop it on `__exit__`.
```python
import time
import sys


class Timer:

    def __enter__(self):
        self.time = time.time()
        return self

    def __exit__(self, *args):
        self.time = time.time() - self.time
        print('Time elapsed: {}s'.format(self.time))


def slow_func():
    time.sleep(10)

with Timer():
    slow_func()
```
Running the above code we get:
```bash
bla@laptop:~$ python context_timer.py
Time elapsed: 10.008252143859863s
```
