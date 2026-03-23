# training-java

----

# Object Model 

equals/hashCode
equals() contract (reflexive, symmetric, transitive, consistent, non-null)
The relationship with hashCode()
Null-safety and type-safety
Best practices (immutability, Objects.equals, Objects.hash)
Performance considerations

	* best implementation of equals() follows the Java contract, 
		* compares only the fields that define logical identity, 
		* Prefer immutable fields when possible
		* ensures symmetry using getClass() (or carefully with instanceof), 
		* always consistent with hashCode().
		* Use the same fields in both methods
---
public final class Person {
    private final String name;
    private final int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
@Override
public boolean equals(Object o) {
    if (this == o) return true;                 // same reference
    if (o == null || getClass() != o.getClass()) return false;
    Person person = (Person) o;
    return age == person.age &&
           Objects.equals(name, person.name);
}
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
---
	* Equals contract:
		* Reflexive: x.equals(x) is true
		* Symmetric: x.equals(y) == y.equals(x)
		* Transitive
		* Consistent
		* x.equals(null) must return false

	* If possible Use a record (Java 16+)
public record Person(String name, int age) {}
		* Records automatically generate:
	equals()
	hashCode()
	toString()
	* Why must we override hashCode() whenever we override equals()?
		* general contract of hashCode() defined in Object:
		* two objects are equal according to equals(), they must have the same hash code.
		* Hash-based collections rely on this invariant:
	HashMap
	HashSet
	ConcurrentHashMap
	HashTable
		* These structures:
	Use hashCode() to determine the bucket
		* Use equals() to resolve collisions 
	* The Rule
		* If: a.equals(b) == true
		*  Then: a.hashCode() == b.hashCode()
		* The reverse is not required (collisions are allowed).


	*  Why Objects.equals(a, b) instead of a.equals(b)?
		* Because of null safety.
		* What Objects.equals() does internally
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
What is the difference between == and equals()?
--
	* 1. == (Relational Operator) 
	* 	Purpose: Compares references (memory addresses) for objects, or values for primitives (int, char, etc.).
	* 	Behavior: It returns true only if both variables point to the exact same location in memory.
	* Use case: 
	* 	Use it for primitive types or to check if two references are exactly the same instance. 
	* 2. equals() (Method)
	* 	Purpose: 
	* 		Compares the state or content of two objects.
	* 	Behavior: 
	* 		Its default implementation in the Object class behaves exactly like ==. 
	* 		However, most classes (like String, Integer, or ArrayList) override it to compare actual data.
	* Use case: 
	* 	Use it whenever you want to know if two objects represent the same "thing" 

Why must you override hashCode() when overriding equals()?
--
	* if don't override hashCode(), 
	* 	break the Hash Contract, 
	* 	causes hash-based collections (like HashMap, HashSet, and Hashtable) to fail.
	* The Contract
	* 	If obj1.equals(obj2) is true, then obj1.hashCode() must be equal to obj2.hashCode().
	* What happens if you skip it?
	* 	User class where equals() compares the userId. 
	* 	don't override hashCode(), the JVM uses the default implementation (based on memory address).
	* Storage: 
	* 	put a user with ID 123 into a HashMap. 
	* 	map calculates a hash based on the object's memory address and stores it in a specific "bucket." 
	* Retrieval: 
	* 	You create a new instance of a user with the same ID 123 to look it up.
	* The Failure: 
	* 	new object, it has a different memory address, resulting in a different hashCode. 
	* 	HashMap looks in the wrong bucket and returns null. [10, 11, 15]
	* 	Even though equals() would say the objects are the same, the collection can't even find the object to compare it.

What is the equals/hashCode contract?
--
	* Igualdade Implica Hash Igual: 
	* 	Se x.equals(y) é verdadeiro, então x.hashCode() deve ser igual a y.hashCode().
	* Consistência: 
	* 	hashCode deve retornar o mesmo valor múltiplas vezes na mesma execução, 
	* 	desde que os campos usados no equals não sejam alterados.
	* Hash Igual NÃO Implica Igualdade: 
	* 	Dois objetos diferentes podem ter o mesmo hashCode (conhecido como colisão de hash). 
	* 	Embora permitido, um bom design deve tentar minimizar isso para melhor performance.
	* Se Sobrescrever Um, Sobrescreva Ambos: 
	* 	Sempre que alterar a lógica de equals(), deve também sobrescrever hashCode() para manter o contrato. 
	* 	Caso contrário, objetos "iguais" podem acabar em "baldes" (buckets) diferentes numa HashMap, 
	* 	tornando impossível encontrá-los. 

What is immutability? How do you design an immutable class?
--
	* Immutability means an object's state cannot be modified after created. 
	* Immutable classes enhance thread safety, security, and caching reliability. 
	* design principles include 
		* declaring the class final, 
		* all fields private and final, 
		* avoiding setters, 
		* Initialize via Constructor
		* Defensive Copies for Mutable Fields: 
		* 	If contains mutable objects (e.g., Date, List), perform deep copies to prevent external modification of internal state. 
    public List<String> getMutableList() {
        // Defensive copy for getter
        return new ArrayList<>(mutableList);
    }

Why are String, Integer, etc. immutable?
--
Benefits:
	* thread safety
	* hash caching
	* security
	* string pool optimization
Example:
1.	String pool reuse

What are value objects?
What are records and when should you use them?

---- 

# OOP

What is the difference between composition and inheritance?
--
	* Inheritance focuses on what an object is, while Composition focuses on what an object has.
	* 1. Inheritance ("IS-A" Relationship)
	* 	Inheritance allows a subclass to derive properties and behaviors from a superclass.
	* 	Coupling: High. 
	* 		Changes superclass automatically ripple down to all subclasses (the "fragile base class" problem).
	* 	Flexibility: Low. 
	* 		relationship is fixed at compile-time; 
	* 		a class cannot change its parent during runtime.
	* 	Example: A Car is a Vehicle.
	* 2. Composition ("HAS-A" Relationship)
	* 	Composition involves wrapping one or more objects within another class to achieve new functionality.
	* 	Coupling: Low. 
	* 		host class only interacts with the internal object through its public interface.
	* 	Flexibility: High. 
	* 		can swap the internal object at runtime (e.g., changing a PetrolEngine to an ElectricEngine).
	* 	Example: A Car has an Engine.
	* Comparison Summary
	* Relationship	
	* 	IS-A (e.g., Sparrow is a Bird)	
	* 	HAS-A (e.g., Bird has Wings)
	* Visibility	
	* 	White-box (subclass sees internals)	
	* 	Black-box (only sees public API)
	* Binding	
	* 	Static (Compile-time)	
	* 	Dynamic (Runtime)
	* Design Rule	
	* 	Use for strict taxonomies	
	* 	"Favor composition over inheritance"
	* Why "Favor Composition over Inheritance"?
	* 	core principle from the Design Patterns (GoF) book. 
	* 	Inheritance often leads to s
	* 		deep, rigid hierarchies that 
	* 		are hard to maintain. 
	* 	Composition allows you to build complex behavior by 
	* 		combining small, focused objects, 
	* 		making your code more modular and 
	* 		much easier to unit test.

	* What is polymorphism in Java?
	* What is method overloading vs overriding?
 
Collections Framework
HashMap internal working
	* HashMap in Java 
	* operates on the principle of hashing, 
	* using an array of Node objects (buckets) to 
	* store key-value pairs with average time complexity. 
	* calculates a hash code to determine the bucket index, 
	* using linked lists or balanced trees (Java 8+) for collision management. 
	* Key components include put(), get(), hashing, and resizing. 
	* Key Internal Components and Mechanisms
		* Node Array (Buckets): 
		* foundation is an array (default capacity 16) 
	each position is a "bucket" that can hold a node or a linked list 
		* Node Structure: 
		* 	node contains the 
	* int hash, K key, V value, and a Node<K,V> next reference.
		* Hash Calculation: 
	hashCode() method converts keys into integer values, 
	which are then processed to calculate index = hash(key) & (n-1).
		* Collision Handling: 
	two different keys produce the same index, they are stored in a linked list. 
	bucket's linked list exceeds a threshold (TREEIFY_THRESHOLD = 8), 
	is converted into red-black tree improve performance from O(n) to O(log n)
		* Resizing (Rehashing): 
	When the number elements exceeds the capacity * load factor (default 0.75), 
	array size is doubled, 
	all existing elements are rehashed and moved to new, larger buckets. 
		* Key Methods
		* put(K key, V value):
	Calculates hash and determines the bucket index.
	If the bucket is empty, the node is inserted.
	If a collision occurs, 
		traverses the list/tree to check for an existing key (using equals()):
	If the key exists, the value is updated.
	If not, the new node is appended to the list/tree.
		* get(K key):
	Calculates the hash to find the bucket index.
	Traverses the linked list/tree within that bucket, using equals() to compare keys, and returns the corresponding value. 
		* computeIfAbsent(key, mappingFunction): 
	Se a chave não existir, calcula o valor e insere-o no mapa; caso contrário, retorna o valor atual. Ideal para inicializar coleções (ex: Map<String, List>).
		* computeIfPresent(key, remappingFunction): 
	Se a chave existir, calcula um novo valor baseado no antigo e atualiza-o; se a função retornar null, a chave é removida.
		* compute(key, remappingFunction): 
	Atualiza ou insere um valor independentemente de a chave existir ou não, permitindo lógica personalizada para ambos os casos num único passo.
		* merge(key, value, remappingFunction): 
	Combina um novo valor com o existente (se houver) usando uma função de concatenação ou soma. Excelente para contadores ou acumular strings.
		* putIfAbsent(key, value): 
	Insere o par chave-valor apenas se a chave ainda não estiver presente ou for null. Diferente do computeIfAbsent, recebe o valor já instanciado.
		* getOrDefault(key, defaultValue): 
	Retorna o valor associado à chave ou um valor padrão caso a chave não exista, evitando verificações manuais de null.
		* forEach(action): 
	Permite iterar sobre todos os pares chave-valor usando uma expressão lambda, tornando o código mais limpo que um loop for tradicional.
		* replaceAll(function): 
	Substitui o valor de todas as entradas do mapa com base num cálculo aplicado a cada par chave-valor atual.
Queres ver como usar o merge para criar um contador de frequências em apenas uma linha?

	
	* Crucial Considerations
		* Null Keys: 
		* 	HashMap allows one null key, which is always stored at index 0.
		* Immutable Keys: 
	highly recommended to use immutable objects (like String or Integer) as keys to prevent hash changes, 
	which would make the key unretrievable.
		* Performance: 
	Average time complexity for put and get is O(1)

ArrayList vs LinkedList
	* ArrayList 
	* 	uses a dynamic array for fast O(1) random access but 
	* 	slow O(n)insertion/deletion due to element shifting. 
	* LinkedList 
	* 	uses a doubly linked list, offering fast O(1)
	* 	insertions/deletions but slow O(n) traversal. 
	* ArrayList for reading data, and 
	* LinkedList for frequent modifications. 
	* Key Differences:
	* 	Data Structure: 
	* 		ArrayList is backed by a resizable array, while 
	* 		LinkedList is backed by a doubly linked list.
	* 	Access Time: 
	* 		ArrayList provides O(1) time complexity for index-based access,
	* 		whereas LinkedList requires O(n) time.
	* 	Insertion/Deletion: 
	* 		LinkedList is more efficient for adding or removing elements (O(1)) in the middle or at the beginning, 
	* 		while ArrayList takes O(n) because it requires shifting elements.
	* 	Memory Overhead: 
	* 		LinkedList consumes more memory than ArrayList because it stores pointers for the next and previous nodes.
	* 	Use Case: 
	* 		ArrayList is ideal for read-heavy applications, and 
	* 		LinkedList is better for scenarios with frequent insertions and deletions. 
	* Conclusion:
	* 	ArrayList for the majority of scenarios due to better cache performance and lower overhead, 
	* 	switching to LinkedList only if you are frequently adding/removing elements from the middle of large lists


HashMap vs ConcurrentHashMap
	* HashMap is designed for single-threaded speed, while 
	* ConcurrentHashMap is engineered for high performance in multi-threaded environments
	* Thread Safety	
	* 	❌ Not thread-safe	
	* 	✅ Thread-safe
	* Null Keys/Values	
	* 	✅ Allows 1 null key and multiple null values	
	* 	❌ Does not allow null keys or values
	* Iterator Type	
	* 	Fail-fast: 
	* 		Throws ConcurrentModificationException if modified during iteration	
	* 	Fail-safe (Weakly consistent): 
	* 		Does not throw exceptions during concurrent modification
	* Performance	
	* 	Faster in single-threaded scenarios (no overhead)	
	* 	Optimized for multi-threaded scenarios; better throughput than synchronized maps
	* Locking Mechanism	
	* 	No internal locking	
	* 	Fine-grained locking: Locks only specific buckets (nodes) rather than the entire map
	* Key Technical Differences
	* 	Concurrency Handling: 
	* 		In a multi-threaded environment, 
	* 		HashMap can suffer from data corruption or infinite loops during resizing. 
	* 		ConcurrentHashMap avoids this by using Compare-And-Swap (CAS) and 
	* 			individual bucket locks to allow multiple threads to read and write simultaneously without a global lock.
	* 	Null Safety Strategy: 
	* 		ConcurrentHashMap disallows nulls to avoid ambiguity. 
	* 		In concurrent code, get(key) returning null could mean the key is missing or the value is truly null; 
	* 		checking containsKey() after a get() could result in a race condition if another thread modifies the map in between.
	* 	Iteration Behavior: 
	* 		HashMap iterators immediately fail if the structure is modified. 
	* 		ConcurrentHashMap iterators provide a "weakly consistent" view, 
	* 		meaning they reflect the state of the map at the time of iterator creation and 
	* 		may (but are not guaranteed to) show subsequent updates. 
	* 
	* When to Use Which?
	* 	HashMap for local variables or single-threaded applications 
	* 		where performance is critical and synchronization is unnecessary.
	* 	ConcurrentHashMap 
	* 	for shared caches or data structures accessed by multiple threads to 
	* 		ensure data integrity without heavy performance cost of synchronized map. 

What happens when HashMap resizes?
Capacity doubles.
Process:
newCapacity = oldCapacity * 2
rehash entries
Rehashing changes bucket positions.

Difference between HashMap and TreeMap
		* 1. Main Differences
		* HashMap	
		* 	Hash Table	
		* 	No order guaranteed	
		* 	Search Time	O(1) (Constant time)	
		* 	Insertion Time	O(1) (Amortized)	
		* 	Allows one null key	
		* 	Memory Usage	Higher (due to bucket overhead)	
		* TreeMap
		* 	Red-Black Tree (Self-balancing)
		* 	Natural Order (or custom Comparator)
		* 	Search Time	O(log n) (Logarithmic time)
		* 	Insertion Time	O(log n)
		* 	No null keys allowed (throws NullPointerException)
		* 	Memory Usage	Lower (but higher per-node overhead)
		* 2. When to use which?
		* 	HashMap 
		* 		for almost every standard scenario. 
		* 		significantly faster for basic operations like get(), put(), and remove() 
		* 		because it calculates a hash directly rather than traversing a tree.
		* 		If you want a HashMap that maintains insertion order (instead of sorted order), use LinkedHashMap. 
		* 		It gives you  speed with predictable iteration order!
		*  speed with predictable iteration order!
		* 	TreeMap 
		* 		only when you need the keys to be sorted. For example:
		* 		Iterating through keys in alphabetical or numerical order.
		* 		Finding the "closest" match (e.g., ceilingKey() or floorKey()).
		* 		Performing range queries (e.g., subMap(from, to)).
How does HashMap work internally?
What happens on hash collision?

What is the time complexity of common collections?
		* Use ArrayList as your default List unless you are frequently inserting/removing from the beginning.
		* Use HashMap for the fastest possible lookups 
		* Use TreeMap only if you need the keys to be sorted or need to perform range queries (e.g., subMap).
		* Use ArrayDeque instead of Stack for LIFO operations; it's faster and more modern.


What changed in HashMap after Java 8?
		* 1. The Main Change: "Treeification"
		* Before Java 8, when multiple keys had the same hash (a collision), they were stored in a simple LinkedList. In the worst-case scenario (many collisions), searching a bucket took  O(n) time.
		* Java 8+ Logic: If a bucket reaches a specific threshold (TREEIFY_THRESHOLD = 8), the HashMap converts that LinkedList into a Red-Black Tree.
		* Performance Gain: This reduces the worst-case search time from O(n) to O(log n)

How does ConcurrentHashMap avoid locking the whole map?

Fail-fast vs fail-safe iterators
		* difference  two iterator behaviors how they handle modifications to the collection an iteration is in progress.
		* 1. Fail-Fast Iterators
		* 	These iterators operate directly on the collection's internal data. 
		* 	If any modification occurs while iterating,
		* 	immediately throws a ConcurrentModificationException.
		* 	* How it works: It uses an internal flag called modCount. If the count changes during iteration, it "fails fast."
		* 	* Behavior: It does not guarantee 100% safety, but it tries its best to detect bugs early.
		* 	* Examples: ArrayList, HashMap, HashSet, LinkedList.
		* 	* Note: You can only remove elements safely using the iterator.remove() method, not list.remove().
		* 
		* 2. Fail-Safe (Safe-Copy) Iterators
		* 	These iterators work on a clone or a specific view of the collection, 
		* 	allowing modifications without crashing the program.
		* 	* How it works: It creates a "snapshot" of the data at the moment the iterator is created. Any changes made to the original collection are not reflected in the current iteration.
		* 	* Behavior: It never throws ConcurrentModificationException.
		* 	* Examples: CopyOnWriteArrayList, ConcurrentHashMap (key/entry sets).
		* 	* Trade-off: Creating a copy (like in CopyOnWriteArrayList) consumes more memory and is slower for frequent writes.
		* 
		* 1. Fail-Fast (ArrayList)
		* 	iterator checks the modCount (modification count) on every step.
		* 	Since list.remove() changes this count, the iterator detects a mismatch and crashes.
		* 
		* List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
		* try {
		*     for (String item : list) {
		*         if (item.equals("B")) {
		*             list.remove(item); // CRASHES HERE
		*         }
		*     }
		* } catch (ConcurrentModificationException e) {
		*     System.out.println("ArrayList failed: ConcurrentModificationException thrown!");
		* }
		* 
		* 2. Fail-Safe (CopyOnWriteArrayList)
		* 	CopyOnWriteArrayList creates a snapshot (a copy) of the internal array when the iterator is started. 
		* 	Modifications happen on a new copy, so the iterator continues safely on the old one.
		* 
		* List<String> safeList = new CopyOnWriteArrayList<>(Arrays.asList("A", "B", "C"));
		* for (String item : safeList) {
		*     if (item.equals("B")) {
		*         safeList.remove(item); // WORKS PERFECTLY
		*     }
		* }
		* 
		* 
		* 1. Using Iterator.remove()
		* 	classic way. 
		* 	When you use the iterator's own remove() method, 
		* 	it updates the internal modCount correctly, 
		* 	so the iterator stays in sync with the list.
		* 
		* List<String> list = new ArrayList<>(Arrays.asList("Apple", "Banana", "Cherry"));
		* Iterator<String> it = list.iterator();
		* 
		* while (it.hasNext()) {
		*     String fruit = it.next();
		*     if (fruit.equals("Banana")) {
		*         it.remove(); // Safely removes "Banana"
		*     }
		* }
		* 
		* 2. Using removeIf() (Java 8+)
		* 	modern, functional approach. 
		* 	internally optimized and much more concise. 
		* 	uses a Predicate to define what should be deleted.
		* java
		* List<String> list = new ArrayList<>(Arrays.asList("Apple", "Banana", "Cherry"));
		* 
		* // Removes any element that equals "Banana"
		* list.removeIf(fruit -> fruit.equals("Banana"));
 
MapInterface: differences between Map implementations
 lie in ordering, performance, and null handling.
Here is the breakdown:
1. HashMap
	* Ordering: None. The order of elements can change over time.
	* Performance: Fastest for general purpose ($O(1)$ for get and put).
	* Nulls: Allows one null key and multiple null values.
	* Best for: General storage where order doesn't matter.
2. LinkedHashMap
	* Ordering: Maintains insertion order (or access order if configured). It uses a doubly-linked list running through its entries.
	* Performance: Slightly slower than HashMap due to the overhead of maintaining the links, but still $O(1)$.
	* Nulls: Allows one null key and multiple null values.
	* Best for: Caches (LRU) or when you need to iterate in the same order you added items.
3. TreeMap
	* Ordering: Maintains natural order (alphabetical/numerical) or a custom Comparator. It is a Red-Black tree under the hood.
	* Performance: Slower ($O(\log n)$) because it must sort every insertion.
	* Nulls: Does not allow null keys (it needs to compare them), but allows null values.
	* Best for: When you need the keys to stay sorted at all times.
4. Hashtable (Legacy)
	* Ordering: None.
	* Thread-Safety: Synchronized (Thread-safe). However, it is considered obsolete; for concurrency, ConcurrentHashMap is preferred.
	* Nulls: Does not allow null keys or null values.
	* Best for: Legacy code only.
Comparison Summary
Feature	HashMap	LinkedHashMap	TreeMap
Iteration Order	Random	Insertion Order	Sorted (Natural/Comparator)
Get/Put Time	O(1)	O(1)$	O(\log n)
Null Keys	Yes	Yes	No
Structure	Hash Table	Hash Table + Link	Red-Black Tree

QueueInterface: differences between implementations
main differences depend on how they handle ordering, concurrency, and capacity. 
1. PriorityQueue
	* Ordering: Not FIFO. It orders elements based on their natural priority (lowest value first) or a custom Comparator.
	* Performance: $O(\log n)$ for insertion and removal.
	* Nulls: Does not allow null elements.
	* Best for: Processing tasks based on importance rather than arrival time. [2, 3, 4, 5, 6] 
2. LinkedList (as a Queue)
	* Ordering: Standard FIFO (First-In, First-Out).
	* Performance: $O(1)$ for basic operations (add/poll).
	* Nulls: Allows nulls (though generally discouraged in queues).
	* Best for: Simple, non-thread-safe FIFO operations where you also need List functionality. [7, 8, 9, 10, 11] 
3. ArrayDeque
	* Ordering: Standard FIFO.
	* Performance: Generally faster than LinkedList because it uses a resizable array instead of node objects (less overhead, better cache locality).
	* Nulls: Does not allow nulls.
	* Best for: High-performance FIFO queues or stacks where you don't need thread safety. [12, 13, 14, 15] 
4. Concurrent Queues (Blocking vs. Non-Blocking) 
When working in multi-threaded environments, the choice shifts:
	* ArrayBlockingQueue: Fixed capacity (bounded). Uses a single lock. Best for backpressure (the producer waits if the queue is full).
	* LinkedBlockingQueue: Optionally bounded. Uses separate locks for put/take, providing better throughput than ArrayBlockingQueue.
	* ConcurrentLinkedQueue: Unbounded and non-blocking (uses Wait-Free algorithms). Fastest for high-concurrency scenarios where you don't need to block threads. [17, 18, 19, 20, 21] 
Comparison Summary
Feature [22, 23, 24, 25, 26] 		PriorityQueue	LinkedList	ArrayDeque	LinkedBlockingQueue
Ordering		Priority/Sorted	FIF		* FIF		* FIFO
Thread-Safe		N		* N		* N		* Yes (Blocking)
Capacity		Unbounded	Unbounded	Unbounded	Bounded or Unbounded
Internal Store		Heap (Array)	Doubly Linked List	Circular Array	Linked Nodes


SetInterface: differences between implementations
differences mirror those of the Map interface, 
	as most sets are internally backed by their corresponding map versions. 
The primary goal of a Set is to ensure uniqueness (no duplicate elements).
1. HashSet
	* Ordering: No guarantee. The iteration order can change over time.
	* Performance: The fastest for basic operations like add, remove, and contains, offering $O(1)$ constant time.
	* Nulls: Allows one null element.
	* Best for: General-purpose use where you only care about uniqueness and speed.
2. LinkedHashSet
	* Ordering: Maintains insertion order. If you iterate through the set, elements appear in the order they were added.
	* Performance: Slightly slower than HashSet due to the overhead of maintaining a doubly-linked list, but still $O(1)$.
	* Nulls: Allows one null element.
	* Best for: Scenarios where you need to preserve the order of elements (e.g., a "recently viewed" list).
3. TreeSet
	* Ordering: Maintains natural order (alphabetical/numerical) or a custom Comparator.
	* Performance: Slower than the others, with $O(\log n)$ time for basic operations because it uses a Red-Black Tree structure.
	* Nulls: Does not allow null elements (it needs to compare elements to sort them).
	* Best for: When you need the elements to be sorted at all times or need to perform range searches (e.g., headSet, tailSet).
4. EnumSet
	* Ordering: Natural order of the Enum constants.
	* Performance: Extremely fast (faster than HashSet). It uses a bit-vector (usually a single long) to represent the set.
	* Nulls: Does not allow null elements.
	* Best for: Working exclusively with Enum types. It is the most efficient set implementation for this specific use case.
Comparison Summary
Feature	HashSet	LinkedHashSet	TreeSet	EnumSet
Iteration Order	Random	Insertion Order	Sorted	Enum Definition Order
Search/Add Time	O(1)	O(1)	O(log n)	O(1)
Allows Null	Yes	Yes	N		* No
Internal Store	HashMap	LinkedHashMap	TreeMap	Bit-vector

 
ListInterface: differences between implementations 
In Java, the List interface has two primary implementations used in 99% of cases: ArrayList and LinkedList. 
The choice between them usually comes down to how you plan to access and modify the data.
1. ArrayList
	* Structure: Backed by a dynamically resizing array.
	* Performance:
		* Get/Set: $O(1)$ (Constant time). It’s incredibly fast for accessing elements by index because it calculates the memory address directly.
		* Add/Remove: $O(n)$ (Linear time). If you remove an element from the middle, every subsequent element must be shifted one position to the left.
	* Memory: Very efficient. It only stores the data itself in a contiguous block of memory.
	* Best for: Frequent reading and occasional adding to the end of the list.
2. LinkedList
	* Structure: Backed by a doubly-linked list (each element/node has a pointer to the next and the previous one).
	* Performance:
		* Get/Set: $O(n)$ (Linear time). To find the 50th element, it must start at the beginning and follow 50 pointers.
		* Add/Remove: $O(1)$ (Constant time) if you are already at the position (like the head or tail). No shifting is required; only the pointers change.
	* Memory: Higher overhead. Every single element requires extra memory to store the two pointers (next and prev).
	* Best for: Frequent insertions and deletions at the beginning or end of the list (often used as a Queue or Deque).
3. Vector (Legacy)
	* Structure: Similar to ArrayList (array-based).
	* Thread-Safety: Synchronized. Every method is thread-safe, which makes it much slower than ArrayList in single-threaded scenarios.
	* Best for: Legacy code. For modern thread-safe lists, use CopyOnWriteArrayList.
4. CopyOnWriteArrayList (Concurrent)
	* Structure: Array-based, but creates a fresh copy of the underlying array every time it is modified (add/set/remove).
	* Best for: Scenarios where you have many reads but very few writes, and you need thread safety without explicit synchronization.

Comparison Summary
Feature	ArrayList	LinkedList	Vector
Data Structure	Resizable Array	Doubly-Linked List	Resizable Array
Random Access	O(1)	O(n)	$O(1)$
Insertion/Deletion	O(n)	O(1)	$O(n)$
Thread-Safe	N		* N		* Yes
Memory Efficiency	High	Low (Pointer overhead)	High
Pro-tip: In modern hardware, ArrayList is almost always faster than LinkedList even for many insertions, because contiguous memory is much more "cache-friendly" for the CPU.

 
Memory Model & JVM
JVM Internals: From Basics to Advanced Explained
https://www.linkedin.com/pulse/jvm-internals-from-basics-advanced-explained-akshay-kumar-iihje/
Heap vs Stack
Garbage collection basics
What is the difference between StackOverflowError and OutOfMemoryError?
What is the Java Memory Model (JMM)?
What is garbage collection? How does it work?
What are different GC algorithms? (G1, CMS, ZGC)
What causes memory leaks in Java?
		* Common Causes of Memory Leaks
		* 	Static Fields and Collections: 
		* 		Static variables live for the entire lifetime of the JVM. Storing large objects or growing collections (like a HashMap or ArrayList) in static fields without clearing them is the most common cause of leaks.
		* 	Unclosed Resources: 
		* 		Failing to close database connections, network sockets, or file streams leaves underlying OS resources and their associated Java objects in memory. Using try-with-resources is the standard fix for this.
		* 	ThreadLocal Misuse: 
		* 		In managed environments like Spring Boot, threads are often reused in a pool. If you set a value in a ThreadLocal but don't call .remove(), the object remains tied to that thread forever, even after the request is finished.
		* 	Inner Class References: 
		* 		Non-static inner classes (and anonymous classes) hold an implicit reference to their outer class instance. 
		* 		If the inner class object lives a long time (e.g., in a long-running thread), 
		* 		it prevents the entire outer class from being garbage collected.
		* 	Forgotten Listeners & Callbacks: 
		* 		Registering an object as a listener (e.g., to an event bus or GUI component) creates a reference. 
		* 		If you don't explicitly unregister the listener when it's no longer needed, the event source will keep it alive.
		* 	Improper equals() and hashCode(): 
		* 		If you add custom objects to a HashSet or HashMap but fail to override these methods correctly, 
		* 		the collection may fail to identify duplicates and will grow indefinitely as "new" unique entries. 
		* Detection & Diagnosis
		* 	Monitor the "Live Set": 
		* 		Use tools like JConsole or VisualVM to watch if heap usage increases steadily after multiple Full GC cycles.
		* 	Heap Dump Analysis: 
		* 		Capture a snapshot of memory using jmap or jcmd and analyze it with the Eclipse Memory Analyzer (MAT). Look for:
		* 	Dominator Tree: 
		* 		Shows which objects are holding the most memory.
		* 	Path to GC Roots: 
		* 		Traces why a specific object is still reachable.
		* 	Profiling: 
		* 		Use the NetBeans Profiler or JProfiler to track object allocation patterns in real-time.
What are strong, weak, soft, and phantom references?
		* 1. Strong Reference
		* 	The default type (e.g., Object obj = new Object();).
		* 	Behavior: As long as a strong reference exists, the object cannot be collected by the GC.
		* 	Result: Can lead to OutOfMemoryError if references are held longer than necessary.
		* 2. Soft Reference (SoftReference<T>)
		* 	Used for memory-sensitive caches.
		* 	Behavior: The GC will only collect these objects if the JVM critically needs memory.
		* 	Use Case: Ideal for caching large objects (like images) that you want to keep as long as memory allows, 
		* 	but are okay with losing if RAM is full.
		* 3. Weak Reference (WeakReference<T>)
		* 	Used for mapping metadata to objects without preventing their collection.
		* 	Behavior: The GC collects these objects immediately during the next cycle if no strong references exist.
		* 	Use Case: WeakHashMap. It allows keys to be collected when they are no longer used 
		* 	elsewhere in the application, preventing memory leaks.
		* 4. Phantom Reference (PhantomReference<T>)
		* 	The weakest type; it doesn't even allow you to retrieve the object (calling .get() always returns null).
		* 	Behavior: Used to track when an object has been removed from memory. 
		* 	The reference is put into a ReferenceQueue after the object is finalized but before its memory is reclaimed.
	Use Case: Performing low-level cleanup actions (like closing native resources) more reliably than the deprecated finalize() method
What are the different memory areas in the JVM?
The JVM divides memory into several runtime data areas:
	* Heap — 
	* 	Stores objects and class instances.
	* Stack — 
	* 	Stores method call frames, local variables, and references.
	* Method Area (MetaSpace in Java 8+) — 
	* 	Stores class metadata, static variables, and method bytecode.
	* Program Counter (PC) Register — 
	* 	Holds the address of the currently executing instruction.
	* Native Method Stack — 
	* 	Manages native (non-Java) method calls.

4.2. What is the difference between Heap and Stack memory in Java?
 

4.3. What is the Java Heap and how is it structured?
The Java Heap is divided into three generations for efficient memory management:
	* Young Generation (Eden + Survivor Spaces) — 
	* 	Stores newly created objects; frequent garbage collection.
	* Old (Tenured) Generation — 
	* 	Stores long-lived objects promoted from the Young Generation.
	* Permanent Generation (removed in Java 8, replaced by MetaSpace) — 
	* 	Used for class metadata storage.

4.4. What happens when an object is created in Java?
1.	The new keyword is used to allocate memory for the object in the Heap.
2.	JVM initializes the object and assigns a memory address.
3.	The reference to the object is stored in the Stack.
4.	When there are no active references, the object becomes eligible for Garbage Collection.
4.5. How does Garbage Collection work in JVM?
Garbage Collection (GC) automatically deallocates memory by removing unreachable objects. 
JVM uses different GC algorithms:
	* Mark and Sweep — 
	* 	Identifies and removes unused objects.
	* Generational GC — 
	* 	Divides objects into Young and Old generations for efficient collection.
	* Reference Counting — 
	* 	Keeps track of references; discarded if count reaches zero (not used in modern JVMs).

4.6. What are common OutOfMemoryError scenarios in JVM?
Java.lang.OutOfMemoryError: Heap space — 
	Insufficient Heap memory; increase -Xmx value.
	* Java.lang.OutOfMemoryError: Metaspace — 
	* 	Too many classes loaded; adjust -XX:MaxMetaspaceSize.
	* Java.lang.StackOverflowError — 
	* 	Deep recursion or infinite loop in methods.

4.7. How can you monitor and optimize JVM memory usage?
	* Use JVisualVM, JConsole, or Java Mission Control for monitoring.
	* Tune JVM parameters (-Xms, -Xmx, -XX:NewRatio, etc.).
	* Optimize object creation and use StringBuilder instead of String concatenation.
	* Avoid memory leaks by properly handling static references, collections, and event listeners.

Java: Heap Down and Thread Down
	* "Heap" and "Thread" coupled with "Down" usually refer to two critical diagnostic actions: 
	* 	Dumping (exporting state) and 
	* 	Downsizing (reducing resource usage).
	* 1. Heap Down (Heap Dump / Downsizing)
	* 	Heap Dump (Snapshot): 
	* 		file containing a point-in-time "photograph" of all objects in your application's memory.
	* 		The Process: 
	* 			trigger a command (like jmap in Java) that freezes the memory state and writes it to a file.
	* 		The Goal: 
	* 			find Memory Leaks.
	* 			can see which objects are taking up the most space and 
	* 			why they aren't being cleaned up by the Garbage Collector.
	* 	Heap Downsizing: 
	* 		when the system shrinks the allocated memory pool.
	* 		The Process: 
	* 			After a Garbage Collection cycle, 
	* 			if application is using significantly less memory than the maximum limit, 
	* 			JVM or Database engine may return unused memory back to the Operating System to save resources.
	* 2. Thread Down (Thread Dump / Down-scaling)
	* 	Thread Dump (Execution Snapshot): 
	* 		text-based report showing exactly what every active thread in the system is doing at that exact millisecond.
	* 	The Process: 
	* 		lists the Stack Trace for every thread. You can see which threads are "Running," "Waiting," or "Blocked."
	* 	The Goal: 
	* 		Crucial for diagnosing Deadlocks (where two threads are stuck waiting for each other) or 
	* 		high CPU usage. If your app is "frozen," a thread dump tells you where it is stuck.
	* 	Thread Down-scaling (Pool Shrinking): 
	* 		This refers to reducing the number of active threads.
	* 	The Process: 
	* 		In a Thread Pool, if threads remain idle for too long (a "keep-alive" timeout), 
	* 		the system kills those threads to free up system overhead.
	* Comparison Summary
	* 	Feature	
	* 		Focus	
	* 		Primary Goal
	* 	Heap	
	* 		Data/Objects (Memory)	
	* 		Fix OutOfMemoryError and Leaks.
	* 	Thread	
	* 		Action/Execution (CPU)	
	* 		Fix Performance hangs and Deadlocks.


5.1. What is garbage collection in Java, and why is it needed?
Garbage collection (GC) in Java is an automatic memory management process that reclaims memory occupied by objects that are no longer in use. 
It eliminates the need for manual memory deallocation, reducing memory leaks and improving application stability.

5.2. How does garbage collection work in Java?
The JVM automatically detects objects that are no longer reachable from active threads or static references and removes them to free up memory. The process involves multiple algorithms such as Mark and Sweep, Copying, and Generational GC, which optimize memory management based on object lifetimes.

5.3. What are the different types of garbage collectors in Java?
Java provides several garbage collectors, each optimized for different workloads:
	* Serial GC — 
	* 	Best suited for single-threaded applications with small heaps.
	* Parallel GC (Throughput GC) — 
	* 	Uses multiple threads to perform garbage collection and is ideal for multi-core processors.
	* G1 (Garbage First) GC — 
	* 	Prioritizes low-latency applications by dividing the heap into regions and collecting garbage incrementally.
	* ZGC & Shenandoah GC — 
	* 	Designed for ultra-low pause times and large heaps, introduced in Java 11+ and 12+.

5.4. What is Generational Garbage Collection in Java?
Generational garbage collection divides the heap into Young Generation, Old (Tenured) Generation, and Permanent (Metaspace in Java 8+) Generation. Objects are initially allocated in the Young Generation, and those that survive multiple GC cycles are promoted to the Old Generation. This approach optimizes performance by collecting short-lived
JVM Garbage Collection
Core Concept
Java manages memory automatically using Garbage Collection (GC).

Key areas:
heap
stack
metaspace

Heap structure:
young generation
old generation

Important Collectors
Common GC algorithms:
G1GC (default)
ZGC
Shenandoah
Parallel GC

Concepts Interviewers Expect
	* minor vs major GC
	* object promotion
	* memory leaks in Java
	* 
Example causes:
static collections
ThreadLocal misuse
listeners not removed
 



 
Concurrency & Multithreading
1.	Runnable vs Callable
2.	submit() vs execute()
3.	synchronized vs volatile
4.	What is a deadlock
________________________________________
What is the Java Memory Model (JMM)?
The JMM defines how threads interact through memory and guarantees visibility, ordering, and atomicity of shared variables.
Key rules:
	* Threads can cache variables locally
	* Without synchronization, updates may not be visible
	* volatile, synchronized, and final enforce visibility guarantees.

	* Java Memory Model (JMM) is a specification (JSR-133) 
	* 	defines how Java threads interact through memory, 
	* 	rules for variable visibility, ordering, and atomicity in multithreaded applications. 
	* 	ensures that actions in one thread are properly visible to others, 
	* 	preventing data races caused by CPU caching or compiler instruction reordering. 
	* Key aspects :
	* 	Main Memory vs. Working Memory: 
	* 		JMM uses an abstract model 
	* 			all variables reside in a shared main memory, 
	* 			threads cache variables in local "working memory" (registers/caches) for performance.
	* 	Visibility & Atomicity: 
	* 		governs when writes by one thread become visible to others, 
	* 		managed by keywords like volatile (ensures immediate visibility) and synchronized.
	* 	Happens-Before Relationship: 
	* 		core concept guarantees that memory writes by one specific statement are visible to another, 
	* 		defining the ordering of memory operations.
	* 		"happens-before" does not mean "physically executes before," 
	* 		but the effects of the first action are guaranteed to be visible to the second.
	* 	Purpose: 
	* 		guarantees that properly synchronized code behaves consistently across different hardware platforms and JVM implementations. 
	* Without the JMM, modern compilers and CPUs could reorder code or cache values, leading to unpredictable, incorrect, or stale data in concurrent applications.



________________________________________
2️ What does volatile actually guarantee?
	* volatile guarantees:
		* 1. Visibility
		* 2. Ordering (happens-before)
	* But NOT atomicity.
		* Example problem:
		* volatile int counter;
		* counter++;   // NOT atomic

	* volatile keyword is a modifier used on variables to 
	* 	guarantee visibility and ordering across multiple threads, 
	* 	ensuring that threads do not use cached, stale data. 
	* Core Guarantees of volatile (Java)
	* 	declaring a field as volatile provides the following guarantees:
	* 	Visibility: 
	* 		thread reading a volatile variable will always see the most recent write by any other thread. 
	* 		forces the variable to be read from and written directly to main memory (RAM), bypassing CPU registers or local caches.
	* 	Happens-Before Relationship: 
	* 		write to a volatile variable "happens-before" every subsequent read of that same variable. 
	* 		means Thread A writes to volatile, and Thread B reads , 
	* 		all variables visible to Thread A before writing the volatile variable are 
	* 			guaranteed to be visible to Thread B after it reads the volatile variable.
	* 	Preventing Reordering: 
	* 		compiler and runtime are forbidden from reordering reads and writes of volatile variables with surrounding operations.
________________________________________
3️ What is the "happens-before" relationship?
	* A memory visibility guarantee.
	* Example:
		* unlock → lock
		* write volatile → read volatile
		* thread start → thread run
		* thread finish → join
	* If A happens-before B, B sees A's changes.

	* a foundational guarantee in concurrent programming
	* ensuring that memory writes by one action are visible to another. 
	* Key Aspects :
	* 	Visibility & Ordering: 
	* 		ensures that if Thread 1 writes to a variable and Thread 2 reads it, Thread 2 sees the updated value. 
	* 		also prevents the compiler/CPU from reordering actions in a way that violates this order.
	* 	Program Order: 
	* 		Each action in a single thread happens-before every action in that thread that comes later in the program order.
	* 	Volatile Variables: 
	* 		write to a volatile field happens-before every subsequent read of that same field.
	* 	Locking: 
	* 		unlock (synchronized block exit) on a monitor happens-before every subsequent lock on that same monitor.
	* 	Thread Termination: 
	* 		All actions in a thread happen-before any other thread detects that thread has terminated (via Thread.join() or Thread.isAlive()).
	* 	Thread Start: 
	* 		call to Thread.start() happens-before any action in the started thread.
	* 	Transitivity: 
	* 		A happens-before B, and B happens-before C, then A happens-before C. 
	* 
	* Ao contrário do volatile, que atua em variáveis individuais, o synchronized afeta toda a memória visível para a thread: 
		* Ao Entrar (lock): 
		* 	thread entra num bloco synchronized, ela adquire o monitor do objeto. 
		* 	força a thread a invalidar o seu cache local e recarregar os valores das variáveis diretamente da memória principal.
		* Durante a Execução: 
		* 	thread tem acesso exclusivo ao código dentro do bloco (exclusão mútua), 
		* 	garantindo que nenhuma outra thread altere esses dados simultaneamente.
		* Ao Sair (unlock): 
		* 	Antes de libertar o monitor, a thread é obrigada a escrever (flush) todas alterações locais na  memória principal

________________________________________
4️ Why is Double Checked Locking broken without volatile?
	* Example:
if(instance == null) {
  synchronized(this) {
     if(instance == null)
         instance = new Singleton();
  }
}
	* Without volatile, instruction reordering may expose a partially constructed object.
	* Correct version:
	* private static volatile Singleton instance;

	* Double-Checked Locking (DCL) 
	* 	without the volatile keyword because of instruction reordering during object creation. 
	* The Root Cause: Non-Atomic Object Creation	
	* 	instance = new Singleton(); 
	* 		is not a single atomic operation. 
	* 	three distinct steps at the bytecode level: 
	* 		Allocate memory for the object.
	* 		Execute the constructor to initialize fields.
	* 		Assign the reference of the allocated memory to the instance variable.
	* Why it Fails (The Race Condition)
	* 	The JVM or CPU is allowed to reorder these steps to optimize performance. 
	* 	Specifically, it may swap steps 2 and 3: 
	* 		Thread A enters the synchronized block, allocates memory (Step 1), and then assigns the reference to the variable (Step 3) before the constructor (Step 2) has finished running.
	* 		Thread B calls getInstance(). It performs the first null check outside the synchronized block.
	* 		Thread B sees that instance is not null (because Thread A already assigned the reference) and returns it immediately.
	* 		Result: Thread B receives a partially constructed object. When it tries to use this object, it may encounter default values (like 0 or null) instead of the intended initialized values, leading to unpredictable crashes. 
	* How volatile Fixes It
	* 	variable as volatile, you invoke specific rules of the Java Memory Model
	* 	Prevents Reordering: 
	* 		compiler and CPU are forbidden from reordering a write to a volatile variable with any preceding writes. 
	* 		This ensures the constructor (Step 2) must complete before the reference assignment (Step 3).
	* Happens-Before Guarantee: 
	* 	write to a volatile field happens-before every subsequent read of that same field. 
	* 	This ensures that once Thread B sees a non-null instance, 
	* 	guaranteed to also see all the initialized state of that object.
________________________________________
5️⃣ What causes thread starvation?
	* When a thread never gets CPU time.
	* Common causes:
		* high priority threads
		* unbounded queues
		* lock contention
		* unfair scheduling
	* 
	* Thread Starvation 
	* 	occurs when thread is perpetually denied the resources it needs to make progress. 
	* 	While technically "alive" and ready to run,  scheduler keeps picking other threads instead.
	* Primary causes:
	* 1. Priority Inequity (The "Bully" Problem)
	* 	threads have priorities. 
	* 	constant stream of High-Priority threads intensive work, Low-Priority threads may never get a chance to execute. 
	* 	stay at the back of the line forever.
	* 2. Synchronized Block "Greed"
	* 	thread enters a synchronized block (or acquires a Lock) and stays there for a very long time 
	* 	other threads waiting for that same lock will starve.
	* 	"Non-Fair" Lock: 
	* 		default, Java's ReentrantLock and synchronized are unfair. 
	* 		means when a lock is released, any waiting thread can grab it—potentially a thread just arrived, 
	* 		jumping ahead of threads that have been waiting for minutes.
	* 3. Poorly Designed "Wait/Notify" Logic
	* 	thread is waiting on a condition (object.wait()) and the notification logic is flawed (e.g., using notify() instead of notifyAll()), 
	* 	"wrong" thread might be woken up repeatedly while the "right" one stays asleep in the waiting pool indefinitely.
	* 4. Excessive Thread Contention
	* 	too many threads compete for a small number of resources, 
	* 	overhead of context switching and lock management becomes so high that 
	* 	some threads simply never get their turn in the "hot seat" of the CPU.
	* How to Fix It?
	* 	Fair Locks: 
	* 		Use new ReentrantLock(true). 
	* 		true parameter enforces a "First-In, First-Out" (FIFO) policy, so the thread waiting the longest gets the lock next.
	* 	Avoid Fixed Priorities: 
	* 		Stick to default priorities unless you have a very specific, low-level hardware reason to change them.
	* 	Keep Locks Short: 
	* 		Perform the absolute minimum amount of work inside synchronized blocks.

________________________________________
Difference between synchronized and ReentrantLock?
synchronized	ReentrantLock
JVM managed	Java API
simple	more flexible
no timeout	tryLock()
no fairness control	fairness option
cannot interrupt	interruptible

	* both provide mutual exclusion, 
	* 	synchronized is a built-in language keyword, 
	* 	ReentrantLock is a sophisticated utility class from the java.util.concurrent.locks package.
	* 1. Flexibility & Control
	* 	synchronized: Implicit. 
	* 		enter a block, and the lock is released automatically when you exit (even if an exception occurs). 
	* 		You cannot "try" to acquire it without blocking.
	* 	ReentrantLock: Explicit. 
	* 		must manually call lock() and unlock(). 
	* 		features like tryLock() (attempt to lock without waiting forever) and lockInterruptibly().
	* 2. Fairness
	* 	synchronized: Always unfair. 
	* 		no guarantee which waiting thread will get the lock next; 
	* 		JVM picks one (often the most recent one).
	* 	ReentrantLock: 
	* 		optional fairness parameter (new ReentrantLock(true)). 
	* 		ensures the thread waiting the longest gets the lock first, preventing thread starvation.
	* 3. Condition Variables
	* 	synchronized: 
	* 		only one wait-set per object via wait(), notify(), and notifyAll(). 
	* 		threads wait on the same "signal."
	* 	ReentrantLock: 
	* 		multiple Condition objects. 
	* 		allows you to wake up specific groups of threads (e.g., "Not Full" vs. "Not Empty" in a buffer), 
	* 		which is much more efficient.
	* 4. Performance
	* 	Java (8+), synchronized has been heavily optimized by the JVM (using biased locking and spinning). 
	* 	simple cases, it is often as fast as or faster than ReentrantLock.	
________________________________________
7️⃣ What is a deadlock?
Two threads waiting for locks held by each other.
Example:
Thread A → lock1 → waiting lock2
Thread B → lock2 → waiting lock1
Avoid using consistent lock ordering.


	* A deadlock is a condition where two or more threads are blocked forever, 
	* 	each waiting for a resource that the other thread currently holds.
	* The "Deadly Embrace" Example
	* 	Thread A holds "Lock 1" and needs "Lock 2" to finish its work. 
	* 	Thread B holds "Lock 2" and needs "Lock 1" to finish its work.
	* 	Thread A: "I won't release Lock 1 until I get Lock 2."
	* 	Thread B: "I won't release Lock 2 until I get Lock 1."
	* 	Result: Neither thread ever moves. The program "freezes" or stops responding.
	* The 4 Coffman Conditions (All must be true for a deadlock)
	* 	Mutual Exclusion: 
	* 		Only one thread can use a resource at a time.
	* 	Hold and Wait: 
	* 		thread holds at least one resource while waiting for another.
	* 	No Preemption: 
	* 		resource cannot be forcibly taken from a thread; it must be released voluntarily.
	* 	Circular Wait: 
	* 		chain of threads exists where each thread is waiting for a resource held by the next member in the chain.
	* How to Prevent Deadlocks
	* 	Lock Ordering: 
	* 		acquire locks in the exact same order across all threads (e.g., always Lock 1, then Lock 2). 
	* 		breaks the "Circular Wait" condition.
	* 	Lock Timeouts: 
	* 		Use ReentrantLock.tryLock(timeout). 
	* 		If cannot acquire the second lock within a few milliseconds, it gives up, releases its first lock, and tries again later.
	* 	Deadlock Detection: 
	* 		tools like jstack or VisualVM. 
	* 		The JVM can actually scan your running threads and tell you: "Found 1 deadlock."
java
// Thread 1
synchronized(lock1) {
    synchronized(lock2) { ... }
}
// Thread 2 (DANGER: Reverse order!)
synchronized(lock2) {
    synchronized(lock1) { ... }
}

// Thread 1
synchronized(lock1) {
    System.out.println("Thread 1: Holding lock 1...");
    synchronized(lock2) {
        System.out.println("Thread 1: Holding lock 1 & 2...");
    }
}

// Thread 2 - FIXED (Same order as Thread 1)
synchronized(lock1) {
    System.out.println("Thread 2: Holding lock 1...");
    synchronized(lock2) {
        System.out.println("Thread 2: Holding lock 1 & 2...");
    }
}
	* Professional Strategies for Deadlock Prevention
	* 	Lock Ordering by HashCode: 
	* 		two arbitrary objects (objA and objB), you can compare their System.identityHashCode() to decide which one to lock first.
	* 	Try-Lock with Timeout: 
	* 		Use ReentrantLock.tryLock(timeout, unit). 
	* 		If a thread can't get the second lock within 50ms, it releases the first lock and tries again later. 
	* 		This breaks the Hold and Wait condition.
	* 	Global Lock: 
	* 		Instead of locking individual resources, use a single "Master Lock" to protect the entire operation 
	* 		(though this reduces performance).

________________________________________
8️⃣ What is livelock?
Threads keep reacting to each other but never progress.
Example:
Two threads repeatedly retry operations and yield.

	* livelock 
	* 	situation where two or more threads continuously change their state in response to each other, 
	* 	but none of them make any actual progress. 
	* 	in a deadlock threads are "frozen" (waiting), 
	* 	in livelock threads are "too active"—they are running, consuming CPU cycles, and constantly yielding to each other. 
	* The "Hallway Analogy"
	* 	Imagine two people meeting in a narrow hallway.
	* 	Person A moves to their left to let Person B pass.
	* 	At the same time, Person B moves to their right to let Person A pass.
	* 	They are now blocking each other again.
	* 	They both apologize and move to the opposite side at the same time.
	* 	They continue this "dance" forever. 
	* 	They are not "stuck" (they are moving), but they are getting nowhere.
	* Common Causes
	* 	Over-politeness: 
	* 		threads programmed to release a resource if they see another thread waiting for it.
	* 	Poorly designed retry logic: 
	* 		Two threads fail an operation, wait exactly 100ms, and then both retry at the exact same time, failing again.
	* 	Automatic recovery: 
	* 		system detects an error and tries to fix it, but the "fix" triggers the same error again in a loop. 
	* How to Fix It
	* 	Randomized Backoff: 
	* 		Instead of retrying immediately or at fixed intervals, 
	* 		have threads wait for a random amount of time (e.g., between 10ms and 50ms). 
	* 		This ensures one thread will likely start before the other and break the loop. 
	* 		This is how Ethernet protocols handle collisions.
	* 	Priority: 
	* 		Designate one thread as the "primary" that doesn't yield in specific conflict scenarios. 
	* 
	* 1. Livelock (The "Polite Dance")
	* 	What it is: Threads are active and running, but they are stuck in a loop of responding to each other.
	* 	The State: The thread is RUNNING, consuming CPU cycles, but doing zero useful work.
	* 	Analogy: Two people in a hallway constantly stepping in the same direction to let the other pass, over and over.
	* 2. Thread Starvation (The "Ignored Waiter")
	* 	What it is: A thread is ready to run but is denied CPU time because other "greedier" or higher-priority threads keep jumping ahead in line.
	* 	The State: The thread is WAITING or BLOCKED, consuming almost no CPU, just sitting there.
	* 	Analogy: You are at a busy restaurant, but the waiter keeps serving every new table that arrives while you’ve been waiting for an hour.

________________________________________
9️⃣ Difference between Callable and Runnable?
Runnable	Callable
no return value	returns value
no checked exceptions	can throw
run()	call()
Used with:
ExecutorService
Future

	* Callable is more powerful and flexible than Runnable.
	* 1. Return Value
	* 	Runnable: 
	* 		run() method has a void return type. 
	* 		cannot return a result to the caller.
	* 	Callable: 
	* 		call() method returns a V (a generic value). 
	* 		allows the task to send a result back after execution.
	* 2. Exception Handling
	* 	Runnable: 
	* 		run() method cannot throw checked exceptions. 
	* 		must handle any IOException or SQLException inside a try-catch block within the method.
	* 	Callable: 
	* 		call() method signature includes throws Exception. 
	* 		can throw checked exceptions, which will then be caught by the thread that retrieves the result.
	* 3. Invocation Method
	* 	Runnable: 
	* 		executed by the 
	* 			Thread class (new Thread(runnable).start()) or 
	* 			ExecutorService.
	* 	Callable: 
	* 		must be submitted to an ExecutorService, 
	* 		returns a Future<V> object to track the result.
	* 
// Runnable: Fire and forget
Runnable task1 = () -> System.out.println("Doing work...");

// Callable: Calculate and return
Callable<Integer> task2 = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task2);

// This blocks until the result is ready
Integer result = future.get(); 
	* 
	* O método future.get() pode lançar três tipos principais de exceções:
	* 	ExecutionException: 
	* 		mais importante. 
	* 		se Callable lançou IOException, por exemplo, ela estará "escondida" aqui dentro. 
	* 		aceder à causa real usando e.getCause().
	* 	InterruptedException: 
	* 		se a thread atual (a que está à espera do resultado) for interrompida antes do cálculo terminar.
	* 	CancellationException: 
	* 		se a tarefa tiver sido cancelada através do método future.cancel().
	* 
Callable<Integer> tarefaPerigosa = () -> {
    throw new IOException("Falha no Disco!"); // Exceção original
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(tarefaPerigosa);

try {
    Integer resultado = future.get(); // Bloqueia e espera
} catch (ExecutionException e) {
    // Aqui capturamos o erro que veio da outra thread
    Throwable causaReal = e.getCause(); 
    System.err.println("A tarefa falhou com: " + causaReal.getMessage());
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // Boa prática
}
	* 
	* In Java 8+, CompletableFuture revolutionized error handling 
	* 	allowing you to handle exceptions functionally, 
	* 	without the clunky try-catch blocks required by the older Future.get().
	* 	.exceptionally() method acts like a safety net at the end of your processing chain.
	* 	1. How it works
	* 		any stage in your asynchronous pipeline throws an exception, 
	* 		execution skips the remaining "happy path" steps and jumps straight to the .exceptionally() block. 
	* 		block receives the Throwable and allows you to return a fallback value.
	* 	2. Code Example: The Functional Way
	* 	
	CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
		if (Math.random() > 0.5) {
			throw new RuntimeException("Database Timeout!");
		}
		return 42;
	});

	future.thenApply(result -> result * 2) // Happy path
		  .exceptionally(ex -> {           // Error path
			  System.err.println("Task failed: " + ex.getMessage());
			  return 0;                    // Fallback value
		  })
		  .thenAccept(finalResult -> {
			  System.out.println("Result: " + finalResult);
		  });
	* 
	* 	3. Key Methods for Error Handling
	* 		.exceptionally(ex -> ...): 
	* 			Best for providing a simple default/fallback value if something goes wrong.
	* 		.handle((result, ex) -> ...): 
	* 			"Swiss Army Knife." 
	* 			always executed, receiving both the result and the exception. 
	* 			can check if ex != null and decide what to return.
	* 		.whenComplete((result, ex) -> ...): 
	* 			Similar to finally. 
	* 			lets you perform an action (like logging) but does not change the result or the exception.
	* 	Why is this better?
	* 		Unlike Future.get(), which blocks your thread until the result is ready (or fails), 
	* 		CompletableFuture is non-blocking. 
	* 		main thread continues working, 
	* 		error handler is triggered automatically whenever the background task fails.


________________________________________
10 What is the difference between submit() and execute()?
execute() → Runnable only
submit() → Runnable or Callable
submit() returns:
Future

	* both methods used to send tasks to an ExecutorService, 
	* choice depends on whether you care about the task's result or its outcome.
	* 1. execute() (Fire and Forget)
	* 	Source: 
	* 		Defined in the Executor interface.
	* 	Input: 
	* 		Accepts only Runnable tasks.
	* 	Return Value: 
	* 		Returns void. 
	* 		no way of knowing when the task finishes or if it succeeded.
	* 	Exception Handling: 
	* 		task throws an uncaught exception, 
	* 		thread typically prints the stack trace to the console and may die (depending on the pool type). 
	* 		cannot "catch" this error from the calling thread.
	* 2. submit() (Track and Retrieve)
	* 	Source: 
	* 		Defined in the ExecutorService interface.
	* 	Input: 
	* 		Accepts both Runnable and Callable tasks.
	* 	Return Value: 
	* 		Returns a Future<?> object. 
	* 		your "receipt" to check the status or get the result.
	* 	Exception Handling: 
	* 		Any exception thrown inside the task is captured by the Future. 
	* 		It won't crash the console immediately; 
	* 		thrown as an ExecutionException only when you call future.get().
	* Quick Rule of Thumb
	* 	Use execute() 
	* 		just want to run a background task and don't need to hear from it again.
	* 	Use submit() 
	* 		need to wait for the task to finish, 
	* 		get a return value, or 
	* 		handle potential errors gracefully.
	* import java.util.*;
	* import java.util.concurrent.*;
	* 
	* public class ExemploInvokeAll {
	*     public static void main(String[] args) throws InterruptedException {
	*         ExecutorService executor = Executors.newFixedThreadPool(3);
	* 
	*         // 1. Criar uma lista de tarefas (Callables)
	*         List<Callable<String>> tarefas = Arrays.asList(
	*             () -> { Thread.sleep(2000); return "Relatório A pronto"; },
	*             () -> { Thread.sleep(1000); return "Relatório B pronto"; },
	*             () -> { throw new RuntimeException("Erro no Relatório C"); }
	*         );
	* 
	*         System.out.println("A iniciar processamento em lote...");
	* 
	*         // 2. invokeAll() submete todas e ESPERA que todas acabem
	*         List<Future<String>> resultados = executor.invokeAll(tarefas);
	* 
	*         System.out.println("Todas as tarefas terminaram. A ler resultados:");
	* 
	*         // 3. Processar os resultados
	*         for (Future<String> f : resultados) {
	*             try {
	*                 // Como invokeAll esperou, o get() aqui é instantâneo
	*                 System.out.println("Resultado: " + f.get());
	*             } catch (ExecutionException e) {
	*                 System.err.println("Tarefa falhou: " + e.getCause().getMessage());
	*             }
	*         }
	* 
	*         executor.shutdown();
	*     }
	* }
Thread vs Process

A thread is a lightweight, independent unit of execution inside a program (process).
	* A process can have multiple threads.
	* Each thread runs independently but shares the same memory

Ways to Create Threads
1. Extending the Thread class
	* We create a class that extends Thread and override its run() method to define the task. 
	* Use extends Thread: if your class does not extend any other class.

class CookingTask extends Thread {
…
    public void run() {
        System.out.println(task + " is being prepared by " +
            Thread.currentThread().getName());
    }
}
public class Restaurant {
    public static void main(String[] args) {
        Thread t1 = new CookingTask("Pasta");
        t1.start();

2. Implementing the Runnable Interface
	* We create a new class which implements java.lang.Runnable interface and define the run() method there. 
class CookingJob implements Runnable {
…
    public void run() {
        System.out.println(task + " is being prepared by " +
            Thread.currentThread().getName());
    }
}
public class RestaurantRunnable {
    public static void main(String[] args) {
        Thread t1 = new Thread(new CookingJob("Soup"));
        t1.start();

Life Cycle of a Thread
	* During its thread life cycle, a Java thread transitions through several states from creation to termination.
	* New Thread: 
	* When a new thread is created, it is in the new state. 
	* Runnable State: 
	* A thread that is ready to run is moved to a runnable state. 
	* In this state, a thread might actually be running or it might be ready to run at any instant of time. 
	* It is the responsibility of the thread scheduler to give the thread, time to run. 
	* Each and every thread get a small amount of time to run. 
	* After running for a while, a thread pauses and gives up the CPU so that other threads can run.
	* Blocked: 
	* when it is trying to acquire a lock but currently the lock is acquired by the other thread. 
	* will move from the blocked state to runnable state when it acquires the lock.
	* Waiting state: 
	* in waiting state when it calls wait() method or join() method. 
	* move to the runnable state when other thread will notify or that thread will be terminated.
	* Timed Waiting: 
	* lies in a timed waiting state when it calls a method with a time-out parameter. 
	* in this state until the timeout is completed or until a notification is received. 
	* when a thread calls sleep or a conditional wait, it is moved to a timed waiting state.
	* Terminated State: 
	* terminates because of either of the following reasons: 
	* it exits normally. 
	* occurred some unusual erroneous event, like a segmentation fault or an unhandled exception.

Java Thread.start() vs Thread.run() Method
	* Thread.start() Method
	* The start() method is used to begin a new thread of execution. It performs two main tasks:
	* Allocates resources for a new thread.
	* Calls the run() method internally in the new thread.
	* Thread.run() Method
	* The run() method contains the code executed by the thread. However, calling run() directly does not create a new thread. Instead, it behaves like a normal method call executed in the current thread.
Java Thread.sleep() Method
	* used to pause the execution of the currently running thread for a specified amount of time. 
	* duration ends, the thread becomes runnable again and continues execution based on thread scheduling.
Java Runnable Interface
	* part of the java.lang package and is used to define a task that can be executed by a thread.
Step 1: Implement Runnable
	* class that implements the Runnable interface and 
	* override the run() method to define the task to be executed by the thread.
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}
Step 2: Create Thread Object
	* Create a Thread object by passing the Runnable instance and 
	* call start() to execute the task in a new thread.
    public static void main(String[] args){        
        MyTask task = new MyTask();
        Thread t = new Thread(task);
        t.start(); // starts a new thread
    }
Runnable with Lambda Expression
	* Since Runnable is a functional interface, it can be written using lambda expressions.
        Runnable task = () -> {
            System.out.println("Thread running using lambda");
        };
        Thread t = new Thread(task);
        t.start();
    
Runnable with Executor Framework
	* Executor Framework provides a high-level API for managing threads efficiently. 
	* Instead of creating threads manually, tasks are submitted to a thread pool.
        
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.execute(() -> {
            System.out.println("Task executed by Executor");
        });
        executor.shutdown();
    
Joining Threads in Java
	* java.lang.Thread class provides the join() method which allows one thread to wait until another thread completes its execution. 
	* t.join() will make sure that t is terminated before the next instruction is executed by the program.
	* three overloaded join functions.
	* 1. join(): 
	* put the current thread on wait until the thread on which it is called is dead. 
	* If a thread is interrupted then it will throw InterruptedException.
	* 2. join(long millis): 
	* put the current thread on wait until the thread on which it is called is dead 
	* or wait for the specified time (milliseconds).
	* 3. join(long millis, int nanos): 
	* put the current thread on wait until the thread on which it is called is dead or 
	* wait for the specified time (milliseconds + nanos).
Main thread in Java
	* When program starts, the JVM creates a thread automatically called the main thread. 
	* This thread executes the main() method and controls the overall execution flow of the program.
	* is the parent thread from which all other user-defined threads are created.
	* default name of the main thread is "main".
	* The default priority of the main thread is 5.
	* It usually finishes last as it may perform cleanup and shutdown tasks.
Relationship Between main() Method and Main Thread
	* For every Java program, JVM creates the main thread first.
	* The main thread checks for the existence of the main() method, then initializes the class.
	* Since JDK 6, the main() method is mandatory for a standalone Java application.
Deadlock Using Main Thread
	* Even a single main thread can cause deadlock if it waits for itself.
      // Joining the current thread
      Thread.currentThread().join();

      // This statement will never execute
      System.out.println("This statement will never execute");

Java Thread Priority in Multithreading
	* multiple threads run concurrently and the Thread Scheduler decides their execution order. 
	* Each thread is assigned a priority (1–10) that influences scheduling but does not guarantee execution order.
	* Thread priority ranges from 1 (MIN_PRIORITY) to 10 (MAX_PRIORITY)
	* Higher-priority threads are generally preferred by the scheduler
	* Actual execution order depends on the JVM and underlying OS
Java Daemon Thread
	* daemon thread is a low-priority background thread in Java that supports user threads and 
	* does not prevent the JVM from exiting. VM exits automatically when all user threads finish.
	* ideal for background tasks like monitoring, logging, and cleanup.
	* Runs in the background to support user (non-daemon) threads.
	* Created using the Thread class and marked as daemon with setDaemon(true).
	* setDaemon(true) must be called before starting the thread, or it throws IllegalThreadStateException.
	* Garbage Collection: 
	* Garbage Collector (GC) in Java runs as a daemon thread.
	* Background Monitoring: 
	* can monitor the state of application components, resources, or connections.
	* Logging and Auditing Services: 
	*  can be used to log background activities continuously.
	* Cleanup Operations: 	
	*  may periodically clear temporary files, release unused resources, or perform cache cleanup.
	* Scheduler or Timer Tasks: 
	* Background schedulers often use daemon threads to trigger tasks at fixed intervals.

Synchronization in Java
	* mechanism that ensures that only one thread can access a shared resource (like a variable, object, or method) at a time. 
	* prevents concurrent threads from interfering with each other while modifying shared data.
	* Prevents Data Inconsistency:  multiple threads don’t corrupt shared data 
	* Avoids Race Conditions:  only one thread to execute a critical section at a time
	* Maintains Thread Safety: Protects shared resources from concurrent modification.
	* Ensures Data Integrity: Keeps shared data accurate and consistent 
1. Synchronized Methods
	* A synchronized method ensures that only one thread can execute it at a time on the same object instance
    public synchronized void inc(){
        c++; 
2. Synchronized Blocks
	* allows synchronization on specific blocks of code. 
	* improves performance by locking only the necessary section.
    public void inc(){
        synchronized (this) { c++; }
    }

3. Static Synchronization
	* Static synchronization is used to synchronize static methods. 
	* the lock is placed on the class object rather than the instance.
    synchronized static void printTable(int n){   
        for (int i = 1; i <=3; i++){            
            System.out.println(n * i);
            try {
            } catch (Exception e) {
                System.out.println(e);
            }
        }
    }

Types of Synchronization
1. Process Synchronization
	* Process Synchronization is a technique used to coordinate the execution of multiple processes. 
	* ensures that the shared resources are safe and in order.
2. Thread Synchronization in Java
	* Thread Synchronization is used to coordinate and ordering of the execution of the threads in a multi-threaded program. There are two types of thread synchronization are mentioned below:
		* Mutual Exclusive
		* Cooperation (Inter-thread communication in Java)
Volatile Keyword
	* volatile keyword in Java ensures that all threads have a consistent view of a variable's value. 
	* prevents caching of the variable's value by threads, ensuring that updates to the variable are immediately visible to other threads.
	* It applies only to variables.
	* volatile guarantees visibility i.e. any write to a volatile variable is immediately visible to other threads.
	* It does not guarantee atomicity, meaning operations like count++ (read-modify-write operations) can still result in inconsistent values
Inter-thread Communication in Java
	* Inter-thread communication in Java 
		*  mechanism in which a thread is paused from running in its critical section, and another thread is allowed to enter (or lock) the same critical section to be executed.
		* also known as Cooperation in Java.
		* Java uses three methods, namely, wait(), notify(), and notifyAll(). 
	All these methods belong to the object class, so all classes have them. 
	They must be used within a synchronized block only. 
		* wait(): 
	tells the calling thread to give up the lock and go to sleep until some other thread calls notify().
		* notify(): 
	wakes up one single thread called wait() on the same object. 
	should be noted that calling notify() does not give up a lock on a resource.
		* notifyAll(): 
	wakes up all the threads called wait() on the same object.
	* Producer-Consumer Problem
    private static final Queue<Integer> queue = new LinkedList<>();    
    // Maximum capacity of the queue
    private static final int CAPACITY = 10;
    // Producer task
    private static final Runnable producer = new Runnable() {
        public void run() {
            while (true) {
                synchronized (queue) {
                    
                    // Wait if the queue is full
                    while (queue.size() == CAPACITY) {
                        try {
                            System.out.println("Queue is at max capacity");
                            queue.wait(); // Release the lock and wait
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    // Add item to the queue
                    queue.add(10);
                    System.out.println("Added 10 to the queue");
                    queue.notifyAll(); // Notify all waiting consumers
                    try {
                        Thread.sleep(2000); 
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    };


Java Thread Safety and How to Achieve it?
1. Using Synchronization
	* synchronized keyword to ensure only one thread can access a method or block of code at a time. 
	* helps in preventing race conditions.
2. Using Volatile Keyword
	* Volatile Keyword ensures visibility of variable changes across threads. 
	* does not guarantee atomicity, but ensures every thread reads the latest value from main memory, not cache.
3. Using Atomic Variable
	* an atomic variable is another way to achieve thread-safety in java. 
	* variables are shared by multiple threads, the atomic variable ensures that threads don't crash into each other. 
4. Using Immutable Objects
	* Immutable objects are inherently thread-safe, as their state cannot change after creation. 
	* Mark fields as final and don’t provide setters.

How to Use Locks in Multi-Threaded Java Program?
	* 
	* 
	* 
	* a lock is a synchronization mechanism that ensures mutual exclusion for critical sections in a multi-threaded program. 
	* controls access to shared resources, ensuring thread safety. 
	* Exclusive Lock(ReentrantLock): 
	* Only one thread can acquire the lock at a time.
	* Shared Lock(ReadWriteLock): 
	* Multiple threads can hold the lock simultaneously under certain conditions.
	* Basic usage of a Lock:
Lock lock = new ReentrantLock();
lock.lock(); // Acquire the lock
// Critical section
lock.unlock(); // Release the lock

1. ReentrantLock
	* A ReentrantLock() is an implementation of Lock that allows a thread to reacquire the lock it already holds
2. ReadWriteLock
	* A ReadWriteLock allows 
	* multiple threads to read a resource concurrently, 
	* only one thread to write, 
	* ensuring no reads happen during writing.

Difference Between Lock and Monitor in Java Concurrency
	* Monitor (synchronized)	
	* Lock (ReentrantLock)
	* Origin
	* M: JVM intrinsic, low-level primitive	
	* L: Java 5, high-level API
	* Implementation	
	* M:Implicit, JVM-managed	
	* L:Explicit, programmer-managed
	* Critical Section Management
	* M:Automatic	
	* L:Manual (lock() / unlock())
	* Thread Queueing	
	* M:JVM manages waiting threads	
	* L:Programmer can choose fairness policies
	* Features	
	* M:Simple mutual exclusion	
	* L:Advanced: fairness, interruptible, tryLock, multiple conditions
	* Performance	
	* M:Lightweight for small threads	
	* L:Slightly higher overhead, but more flexible
	* Usage	
	* M:Simple synchronization, small thread pools	
	* L:Complex synchronization, high concurrency scenarios
	* Deadlock Handling	
	* M:Less control	
	* L:More explicit control but can still occur

Java Lock Framework vs Thread Synchronization
	* Thread Synchronization
	* is the simplest mechanism to control access to shared resources. 
	* Rule of Thumb: 
	* simple scenarios, synchronized is enough. 
	* scalable, complex multi-threading, prefer Lock framework.
ReentrantLock in Java
	* Reentrant Lock 
	* provides a more flexible mechanism for thread synchronization compared to the synchronized keyword. 
	* allows threads to enter a lock multiple times (reentrant behavior) without causing deadlock on itself.
	* features like:
	* Reentrancy: 
	* same thread can acquire the lock multiple times. 
	* Each lock acquisition must be paired with a corresponding unlock.
	* Explicit Locking: 
	* requires manual locking and unlocking using lock() and unlock().
	* Interruptible: 
	* Threads waiting for a lock can be interrupted.
	* TryLock() Support: 
	* Threads can attempt to acquire a lock without waiting indefinitely.
	* Fairness Policy: 
	* Locks can be configured to grant access in first-come-first-serve order

Deadlock in Java Multithreading
	* Deadlock is where two or more threads are permanently blocked because each one is waiting for the other to release a required lock. 
	* Preventing Deadlocks
	* Avoid Nested Locks: 
	* main reason for deadlock. 
	* Mainly happens when we give locks to multiple threads. 
	* Avoid giving lock to multiple threads if we already have given to one.
	* Avoid Unnecessary Locks: 
	* should have lock only those members who are required. 
	* lock on unnecessarily can lead to deadlock.
	* Using thread join:
	* Deadlock condition appears when one thread is waiting for the other to finish. 
	* If this condition occurs we can use Thread. Join 
	* Detecting Deadlocks
	* We can detect deadlocks in a running Java program using the following steps:
	* 1. List the active Java processes:
	* jps -l
	* Lock ordering 
	*  technique used to prevent deadlocks  
	* by enforcing a strict, consistent sequence in which threads or processes must acquire multiple locks. 
	* ensuring all components request resources in the same order (e.g., lock A then lock B), you eliminate circular wait conditions, a primary cause of deadlock
Thread Pool in Java
	* a collection of pre-created, reusable threads that are kept ready to perform tasks. 
	* Instead of creating a new thread every time you need to run something (which is costly in terms of memory and CPU), a thread pool maintains a fixed number of threads. 
	* When a task is submitted:
	* If a thread is free, it immediately picks up the task and runs it.
	* If all threads are busy, the task waits in a queue until a thread becomes available.
	* After finishing a task, the thread does not die, goes back to the pool and waits for the next task.
	* Benefits of Thread Pool
	* Better Performance: 
	* are reused instead of being created and destroyed repeatedly.
	* Faster Response Time: 
	* Tasks don’t need to wait for a new thread to be created.
	* Reusability: 
	* Threads remain alive after finishing tasks and are reused for future tasks.
	* Resource Management: 
	* Limits the number of concurrent threads, preventing OutOfMemoryError or CPU overload.
Java Executor Framework
	* Executor Framework is a part of java.util.concurrent package introduced in Java 5 provides a high-level API for managing thread execution. 
	* developers submit tasks without manually creating or controlling threads, as the framework handles scheduling and execution.
1. Executor Interface
	* Executor Interface is root interface of the framework is used to execute submitted tasks without explicitly creating threads. 
	* Defines a single method:
		* Executor executor = command -> new Thread(command).start();
executor.execute(() -> System.out.println("Task executed"));
2. ExecutorService Interface
	* extends the Executor interface 
	* advanced methods for task management, such as 
		* submitting tasks that return results and 
		* controlling executor shutdown.
	* Supports both Runnable and Callable tasks.
	* Provides lifecycle management (shutdown, awaitTermination).
	* Can execute multiple tasks simultaneously.
		* ExecutorService service = Executors.newFixedThreadPool(2);
service.submit(() -> System.out.println("Running a task"));
service.shutdown();
3. ScheduledExecutorService Interface
	* extends ExecutorService and 
	* supports task scheduling, running tasks periodically or after a delay.
		* ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
scheduler.scheduleAtFixedRate(task, 0, 5, TimeUnit.SECONDS);
4. ThreadPoolExecutor Class
	* ThreadPoolExecutor is the most commonly used implementation of ExecutorService. It manages a pool of worker threads to execute tasks efficiently, reusing threads to reduce overhead.
	* Controls core pool size, maximum pool size, and queue capacity.
		* ExecutorService executor = Executors.newFixedThreadPool(3);
executor.execute(task);
5. AbstractExecutorService Class
	* A base class that provides default implementations for ExecutorService methods. 
	* Simplifies creating custom executors by handling common functionalities like submit() and invokeAll().
Common Types of Executors in Java
	* utility class provides factory methods to easily create different kinds of thread pools. 
	* Each type is designed for specific concurrency requirements.
	* SingleThreadExecutor	
		* Creates a thread pool with a single thread that executes tasks sequentially.	
		* ExecutorService executor = Executors.newSingleThreadExecutor();
	* FixedThreadPool	
		* Creates a pool with a fixed number of threads. Excess tasks are queued until a thread becomes available.	
		* ExecutorService pool = Executors.newFixedThreadPool(2);
	* CachedThreadPool	
		* Creates threads as needed and reuses idle ones. Suitable for many short-lived asynchronous tasks.	
		* ExecutorService pool = Executors.newCachedThreadPool();
	* ScheduledThreadPool	
		* Executes tasks periodically or after a specified delay.	
		* ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

Difference Between Callable and Runnable in Java
	* Callable interface and Runnable interface are used to encapsulate tasks supposed to be executed by another thread.
	* Runnable instances can be run by Thread class as well as ExecutorService but Callable instances can only be executed via ExecutorService.
	* Callable Interface 
		* basically throws a checked exception and returns some results. 
		* the major differences between the upcoming Runnable interface where no value is being returned. 
		* it simply computes a result else throws an exception if unable to do so.
		* contains a single, no-argument method, called call() method
		* can’t create a thread by passing callable as a parameter.
		* Callable can return results. call() method contains the “throws Exception” clause, so we can easily propagate checked exceptions further.
	* Runnable interface 
		* is used to create a thread, starting the thread causes the object run method to be called in a separately executing thread. 
		* represents a task in Java that is executed by Thread.
		* two ways to start a new thread using Runnable, implementing the Runnable interface and by subclassing the Thread class.
		* cannot return the result of computation which is essential if you are performing some computing task in another thread, and 
		* cannot throw checked exceptions.

Callable vs Future in Java
	* Callable Interface
	* The Callable interface represents a task that returns a result and may throw an exception. 
		* ExecutorService executor = Executors.newSingleThreadExecutor();
		*         Callable<Integer> task = () -> {
		*             int sum = 0;
		*             for (int i = 1; i <= 5; i++) sum += i;
		*             return sum;  // returns result
		*         };
		*         Future<Integer> future = executor.submit(task);
		* 
		*         System.out.println("Result: " + future.get()); 
		*         executor.shutdown();
	* Future Interface
		* represents the result of an asynchronous computation. 
		* •  submit(Runnable) → returns Future<?>
		* •  submit(Callable<T>) → returns Future<T>
		* •  execute(Runnable) → returns void (no Future)
		* Since Runnable does not produce a result, the returned Future:
	Returns null from get() if execution completes successfully
	Allows you to:
	* Check completion (isDone())
	* Cancel the task (cancel(true))
	* Detect exceptions (wrapped in ExecutionException when calling get())
CompletableFuture vs Future
	* was introduced in Java 8 with additional features like option to chain function calls onto the result of the initial task.


1. Blocking vs Non-Blocking
	* Future: 
	* you submit a task to an ExecutorService, and it returns a Future object. 
	* to get the result of the computation, you must call the get() method, which blocks the calling thread until the task is complete. 
	* make your application less responsive, as it waits for the computation to finish.
	* CompletableFuture:
	* get non-blocking execution. you define what happens when the computation is done through methods like thenApply(), thenAccept(), and thenRun(). 
	* makes your application much more responsive, as it doesn't block the main thread while waiting for the result.
		* // Using Future
		* Future<String> future = executor.submit(() -> {
		*     Thread.sleep(2000);
		*     return "Hello, Future!";
		* });
		* String result = future.get();  // Blocks until the result is ready
		* 
		* // Using CompletableFuture
		* CompletableFuture.supplyAsync(() -> {
		*     Thread.sleep(2000);
		*     return "Hello, CompletableFuture!";
		* }).thenAccept(System.out::println);  // Non-blocking
2. Callback Support
	* Future: 
	*  doesn't allow you to attach callbacks that define what to do when the computation is done. 
	* CompletableFuture: 
	* provides full support for chaining tasks. 
	* can attach various callbacks using thenApply(), thenAccept(), or thenRun() to define what should happen after the task completes, 
3. Exception Handling
	* Future: 
	* provides no built-in mechanism to handle exceptions. 
	* must catch exceptions when you call get(), making error handling a bit clunky.
	* CompletableFuture: 
	* robust exception handling with methods like exceptionally() and handle(). 
	* allow you to define fallback actions or recovery mechanisms within the same async computation chain, ensuring smoother error management.
	CompletableFuture.supplyAsync(() -> {
	    if (new Random().nextBoolean()) throw new RuntimeException("Oops!");
	    return "Task completed";
	})
	.exceptionally(ex -> "Error: " + ex.getMessage())
	.thenAccept(System.out::println);
4. Composing Results
	* Future: 
	* inability to combine multiple Future instances or chain tasks. 
	* CompletableFuture: 
	* allows you to compose and combine asynchronous tasks easily using methods like thenCompose(), thenCombine(), or run multiple tasks in parallel with allOf().
	* CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Task 1");
		* CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "Task 2");
		* 
		* future1.thenCombine(future2, (result1, result2) -> result1 + " and " + result2)
		*        .thenAccept(System.out::println);  // Combines results of both tasks

5. Asynchronous Execution Support
	* Future: 
	* needs to be paired with an ExecutorService, which adds some overhead. 
	* CompletableFuture: 
	* asynchronous execution is built-in. 
	* supplyAsync() and runAsync() allow you to run tasks asynchronously without needing an ExecutorService.
________________________________________
		* // Using Future
		* ExecutorService executor = Executors.newFixedThreadPool(2);
		* Future<String> future = executor.submit(() -> {
		*     // Simulating long-running task
		*     Thread.sleep(2000);
		*     return "Hello, Future!";
		* });
		* 
		* try {
		*     // Blocking call, waiting for the result
		*     String result = future.get();
		*     System.out.println(result);
		* } catch (Exception e) {
		*     e.printStackTrace();
		* }
		* 
		* // Using CompletableFuture
		* CompletableFuture.supplyAsync(() -> {
		*     // Simulating long-running task
		*     try { Thread.sleep(2000); } catch (Exception e) {}
		*     return "Hello, CompletableFuture!";
		* })
		* .thenApply(result -> result + " with Async Chaining!")
		* .thenAccept(System.out::println)
		* .exceptionally(e -> {
		*     System.out.println("Exception occurred: " + e);
		*     return null;
		* });
6. CompletableFuture methods

•  supplyAsync: 
Executes a task asynchronously and returns the result in a CompletableFuture.
•  runAsync
Executes a task asynchronously without returning any result.
•  thenApply: 
	Transforms the result of a CompletableFuture once it completes.
•  thenAccept: 
	Consumes the result of a CompletableFuture after completion without returning a value.
•  thenRun: 
	Runs a runnable task after the CompletableFuture completes without using its result.
•  thenCompose: 
	Chains two CompletableFuture tasks sequentially by using the result of the first to trigger the second.
•  thenCombine: 
	Combines results of two independent CompletableFuture tasks and processes them together.
•  allOf: 
	Waits for all CompletableFuture tasks to complete and combines them.
•  anyOf: 
	Waits for the first CompletableFuture task to complete, then proceeds.
•  exceptionally: 
	Handles exceptions in the async pipeline by providing a fallback value.
•  handle: 
	Processes both the result and any exceptions from a CompletableFuture.
•  complete: 
	Manually completes the CompletableFuture with a result.
•  completeExceptionally: 
	Completes the CompletableFuture exceptionally with a given error.
•  join: 
	Similar to get(), but throws unchecked exceptions, making it more convenient to use in async chains.
•  orTimeout: 
	Sets a timeout for the completion of the CompletableFuture and triggers an exception if exceeded.
•  completeOnTimeout: 
	Completes the CompletableFuture with a default value if the task takes too long.
3.	 
Concurrency & Multithreading

Core Concept
Java concurrency revolves around:
threads
shared memory
synchronization

Key tools:
synchronized
volatile
locks
executors
CompletableFuture

Important Concepts
Java Memory Model
Defines visibility rules between threads.
Happens-before relationship
Guarantees memory visibility.

Example:
volatile write -> volatile read
unlock -> lock
thread start -> thread execution
Common Pitfalls
	* race conditions
	* deadlocks
	* excessive locking
	* misuse of shared mutable state

________________________________________
1.	CompletableFuture

Core Concept
Represents asynchronous computation pipelines.
Example:
CompletableFuture.supplyAsync(() -> fetchData())
    .thenApply(data -> process(data))
    .thenAccept(result -> save(result));
Important Concepts
Non-blocking pipelines
thenApply
thenCompose
thenCombine
Error handling
exceptionally
handle
whenComplete
Common Pitfalls
	* blocking with get()
	* ignoring error handling
	* mixing blocking IO with async tasks

________________________________________
2.	Virtual Threads (Java 21)

Core Concept
Virtual threads are lightweight threads managed by the JVM.
Example:
Thread.startVirtualThread(() -> handleRequest());

Key Idea
Traditional threads are expensive.
Virtual threads allow:
millions of concurrent tasks

Ideal for:
IO-bound workloads
microservices
web servers
Important Concept
Thread-per-request model becomes viable again.


3.	4️⃣ Concurrency & Multithreading Questions
Extremely Common
	* Difference between Runnable and Callable
	* What is ExecutorService?
	* What is the difference between submit() and execute()?
	* What is synchronization?
	* What is the volatile keyword?
	* What is a race condition?
	* What is a deadlock?
	* Runnable vs. Callable: 
	* 	Runnable defines a task that does not return a result or throw checked exceptions. 
	* 	Callable returns a value (via Future) and can throw checked exceptions.
	* ExecutorService: 
	* 	A framework that manages a pool of threads, 
	* 	decoupling task submission from thread creation and lifecycle management.
	* submit() vs. execute(): 
	* 	execute() is for Runnable tasks and returns nothing (void). 
	* 	submit() can take Runnable or Callable and returns a Future object to track the result or status.
	* Synchronization: 
	* 	A mechanism that ensures only one thread at a time can 
	* 	access a shared resource or block of code, preventing data inconsistency.
	* volatile Keyword: 
	* 	Ensures that changes to a variable are always visible to all threads 
	* 	by reading from/writing to main memory instead of CPU caches. It does not guarantee atomicity.
	* Race Condition: 
	* 	A flaw where the output depends on the unpredictable timing of 
	* 	threads accessing shared mutable state 
	* 	(e.g., two threads incrementing a counter simultaneously).
	* Deadlock: 
	* 	A situation where two or more threads are blocked forever, each waiting for a lock held by the other.

Senior-Level
	* What is happens-before?
	* Difference between synchronized and ReentrantLock
	* How does ConcurrentHashMap work internally?
	* What is thread starvation?
	* What is lock contention?
	* Happens-Before: A memory visibility guarantee in the Java Memory Model (JMM) ensuring that memory writes by one specific action are visible to another specific action.
	* synchronized vs. ReentrantLock: 
	* 	synchronized is built-in and easy to use (implicit unlocking). 
	* 	ReentrantLock offers advanced features like fairness, timed polls (tryLock), and 
	* 	interruptible locks, but requires manual unlocking in a finally block.
	* ConcurrentHashMap Internals: It uses lock stripping (segmentation) or CAS (Compare-And-Swap) operations to allow multiple threads to read and write concurrently without locking the entire map.
	* Thread Starvation: 
	* 	A situation where a thread is perpetually denied access to resources (CPU or locks) because other "higher priority" threads are constantly hogging them.
	* Lock Contention: 
	* 	Occurs when multiple threads attempt to acquire the same lock at the same time, forcing threads to wait and reducing system throughput.

 

 
Exception Handling

Exceptions Hierarchy
Object
└── Throwable
    ├── Error (Unchecked)
    │   └── VirtualMachineError, OutOfMemoryError, etc.
    └── Exception
        ├── RuntimeException (Unchecked)
        │   └── NullPointerException, ArithmeticException, etc.
        └── [Other Checked Exceptions]
            └── IOException, SQLException, etc.

Checked vs unchecked exceptions
	* 1. Checked Exceptions
	* 	exceptions compiler requires you to handle or declare. 
	* 	represent conditions that a reasonable application should try to catch and recover from (usually external factors).
	* 	Inheritance: 
	* 		extend Exception (but not RuntimeException).
	* 	Mechanism: 
	* 		use a try-catch block or 
	* 		declare in method signature using throws keyword.
	* 	Examples: 
	* 		IOException, SQLException, ClassNotFoundException.
	* 2. Unchecked Exceptions
	* 	not checked at compile-time. 
	* 	represent programming errors (bugs) or 
	* 	improper use of an API.
	* 	Inheritance: 
	* 		extend RuntimeException.
	* 	Mechanism: 
	* 		compiler does not force you to handle them. 
	* 		developer's responsibility to write better code to avoid them.
	* 	Examples: 
	* 		NullPointerException, ArrayIndexOutOfBoundsException, IllegalArgumentException.
	* The "Rule of Thumb"
	* 	Checked exceptions for errors that the caller can realistically recover 
	* 	Unchecked exceptions for errors that indicate a bug in the code 

Difference between throw and throws
	* 1. throw (The Action)
	* 	Purpose: Used to explicitly trigger an exception instance.
	* 	Usage: It is used inside a method body.
	* 	Syntax: Followed by an instance of a Throwable class (usually using the new keyword).
	* 	Behavior: When the program reaches a throw statement, the execution of the current block stops, and the JVM looks for a matching catch block.
	* 2. throws (The Declaration)
	* 	Purpose: Used to declare that a method might throw one or more exceptions during its execution.
	* 	Usage: It is part of the method signature.
	* 	Syntax: Followed by the class names of the exceptions, separated by commas.
	* 	Behavior: It acts as a warning to the caller of the method: "You must handle these exceptions (using try-catch) or declare them yourself."

What happens if an exception is not caught?
	* 1. Stack Propagation (Unwinding)
	* 	JVM looks at the current method for a try-catch block. 
	* 	doesn't find one, it terminates that method and moves "up" to the caller method. 
	* 	This process repeats until:
	* 		matching catch block is found.
	* 		Or it reaches the "top" of the call stack (the main method or the run method of a thread).
	* 2. Thread Termination
	* 	reaches the top of the stack without being caught, the thread that threw the exception terminates abruptly.
	* 	single-threaded application, 
	* 		entire program crashes.
	* 	multi-threaded application, 
	* 		only specific thread that encountered the error dies. 
	* 		Other threads continue running.
	* 3. Default Exception Handler
	* 	Before the thread dies, JVM invokes the UncaughtExceptionHandler. 
	* 	By default, does two things:
	* 		Prints the Stack Trace: 
	* 			outputs the exception name, the message, and the sequence of method calls to System.err.
	* 		Exits with an Error Code: 
	* 			If was the main thread, the JVM usually exits with a non-zero status code (indicating a failure).
	* 		Critical Side Effect: finally blocks
	* 			any finally blocks encountered during the "stack unwinding" process execute before thread terminates. 
	* 			ensures that resources like file streams or db connections close even during a crash.

What is try-with-resources?
	* Try-with-resources is a feature introduced in Java 7 
	* 	automates the closing of resources (like files, database connections, or network sockets) once they are no longer needed.
	* 1. How it works
	* 	class that implements the AutoCloseable or Closeable interface can be used. 
	* 	declare the resource inside parentheses right after the try keyword. 
	* 	Java guarantees that the resource will 
	* 		closed automatically as soon as the block finishes,
	* 		whether an exception occurs or not.
	* 2. The "Before vs. After" Comparison
	* 	The Old Way (Verbose & Error-prone):
	* 		manually close resources in a finally block, 
	* 		often led to nested try-catches and potential NullPointerExceptions.
java
Scanner scanner = null;
try {
    scanner = new Scanner(new File("test.txt"));
    // Use scanner
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close(); // Mandatory manual closing
    }
}
	* 	The Modern Way (Try-with-resources):
	* 		code is cleaner, safer, and shorter.
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    // Use scanner
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
// scanner.close() is called automatically here by the JVM!
	* 3. Key Benefits
	* 	No more finally blocks: 
	* 		eliminates boilerplate code for closing streams.
	* 	Multiple Resources: 
	* 		can manage multiple resources by separating them with a semicolon: 
	* 			try (Res1 r1 = new Res1(); Res2 r2 = new Res2()) { ... }.
	* 	Suppressed Exceptions: 
	* 		both the try block and the close() method throw an exception, 
	* 			Java keeps the original exception as the primary one and 
	* 			attaches the closing error as a "suppressed exception." 		
	* 		modern Stack Trace, 
	* 			section at the bottom labeled Suppressed: .... 
	* 			tells you exactly what went wrong during the cleanup phase without hiding the actual crash 

How does exception propagation work?
	* exception occurs in a method, the JVM searches for a handler (a catch block). 
	* it doesn't find one, it pops the current method off the Call Stack and looks in the caller method. 
	* "bubbling up" process is called Exception Propagation. 
	* The Call Stack Process
	* 	Throwing: 
	* 		exception is thrown in a low-level method (e.g., methodC()).
	* 	Unwinding: 
	* 		methodC has no catch, it terminates immediately. 
	* 		JVM moves up to methodB() (the caller).
	* 	Searching: 
	* 		JVM checks if methodB has a try-catch. 
	* 		not, it moves up to methodA().
	* 	Termination: 
	* 		reaches the main() method and is still not caught, the thread dies, and the JVM prints the Stack Trace. 
	* Checked vs. Unchecked Propagation
	* 	Unchecked (RuntimeException): 
	* 		Propagates automatically. 
	* 		don't need to declare anything.
	* 	Checked (Exception): 
	* 		not propagate automatically. 
	* 		must explicitly use the throws keyword in the method signature to allow it to move to the caller.

Why are checked exceptions controversial?
	* 1. The "Boilerplate" Problem
	* 	"Catch-or-Declare" noise. 
	* 		low-level method throws a checked exception, every single method in the call stack above it must either catch it or add a throws clause. 
	* 		litters the code with technical details that don't belong in high-level business logic. 
	* 2. Exception Swallowing (The Empty Catch)
	* 	developers are forced to handle them, they often take the path of least resistance to make the compiler happy:
	* 			try {
	* 				readFile();
	* 			} catch (IOException e) {
	* 				// "I'll fix this later" (but they never do)
	* 			}
	* 	swallows the error, making bugs nearly impossible to track because the 
	* 	program continues in an invalid state without any log. [1, 2]
	* 3. Brittleness and Encapsulation
	* 	Checked exceptions break encapsulation. 
	* 	change a data-layer implementation from a File to a Database, 
	* 	method signature changes from throws IOException to throws SQLException. 
	* 	forces to change every method that calls it, all the way up the stack, violating the Open-Closed Principle.
	* 4. Conflict with Functional Programming
	* 	Streams and Lambdas do not play well with checked exceptions. 
	* 	Standard functional interfaces (like Consumer or Function) do not declare exceptions. 
	* 	If use a method that throws a checked exception inside a stream, have to wrap it in a clunky try-catch inside the lambda, which ruins readability.
	* 5. Shift Toward RuntimeExceptions
	* 	modern Java frameworks (like Spring and Hibernate) have moved almost entirely to RuntimeException. 
	* 	argue that if a database connection fails, there is usually nothing the code can do to recover right then and there, 
	* 	so it's better to let a global handler catch it and return a 500 error.

Finally block when return is encountered 
	* Java finally block when return statement is encountered
	* Will finally would execute even if there is a return statement in try block as well as in catch block?

try {
//try block
...
return success;
}
catch (Exception ex) {
//catch block
.....
return failure;
}
finally {
System.out.println("Inside finally");
return this_prevails;
}

	* The answer is yes. finally block will execute. 
	* 	only case where it will not execute is when it encounters System.exit().

	* Does finally block Override the values returned by try-catch block?
	* 	Yes. 
	* 	Finally block overrides the value returned by try and catch blocks 
Strings 
String immutability

Regular Expressions 
In Java, regular expressions are supported through the java.util.regex package, which mainly consists of the following classes:
	* Pattern: Defines the regular expression.
	* Matcher: Used to perform operations such as matching, searching and replacing.
	* PatternSyntaxException: Indicates a syntax error in the regular expression.



 
Functional Programming (Java 8+)
Stream API basics
	* A Stream is not a data structure; it just takes input from Collections, Arrays or I/O channels.
	* Streams do not modify the original data; they only produce results using their methods.
	* Intermediate operations (like filter, map, etc.) are lazy and return another Stream, so you can chain them together.
	* A terminal operation (like collect, forEach, count) ends the stream and gives the final result.
Comparable vs Comparator
	* 1. Comparable (Internal)
		* The sorting logic is defined inside the class of the objects being sorted.
		* Method: compareTo(T o)
		* Package: java.lang
		* Usage: Defines the "Natural Ordering" of the class (e.g., Strings are sorted alphabetically, Integers numerically).
		* Limitation: You can only have one natural ordering. If you want to change it, you must modify the class code.
	* 2. Comparator (External)
		* The sorting logic is defined in a separate class (or via a Lambda expression).
		* Method: compare(T o1, T o2)
		* Package: java.util
		* Usage: Defines multiple/custom sorting strategies (e.g., sorting Employees by Salary, then by Name, then by Age).
		* Benefit: You don't need to modify the original class. You can have as many Comparators as you want.
Java Lambda Expressions
	* Lambda expressions implement a functional interface (An interface with only one abstract function)
		* Enable passing code as data (method arguments).
		* Allow defining behavior without creating separate classes.

	* Functional interface
		* has exactly one abstract method. 
		* Lambda expressions provide its implementation. 
		* @FunctionalInterface annotation is optional but recommended to enforce this rule at compile time.
	* Why Use Lambda Expressions?
		* Concise Code: Reduce boilerplate compared to anonymous classes.
		* Functional Programming: Treat functions as first-class citizens.
		* Improved Readability: Code is easier to read and maintain.
		* Enhanced Collections and Streams: Simplify operations like filtering, mapping, and iterating.
	* Common Built-in Functional Interfaces
		* Predicate	 boolean test(T t)	
		* 	Tests a given condition and returns true or false.
		* Consumer void accept(T t)	
		* 	Performs an action on the given argument without returning a result.
		* Supplier	T get()	
		* 	Supplies or generates a result without taking any input.
		* Comparator<T>	int compare(T o1, T o2)	Compares two objects to determine their order.
		* Comparable<T>	int compareTo(T o)	Defines the natural ordering for objects of a class.

    public static void fizzBuzz(int n) {
    // Write your code here
        StringBuilder sb = new StringBuilder();
        Consumer<Integer> print = k -> {
            boolean div3= k%3==0;
            boolean div5= k%5==0;
            sb.setLength(0);    
            if (div3)
                sb.append("Fizz");
            if (div5)
                sb.append("Buzz");
            if (!div3&&!div5)
                sb.append(k);
            System.out.println(sb);
        };
        Stream.iterate(1, i->i+1).limit(n)
        .forEach(print);
    }


Method References
	* shorthand way to refer to an existing method without invoking it
	* introduced in Java 8 to make lambda expressions shorter, cleaner, and more readable
	* use the double colon (::) operator and are mainly used with functional interfaces.
	* Reference to a Static Method
		* ClassName::staticMethodName
		* list.forEach(MathUtil::square);
	* Reference to an Instance Method of a Particular Object
		* objectReference::instanceMethod
		*      Printer printer = new Printer();
		*         List<String> data = Arrays.asList("Java", "Spring", "Boot");
		*         data.forEach(printer::print);
	* Reference to an Instance Method of an Arbitrary Object
		* ClassName::instanceMethod
		*         names.stream()
		*              .map(String::toUpperCase)
		*              forEach(System.out::println);
	* Reference to a Constructor
		* ClassName::new
		*   Supplier<Student> supplier = Student::new;
		*   supplier.get();

	* Method references work only with functional interfaces, which contain exactly one abstract method.
		* Method signature must match abstract method
		* Common functional interfaces include Consumer, Supplier, Function, Predicate
		* Used heavily with Streams and Collections
Java Comparable vs Comparator
	* In Java, both Comparable and Comparator interfaces are used for sorting objects. The main difference between Comparable and Comparator is:
	* Comparable: It is used to define the natural ordering of the objects within the class.
	* Comparator: It is used to define custom sorting logic externally.

Why Lambdas only access the Heap
	* Lambdas are implemented as objects behind the scenes. 
	* In Java, objects live on the Heap, while local variables live on the Stack.
	* Stack Volatility: 
		* The Stack is short-lived. 
		* When a method returns, its Stack frame is popped and all local variables are destroyed. 
		* If a lambda (which can be passed around and executed later) 
		* tried to reference a Stack variable, it would point to "garbage" memory once the method ends.
	* Heap Persistence: 
		* The Heap is managed by the Garbage Collector. 
		* By capturing a copy of the local variable (or a reference to an object), 
		* the lambda stores that data on the Heap. This ensures the data remains 
		* available for the lambda's entire lifecycle, regardless of whether the original method is still running.
	* The this Context: 
		* When a lambda accesses an instance variable, it is actually capturing the this reference. 
		* Since this is an object on the Heap, the lambda can access it freely.

What is a functional interface?

Java Streams
Features of Streams
	* Declarative: 
	* 	Write concise and readable code using functional style.
	* Lazy Evaluation: 
	* 	Operations are executed only when needed (terminal operation).
	* Parallel Execution: 
	* 	Supports parallel streams to leverage multi-core processors.
	* Reusable Operations: 
	* 	Supports chaining of operations like map(), filter(), sorted().
	* No Storage: 
	* 	Streams don’t store data; they only process it.
How does Stream Work Internally?
	* Create a Stream: From collections, arrays or static methods.
	* Apply Intermediate Operations: Transform data (e.g., filter(), map(), sorted()).
	* Apply Terminal Operation: Produce a result (e.g., forEach(), collect(), reduce()).
Streams Creation:
	* From a Collection: Create a stream directly from a List, Set or any Collection using stream()
	* From an Array: Use Arrays.stream(array) to convert an array into a stream.
	* Using Stream.of(): Create a stream from a fixed set of values using Stream.of().
	* Infinite Stream: Generate an unbounded sequence using Stream.iterate() or Stream.generate()

        // 1. From a Collection
        List<String> list = Arrays.asList("Java", "Python", "C++");
        Stream<String> stream1 = list.stream();
        // 2. From an Array
        String[] arr = {"A", "B", "C"};
        Stream<String> stream2 = Arrays.stream(arr);
        // 3. Using Stream.of()
        Stream<Integer> stream3 = Stream.of(1, 2, 3, 4, 5);
        // 4. Infinite Stream (limit to avoid infinite loop)
        Stream<Integer> stream4 = Stream.iterate(1, n -> n + 1).limit(5);
        stream4.forEach(System.out::println);

Stream.iterate(0, n -> n < 10, n -> n + 2)


Stream Pipeline
Source
	* source provides the data for the stream. 
	* It can be a collection, array, file or even an infinite generator.
	* 
List<Integer> numbers = Arrays.asList(10, 20, 30, 40);
Stream<Integer> stream = numbers.stream(); // Source
Stream<Integer> stream = IntStream.range(2, 8)
	* iterate(seed, predicate, next)
	* 	nova sobrecarga do método iterate que funciona semelhante a um ciclo for tradicional, 
	* 	permitindo definir uma condição de paragem.
	* 	Parâmetros: (valor inicial, condição para continuar, como gerar o próximo).
	* Stream.iterate(0, n -> n < 10, n -> n + 2)
	*       .forEach(System.out::print);
// Saída: 02468
Intermediate Operations
	* Intermediate operations transform a stream into another stream. Some common intermediate operations include:
		* Skip(): Skip first n elements.
		* 
		* map(): 
	return a stream consisting of the results of applying the given function to the elements of this stream.
	<R> Stream<R> map(Function<? super T, ? extends R> mapper)
		* filter():
	used to select elements as per the Predicate passed as an argument.
	Stream<T> filter(Predicate<? super T> predicate)
		* sorted(): 
	used to sort the stream.
	Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator)
		* flatMap(): 
	to flatten a stream of collections into a single stream of elements.
		* distinct(): 
	Removes duplicate elements. 
	returns a stream consisting of the distinct elements (according to Object.equals(Object)).
	Stream<T> distinct()
		* peek(): 
	Performs an action on each element without modifying the stream. 
	returns a stream consisting of the elements of this stream, additionally performing the provided action on each element 
	Stream<T> peek(Consumer<? super T> action)
        List<Integer> numbers = Arrays.asList(5, 10, 20, 10, 30, 40);
        numbers.stream()
               .filter(n -> n > 10)   // keep > 10
               .map(n -> n * 2)       // double them
               .distinct()            // remove duplicates
               .sorted()              // sort ascending
               .forEach(System.out::println);



Terminal Operations
	* Terminal Operations are the operations that on execution return a final result as an absolute value.
		* ForEach(): It iterates all the elements in a stream.
		* collect(Collectors.toList()): It collects stream elements into a list (or other collections like set/map).
	Mutability: 
	* Collectors.toList() returns a mutable list (e.g., ArrayList). 
	* Stream.toList() returns an unmodifiable/immutable list.
	Conciseness: stream.toList() is cleaner and more direct.
	Performance: stream.toList() can be more efficient, as it can be optimized to know the final size, unlike Collectors.toList() which relies on ArrayList resizing.
	Compatibility: Collectors.toList() works on Java 8+. stream.toList() requires Java 16 or later. 
		* reduce(): It reduces stream elements into a single aggregated result.
		* count(): It returns the total number of elements in a stream.
		* anyMatch() / allMatch() / noneMatch(): They check whether elements match a given condition.
		* findFirst() / findAny(): They return the first or any element from a stream.
		* collect(): 
	return the result of the intermediate operations performed on the stream.
	<R, A> R collect(Collector<? super T, A, R> collector)
		* forEach(): 
	used to iterate through every element of the stream.
	void forEach(Consumer<? super T> action)
		* reduce(): 
	used to reduce the elements of a stream to a single value. 
	takes a BinaryOperator as a parameter.
	T reduce(T identity, BinaryOperator<T> accumulator)
Optional<T> reduce(BinaryOperator<T> accumulator)
		* count(): 
	Returns the count of elements in the stream.
	long count()
		* findFirst(): 
	the first element of the stream, if present.
	Optional<T> findFirst()
		* allMatch():
	 Checks if all elements of the stream match a given predicate.
	boolean allMatch(Predicate<? super T> predicate)
		* anyMatch(): 
	Checks if any element of the stream matches a given predicate.
	boolean anyMatch(Predicate<? super T> predicate)
		* 
        List<String> names = Arrays.asList("Amit", "Riya", "Rohan", "Amit");

        // Collect into Set (removes duplicates)
        Set<String> uniqueNames = names.stream().collect(Collectors.toSet());
        System.out.println(uniqueNames);

        // Count names starting with 'R'
        long count = names.stream().filter(n -> n.startsWith("R")).count();
        System.out.println("Names starting with R: " + count);

        // Reduce (concatenate names)
        String result = names.stream().reduce("", (a, b) -> a + b + " ");
        System.out.println(result);

		* takeWhile(Predicate p)
		* 	Extrai elementos do Stream enquanto a condição for verdadeira. 
		* 	Assim que encontrar o primeiro elemento que não satisfaz o predicado, o Stream é interrompido.
	List<Integer> numeros = List.of(1, 2, 10, 3, 4);
	numeros.stream()
		   .takeWhile(n -> n < 5)
		   .forEach(System.out::print); 
	// Saída: 12 (Para no 10, ignorando o 3 e 4 que vêm depois)

		* dropWhile(Predicate p)
		* 	oposto do takeWhile. 
		* 	descarta os elementos enquanto a condição for verdadeira e, 
		* 	assim que encontrar o primeiro elemento que falha, mantém todos os restantes elementos do Stream.
	List<Integer> numeros = List.of(1, 2, 10, 3, 4);
	numeros.stream()
		   .dropWhile(n -> n < 5)
		   .forEach(System.out::print);
	// Saída: 1034 (Descarta 1 e 2, mantém o resto a partir do 10)


Types of Streams
Sequential Stream
	* Processes elements one by one in a single thread.
	* Created by default when you call stream().
        List<String> names = Arrays.asList("A", "B", "C", "D");
        names.stream()
             .forEach(System.out::println); // Executes sequentially
Parallel Streams
	* can perform operations concurrently on multiple threads. 
	* meant to make use of multiple processors or cores available to speed the processing. 
	* two methods:
		* Using the parallel() method on a stream
		* Using parallelStream() on a Collection
        List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7,8,9);
        numbers.parallelStream().forEach(n -> System.out.println(n + " " + Thread.currentThread().getName()));
Infinite Streams
	* Streams can also generate unbounded sequences. 
	* Use limit() to avoid infinite execution.
        Stream.iterate(1, n -> n + 1)
              .limit(5)
              .forEach(System.out::println);


Primitive Streams
	* Java provides specialized streams for primitive data types:
		* IntStream -> for int values
		* LongStream -> for long values
		* DoubleStream -> for double values
        IntStream.range(1, 5).forEach(System.out::println);


File Read Operation
        // Step 1: Create a Stream of lines from the file
        try (Stream<String> lines = Files.lines(Paths.get(fileName))) {

File Write Operation

        String[] words
            = { "Geeks", "for", "Geeks", "Hello", "World" };

        // Replace with the actual file path

        String fileName = "path/to/your/file.txt";

        // Step 1: Create a PrintWriter to write to the file
        try (PrintWriter pw
             = new PrintWriter(Files.newBufferedWriter(
                 Paths.get(fileName)))) {

            // Step 2: Use Stream to write each word to the file
            Stream.of(words).forEach(pw::println);


Examples

        List<Transaction> transactions = Arrays.asList(
            new Transaction(1, 100, "GROCERY"),
            new Transaction(3, 80, "GROCERY"),
            new Transaction(6, 120, "GROCERY"),
            new Transaction(7, 40, "ELECTRONICS"),
            new Transaction(10, 50, "GROCERY")
        );

        // Stream pipeline based on your diagram
        List<Integer> transactionIds = transactions.stream()
                .filter(t -> t.getType().equals("GROCERY"))       // keep only groceries
                .sorted(Comparator.comparing(Transaction::getValue).reversed()) // sort by value desc
                .map(Transaction::getId)                         // map to id
                .collect(Collectors.toList());

What is Java Parallel Streams?
	* we can divide the code into multiple streams that are executed in parallel on separate cores 
	* the final result is the combination of the individual outcomes. 
	* order of execution, however, is not under our control.
Using the parallel() Method
	* used on an existing sequential stream to convert it into a parallel stream. 
	* return a parallel stream, which can then be processed using various stream operations.
Using the parallelStream() Method
	* to create a parallelstream from a Collection. 
	* can directly call parallelStream() on Collections (such as List, set) to obtain parallel stream.

Stream.reduce() in Java with examples
	*  used to perform a reduction on the elements of a stream using an associative accumulation function 
	* returns an Optional
	* commonly used to aggregate or combine elements into a single result, such as computing the maximum, minimum, sum, or product.

Get the Longest String
        Optional<String> longestString = words.stream()
            .reduce((word1, word2) -> word1.length() > word2.length() ? word1 : word2);

        // Displaying the longest String
        longestString.ifPresent(System.out::println); 

    Combine Strings
        // Using reduce to concatenate strings with a hyphen
        Optional<String> combinedString = Arrays.stream(array)
            .reduce((str1, str2) -> str1 + "-" + str2);

        // Displaying the combined String
        combinedString.ifPresent(System.out::println);
 Sum of All Elements
       // Using reduce to find the sum of all elements
        int sum = numbers.stream()
            .reduce(0, (element1, element2) -> element1 + element2);

        // Displaying the sum of all elements
        System.out.println("The sum of all elements is " + sum);

Product of All Numbers in a Range
        // Calculating the product of all numbers in the range [2, 8)
        int product = IntStream.range(2, 8)
            .reduce((num1, num2) -> num1 * num2)
            .orElse(-1); // Provides -1 if the stream is empty

        // Displaying the product
        System.out.println("The product is : " + product);

	* In Java Streams, the logic is identical: map transforms one element into another, while flatMap transforms one element into a stream of elements and then merges (flattens) them all into a single stream.
	* The Scenario: Lists of Lists
	* Imagine you have a list of Departments, and each department has a list of Employees. You want a single list of all employee names in the company.
	* 
	* List<Department> departments = Arrays.asList(
	*     new Department("IT", Arrays.asList("Alice", "Bob")),
	*     new Department("HR", Arrays.asList("Charlie", "David"))
	* );
	* 
	* 1. Using map() (The "Nested" Problem)
	* If you use map, you get a stream where each element is a List.
	* 
	* var result = departments.stream()
	*     .map(dept -> dept.getEmployees()) // Returns Stream<List<String>>
	*     .collect(Collectors.toList());
	* // Result: [[Alice, Bob], [Charlie, David]] // (A list of lists - not what we usually want)
	* 
	* 2. Using flatMap() (The "Flattened" Solution)
	* flatMap expects a function that returns a Stream. it then takes all those individual streams and joins them into one.
	* 
	* List<String> allEmployees = departments.stream()
	*     .flatMap(dept -> dept.getEmployees().stream()) // Flattens List<String> into the main stream
	*     .collect(Collectors.toList());
	* // Result: [Alice, Bob, Charlie, David] // (A single, flat list of all names)
	* 
	* Key Comparison
	* 
	* | Feature | map() | flatMap() |
	* |---|---|---|
	* | Function returns | A single value (T) | A Stream<T> |
	* | Output | Stream<T> | Stream<T> |
	* | Visual Analogy | 1 input $\rightarrow$ 1 output | 1 input $\rightarrow$ Many outputs (merged) |
	* | Common Use | Changing data format (e.g., String to Integer) | Dealing with nested collections or "one-to-many" relations |

Stream Pitfalls
	* While Java Streams are powerful and elegant, they introduce several performance trade-offs compared to traditional for loops. Common pitfalls include: 
	* Excessive Boxing and Unboxing: 
	* 	Every time a primitive (like int) is used in a regular Stream<Integer>, it must be "boxed" into an object, and later "unboxed" back to a primitive. 
	* 	This "conversion tax" can be devastating in tight loops.
	* 	Fix: Use specialized primitive streams like IntStream, LongStream, or DoubleStream.
	* Parallel Stream Overhead: 
	* 	Developers often assume parallelStream() is always faster. However, splitting data, managing the common Fork/Join pool, and merging results adds significant overhead. For small datasets (typically < 10,000 elements) or non-CPU-intensive tasks, parallel streams are often slower than sequential ones.
	* Inefficient Operation Order: 
	* 	The Stream API does not automatically reorder your operations for efficiency.
	* 	Pitfall: Performing an expensive map() before a filter().
	* 	Fix: Place filter() and limit() operations as early as possible to reduce the workload for subsequent stages.
	* Memory and Garbage Collection Pressure:
	* 	 Streams create a pipeline of objects (Spliterators, lambda objects, intermediate results). This increases memory consumption and Garbage Collection (GC) pauses, which can freeze high-throughput applications.
	* Misusing Terminal Operations:
	* 	Overusing collect(): Creating intermediate lists just to iterate over them (e.g., .collect(Collectors.toList()).forEach(...)) is slower and more memory-intensive than using forEach() directly on the stream.
	* 	Using count() for existence checks: Running stream.filter(...).count() > 0 is less efficient than using anyMatch(...), which can stop processing as soon as a match is found.
	* Abuse in "Hot Paths": 
	* 	If code runs thousands of times per second (like in a high-traffic API), the 2–3x performance penalty of a stream versus a for loop can lead to significant latency. 
What is a Stream?
Intermediate vs terminal operations
Are streams lazy?
Parallel streams — when to use / avoid?

 
1. Streams API

Core Concept
The Streams API enables declarative, functional-style operations on collections.

List<Integer> result = list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .toList();

Important Concepts
	* Intermediate operations (lazy)
	* Terminal operations (trigger execution)
	* Pipelines
	* Functional interfaces

Types of operations:
map
filter
flatMap
sorted
distinct
reduce
collect

Common Pitfalls
1. Streams are single-use
Stream s = list.stream();
s.count();
s.findFirst(); // error

2. Side effects inside streams
Bad practice:
list.stream().forEach(x -> externalList.add(x));

3. Overusing streams for complex logic
Sometimes loops are more readable.

Optionals

API
	* 1. Checking Presence
	* * isPresent(): 
	* 	Returns true if a value is present, otherwise false. 
	* 	often used as a direct replacement for if (value != null). 
	* 
	* 2. Retrieving with Fallbacks (Defaults)
	* * orElse(T other): 
	* 	Returns the value if present; 
	* 	otherwise, it returns the provided default value (other).
	* 	* Note: 
	* 		default value is always evaluated, even if the Optional contains a value. 
	* 		This can be a performance hit if the default involves a method call.
	* * orElseGet(Supplier<? extends T> supplier): 
	* 	Returns the value if present; 
	* 	otherwise, it invokes the supplier and returns its result.
	* 	* Best Practice: 
	* 		Use lazy evaluation when default value is expensive to compute (e.g., a database call), as
	* 		 it only runs if the Optional is empty.
	* * orElseThrow(): 
	* 	Returns the value if present; 
	* 	otherwise, it throws a NoSuchElementException (or a custom exception if a supplier is provided). 
	* 
	* 3. Transforming Values
	* * map(Function<? super T, ? extends U> mapper): 
	* 	If a value is present, it applies the function to it and wraps the result in a new Optional.
	* * 	If the original Optional is empty, it simply returns Optional.empty().
	* * flatMap(Function<? super T, Optional<U>> mapper): 
	* 	Similar to map, but used when the mapping function already returns an Optional.
	*   	Instead of creating a nested Optional<Optional<T>>, it 
	* 	"flattens" the result into a single Optional<T>. 
	* 
	* | Method [7, 11, 13, 15, 18] | Behavior if Present | Behavior if Empty | Key Difference |
	* | orElse | Returns value | Returns default | Evaluates default immediately. |
	* | orElseGet | Returns value | Calls Supplier | Evaluates default lazily. |
	* | map | Transforms value | Returns empty() | Wraps result in Optional. |
	* | flatMap | Transforms & Flattens | Returns empty() | Prevents nested Optionals. |
	* 
	* Would you like to see a code example demonstrating when map would fail but flatMap would work?
	* 
	* The main reason you would choose flatMap over map is to avoid "Nested Optionals" (Optional<Optional<T>>). 
	* This happens when the transformation function you are using already returns an Optional. [
	* The Problem: Using map with nested data
	* Imagine a Person who has an Address, and that Address might have an ApartmentNumber. In modern Java, both would be represented as Optional.
	* 
class Person {
    Optional<Address> getAddress() { ... }
}
class Address {
    Optional<String> getApartmentNumber() { ... }
}
	* 
	* If you use map to try and get the apartment number from a Person, the result becomes a mess:
	* 
Optional<Person> person = Optional.of(new Person());
// map() wraps the result in ANOTHER Optional
Optional<Optional<String>> nested = person
    .map(Person::getAddress) // Returns Optional<Address>
    .map(Address::getApartmentNumber); // map wraps it into Optional<Optional<String>>
	* 
	* Because the result is nested, you cannot easily call further Optional methods (like orElse) on the actual String value. [3, 4] 
	* The Solution: Using flatMap
flatMap "flattens" the structure by merging the two Optional layers into one. [5] 

Optional<String> apartment = person
    .flatMap(Person::getAddress)      // Returns Optional<Address> (unwrapped)
    .flatMap(Address::getApartmentNumber); // Returns Optional<String> (unwrapped)

String result = apartment.orElse("No Apartment");
	* 
	* Summary Comparison
	* 
	* | Method [2, 5, 6] | Transformation Function Returns | Resulting Type | Best Use Case |
	* | map | A plain object (e.g., String) | Optional<String> | Simple transformations (1-to-1). |
	* | flatMap | An Optional (e.g., Optional<String>) | Optional<String> | Chaining methods that return Optionals. |
	* 
	* 
	* 
	* In a Spring Boot application using Spring Data JPA, you often deal with Optional when fetching data by ID.
	* Here is a common scenario: you want to find a User, then find their Profile, and finally get a specific Setting. If any step fails (user not found, profile missing), you want a safe default.
	* The Problem: Nested Optionals
	* If UserRepository.findById returns an Optional<User> and 
	* user.getProfile() also returns an Optional<Profile>, 
	* using map creates a "wrapper inside a wrapper."
	* 
	* The Solution: Using flatMap for Clean Chaining
	* 
@Servicepublic class UserService {

    @Autowired
    private UserRepository userRepository;

    public String getUserTheme(Long userId) {
        return userRepository.findById(userId)           // Returns Optional<User>
            .flatMap(User::getProfile)                  // Returns Optional<Profile> (flattened)
            .map(Profile::getTheme)                     // Returns Optional<String>
            .orElse("LIGHT_MODE");                      // Default if any step was empty
    }
}
	* 
	* Why this is powerful in Spring Boot:
	* 
	*    1. No NullPointerExceptions: Even if the database returns nothing for the ID, or the user has no profile, the code won't crash.
	*    2. No Nested if blocks: Without flatMap, you would need 3 layers of if (x.isPresent()) or null checks.
	*    3. Readability: It reads like a story: "Find user, get their profile, take the theme, or use light mode."
	* 
	* Bonus: flatMap with orElseThrow
	* In a REST Controller, you might want to throw a 404 error if the chain fails:
	* 
@GetMapping("/{id}/theme")
public String getTheme(@PathVariable Long id) {
    return userRepository.findById(id)
        .flatMap(User::getProfile)
        .map(Profile::getTheme)
        .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Profile not found"));
}


Important Methods
isPresent()
orElse()
orElseGet()
orElseThrow()
map()
flatMap()
Example:
String name = user.map(User::getName)
                  .orElse("Unknown");

Common Misuses
❌ Optional as fields
class User {
   Optional<String> name; // bad
}
❌ Optional parameters
❌ Optional in collections
Correct use:
method return types


 
Core Java fundamentals questions

 
________________________________________
________________________________________














________________________________________
6️⃣ Strings & Interning
Very Common
	* Why is String immutable?
		* String Pool (Memory Efficiency): 
		* 	special memory region called the String Constant Pool. 
		* 	multiple variables have the same value (e.g., "Hello"), 
		* 	all point to the same object in memory. 
		* 	were mutable, changing the value for one variable would "break" it for all others sharing that reference.
		* Security: 
		* 	Strings are heavily used for sensitive data like database URLs, usernames, passwords, and file paths. 
		* Thread Safety: 
		* 	Strings are inherently thread-safe. 
		* 	Multiple threads can share the same String instance without the need for synchronization, avoiding race conditions and deadlocks.
		* Caching HashCodes: 
		* 	the hashCode() of a String is calculated once and then cached. 
		* 	makes Strings incredibly fast when used as keys in a HashMap or HashSet.
	* What is the string pool?
		* String Pool (or String Constant Pool) 
		* 	special memory region within the Java Heap that stores a single copy of every unique String literal.
		* 1. How it works
		* 	create a String using double quotes (a literal), the JVM checks the pool first:
		* 		If the String exists: It returns a reference to the existing object.
		* 		If it doesn't exist: It creates a new String in the pool and returns that reference.
		* 2. Literal vs. new String()
		* 	The way you declare a String changes its memory location:
		* 		Literal (String s = "Java";): Goes into the String Pool. 
		* 			If another variable also uses "Java", 
		* 			they both point to the exact same memory address.
		* 		Constructor (String s = new String("Java");): 
		* 			Forces the JVM to create a new object in the normal Heap area, 
		* 			even if "Java" already exists in the pool. 
		* 			This is generally considered a waste of memory.
	* Difference between String, StringBuilder, StringBuffer
		* Use String for constant values, database keys, or small concatenations. It is the safest and easiest to use but creates a new object every time you modify it (e.g., s + " more").
		* Use StringBuilder for heavy string manipulation, such as building a large text block inside a loop. It modifies the same memory buffer, avoiding thousands of object allocations.
		* Use StringBuffer only in legacy code or rare cases where multiple threads are modifying the same string builder instance simultaneously. Because its methods are synchronized, it has extra overhead.
		* 3. Performance Example (The "Loop Trap")
		* 	If you use String inside a loop, the JVM creates a new object in every iteration.
		* 
	* 
	* What does intern() do?
		* You can manually move a String from the Heap into the Pool by calling .intern(). If the pool already contains an equal string, it returns the pooled reference; otherwise, it adds the string to the pool.


8️⃣ Java Language Mechanics
Very Common
	* What is final used for?
	* What is the difference between abstract and interface?
	* Can an interface have methods with implementation?
	* What are default methods?
	* What is autoboxing?

9️⃣ Serialization
	* What is serialization?
1.	Serialization is the process of converting an object's state into a byte stream. 
2.	This byte stream can then be:
3.		Stored: Written to a file or a database.
4.		Transmitted: Sent over a network to another JVM (e.g., in microservices or RMI).
5.		reverse process—reconstructing a live Java object from that byte stream—is called Deserialization.
6.	How it works in Java
7.		make object serializable, must implement the java.io.Serializable interface. 
8.		a marker interface (it has no methods); 
9.		simply tells the JVM: "It is safe to serialize this object."
10.	
11.	public class User implements Serializable {
12.	    private static final long serialVersionUID = 1L; // Recommended: unique ID for versioning
13.	    private String name;
14.	    private transient String password; // 'transient' means this field is NOT serialized
15.	}
16.	
17.	Key Concepts
18.		Byte Stream: 
19.			platform-independent sequence of bytes representing the object's data and class metadata.
20.		serialVersionUID: 
21.			unique identifier used during deserialization to ensure the sender and receiver have compatible classes.
22.		transient Keyword: 
23.			sensitive or unnecessary fields (like passwords or temporary caches) so they are skipped during serialization.
24.	Why is it used?
25.		Persistence: Saving application state so it can be restored after a restart.
26.		Deep Copy: Creating a complete, independent copy of an object graph.
27.		Distributed Systems: Moving objects between different servers in a cluster.
	* What is serialVersionUID?
1.	serialVersionUID is a unique identifier (a static final long) 
2.		used during deserialization to verify that the sender and receiver loaded classes for that object that are compatible.
3.	How it works
4.		serialized, serialVersionUID is stamped into the byte stream. 
5.		later deserialized, the JVM compares the ID from the byte stream with the ID of the local class:
6.		If they match: 
7.			The object is created successfully.
8.		If they mismatch: 
9.			JVM throws an InvalidClassException, 
10.	Why you should define it manually
11.		If you don't define a serialVersionUID, 
12.			Java compiler will automatically generate one based on the class's fields, methods, and modifiers.
13.		The Danger: Even a tiny, invisible change to your code (like changing a method from private to protected) will cause the compiler to generate a different ID.
14.		The Result: You will be unable to read any data that was saved before that minor code change, even if the actual data fields are exactly the same.
	* Why is default Java serialization often discouraged?
1.	Java serialization (using Serializable) is widely discouraged 
2.	for three critical reasons:
3.		Security Vulnerabilities: 
4.			major vector for Remote Code Execution (RCE) attacks. 
5.			During "deserialization," the JVM creates objects before verifying their data. 
6.			Attackers can craft malicious byte streams ("gadget chains") 
7.			that execute arbitrary code the moment readObject() is called. 
8.		Brittleness and Maintenance: 
9.			tightly couples the byte stream to the internal private structure of the class. 
10.			If you rename a field, change a data type, or forget the serialVersionUID, 
11.			can no longer read previously saved data, leading to InvalidClassException.
12.		Performance Overhead: 
13.			significantly slower and produces much larger payloads than modern alternatives. 
14.			It carries extensive metadata about the class hierarchy, which wastes bandwidth and CPU cycles 
15.			compared to optimized formats.

🔟 Misc Frequently Asked
	* What is the difference between Comparable and Comparator?
	* What is Optional?
	* Why is Optional not meant for fields?
	* What is reflection?
	* What is class loading in Java?
 
 
20 ADVANCED JAVA QUESTIONS 
________________________________________
________________________________________
1️⃣2️⃣ Why is HashMap not thread safe?
Because concurrent writes can cause:
lost updates
infinite loops (pre Java 8)
data corruption
Use:
ConcurrentHashMap
________________________________________
1️⃣3️⃣ How does ConcurrentHashMap work?
Java 8 design:
CAS operations
synchronized on buckets when needed
lock striping removed
Reads are mostly lock-free.
________________________________________
1️⃣4️⃣ What is false sharing?
When two threads update variables in the same CPU cache line.
Result:
cache invalidations
huge performance drop
Mitigation:
padding
@Contended
________________________________________
1️⃣5️⃣ What causes memory leaks in Java?
Typical causes:
static collections
listeners not removed
ThreadLocal misuse
caches without eviction
classloader leaks
________________________________________
1️⃣6️⃣ What is the difference between stack and heap?
Stack	Heap
per thread	shared
method frames	objects
fast allocation	slower
auto cleanup	GC
________________________________________
1️⃣7️⃣ What is escape analysis?
JVM optimization that detects if an object does not escape a method.
Possible optimizations:
stack allocation
lock elimination
scalar replacement
________________________________________
1️⃣8️⃣ What is the difference between soft, weak, and phantom references?
Type	GC Behavior
SoftReference	cleared when memory low
WeakReference	cleared next GC
PhantomReference	used for cleanup tracking
Used in caches and resource management.
________________________________________
________________________________________
2️⃣0️⃣ What is the difference between CompletableFuture and Future?
Future:
blocking get()
no chaining
CompletableFuture:
async pipelines
non-blocking
callbacks
combination of tasks
Example:
CompletableFuture
  .supplyAsync(...)
  .thenApply(...)
  .thenCombine(...)
________________________________________
⭐ 5 SUPER HARD Questions (Principal Engineer Level)
These appear in very senior interviews.
1️⃣ Why can HashMap perform poorly with bad hash functions?
Too many collisions → linked lists or trees → O(n).
Java 8 uses red-black trees after 8 collisions.
________________________________________
2️⃣ What is lock striping?
Technique to reduce contention:
multiple locks protecting subsets of data
Used in early ConcurrentHashMap.
________________________________________
3️⃣ What is mechanical sympathy?
Designing software aligned with CPU architecture:
cache lines
memory locality
branch prediction
Important in high-performance systems.
________________________________________
4️⃣ What is the difference between wait() and sleep()?
wait	sleep
releases lock	keeps lock
used in synchronization	thread pause
________________________________________
5️⃣ Why can too many threads reduce performance?
Context switching overhead.
Modern servers prefer:
thread pools
reactive programming
async I/O
________________________________________
⭐ Interview Tip for Senior Candidates
Strong answers include:
	* why the feature exists
	* real production issues
	* performance implications
Example:
Bad answer:
volatile ensures visibility.
Good answer:
volatile ensures visibility and prevents reordering, but it doesn't guarantee atomicity, so operations like counter++ are still unsafe.



 
 
 
Generics
Type Erasure 
	* (apagamento de tipos)  o processo pelo qual o compilador do Java remove todas as informaes de parmetros de tipos genricos (<T>, <String>, etc.) durante a compilao.
	* objetivo principal quando os Generics foram introduzidos (Java 5) era garantir a retrocompatibilidade com cdigo Java mais antigo que no os utilizava [1, 2].
	* Como funciona na prtica?
	* compilador segue estas regras bsicas:

	* Substituio por Bounds: Se um tipo for "unbounded" (ex: <T>), o compilador substitui T por Object. Se for "bounded" (ex: <T extends Number>), substitui por Number [1, 2].
	* Insero de Casts: O compilador insere type casts automaticamente para garantir que o tipo correto seja retornado ao cdigo que chama o mtodo [1, 2].
	* Bridge Methods: O compilador pode criar mtodos auxiliares ("bridge methods") para manter o polimorfismo em tipos genricos estendidos [1].

	* Exemplo de Compilao
	* cdigo que escreve:

public class Box<T> {
private T data;
public void set(T data) { this.data = data; }
public T get() { return data; }
}

	* que o JVM v (aps o Type Erasure):

public class Box {
private Object data; // T virou Object
public void set(Object data) { this.data = data; }
public Object get() { return data; }
}

	* Consequencias Importantes

	* No existem Generics em Runtime: No tempo de execuo, a JVM no sabe se uma List foi declarada como List<String> ou List<Integer>. Para ela, so todas apenas List [2].
	* Restricoes: Devido ao erasure, 
	* 	no pode usar instanceof T, 
	* 	criar um new T(), ou 
	* 	criar arrays de tipos genricos (new T[10]) [1].
	* Sobrecarga: 
	* 	Nao pode ter dois metodos na mesma classe que se tornariam identicos apos o erasure (ex: 
	* 	doWork(List<String> l) e doWork(List<Integer> l) causam erro de compilao) [2].

PECS

The "Wildcard" Breakdown
? extends T (Upper Bound)
Logic: "I don't know the exact type, but it is T or a subclass of T."
Capability: You can READ from it as a T.
Restriction: You CANNOT WRITE to it (except null) because the compiler can't guarantee if the underlying list is a List<SubtypeA> or List<SubtypeB>.
Mnemonic: Producer Extends (The list produces items for your code).
? super T (Lower Bound)
Logic: "I don't know the exact type, but it is T or a superclass of T."
Capability: You can WRITE a T (or any subclass of T) into it safely.
Restriction: When you READ, you only get an Object because the list could be a List<Object>.
Mnemonic: Consumer Super (The list consumes items from your code).

In Spring Data JPA, the PECS principle is used to make repositories flexible. For example, when you define a method that takes a collection of IDs or criteria, Spring uses these wildcards so you can pass lists of sub-types.
1. Spring Data Repository Example
The findAllById method in the CrudRepository interface is defined like this:
java
Iterable<T> findAllById(Iterable<? extends ID> ids);
Use o código com cuidado.

Why ? extends ID? (The Producer)
Spring is the consumer of the list you provide, but the list itself is the producer of IDs. By using ? extends ID, Spring allows you to pass a List<Long> even if the entity ID is defined as a Number, or a List<Integer> if it expects a Number.
2. Custom Specification Example
If you are building a search filter using Specifications, you might see something like this:
java
public interface Specification<T> {
    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
}


 
Basics

Differences Between JDK, JRE and JVM

JDK (Java Development Kit)
	* contains the JRE and a set of development tools.
	* Includes compiler (javac), debugger, and utilities like jar and javadoc.
	* Provides the JRE, so it also allows running Java programs.
	* Required by developers to write, compile, and debug code.
Components of JDK:
	* JRE (JVM + libraries)
	* Development tools (compiler, jar, javadoc, debugger)
Note:
	* JDK is only for development (it is not needed for running Java programs)
	* JDK is platform-dependent (different version for windows, Linux, macOS)
JRE (Java Runtime Environment)
	* environment to run Java programs 
	* does not include development tools. 
Contains the JVM and standard class libraries.
	* Provides all runtime requirements for Java applications.
	* Does not support compilation or debugging.
	* It is platform-dependent (different builds for different OS).
Working of JRE:
1.	Class Loading: Loads compiled .class files into memory.
2.	Bytecode Verification: Ensures security and validity of bytecode.
3.	Execution: Uses the JVM (interpreter + JIT compiler) to execute instructions and make system calls.
JVM (Java Virtual Machine)
core execution engine of Java. 
responsible for converting bytecode into machine-specific instructions.
	* Performs memory management and garbage collection.
	* Provides portability by executing the same bytecode on different platforms.
Note:
	* JVM implementations are platform-dependent.
	* Bytecode is platform-independent and can run on any JVM.
	* Modern JVMs rely heavily on Just-In-Time (JIT) compilation for performance.

Wrapper Classes in Java
	* wrapper classes allow primitive data types to be represented as objects
	* Why Wrapper Classes Are Needed
	* Wrapper classes are required in Java for the following reasons:
	* Java collections (ArrayList, HashMap, etc.) store only objects, not primitives.
	* Wrapper objects allow primitives to be used in object-oriented features like methods, synchronization, and serialization.
	* Objects support null values, while primitives do not.
	* Wrapper classes provide utility methods such as compareTo(), equals(), and toString()
. Autoboxing
	* The automatic conversion of primitive types to the object of their corresponding wrapper classes is known as autoboxing.
char ch = 'a';

        // Autoboxing: char -> Character
        Character c = ch;

        ArrayList<Integer> list = new ArrayList<>();
        // Autoboxing: int -> Integer
        list.add(25);
2. Unboxing
	* Unboxing is the automatic conversion of a wrapper class object back into its corresponding primitive type
   Character ch = 'a';
        // Unboxing: Character -> char
        char c = ch;

        ArrayList<Integer> list = new ArrayList<>();
        list.add(24);
        // Unboxing: Integer -> int
        int num = list.get(0);
 
Typecasting in Java
Widening Type Casting (Implicit Casting)
	* A lower data type is transformed into a higher one by a process known as widening type casting. Implicit type casting and casting down are some names for it. It occurs naturally. Since there is no chance of data loss, it is secure. Widening Type casting occurs when:
	* The target type must be larger than the source type.
	* int i = 10;
	* 
	*         // Widening Type Casting (Automatic Casting) from int to long
	*         long l = i;
	* 
	*         // Widening Type Casting (Automatic Casting) from int to double
	*         double d = i;
2. Narrow Type Casting (Explicit Casting)
	* The process of downsizing a bigger data type into a smaller one is known as narrowing type casting. Casting up or explicit type casting are other names for it. It doesn't just happen by itself. If we don't explicitly do that, a compile-time error will occur. 
	* 
	*         // Narrowing Type Casting
	*         short j = (short)i;
	*         int k = (int)i;
	* 
 
	* Mainly there are two types of Explicit Casting:
	* Explicit Upcasting
	* Explicit Downcasting
	* 1. Explicit Upcasting 
	* Upcasting is the process of casting a subtype to a supertype in the inheritance tree's upward direction. When a sub-class object is referenced by a superclass reference variable, an automatic process is triggered without any further effort. 
	* { // Upcasting
	*         Animal animal = new Dog();
	* 
	* 2. Explicit Downcasting
	* When a subclass type refers to an object of the parent class, the process is referred to as downcasting. If it is done manually, the compiler issues a runtime ClassCastException error. It can only be done by using the instanceof operator. 
	*         // Explicit downcasting
	*         Cat cat = (Cat)animal;
 
Questions

28Stone
Java: Lambdas with local variables
	* Variáveis Locais: No Java, lambdas só podem aceder a variáveis locais que sejam final ou "effectively final" (que não mudam de valor após a inicialização). Como counter++ tenta modificar o valor, a regra é quebrada e ocorre um erro de compilação.
	* Variáveis de Instância/Static: Estas variáveis residem na Heap (memória partilhada) e não na Stack (memória local do thread). O Java permite que lambdas as modifiquem diretamente porque o ciclo de vida delas não está preso à execução do método. 
Java: Finalize
	* finalize() é um recurso do Java (herdado da classe Object) que permite que um objeto execute um "último desejo" ou limpeza antes de ser removido da memória pelo Garbage Collector (GC)

Spring: difference Spring vs SpringBoot
	* Opinionated Spring configuration
	* Auto-configuration
	* Embedded server
	* Starter dependencies
Java: Synchronize over constructors
	* 1. What is Forbidden?
		* The synchronized Keyword on Signature: 
		* 	You cannot write public synchronized MyClass(). 
		* 	compiler will throw an error because a constructor is not an instance method and 
		* 	can only be called once per object creation by the thread that invokes new.
		* Logical Impossibility: 
		* 	Since the object is still being born, no other thread can (theoretically) have a reference to it yet. 
		* 	Locking it before it exists is logically impossible. 
	* 2. What is Allowed?
		* Synchronized Blocks Inside: 
		* 	can use synchronized(this) { ... } or synchronized(SomeClass.class) { ... } inside the constructor body.
		* Synchronizing on Shared Resources: 
		* 	If your constructor modifies static variables or shared external objects, you should use a synchronized block on that specific shared resource. 
	* 3. Recommended Best Practices
		* ensure "Safe Publication"—
		* 	meaning other threads see your object only when it is fully ready. 
		* Avoid "The This Escape":
		* 	 Never pass this to another thread, register it in a listener, or store it in a public static map from within the constructor. 
		* 	If you do, other threads can access the object before it is fully initialized, leading to unpredictable bugs.
		* Use final Fields: 
		* 	Marking fields as final provides a built-in "happens-before" guarantee. 
		* 	Java ensures that any thread seeing the object after the constructor finishes 
		* 	will see the correct values of all final fields without needing extra synchronization.
		* Factory Methods: 
		* 	Instead of complex logic or synchronization in a constructor, use a Static Factory Method. 
		* 	Create the object first, then perform any necessary registration or thread-starting outside the constructor.
		* Immutability: 
		* 	The best way to avoid synchronization issues is to make your class immutable. 
		* 	If the state never changes after construction, no locking is ever needed. 
Java: Heap Down and Thread Down
	* "Heap" and "Thread" coupled with "Down" usually refer to two critical diagnostic actions: 
	* 	Dumping (exporting state) and 
	* 	Downsizing (reducing resource usage).
	* 1. Heap Down (Heap Dump / Downsizing)
	* 	Heap Dump (Snapshot): 
	* 		file containing a point-in-time "photograph" of all objects in your application's memory.
	* 		The Process: 
	* 			trigger a command (like jmap in Java) that freezes the memory state and writes it to a file.
	* 		The Goal: 
	* 			find Memory Leaks.
	* 			can see which objects are taking up the most space and 
	* 			why they aren't being cleaned up by the Garbage Collector.
	* 	Heap Downsizing: 
	* 		when the system shrinks the allocated memory pool.
	* 		The Process: 
	* 			After a Garbage Collection cycle, 
	* 			if application is using significantly less memory than the maximum limit, 
	* 			JVM or Database engine may return unused memory back to the Operating System to save resources.
	* 2. Thread Down (Thread Dump / Down-scaling)
	* 	Thread Dump (Execution Snapshot): 
	* 		text-based report showing exactly what every active thread in the system is doing at that exact millisecond.
	* 	The Process: 
	* 		lists the Stack Trace for every thread. You can see which threads are "Running," "Waiting," or "Blocked."
	* 	The Goal: 
	* 		Crucial for diagnosing Deadlocks (where two threads are stuck waiting for each other) or 
	* 		high CPU usage. If your app is "frozen," a thread dump tells you where it is stuck.
	* 	Thread Down-scaling (Pool Shrinking): 
	* 		This refers to reducing the number of active threads.
	* 	The Process: 
	* 		In a Thread Pool, if threads remain idle for too long (a "keep-alive" timeout), 
	* 		the system kills those threads to free up system overhead.
	* Comparison Summary
	* 	Feature	
	* 		Focus	
	* 		Primary Goal
	* 	Heap	
	* 		Data/Objects (Memory)	
	* 		Fix OutOfMemoryError and Leaks.
	* 	Thread	
	* 		Action/Execution (CPU)	
	* 		Fix Performance hangs and Deadlocks.

DB: Window Functions
	* SQL window functions are three main types: 
	* 	Aggregate, 
	* 	Ranking, and 
	* 	Value (or Offset/Analytic) 
	* perform calculations across a set rows 
	* 	related to the current row 
	* 	without collapsing them into a single result, 
	* 	preserving individual row details. 
	* Aggregate Window Functions
	* 	standard aggregate functions used within the OVER clause to calculate values over the defined window, 
	* 		such as running totals or moving averages. 
	* 	SUM(): 
	* 		Calculates the total sum of a column within the window.
	* 	AVG(): 
	* 		Computes the average value of a column within the window.
	* 	COUNT(): 
	* 		Returns the count of rows within the window.
	* 	MIN(): 
	* 		Finds the minimum value in a column within the window.
	* 	MAX(): 
	* 		Finds the maximum value in a column within the window. 
	* Ranking Window Functions
	* 	assign a rank or number to each row within a partition, 
	* 			typically requiring an ORDER BY clause within OVER() to define the ordering criteria. 
	* 	ROW_NUMBER(): 
	* 		Assigns a unique, sequential integer to each row, starting from 1.
	* 	RANK(): 
	* 		Assigns a rank to each row, with gaps in the ranking sequence for ties (e.g., 1, 1, 3).
	* 	DENSE_RANK(): 
	* 		Assigns a rank to each row without gaps for ties (e.g., 1, 1, 2).
	* 	PERCENT_RANK(): 
	* 		Calculates the relative rank of a row as a percentage (between 0 and 1).
	* 	NTILE(n): 
	* 		Divides the rows into a specified number (n) of approximately equal groups (buckets) and assigns a bucket number to each row. 
	* Value Window Functions (Offset/Analytic)
	* 	retrieve a value from another row within the window, 
	* 		useful for comparing values between the current and surrounding rows. 
	* 	LAG(): 
	* 		Accesses data from a row a specified physical offset before the current row.
	* 	LEAD(): 
	* 		Accesses data from a row a specified physical offset after the current row.
	* 	FIRST_VALUE(): 
	* 		Returns the value of a specified column from the first row in the window frame.
	* 	LAST_VALUE(): 
	* 		Returns the value of a specified column from the last row in the window frame.
	* 	NTH_VALUE(n): 
	* 		Returns the value from the nth row in the window frame.
	* 	CUME_DIST(): 
	* 		Calculates the cumulative distribution of a value within the set. 
	* 
	* 1. Funções de Classificação (Ranking)
	* 	Ideal para criar "Top 10" ou rankings internos.
	* 
	* SELECT 
	*     Vendedor, 
	*     Valor,
	*     ROW_NUMBER() OVER(ORDER BY Valor DESC) AS Sequencial,
	*     RANK() OVER(ORDER BY Valor DESC) AS Ranking_Com_Salto,
	*     DENSE_RANK() OVER(ORDER BY Valor DESC) AS Ranking_Sem_SaltoFROM Vendas;
	* 
	* * Utilidade: O DENSE_RANK é ótimo quando não queres saltar posições se houver empate no pódio.
	* 
	* 2. Funções de Agregação
	* 	Ideal para comparar o desempenho individual com o total ou calcular totais acumulados.
	* 
	* SELECT 
	*     Vendedor, 
	*     Departamento,
	*     Valor,
	*     SUM(Valor) OVER(PARTITION BY Departamento) AS Total_Depto,
	*     SUM(Valor) OVER(ORDER BY Data) AS Total_Acumulado_HistoricoFROM Vendas;
	* 
	* * Utilidade: Comparar quanto cada vendedor representa face ao total do seu departamento sem perder a linha individual.
	* 
	* 3. Funções de Valor (Navegação)
	* Essenciais para análises de crescimento (mês a mês).
	* 
	* SELECT 
	*     Data, 
	*     Valor,
	*     LAG(Valor) OVER(ORDER BY Data) AS Valor_Dia_Anterior,
	*     Valor - LAG(Valor) OVER(ORDER BY Data) AS Diferenca_CrescimentoFROM VendasWHERE Vendedor = 'João';
	* 
	* 
	* * Utilidade: O LAG "olha" para a linha de cima, permitindo calcular variações percentuais ou absolutas entre períodos.
	* 
	* 4. Funções Estatísticas
	* 	Úteis para análise de distribuição.
	* SELECT 
	*     Vendedor, 
	*     Valor,
	*     PERCENT_RANK() OVER(ORDER BY Valor) AS Percentil_VendaFROM Vendas;
	* * Utilidade: Saber em que percentil de performance um vendedor se encontra (ex: "Este vendedor está acima de 90% da equipa").

DB: TRUNCATE
	* The TRUNCATE TABLE command is a 
	* 	Data Definition Language (DDL) operation 
	* 	used to quickly remove all records from a table while keeping its structure (columns, constraints, and indexes) 
	* 	primarily used to "reset" a table to its original empty state. 
	* Key Features
	* 	* Performance: 
	* 		significantly faster than DELETE because it deallocates data pages instead of removing rows one by one.
	* 	* Minimal Logging: 
	* 		logs only the page deallocations rather than individual row deletions, which saves system resources.
	* 	* Identity Reset: 
	* 		resets any auto-increment (identity) columns back to their original seed value.
	* 	* Triggers: 
	* 		does not fire any DELETE triggers.
	* 	* No Filters: 
	* 		cannot use a WHERE clause; it is an "all-or-nothing" operation. [4, 5, 6, 7, 8, 9, 10, 11, 12] 
	* 
	* Syntax
	* 
	* TRUNCATE TABLE table_name;
	* 
	* 
	* *Note: 
	* 	In SQL Server and PostgreSQL, TRUNCATE can be rolled back if executed within a transaction. 
	* 	In MySQL and Oracle, it causes an implicit commit and cannot be undone. [5, 9, 21, 22] 
	* Limitations
	* 	* Foreign Keys: You generally cannot truncate a table that is referenced by a FOREIGN KEY constraint unless you drop the constraint first.
	* 	* Permissions: Requires ALTER table permissions, whereas DELETE only requires DELETE permissions. [16, 19, 21, 23]

Java: Mockito ways to mock
	* primary alternatives to the standard mock() functionality are Spies and Stubs, 
	* 1. Mockito Spy (@Spy)
	* 	A Spy is a "partial mock" that wraps a real instance of an object. [4] 
	* 	* Behavior: Unlike a regular mock, a spy calls the real methods of the object unless they are explicitly stubbed.
	* 	* Usage: Use it when you want to track interactions or modify only specific behaviors of a real object without mocking the entire class.
	* 	* Syntax: Mockito.spy(myObject) or the @Spy annotation. [5, 6, 7, 8] 
	* 
	* 2. Mockito Stubbing
	* 	While "mocking" creates the object, Stubbing is the process of defining exactly what that object should return when a method is called. [1] 
	* 	* Behavior: You replace a method's logic with a fixed response (e.g., thenReturn(value)).
	* 	* Usage: Use it when you simply need a dependency to provide data to your test without needing to verify how it was called. [2, 6, 7, 9, 10] 
	* 
	* 3. Manual Test Doubles
	* 	If you want to avoid Mockito's "magic" entirely, 
	* 	you can create manual implementations: 
	* 	* Fake: A simplified but working implementation (e.g., an in-memory database instead of a real one).
	* 	* Dummy: An object passed into a method just to satisfy a parameter requirement; its methods are never actually called. [2, 9, 13, 14, 15] 



SQL Tests on SpringBoot
	* test database queries in a Spring Boot environment while 
	* 	ensuring accuracy relative to production, 
	* 	industry-standard approach is to use Testcontainers 
	* 	rather than in-memory databases like H2. 
	* 
	* 1. Use Testcontainers for Environmental Parity [5] 
	* 	Testcontainers allows you to spin up a Docker container running the exact same database version used in production (e.g., PostgreSQL 15, MySQL 8). [3, 5, 6] 
	* 	* Setup: Use @DataJpaTest with @AutoConfigureTestDatabase(replace = Replace.NONE) to prevent Spring from replacing your containerized DB with H2.
	* 	* Benefits: You catch schema drift and dialect-specific bugs early. [2, 6, 7] 
	* 
	* 2. Manage Test Data and State
	* 	Testing "production conditions" requires realistic data without touching the actual production server. [4] 
	* 	* Schema Initialization: Use tools like Flyway or Liquibase to ensure the test database schema is identical to production.
	* 	* Data Seeding:
	* 	* @Sql annotation: Use this to run .sql scripts before specific tests to set up the required state.
	* 	   * Captured Traces: Tools like [BitDive](https://bitdive.io/java-testcontainers-integration-testing/) can capture production traces and auto-seed a Testcontainer with that specific state. [2, 6, 8, 9] 
	* 
	* 3. Safe "Read-Only" Production Validation
	* 	If you must run tests against the actual production database to check conditions (highly discouraged for standard CI/CD):
	* 	* Read-Only Profile: Define a specific Spring Profile (e.g., prod-check) with a read-only database user.
	* 	* Read-Only Transactions: Use @Transactional(readOnly = true) to ensure the test cannot inadvertently modify production data. [10, 11, 12] 
	* 
	* Recommended Implementation Pattern
	* 
	* @DataJpaTest
	* @Testcontainers
	* @ActiveProfiles("test") // Use a separate application-test.yml
	* @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)class ProductionQueryTest {
	* 
	*     @Container
	*     static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");
	* 
	*     @Test
	*     @Sql("/scripts/setup_production_scenario.sql") // Seed with production-like data
	*     void testComplexProductionQuery() {
	*         // Run your repository/service queries here
	*         // Assert conditions based on the seeded data
	*     }
	* }
	* 
	* Summary of Best Practices
	* 	* Isolation: Every test should run in its own transaction and rollback by default to maintain a clean state.
	* 	* Slices: Use @DataJpaTest to load only the persistence layer, making tests much faster than a full @SpringBootTest.
	* 	* CI/CD: Ensure your build pipeline supports Docker to run these Testcontainers automatically. [2, 9, 13, 14, 15] 



SpringBoot security filter chain
	* Security Filter Chain
	* 	runs before your Controller to handle authentication 
	* 	Every incoming HTTP request is intercepted by a series of filters that 
	* 	verify identity and permissions before the request ever reaches your business logic. 
	* 1. The Core Component: SecurityFilterChain
	* 	The "chain" is a sequence of ordered filters. You configure this by defining a SecurityFilterChain bean. [2, 5] 
	* 	* Common Built-in Filters:
	* 	* UsernamePasswordAuthenticationFilter: Checks for login form submissions.
	* 	   * BasicAuthenticationFilter: Processes "Basic" Auth headers.
	* 	   * BearerTokenAuthenticationFilter: Used for JWT/OAuth2 token validation. [5, 6, 7, 8] 
	* 
	* 2. Creating a Custom Filter
	* 	If you need custom logic (like checking a specific header or a custom JWT format), you typically extend OncePerRequestFilter. [9, 10] 
	* 
	* public class MyCustomFilter extends OncePerRequestFilter {
	*     @Override
	*     protected void doFilterInternal(HttpServletRequest request, 
	*                                     HttpServletResponse response, 
	*                                     FilterChain filterChain) throws ServletException, IOException {
	*         
	*         // 1. Extract credentials (e.g., from Header)
	*         String token = request.getHeader("X-Custom-Token");
	* 
	*         // 2. Authenticate and set the SecurityContext
	*         if ("valid-token".equals(token)) {
	*             SecurityContextHolder.getContext().setAuthentication(
	*                 new UsernamePasswordAuthenticationToken("user", null, new ArrayList<>())
	*             );
	*         }
	* 
	*         // 3. Continue to the next filter (or the Controller)
	*         filterChain.doFilter(request, response);
	*     }
	* }
	* 
	* 3. Registering the Filter
	* You must explicitly add your filter to the chain in your security configuration using addFilterBefore or addFilterAfter. [9, 11] 
	* 
	* @Beanpublic SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	*     http
	*         .addFilterBefore(new MyCustomFilter(), UsernamePasswordAuthenticationFilter.class)
	*         .authorizeHttpRequests(auth -> auth
	*             .anyRequest().authenticated()
	*         );
	*     return http.build();
	* }
	* 
	* Key Summary Table
	* | DelegatingFilterProxy | The entry point that bridges the Servlet container to Spring Security. |
	* | FilterChainProxy | Manages and executes the list of security filters in order. |
	* | SecurityContext | Where the authenticated user information is stored for the duration of the request. |
	* | @PreAuthorize | An alternative check that happens at the method level after authentication but right before the controller method executes. |

 
Java Evolution (8 → 21)

Java switched to a 6-month release cycle starting with Java 9, so some versions are small. The most important LTS versions are:
	* Java 8 (2014)
	* Java 11 (2018)
	* Java 17 (2021)
	* Java 21 (2023)


________________________________________
Java 8 (2014) — Major Functional Programming Upgrade

This was one of the biggest Java releases ever.
1. Lambda Expressions
Introduced functional programming style.
list.forEach(x -> System.out.println(x));

2. Stream API
Declarative data processing.
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .collect(Collectors.toList());
	* functional style
	* parallel processing
	* less boilerplate

3. Functional Interfaces
Example:
Predicate
Function
Supplier
Consumer

4. Default Methods in Interfaces
Interfaces can have implementations.
default void log() {
    System.out.println("log");
}

5. Optional
Better null handling.
Optional<String> name = Optional.of("John");

6. New Date & Time API
Replaced problematic java.util.Date.
Examples:
LocalDate
LocalDateTime
Instant
Duration
 
________________________________________
Java 9 (2017) — Modular System

Project Jigsaw (Modules)
Introduced Java Platform Module System (JPMS).
Example:
module-info.java
Benefits:
	* strong encapsulation
	* smaller runtimes
	* better dependency management

Other features:
JShell (REPL)
Interactive Java shell.
jshell

Stream API improvements
New methods:
takeWhile
dropWhile
iterate
________________________________________
Java 10 (2018)

Local Variable Type Inference
var list = new ArrayList<String>();
var lets the compiler infer type.
Rules:
	* only for local variables
	* type still static
________________________________________
Java 11 (2018) — LTS
Major production adoption.
1. New String methods
isBlank()
lines()
strip()
repeat()

2. HTTP Client API
Modern replacement for HttpURLConnection.
HttpClient client = HttpClient.newHttpClient();
Supports:
	* HTTP/2
	* async calls

3. Lambda parameter var
(var x, var y) -> x + y

4. Removal of Java EE modules
Removed from JDK:
JAXB
CORBA
JAX-WS
Moved outside JDK.

________________________________________
Java 12–13 (Small Improvements)
Important feature:

Switch Expressions (preview → stable later)
Old switch:
switch(day) {
  case MONDAY:
     return 1;
}
New:
int value = switch(day) {
  case MONDAY -> 1;
  case TUESDAY -> 2;
  default -> 0;
};
Benefits:
	* cleaner
	* expression-based
	* no fallthrough

________________________________________
Java 14 (2020)

Records (Preview)
Simplified immutable data classes.
Example:
record User(String name, int age) {}
Automatically generates:
	* constructor
	* getters
	* equals
	* hashCode
	* toString

Also:
Helpful NullPointerExceptions
Shows which variable is null.

________________________________________
Java 15

Text Blocks
Multiline strings.
String json = """
{
   "name": "John"
}
""";
Removes escape clutter.

________________________________________
Java 16

Records (Final)
Records became official feature.

Pattern Matching for instanceof
Old:
if (obj instanceof String) {
    String s = (String) obj;
}
New:
if (obj instanceof String s) {
    System.out.println(s.length());
}

________________________________________
Java 17 (2021) — LTS

Very widely adopted.

1. Sealed Classes
Control inheritance.
public sealed class Shape
    permits Circle, Square {}
Used with:
sealed
non-sealed
final
Great for domain modeling.

2. Pattern Matching improvements
Improves type checks and casting.

3. New macOS rendering pipeline
Performance improvements.

________________________________________
Java 18–19

Virtual Threads (Preview)
Part of Project Loom.
Allows massive concurrency.
Example:
Thread.startVirtualThread(() -> {
    handleRequest();
});
Benefits:
	* millions of threads
	* lightweight
	* simpler than reactive programming

________________________________________
Java 20
Continued improvements to:
	* virtual threads
	* pattern matching
	* record patterns



________________________________________
Java 21 (2023) — LTS

Major modern Java version.

1. Virtual Threads (FINAL)
Game-changing feature.
Thread.ofVirtual().start(() -> doWork());
Benefits:
	* massive concurrency
	* simpler than reactive frameworks
	* better for IO-heavy services

2. Pattern Matching for switch
switch(obj) {
  case String s -> s.length();
  case Integer i -> i * 2;
  default -> 0;
}
Cleaner type handling.

3. Record Patterns
Destructure objects.
Example:
if (point instanceof Point(int x, int y)) {
    System.out.println(x + y);
}

4. Sequenced Collections
New interfaces for collections with defined order.

________________________________________
Summary
Version	Major Feature
Java 8	Lambdas, Streams, Optional, Date API
Java 9	Modules (JPMS), JShell
Java 10	var
Java 11	HTTP Client, String improvements
Java 14	Records (preview)
Java 15	Text blocks
Java 16	Records final
Java 17	Sealed classes
Java 19	Virtual threads (preview)
Java 21	Virtual threads final, pattern matching
________________________________________
 
Features Senior Engineers Should Know Well
Most important to understand deeply:
1.	Streams
2.	Optional
3.	Records
4.	Sealed classes
5.	Pattern matching
6.	Virtual threads
7.	Modules
8.	var

 
________________________________________

________________________________________

________________________________________
4.	Records

Core Concept
Records provide compact immutable data classes.
Example:
record User(String name, int age) {}
Automatically generates:
constructor
getters
equals
hashCode
toString

Important Properties
Records are:
final
immutable
shallowly immutable

Common Use Cases
DTOs
API responses
domain value objects
________________________________________
5.	7. Sealed Classes

Core Concept
Sealed classes restrict which classes can extend a class.
Example:
public sealed class Shape
    permits Circle, Square {}
Subclasses must be:
final
sealed
non-sealed
Benefits
	* safer inheritance
	* better domain modeling
	* useful for pattern matching
public sealed class Animal permits Dog, Cat {}

// 1. Final: Ninguém pode herdar de Dog
public final class Dog extends Animal {}

// 2. Non-sealed: Qualquer um pode herdar de Cat (ex: SiameseCat, StreetCat)
public non-sealed class Cat extends Animal {}

// Esta classe é perfeitamente válida:
public class SiameseCat extends Cat {}

________________________________________
6.	8. Pattern Matching
Core Concept
Reduces boilerplate when checking types.
Old code:
if (obj instanceof String) {
   String s = (String) obj;
}
New code:
if (obj instanceof String s) {
   System.out.println(s.length());
}
Also works with switch expressions.
Example:
switch(obj) {
   case String s -> s.length();
   case Integer i -> i * 2;
}
Benefits
cleaner type checks
less casting
safer code






________________________________________
7.	9. Generics

Core Concept
Generics provide compile-time type safety.
Example:
List<String> names = new ArrayList<>();

Important Concepts
Type erasure
Generic types are erased at runtime.
Example:
List<String>
List<Integer>
Both become:
List

Wildcards
? extends T
? super T
Rule:
Producer → extends
Consumer → super
(PECS principle)

________________________________________
________________________________________
What Senior Developers Should Understand Deeply
Most critical concepts:
1.	Streams & functional programming
2.	Concurrency & Java Memory Model
3.	CompletableFuture async pipelines
4.	Virtual threads
5.	Records and sealed classes
6.	Generics and type erasure
7.	Garbage collection basics
________________________________________
✅ Interview Tip
Senior-level interviews usually probe why a feature exists and when to use it, not just syntax.
Example:
Bad answer:
Streams allow filtering collections.
Better answer:
Streams enable declarative pipelines that support lazy evaluation and parallel processing, which can improve readability and performance when processing collections.
________________________________________
12 most common Java coding interview problems 
For each problem I summarize:
	* What the problem tests
	* Key concepts
	* Typical approach
________________________________________
1. LRU Cache
Problem
Implement an LRU (Least Recently Used) cache with:
get(key)
put(key, value)
Both must run in O(1).
Concepts Tested
	* HashMap
	* Doubly Linked List
	* Data structure design
Typical Approach
Use:
HashMap → key → node
Doubly Linked List → usage order
Java actually provides a shortcut:

// O terceiro parâmetro 'true' ativa a ordem de acesso
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);

Ordem de Inserção (Padrão/false): O get() apenas retorna o valor. A ordem dos elementos na iteração permanece a mesma de quando foram inseridos.
Ordem de Acesso (true): Sempre que chamas get(key) ou put(key, value), essa entrada é movida para o final da lista ligada. O elemento no "topo" (início) da lista passa a ser o que não é acedido há mais tempo.
 método removeEldestEntry funciona como um gatilho (trigger): ele é chamado automaticamente pelo LinkedHashMap sempre que fazes um put ou putAll.

class LRUCache<K,V> extends LinkedHashMap<K,V> {

    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > capacity;
    }
}

 
________________________________________
2. Top K Frequent Elements
Problem
Find the K most frequent elements in a list.
Concepts
	* HashMap frequency counting
	* Heap / PriorityQueue
Approach
1 count frequency
2 use min heap of size K
Complexity:
O(n log k)

    public List<Integer> topKFrequent(int[] nums, int k) {
        // 1. Contar frequências
        Map<Integer, Integer> count = new HashMap<>();
        for (int n : nums) {
            count.put(n, count.getOrDefault(n, 0) + 1);
        }
        // 2. Criar uma Min-Heap baseada na frequência
        PriorityQueue<Integer> heap = new PriorityQueue<>(
            (n1, n2) -> count.get(n1) - count.get(n2)
        );
        // 3. Manter apenas os K elementos mais frequentes na heap
        for (int n : count.keySet()) {
            heap.add(n);
            if (heap.size() > k) {
                heap.poll(); // Remove o elemento com menor frequência
            }
        }
        // 4. Converter para lista (resultado final)
        List<Integer> result = new ArrayList<>(heap);
        Collections.reverse(result); // Opcional: ordenar do mais para o menos frequente
        return result;
    }
}
public List<Integer> topKFrequent(int[] nums, int k) {
    // 1. Contar frequências (Stream de agrupamento)
    Map<Integer, Long> count = Arrays.stream(nums)
        .boxed()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

    // 2. Filtrar os K mais frequentes
    return count.keySet().stream()
        .sorted((a, b) -> count.get(b).compareTo(count.get(a)))
        .limit(k)
        .toList();
}


________________________________________
3. First Non-Repeating Character
Problem
Return the first character in a string that appears only once.
Concepts
	* HashMap counting
	* Ordered traversal
Approach
1 build frequency map
2 scan string again
Complexity:
O(n)

________________________________________
4. Merge Intervals
Problem
Given overlapping intervals, merge them.
Example:
[1,3], [2,6], [8,10]
→ [1,6], [8,10]
Concepts
	* Sorting
	* Greedy algorithms
Approach
1 sort by start
2 iterate and merge
Complexity:
O(n log n)
________________________________________
5. Detect Cycle in Linked List
Problem
Detect if a linked list contains a cycle.
Concepts
	* Floyd's cycle detection
	* Fast / slow pointers
Approach
slow = slow.next
fast = fast.next.next
If they meet → cycle exists.
Complexity:
O(n)
Memory:
O(1)
________________________________________
6. Longest Substring Without Repeating Characters
Problem
Example:
abcabcbb → 3
Concepts
	* Sliding window
	* HashSet / HashMap
Approach
two pointers
expand window
shrink when duplicate
Complexity:
O(n)
________________________________________
7. Reverse a Linked List
Problem
Reverse a singly linked list.
Concepts
	* pointer manipulation
Iterative Solution
Node prev = null;
Node curr = head;

while(curr != null) {
    Node next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}

return prev;
________________________________________
8. Implement Rate Limiter
Problem
Limit API requests per user.
Example:
100 requests per minute
Concepts
	* System design
	* Sliding window
	* token bucket
Approaches
Common algorithms:
fixed window
sliding window
token bucket
leaky bucket
________________________________________
9. Producer Consumer Problem
Problem
Multiple producers and consumers share a queue.
Concepts
	* concurrency
	* thread synchronization
Java Solution
Use:
BlockingQueue
Example:
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
Benefits:
thread safe
built-in blocking
________________________________________
10. Implement a Thread-Safe Singleton
Concepts
	* concurrency
	* memory visibility
	* volatile
Correct Implementation
public class Singleton {

    private static volatile Singleton instance;

    public static Singleton getInstance() {
        if(instance == null) {
            synchronized(Singleton.class) {
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
Uses:
double-checked locking
________________________________________
11. Flatten a Nested List
Example:
[1,[2,[3,4],5]]
→ [1,2,3,4,5]
Concepts
	* recursion
	* stack
Approach
DFS traversal
________________________________________
12. Find Duplicate Elements in Large Dataset
Problem
Dataset too large for memory.
Concepts
	* hashing
	* bloom filters
	* external sorting
Typical solutions:
hash partitioning
bitmaps
stream processing
Often asked in big data / backend interviews.
________________________________________
What Senior Interviewers Actually Evaluate
Not just correctness, but:
1. Code clarity
Good naming, readable logic.
________________________________________
2. Time complexity
Example:
O(n)
O(n log n)
O(1)
You should state complexity before coding.
________________________________________
3. Edge cases
Examples:
null input
empty collections
duplicates
large inputs
________________________________________
4. Use of modern Java
Strong candidates use:
Streams
Optional
Collections
Concurrent utilities
________________________________________
Coding Problems Most Often Asked to Java Backend Engineers
Most common ones:
1.	LRU Cache
2.	Longest substring without repeating characters
3.	Merge intervals
4.	Top K elements
5.	Cycle detection
6.	Thread-safe singleton
7.	Producer-consumer problem
________________________________________
If you'd like, I can also show you 5 realistic backend coding exercises used in senior Java interviews (they are much closer to real work than algorithm puzzles).
Below are 5 realistic backend coding exercises often used in senior Java interviews. These are closer to real backend engineering tasks than typical algorithm puzzles. Interviewers evaluate design, clean code, concurrency handling, and correctness, not just raw algorithm skills.
________________________________________
1. Implement an In-Memory Key-Value Store
Problem
Create a simple in-memory database with operations:
put(key, value)
get(key)
delete(key)
Optionally support:
getAllKeys()
expiration (TTL)
________________________________________
Concepts Tested
	* Data structures
	* Thread safety
	* Clean API design
	* Concurrency
________________________________________
Simple Implementation
class KeyValueStore<K, V> {

    private final Map<K, V> store = new ConcurrentHashMap<>();

    public void put(K key, V value) {
        store.put(key, value);
    }

    public V get(K key) {
        return store.get(key);
    }

    public void delete(K key) {
        store.remove(key);
    }
}
________________________________________
Senior-Level Discussion Topics
	* Why ConcurrentHashMap
	* TTL support
	* eviction strategies
	* persistence
	* scalability
________________________________________
2. Implement a Rate Limiter
Problem
Limit API requests:
100 requests per user per minute
________________________________________
Concepts Tested
	* concurrency
	* time windows
	* thread safety
	* system design thinking
________________________________________
Sliding Window Example
class RateLimiter {

    private final Map<String, Queue<Long>> requests = new ConcurrentHashMap<>();
    private final int limit = 100;
    private final long window = 60000;

    public synchronized boolean allowRequest(String user) {

        long now = System.currentTimeMillis();

        requests.putIfAbsent(user, new LinkedList<>());
        Queue<Long> queue = requests.get(user);

        while (!queue.isEmpty() && now - queue.peek() > window) {
            queue.poll();
        }

        if (queue.size() < limit) {
            queue.add(now);
            return true;
        }

        return false;
    }
}
________________________________________
Senior-Level Discussion
	* distributed rate limiting
	* Redis implementation
	* token bucket algorithm
	* sliding window vs fixed window
________________________________________
3. Implement a Simple Task Scheduler
Problem
Execute tasks after a delay.
Example:
schedule(task, delay)
________________________________________
Concepts Tested
	* concurrency
	* scheduling
	* Java concurrency utilities
________________________________________
Java Implementation
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(2);

scheduler.schedule(() -> {
    System.out.println("Task executed");
}, 5, TimeUnit.SECONDS);
________________________________________
Senior-Level Discussion
	* scheduling reliability
	* persistence of tasks
	* distributed scheduling
	* cron systems
________________________________________
4. Design a Log Aggregator
Problem
Create a system that collects logs and allows querying.
Operations:
addLog(service, message)
getLogs(service)
________________________________________
Concepts Tested
	* data modeling
	* indexing
	* scalability thinking
________________________________________
Example Implementation
class LogAggregator {

    private final Map<String, List<String>> logs = new ConcurrentHashMap<>();

    public void addLog(String service, String message) {
        logs.computeIfAbsent(service, k -> new ArrayList<>())
            .add(message);
    }

    public List<String> getLogs(String service) {
        return logs.getOrDefault(service, List.of());
    }
}
________________________________________
Senior-Level Discussion
Real systems use:
Kafka
Elasticsearch
Logstash
OpenTelemetry
________________________________________
5. Implement a Concurrent Job Queue
Problem
Multiple producers submit jobs, multiple workers consume them.
________________________________________
Concepts Tested
	* thread safety
	* blocking queues
	* producer-consumer pattern
________________________________________
Java Implementation
class JobQueue {

    private final BlockingQueue<Runnable> queue =
            new LinkedBlockingQueue<>();

    public void submit(Runnable job) {
        queue.add(job);
    }

    public void worker() {
        while (true) {
            try {
                Runnable job = queue.take();
                job.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
________________________________________
Senior-Level Discussion
Topics interviewers may explore:
	* backpressure
	* queue overflow
	* priority queues
	* distributed workers
________________________________________
What Interviewers Evaluate in These Exercises
Not just code, but:
1. API Design
Example:
clear method names
clean interfaces
________________________________________
2. Thread Safety
Important tools:
ConcurrentHashMap
BlockingQueue
synchronized
Locks
________________________________________
3. Scalability Thinking
Senior engineers should discuss:
horizontal scaling
distributed systems
fault tolerance
________________________________________
4. Edge Cases
Examples:
null input
empty queues
concurrent access
memory growth
________________________________________
Most Common Real-World Backend Interview Tasks
These appear very often:
1.	Rate limiter
2.	LRU cache
3.	Task scheduler
4.	Job queue
5.	In-memory database
They test engineering judgment more than algorithms.
________________________________________
✅ If you'd like, I can also show you one extremely common senior Java interview exercise: designing a URL shortener service, which combines coding + system design and appears very frequently in backend interviews.

 
 
