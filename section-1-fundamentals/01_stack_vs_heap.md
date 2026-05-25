# Module 01: Core Architecture Notes — Stack vs. Heap Mechanics

## 1. Stack Memory: The LIFO Execution Engine

The stack is a super-fast, lightweight, temporary memory zone managed in a strict LIFO (Last In, First Out) structure by the CPU.

### A. Lifecycle Mechanics

Variables created inside a function exist only while that function is executing. As soon as the function returns, its whole stack frame is popped off and instantly cleared.

### B. Structural Trace (calculate())

```python
def calculate():
    a = 10
    b = 20 
    c = a + b
calculate()
```

When this runs, the data blocks pile upward and clear from the top down:

```plaintext
|  c = 30  |  (Last in, First out)
+----------+
|  b = 20  |
+----------+
|  a = 10  |  (First in, Last out)
```

## 2. Heap Memory: The Dynamic Warehouse

The heap is a massive, flexible warehouse used to store heavy, unpredictable, or mutable data structures (like lists, images, arrays, and JSON objects).

### A. Reference Mechanics

```python
a = [1, 2, 3]
b = a
```

In this scenario, b is not a copy of the data. Instead, both a and b exist on the Stack as small 64-bit reference IDs. They both point to the exact same single array instance sitting out in the Heap warehouse.

### B. Function Closure & The Garbage Collector (GC)

```python
def create():
    arr = [1, 2, 3]
create()
```

When this function executes, the memory engine orchestrates this exact sequence:

Stack Frame Created: A temporary execution context is pushed onto the stack.

Heap Object Allocation: The mutable array [1, 2, 3] is written into the open heap warehouse space.

Reference Mapping: The local stack variable arr stores the address pointer ID of that heap object.

Stack Frame Destruction: The function ends. The local stack variable arr is completely wiped out instantly.

Unreachable State Triggered: The heap object [1, 2, 3] is left sitting in the warehouse, but it is now completely unreachable because its stack pointer is gone.

GC Reclamation: The Garbage Collector scans the heap, notices the reference count has hit zero, and frees up that memory slot.

## 3. Crash Conditions: Bad Code Architecture

### A. Stack Overflow (The Infinite Frame Trap)

The stack has a tiny, rigid memory limit. If your code runs an infinite execution loop without hit conditions, it will crash.

```python
def recurse():
    recurse()
recurse()
```

Execution Failure Pattern:

```plaintext
[  recurse() Frame  ] -> Pushes past hardware guard page! (CRASH: Stack Overflow)
[  recurse() Frame  ]
[  recurse() Frame  ]
[  recurse() Frame  ]
[  main() Start     ]
```

The CPU hits its maximum physical allocation limit, halts immediately, and throws a Stack Overflow error.

### B. Out Of Memory / OOM (The Memory Leak Trap)

The heap can grow until it consumes all of your system's RAM. If you keep objects alive for too long by holding active reference pointers, the Garbage Collector is blocked from cleaning them up.

```python
cache = []
while True:
    huge = [1] * 10000000
    cache.append(huge)
```

Execution Failure Pattern:

```plaintext
Memory Grid Consumption:
+------------------------------------------+
|  [Huge List Array Allocation Node 1]      | -> Active reference held by 'cache'
+------------------------------------------+
|  [Huge List Array Allocation Node 2]      | -> Active reference held by 'cache'
+------------------------------------------+
|  [Huge List Array Allocation Node 3]      | -> Active reference held by 'cache'
+------------------------------------------+
```

Because the cache list lives in a scope that never ends, the Reference Count for every huge list array remains stuck above 0. The Garbage Collector cannot reclaim the slots. The heap warehouse grows completely full until the operating system throws an Out Of Memory (OOM) error and kills your application process container.