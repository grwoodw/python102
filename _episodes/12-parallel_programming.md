---
title: Parallel Programming in Python
teaching: 60
exercises: 30
questions:
- "How can I make my code go faster?"
objectives:
- "A brief exposure to parallel programming in Python"
keypoints:
- "Identify independent sections of code where parallelism is possible"
- "Use the Multiprocessing library to parallelize your code"
---

CPU clock speeds have been fairly stagnant for years.  To make our tasks finish in less time and sell new processors Engineers have had to get creative.  They have tried improving and increasing cache, new memory models and fetching techniques but so far the most successful has been increasing the number of cores or processors on a CPU.  Unfortunately this is not a plug and play solution for most software.  Software has to be written to take advantage of multiple cores or processors and not all problems are well suited to parallel processing.  To take advantage of parallel architectures problems must have sections of work that can be processed independently.  Parallel programming takes time to master, in this module we hope to get your feet wet with a couple examples and provide references for continued learning.

> ## Note on Parallel Programming in Windows
>
> Parallel programming in windows can be a little more challenging and yield smaller gains than the same process in linux.  This is due to the way the Operating Systems handle multiple threads and processes.  One of these challenges is parallel code will not run in Jupyter Notebooks (ipython interpreter) on windows, while it works just fine in linux.  To get around this in windows we can write our script in a `.py` file and run it from the command line with `$ python myScript.py`.  One other challenge is that these scripts must have a main function defined.  So create a starting point for your script in an if statement like you see below:
>
> ~~~
> if __name__ == '__main__':
>     grid_size = 512
>     graph = generate_grid(grid_size)
>     . . . 
> ~~~
> {: .python}
{: .callout}


## All Pairs Shortest Path
---
We'll work through an example trying to find the all pairs shortest path in a weighted directed network graph using the Floyd Warshall algorithm.  First we'll create our graph and network path weights.  The graph and weights are represented by a 2d matrix where `grid[r,c]` is the cost to go from node `r` to node `c` where no connection is represented by a weight of infinity.  We'll create this as a function so we can use it over and over again.

~~~
import numpy as np
import math

def generate_grid(size):
    grid = np.random.randint(1,20,size=(size,size))
    def inf(x):
        return math.inf if x >= 5 else float(x)
    inf = np.vectorize(inf)
    return inf(grid)
~~~
{: .python}

I've chosen to create a graph using random weights between 1 and 20 (inclusive) and changing any value greater than or equal to 5 to infinity which limits the edges between nodes and makes for a more interesting result.

~~~
print(generate_grid(8))
~~~
{: .python}
~~~
[[inf  1. inf inf inf inf inf inf]
 [inf inf inf inf inf inf inf inf]
 [inf  4. inf  1. inf inf inf  3.]
 [inf  3. inf inf  2. inf inf inf]
 [inf inf inf inf inf inf inf  1.]
 [inf inf inf inf inf  3. inf  1.]
 [inf inf inf  1. inf inf inf  4.]
 [inf  1. inf inf inf inf  4. inf]]
~~~
{: .output}
  
Next let's define our Floyd Warshall algorithm.  Don't worry if it doesn't make sense at this time, just know that it works and it takes O^3 time to compute:

~~~
def floydWarshall(g):
    n = g.shape[0]
    for k in range(n): 
        # pick all vertices as source one by one 
        for i in range(n): 
            # Pick all vertices as destination for the 
            # above picked source 
            for j in range(n): 
                # If vertex k is on the shortest path from  
                # i to j, then update the value of g[i][j] 
                g[i][j] = min(g[i][j],g[i][k]+ g[k][j])
    return g
~~~
{: .python}

Let's run our algorithm and review our results:

~~~
# Start with a small grid so it's easy to view the results
grid_size = 8
graph = generate_grid(grid_size)
print('Graph:\n',graph)
shortest_paths = floydWarshall(graph)
print('\nShortest Paths:\n',shortest_paths)
~~~
{: .python}
~~~
Graph:
 [[inf inf inf inf inf inf inf inf]
 [inf inf  4.  3. inf inf inf  4.]
 [inf  4. inf  4. inf inf inf inf]
 [ 4. inf  3. inf inf inf inf  3.]
 [ 4.  2. inf  2. inf  2. inf inf]
 [inf inf inf inf inf  4. inf inf]
 [ 2. inf inf inf inf  4. inf inf]
 [inf inf inf inf  4.  2. inf inf]]

Shortest Paths:
 [[inf inf inf inf inf inf inf inf]
 [ 7.  8.  4.  3.  8.  6. inf  4.]
 [ 8.  4.  7.  4. 11.  9. inf  7.]
 [ 4.  7.  3.  7.  7.  5. inf  3.]
 [ 4.  2.  5.  2.  9.  2. inf  5.]
 [inf inf inf inf inf  4. inf inf]
 [ 2. inf inf inf inf  4. inf inf]
 [ 8.  6.  9.  6.  4.  2. inf  9.]]
~~~
{: .output}

The output `shortest_paths[r,c]` shows the weight or cost of the shortest path from node `r` to node `c`.  This algorithm can be modified to provide the actual path as well, but we won't complicate our example with that at this time.  There are two things we care about when making code parallel:
1. The time it takes to run our code should improve
2. The answer has not changed

Let's start by timing our code:

~~~
import time as t

grid_size = 8
graph = generate_grid(grid_size)

t1 = t.time()
shortest_paths = floydWarshall(graph)
t2 = t.time()
print('serial: ',t2 - t1, 's')
~~~
{: .python}
~~~
serial:  0.004255056381225586 s
~~~
{: .output}

Try increasing grid_size to see what it does to the execution time.  Each time you double the grid size it should increase the execution time by about 8x.  I would not recommend going above 200 for now.

## Parallel All Pairs Shortest Path
---
Next let's try making a parallel version of our code.  At first glance it's not obvious, but iterations of the outter loop `k` are not independent, so let's start by trying to parallelize the middle loop.  To do this we will split our algorithm into two parts: a serial part (p1) and a parallel part (p2).  We will create a processing pool (group of independent processes that we will assign work to) and a tool in the multiprocessing library called `multiprocessing.map` which will map input to a function and a process in our pool.  Essentially what we have done is replace our `i` loop with a call to `mp.map` which runs each iteration of the `i` loop independently and then we merge our results together before we continue to the next iteration of our `k` loop.

~~~
import multiprocessing as mp
from functools import partial

def floydWarshall_p2(i, g, n, k):
    # Pick all vertices as destination for the 
    # above picked source 
    for j in range(n): 
        # If vertex k is on the shortest path from  
        # i to j, then update the value of dist[i][j] 
        g[i][j] = min(g[i][j],g[i][k]+ g[k][j])
    return (i,g[i])

def floydWarshall_p1(g):
    n = g.shape[0]
    pool = mp.Pool(processes=mp.cpu_count())
    for k in range(n):
        p = partial(floydWarshall_p2, g=g,n=n,k=k)
        result_list = pool.map( p,range(n))
        for result in result_list:
            g[result[0]] = result[1]
    pool.close()
    pool.join()
    return g
~~~
{: .python}

Let's run our new parallel code and see how much faster it is:

~~~
grid_size = 8
graph = generate_grid(grid_size)

t1 = t.time()
shortest_paths = floydWarshall_p1(graph)
t2 = t.time()
print('parallel: ',t2 - t1, 's')
~~~
{: .python}
~~~
parallel:  0.13022136688232422 s
~~~
{: .output}

Wait?  Why is our code slower?  It takes time to create a parallel pool and assign work to all those processes, if we don't have enough work to do the time to create and assign work to our parallel processes can easily dwarf the time it takes to process.  Let's try both serial and parallel methods again with a larger grid size.

~~~
grid_size = 200
graph = generate_grid(grid_size)

t1 = t.time()
shortest_paths = floydWarshall(graph)
t2 = t.time()
print('serial: ',t2 - t1, 's')

t1 = t.time()
shortest_paths = floydWarshall_p1(graph)
t2 = t.time()

print('parallel: ',t2 - t1, 's')
print('number of cores: ',mp.cpu_count())
~~~
{: .python}
~~~
serial:  8.03958797454834 s
parallel:  3.931276559829712 s
number of cores:  8
~~~
{: .output}

Ok, our parallel version is roughly 2x faster (your results may vary depending on your hardware), but how do we know we got the right answer?  Let's save our results into two different arrays and compare them.  Numpy makes this easy with `np.array_equal()`:

~~~
grid_size = 200
graph = generate_grid(grid_size)

t1 = t.time()
shortest_paths_1 = floydWarshall(graph)
t2 = t.time()
print('serial: ',t2 - t1, 's')

t1 = t.time()
shortest_paths_2 = floydWarshall_p1(graph)
t2 = t.time()

print('parallel: ',t2 - t1, 's')
print('number of cores: ',mp.cpu_count())
print('results equal (1vs2): ',np.array_equal(shortest_paths_1, shortest_paths_2))
~~~
{: .python}
~~~
serial:  7.52128005027771 s
parallel:  3.434966564178467 s
number of cores:  8
results equal (1vs2):  True
~~~
{: .output}

> ## Challenge
> 
> Why did we only get a 2x speedup when we used 8 processors, shouldn't we get an 8x speedup?
> > ## Solution
> > There are two primary reasons for this:
> > 1. We did not parallelize the entire algorithm, only a portion of it and our measurements included both the parallel and serial portions
> > 2. This problem is memory bound, meaning we spend more time moving data in and out of the processor than we do calculating results based on that data, so our processors are often waiting for data to be loaded and not 100% utilized
> > 
> {: .solution}
{: .challenge}

In this section we have only touched on one of the methods Python has available for multiprocessing.  Please see [Python's Documentation](for more information)