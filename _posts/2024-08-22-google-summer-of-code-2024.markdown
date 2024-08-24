---
layout: single
title:  "Google Summer of Code, 2024"
date:   2024-08-24 15:48:19 +0530
tags: open-source gsoc cpp python
---

I participated in the Google Summer of Code, 2024 under the Python Software Foundation. I worked in **[LPython](https://lpython.org)**, a high performance typed Python compiler, and in particular **implementing and improving advanced data structures**. This project entailed:

- working primarily on dictionaries and sets, improving and implementing the in-built methods and functionalities on them
- working with nested data structures, improving their performance, both in time and memory usage
- benchmarking LPython with C++ using a large number of graph algorithms to identify where LPython lags behind, and identify where bugs can pop up in dictionary usage
- eliminate segmentation faults and other errors caused due to large data structures

> Weekly blogposts can be found [here](https://social.python-gsoc.org/@advikkabra)

### Benchmarks
I benchmarked LPython against C++ (`std::unordered_map`, and other hash maps as well), for a large number of algorithms.

> **[Detailed writeup and code](https://gist.github.com/advikkabra/0bf7a4fb8d8e1d050cb9e4e16841e2b7)**

Here is a quick summary, when both `--fast` and `-O3` are used respectively, using relative times (lower is better):

| Algorithm | LPython | `std::unordered_map` | `tsl::robin_map` |
| --------------- | --------------- | --------------- | --------------- |
| Dijkstra's algorithm | 1 | 40.55 | 11.75 |
| Depth First Search | 1 | 1.19 | 0.62 |
| Breadth First Search | 1 | 1.77 | 0.66 |
| Weakly Connected Components | 1 | 3.63 | 0.83 |
| Bron-Kerbosch Algorithm | 1* | 3.83 | 0.63 |
| PageRank | 1 | 0.89 | 0.59 |
| Community Detection using Label Propagation | 1 | 0.73 | 0.44 |
| Local Clustering Coefficient | 1 | 1.12 | 0.47 |

*: LPython was not compiled using `--fast`, as there is no support for it yet for this algorithm.

*Note that full times can be found in the writeup, and times when they are not compiled with additional flags.*


### Merged pull requests
- [#2705](https://github.com/lcompilers/lpython/pull/2705): **Initialize empty value to dictionaries without type given**: Quality of life change, dictionaries are initialized empty
- [#2710](https://github.com/lcompilers/lpython/pull/2710): **Add looping over dictionaries and sets**: Iterating over dictionaries (`for x in dict`)
- [#2711](https://github.com/lcompilers/lpython/pull/2711): **Add membership checks in dictionaries and sets**: `x in dict`
- [#2733](https://github.com/lcompilers/lpython/pull/2733): **Added debug capabilities for lists and sets**: Allow programs to be compiled with the `-g` flag using LLVM debug capabilities
- [#2735](https://github.com/lcompilers/lpython/pull/2735): **Allow subscripts to have methods called**: QoL change, `x[0].method()`
- [#2738](https://github.com/lcompilers/lpython/pull/2738): **Set deepcopy**
- [#2747](https://github.com/lcompilers/lpython/pull/2747): **Add clear method to dictionary and set**
- [#2749](https://github.com/lcompilers/lpython/pull/2749): **Add `set.pop` method**
- [#2751](https://github.com/lcompilers/lpython/pull/2751): **Fix segmentation fault if item is not in the list** 
- [#2753](https://github.com/lcompilers/lpython/pull/2753): **Fix memory leaks in the code**: Recursive freeing for nested data structures, freeing when strcutures are cleared, rehashed, popped, or deleted.
- [#2771](https://github.com/lcompilers/lpython/pull/2771): **Use `builder0` to allocate properly**: Use correct LLVM to call `createAlloca` at the start of function blocks
- [#2778](https://github.com/lcompilers/lpython/pull/2778): **Add support for in place sets and dictionaries in function calls**: Support for `fn({1, 2, 3})`
- [#2779](https://github.com/lcompilers/lpython/pull/2779): **Add `set()` for creating an empty set**
- [#2780](https://github.com/lcompilers/lpython/pull/2780): **Add additional rehashing for small sets**: Rehashing when the set is small, covering an edge case
- [#2783](https://github.com/lcompilers/lpython/pull/2783): **Prevent segmentation faults in particular cases of read checking**: There was a flaw in the previous search algorithm, which I fixed.
- [#2790](https://github.com/lcompilers/lpython/pull/2790): **Declare variables using builder0 to allocate**: Similar to #2771.
- [#2791](https://github.com/lcompilers/lpython/pull/2791): **Shallow assignment**: Prevent deepcopying in some cases of dictionary and set iteration.
- [#2802](https://github.com/lcompilers/lpython/pull/2802): **Prevent the hashing function being called when capacity is zero**: Prevent a floating point error (div by zero)
- [#2805](https://github.com/lcompilers/lpython/pull/2805): **Use memmove in list pop**: Significantly improves performance of popping in lists (upto 60x), by using `memmove` instead of individual moving.

### What I did
Over the course of the summer, I added many functionalities to dictionaries and sets - including iterating over them, searching, popping, clearing, deep copying, and properly freeing the memory used by them. Moreover, I added many quality-of-life updates, both in writing LPython code and developing the compiler. I reduced the memory leaks in LPython code, by implementing a recursive freeing logic when dictionaries and sets are rehashed, as well as other data structures being cleared. I benchmarked LPython against C++ code, and the exact details can be found [here](https://gist.github.com/advikkabra/0bf7a4fb8d8e1d050cb9e4e16841e2b7). LPython performs *significantly* better than C++ in most cases, and LPython was not on par only in the breadth first search algorithm, so I profiled and fixed the slowdowns as well. As a result of my work, LPython lists, dictionaries and sets are more stable, usable, use significantly less memory and faster than before.

### What's left to do
Right now, dictionaries and sets are in a very usable state and most commonly used methods are implemented. Additional methods, which are used less often need to be added (including `setdefault`, `intersection`, and more). In my benchmarking, LPython was slower than C++ in depth first search, PageRank, and breadth first search algorithms. I suspect is due to some optimization we are not doing when the `--fast` compile flag is passed. I think the the performance of LPython when `--fast` is passed can be improved in dictionaries and sets. Also, memory is leaked when dictionaries and sets go out of scope. This will be fixed by an ASR `Finalize` node which will be added in both LFortran and LPython. Printing dictionaries and sets by using `print()` can also be added as an ASR pass, like printing lists is done. 

### What I learned
This was my first experience in a large codebase, and I learnt how to navigate, identify bugs and errors, and development in such a large codebase. I also learnt other basic skills, like debugging with different tools (considering the incomplete debug flag support in LPython), profiling, identifying segmentation faults, memory leaks, slowdowns and more. In particular, I learnt how compilers are built from the ground up, and gained valuable experience in implementing performant, clean and safe code. I learnt the intricacies of [LLVM](https://llvm.org), and how to implement the algorithms I learnt in my courses in a compiler and how to account for all sorts of data input. I learnt how to work in a open source community, and how to write my code in the correct manner in the architecture set up in the project already. I have got a keen interest in compilers and programming languages thanks to this experience, and I will continue to explore and improve my skills in this field.

### Acknowledgements
I would like to thank my mentors, [Ondřej Čertík](https://github.com/certik/) and [Gagandeep Singh](https://github.com/czgdp1807). They were immensely helpful before and during this entire project. Additionally, a big thank you to everyone who has worked on this project before, and all the best to everyone who works on this in the future, and builds upon my work. I would also like to thank the Python Software Foundation for giving me this opportunity, and Google for organizing the Google Summer of Code program.

**Thank you!** If you want to know more about my project, you can contact me using my socials at the footer of this page, and on the sidebar.
