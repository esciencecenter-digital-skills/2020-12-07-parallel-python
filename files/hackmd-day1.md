# 2020-12-07 Parallel Python, day 1
Welcome to The Workshop Collaborative Document


This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

All content is publicly available under the Creative Commons Attribution License

https://creativecommons.org/licenses/by/4.0/

 ----------------------------------------------------------------------------

## 👮Code of Conduct
Participants are expected to follow those guidelines:

- Use welcoming and inclusive language
- Be respectful of different viewpoints and experiences
- Gracefully accept constructive criticism
- Focus on what is best for the community
- Show courtesy and respect towards other community members


## ⚖️ License
All content is publicly available under the Creative Commons Attribution License: https://creativecommons.org/licenses/by/4.0/


## 🙋Getting help

- icons below participants list
    - go slower
    - go faster
    - when finished assigned task: ‘yes’ green tick icon
    - when could not finish the assigned task ‘no’ cross icon
    - to ask a question, type /hand in the chat window
- to get help, type /help in the chat window
- you can ask questions in the document or chat window and helpers will try to help you


## 🖥 Workshop website
[https://escience-academy.github.io/2020-12-07-parallel-python/](https://escience-academy.github.io/2020-12-07-parallel-python/)


## 🛠 Setup



## 🧑‍🏫 Instructors
Johan Hidding, Pablo Rodriguez-Sanchez


## 🧑‍🙋 Helpers
Djura Smits, Hanno Spreeuw

## 🧑‍🔬🧑‍💻🧑‍🚀🧑‍⚕️🧑‍🏭🧑‍🔧 Roll Call

## 🗓️ Agenda
| time | what|
|---|---|
| Before |	Pre-workshop survey|
| 09:00 |	Welcome and icebreaker|
| 09:15 |	Introduction|
| 10:05 |	Break|
| 10:15 |	Measuring performance|
| 11:15 |	Coffee break|
| 11:30 |	Parallelization using Dask Arrays|
| 12:45 |	Wrap-up|
| 13:00 |	END|


# 🧠 Collaborative Notes

## 1. Introduction
In this course we will show several possible ways of speeding up your program and making it ready to function in parallel. We will be introducing the following modules:
1. `Threading`
2. `dask`
3. `numba`
4. `Snakemake`

We will not talk about **distributed programming**.

### Dependency diagrams
Suppose we have a computation where each step **depends** on a previous one. We can represent this situation like in the figure below, known as a dependency diagram:

![Serial computation](https://carpentries-incubator.github.io/lesson-parallel-python/fig/serial.png)

In these diagrams the inputs and outputs of each function are represented as rectangles. The inward and outward arrows indicate their flow. Note that the output of one function can become the input of another one. The diagram above is the typical diagram of a **serial computation**. If you ever used a loop to update a value, you used serial computation.

If our computation involves **independent work** (that is, the results of the application of each function are independent of the results of the application of the rest), we can structure our computation like this:

![Parallel computation](https://carpentries-incubator.github.io/lesson-parallel-python/fig/parallel.png)

This scheme corresponds to a **parallel computation**.

### How can parallel computing improve our code execution speed?
Nowadays, most personal computers have 4 or 8 processors (also known as cores). In the diagram above, we can assign each of the three functions to one core, so they can be performed simultaneously.

However, in reality it is often more complicated.

### Parallelizable and non-paralellizable tasks
#### Example of inherently serial task

Factorial of a number. For instance: `4! = 4 * 3 * 2 *1`
```python=3
n = 4
for i in range(4):
    temp = temp * (i + 1)

```
```python=3
temp

24
```

#### Example of a parallelizable task
```python=3
y = [n**2 for n in [1, 2, 3, 4]]
```
This is a **embarrasingly parallel problem**.

### Embarrassingly parallel problems
Although we talked about embarrassingly parallel problems, it would be more correct to talk about
embarrassingly parallel algorithms.

Often, the parallelizability of a problem depends on its specific implementation. For instance, in our
first example of a non-parallelizable task, we mentioned the calculation of the factorial of 4 using
the algorithm of multiplying, one by one, by all the integers below that number (that is, 4, 3, 2, 1).
If, instead, we use another algorithm, such as the [gamma function](https://en.wikipedia.org/wiki/Gamma_function#Motivation), the same problem accepts parallelization.

Last but not least, don't let the name demotivate you: if your algorithm happens to be embarrassingly parallel, that's good news! The "embarrassingly" refers to the feeling of "this is great!,
how did I not notice before?!"



### Challenge: Parallelised Pea Soup
We have the following recipe:
1.  (1 min) Pour water into a soup pan, add the split peas and bay leaf and bring it to boil.
2. (60 min) Remove any foam using a skimmer and let it simmer under a lid for about 60 minutes.
3. (15 min) Clean and chop the leek, celeriac, onion, carrot and potato.
4. (20 min) Remove the bay leaf, add the vegetables and simmer for 20 more minutes. Stir the soup
occasionally.
5.  (1 day) Leave the soup for one day. Reheat before serving and add a sliced smoked sausage (vegetarian options are also welcome). Season with pepper and salt.

Imagine you're cooking alone.
- Can you identify potential for parallelisation in this recipe?
- And what if you are cooking with the help of a friend help? Is the soup done any faster?
- Draw a dependency diagram.

#### Solution
- You can cut vegetables while simmering the split peas.
- If you have help, you can parallelize cutting vegetables further.
- There are two ‘workers’: the cook and the stove.

![dependency diagram](https://carpentries-incubator.github.io/lesson-parallel-python/fig/soup.png)


## Break
Let's get back at 10:17

## 2. Measuring performance
Without parallelization
```python=3
import numpy as np

# Arange creates an ordered list of numbers
result = np.arange(10**9).sum()
```

With parallelization
```python=3
import dask.array as da

# Dask will not immediately output result
work = da.arange(10**9).sum()
result = work.compute()
```

### Note: Try a heavy enough task
It could be that a task this small does not register on your radar. Depending on your computer you will have to raise the power to 10**8 or 10**9 to make sure that it runs long enough to observe the effect. But be careful and increase slowly. Asking for too much memory can make your computer slow to a crawl.

![system monitor](https://carpentries-incubator.github.io/lesson-parallel-python/fig/system-monitor.jpg)

### More systematic measurements
How can we test this in a more practical way? In Jupyter we can use some line magics, small “magic words” preceded by the symbol %% that modify the behaviour of the cell.

```python=3
%%time
result = np.arange(10**9).sum()
```

The `%%time` line magic checks how long it took for a computation to finish. It does nothing to change the computation itself. In this it is very similar to the time shell command.

If run the chunk several times, we will notice a difference in the times. How can we trust this timer, then? A possible solution will be to time the chunk several times, and take the average time as our valid measure. The %%timeit line magic does exactly this in a concise an comfortable manner! %%timeit first measures how long it takes to run a command one time, then repeats it enough times to get an average run-time. Also, `%%timeit` can measure run times without the time it takes to setup a problem, measuring only the performance of the code in the cell. This way we can trust the outcome better.

Without parallelization
```python=3
%%timeit
result = np.arange(10**9).sum()
```

With parallelization
```python=3
%%timeit
work = da.arange(10**9).sum()
result = work.compute()
```

### Using many cores
Using more cores for a computation can decrease the run time. The first question is of course: how many cores do I have?

On linux:
```bash=
lscpu
```

On mac:
```bash=
sysctl -n hw.physicalcpu
```

On windows:
```shell=
WMIC CPU Get NumberOfCores,NumberOfLogicalProcessors
```

In python:
```python=
import multiprocessing
multiprocessing.cpu.count()
```

However, even with simple examples performance may scale unexpectedly. There are many reasons for this. One of the most relevant is hyper-threading, that is, the fact that the number of visible CPUs is often not equal to the number of physical cores.

### Memory profiling

Install memory profiler:
```shell=
pip install memory_profiler
```

```python=
from memory_profiler import memory_usage
import numpy as np
import dask.array as da

def sum_with_numpy():
    # Serial implementation
    np.arange(10**9).sum()

def sum_with_dask():
    # Parallel implementation
    work = da.arange(10**9).sum()
    work.compute()

memory_numpy = memory_usage(sum_with_numpy, interval=0.01)
memory_dask = memory_usage(sum_with_dask, interval=0.01)
```

```python=
import matplotlib.pyplot as plt

plt.plot(memory_numpy, label='numpy')
plt.plot(memory_dask, label='dask')
plt.xlabel('Time step')
plt.ylabel('Memory / MB')
plt.legend()
plt.show() # Or maybe skip this line if you don't see a plot
```

The figure should be similar to the one below:
![memory comparison](https://carpentries-incubator.github.io/lesson-parallel-python/fig/memory.png)

### Plotting time performance
However, even with simple examples performance may scale unexpectedly. There are many reasons for this. One of the most relevant is hyper-threading, that is, the fact that the number of visible CPUs is often not equal to the number of physical cores.

See for instance the example below:

On a machine with 8 listed cores doing this (admittedly oversimplistic) benchmark:


```python=
import timeit

x = [timeit.timeit(stmt=f'da.arange(10**8).sum().compute(num_workers={n})', setup='import dask.array as da', number=1) for n in range(1, 9)]
```

```python=
import pandas as pd

data = pd.DataFrame({'n': range(1,9), 't': x})
data.set_index('n').plot()
```
![timeit plot](https://carpentries-incubator.github.io/lesson-parallel-python/fig/more-cores.svg)

There is also `%memit` for memory usage profiling, but this does not provide the 'interval' option.

### Key points
- It is often non-trivial to understand performance
- Memory is just as important as speed
- Measuring is knowing

### Hyperthreading
Why is the runtime increasing if we add more than 4 cores? This has to do with hyper-threading. On most architectures it makes not much sense to use more workers than the number of physical cores you have.

### Note
Other processes running on your pc can influence the performance a lot. You can try to run the code again after the lessons when you have closed zoom to see if it is any different.

## Break
We'll be back at 11:33

## Understanding parallelization in Python
![pi approximation](https://carpentries-incubator.github.io/lesson-parallel-python/fig/calc_pi_3_wide.svg)

```python=
import random

def calc_pi(N):
    # N is sample size
    M = 0

    # The for loop makes this implementation very slow
    for i in range(N):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)
        if x**2 + y**2 < 1.0:
            M += 1

    return 4 * M / N
```
```python=
%%timeit
calc_pi(10**6)

# Example output: 697 ms ± 15.2 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

```
Before we start to parallelize this program, we need to do our best to make the inner function as efficient as we can. We show two techniques for doing this: vectorization using numpy and native code generation using numba.

We first demonstrate a Numpy version of this algorithm.

Numpy version (faster but not yet parallel)
```python=
def calc_pi_numpy(N):
    # Simulate impact coordinates
    pts = np.random.uniform(-1, 1, (2, N))
    # Count number of impacts inside the circle
    M = np.count_nonzero((pts**2).sum(axis=0) < 1)
    return 4 * M / N
```
```python=
%%timeit
calc_pi_numpy(10**6)

# Example output:
```
This is a vectorized version of the original algorithm. It nicely demonstrates data parallelization, where a single operation is replicated over collections of data. It contrasts to task parallelization, where different independent procedures are performed in parallel (think for example about cutting the vegetables while simmering the split peas).

If we compare with the ‘naive’ implementation above, we see that our new one is much faster:

### Challenge: Daskify
Write `calc_pi_dask` to make the Numpy version parallel. Compare speed and memory performance with the Numpy version. NB: Remember that dask.array mimics the numpy API.

#### Solution
```python=
import dask.array as da

def calc_pi_dask(N):
    # Simulate impact coordinates
    pts = da.random.uniform(-1, 1, (2, N))
    # Count number of impacts inside the circle
    M = da.count_nonzero((pts**2).sum(axis=0) < 1)
    return 4 * M / N

%timeit calc_pi_dask(10**6).compute()

# Output:
# 4.68 ms ± 135 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

### Numba
```python=
import numba

@numba.jit
def sum_range_numba(a: int):
    """Compute the sum of the numbers in the range [0, a)."""
    x = 0
    for i in range(a):
        x += i
    return x
```

Plain python version
```python=
%timeit sum(range(10**7))

# 190 ms ± 3.26 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```
Numpy version:
```python=
%timeit np.arange(10**7).sum()
# 17.5 ms ± 138 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```
Numba version:
```python=
%timeit sum_range_numba(10**7)
# 17.5 ms ± 138 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```


```python=
@numba.jit
def sum_range_numba_nontyped(a):
    """Compute the sum of the numbers in the range [0, a)."""
    x = 0
    for i in range(a):
        x += i
    return x
```
```python
%timeit sum_range_numba(10**7)
# Performance does not change a lot
```

### Challenge: Numbify calc_pi
Create a Numba version of comp_pi. Time it.

To remind you, the original function is this:
```python=
def calc_pi(N):
    # N is sample size
    M = 0

    # The for loop makes this implementation very slow
    for i in range(N):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)
        if x**2 + y**2 < 1.0:
            M += 1

    return 4 * M / N
```

#### Solution

```python=
@numba.jit
def calc_pi_numba(N):
    M = 0
    for i in range(N):
        # Simulate impact coordinates
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)

        # True if impact happens inside the circle
        if x**2 + y**2 < 1.0:
            M += 1
    return 4 * M / N

%timeit calc_pi_numba(10**6)

# 13.5 ms ± 634 µs per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

Creating a parallel version with numba:
We'll cover it tomorrow!

# 📚 Resources
- [Dask documentation](https://dask.org/)

# Questions
Q1: I may have missed it because I was dealing with an error, but why is the memory usage of dask so much lower and why does it have this saw tooth profile?
    - dask divides the work in chunks, so that it can handle arrays that are larger than your physical memory.
