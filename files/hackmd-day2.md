# 2020-12-07 Parallel Python, day 2


Welcome to The Workshop Collaborative Document


This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

All content is publicly available under the Creative Commons Attribution License

https://creativecommons.org/licenses/by/4.0/

 ----------------------------------------------------------------------------

This is the Document for today: [link](https://hackmd.io/@O0tsDNPbTlyhyGiiCMaLIw/rylYGmScD)

Collaborative Document day 1: [link](https://hackmd.io/@O0tsDNPbTlyhyGiiCMaLIw/ByWbNGE5w)

Collaborative Document day 2: [link](https://hackmd.io/@O0tsDNPbTlyhyGiiCMaLIw/rylYGmScD)



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
[link]



## 🛠 Setup
[link]


## 🧑‍🏫 Instructors

Johan Hidding, Pablo Rodríguez-Sánchez


## 🧑‍🙋 Helpers

Helper 1, Helper 2


## 🗓️ Agenda

| time  | what |
|-------|------------------------|
| 09:00 | Welcome and icebreaker |
| 09:15 | Recap |
| 09:30 | Accelerate code using Numba |
| 10:30 | Coffee break |
| 10:45 | Parallel design patterns with Dask Bags |
| 11:45 | Tea break |
| 12:00 | Workflows with Snakemake |
| 12:45 | Post-workshop Survey |
| 13:00 | END |



# 🧠 Collaborative Notes

## Accelerate code using numba
We will need the following implementation of calculating pi using numba:

```python=
@numba.jit
def calc_pi(N):
    M = 0
    for i in range(N):
        # Simulate impact coordinates
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)

        # True if impact happens inside the circle
        if x**2 + y**2 < 1.0:
            M += 1
    return 4 * M / N

%timeit calc_p(10**6)
```

## Parralelizing our function
```python=
pi_estimates = [calc_pi(10**7) for i in range(10)]
sum(pi_estimates)/len(pi_estimates)
```

Let's look into parallelizing this using basic python libraries:
```python=
import threading
import queue
from itertools import repeat

def worker(q: queue.Queue):
    while True:
        try:
            x = q.get(block=False)
            print(calc_pi(x), end= ' ', flush=True)
        except queue.Empty:
            # Queue.Empty is an exception that is thrown when q.get is called while the queue is empty
            break

work_queue = queue.Queue()
input_range = repeat(10**7, 10)

# Filling the queue
for i in input_range:
    work_queue.put(i)

ncpus = 4
threads = [
    threading.Thread(target=worker, args=(work_queue,))
    for i in range(ncpus)]

for t in threads:
    t.start()

# Wait until all threads are done
for t in threads:
    t.join()
```

In jupyter: shift-m to merge cells

## A few words about the Global Interpreter Lock
The Global Interpreter Lock (GIL) is an infamous feature of the Python interpreter. It both guarantees inner thread sanity, making programming in Python safer, and prevents us from using multiple cores from a single Python instance. When we want to perform parallel computations, this becomes an obvious problem. There are roughly two classes of solutions to circumvent/lift the GIL:

- Run multiple Python instances: multiprocessing
- Have important code outside Python: OS operations, C++ extensions, cython, numba

The downside of running multiple Python instances is that we need to share program state between different processes. To this end, you need to serialize objects using pickle, json or similar, creating a large overhead. The alternative is to bring parts of our code outside Python. Numpy has many routines that are largely situated outside of the GIL. The only way to know for sure is trying out and profiling your application.

To write your own routines that do not live under the GIL there are several options: fortunately numba makes this very easy.

```python=
@numba.jit(nopython=True, nogil=True)
def calc_pi_numba(N):
    M = 0
    for i in range(N):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)

        if x**2 + y**2 < 1.0:
            M += 1
    return 4 * M / N

```
The nopython argument forces Numba to compile the code without referencing any Python objects, while the nogil argument enables lifting the GIL during the execution of the function.

### Exercise:
Change number of jobs and input parameter for `calc_pi`. Make a plot of the time performance. (Memory profile is not needed)

Tips:
- Use semicolons for multiline setup

#### Progress
- Room 1: For using the setup you need to use `setup="from __main__ import YOURDEF"`
- Room 2:
- Room 3: trying to figure out how to use the timeit.timeit
    - Multiline instructions can be passed to `timeit.timeit` as `"<instruction 1>;<instruction 2>""`. For example: `setup="import dask.array as da; import numpy as np"`
- Room 4:
- Room 5: how do we setup for timeit?
    - Multiline instructions can be passed to `timeit.timeit` as `"<instruction 1>;<instruction 2>""`. For example: `setup="import dask.array as da; import numpy as np"`
    - I used globals() to import the global scope, so I could sue a setup function and then call the other function --> I think this is a cleaner solution no?  (link redacted)

#### A horribly complicated solution that was not needed
Use `In[<number>]` to get a string containing code in a cell.

```python=
from timeit import timeit
from functools import partial
from IPython.core.magic import (needs_local_scope, register_cell_magic)
import ast

@register_cell_magic
@needs_local_scope
def tune(line, cell, local_ns):
    setup, pars = eval(line, None, local_ns)
    return [timeit(
                stmt=cell.format(**p),
                setup=setup, number=1)
                for p in pars]
```

This cell contains all the commands needed for `setup`. You may access this by entering it into Jupyter and then use `In[...]` with the number of the input cell.

```python
import threading
import queue
from itertools import repeat
import numba
import numpy as np
import random


@numba.jit(nopython=True, nogil=True)
def calc_pi(N: np.int64) -> np.float64:
    M = 0
    for i in range(N):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)
        if x**2 + y**2 < 1.0:
            M += 1
    return 4 * M / N


def worker(q: queue.Queue):
    while True:
        try:
            x = q.get(block=False)
            print(calc_pi(x), end=' ', flush=True)
        except queue.Empty:
            break
```

The `tune` magic takes two arguments: the setup script, and a list of dictionaries containing the tuning parameters. These parameters are expanded using `str.format`. This means that if you use any `{` or `}` characters in the code enter them twice (not the case here).

```python=
%%tune In[32], [{"ncpus": n} for n in range(1, 5)]

work_queue = queue.Queue()
input_range = repeat(10**7, 10)
for i in input_range:
    work_queue.put(i)

ncpus = {ncpus}
threads = [
    threading.Thread(target=worker, args=(work_queue,))
    for i in range(ncpus)]

for t in threads:
    t.start()

for t in threads:
    t.join()
```

#### Nicer solution (by Johan vdW)
Create two functions: `setup` and `fill_queues`, then:

```python=
x = [timeit.timeit(
        stmt=f'fill_queue({n}, tasks=20)',
        setup='worker = setup()',
        globals=globals(),
        number=1) for n in range(1, 3)]
```

## ☕ Break
Until 10:24

## Parallel design patterns with Dask Bags
Dask is one of the many tools available for parallelizing Python code in a comfortable way. We’ve seen a basic example of `dask.array` in a previous episode. Now, we will focus on the `bag` and `delaye` sub-modules. Dask has a lot of other useful components, such as dataframe and futures, but we are not going to cover them in this lesson.

Dask bags let you compose functionality using several primitive patterns: the most important of these are map, filter, groupby and reduce

Operations on this level can be distinguished in several categories:

- map (N to N) applies a function one-to-one on a list of arguments. This operation is embarrassingly parallel.
- filter (N to <N) selects a subset from the data.
- groupby (N to <N) groups data in subcategories.
- reduce (N to 1) computes an aggregate from a sequence of data; if the operation permits it (summing, maximizing, etc) this can be done in parallel by reducing chunks of data and then further processing the results of those chunks.

Let’s see an example of it in action:

First, let’s create the bag containing the elements we want to work with (in this case, the numbers from 0 to 5).

```python=
import dask.bag as db

bag = db.from_sequence(range(6)) # Contains 0, 1, 2, 3, 4, 5
```

### Map
To illustrate the concept of map, we’ll need a mapping function. In the example below we’ll just use a function that squares its argument:

```python=
# Create a function for mapping
def f(x):
    return x**2

# Create the map and compute it
bag.map(f).compute()

# out: [0, 1, 4, 9, 16, 25]
```

```python=
# Visualize the map
bag.map(f).visualize()
```
![map diagram](https://carpentries-incubator.github.io/lesson-parallel-python/fig/dask-bag-map.svg)

### Filter
To illustrate the concept of filter, it is useful to have a function that returns a boolean. In this case, we’ll use a function that returns True if the argument is an even integer, and False if it is odd:
```python=
# Return True if x is even, False if not
def pred(x):
    return x % 2 == 0

bag.filter(pred).compute()
```

#### Question
Without executing it, try to forecast what would be the output of `bag.map(pred).compute()`?

Answer:
The output will be `[True, False, True, False, True, False]`.

### Reduction
```python=
bag.reduction(sum, sum).visualize()
```
### Challenge
Look at the `mean`, `pluck`, `distinct`, and `topk` methods. Match them up with map, filter, groupby and reduce methods.

Answers:

- `mean`: reduction
- `topk` : filter
- `pluck`: ~~filter~~ map
- `distinct` : groupby | map (count)

### Challenge
Rewrite the following program in terms of a Dask bag. Make it spicy by using your favourite literature classic from project Gutenberg as input. Example: Adventures of Sherlock Holmes, (https://www.gutenberg.org/files/1661/1661-0.txt).

#### Hints
- db has a method called `read_text`
- Suggested first line `bag = db.read_text('1661-0.txt', blocksize="32k")`
- Suggested second line `raw_words = bag.str.split().flatten()`
- Bag has a method called `distinct`
- Bag has a method called `count`


```python=
from nltk.stem.snowball import PorterStemmer # pip install nltk
stemmer = PorterStemmer()

def good_word(w):
    return len(w) > 0 and not any(i.isdigit() for i in w)

def clean_word(w):
    return w.strip("*!?.:;'\",“’‘”()_").lower()
```

Apply it to a text example:
```python=
text = 'Lorem ipsum'
```

```python=
clean_words = map(clean_word, text.split())
good_words = filter(good_word, clean_words)
stems = map(stemmer.stem, good_words)
words = set(stems)

print(f'This corpus contains {len(words)} unique words.")

# Out: This corpus contains 2 unique words.
```
(https://www.gutenberg.org/files/1661/1661-0.txt)

You have until 11:17

### With dask
```python=
bag = db.read_text('../1661-0.txt', blocksize='32k')
raw_words = bag.str.split().flatten()
clean_words = raw_words.map(clean_word).filter(good_word)
stems = clean_words.map(stemmer.stem)
unique_words = stems.distinct().count()

unique_words.compute(scheduler='processes', num_workers=4)

#out: 6323
```
Run `unique_words.visualize()` to learn the execution structure.

### Dask delayed
A lot of the functionality in Dask is based on top of a framework of delayed evaluation. The concept of delayed evaluation is very important in understanding how Dask functions, which is why we will go a bit deeper into `dask.delayed`.

```python=
from dask import delayed
```
The delayed decorator builds a dependency graph from function calls.
```python
@delayed
def add(a, b):
    result = a + b
    print(f"{a} + {b} = {result}")
    return a + b
```

A `delayed` function stores the requested function call inside a promise. Nothing is being done yet.
```python
x_p = add(1, 2)
```
We can check that x_p is now a Delayed value.
```python
type(x_p)
#out: dask.delayed.Delayed
```
#### Note
It is often a good idea to suffix variables that you know are promises with `_p`. That way you keep track of promises versus immediate values.




From Delayed values we can create larger workflows.
```python=
x_p = add(1, 2)
y_p = add(x_p, 3)
z_p = add(x_p, y_p)
z_p.visualize(rankdir="LR")
```
Output:
![delay visualization](https://carpentries-incubator.github.io/lesson-parallel-python/fig/dask-workflow-example.svg)

#### calculating pi with delayed
Our old calc_pi function
```python=
import random

@numba.jit
def calc_pi(N):
    M = 0
    for i in range(N):
        # Simulate impact coordinates
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)

        # True if impact happens inside the circle
        if x**2 + y**2 < 1.0:
            M += 1
    return 4 * M / N
```

```python=
x_p = delayed(calc_pi)(10**7)
```

```python=
@delayed
def gather(*args):
    return list(args)

@delayed
def mean(*args):
    return sum(args)/len(args)
```

```python=
pi_p = mean( *(x_p for i in range(10)))
pi_p.compute()

# out: 3.14153852
```

### Key points
- Use abstractions to keep programs manageable


## 🍵 Break
We'll be back at 12:00
then 📸


## Snakemake
Bags use multiprocessing and this will create overhead.

### Cosmic structure example
https://github.com/escience-academy/parallel-python-workshop

Notebook dir:
https://github.com/escience-academy/parallel-python-workshop/tree/main/cosmic_structure

Steps:
- Create new folder
- Create a new file named Snakefile
- Write rules in snakefile (see below at dependency diagram)
- Press + sign in upper left of jupyter lab
- Create new terminal
- Go to directory with new Snakefile within terminal
- run `snakemake -j1` in terminal

### Rules
A rule looks as follows
```python=
rule <rule name>:
    <rule specs>
```
Each rule may specify input and output files and some action that describes how to execute the rule.


### Dependency diagram
Example Snakefile
```python=
rule all:
    input:
        "allcaps.txt"

rule generate_message:
    output:
        "message.txt"
    shell:
        "echo 'Hello, World!'>{output}"

rule shout_message:
    output:
        "allcaps.txt"
    input:
        "message.txt"
    shell:
        "tr [a-z] [A-Z] < {input} > {output}"

```

View the dependency diagram: `snakemake --dag | dot -Tsvg > hello-graph.svg`, and run the workflow `snakemake -j1`.

On debian/ubuntu you can use `apt install graphviz`, on mac `brew install graphviz` to have the dot binary. Or use `conda install graphviz` from your environment.

#### Exercise
Edit snakefile so that it concatenates the output from `message.txt` and `allcaps.txt`.

Hint: use the `cat` command to concatenate files.
Hint2: `{input}` will expand a list of multiple input files.

until 12:40


For mac users, to create dot files you have to install graphviz (`brew install graphviz`)

This will change your Python to 3.9

- Graphviz is also included in the conda environment, so you should be able to call it if you activated.

#### Status
- Group 1: We came up with this:
```python=
rule all:
    input:
        "long_file.txt"

rule generate_message:
    output:
        "message.txt"
    shell:
        "echo 'Hello, World!' > {output}"


rule shout_message:
    input:
        "message.txt"
    output:
        "allcaps.txt"
    shell:
        "tr [a-z] [A-Z] < {input} > {output}"

rule long_file:
    input:
        f="message.txt",
        s="allcaps.txt"
    output:
        "long_file.txt"
    shell:
        "cat {input.f} {input.s} > {output}"
```

- Group 2:
- Group 3: concatenation works, some of us can not write the file to disk (no error from snakemake)
    - Any specifics about the environment? Mac, windows, linux?
    - I think we all got it working in the end.  Thanks.
- Group 4:
- Group 5:

# 🦸‍♀️🦸‍♂️ post-workshop survey
https://www.surveymonkey.com/r/YZS2BBQ

# 💁 🎩 tip/top
## one thing you liked most
- Jeroen: already put feedback in survey
  - Many thanks to you guys!
- I really liked the breakouts. For most of the breakout sessions, the number of conceptual steps that we had to make together was just right, so we didn't get stuck and didn't get bored (and when we did, we just looked for multiple solutions).
- Great course, thanks!  I also liked the breakouts.
- Really like the collaborative document concept :+1:
-

## one tip on how we can improve
- Jeroen: already put feedback in survey
- I kept having connection problems. When I came back, I didn't know if anything important was shared in the Zoom chat, and couldn't see the Zoom chat history, afaik. Similarly, I keep losing connection to the collab doc, which makes it so hard to contribute anything :cry:

# 📚 Resources
- [Timeit docs](https://docs.python.org/3/library/timeit.html)
- [Dask docs](https://docs.dask.org/en/latest/)

# Questions
- Can you make your screen full width? I can't read the code with my elderly eyes
    - Is the size ok now?
    - Yes, thanks

- how to activate the help as Johan has just did in a notebook, I mean to see the details of commands in python modules?
    - combine the keystrokes SHIFT + TAB when the cursor is on the function

- And the color combination, how can we tune it in our notebooks? Thanks

- What was the combination to combine?
    - shift-m
    - you can use Shift-J/K to select multiple cells and Shift-M will merge all the selected cells.

- On returning values to the main thread: in the "slow" (actually not parallel) calc_pi on threading, the value of $\pi$ is printed for each thread. In the fast version, the value of $\pi$ is returned. However, I do not see a mechanism that catches the 10 approximations of $\pi$ on the main thread. So, each thread computes $\pi$ and returns it into the void?
    - Yes, we're just timing it for now

- I used globals() to import the global scope, so I could sue a setup function and then call the other function --> I think this is a cleaner solution no? (link redacted out)
    - Thanks! that is much more elegant

- Do the elements in the bag, need to be of the same datatype?
    - No, but it is strongly adviced to do so.

- How can you see the content of one bag's content?
    - You can `.compute()` it, if you like to save the computation, (for use in subsequent evaluation), call `.persist()` method. That way the computation stays in memory.

- When using `Snakemake`, it says `Building DAG of jobs...`. Does DAG stand for Directed Acyclic Graph...?
    - yes

- What is gone wrong when terminal says: `zsh: command not found: snakemake`? The package seems installed.
    - you need to activate the conda environment using `conda activate parallel-python`

- So with Pablo's solution, why does the pdf generating rule not show up in the dependency diagram?
    - It was a command that was independent. It was not linked to any other rule or output.
        - Any commands to force it to also display the independent rules?
    - It is better to use it directly on the command line: `snakemake --forceall --dag | dot -Tsvg > diagram.svg `

- Why do we use the terminal now, and start up conda?
    - To call Snakemake. You should have activated conda before. `snakemake` is a command-line tool.

- Can  you refer to the outputs in a different way rather than just the filenames
    - No (not that I know of), snakemake is file based.
    - Djura: breakout room 1 found out that you can assign variables to inputs, like
    ```python
    input:
        a="file1.txt"
    shell:
        "cat {input.f} > {output}"
    ```


- When including python into the snakefile, how to test/debug these python commands/snippets?
    - Not sure about the most proper way to do this: you can see the output files, test code before putting it in snakemake, write tests using PyTest, etc..
        - Q? test code before putting into snakemake is fine for once, but what about maintainability when code further evolves? Duplicate code with a pytest-ed version and the snakefile version?


- How would you go about using `dask` in a situation where the functions you call are actually just binaries (e.g. compiled `C++` programs)? Does it make sense to use `dask` in that context?
    - It could make more sense to execute your workflow in snakemake depending on the context. If you want to execute a lot of data transformations using dask you could use that and call the binaries using a`subprocess` call.
- Somehow my snakemake command only operates the first rule and stops before executing the second and following ones...
    - It seems by design - it will only run other steps if they are required for that step. You can run `snakemake shout_message` to run another step.
        - Solved, thanks!
