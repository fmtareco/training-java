##  Java Collections Framework


----

# MapInterface: differences between Map implementations

* differences  lie in ordering, performance, and null handling.

***HashMap***
--	
* Ordering: **None**. The order of elements can change over time.
* Performance: Fastest for general purpose ***O(1)*** for get and put.
* Nulls: **Allows one null key** and multiple null values.
* Best for: General storage where order doesn't matter.
	
***LinkedHashMap***
--
* Ordering: **Maintains insertion order** (or access order if configured). 
	* It uses a **doubly-linked** list running through its entries.
* Performance: Slightly slower than HashMap due to the overhead of maintaining the links, but still ***O(1)***.
* Nulls: Allows one null key and multiple null values.
* Best for: Caches (LRU) or when you need to iterate in the same order you added items.
	
***TreeMap***
--
* Ordering: Maintains **natural order** (alphabetical/numerical) or a custom Comparator. 
	* It is a Red-Black tree under the hood.
* Performance: Slower ***O(log n)*** because it must sort every insertion.
* Nulls: Does **not allow null keys** (it needs to compare them), but **allows null values**.
* Best for: When you need the keys to stay sorted at all times.
	
***Hashtable*** (Legacy)
--
* Ordering: **None**.
* Thread-Safety: Synchronized (Thread-safe). 
	* However, it is considered obsolete; 
	* for concurrency, ***ConcurrentHashMap*** is preferred.
* Nulls: Does **not allow null keys** or **null values**.
* Best for: Legacy code only.
	
Comparison Summary
--
| Feature | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Iteration Order | Random | Insertion Order | Sorted (Natural/Comparator) |
| Get/Put Time | O(1) | O(1) | O(log n) |
| Null Keys | Yes | Yes | No |
| Structure | Hash Table | Hash Table + Link | Red-Black Tree |

----

# QueueInterface: differences between implementations

* main differences depend on how they handle ordering, concurrency, and capacity. 

***PriorityQueue***
--
* Ordering: **Not FIFO**. 
	* It orders elements based on their natural priority (lowest value first) or a custom Comparator.
* Performance: ***$O(log n)*** for insertion and removal.
* Nulls: Does **not allow null elements**.
* Best for: Processing tasks based on importance rather than arrival time. 
	
***LinkedList*** (as a Queue)
--
* Ordering: **Standard FIFO** (First-In, First-Out).
* Performance: ***O(1)*** for basic operations (add/poll).
* Nulls: **Allows nulls** (though generally discouraged in queues).
* Best for: Simple, non-thread-safe FIFO operations where you also need List functionality.
	
***ArrayDeque***
--
* Ordering: **Standard FIFO**.
* Performance: Generally **faster than LinkedList** because it uses a resizable array instead of node objects (less overhead, better cache locality).
* Nulls: Does **not allow nulls**.
* Best for: High-performance FIFO queues or stacks where you don't need thread safety. 
	
Concurrent Queues (Blocking vs. Non-Blocking) 
--
* When working in multi-threaded environments, the choice shifts:
	* ***ArrayBlockingQueue***: 
		* Fixed capacity (bounded). 
		* Uses a single lock. 
		* Best for backpressure (the producer waits if the queue is full).
	* ***LinkedBlockingQueue***: 
		* Optionally bounded. 
		* Uses separate locks for put/take, providing better throughput than ArrayBlockingQueue.
	* ***ConcurrentLinkedQueue***: 
		* Unbounded and non-blocking (uses Wait-Free algorithms). 
		* Fastest for high-concurrency scenarios where you don't need to block threads. [17, 18, 19, 20, 21] 
		
Comparison Summary
--
| Feature | PriorityQueue | LinkedList | ArrayDeque | LinkedBlockingQueue |
|---|---|---|---|---|
| Ordering | Priority/Sorted | FIFO | FIFO | FIFO |
| Thread-Safe | No | No | No | Yes (Blocking) |
| Capacity | Unbounded | Unbounded | Unbounded | Bounded or Unbounded |
| Internal Store | Heap (Array) | Doubly Linked List | Circular Array | Linked Nodes |

----

# SetInterface: differences between implementations

* differences mirror those of the Map interface, as most sets are internally backed by their corresponding map versions. 
* The primary goal of a Set is to ensure uniqueness (no duplicate elements).

***HashSet***
--
* Ordering: **No guarantee**. 
	* The iteration order can change over time.
* Performance: 
	* The fastest for basic operations like add, remove, and contains, offering ***O(1)*** constant time.
* Nulls: 
	* Allows **one null element**.
* Best for: 
	* General-purpose use where you only care about uniqueness and speed.
	
***LinkedHashSet***
--
* Ordering: Maintains **insertion order**. 
	* If you iterate through the set, elements appear in the order they were added.
* Performance: 
	* Slightly slower than HashSet due to the overhead of maintaining a doubly-linked list, but still ***O(1)***.
* Nulls: Allows **one null element**.
* Best for: Scenarios where you need to preserve the order of elements (e.g., a "recently viewed" list).
	
***TreeSet***
--
* Ordering: Maintains **natural order** (alphabetical/numerical) or a custom Comparator.
* Performance: 
	* Slower than the others, with ***O(log n)*** time for basic operations because it uses a Red-Black Tree structure.
* Nulls: 
	* Does **not allow** null elements (it needs to compare elements to sort them).
* Best for: 
	* When you need the elements to be sorted at all times or need to perform range searches (e.g., headSet, tailSet).
		
***EnumSet***
--
* Ordering: Natural order of the Enum constants.
* Performance: Extremely fast (faster than HashSet). It uses a bit-vector (usually a single long) to represent the set.
* Nulls: Does not allow null elements.
* Best for: Working exclusively with Enum types. It is the most efficient set implementation for this specific use case.
	
Comparison Summary
--
| Feature | HashSet | LinkedHashSet | TreeSet | EnumSet |
|---|---|---|---|---|
| Iteration Order | Random | Insertion Order | Sorted | Enum Definition Order |
| Search/Add Time | O(1) | O(1) | O(log n) | O(1) |
| Allows Null | Yes | Yes | No | No |
| Internal Store | HashMap | LinkedHashMap | TreeMap | Bit-vector |

----
 
# ListInterface: differences between implementations 

* the List interface has two primary implementations used in **99% of cases**: 
	* ArrayList and 
	* LinkedList. 
* The choice between them usually comes down to how you plan to access and modify the data.

***ArrayList***
--
* Structure: Backed by a dynamically resizing array.
* Performance:
	* Get/Set: ***O(1)*** (Constant time). 
		* It’s incredibly fast for accessing elements by index because it calculates the memory address directly.
	* Add/Remove: ***O(n)*** (Linear time). 
		* If you remove an element from the middle, every subsequent element must be shifted one position to the left.
* Memory: Very efficient. 
	* It only stores the data itself in a contiguous block of memory.
* Best for: 
	* Frequent reading and occasional adding to the end of the list.

***LinkedList***
--
* Structure: Backed by a doubly-linked list (each element/node has a pointer to the next and the previous one).
* Performance:
	* Get/Set: ***O(n)*** (Linear time). 
		* To find the 50th element, it must start at the beginning and follow 50 pointers.
	* Add/Remove: ***O(1)*** (Constant time) 
		* if you are already at the position (like the head or tail).
		* No shifting is required; only the pointers change.
* Memory: Higher overhead.
	* Every single element requires extra memory to store the two pointers (next and prev).
* Best for: 
	* Frequent insertions and deletions at the beginning or end of the list (often used as a Queue or Deque).
		
***Vector*** (Legacy)
--
* Structure: Similar to ArrayList (array-based).
* Thread-Safety: Synchronized. 
	* Every method is thread-safe, which makes it much slower than ArrayList in single-threaded scenarios.
* Best for: Legacy code. 
	* For modern thread-safe lists, use **CopyOnWriteArrayList**.
		
***CopyOnWriteArrayList*** (Concurrent)
--
* Structure: 
	* Array-based, but **creates a fresh copy** of the underlying array every time it is modified (add/set/remove).
* Best for: 
	* Scenarios where you have many reads but very few writes, and you need thread safety without explicit synchronization.

Comparison Summary
--
| Feature | ArrayList | LinkedList | Vector |
|---|---|---|---|
| Data Structure | Resizable Array | Doubly-Linked List | Resizable Array |
| Random Access | O(1) | O(n) | O(1) |
| Insertion/Deletion | O(n) | O(1) | O(n) |
| Thread-Safe | No | No | Yes |
| Memory Efficiency | High | Low (Pointer overhead) | High |

Tip: 
* In modern hardware, ArrayList is almost always faster than LinkedList even for many insertions, 
	* because contiguous memory is much more "cache-friendly" for the CPU.

 
