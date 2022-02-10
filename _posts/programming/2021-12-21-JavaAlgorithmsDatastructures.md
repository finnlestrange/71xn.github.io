---
layout: single
title: "Java Algorithms & Data Structures"
date: 2021-12-05 08:49:23 +0100
categories: programming
excerpt: ☕ Advanced data structures and algorithms in Java 16
---

# ☕ Data Structures & Algorithms Implemented in Java 16

## Data Structures

> [Source Code Folder](https://github.com/71xn/algorithmsDataStructures/tree/main/dataStructures/src/tech/finnlestrange)

- [Stack](https://github.com/71xn/algorithmsDataStructures/blob/4e090e71e609c6e3af82a00d8e426040200dd254/dataStructures/src/tech/finnlestrange/JavaStack.java)
- [Queue](https://github.com/71xn/algorithmsDataStructures/blob/4e090e71e609c6e3af82a00d8e426040200dd254/dataStructures/src/tech/finnlestrange/JavaQueue.java)
- [Priority Queue](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/JavaPriorityQueue.java)
- [Dynamic Arrays](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/JavaDynamicArray.java)
- [LinkedLists](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/JavaLinkedList.java)
- [List Comparison](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/ListComparison.java)
- [Hash Tables / HashMaps](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/HashTables.java)
- [Graphs](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/Graphs.java)
- [Adjacency Lists](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/AdjacencyLists.java)
- [Binary Search Tree](https://github.com/71xn/algorithmsDataStructures/blob/main/dataStructures/src/tech/finnlestrange/BinarySearchTree.java)

## Algorithms

> [Source Code Folder](https://github.com/71xn/algorithmsDataStructures/tree/main/algorithms/src/tech/finnlestrange)

- [Linear Search](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/LinearSearch.java)
- [Binary Search](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/BinarySearch.java)
- [Recursion](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/Recursion.java)
- [Interpolation Search](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/InterpolationSearch.java)
- [Bubble Sort](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/BubbleSort.java)
- [Insertion Sort](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/InsertionSort.java)
- [Selection Sort](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/SelectionSort.java)
- [Merge Sort](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/MergeSort.java)
- [Quick Sort](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/QuickSort.java)
- [Depth First Search](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/DepthFirstSearch.java)
- [Breadth First Search](https://github.com/71xn/algorithmsDataStructures/blob/main/algorithms/src/tech/finnlestrange/BreadthFirstSearch.java)
- [Tree Traversal + Notes](https://github.com/71xn/algorithmsDataStructures/blob/f5651ad37a1dcb67b20359b6fb383eb86050a2f7/dataStructures/src/tech/finnlestrange/BinarySearchTree.java#L17)

## Notes

> The course content and lessons followed can be found [here](https://youtu.be/CBYHwZcbD-s).

### Definitions

**Data Structure** - `A named location that can be used to store and organize data`

**Algorithm** - `A collection of steps to solve a problem`

### Big O Notation

**BigO Notation** - `How code slows as data grows.`

1. Describes the performance of an algorithm as the amount of data increases
2. Machine independent (# of steps to completion)
3. Ignore smaller operations (O(n + 1) -> O(n)

_Examples:_

```
n = "ammount of data";

O(1) = performance stays the same as data increases

O(n) = performance scales linearly as with data size change e.g. Linear search (single for loop)

O(log n) = performace scales logarithmicly with data size change

O(n^2) = performance scales exponentially with power 2
```

\
O(n)

```java
class Example {
    public int addUp(int n) {
        int sum = 0;
        for (int i = 0; i<= n; i++) {
            sum += i;
        }
    }
}
```

\
O(1)

```java
class Example {
    public int addUp(int n) {
        int sum = n * (n + 1) / 2;
        return sum;
    }
}
```

\
**Examples for common Big O**

- `O(1)` = constant time
  - random access of array element
  - inserting at beginning of list
- `O(log n)` = logarithmic time
  - binary search
- `O(n)` = linear time
  - looping through array elements
  - single for loop
- `O(n log n )` = quasilinear time
  - quicksort
  - mergesort
  - heapsort
- `O(n^2)` = quadratic time
  - insertion sort
  - selection sort
  - bubblesort
- `O(!n)` = factorial time
  - TSP
