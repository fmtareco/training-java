##  TDD (Test-Driven Development) in a Java/Spring Boo

To master TDD (Test-Driven Development) in a Java/Spring Boot context, 
you must shift your mindset from "*testing code*" to "**using tests to design code**."

----

# Fundamentals: The Red-Green-Refactor Cycle

   1. ***Red***: 
		* Write a small, failing test for a piece of functionality that doesn't exist yet.
   2. ***Green***: 
		* Write the minimum amount of code to make the test pass. 
		* (It’s okay if the code is "ugly" at this stage).
   3. ***Refactor***: 
		* Clean up the code while ensuring the test stays green. 
		* Improve readability and remove duplication.

----

# TDD Strategies in Spring Boot


* In Spring Boot, TDD follows a "Double Loop" approach:

	* Outer Loop (Integration): 
		* Test the API endpoint (Controller) using MockMvc.
	* Inner Loop (Unit): 
		* Test the business logic (Service/Domain) using Mockito.

A. Unit Testing Strategy (The **Inner Loop**)
--
* Focus on isolated logic. 
* Use Mockito to mock dependencies.

---
	// 1. RED: Test doesn't compile or fails because Service doesn't exist
	
	@ExtendWith(MockitoExtension.class)
	class BonusServiceTest {
		@InjectMocks
		private BonusService bonusService;

		@Test
		void shouldCalculateTenPercentBonus() {
			double result = bonusService.calculate(100.0);
			assertThat(result).isEqualTo(10.0);
		}
	}
	// 2. GREEN: Minimum code in BonusService.javapublic double calculate(double amount) {
		return amount * 0.1;
	}
---

B. Integration Testing Strategy (The **Outer Loop**)
--
* Use ***@WebMvcTest*** for lightweight Controller testing 
* or ***@SpringBootTest*** for full context.

---
	@WebMvcTest(ProductController.class)
	class ProductControllerTest {
		@Autowired
		private MockMvc mockMvc;

		@MockBean
		private ProductService productService;

		@Test
		void shouldReturnProductList() throws Exception {
			when(productService.findAll()).thenReturn(List.of(new Product("Apple")));

			mockMvc.perform(get("/api/products"))
				   .andExpect(status().isOk())
				   .andExpect(jsonPath("$[0].name").value("Apple"));
		}
	}
---

----

# Common Questions 

Q1: What is the difference between TDD and Unit Testing?
--
* Unit Testing is a tactic to verify code. 
* TDD is a design process where tests are written before the code to define the requirements and interface.

Q2: How do you handle a database in TDD?
--
* For unit tests, mock the Repository. 
* For integration tests, use an in-memory DB like H2 or, ideally, Testcontainers (Docker-based) to ensure the test environment matches production.

Q3: What makes a "good" test in TDD?
--
* Use the ***FIRST*** principle:
	* ***Fast***: Tests must run in seconds.
	* ***Independent***: No test should depend on another.
	* ***Repeatable***: Same result every time.
	* ***Self-validating***: Pass or fail (no manual log checking).
	* ***Timely***: Written before the production code.

Q4: When should you NOT use TDD?
--
* In exploratory research (SPIKE), UI/UX-heavy design 
* where the "look" matters more than logic, 
* or legacy systems where the architecture is too coupled to support easy testing.

----

# Relevant Aspects 

* London vs. Chicago School: 
	* "***London***" (Mockist - top-down from Controller) vs. 
	* "***Chicago***" (Classicist - bottom-up from Domain).
* *Mutation Testing*: 
	* ***PITest*** to check if your tests are actually effective (by "mutating" your code and seeing if tests fail).
* ***AssertJ*** & ***Hamcrest***: 
	* Use these for more readable assertions (e.g., assertThat(x).isEqualTo(y)).


----

# TDD Red-Green-Refactor exercise. 

* The goal is to build a Discount Service for an e-commerce platform using the Red-Green-Refactor cycle.

The Requirement
--
* Feature: Apply a 10% discount to orders over $100.
* Input: A double amount.
* Output: The discounted amount.


Step 1: RED (The Failing Test)
--
* We write the test first, even though the class DiscountService doesn't exist yet.
---

	class DiscountServiceTest {

		@Test
		void shouldApplyTenPercentDiscountWhenAmountIsOverOneHundred() {
			DiscountService service = new DiscountService();
			double result = service.applyDiscount(120.0);
			
			// 120 - 10% = 108
			assertThat(result).isEqualTo(108.0);
		}
	}
---

Status: 🔴 Fails to compile (Class DiscountService not found).


Step 2: GREEN (The Minimum Code)
--
* Write just enough code to make the test pass. 
* Don't worry about other cases yet.

---
	public class DiscountService {
		public double applyDiscount(double amount) {
			if (amount > 100) {
				return amount * 0.9;
			}
			return amount;
		}
	}
---

Status: 🟢 Passes.


Step 3: REFACTOR (Cleanup)
--
* We look at the code. It’s simple, but maybe we want to avoid "magic numbers" like 0.9 or 100.
---
	public class DiscountService {
		private static final double THRESHOLD = 100.0;
		private static final double DISCOUNT_RATE = 0.10;

		public double applyDiscount(double amount) {
			if (amount > THRESHOLD) {
				return amount - (amount * DISCOUNT_RATE);
			}
			return amount;
		}
	}
---

Status: 🟢 Still Passes. (Regression testing).

Step 4: REPEAT (The Next Requirement)
--
* What if the amount is exactly $100? 
* The current logic returns $100. Let's add a test for that.

---
	@Test
	void shouldNotApplyDiscountWhenAmountIsExactlyOneHundred() {
		DiscountService service = new DiscountService();
		double result = service.applyDiscount(100.0);
		
		assertThat(result).isEqualTo(100.0);
	}
---

Status: 🟢 Already Passes. No new code needed (this is a "Goldilocks" test).


Step 5: Handling Edge Cases (Exceptions)
--
*  What if the amount is negative?
   1. RED: Write a test expecting an IllegalArgumentException.
   2. GREEN: Add if (amount < 0) throw new IllegalArgumentException();.
   3. REFACTOR: Ensure the error message is clear.

Takeaways
--
* Keep tests small: Test one behavior at a time.
* Don't skip the "Refactor" step: It’s where you prevent Technical Debt.
* Name your tests well: Use descriptive names like shouldReturnFullPriceWhenAmountIsBelowThreshold.


----

# Mocking  

* In a real-world Spring Boot application, the discount rate wouldn't be hardcoded; 
* it would come from a database or a configuration service. 
* This is where Mockito becomes essential to isolate your test.

The Requirement
--
* Feature: The DiscountService must fetch the active discount percentage from a ConfigurationRepository.
* Constraint: If the repository is down or returns zero, no discount is applied.

Step 1: RED (The Failing Test with Mocks)
--
* We define the dependency (ConfigurationRepository) and tell the test how it should behave using when(...).thenReturn(...).

---
	import org.junit.jupiter.api.Test;import 
	org.junit.jupiter.api.extension.ExtendWith;import 
	org.mockito.InjectMocks;import org.mockito.Mock;import 
	org.mockito.junit.jupiter.MockitoExtension;
	import static org.mockito.Mockito.*;
	import static org.assertj.core.api.Assertions.assertThat;

	@ExtendWith(MockitoExtension.class) // Initializes @Mock and @InjectMocks
	class DiscountServiceTest {

		@Mock
		private ConfigurationRepository repository;

		@InjectMocks
		private DiscountService service;

		@Test
		void shouldApplyDynamicDiscountFromRepository() {
			// Arrange: Tell the mock to return 20% (0.20)
			when(repository.getActiveDiscountRate()).thenReturn(0.20);

			// Act
			double result = service.applyDiscount(200.0);

			// Assert: 200 - 20% = 160
			assertThat(result).isEqualTo(160.0);
			
			// Verify: Ensure the repository was actually called
			verify(repository, times(1)).getActiveDiscountRate();
		}
	}
---

Status: 🔴 Fails to compile (No ConfigurationRepository or constructor in DiscountService).

Step 2: GREEN (The Minimum Code)
--
* We create the interface and update the service to use Constructor Injection.
---
	// The Dependency
	interface ConfigurationRepository {
		double getActiveDiscountRate();
	}
	// The Service
	public class DiscountService {
		private final ConfigurationRepository repository;

		public DiscountService(ConfigurationRepository repository) {
			this.repository = repository;
		}

		public double applyDiscount(double amount) {
			double rate = repository.getActiveDiscountRate(); // Fetch from mock
			if (amount > 100.0) {
				return amount * (1 - rate);
			}
			return amount;
		}
	}
---
Status: 🟢 Passes.

Step 3: REFACTOR (The Edge Case)
--
* What if the repository returns a negative value or throws an exception? 
* In TDD, you would:
   1. Write a test where repository.getActiveDiscountRate() throws a RuntimeException.
   2. Update applyDiscount with a try-catch to return the original amount as a fallback.

💡 "Watch-Outs"
--
* Don't Mock what you don't own: 
	* Avoid mocking standard Java libraries (like List or LocalDate). 
	* Mock your own services and repositories.
* Verification: 
	* Only use verify(...) if the side effect (like saving to a DB or sending an email) is part of the requirement. 
	* For simple calculations, checking the return value is usually enough.
* Constructor Injection: 
	* Always prefer this over @Autowired on fields. 
	* It makes unit testing much easier because you can pass mocks manually in the constructor.

----

# TDD Critical Pillars  

* While Red-Green-Refactor is the "engine" of TDD, it is just the mechanical part. 
* there are other philosophical and architectural aspects that make it work in a professional environment.

1. The "***Double Loop***" (Inner vs. Outer)
--
* In Spring Boot, you don't just test one method. You use two loops:
	* Outer Loop (Acceptance/Integration): 
		* Write a failing test for a high-level feature (e.g., a REST API call). 
		* This stays Red for a long time.
	* Inner Loop (Unit): 
		* To make the API work, you write many small unit tests for Services and Models. 
		* This is where you do the fast Red-Green-Refactor.
	* Goal: 
		* Once all unit tests (Inner) pass, the API test (Outer) finally turns Green.

2. Behavioral over Structural Testing
--
* A common mistake is testing how the code works (private methods, internal variables).

* Fundamental: 
	* Test what the code does (Input -> Output).
	* Why? 
		* If you test internal implementation, your tests will break every time you refactor, even if the result is the same. 
		* TDD should enable refactoring, not hinder it.

3. Dependency Injection & Testability
--
* TDD forces you to ***write better code***. 
	* If a class is hard to test, it usually means:
		* It has too many responsibilities (**Single Responsibility Principle violation**).
		* It is "*tightly coupled*" to other classes.
		* Result: 
			* TDD pushes you toward Constructor Injection and Interfaces, making the system modular.

4. Triangulation
--
* This is a strategy to ensure your logic is correct.
	* Don't just write one test for "10% discount."
	* Write one for $120 -> $108.
	* Then write another for $200 -> $180.
	* Triangulation ensures you didn't just hardcode the result for the first test case.

5. The "Transformation Priority Premise" (TPP)
--
* When moving from Red to Green, you should make the simplest possible change. 
* There is a hierarchy of complexity:
   1. Constant: Return 10.
   2. Scalar: Use a variable amount * 0.1.
   3. Statement: Add an if.
   4. Loop: Add a for.
* Rule: 
	* Always try to stay as high up the list as possible to keep the code simple.

6. Social Aspects: Documentation & Confidence
--
* Living Documentation: 
	* In TDD, the test suite is the most up-to-date documentation of how the system is supposed to behave.
* Fearless Refactoring: 
	* Because you have 100% coverage of the logic you just wrote, you can change the architecture 
	* (e.g., move from Monolith to Microservices) with the confidence that the business rules still work.

Summary :
--
* ***F.I.R.S.T Principles***: 
	* Fast, Independent, Repeatable, Self-Validating, Timely.
* ***A.A.A Pattern***: 
	* Arrange (setup), Act (execute), Assert (verify).
* ***Mocking Strategy***: 
	* Use Mocks for external boundaries (DB, API); use Real Objects for internal domain logic.
* ***Test Coverage*** vs. ***Test Quality***: 
	* 100% coverage is useless if the assertions are weak.

----

# TDD "schools of thought" 

London vs. Chicago Schools (The Styles of TDD)
--

* The London School (Mockist/Top-Down):
	* Approach: 
		* Start from the Controller or API and work your way down.
	* Strategy: 
		* You mock every dependency. If you're testing a Service, you mock the Repository. 
		* If you're testing a Controller, you mock the Service.
	* Pros: 
		* Great for Spring Boot because it aligns with dependency injection. 
		* It forces you to define interfaces early.
	* Cons: 
		* You can end up with "brittle" tests that break if you change a method signature, even if the logic is fine.

* The Chicago School (Classicist/Bottom-Up):
	* Approach: 
		* Start from the Domain Model (the "heart" of the app) and work up to the API.
	* Strategy: 
		* Avoid mocks unless necessary (like for a real Database or External API). 
		* You use real objects for your dependencies.
	* Pros: 
		* Tests are very robust and focus on final results. 
		* Refactoring internal logic is easier because mocks don't "lock" the structure.
	* Cons: 
		* A bug in a low-level class can cause 100 tests to fail simultaneously, making it harder to find the root cause.

Tip: 
--
* Say you prefer a pragmatic mix. 
* Use London for the outer layers (Web/API) and Chicago for the core business logic.

----

# The Testing Pyramid & Coverage Percentages
--
* There is no "perfect" number, but the industry standard follows the Testing Pyramid to ensure the suite stays fast.

* Unit Tests (The Base)
	* Target Coverage: 
		* 80% - 90%
	* Why: 
		* These are the cheapest and fastest. 
		* You should cover almost every "if/else" and edge case here.
	* Focus: 
		* Business rules, calculations, and data transformations.

* Integration Tests (The Middle)
	* Target Coverage: 
		* 15% - 20%
	* Why: 
		* These are slower and heavier. 
		* They test if your code actually talks to the Database (using Testcontainers/H2) or External APIs.
	* Focus: 
		* Repository queries, Security filters, and API contracts.

* End-to-End (E2E) Tests (The Top)
	* Target Coverage: 
		* 5% - 10%
	* Why: 
		* Extremely slow and "flaky" (prone to failing due to network issues). 
		* They test the "Happy Path" from the user's perspective.
	* Focus: 
		* Critical user journeys (e.g., "User can login, add item to cart, and checkout").


3. The "100% Coverage" Trap
--
* "Is 100% coverage the goal?", the correct answer is No.

* Diminishing Returns: 
	* Getting from 90% to 100% usually involves testing trivial code (like getters/setters or simple constructors) that doesn't provide real value.
* False Security: 
	* High coverage doesn't mean high quality. 
	* You can have 100% coverage with weak assertions (tests that pass but don't actually check the result).

Summary 
--
* "Mutation Testing" (PITest): 
	* It’s the only way to prove your 80% coverage is actually "killing bugs."
* Value over Volume: 
	* Focus on covering complex logic rather than total line count.
* The "Double Loop": 
	* Use the London style for the Outer Loop (Acceptance) and 
	* Chicago/Classicist for the Inner Loop (Unit).

----

# @DataJpaTest

* In a Spring Boot TDD strategy, @DataJpaTest is the standard tool for the Integration Testing layer (the middle of the pyramid). 
* It provides a way to test your data layer without starting the full application context.

1. What does @DataJpaTest actually do?
--
	* Selective Loading: 
		* It only loads @Repository, EntityManager, and DataSource. 
		* It ignores @Service, @Controller, and other beans, making it much faster than @SpringBootTest.
	* Transactional by Default: 
		* Every test runs in its own transaction and rolls back at the end. 
		* Your database stays clean for the next test.
	* In-Memory Database: 
		* By default, it looks for an in-memory DB (like H2) on the classpath.


2. TDD Example: Custom Query
--
* Imagine you need a feature to find users by their email domain (e.g., all @company.com users).

* Step 1: RED (The Test)
---
	@DataJpaTestclass UserRepositoryTest {

		@Autowired
		private UserRepository userRepository;

		@Test
		void shouldFindUsersByEmailDomain() {
			// Arrange
			userRepository.save(new User("Alice", "alice@gmail.com"));
			userRepository.save(new User("Bob", "bob@work.com"));

			// Act
			List<User> result = userRepository.findByEmailContaining("@work.com");

			// Assert
			assertThat(result).hasSize(1);
			assertThat(result.get(0).getName()).isEqualTo("Bob");
		}
	}
---

* Step 2: GREEN (The Repository)
---
	@Repository
	public interface UserRepository extends JpaRepository<User, Long> {
		List<User> findByEmailContaining(String domain);
}
---


3. Key Trade-offs: H2 vs. Testcontainers
--
* When using @DataJpaTest, the will likely ask: "Do you use H2 or a real database?"

| Strategy | Pros | Cons |
|---|---|---|
| H2 (In-Memory) | Extremely fast; No setup required. | Compatibility risk. Some Postgres/Oracle features (JSONB, specific functions) don't work in H2. |
| Testcontainers | Identical to Prod. Tests run against a real Docker instance of your DB. | Slower (adds ~10s to start Docker); Requires Docker on the machine. |

Tip: 
* say: "I use @DataJpaTest with Testcontainers for complex queries to ensure the SQL is 100% compatible with production, 
* but I use H2 for simple CRUD logic to keep the build fast."

4. Summary: When to use what?
--
* Use Mockito (Unit Test): 
	* To test the Service logic. 
	* You mock the Repository because you don't care about the SQL; 
	* you only care that save() was called.
* Use @DataJpaTest (Integration Test): 
	* To test the Repository. 
	* You want to ensure your @Query or Method Name generates the correct SQL and returns the right data.


----

# How to configure Testcontainers with @DataJpaTest


1. The Strategy: @ServiceConnection
--
* In modern Spring Boot (3.1+), the simplest way to integrate Testcontainers with @DataJpaTest is using the ***@ServiceConnection*** annotation. 
* It automatically configures the datasource properties (URL, username, password) for you.

2. Code Example (PostgreSQL)
--
* Dependencies (Maven):
	* testcontainers, 
	* junit-jupiter, and 
	* postgresql.
	
The Test Class:
---
	@DataJpaTest
	@Testcontainers // Starts/Stops Docker containers automatically
	@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // Don't use H2!
	class ProductRepositoryTest {

		// Define the container (Postgres image)
		@Container
		@ServiceConnection // Magic: Maps the container port/host to Spring properties
		static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

		@Autowired
		private ProductRepository repository;

		@Test
		void shouldSaveAndRetrieveProduct() {
			// Red Step: Write the test for a specific JPA behavior
			Product p = new Product("Laptop", "Electronics");
			repository.save(p);

			Optional<Product> found = repository.findByName("Laptop");

			assertThat(found).isPresent();
			assertThat(found.get().getCategory()).isEqualTo("Electronics");
		}
	}
---

3. Why not just use H2?:
--
   1. SQL Dialect Gaps: 
		* H2 doesn't support specific features like *JSONB*, *PostGIS*, *Window Functions*, or 
		* complex Native Queries exactly like Postgres does.
   2. Schema Migration Validation: 
		* You can run your Flyway or Liquibase migrations against the Testcontainer. 
		* This proves your production scripts actually work before you deploy.
   3. No "Works on my machine": 
		* If it passes on the container, it passes on the CI/CD pipeline and in Production.


4. The "Double Loop" Architecture Summary
--
* remember this hierarchy:

| Level | Tool | Strategy |
|---|---|---|
| Service (Unit) | Mockito | Mock the Repository. Focus on Business Logic. |
| Repo (Integration) | @DataJpaTest + Testcontainers | Real DB. Focus on Data Integrity/SQL. |
| API (Web) | MockMvc + @WebMvcTest | Mock the Service. Focus on JSON & HTTP Status. |
| System (E2E) | RestAssured + @SpringBootTest | Full context. Focus on End-to-End flow. |

🚀 Final Checklist :
--
	* TDD Goal: 
		* Design better code, not just find bugs.
	* Refactoring: 
		* Never skip it; it's where you remove technical debt.
	* Mocking: 
		* Mock what you don't control (external APIs, DB); 
		* test what you own (Domain).

----

# The Testing Pyramid match on SpringBootTest

* The Testing Pyramid is a strategic framework that tells you how many tests of each type you should write to create a fast, maintainable, and reliable suite.
* In a Spring Boot environment, the pyramid is essential because it prevents 
	* the "Ice Cream Cone" anti-pattern (too many slow, flaky end-to-end tests and not enough unit tests).


1. The Pyramid Structure & Spring Boot Mapping
--
Level 1: Unit Tests (The Base - 70-80%)
	* Concept: 
		* Test a single class in isolation. 
		* No Spring Context, no Database, no Network.
	* Spring Boot Match: 
		* JUnit 5 + Mockito.
	* Strategy: 
		* Use @ExtendWith(MockitoExtension.class). 
		* Mock all dependencies.
	* Why: 
		*They run in milliseconds. 
		* You can test 100 edge cases (nulls, empty strings, exceptions) instantly.

Level 2: Integration Tests (The Middle - 15-20%)
	* Concept: 
		* Test the interaction between two or more components 
		* (e.g., Service + Database or Controller + Service).
	* Spring Boot Match: 
		* Slicing Annotations (@DataJpaTest, @WebMvcTest, @JsonTest).
	* Strategy: 
		* Load only a slice of the Spring Context. 
		* Use Testcontainers for real database interaction.
	* Why: 
		* They verify "contracts" 
		* (Does the SQL query work? Does the JSON serialize correctly?). 
		* They are slower because they start a partial Spring context.

Level 3: End-to-End / System Tests (The Top - 5-10%)
	* Concept: 
		* Test the entire application from the user's perspective, 
		* from the HTTP request to the Database and back.
	* Spring Boot Match: 
		* @SpringBootTest + RestAssured / WebTestClient.
	* Strategy: 
		* Use @SpringBootTest(webEnvironment = RANDOM_PORT). 
		* This starts the entire application, including all Beans, Security, and Filters.
	* Why: 
		* They provide the highest confidence but are very slow and "brittle" 
		* (a small change in a secondary service can break the whole test).

2. How they match: A Practical Comparison
--
| Feature | Unit Test | Integration Test (Slicing) | E2E / System Test |
|---|---|---|---|
| Spring Annotation | None (Just JUnit/Mockito) | @WebMvcTest / @DataJpaTest | @SpringBootTest |
| Context Loaded | None | Partial (Only Web or JPA) | Full (Everything) |
| Speed | ⚡ Fast (ms) | 🐢 Medium (seconds) | 🐌 Slow (minutes) |
| Database | Mocked | Real (Testcontainers/H2) | Real (Testcontainers/Prod-like) |
| Mocking | Everything except the class | Mocks the layers outside the slice | Usually no mocks (Black box) |


3. The "Double Loop" TDD Connection
--
* the Pyramid supports the Double Loop:
   1. Outer Loop (E2E/Integration): 
		* You write a high-level test (e.g., @WebMvcTest) that fails because the feature doesn't exist.
   2. Inner Loop (Unit): 
		* You write multiple small Unit Tests (JUnit/Mockito) to build the business logic piece by piece.
   3. Result: 
		* When the Unit tests pass (Base of the pyramid), the Integration test (Middle of the pyramid) eventually passes too.


4. Takeaway: The Cost of Testing
--
why we don't just use @SpringBootTest for everything:
	* The cost of a test isn't just writing it; 
	* it's maintaining it and the time it takes to run. 
	* A suite of 1,000 unit tests takes 10 seconds; 1,000 @SpringBootTest could take 30 minutes. 
	* This slows down the CI/CD pipeline and discourages developers from running tests frequently.


----

# Handle External REST APIs using WireMock

* When testing a Spring Boot application that consumes an External REST API (e.g., a Weather API or a Payment Gateway), you cannot rely on the actual third-party service during tests. 
* It would be slow, cost money, and fail if the internet is down.
* The standard tool for this in the Testing Pyramid is WireMock. 
* It acts as a "Mock Server" that runs on a local port and returns predefined JSON responses.

1. Where does it fit in the Pyramid?
--
	* Unit Level (Base): 
		* You Mock the RestTemplate or WebClient bean using Mockito. (Fastest).
	* Integration Level (Middle): 
		* Use WireMock. 
		* You test your actual HttpClient code against a real HTTP server (WireMock). 
		* This is the "Sweet Spot."
	* E2E Level (Top): 
		* WireMock is used to simulate the external dependency so the entire internal flow can complete.

2. Integration Test Example: WireMock + WebClient
--

* Scenario: Your WeatherService calls an external API at ://api.weather.com.
	* Step 1: Add Dependency (Maven)
---
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-wiremock</artifactId>
    <scope>test</scope>
</dependency>
---

	* Step 2: The Test Code
---
	@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
	@AutoConfigureWireMock(port = 0) // Starts WireMock on a random port
	class WeatherServiceIntegrationTest {

		@Autowired
		private WeatherService weatherService;

		@Test
		void shouldReturnWeatherFromExternalApi() {
			// 1. Arrange: Stub the external API behavior
			stubFor(get(urlEqualTo("/v1/forecast?city=Lisbon"))
				.willReturn(aResponse()
					.withStatus(200)
					.withHeader("Content-Type", "application/json")
					.withBody("{\"temp\": 25, \"status\": \"Sunny\"}")));

			// 2. Act: Call your internal service
			WeatherResponse response = weatherService.getForecast("Lisbon");

			// 3. Assert
			assertThat(response.getTemp()).isEqualTo(25);
			assertThat(response.getStatus()).isEqualTo("Sunny");
		}
	}
---

3. Key Trade-offs: Mockito vs. WireMock

* Why use WireMock if I can just Mock the Service with Mockito?

| Feature | Mocking the Client (Mockito) | Mocking the Server (WireMock) |
|---|---|---|
| What is tested? | Only your Java logic. | Your logic + JSON Parsing + HTTP Headers + Timeouts. |
| Realism | Low (No actual network call). | High (A real HTTP call is made to localhost). |
| Use Case | Fast Unit Tests. | Integration Tests to ensure your DTOs match the external JSON. |


4. Advanced TDD: Testing "Failure Resilience"
--
* One of the best uses for WireMock in TDD is testing Error Handling. 
* In your test, you can easily simulate:
	* Slow Response: .withFixedDelay(5000) to test your Timeouts.
	* Server Error: .withStatus(500) to test your Retry logic or Circuit Breaker (Resilience4j).
	* Bad Data: Return a malformed JSON to test your Exception Handling.

🚀 Summary of the "Java/Spring TDD" Prep:
--
   1. Red-Green-Refactor: Always start with a failing test.
   2. Double-Loop: Use MockMvc for the outer loop and Mockito for the inner loop.
   3. Data Layer: Use @DataJpaTest + Testcontainers for real SQL validation.
   4. External APIs: Use WireMock to simulate third-party dependencies.
   5. Clean Code: TDD is a design tool that results in decoupled, modular code.


----

# @InjectMocks & @Mock

* In the context of Spring Boot (SB) unit testing (specifically using Mockito), these two annotations work together to set up your test environment.
* The simplest way to remember it: 
	* ***@Mock*** creates the "fake" dependencies, and 
	* ***@InjectMocks*** creates the actual class you are testing and "plugs" those fakes into it.
	
1. @Mock
--
* This creates a simulated instance (a proxy) of a class or interface. 
* It has no real logic; it’s a hollow shell.

	* Purpose: 
		* To replace real dependencies (like a Database Repository or an External API Service) so you don't actually hit the DB or network during a test.
	* Behavior: 
		* By default, its methods do nothing and return null (or 0/false) unless you explicitly tell them what to do using when(...).thenReturn(...).

2. @InjectMocks
--
* This creates a real instance of the class you want to test (the "System Under Test").

	* Purpose: 
		* It tells Mockito: "Create this object, and then look for any fields inside it that match the @Mock objects I defined in this test file. If you find a match, inject the mock into this object."
	* How it works: 
		* It tries to inject mocks via Constructor, then Setter, and finally Field injection.

The Difference in a Real Scenario
--
* Imagine you are testing a UserService that needs a UserRepository.
---
	@ExtendWith(MockitoExtension.class)
	class UserServiceTest {

		@Mock
		private UserRepository userRepository; // The "Fake" DB

		@InjectMocks
		private UserService userService;    // The "Real" Service being tested

		@Test
		void testGetUser() {
			// 1. Tell the @Mock how to behave
			when(userRepository.findById(1L)).thenReturn(Optional.of(new User("John")));

			// 2. Call the @InjectMocks instance
			User result = userService.getUserDetails(1L);

			// 3. Verify logic
			assertEquals("John", result.getName());
		}
	}
---

Key Differences Summary
--

| Feature | @Mock | @InjectMocks |
|---|---|---|
| Object Type | A fake/dummy proxy. | A real instance of the class. |
| Logic | None (returns null by default). | Executes the actual code in the class. |
| Role | Represents a dependency. | Represents the target of the test. |
| Analogy | A "stunt double" for an actor. | The Director who uses the stunt double. |

Important Note for Spring Boot
--
* If you are doing an Integration Test (using @SpringBootTest), you should use @MockBean instead of @Mock. 
* @MockBean replaces a bean in the actual Spring Application Context, whereas @Mock is just a plain Mockito object used in fast, isolated Unit Tests.


