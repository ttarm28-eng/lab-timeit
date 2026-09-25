# Lab: timeit

In this lab you will learn how to measure the runtime of code empirically.
You will then observe that the observed runtimes match the predictions from big-O notation.

<img src=img/meme.jpg />

The joke above is that the choice of *algorithm* and *data structure* combination matters much more than the choice of programming language.
Good code in a "bad" language will be much faster than bad code in a "good" language.

## Background

In this background section we will see a real world difference in runtimes between good and bad code for working with AI.

In a previous lab you all installed the `llm` package.
I have stopped personally using `llm` this semester, however, because recent updates have made it very slow.
In the shell, you can use the `time` command to time something.
For example:
```
$ time llm -m groq-qwen test
<...>

real    0m3.348s
user    0m2.124s
sys     0m0.108s
```
Your `llm` command will give you some *non-deterministic* output which is represented by `<...>` above.
Then the `time` command will show you how much time the program took.
The most important of those numbers is the `real` time, which is the actual wall clock time elapsed.
(`user` is how much time the CPU spent doing work, and `sys` is how much time the operating system spent doing work.)

This is really slow, because the whole point of the groq company was fast response times to queries.
The fundamental cause of this slowness is that `llm` is a fully vibe-coded app and the author has stopped paying attention to speed.
So I forked `llm` and created my own program that I call `dic`
("dic" is latin for speak and is a traditional medieval command for invoking demons;
it is pronounced like English "deek".)

`dic` is backwards compatible with `llm` for all of our use cases, but much faster.
It can be installed with the command
```
$ pip3 install git+https://github.com/mikeizbicki/dic
```
> **NOTE:**
> A more detailed summary of why the `llm` system has become slow and how `dic` avoids this slowdown is available at <https://github.com/mikeizbicki/dic>.

The `groq` API key gets registered with
```
$ export GROQ_API_KEY=<...>
```
(Depending on exactly how you installed `llm`, you may have this API registered globally like this, or you may have used an `llm`-specific install method.  If the former, you don't need to do anything; if the latter, you need to run the export command above and put it in your bashrc file.)

Now timing dic is much faster.
```
$ time dic -m groq+qwen test
<...>

real    0m0.453s
user    0m0.172s
sys     0m0.027s
```
You can use either `llm` or `dic` in this class for accessing AI.
I recommend modifying your bash alias for `qwen` to
```
alias qwen='dic -m groq+qwen -s "answer in 1-20 sentences with high signal to noise"'
```
so that you get the more efficient version.

## Main Lab

### Part A: Setup

Fork this repo and clone your fork on the lambda server.
The instructions below will ask you to directly edit the README file of your fork.

### Part B: Asymptotic Runtimes

The file `palindrome.py` contains several functions that check whether the input container is a palindrome.

Run the doctests to verify that all functions are correct:
```
$ python3 -m doctest palindrome.py
```

Read through the code and complete the table below.
Write the runtimes in terms of `n=len(container)` using big-O notation.

|                        | `str`  | `list` | `deque` |
| ---------------------- | ------ | ------ | ------- |
| `check_palindrome_1`   | $O(n)$ | $O(n))$|$O(n^2)$ |
| `check_palindrome_2`   | $O(n)$ | $O(n)$ | $O(n)$  |
| `check_palindrome_3`   |   --   |$O(n^2)$| $O(n)$  |

> **NOTE**:
> The `str` type is *immutable* and so does not support being modified.
> The `_3` function above requires modifying the input container and so cannot work with `str`.
> Runtimes of `str` are the same as for `[]` on all supported operations.

### Part C: Empirical Runtimes

Now you will use the [timeit module](https://docs.python.org/3/library/timeit.html) in python to measure the runtimes of the palindrome functions.
This module is used in the terminal in the following way:
```
$ python3 -m timeit -s "$SETUP_CODE" "$CODE_TO_TIME"
```
where `$SETUP_CODE` is python code whose runtime we don't want to measure (but need to run to setup the problem),
and `$CODE_TO_TIME` is python code whose runtime we will measure.
The `$CODE_TO_TIME` will get run many times,
and `timeit` will report the average runtime.

For example, in order to measure the runtime of the `check_palindrome_1` function on a list and deque of length 5, we could run the commands:
```
$ CODE_TO_TIME='palindrome.check_palindrome_1(xs)'
$ python3 -m timeit -s 'import palindrome; xs=[1,2,3,2,1]' "$CODE_TO_TIME"
$ python3 -m timeit -s 'import palindrome; from collections import deque; xs=deque([1,2,3,2,1])' "$CODE_TO_TIME"
```
Because these containers are so small,
the runtimes are insignificant.
(I get about 0.7 microseconds for both examples).
It is common to use the letter $n$ to denote the length of a container.
With this notation, we can also say that because $n$ is small, the runtimes are insignificant.

What we're really interested in, however, is when $n$ is large.
We can easily generate large containers using python's container multiplication operator.
For example `[1]*65536` will give us a container of length one hundred thousand with all ones.
(65536 is $2^{16}$.  Since it is the largest number that can be stored in two bytes, it appears in many places.)
If you've never used python's container multiplication feature before,
open up an interactive python session and try it:
```
$ python3
>>> [1]*16
>>> [1]*65536
```

Now time the `check_palindrome_1` function on a deque of that size:
```
$ python3 -m timeit -s 'import palindrome; from collections import deque; xs=deque([1]*65536)' 'palindrome.check_palindrome_1(xs)'
```

Complete the following table with actual measured runtimes by substituting the values for `xs` and the function in the command above.

|                        | `xs=("1"*65536)` | `xs=([1]*65536)` | `xs=deque([1]*65536)` |
| ---------------------- | ---------------- | ---------------- | --------------------- |
| `check_palindrome_1`   |    3.24 msec     |    3.24 msec     |        55.3 msec      |
| `check_palindrome_2`   |    1.81 msec     |    1.81 msec     |        1.85 msec      |
| `check_palindrome_3`   |       --         |    128 msec      |        2.67 msec      |

You should observe that the slow runtimes here correspond with the $O(n^2)$ asymptotic runtimes,
and the fast runtimes correspond with the $O(n)$ runtimes.

This tells us that the runtime of a function depends on:
(1) the algorithm that it is implemented with, and
(2) the data types it is run on.

### Part C: Empirical Runtimes (II)

Muggles think that $2^{16} = 65536$ is a large number.
But in the computer science world, this is considered small.

This part of the lab will help give you a sense of just how bad an $O(n^2)$ algorithm can be as $n$ gets large.

Consider the shell code below.
It runs `timeit` on `check_palindrome_3` using a `deque` of size `2**16`.

```
$ N=16
$ CONTAINER=deque
$ python3 -m timeit -s "import palindrome; from collections import deque; xs=$CONTAINER([1]*2**$N)" "palindrome.check_palindrome_3(xs)"
```

The output runtime of the command above should be placed in the top right corner of the table below.
Complete the table by modifying the `N` and `CONTAINER` variables in the shell code above for each cell.

|                        | `CONTAINER=list` | `CONTAINER=deque`     |
| ---------------------- | ---------------- | --------------------- |
| `N=16`                 |   129 msec       |     2.46 msec         |
| `N=17`                 |   554 msec       |     4.97 msec         |
| `N=18`                 |   3.95 sec       |     9.95 msec         |
| `N=19`                 |   19.3 sec       |     19.97 msec        |

You should observe that the quadratic algorithm/container combination gets *really* slow *really* fast.
The takeaway: **$O(n^2)$ is bad**.

## Submission

Upload your changes to github and submit the url to canvas.
