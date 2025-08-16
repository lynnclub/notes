---
title: "Data Structures"
type: "docs"
weight: 2
---

## Common Structures

1. Linked List: Non-contiguous memory allocation, each element contains two nodes: data field and pointer.
2. Array: Contiguous memory allocation, accessed through indices.
3. Queue: One end in, one end out, first-in-first-out (FIFO).
4. Stack: Operations at the top end only, last-in-first-out (LIFO).
5. Heap: Usually can be seen as an array object of a complete binary tree.
6. Tree: Finite nodes with hierarchical relationships.
7. Graph: Vertices and edges, divided into directed and undirected graphs.
8. Hash Table: Key-value pairs. Uses buckets to improve efficiency, handles conflicts through linked lists or red-black trees.

Linked lists, arrays, queues, and stacks are linear data structures, while heaps, trees, graphs, and hash tables are non-linear data structures. Linked lists, arrays, queues, stacks, heaps, trees, and graphs support sequential access; arrays also support random access; hash tables only support random access.

Tree sequential access methods include pre-order traversal, in-order traversal, and post-order traversal. Graph sequential access methods include depth-first traversal and breadth-first traversal.

## Heap and Stack in Memory

Heap and Stack are two important data structures in computer science used for storing and managing data in memory.

Stack organizes data in a contiguous manner, used for storing function calls, local variables, parameters, and program execution context at runtime. When a function is called, the stack allocates memory space for storing the function's parameters and local variables. When function execution completes, the stack releases the memory space occupied by that function - automatic allocation, automatic release.

Heap is a method of dynamic memory allocation that stores data in an unordered manner and can be allocated and released at any time. Used for creating objects, data structures, or dynamic arrays at runtime. Memory allocated in the heap needs to be manually released, otherwise it may cause memory leaks.

## Related Algorithm Problems

Question: Given 100 billion integers, find the largest 100.
Answer: Take 100 numbers to build a binary tree, iterate to eliminate.
