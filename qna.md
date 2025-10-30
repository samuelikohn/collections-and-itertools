# Q&A

### 1. In what types of workloads does collections.deque significantly outperform list, and how do their underlying data structures explain the difference in time complexity for insertions and deletions at both ends?

`deque`s are useful when you need to access, modify, add, or delete elements from both ends of a sequence. `collections.deque` actually uses a hybrid structure under the hood, pointers to the elements are stored in fixed-size, static arrays, a.k.a. "blocks", which are connected via a double-linked list. The structure keeps track of pointers to the first and last blocks in the list and indices of the first and last elements, which is why you can access and modify those elements in constant time. Appending and popping from the ends of a `deque` are constant time on average, extra operations occur when a block is added or deleted. You do get similar behavior from one end of a `list`, the `deque` has the advantage of providing this behavior at _both_ ends simultaneously. The downside is that there is no random access to the middle of a `deque` so operations performed on elements in the middle of a `deque` occur in linear time.

### 2. Do we know any tradeoff between using the first way to store counts in a dictionary (the first method you used) vs using Counter? Same for rotation with and without deque?

In terms of storing data, there's not much difference between a `Counter` and a `dict`. However, we typically store data so that we can use it in some way. Because a `Counter` _is_ a `dict`, any attributes or methods available for a `dict` are also available for a `Counter`. For the purpose of storing counts, `Counter`s offer extra functionality that `dict`s don't, such as methods to get the `n` most frequent elements or to get the sum of all counts. There may be some minor performance differences between the classes, but those are negigible (and if those differences matter, you probably shouldn't be using Python to begin with).

`deque`s have more interesting pros and cons compared to `list`s, which I mention more of in question 8. For the purpose of rotation specifically, `deques` are better, as the necessary operation occur only at the ends of a sequence, which is what `deque`s are optimized for.

### 3. Are there tradeoffs in the implementation used to achieve O(1) time for popping at both ends of a deque? Would performance be worse if we used deques instead of lists in every case?

Yes, due to how `deque`s are structured under the hood, accessing and operating on elements in the middle of a `deque` occur in linear time. I talk about this more in question 8. Whether to use a `deque`, `list`, or something else depends on your use case, `deque`s are the most useful when you need to access and modify both ends of a sequence of data. Say for example, you need to store a stream of data and look up values from any index frequently, but don't need to modify the structure other than appending new data. A `list` would be much better than a `deque` in this case. That's the point of learning data structures, to figure out which ones are best for your use case.

### 4. When using itertools.combinations, what happens internally if the input iterable is extremely large — does Python generate all combinations in memory, or does it yield them lazily?

They are lazily evaluated. All of the `itertools` functions are examples of `Iterator` objects. The way I like to think of these is that instead of storing data like a sequence does, iterators store instructions for how to generate that data. Each combination is only calculated after the iterator's `__next__` method is invoked, either via a `for` loop, the built-in `next()` function, or by a function that consumes iterators, such as `sum()`. As a result of this, you can't index or slice `Iterator` objects, even if the eventual sequence of data they produce is known, as with `combinations`.

### 5. I was curious about what the disadvantages of deque compared to a normal list: since it has more methods and keeps more info about the array, would it have a bigger space complexity compared to list?

Yes, the `deque`s internal structure maintains more information than a `list` does, so the actual memory used by a `deque` would be greater than a `list` for the same sequence of data. I explain more about how `deque`s work under the hood in question 1. I'm not sure whether a `list` or `deque` has more methods and class attributes, but the memory difference due to those is negligible. The difference comes from the extra pointers needed to link blocks together in a `deque`. In terms of asymptotic complexity, there is no difference, both are `O(n)`.

### 6. In this example, we count words as dictionary keys. How does Python determine which types of objects can serve as valid keys, and what would happen if we accidentally tried to use a mutable object like a list as a key?

`Counter`s don't change hashability rules from `dict`s. The same restrictions that we've learned about `dict`s apply to `Counter`s as well. Trying to instantiate a `Counter` with non-hashable types as keys would result in a `TypeError`.

### 7. When you switch from the manual dict frequency counter to Counter, does the memory usage for m unique words change, or is it the same up to constant factors?

I ran a few tests, for the same sequence of data a `Counter` object uses ever-so-slightly more memory than a `dict`. A `Counter` _is_ a `dict`, just with some extra functionality tacked on, so asymptotically (and practically), the memory usage is the same. If a few extra bytes matter that much for your use case, Python isn't the best choice to begin with.

### 8. Is using a deque worse in cases where you have to add elements in the middle of the list (based on how it is implemented internally) or in cases where a lot of indexing/slicing is done rather than updates?

Yes, one of the downsides of a `deque` is that accessing elements in the middle of the structure occurs in linear time, which is worse than a `list`. If you're use case requires access to random elements, `deque` is not the best option. Interestingly, `collections.deque` doesn't support slicing, although the concept of slicing is still valid. If you're curious about the internal structure of a `deque`, I mention it more in question 1. Inserting elements into the middle of `list`s or `deque`s are both linear time operations on average, barring any resize operations. However, inserting elements in the middle of a `deque` is actually slightly slower than a `list`, as you need to shift all elements in the block you're inserting into, which cascades through every block in the `deque`.

### 9. Using the timeit module was a nice touch. You mentioned that a lot of the functions are written in C, making it more efficient than when done in python. If you were to increase the size of each combination from 3 to another number, would the time saved still be around 15% as it was during your example?

Interesting question. I tested this a few different ways, if you want to see the details I added everything to the bottom of the notebook [here](https://github.com/samuelikohn/collections-and-itertools/blob/main/main.ipynb). TL;DR, as you scale up both the number of combinations and the length of each combination, the percentage performace gain increases. In both languages, the length of the combination affects the time more than the number of combinations, so while both are significant (over 90% faster in some test cases), `itertools.combinations` helps you more for longer combinations. Although, it seems that the operation you're performing on the combinations matters more than the iteration method, based on the numbers I found vs. the 10-15% in the video. Interestingly, there was a case where doing things without `itertools` was faster, which may warrant more looking into.

### 10. The use of ordered dict in your 'dollar-store-LRU cache' was very clever. I am trying to see how the running time of this code might be if we scale up into a longer dict. Right now it appears to be O(1) time Is there a way to dynamically resize the cache using existing itertools in collections?

My implementation of the LRU cache isn't great, I just wanted to give a simple example of how `OrderedDict` works and where it could be useful. You wouldn't want to modify your function's behavior to add cache functionality, which is exactly what decorators intend to solve. The regular Python `dict` actually keeps its keys in the order they were inserted, the advantage of `OrderedDict` is that you can manipulate the keys into any order to fit your use case (and the ordering is considered in `OrderedDict` for equality tests). The reason I mention this is because of how `OrderedDict` maintains its ordering: via a double-linked list. Most `OrderedDict` operations, including `move_to_end` and `popitem`, are still `O(1)`, but with a larger constant factor compared to `dict`. So, maintaining an LRU cache of any size is `O(1)`.

The `functools.lru_cache` decorator doesn't actually let you change its `max_size`. Pragmatically, if you want the size of your cache to be dynamic, the `cache` decorator is probably better, as it doesn't have a size limit and is faster than `lru_cache`. But to answer your question, I hard coded a limit of `5` entries into my implementation, you could easily replace that with an expression that evaluates whatever conditions you're interested in to calculate a size limit.

### 11. If you chain multiple itertools functions together (like using combinations inside a product or chain), how does Python handle that under the hood? Does it produce it one result at a time as you loop over them or does it ever materailize intermediate results in memory?

Good question. Generally, `Iterator` objects are lazily evaluated. If you want more detail, look at question 4. If I nest multiple iterators inside one another, Python will evaluate them as lazily as possible. In my "Just for fun" example from the notebook, I nest `itertools.count` inside of `map`. The outer function, in this case, `map`, will always be lazily evaluated. `map` consumes one element of an iterable at a time, and `count` produces one element each time it is iterated over, so `count` is also lazily evaluated in this case. To your question, `combinations` and `product` both require multiple elements from the iterables they consume and don't evaluate them in order, so the inner iterators will be fully materialized and held in memory.

This code demonstrates this with a generator as the inner iterable.

```python
import itertools

def my_generator():
    print("Generating 1")
    yield 1
    print("Generating 2")
    yield 2
    print("Generating 3")
    yield 3

# This line will evaluate the entire generator
combos = itertools.combinations(my_generator(), 2)
print("Created combinations object")

# Now iterate through combinations
for combo in combos:
    print(combo)
```

```
Generating 1
Generating 2
Generating 3
Created combinations object
(1, 2)
(1, 3)
(2, 3)

```
