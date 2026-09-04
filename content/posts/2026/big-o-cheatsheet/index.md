---
date: '2026-09-04T10:00:00+03:00'
draft: false
title: 'Big O Cheatsheet: Time and Space Complexity Explained'
tags: ["cheatsheet", "algorithms"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "The complexity classes, the rules for simplifying them, and what actually costs time and memory in a function."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

Big O describes how the cost of an algorithm grows as its input grows. Not how fast it is on your laptop — how it *scales*. An O(n²) function can beat an O(n log n) one on ten items and lose catastrophically on ten million, and the whole point of the notation is to make that crossover predictable without benchmarking anything.

## The Complexity Classes

From best to worst, these are the ones worth memorising:

| Notation | Name | Typical source |
| --- | --- | --- |
| O(1) | Constant | No loops — array index, hash lookup, arithmetic |
| O(log n) | Logarithmic | Halving the search space each step — binary search, balanced tree lookup |
| O(n) | Linear | One pass over n items |
| O(n log n) | Linearithmic | Comparison sorts — merge sort, heapsort, Timsort |
| O(n²) | Quadratic | Two nested loops over the same collection |
| O(2ⁿ) | Exponential | Naive recursion that branches twice per call |
| O(n!) | Factorial | Generating every permutation |

**O(1)** means the work does not depend on the input size at all. Looking up `users[500]` costs the same whether the array holds a thousand entries or a billion.

**O(log n)** shows up whenever each step discards a fixed fraction of the remaining work. Binary search is the canonical case, and it requires the input to be sorted already — sorting it first costs O(n log n), so a single binary search over an unsorted array is not a win.

**O(n)** is one loop through n items. It is also the floor for any problem where you genuinely have to look at every element: you cannot find the maximum of an unsorted array in less than O(n), because skipping any element risks skipping the answer.

**O(n log n)** is the practical ceiling for general-purpose sorting. Any sort that works by comparing pairs of elements cannot do better than this — it is a proven lower bound, not just an engineering limit. Sorts that beat it, like counting sort or radix sort, do so by not comparing elements at all, which means they only work on keys with restricted structure.

**O(n²)** is where naive nested loops land: comparing every element to every other element. Fine for a hundred items, painful at ten thousand, hopeless at a million.

**O(2ⁿ)** and **O(n!)** are the classes where the input size stops mattering because nothing is tractable. Naive recursive Fibonacci is O(2ⁿ) because each call spawns two more; brute-forcing the travelling salesman by trying every route is O(n!). If your algorithm is in either class, the fix is a different algorithm — memoisation, dynamic programming, or accepting an approximation.

## What Actually Costs Time

When you are counting operations in a function, these are the things that count:

- **Arithmetic** — `+`, `-`, `*`, `/`
- **Comparisons** — `<`, `>`, `==`
- **Loops** — `for`, `while`, and every form of iteration built on them
- **Function calls** — including the ones you did not write

That last one is the trap. A call to an outside function is not free, and it is not O(1) just because it is one line in your code. Sorting inside a loop, or a helper that quietly scans a list, is where an innocent-looking O(n) function turns into O(n²).

```python
def has_duplicates(items):
    for item in items:
        if items.count(item) > 1:   # count() is itself O(n)
            return True
    return False
```

This reads like one loop, so it looks linear. It is O(n²), because `count()` walks the whole list on every iteration. The fix is a set, which trades O(n) space for O(n) time.

## The Four Rules

Big O is a simplification, and there are four rules for doing the simplifying.

### Rule 1: Always assume the worst case

Big O describes the upper bound. If you are searching a list for a value and it happens to sit at index 0, that particular run was O(1) — but linear search is still O(n), because you have to assume the value is at the end or missing entirely.

```python
def find(items, target):
    for item in items:       # worst case: every element
        if item == target:
            return True
    return False
```

This is why "usually fast" is not a complexity claim. Complexity is about the case that hurts.

### Rule 2: Drop the constants

O(2n) is O(n). O(n/2) is O(n). A function with two sequential loops over the same array is still linear — doubling the work does not change how it scales, only the slope.

The corollary catches people out: **iterating over half a collection is still O(n)**. Halving the constant factor is a real speedup on real hardware, but it is invisible to Big O.

### Rule 3: Different inputs get different variables

If a function takes two collections, they need two variables. Calling them both "n" hides the actual behaviour:

```python
def compare(a, b):
    for x in a:          # O(a)
        pass
    for y in b:          # O(b)
        pass
    # total: O(a + b)

def cross(a, b):
    for x in a:
        for y in b:      # runs len(a) * len(b) times
            pass
    # total: O(a * b)
```

The shorthand is worth committing to memory:

- **`+` for steps in sequence** — one loop after another
- **`*` for steps nested** — one loop inside another

Two separate collections iterated one after the other is O(a + b). The same two collections nested are O(a × b). Only when both are the same collection does the second case collapse to the familiar O(n²).

### Rule 4: Drop non-dominant terms

O(n² + n) is O(n²). As n grows, the quadratic term swamps everything else, and the smaller terms stop mattering. Keep only the fastest-growing term.

The one caveat is Rule 3: O(a² + b) does **not** simplify to O(a²), because a and b are independent. You cannot drop a term unless you know it is dominated by another, and separate inputs give you no such guarantee.

## Space Complexity

The same notation applies to memory, and the same rules hold. What consumes space:

- **Variables** you declare
- **Data structures** you allocate
- **Function calls** — each one occupies a stack frame
- **Allocations** in general, including copies you did not mean to make

The function-call entry matters more than it looks. Recursion that goes n levels deep is O(n) space even when it allocates nothing, because n stack frames are live at once. This is exactly how a recursive traversal of a large linked list produces a stack overflow while the equivalent loop runs happily in O(1) space.

Time and space frequently trade against each other. The duplicate check above goes from O(n²) time and O(1) space to O(n) time and O(n) space by keeping a set of what it has already seen. Which trade is right depends on which resource you are short of.

## Beyond the Worst Case

Two refinements come up often enough to be worth knowing.

**Big O is only the upper bound.** Θ (theta) describes a tight bound — the growth rate both above and below — and Ω (omega) the lower bound. Most working conversations say "Big O" while actually meaning theta; that is harmless as long as everyone knows the difference exists.

**Amortised complexity** covers operations that are usually cheap but occasionally expensive. Appending to a dynamic array is O(1) most of the time and O(n) when the array has to grow and copy itself. Averaged over a long run of appends, the cost is O(1) amortised — a genuinely useful claim, and a different one from worst-case O(n).

## Common Operations at a Glance

Complexities for the structures you reach for daily. Hash table figures are average case; the worst case degrades to O(n) when every key collides.

| Operation | Array | Dynamic array | Linked list | Hash table | Balanced BST |
| --- | --- | --- | --- | --- | --- |
| Access by index | O(1) | O(1) | O(n) | — | — |
| Search by value | O(n) | O(n) | O(n) | O(1) | O(log n) |
| Insert at end | — | O(1) amortised | O(1) | O(1) | O(log n) |
| Insert at start | — | O(n) | O(1) | — | O(log n) |
| Delete by value | — | O(n) | O(n) | O(1) | O(log n) |

The pattern behind the table is the useful part: contiguous memory buys you cheap indexing and expensive insertion, pointers buy the reverse, and hashing buys cheap everything at the cost of ordering and worst-case guarantees. Picking a data structure is mostly picking which column you want.
