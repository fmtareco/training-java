## Java Evolution (8 → 21)

Java switched to a 6-month release cycle starting with Java 9, so some versions are small. 
The most important LTS versions are:
* Java 8 (2014)
* Java 11 (2018)
* Java 17 (2021)
* Java 21 (2023)

----

# Java 8 (2014) — Major Functional Programming Upgrade


* This was one of the biggest Java releases ever.

1. Lambda Expressions
--
* Introduced functional programming style.
---
	list.forEach(x -> System.out.println(x));
---

2. Stream API
--
* Declarative data processing.
---
	list.stream()
		.filter(x -> x > 10)
		.map(x -> x * 2)
		.collect(Collectors.toList());
---
* functional style
* parallel processing
* less boilerplate


3. Functional Interfaces
--
* Examples:
	* Predicate
	* Function
	* Supplier
	* Consumer

4. Default Methods in Interfaces
--
* Interfaces can have implementations.
---
	default void log() {
		System.out.println("log");
	}
---

5. Optional
--
* Better null handling.
---
	Optional<String> name = Optional.of("John");
---

6. New Date & Time API
--
* Replaced problematic java.util.Date.
* Examples:
	* LocalDate
	* LocalDateTime
	* Instant
	* Duration


----

# Java 9 (2017) — Modular System


Project Jigsaw (Modules)
--
* Introduced Java Platform Module System (*JPMS*).
* Example:
	* module-info.java
* Benefits:
	* strong encapsulation
	* smaller runtimes
	* better dependency management
---
	* Before JPMS, everything on the "Classpath" was visible to everyone. 
	* With JPMS, internal packages are hidden by default.

	The Core Concept: module-info.java	
        * To turn a project into a module, you must add a file named module-info.java at the root of your source folder. 
            * requires: Declares a dependency on another module.
            * exports: Makes a package accessible to other modules. [11, 12, 13, 14] 

	Why use it? (Benefits)	
		* Strong Encapsulation: 
			* You can have public classes that are still private to the module. 
			* Other developers can't "hack" into your internal logic.
		* Reliable Configuration: 
			* The JVM checks if all requires are present at startup. 
			* No more ClassNotFoundException in the middle of production.
		* Scalability (jlink): 
			* You can create a custom, tiny Java Runtime (JRE) that only includes the modules your app actually uses, reducing size from 200MB+ to ~30MB. 
---


JShell (REPL)
--
* Interactive Java shell.
* jshell


Stream API improvements
--
* New methods:
	* takeWhile
	* dropWhile
	* iterate

----

# Java 10 (2018)


* Local Variable Type Inference
---
	var list = new ArrayList<String>();
---

* var lets the compiler infer type.
* Rules:
	* only for local variables
	* type still static

----

# Java 11 (2018) — LTS

* Major production adoption.


1. New String methods
---
	 isBlank()
	 lines()
	 strip()
	 repeat()
---

2. HTTP Client API
--
* Modern replacement for HttpURLConnection.
---
	HttpClient client = HttpClient.newHttpClient();
---
* Supports:
	* HTTP/2
	* async calls

3. Lambda parameter var
---
	(var x, var y) -> x + y
---

4. Removal of Java EE modules
--
* Removed from JDK:
	* JAXB
	* CORBA
	* JAX-WS
* Moved outside JDK.

----

# Java 12–13 (Small Improvements)

Switch Expressions (preview → stable later)
--
* Old switch:
---
	switch(day) {
	  case MONDAY:
		 return 1;
	}
---
* New:
---
	int value = switch(day) {
	  case MONDAY -> 1;
	  case TUESDAY -> 2;
	  default -> 0;
	};
---

* Benefits:
	* cleaner
	* expression-based
	* no fallthrough

----

# Java 14 (2020)

Records (Preview)
--
* Simplified immutable data classes.
* Example:
---
	record User(String name, int age) {}
---

* Automatically generates:
	* constructor
	* getters
	* equals
	* hashCode
	* toString

Helpful NullPointerExceptions
--
* Shows which variable is null.


----

# Java 15

Text Blocks
--
* Multiline strings.
---
	String json = """
	{
	   "name": "John"
	}
	""";
---

* Removes escape clutter.

----

# Java 16

Records (Final)
--
* Records became official feature.

* Pattern Matching for instanceof
* Old:
---
	if (obj instanceof String) {
		String s = (String) obj;
	}
---
* New:
---
	if (obj instanceof String s) {
		System.out.println(s.length());
	}
---

----

# Java 17 (2021) — LTS

* Very widely adopted.

1. Sealed Classes
--

* Control inheritance.
---
	public sealed class Shape
		permits Circle, Square {}
---

* Used with:
	* sealed
	* non-sealed
	* final
* Great for domain modeling.

2. Pattern Matching improvements
--
* Improves type checks and casting.

3. New macOS rendering pipeline
--
* Performance improvements.

----

# Java 18–19

* Virtual Threads (Preview)
	* Part of Project Loom.
	* Allows massive concurrency.
* Example:
---
	Thread.startVirtualThread(() -> {
		handleRequest();
	});
---

* Benefits:
	* millions of threads
	* lightweight
	* simpler than reactive programming

----

# Java 20

* Continued improvements to:
	* virtual threads
	* pattern matching
	* record patterns

----

# Java 21 (2023) — LTS

* Major modern Java version.

1. Virtual Threads (FINAL)
--
* Game-changing feature.
---
	Thread.ofVirtual().start(() -> doWork());
---

* Benefits:
	* massive concurrency
	* simpler than reactive frameworks
	* better for IO-heavy services

2. Pattern Matching for switch
--
---
	switch(obj) {
	  case String s -> s.length();
	  case Integer i -> i * 2;
	  default -> 0;
	}
---
* Cleaner type handling.

3. Record Patterns
--
* Destructure objects.
* Example:
---
	if (point instanceof Point(int x, int y)) {
		System.out.println(x + y);
	}
---

* example:
---
	record Localizacao(String cidade, String pais) {}
	record Encomenda(String id, double peso, Localizacao destino) {}

	public class Main {
		public static void main(String[] args) {

			// 1. Destruturação com instanceof (Java 21)
			if (obj instanceof Encomenda(String id, double p, Localizacao(String c, String pais))) {
				System.out.println("ID: " + id);
				System.out.println("Cidade de Destino: " + c); // Acedemos à variável 'c' diretamente
			}
			// 2. Destruturação com Pattern Matching no switch (Java 21)
			String mensagem = switch (obj) {
				case Encomenda(var id, var peso, Localizacao(var cid, var p)) when peso > 10 -> 
					"Encomenda pesada para " + cid;
				case Encomenda(var id, var peso, var loc) -> 
					"Encomenda normal para " + loc.cidade();
				default -> "Objeto desconhecido";
			};
			
			System.out.println(mensagem);
		}
	}
---

4. Sequenced Collections
--
* New interfaces for collections with defined order.

Sequenced Collections 
--
* introduce a unified interface for collections with a defined encounter order.

1. The New Hierarchy
--
* Java 21 added three core interfaces:
	* SequencedCollection<E>: For Lists and Sets.
	* SequencedSet<E>: For Sets that maintain order (like LinkedHashSet).
	* SequencedMap<K, V>: For Maps that maintain order (like LinkedHashMap).
	
2. Key Methods
--
* These methods are now available across all sequenced types:
	* addFirst(E), addLast(E)
	* getFirst(), getLast()
	* removeFirst(), removeLast()
	* reversed(): Returns a reverse-order view (O(1) complexity).
---
	public class SequencedExample {
		public static void main(String[] args) {
			// --- SequencedCollection (List) ---
			SequencedCollection<String> list = new ArrayList<>(List.of("B", "C"));
			
			list.addFirst("A"); // [A, B, C]
			list.addLast("D");  // [A, B, C, D]
			
			System.out.println(list.getFirst()); // "A"
			System.out.println(list.reversed()); // [D, C, B, A]

			// --- SequencedSet (LinkedHashSet) ---
			SequencedSet<Integer> set = new LinkedHashSet<>();
			set.add(10);
			set.add(20);
			set.addFirst(5); // [5, 10, 20] - Moves 5 to the front even if it existed!
			
			System.out.println(set.getLast()); // 20
		}
	}

---

----

# Java 22 (March 2024)

Unnamed Variables & Patterns: 
--
* Finalized the use of the underscore (_) for variables or patterns that must be declared but are not used, 
* improving code readability.

Foreign Function & Memory API: 
--
* A permanent feature providing a safer and more efficient way to call native code (C/C++) and access foreign memory, 
* replacing the older JNI.

Statements before super(...): (Preview) 
--
* Allows logic to appear in a constructor before calling super() or this(), 
* as long as it doesn't reference the instance being created.

Stream Gatherers: (Preview) 
--
* Introduced a more flexible way to create custom intermediate operations for Java Streams. 

----

# Java 23 (September 2024)

Markdown Documentation Comments: 
--
* Allows developers to write Javadoc in Markdown instead of just HTML, 
* making documentation much easier to read and maintain.

Primitive Types in Patterns: (Preview) 
--
* Expanded pattern matching to include primitive types (e.g., int, double) in *instanceof* and *switch*.

ZGC Generational Mode: 
--
* Starting in Java 23, the Z Garbage Collector uses Generational mode by default, optimizing memory management for short-lived objects.

----

# Java 24 (March 2025) 

Ahead-of-Time (AOT) Class Loading: 
--
* Part of Project Leyden, 
* this feature significantly reduces application startup times by pre-loading and linking classes before runtime.

Compact Object Headers: (Experimental) 
--
* Reduces the memory overhead of every Java object, 
* improving heap usage efficiency for large-scale applications.

Generational Shenandoah: 
--
* Introduced a generational mode for the Shenandoah GC, 
* aiming for low-latency collection with reduced memory footprint.

----

# Java 25 (September 2025 - LTS) 

* Java 25 acts as the consolidation release, 
* finalizing many features that were in preview in previous versions.

Flexible Constructor Bodies: 
--
* Officially finalized the ability to write code before super() or this() in constructors.

Module Import Declarations: 
--
* Allows developers to import all packages exported by a module with a single line 
* (e.g., import module java.base), simplifying large codebase management.

Instance Main Methods: 
--
* Finalized the ability to run simple "main" methods without the static/public/String-array boilerplate,
* making the language more friendly to beginners.

Performance Story: 
--
* Represents the "leanest" JVM to date, with up to 20-30% reduction in heap usage compared to older versions and faster startup.

----

# Summary

| Version | Major Feature |
|---|---|
| Java 8 | Lambdas, Streams, Optional, Date API | 
| Java 9 | Modules (JPMS), JShell | 
| Java 10 | var | 
| Java 11 | HTTP Client, String improvements | 
| Java 14 | Records (preview )| 
| Java 15 | Text blocks | 
| Java 16 | Records final | 
| Java 17 | Sealed classes | 
| Java 19 | Virtual threads (preview )| 
| Java 21 | Virtual threads final, pattern matching | 
| Java 22 | Unnamed Variables & Patterns , Statements before super(...), Stream Gatherers | 
| Java 23 | Markdown Documentation Comments, Primitive Types in Patterns, ZGC Generational Mode | 
| Java 24 | Ahead-of-Time (AOT) Class Loading, Compact Object Headers, Generational Shenandoah | 
| Java 25 | Flexible Constructor Bodies, Module Import Declarations, Instance Main Methods, Performance Story  | 

