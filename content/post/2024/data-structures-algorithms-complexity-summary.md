---
title: "Summary of Complexity for Various Data Structures and Algorithms"
description: "Summary of complexity for various data structures and algorithms, including selection sort, insertion sort, merge sort, quicksort, heapsort, tree sort, heap, binary search tree, and quickselect."
date: 2024-04-20T10:37:00+10:00
lastmod: 2024-09-04T20:55:00+10:00
categories:
  - Learning
tags:
  - Data Structures and Algorithms
---

# Sorting Algorithms

| Sorting Algorithm | Worst Time Complexity | Average Time Complexity | Best Time Complexity | Space Complexity | Stable | In-Place |
|-------------------|-----------------------|--------------------------|----------------------|------------------|--------|----------|
| Selection Sort    | n^2                   | n^2                      | n^2                  | 1                | No     | Yes      |
| Insertion Sort    | n^2                   | n^2                      | n                    | 1                | Yes    | Yes      |
| Merge Sort        | nlogn                 | nlogn                    | nlogn                | n                | Yes    | No       |
| Quick Sort        | n^2                   | nlogn                    | nlogn                | logn             | No     | Yes      |
| Heap Sort         | nlogn                 | nlogn                    | nlogn                | 1                | No     | Yes      |
| Tree Sort         | n^2                   | nlogn                    | nlogn                | 1                | No     | Yes      |

- The **logn** space complexity of quicksort comes from the recursion stack.
- Whether quicksort is stable or in-place depends on the partition function. Common implementations are in-place but unstable.
- Building a heap has a time complexity of **n**.
- For Tree Sort, building the tree takes **nlogn**, and traversal takes **n**. If the tree becomes highly unbalanced, it degenerates into a linked list, resulting in **n^2** time complexity.

# Data Structures

Time complexity of operations for common data structures.

## Heap

- Insert element: logn  
- Remove top element: logn  

Note: The main logic of insertion and removal is heapify. The time complexity is proportional to the tree height, hence **logn**. Building a heap is **n**, not **nlogn**.

## Binary Search Tree (BST)

- Fast lookup: logn, worst case n  
- Insert: logn, worst case n  
- Delete: logn, worst case n  
- Traversal: n  

Note: BST degenerates to a linked list in the worst case, hence **n**; when balanced, it's **logn**.

## Quickselect

Uses the core idea of quicksort to find the k-th element (k-th smallest or largest) in Θ(**n**) time complexity.
