# Spring Data JPA – Simple Notes

> Short and easy to follow. Each topic: **idea → small code → output → one-liner.**
> Sections 1–11 = concepts · 12 = extras · 13 = common mistakes · 14 = interview answers

## Contents
1. [Big Picture](#1-big-picture)
2. [Project Setup](#2-project-setup)
3. [Entity Basics](#3-entity-basics)
4. [Relationships, Fetch & Cascade](#4-relationships-fetch--cascade)
5. [Repositories, Derived Queries, Pagination](#5-repositories-derived-queries-pagination)
6. [Custom Queries (JPQL, Native, Projections, Specifications)](#6-custom-queries)
7. [N+1 Problem](#7-n1-problem)
8. [Entity Lifecycle & Dirty Checking](#8-entity-lifecycle--dirty-checking)
9. [Transactions](#9-transactions)
10. [Locking](#10-locking)
11. [Caching](#11-caching)
12. [Extras (Auditing, Soft Delete)](#12-extras)
13. [Common Mistakes Cheat Sheet](#13-common-mistakes-cheat-sheet)
14. [Interview Q&A](#14-interview-qa)

---

## 1. Big Picture

| | What it is | Its job |
|---|---|---|
| **JPA** | A *specification* (interfaces + rules) | Defines how Java objects map to tables. Can't run on its own. |
| **Hibernate** | A JPA *implementation* | Actually generates and runs the SQL |
| **Spring Data JPA** | A layer on top of JPA | Writes the repository code for you |

Same idea as `List` (interface) and `ArrayList` (implementation).

```
Your code → Spring Data JPA → JPA → Hibernate → JDBC → Database
```

**Before vs after:**

```java
// Plain JPA – lots of plumbing
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
em.persist(user);
em.getTransaction().commit();
em.close();

// Spring Data JPA – one line
userRepository.save(user);
```

**Why use it:** no boilerplate, queries from method names, built-in pagination/sorting, works with any database, auditing and dynamic queries built in.
**When not to:** tiny apps (use `JdbcTemplate`) or heavy reporting queries (use native SQL / jOOQ / MyBatis).

> The `spring-boot-starter-data-jpa` dependency brings all three, plus the HikariCP connection pool and transaction support.

**One-liner:** JPA is the rulebook, Hibernate does the work, Spring Data JPA saves you from writing the repetitive code.

---

## 2. Project Setup

**Dependencies (Maven):**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>   <!-- use org.postgresql:postgresql for PostgreSQL -->
    <scope>runtime</scope>
</dependency>
```

(Gradle: `implementation 'org.springframework.boot:spring-boot-starter-data-jpa'` + `runtimeOnly 'com.mysql:mysql-connector-j'`.) Only add **one** DB driver. H2 is handy for tests.

**`application.properties`:**

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true                              # print SQL in console
spring.jpa.properties.hibernate.format_sql=true       # make it readable

spring.datasource.hikari.maximum-pool-size=10         # optional: HikariCP connection pool
```

- PostgreSQL URL: `jdbc:postgresql://localhost:5432/mydb`
- Driver and dialect are auto-detected, so you don't need to set them.
- ⚠️ **Create the database first** (`CREATE DATABASE mydb;`) or the app fails on startup.
- A **connection pool** keeps ready-made DB connections so each query doesn't open a new one (slow).

**What happens on startup:** read properties → create connection pool → Hibernate scans `@Entity` classes → applies `ddl-auto` → Spring creates your repository implementations.

**`ddl-auto` – what Hibernate does to your tables on startup:**

| Value | Behaviour | Use in |
|---|---|---|
| `create` | Drops and recreates all tables (**data lost**) | Throwaway local dev |
| `create-drop` | Create on start, drop on shutdown | Tests |
| `update` | Adds new tables/columns, never deletes anything | Local dev / learning |
| `validate` | Checks entities match the schema, fails if not | Staging |
| `none` | Does nothing | **Production** |

> ⚠️ Never use `create` or `update` in production. Use `none` or `validate` with a migration tool (**Flyway** / **Liquibase**) that tracks every schema change as a versioned script.

**One-liner:** Add the starter + one driver, point to your DB, and use `update` for learning but `none` + Flyway in production.

---

## 3. Entity Basics

An **entity** = a Java class mapped to a table. **Object = row, field = column.**

```java
@Entity
@Table(name = "users")                       // set the name explicitly ("user" is a reserved word)
@Getter @Setter @NoArgsConstructor           // JPA needs an empty constructor
public class User {

    @Id                                                  // primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // DB auto-generates 1, 2, 3...
    private Long id;

    @Column(name = "first_name", nullable = false, length = 50)
    private String firstName;

    @Column(nullable = false, unique = true)
    private String email;

    @Enumerated(EnumType.STRING)             // store "ACTIVE", not 0
    private UserStatus status;

    @Column(updatable = false)               // can't change after first insert
    private LocalDateTime createdAt;

    @Transient                               // NOT saved in DB
    private String tempToken;
}
```

| Annotation | Meaning |
|---|---|
| `@Entity` | This class is a table (JPA ignores it otherwise) |
| `@Table(name=…)` | Table name. Use lowercase plural snake_case; avoid reserved words (`user`, `order`, `group`) |
| `@Id` | Primary key – every entity needs exactly one |
| `@GeneratedValue` | ID is generated for you |
| `@Column` | Column name, `nullable`, `unique`, `length`, `updatable` |
| `@Enumerated(STRING)` | Save enum as text |
| `@Transient` | Skip this field |

> ⚠️ **Enum default is `ORDINAL` (a number).** If you later insert a new constant in the middle, every stored number now means the wrong thing. **Always use `EnumType.STRING`.**

**ID generation strategies:**

| Strategy | How | Best for |
|---|---|---|
| `IDENTITY` | DB auto-increment | MySQL, simple apps. Disables Hibernate batch inserts |
| `SEQUENCE` | DB sequence; Hibernate grabs IDs in blocks (`allocationSize`) | PostgreSQL/Oracle, bulk inserts (fastest) |
| `UUID` | Random unique ID | Microservices / distributed systems (bigger index) |
| `TABLE` | Separate table tracks the ID | Legacy only – slowest, avoid |

**One-liner:** `@Entity` + `@Id` + an empty constructor makes a table; use `STRING` enums and pick `IDENTITY` (MySQL) or `SEQUENCE` (PostgreSQL).

---

## 4. Relationships, Fetch & Cascade

### Owner side vs inverse side (understand this first)
- **Owner side** = the entity whose table holds the **foreign key** (uses `@JoinColumn`).
- **Inverse side** = the other entity; uses `mappedBy = "fieldNameOnOwner"` and creates no column.
- `@ManyToOne` is **always the owner** (the FK lives in the "many" table).

### @OneToMany / @ManyToOne (most common)
One `Department` has many `Employee`s.

```java
@Entity
public class Department {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();   // initialise to avoid NPE

    // helper methods keep BOTH sides in sync
    public void addEmployee(Employee e) { employees.add(e); e.setDepartment(this); }
    public void removeEmployee(Employee e) { employees.remove(e); e.setDepartment(null); }
}

@Entity
public class Employee {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)          // override the EAGER default!
    @JoinColumn(name = "department_id", nullable = false)   // the FK column
    private Department department;
}
```

```
departments            employees
| id | name        |   | id | name  | department_id |
| 1  | Engineering |   | 1  | Alice | 1             |
                       | 2  | Bob   | 1             |
```

> ⚠️ Set both sides (`addEmployee()` does it). Only setting the list side means the FK may not get saved.

### @OneToOne
Each `User` has one `UserProfile`. Owner (has the FK) = `User`.

```java
// in User (owner)
@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY, orphanRemoval = true)
@JoinColumn(name = "profile_id")                // FK column in users table
private UserProfile profile;

// in UserProfile (inverse)
@OneToOne(mappedBy = "profile")
private User user;
```

### @ManyToMany
Students ↔ Courses need a **join table**.

```java
// Student (owner – defines the join table)
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
@JoinTable(name = "student_courses",
           joinColumns = @JoinColumn(name = "student_id"),
           inverseJoinColumns = @JoinColumn(name = "course_id"))
private Set<Course> courses = new HashSet<>();     // Set, not List

// Course (inverse)
@ManyToMany(mappedBy = "courses")
private Set<Student> students = new HashSet<>();
```

```
student_courses (join table)
| student_id | course_id |
| 1          | 1         |   Alice → Math
| 1          | 2         |   Alice → Science
| 2          | 1         |   Bob   → Math
```

- Use `Set` (no duplicates, more efficient deletes than `List`).
- ⚠️ **Never `REMOVE`/`ALL` on `@ManyToMany`.** Deleting Alice would try to delete "Math", which Bob still uses.
- Need extra columns in the join table (e.g., `grade`)? Make a separate `Enrollment` entity with two `@ManyToOne`s.

### Fetch types – *when* related data loads

| Annotation | Default | What to do |
|---|---|---|
| `@ManyToOne` | **EAGER** ⚠️ | Change to `LAZY` |
| `@OneToOne` | **EAGER** ⚠️ | Change to `LAZY` |
| `@OneToMany` | LAZY | Keep |
| `@ManyToMany` | LAZY | Keep |

- **EAGER** = load related data immediately (even if unused). Chains of EAGER relations can load half your DB.
- **LAZY** = load only when you first access it. Hibernate puts a **proxy** (placeholder) there until then.

```java
Department d = departmentRepository.findById(1L).get();  // SQL: select ... from departments where id=1
d.getEmployees().size();                                 // SQL runs NOW: select ... from employees where department_id=1
```

> ✅ **Best practice:** make every relationship `LAZY`, and when you need the related data use `JOIN FETCH` / `@EntityGraph` (Section 7).
> ⚠️ **`LazyInitializationException`** = you touched a lazy field *after* the transaction/session closed. Fix: load what you need **inside** the `@Transactional` method (or `JOIN FETCH`).

### Cascade – pass an operation from parent to children

| Type | Effect |
|---|---|
| `PERSIST` | Saving parent saves new children |
| `MERGE` | Updating parent updates children |
| `REMOVE` | Deleting parent deletes children |
| `ALL` | All of the above (+ refresh, detach) |

- Use `ALL` only when the parent **fully owns** the children (Order → OrderItems).
- **`orphanRemoval = true`**: removing a child *from the parent's list* deletes it from the DB. (`REMOVE` only fires when the *parent* is deleted.)

```java
department.getEmployees().remove(employee);   // with orphanRemoval → DELETE FROM employees WHERE id = ?
                                              // without it → row stays in DB as an "orphan"
```

**One-liner:** FK side = owner (`@JoinColumn`), other side = `mappedBy`; make everything LAZY; cascade only when the parent owns the child; never cascade REMOVE on many-to-many.

---

## 5. Repositories, Derived Queries, Pagination

### Creating a repository
Just declare an interface. Spring generates the implementation at startup (using a proxy). You never write the class.

```java
public interface UserRepository extends JpaRepository<User, Long> { }
//                                                    Entity  ID type
```

`JpaRepository` is the one to extend. It includes `CrudRepository` (basic CRUD) and paging/sorting, plus JPA extras like `flush()` and batch deletes.

### Free built-in methods

| Task | Methods |
|---|---|
| Create / update | `save()`, `saveAll()`, `saveAndFlush()` |
| Read | `findById()` → `Optional`, `findAll()`, `findAllById()`, `existsById()`, `count()` |
| Delete | `deleteById()`, `delete()`, `deleteAll()`, `deleteAllInBatch()` |
| Page / sort | `findAll(Pageable)`, `findAll(Sort)` |

```java
User saved = userRepository.save(new User("Alice", "alice@example.com"));
User found = userRepository.findById(1L).orElseThrow(() -> new RuntimeException("Not found"));
```
```
SQL: insert into users (email, first_name, status) values (?, ?, ?)
SQL: select ... from users where id = ?
```

- **`save()` does insert *or* update:** ID is `null` → INSERT; ID exists → UPDATE.
- **`findById()` returns `Optional`**, so use `.orElseThrow()`.
- `deleteById()` loads the entity first (so cascades work). `deleteAllInBatch()` is one fast `DELETE` but skips cascades and callbacks.
- `save()`/`delete()` are already transactional on their own. Use `@Transactional` on your **service** method to group several calls together.

### Derived queries (query from method name)
Write a method name in the right pattern and Spring builds the query.

```
findByFirstNameAndStatusOrderByCreatedAtDesc
 │      │          │     │        └ ORDER BY created_at DESC
 │      │          │     └ AND status = ?
 │      └──────────┘ WHERE first_name = ?
 └ SELECT ...
```

```java
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);                        // WHERE email = ?
    List<User> findByStatus(UserStatus status);                      // WHERE status = ?
    List<User> findByFirstNameAndStatus(String n, UserStatus s);     // WHERE first_name = ? AND status = ?
    List<User> findByAgeBetween(int min, int max);                   // WHERE age BETWEEN ? AND ?
    List<User> findByFirstNameContaining(String kw);                 // WHERE first_name LIKE '%kw%'
    List<User> findByFirstNameIgnoreCase(String name);               // WHERE LOWER(first_name) = LOWER(?)
    List<User> findByStatusIn(List<UserStatus> statuses);            // WHERE status IN (...)
    List<User> findByVerifiedTrue();                                 // WHERE verified = true
    List<User> findTop5ByStatusOrderByCreatedAtDesc(UserStatus s);   // ... ORDER BY created_at DESC LIMIT 5
    long countByStatus(UserStatus status);                           // SELECT COUNT(*) ...
    boolean existsByEmail(String email);                             // fast true/false check
    void deleteByStatus(UserStatus status);                          // DELETE ... WHERE status = ?
    List<User> findByDepartmentName(String deptName);                // joins through relationship: department.name
}
```

**Common keywords:**

| Keyword | SQL | Keyword | SQL |
|---|---|---|---|
| `And` / `Or` | `AND` / `OR` | `IsNull` / `IsNotNull` | `IS NULL` / `IS NOT NULL` |
| `Not` | `!=` | `Containing` | `LIKE %x%` |
| `Between` | `BETWEEN` | `StartingWith` / `EndingWith` | `LIKE x%` / `LIKE %x` |
| `LessThan` / `GreaterThan` | `<` / `>` | `In` / `NotIn` | `IN (...)` / `NOT IN (...)` |
| `Before` / `After` | `<` / `>` (dates) | `True` / `False` | `= true` / `= false` |
| `OrderBy…Asc/Desc` | `ORDER BY` | `Top` / `First` | `LIMIT` |
| `IgnoreCase` | `LOWER()` compare | `countBy` / `existsBy` / `deleteBy` | count / exists / delete |

- Field names are checked **at startup**. A typo like `findByEmali` stops the app with "No property 'emali' found" (good: bugs show up early).
- If a name is ambiguous, use an underscore: `findByDepartment_Name` means `department.name`.
- **Method name getting long (more than ~3–4 keywords)? Switch to `@Query`.**

### Pagination & sorting

| Class | What it is |
|---|---|
| `Pageable` / `PageRequest` | Input: page number + size + sort |
| `Page<T>` | Result with data **and** totals (`totalPages`, `totalElements`). Runs an extra `COUNT` query |
| `Slice<T>` | Data + "is there a next page?" only. **No count query** → faster (good for infinite scroll / "Load more") |
| `Sort` | Which field(s), ascending/descending |

```java
// Repository – just add a Pageable parameter (works on derived and @Query methods too)
Page<User> findByStatus(UserStatus status, Pageable pageable);

// Service
Pageable pageable = PageRequest.of(0, 10, Sort.by("createdAt").descending());   // page 0 = FIRST page
Page<User> page = userRepository.findByStatus(UserStatus.ACTIVE, pageable);

page.getContent();        // the 10 users
page.getTotalElements();  // 150
page.getTotalPages();     // 15
```
```
SQL: select ... from users where status=? order by created_at desc limit 10
SQL: select count(*) from users where status=?          <- extra query because it's a Page
```

- Multiple sort fields: `Sort.by("status").ascending().and(Sort.by("firstName").descending())`
- ⚠️ **Page numbers start at 0.** If your frontend sends `page=1` for the first page, subtract 1.
- In a controller you can take `Pageable` directly. Spring fills it from `?page=0&size=10&sort=createdAt,desc`:

```java
@GetMapping("/api/users")
public Page<UserDto> list(Pageable pageable) {
    return userRepository.findAll(pageable).map(UserMapper::toDto);   // return DTOs, not entities
}
```
```json
{ "content": [ {"id":1,"firstName":"Alice"} ], "totalElements": 150, "totalPages": 15,
  "number": 0, "size": 10, "first": true, "last": false }
```

**One-liner:** Extend `JpaRepository`, get CRUD for free, write simple queries as method names, and add a `Pageable` parameter for paging (0-indexed).

---

## 6. Custom Queries

Order Spring uses to find a query for a repository method:
**1) `@Query` on the method → 2) `@NamedQuery` named `Entity.methodName` → 3) derived from the method name.**

### JPQL – SQL-like, but uses **Java class and field names**
```
JPQL: SELECT u FROM User u WHERE u.email = :email      (class User, field email)
SQL : SELECT * FROM users WHERE email = ?               (table users, column email)
```
Benefits: database-independent, checked at startup, handles joins/aggregates/bulk updates that derived queries can't.

```java
// Named parameters (preferred over ?1 – order doesn't matter)
@Query("SELECT u FROM User u WHERE u.email = :email AND u.status = :status")
Optional<User> findActiveByEmail(@Param("email") String email, @Param("status") UserStatus status);

// JOIN across a relationship
@Query("SELECT u FROM User u JOIN u.department d WHERE d.name = :dept")
List<User> findByDept(@Param("dept") String dept);

// JOIN FETCH = also load the related data in the same query (fixes N+1, Section 7)
@Query("SELECT e FROM Employee e JOIN FETCH e.department")
List<Employee> findAllWithDepartment();

// Case-insensitive search
@Query("SELECT u FROM User u WHERE LOWER(u.firstName) LIKE LOWER(CONCAT('%', :name, '%'))")
List<User> search(@Param("name") String name);

// Aggregate
@Query("SELECT COUNT(u) FROM User u WHERE u.status = :status")
long countActive(@Param("status") UserStatus status);

// DTO – fetch only some columns
@Query("SELECT new com.example.dto.UserSummaryDto(u.id, u.firstName, u.email) FROM User u")
List<UserSummaryDto> findSummaries();

// Bulk UPDATE / DELETE
@Modifying(clearAutomatically = true)
@Transactional
@Query("UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
int updateStatus(@Param("oldStatus") UserStatus oldStatus, @Param("newStatus") UserStatus newStatus);
```
```
SQL: update users set status=? where status=?      (returns number of rows changed)
```

> ⚠️ UPDATE/DELETE queries **must** have `@Modifying` (and a transaction), or Spring tries to run them as a SELECT.
> ⚠️ A bulk update bypasses the persistence context, so already-loaded entities are stale. `clearAutomatically = true` clears them so the next read is fresh.

### Native SQL – raw SQL, passed straight to the DB
Use when JPQL can't do it: **window functions, CTEs (`WITH`), DB-specific functions**, or complex reporting queries.

```java
@Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
Optional<User> findByEmailNative(@Param("email") String email);        // uses TABLE/COLUMN names

// With pagination: give a countQuery
@Query(value = "SELECT * FROM users WHERE status = :status",
       countQuery = "SELECT COUNT(*) FROM users WHERE status = :status",
       nativeQuery = true)
Page<User> findByStatusNative(@Param("status") String status, Pageable pageable);
```

- ❌ **Not validated at startup**: a typo only fails at runtime. Test native queries.
- ❌ Ties you to one database.
- Partial columns/joins come back as `Object[]`. Use an **interface projection** instead (alias names must match the getters: `SELECT id AS id, email AS email …`).

| Use JPQL / derived when… | Use native when… |
|---|---|
| Normal CRUD, joins, filters | Window functions, CTEs, DB-specific syntax |
| You want DB-independence | You're reusing an existing SQL query |

### Named queries (rarely used today)
Define the query on the entity, then name the repository method `Entity.methodName`. They're validated at startup, but `@Query` is simpler because it sits right above the method.

```java
@Entity
@NamedQuery(name = "User.findActive", query = "SELECT u FROM User u WHERE u.status = 'ACTIVE'")
public class User { ... }

List<User> findActive();    // in repository: Spring finds "User.findActive"
```

### Projections – fetch only the columns you need
Why: faster, less memory, and you don't leak fields like `passwordHash`.

```java
// 1) Interface-based (default choice) – read-only view
public interface UserSummary {
    Long getId();
    String getFirstName();
    String getEmail();
}
List<UserSummary> findByStatus(UserStatus status);
```
```
SQL: select u.id, u.first_name, u.email from users u where u.status=?    <- only 3 columns
```

```java
// 2) DTO / record – use constructor expression (see JPQL example above) or return the record directly
public record UserDto(Long id, String firstName, String email) {}
List<UserDto> findDtoByStatus(UserStatus status);

// 3) Dynamic – caller picks the type
<T> List<T> findByStatus(UserStatus status, Class<T> type);
List<UserSummary> a = repo.findByStatus(ACTIVE, UserSummary.class);
List<User> b        = repo.findByStatus(ACTIVE, User.class);
```

- ❌ You can't save a projection back; use it for **read-only** endpoints.
- ⚠️ Adding `@Value("#{…}")` to an interface projection makes it an "open projection", which loads the **whole entity** and loses the benefit.

### Specifications – dynamic filters (many optional conditions)
Without them, 4 optional filters would need 16 method-name combinations. Instead, build each filter as a piece and combine.

```java
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> { }

public class UserSpecs {
    public static Specification<User> hasStatus(UserStatus s) {
        return (root, query, cb) -> s == null ? null : cb.equal(root.get("status"), s);   // null = skip
    }
    public static Specification<User> nameContains(String n) {
        return (root, query, cb) -> n == null ? null
                : cb.like(cb.lower(root.get("firstName")), "%" + n.toLowerCase() + "%");
    }
}

// Only the filters the caller actually provided are applied
List<User> result = userRepository.findAll(hasStatus(status).and(nameContains(name)));
```

**One-liner:** Derived queries for simple cases, `@Query` JPQL for most custom ones, native only when JPQL can't do it, projections for lighter reads, Specifications for dynamic filters.

---

## 7. N+1 Problem

**What:** 1 query loads N parents, then **N more queries** load each parent's lazy relation. 1,000 employees = 1,001 queries instead of 1. Your app works fine, it's just slow.

**Why:** lazy loading + touching the relation inside a loop.

```java
List<Employee> employees = employeeRepository.findAll();          // 1 query
for (Employee e : employees) {
    System.out.println(e.getDepartment().getName());              // 1 query PER employee
}
```
```
SQL: select * from employees
SQL: select * from departments where id = 1
SQL: select * from departments where id = 2
SQL: select * from departments where id = 3
... (one per employee)
```

**Detect:** turn on `spring.jpa.show-sql=true` and look for the same SELECT repeating. In tests, use `datasource-proxy` or `p6spy` to count queries.

**Fixes:**

```java
// 1) JOIN FETCH – default choice
@Query("SELECT e FROM Employee e JOIN FETCH e.department")
List<Employee> findAllWithDepartment();

// 2) @EntityGraph – same result, no JPQL needed (good on derived queries)
@EntityGraph(attributePaths = {"department"})
List<Employee> findByStatus(EmployeeStatus status);

// 3) @BatchSize – loads in groups: 100 queries → ~4
@OneToMany(mappedBy = "department")
@BatchSize(size = 30)
private List<Employee> employees;

// 4) SUBSELECT (Hibernate) – one extra query loads ALL children
@OneToMany(mappedBy = "department")
@Fetch(FetchMode.SUBSELECT)
private List<Employee> employees;
```
```
SQL (JOIN FETCH): select e.*, d.* from employees e join departments d on e.department_id = d.id     <- just 1 query
```

| Fix | Queries | Use when |
|---|---|---|
| `JOIN FETCH` | 1 | Default; you're already writing `@Query` |
| `@EntityGraph` | 1 | Keep a derived method, just add eager loading |
| `@BatchSize` | N/batch + 1 | Several collections; quick, low-effort fix |
| `SUBSELECT` | 2 | You always load all parents |

> ⚠️ Don't `JOIN FETCH` several collections at once (duplicate rows / `MultipleBagFetchException`), and don't combine `JOIN FETCH` on a collection with pagination (Hibernate pages in memory). Use `@BatchSize` there.
> ⚠️ `EAGER` is *not* the fix, it just hides N+1 and makes it worse.

**One-liner:** N+1 = one query for the list plus one per row; fix it with `JOIN FETCH` or `@EntityGraph`.

---

## 8. Entity Lifecycle & Dirty Checking

### The persistence context
A **persistence context** is the "workspace" (Hibernate's first-level cache) where JPA keeps the entities it's tracking during one transaction. The **`EntityManager`** is the JPA object that manages it (`persist`, `merge`, `remove`, `find`). Spring Data repositories use it internally, so you rarely touch it. **`flush`** = send pending SQL to the DB (happens automatically before commit and before queries).

### Four states

| State | Meaning | How you get there | Changes auto-saved? |
|---|---|---|---|
| **Transient** | New Java object, JPA doesn't know it | `new User()` | ❌ |
| **Managed** | JPA is tracking it | after `save()`, `findById()` inside a transaction | ✅ |
| **Detached** | Was managed, but the transaction/session ended | after the transaction ends, `detach()`, `clear()` | ❌ |
| **Removed** | Scheduled for deletion | after `delete()` | DELETE runs on commit |

```
new → TRANSIENT --save()--> MANAGED --transaction ends--> DETACHED --save()/merge() in new tx--> MANAGED
                               |
                           delete() --> REMOVED (DELETE on commit)
```

```java
User user = new User();                        // TRANSIENT (plain Java object)
user.setEmail("a@x.com");
User saved = userRepository.save(user);        // MANAGED   -> INSERT
// ... method ends, transaction ends ...       // DETACHED  -> changes to 'saved' are no longer tracked
```

### Dirty checking – automatic UPDATE for managed entities
When Hibernate loads an entity it takes a **snapshot**. At commit it compares the entity with the snapshot, and if a field changed it runs the UPDATE itself. **No `save()` needed.**

```java
@Transactional
public void updateEmail(Long id, String newEmail) {
    User user = userRepository.findById(id).orElseThrow();   // MANAGED + snapshot taken
    user.setEmail(newEmail);                                 // just change the field
}   // commit → Hibernate sees the difference → UPDATE
```
```
SQL: select ... from users where id=?
SQL: update users set email=?, first_name=?, ... where id=?
```

```java
// ❌ No @Transactional → entity is detached right after findById → change is LOST
public void broken(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    user.setEmail("new@x.com");     // no UPDATE ever runs
}
```

- Works only on **managed** entities (inside a transaction). Not on detached or transient ones.
- By default the UPDATE lists **all** columns. Add `@DynamicUpdate` on the entity to include only changed columns.
- To re-attach a detached entity, call `save()` inside a new `@Transactional` method (it does a `merge`).

**One-liner:** Inside a transaction, loaded entities are tracked and changes save automatically; outside one, changes are silently ignored.

---

## 9. Transactions

A **transaction** = a group of DB operations that **all succeed or all roll back**. (Transfer money: debit A *and* credit B, or neither.)

`@Transactional` = Spring opens the transaction before your method, **commits** if it finishes normally, **rolls back** if it throws a `RuntimeException`. (It works through a **proxy** wrapped around your bean.)

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    @Transactional
    public Order createOrder(OrderRequest req) {
        inventoryService.reserve(req.getProductId(), req.getQty());   // 1
        paymentService.charge(req.getPayment());                      // 2 – if this throws...
        return orderRepository.save(new Order(req));                  // 3 – ...step 1 is rolled back too
    }
}
```

**Useful options:**

```java
@Transactional(readOnly = true)                  // reads only → skips dirty checking, faster. Use on all read methods
@Transactional(rollbackFor = Exception.class)    // also roll back on checked exceptions (default = RuntimeException/Error only)
@Transactional(noRollbackFor = EmailException.class)   // keep the transaction even if this exception happens
@Transactional(timeout = 30)                     // roll back if it runs longer than 30 seconds
```

**Propagation – what happens when one transactional method calls another:**

| Type | Behaviour |
|---|---|
| `REQUIRED` (default) | Join the existing transaction, or start one. Use this ~99% of the time |
| `REQUIRES_NEW` | Always start a new one; the outer one pauses. E.g. **audit log that must be saved even if the main work fails** |
| `NESTED` | Savepoint inside the current transaction; only the nested part rolls back |
| `SUPPORTS` | Use a transaction if one exists, otherwise run without |
| `NOT_SUPPORTED` | Pause any transaction and run without |
| `MANDATORY` | Must already be in a transaction, else exception |
| `NEVER` | Must NOT be in a transaction, else exception |

**Rules (because it works through a proxy):**
- ✅ Put it on **public** methods in the **service layer** (not controllers or repositories).
- ❌ **Private methods:** the proxy can't intercept them, so `@Transactional` is silently ignored.
- ❌ **Self-invocation:** a method calling another `@Transactional` method *in the same class* skips the proxy, so no transaction is applied.

```java
@Service
public class UserService {
    public void outer() {
        this.inner();      // ❌ direct call, proxy bypassed → inner()'s @Transactional ignored
    }
    @Transactional
    public void inner() { ... }
}
// Fix: move inner() into another bean and call that (or inject the service into itself)
```

**One-liner:** `@Transactional` on public service methods gives all-or-nothing behaviour; use `readOnly = true` for reads and watch out for private/self-invocation calls.

---

## 10. Locking

**Problem:** two users edit the same row at the same time, and the last save silently overwrites the other (**lost update**).

### Optimistic locking (default choice)
Assume conflicts are rare. Add a **`@Version`** field; Hibernate checks it on every update.

```java
@Entity
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private int stock;

    @Version                 // Hibernate manages this, never set it yourself
    private int version;
}
```

```
User A and User B both read Product (stock=100, version=1)
A saves → UPDATE products SET stock=90, version=2 WHERE id=1 AND version=1   → 1 row updated ✅
B saves → UPDATE products SET stock=80, version=2 WHERE id=1 AND version=1   → 0 rows updated ❌
          → OptimisticLockingFailureException → retry with fresh data, or return HTTP 409
```

- No DB locks, no waiting, no deadlocks.
- Best for read-heavy apps with few conflicts.

### Pessimistic locking
Lock the row when reading; others **wait** until you finish.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)                  // SELECT ... FOR UPDATE
@Query("SELECT p FROM Product p WHERE p.id = :id")
Optional<Product> findByIdForUpdate(@Param("id") Long id);
```

```java
@Transactional
public void decreaseStock(Long id, int qty) {
    Product p = productRepository.findByIdForUpdate(id).orElseThrow();   // row locked
    if (p.getStock() < qty) throw new InsufficientStockException();
    p.setStock(p.getStock() - qty);
}   // commit → lock released
```

- `PESSIMISTIC_WRITE` = nobody else can read-for-update or write. `PESSIMISTIC_READ` = others can read but not write.
- ⚠️ **Deadlock** risk (A waits for B's row while B waits for A's). Always lock rows in the **same order**.

| | Optimistic | Pessimistic |
|---|---|---|
| How | Check `@Version` on save | Lock row on read |
| Blocks others? | No | Yes |
| On conflict | Exception → retry | Others wait |
| Best for | Most apps (rare conflicts) | Hot rows: inventory, payments |

**One-liner:** Start with `@Version` (optimistic); use `PESSIMISTIC_WRITE` only for specific high-conflict operations.

---

## 11. Caching

Two separate things share the name "caching": **Hibernate's cache** (entities) and **Spring's cache** (any method result).

### A) Hibernate first-level cache (always on)
- Lives inside one transaction/session. Loading the same entity **by ID** twice returns the same object with no 2nd SQL.
- Cleared when the transaction ends. It does **not** cache query results (like `findByEmail`).

```java
@Transactional
public void demo() {
    User u1 = userRepository.findById(1L).get();   // SQL runs
    User u2 = userRepository.findById(1L).get();   // NO SQL, from cache
    System.out.println(u1 == u2);                  // true (same object)
}
```
```
SQL: select ... from users where id=1      <- printed only once
true
```

### B) Hibernate second-level cache (L2, optional)
Shared across **all** transactions in the app. Needs setup, so only use it for data that is **read often and changes rarely** (countries, categories).

```properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.internal.JCacheRegionFactory
spring.jpa.properties.hibernate.javax.cache.provider=com.github.benmanes.caffeine.jcache.spi.CaffeineCachingProvider
# dependencies needed: hibernate-jcache + com.github.ben-manes.caffeine:jcache (and caffeine)
```
```java
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)    // org.hibernate.annotations.Cache
public class Category { ... }
```

| Strategy | Use for |
|---|---|
| `READ_ONLY` | Never changes (countries, currencies), fastest |
| `NONSTRICT_READ_WRITE` | Changes rarely, brief stale data is OK |
| `READ_WRITE` | Changes sometimes and must stay consistent (default pick) |
| `TRANSACTIONAL` | Strongest consistency, slowest, rarely used |

- Query cache (optional): `hibernate.cache.use_query_cache=true` + `@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))` on the query.
- ❌ Don't cache fast-changing data or huge entities. Measure first, cache second.
- ⚠️ Native SQL and `@Modifying` updates bypass Hibernate, so the L2 cache keeps serving **stale** data unless you evict it.

### C) Spring Boot caching (`@Cacheable`) – works on **any** method
Setup: add `spring-boot-starter-cache` and put **`@EnableCaching`** on your main class. (Without it, the annotations are silently ignored.)

| Annotation | What it does | Use on |
|---|---|---|
| `@Cacheable("name")` | Cache has the answer? **Skip the method** and return it. Otherwise run it and store the result | Read methods |
| `@CachePut("name")` | **Always run** the method, then update the cache | Save/update methods |
| `@CacheEvict("name")` | Remove entry (or `allEntries = true` for all) | Delete methods |

```java
@Service
public class RoleService {

    @Cacheable("roles")                                  // key = the 'name' argument
    public Role findByName(String name) { return roleRepository.findByName(name).orElseThrow(); }

    @CachePut(value = "roles", key = "#role.name")       // refresh cache after save
    public Role save(Role role) { return roleRepository.save(role); }

    @CacheEvict(value = "roles", key = "#name")          // remove from cache after delete
    public void deleteByName(String name) { roleRepository.deleteByName(name); }
}
```
```
roleService.findByName("ADMIN");   // SQL runs, result cached
roleService.findByName("ADMIN");   // no SQL, from cache
roleService.findByName("USER");    // different key → SQL runs
```

**How the cache key is chosen:** 1 parameter → that value · several parameters → combination of all · no parameters → one single entry for the method.
Custom key with SpEL: `@Cacheable(value = "users", key = "#user.id")`. Use a stable field (like an ID), **not** a whole object.

> ⚠️ **Never put `@Cacheable` on a save/update method.** On a cache hit the method is skipped, so nothing gets saved.
> ⚠️ Like `@Transactional`, caching works through a proxy, so self-invocation and private methods are ignored.

**Expiry (TTL):** the default in-memory cache **never expires**. Use **Caffeine** for time-based expiry:

```properties
spring.cache.type=caffeine
spring.cache.cache-names=roles,jobs
spring.cache.caffeine.spec=maximumSize=500,expireAfterWrite=10m     # same rule for all caches
```
```java
// Different TTL per cache → define a CacheManager
@Bean
public CacheManager cacheManager() {
    SimpleCacheManager m = new SimpleCacheManager();
    m.setCaches(List.of(
        new CaffeineCache("jobs",  Caffeine.newBuilder().expireAfterWrite(10, TimeUnit.MINUTES).maximumSize(5000).build()),
        new CaffeineCache("roles", Caffeine.newBuilder().expireAfterWrite(1, TimeUnit.DAYS).maximumSize(100).build())));
    return m;
}
```
- `expireAfterWrite` = expires X after it was stored (best for freshness). `expireAfterAccess` = expires X after last use (keeps popular items).
- `maximumSize` = when full, least-recently-used entries are evicted.

| | Spring cache (`@Cacheable`) | Hibernate L2 (`@Cache`) |
|---|---|---|
| Level | Service/any method | Entity (JPA) |
| Key | Method arguments | Entity ID |
| Best for | API responses, lookups, computations | Repeated `findById` of the same entities |

They can be used together.

**One-liner:** L1 is automatic per transaction; for real-world caching, use `@Cacheable` + Caffeine with a TTL, and `@CachePut`/`@CacheEvict` to keep it in sync.

---

## 12. Extras

### Auditing + `@MappedSuperclass` (shared fields)
A `@MappedSuperclass` is a base class whose fields are copied into each child entity's table (the base class has no table).

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter @Setter
public abstract class BaseEntity {
    @CreatedDate @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}

@Entity
public class User extends BaseEntity { ... }    // users table gets created_at and updated_at

// and once, on the main class:  @EnableJpaAuditing
```

### Soft delete (mark as deleted instead of removing)
```java
@Entity
@SQLDelete(sql = "UPDATE users SET deleted = true WHERE id = ?")   // delete() becomes an UPDATE
@SQLRestriction("deleted = false")                                  // every query hides deleted rows (older Hibernate: @Where)
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private boolean deleted = false;
}
```
```
userRepository.delete(user)  →  SQL: update users set deleted = true where id = ?
userRepository.findAll()     →  SQL: select ... from users where deleted = false
```

---

## 13. Common Mistakes Cheat Sheet

| Mistake / error | Fix |
|---|---|
| `LazyInitializationException` | Load data inside `@Transactional`, or use `JOIN FETCH` / `@EntityGraph` |
| `@ManyToOne` / `@OneToOne` left on default EAGER | Set `fetch = LAZY` |
| `@Enumerated` default is ORDINAL | Use `EnumType.STRING` |
| `findById()` returns `Optional` | Use `.orElseThrow()` |
| Editing a detached entity, change lost | Change it inside `@Transactional`, or `save()` it again in a new transaction |
| `@Transactional` on private method / self-call | Make it public and call it from another bean |
| UPDATE/DELETE `@Query` fails or data looks stale | Add `@Modifying(clearAutomatically = true)` |
| `CascadeType.ALL` / `REMOVE` on `@ManyToMany` | Only `PERSIST` + `MERGE` |
| Only one side of a bidirectional relation set | Use helper methods (`addEmployee()`) |
| Page numbers off by one | Pages start at **0** |
| N+1 is silent | `show-sql=true`, count repeated queries |
| `ddl-auto=update/create` in production | `none` / `validate` + Flyway/Liquibase |
| Native query typo not caught | Only validated at runtime, so test them |
| `@Cacheable` on a save method | Use `@CachePut` instead |
| Cache not working | Forgot `@EnableCaching` (or called it from within the same class) |

---

## 14. Interview Q&A
*(Short, natural answers you can say out loud.)*

**1. JPA vs Hibernate vs Spring Data JPA?**
JPA is just the specification, a set of rules and interfaces. Hibernate is the implementation that actually turns those into SQL. Spring Data JPA sits on top and removes the boilerplate by generating repositories for me.

**2. What are EntityManager and the persistence context?**
The persistence context is the workspace, basically Hibernate's first-level cache, where entities are tracked during a transaction. The EntityManager is the JPA object that manages it with `persist`, `merge`, `remove` and `find`. Spring Data uses it behind the scenes.

**3. What does `save()` do?**
It checks the ID. If the ID is null it inserts, and if the ID exists it updates (merge).

**4. What is the owner side and what does `mappedBy` do?**
The owner side is the entity whose table has the foreign key, and it uses `@JoinColumn`. The other side uses `mappedBy` to say "the relationship is already mapped over there", so no extra column is created. `@ManyToOne` is always the owner.

**5. Why `EnumType.STRING`?**
`ORDINAL` stores positions like 0, 1, 2, so reordering or inserting an enum value silently corrupts the old data. `STRING` stores the name, which stays stable.

**6. `CascadeType.REMOVE` vs `orphanRemoval`?**
`REMOVE` deletes children when the parent is deleted. `orphanRemoval` deletes a child when I just remove it from the parent's collection. I normally use both on `@OneToMany`.

**7. EAGER vs LAZY? Which defaults?**
EAGER loads related data right away, LAZY loads it on first access. `@ManyToOne` and `@OneToOne` default to EAGER and the collections default to LAZY, so I switch everything to LAZY and fetch what I need with JOIN FETCH.

**8. What is the N+1 problem and how do you fix it?**
One query loads N parents, then N more queries load each parent's lazy relation. I spot it in the SQL logs and fix it with `JOIN FETCH` or `@EntityGraph`, and `@BatchSize` when there are several collections.

**9. How do you handle `LazyInitializationException`?**
It happens when I touch a lazy field after the session is closed. I fix it by loading the data inside the transaction with JOIN FETCH or EntityGraph, or by returning DTOs from the service. I avoid switching to EAGER.

**10. What is dirty checking?**
Hibernate snapshots a managed entity when it loads it and compares at commit. If something changed it runs the UPDATE itself, so no `save()` is needed. It only works inside a transaction.

**11. Does `@Transactional` work on a private method?**
No. It works through a proxy that can only intercept public calls from outside the bean. Self-invocation has the same problem, so I move the method to another bean.

**12. `REQUIRED` vs `REQUIRES_NEW`?**
`REQUIRED` joins the existing transaction, or creates one if none exists. `REQUIRES_NEW` always starts a separate one and pauses the outer one. I use it for things like audit logs that must be saved even if the main operation rolls back.

**13. Does `@Transactional` roll back on every exception?**
No, by default only on `RuntimeException` and `Error`. For checked exceptions I add `rollbackFor = Exception.class`.

**14. Optimistic vs pessimistic locking?**
Optimistic uses a `@Version` column and fails at save time if someone else changed the row, so there's no blocking. Pessimistic locks the row with `SELECT … FOR UPDATE` so others wait. I default to optimistic and use pessimistic for hot rows like inventory.

**15. JPQL vs native query?**
JPQL uses entity and field names, is database-independent and is checked at startup. Native is raw SQL, so I use it for window functions, CTEs or DB-specific features, but it isn't portable and only fails at runtime.

**16. `Page` vs `Slice`?**
`Page` runs an extra count query so I get total pages and elements. `Slice` skips the count and only tells me if there's a next page, which is faster for infinite scroll.

**17. How do projections help performance?**
They select only the columns I need, like 3 out of 30, so there's less data transfer and memory. Interface projections are the simplest, but they're read-only.

**18. First-level vs second-level cache?**
First-level is always on and lives within one session or transaction. Second-level is shared across the whole app, needs configuration and `@Cache` on entities, and suits read-mostly data.

**19. `@Cacheable` vs `@CachePut` vs `@CacheEvict`?**
`@Cacheable` returns from cache if it can and skips the method. `@CachePut` always runs the method and refreshes the cache. `@CacheEvict` removes the entry. I never put `@Cacheable` on a write method.

**20. How do you build dynamic queries with many optional filters?**
With Specifications. Each filter is a small `Specification`, I extend `JpaSpecificationExecutor`, and combine them with `.and()` / `.or()`. Filters that are null are skipped, so I avoid an explosion of method names.

**21. What is `@MappedSuperclass`?**
It's a base class with common fields like `id`, `createdAt` and `updatedAt`. Each child entity gets those columns in its own table, and the base class itself has no table.

**22. IDENTITY vs SEQUENCE?**
IDENTITY uses auto-increment, so Hibernate must insert each row to get its ID, which disables batch inserts. SEQUENCE pre-fetches IDs in blocks, so it's better for bulk inserts, especially on PostgreSQL.

**23. How do you implement soft delete?**
Add a `deleted` flag, use `@SQLDelete` to turn `delete()` into an UPDATE, and `@SQLRestriction("deleted = false")` so queries hide deleted rows.

**24. How do derived queries work?**
Spring parses the method name, like `findByEmailAndStatus`, into a query at startup and validates the field names. If the name gets long, I switch to `@Query`.

---
*End of notes.*