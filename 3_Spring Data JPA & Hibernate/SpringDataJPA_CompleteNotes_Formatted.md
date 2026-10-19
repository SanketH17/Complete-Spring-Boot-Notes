# Spring Data JPA – Complete Developer Notes

---

## 📋 Table of Contents

1. [Fundamentals](#1-fundamentals)
   - [What is JPA?](#11-what-is-jpa)
   - [What is Hibernate?](#12-what-is-hibernate)
   - [What is Spring Data JPA?](#13-what-is-spring-data-jpa)
   - [How They All Fit Together](#14-how-they-all-fit-together)
   - [Why Use Spring Data JPA?](#15-why-use-spring-data-jpa)
2. [Project Setup](#2-project-setup)
   - [Maven Dependencies](#21-maven-dependencies)
   - [Gradle Dependencies](#22-gradle-dependencies)
   - [application.properties — MySQL Configuration](#23-applicationproperties--mysql-configuration)
   - [application.yml — PostgreSQL Configuration](#24-applicationyml--postgresql-configuration)
   - [DDL Auto Strategies Explained](#25-ddl-auto-strategies-explained)
3. [Entity Mapping](#3-entity-mapping)
   - [Basic Entity — The Building Block](#31-basic-entity--the-building-block)
   - [@GeneratedValue Strategies](#32-generatedvalue-strategies)
   - [Relationships](#33-relationships)
   - [Fetch Types](#34-fetch-types)
   - [Cascade Types](#35-cascade-types)
4. [Repository Layer](#4-repository-layer)
   - [Repository Hierarchy — Understanding the Family Tree](#41-repository-hierarchy--understanding-the-family-tree)
   - [Creating a Repository](#42-creating-a-repository)
   - [Query Methods by Naming Convention (Derived Queries)](#43-query-methods-by-naming-convention-derived-queries)
   - [Pagination & Sorting](#44-pagination--sorting)
5. [Query Mechanisms in Spring Data JPA](#5-query-mechanisms-in-spring-data-jpa)
   - [Which query method should I use?](#51-which-query-method-should-i-use)
   - [JPQL — Query Using Java Class and Field Names](#52-jpql--query-using-java-class-and-field-names)
   - [Native SQL — Write Regular SQL](#53-native-sql--write-regular-sql)
   - [Named Queries — Define a Query Once and Reuse It by Name](#54-named-queries--define-a-query-once-and-reuse-it-by-name)
   - [Quick Comparison: JPQL vs Native SQL](#55-quick-comparison-jpql-vs-native-sql)
   - [Interview Revision](#56-interview-revision)
6. [Advanced Topics](#6-advanced-topics)
   - [The N+1 Problem](#61-the-n1-problem)
   - [JPA Entity Lifecycle](#62-jpa-entity-lifecycle)
   - [Dirty Checking](#63-dirty-checking)
   - [JPA Transactions (`@Transactional`)](#64-jpa-transactions-transactional)
   - [JPA Locking: Optimistic vs Pessimistic](#65-jpa-locking-optimistic-vs-pessimistic)
   - [Caching in JPA and Spring Boot](#66-caching-in-jpa-and-spring-boot)
7. [Quick Revision Cheat Sheet](#7-quick-revision-cheat-sheet)
   - [The Stack](#71-the-stack)
   - [Entity Essentials](#72-entity-essentials)
   - [Relationships](#73-relationships)
   - [Cascade Types](#74-cascade-types)
   - [DDL-Auto Strategies](#75-ddl-auto-strategies)
   - [Repositories](#76-repositories)
   - [Derived Query Keywords (Top 15)](#77-derived-query-keywords-top-15)
   - [Query Resolution Priority](#78-query-resolution-priority)
   - [Pagination & Sorting](#79-pagination--sorting)
   - [N+1 Problem — Solutions](#710-n1-problem--solutions)
   - [Entity Lifecycle States](#711-entity-lifecycle-states)
   - [Dirty Checking](#712-dirty-checking)
   - [@Transactional](#713-transactional)
   - [Locking](#714-locking)
   - [Caching](#715-caching)
   - [Common Gotchas](#716-common-gotchas)
8. [Interview Questions & Answers](#8-interview-questions--answers)
   - [Fundamentals](#81-fundamentals)
   - [Entity Mapping](#82-entity-mapping)
   - [N+1 & Performance](#83-n1--performance)
   - [Transactions](#84-transactions)
   - [Queries](#85-queries)
   - [Advanced](#86-advanced)

---

## 1. Fundamentals

### 1.1 What is JPA?

**JPA (Java Persistence API)** is a **specification** — a set of rules and interfaces that tells Java developers: *"Here's the standard way to save, read, update, and delete Java objects in a relational database."*

JPA itself is **not code you can run**. It's a blueprint. It defines *what* should happen, but not *how* it happens.

> 🧠 **Analogy — The Restaurant Menu:**  
> Think of JPA as a **restaurant menu**. The menu lists every dish you can order (save, find, delete...), describes what each dish should look like, and sets expectations. But the menu **doesn't cook anything** — you need a kitchen (Hibernate) to actually prepare the food.

#### What does JPA give you?

- **Object-Relational Mapping (ORM):** Map your Java classes directly to database tables. A `User` class becomes a `users` table. A field `String email` becomes an `email` column. You work with Java objects — JPA handles the SQL.
- **CRUD operations:** Create, Read, Update, and Delete data using Java methods instead of writing SQL by hand.
- **JPQL (Java Persistence Query Language):** A query language that looks like SQL but uses your **Java class and field names** instead of table and column names.

> 📝 **Beginner Note:** JPA is *just an interface*. By itself, it does nothing. You always need a JPA **implementation** (like Hibernate) to do the actual database work. This is the same idea as `List` being an interface and `ArrayList` being the implementation.

#### What problem does JPA solve?

Before JPA, every developer wrote raw JDBC code to talk to the database — opening connections, writing SQL strings, mapping result sets to objects manually, and closing resources. This was **repetitive**, **error-prone**, and **tightly coupled** to a specific database. JPA standardized all of this so you write Java, and the framework handles SQL.

---

### 1.2 What is Hibernate?

**Hibernate** is the most popular **implementation of JPA**. It is the actual engine that takes your JPA instructions and converts them into real SQL queries that run against your database.

> 🧠 **Analogy — The Kitchen:**  
> If JPA is the restaurant menu, **Hibernate is the kitchen and the chef**. When you order "save this User" from the menu, Hibernate is what actually fires up the stove, writes the `INSERT INTO users ...` SQL, sends it to the database, and gives you back the result.

#### What does Hibernate do behind the scenes?

Here's what happens step-by-step when you save an object:

1. You call `save(user)` (a JPA operation)
2. Hibernate reads the `@Entity` annotations on your `User` class
3. It figures out which table and columns to use
4. It generates the correct SQL: `INSERT INTO users (first_name, email, ...) VALUES (?, ?, ...)`
5. It sends the SQL to the database via JDBC
6. It returns the saved object with the generated ID

You never see steps 2–6. Hibernate does it all for you.

#### Hibernate's bonus features (beyond what JPA requires):

- **Caching** — Stores recently accessed data in memory so repeated reads don't hit the database (1st-level and 2nd-level cache)
- **Lazy Loading** — Loads related data only when you actually access it, not upfront
- **HQL (Hibernate Query Language)** — Hibernate's own query language, very similar to JPQL
- **Automatic Schema Generation** — Can create or update your database tables based on your Java classes

> 📝 **Beginner Note:** You rarely interact with Hibernate directly in a Spring Boot project. Spring Data JPA sits on top and calls Hibernate for you. But understanding that Hibernate is the engine under the hood helps you debug issues and understand logs.

---

### 1.3 What is Spring Data JPA?

**Spring Data JPA** is a **Spring framework module** that sits on top of JPA and Hibernate. Its job is to **eliminate boilerplate code** so you can accomplish database operations with minimal effort.

> 🧠 **Analogy — The Waiter:**  
> You're sitting in a restaurant (your application). JPA is the menu. Hibernate is the kitchen. **Spring Data JPA is your waiter.** You just tell the waiter "I'd like the user with email john@example.com" and the waiter handles everything — walks to the kitchen, places the order, brings back the result. You never step into the kitchen yourself.

#### The before and after

**Without Spring Data JPA** (using raw JPA/Hibernate), you had to write all this code every single time:

```java
// Pure JPA — lots of manual work
EntityManager em = entityManagerFactory.createEntityManager();  // Get a connection
em.getTransaction().begin();                                     // Start a transaction
em.persist(user);                                                // Insert the user
em.getTransaction().commit();                                    // Commit the transaction
em.close();                                                      // Close the connection
```

That's 5 lines of plumbing just to save one object. Now imagine doing this for every CRUD operation across every entity in your app.

**With Spring Data JPA**, the same thing becomes:

```java
userRepository.save(user); // That's it. One line. Done.
```

Spring Data JPA auto-generates the repository implementation at runtime. You just declare an interface, and Spring writes the code for you.

#### What problem does Spring Data JPA solve?

Even with Hibernate doing the heavy SQL work, developers still had to write **repetitive boilerplate**: creating `EntityManager` instances, managing transactions, writing basic CRUD methods over and over for every entity. Spring Data JPA removes all of that by:

- Auto-implementing repository interfaces (you write zero implementation code)
- Generating SQL queries from method names (e.g., `findByEmail(String email)` automatically becomes `SELECT * FROM users WHERE email = ?`)
- Providing built-in pagination, sorting, and batch operations

> 📝 **Beginner Note:** Spring Data JPA doesn't replace Hibernate — it **uses** Hibernate internally. Think of it as a convenience layer. The stack is:  
> **Your Code → Spring Data JPA → JPA (interface) → Hibernate (engine) → JDBC → Database**

---

### 1.4 How They All Fit Together

Here's the full picture of how these technologies are layered:

```text
┌─────────────────────────────────────────────────────┐
│              YOUR APPLICATION CODE                   │
│         (Service classes, Controllers)                │
├─────────────────────────────────────────────────────┤
│                  Spring Data JPA                     │
│   (Convenience layer — auto-generates repositories)  │
├─────────────────────────────────────────────────────┤
│                      JPA                             │
│     (Specification — defines the rules/interfaces)   │
├─────────────────────────────────────────────────────┤
│                   Hibernate                          │
│  (Implementation — converts Java operations to SQL)  │
├─────────────────────────────────────────────────────┤
│                    JDBC                              │
│        (Low-level Java database connection API)      │
├─────────────────────────────────────────────────────┤
│              Database (MySQL/PostgreSQL)              │
└─────────────────────────────────────────────────────┘
```

> 🧠 **Analogy — Ordering Online:**  
> Think of ordering food through a delivery app:
> - **You** = Your application code (the customer placing the order)
> - **Delivery App (Zomato/Swiggy)** = Spring Data JPA (takes your request, simplifies everything)
> - **Restaurant Menu** = JPA (standardized set of things you can order)
> - **Kitchen & Chef** = Hibernate (actually cooks the food / generates SQL)
> - **Delivery Driver** = JDBC (physically carries data between kitchen and your door)
> - **Restaurant** = Database (where the food/data lives)

#### Comparison Table

| Feature | JPA | Hibernate | Spring Data JPA |
|---|---|---|---|
| **What is it?** | Specification (rules) | Implementation (engine) | Abstraction (convenience) |
| **Who creates it?** | Jakarta EE (Java standard) | Red Hat / Community | Spring Team (Pivotal/VMware) |
| **Amount of boilerplate** | High | Medium | Very Low |
| **Extra features** | No (just the standard) | Yes (caching, HQL, schema gen) | Yes (Repositories, Pageable, query methods) |
| **Can work alone?** | No (needs an implementation) | Yes (standalone Hibernate works) | No (needs JPA + Hibernate underneath) |

> 💡 **Important to Remember:** In a Spring Boot project, when you add the `spring-boot-starter-data-jpa` dependency, you get **all three** automatically — Spring Data JPA, JPA interfaces, and Hibernate as the default implementation. You don't need to configure them separately.

---

### 1.5 Why Use Spring Data JPA?

Here's why Spring Data JPA is the go-to choice for database access in modern Spring Boot applications:

- ✅ **Zero boilerplate** — No manual `EntityManager`, no hand-written transactions. Just declare interfaces.
- ✅ **Magic query methods** — Write `findByEmailAndStatus(String email, UserStatus status)` and Spring auto-generates the SQL. No query writing needed for simple cases.
- ✅ **Built-in pagination and sorting** — Paginate results with `PageRequest.of(page, size)`. No manual `LIMIT`/`OFFSET` SQL.
- ✅ **Database-agnostic** — Switch from MySQL to PostgreSQL by changing a config property. Your Java code stays the same.
- ✅ **Works with any JPA provider** — Hibernate is the default, but you can swap in EclipseLink or others without changing your repository code.
- ✅ **Seamless Spring ecosystem integration** — Works beautifully with Spring Boot auto-configuration, Spring Security, Spring Transactions, etc.
- ✅ **Production-ready features** — Auditing (`@CreatedDate`), specifications (dynamic queries), entity graphs, and more — all built in.

> 📝 **Beginner Note — When NOT to use Spring Data JPA:**  
> - For extremely simple apps with 1–2 tables, `spring-boot-starter-jdbc` with `JdbcTemplate` might be simpler.  
> - For read-heavy analytics or complex reporting queries, consider using a lighter tool like **jOOQ** or **MyBatis** alongside JPA.  
> - JPA adds a learning curve — it's worth it for most CRUD-heavy business apps, but not for every use case.

---

## 2. Project Setup

> This section walks you through everything you need to set up Spring Data JPA in a Spring Boot project — from adding dependencies to configuring your database connection.

### 2.1 Maven Dependencies

> 🧠 **Analogy — Buying Ingredients Before Cooking:**  
> Before you can cook a meal, you need to buy the right ingredients. Dependencies are the ingredients for your project. Without the right ones, your app won't know how to talk to a database.

Add these dependencies to your `pom.xml`. Each one has a specific role:

```xml
<!-- pom.xml -->
<dependencies>

    <!-- 
        Spring Data JPA Starter
        This single dependency pulls in EVERYTHING you need:
        - Spring Data JPA (the convenience layer)
        - JPA API (the specification interfaces)
        - Hibernate (the default JPA implementation / engine)
        - HikariCP (the default connection pool)
        - Spring Transaction Management
    -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- 
        MySQL Database Driver
        This is the "translator" between Java and MySQL.
        Without it, your app can't communicate with a MySQL database.
        scope=runtime means it's only needed when the app runs, not during compilation.
    -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- 
        PostgreSQL Database Driver (use INSTEAD of MySQL if your DB is PostgreSQL)
        Uncomment this and remove MySQL driver if using PostgreSQL.
    -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- 
        H2 Database — Lightweight in-memory database for testing
        Your tests can run without needing a real MySQL/PostgreSQL server.
        scope=test means it's only available during testing, not in production.
    -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- 
        Lombok — Reduces Java boilerplate (getters, setters, constructors, builders)
        optional=true means projects that depend on YOUR project won't get Lombok automatically.
    -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

</dependencies>
```

> 📝 **Beginner Note:** You only need **one** database driver — pick MySQL *or* PostgreSQL based on your database. Don't include both in a real project.

> 💡 **Important to Remember:** The `spring-boot-starter-data-jpa` dependency is the key. It automatically brings in Hibernate, JPA, and connection pooling. You don't need to add them separately.

### 2.2 Gradle Dependencies

If you use Gradle instead of Maven, add these to your `build.gradle`:

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.mysql:mysql-connector-j'         // MySQL driver
    // runtimeOnly 'org.postgresql:postgresql'          // Uncomment for PostgreSQL
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'     // Lombok needs annotation processing
    testRuntimeOnly 'com.h2database:h2'                // H2 for tests only
}
```

> 📝 **Beginner Note — Maven vs Gradle:** Both do the same thing (manage dependencies). Maven uses XML (`pom.xml`), Gradle uses Groovy/Kotlin (`build.gradle`). Pick whichever your team uses. Most Spring Boot tutorials use Maven.

---

### 2.3 application.properties — MySQL Configuration

This is where you tell Spring Boot **how to connect to your database**. This file lives at `src/main/resources/application.properties`.

> 🧠 **Analogy — Filling Out a Delivery Address Form:**  
> Imagine you're ordering something online. You need to tell the delivery service: where to deliver (URL), who you are (username), and your credentials (password). That's exactly what these properties do — they tell Spring Boot where the database lives and how to log in.

```properties
# ─── Database Connection ───────────────────────────────
# URL Format: jdbc:mysql://HOST:PORT/DATABASE_NAME?options
# localhost  = database is on your machine
# 3306       = default MySQL port
# mydb       = your database name (create it first in MySQL!)
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ─── JPA / Hibernate Settings ─────────────────────────
# ddl-auto controls what Hibernate does with your database schema on startup
# See section 2.5 below for a detailed explanation of each option
spring.jpa.hibernate.ddl-auto=update

# Show the SQL that Hibernate generates in the console
# Extremely useful for learning and debugging — you can see every query!
spring.jpa.show-sql=true

# Format the SQL nicely (instead of one long unreadable line)
spring.jpa.properties.hibernate.format_sql=true

# Tell Hibernate which SQL dialect to use for MySQL 8
# Spring Boot usually auto-detects this, but being explicit avoids surprises
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# ─── Connection Pool (HikariCP) ───────────────────────
# A connection pool keeps a set of database connections ready to use,
# so your app doesn't create a new connection for every single query (expensive!).
# HikariCP is Spring Boot's default pool — fast and reliable.
spring.datasource.hikari.maximum-pool-size=10       # Max 10 connections open at once
spring.datasource.hikari.minimum-idle=5             # Keep at least 5 idle connections ready
spring.datasource.hikari.connection-timeout=30000   # Wait max 30 seconds for a connection
```

#### What happens step-by-step when your app starts:

1. Spring Boot reads `application.properties`
2. It creates a **HikariCP connection pool** with the database URL, username, and password
3. Hibernate starts up and scans for classes annotated with `@Entity`
4. Based on `ddl-auto`, it creates/updates/validates the database tables
5. Spring Data JPA auto-generates implementations for your repository interfaces
6. Your app is ready to handle database operations!

> ⚠️ **Common Mistake:** Forgetting to create the database first! `spring.datasource.url` points to `mydb`, but if that database doesn't exist in MySQL, the app will fail on startup. Run `CREATE DATABASE mydb;` in MySQL first.

> 🧠 **Analogy — Connection Pool:**  
> Think of a connection pool like a **taxi stand** at a hotel. Instead of calling a new taxi every time a guest needs a ride (slow and expensive), the hotel keeps 10 taxis parked and ready. When a guest needs one, they grab an available taxi. When done, the taxi returns to the stand for the next guest. That's exactly what HikariCP does with database connections.

---

### 2.4 application.yml — PostgreSQL Configuration

Spring Boot also supports YAML format (`application.yml`). YAML uses indentation instead of repeated prefixes, making it cleaner for nested properties:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb   # PostgreSQL default port is 5432
    username: postgres
    password: yourpassword
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5

  jpa:
    hibernate:
      ddl-auto: update         # Options: create, create-drop, validate, update, none
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
        # Batch insert optimization — groups multiple INSERTs into one round-trip
        jdbc:
          batch_size: 50
        order_inserts: true    # Reorder INSERT statements to enable batching
        order_updates: true    # Reorder UPDATE statements to enable batching
```

> 📝 **Beginner Note — .properties vs .yml:** Both do the same thing. Use whichever your team prefers. YAML looks cleaner for deeply nested config, but indentation mistakes can break it. Properties files are simpler and more forgiving. **Never use both files for the same properties** — Spring Boot loads both, and conflicts can cause confusing bugs.

---

### 2.5 DDL Auto Strategies Explained

The `spring.jpa.hibernate.ddl-auto` property tells Hibernate what to do with your database tables when the application starts up. This is one of the most important settings to understand.

> 🧠 **Analogy — Setting Up a Room:**  
> Imagine you're assigned a hotel room (database) every day:
> - **`create`** = Throw away everything in the room and set it up fresh every morning (destructive!)
> - **`create-drop`** = Set up the room when you arrive, destroy everything when you leave
> - **`update`** = Walk in and rearrange the room to match your needs, but keep existing stuff
> - **`validate`** = Walk in and just check if the room matches your expectations. If not, refuse to stay (app crashes)
> - **`none`** = Don't touch the room at all. Assume someone else set it up perfectly.

| Value | What Happens on App Startup | When to Use |
|---|---|---|
| `create` | **Drops all tables** and recreates them from scratch. **All data is lost!** | Local development only (when you don't care about data) |
| `create-drop` | Creates tables on startup, **drops them when the app shuts down** | Automated testing (clean state every test run) |
| `update` | Compares your entities to existing tables. Adds new columns/tables but **never deletes** existing ones. | Development (convenient, but see warning below) |
| `validate` | Checks that your entities match the existing database schema. If they don't match, the **app refuses to start** with an error. | Staging / Pre-production |
| `none` | Hibernate does nothing to the schema. You manage it yourself. | **Production** (use Flyway or Liquibase for migrations) |

> ⚠️ **Common Mistake — NEVER use `create` or `update` in production!**
> - `create` destroys all your data on every restart.
> - `update` seems safe, but it can silently make schema changes that are hard to track or reverse. It also **never drops columns** — if you remove a field from your entity, the column stays in the database forever.
> - In production, use a **database migration tool** like **Flyway** or **Liquibase**. These tools track every schema change as versioned scripts, making changes reviewable, repeatable, and reversible.

> 📝 **Beginner Note — Recommended setup for learning:**  
> Use `ddl-auto=update` while learning. It automatically creates and modifies tables as you change your entity classes, so you can focus on learning JPA without writing SQL schema scripts. Switch to `validate` + Flyway when building real projects.

---

## 3. Entity Mapping

> Entity mapping is the heart of JPA. It's how you tell Java: "this class represents a database table, and these fields represent its columns." Once you master this section, you'll be able to model any database schema using pure Java code.

### 3.1 Basic Entity — The Building Block

#### What is an Entity?

An **entity** is a plain Java class that JPA maps to a database table. Each **instance** of the class represents one **row** in that table. Each **field** in the class represents one **column**.

> 🧠 **Analogy — A Spreadsheet:**  
> Think of a database table as a **Google Sheet**:
> - The **class** (`User`) is the sheet template — it defines the column headers (firstName, email, status...)
> - Each **object** (`new User("John", "john@example.com")`) is one **row** of data in that sheet
> - The **annotations** (`@Column`, `@Id`) are like formatting rules you set on each column (required, unique, max length...)

#### A Complete Entity Example

Here's a real-world `User` entity with every common annotation explained:

```java
// File: src/main/java/com/example/entity/User.java

package com.example.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDateTime;

@Entity                         // ① Tells JPA: "This class maps to a database table"
@Table(
    name = "users",             // ② Explicitly set table name. Default would be "User",
                                //    but "user" is a reserved word in many databases!
    indexes = {
        @Index(name = "idx_email", columnList = "email"),       // ③ Database index = faster lookups
        @Index(name = "idx_status", columnList = "status")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uc_email", columnNames = {"email"}) // ④ No duplicate emails
    }
)
@Getter                         // Lombok: auto-generates all getter methods
@Setter                         // Lombok: auto-generates all setter methods
@NoArgsConstructor              // Lombok: generates empty constructor (JPA requires this!)
@AllArgsConstructor             // Lombok: generates constructor with all fields
@Builder                        // Lombok: enables builder pattern — User.builder().name("John").build()
public class User {

    @Id                                     // ⑤ This field is the PRIMARY KEY
    @GeneratedValue(strategy = GenerationType.IDENTITY) // ⑥ Database auto-generates this value (1, 2, 3...)
    private Long id;

    @Column(
        name = "first_name",                // ⑦ Column name in DB (default would be "firstName")
        nullable = false,                   // NOT NULL — this field is required
        length = 50                         // VARCHAR(50) — max 50 characters
    )
    private String firstName;

    @Column(nullable = false, unique = true, length = 100)  // Required + must be unique
    private String email;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Enumerated(EnumType.STRING)            // ⑧ Store enum as text "ACTIVE", not number 0
    @Column(nullable = false, length = 20)
    private UserStatus status;

    @Column(columnDefinition = "TEXT")      // ⑨ TEXT type for long strings (no length limit)
    private String bio;

    @Column(name = "is_verified")
    private boolean verified = false;       // Default value

    @Column(name = "created_at", updatable = false) // ⑩ Cannot be changed after initial insert
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

```java
// Enum for user status — used by the @Enumerated field above
public enum UserStatus {
    ACTIVE, INACTIVE, BANNED, PENDING_VERIFICATION
}
```

#### Annotation-by-Annotation Breakdown

Let's walk through every annotation used above so there's zero confusion:

| # | Annotation | What It Does | Why It Matters |
|---|---|---|---|
| ① | `@Entity` | Marks this class as a JPA entity (a DB table) | Without it, JPA completely ignores this class |
| ② | `@Table(name = "users")` | Sets the table name explicitly | Avoids conflicts with reserved words like "user" |
| ③ | `@Index` | Creates a database index on specified columns | Makes `WHERE email = ?` queries much faster |
| ④ | `@UniqueConstraint` | Ensures no duplicate values in specified columns | Prevents two users from having the same email |
| ⑤ | `@Id` | Marks the primary key field | Every entity MUST have exactly one `@Id` |
| ⑥ | `@GeneratedValue` | Auto-generates the ID value | You don't set the ID — the database assigns it |
| ⑦ | `@Column(name = ...)` | Customizes column name, nullability, length | Controls how the field maps to the actual column |
| ⑧ | `@Enumerated(STRING)` | Stores enum as readable text, not number | See warning below about ORDINAL |
| ⑨ | `columnDefinition = "TEXT"` | Uses DB-specific column type | For large text fields that exceed `VARCHAR` limits |
| ⑩ | `updatable = false` | Prevents this column from being changed after insert | Perfect for `createdAt` timestamps |

> ⚠️ **Common Mistake — `@Enumerated(EnumType.ORDINAL)` (the default!):**  
> If you use `ORDINAL`, the enum is stored as a number (ACTIVE=0, INACTIVE=1, BANNED=2). Now imagine you add a new status `SUSPENDED` between ACTIVE and INACTIVE — suddenly all your database values shift and mean the wrong thing! **Always use `EnumType.STRING`** to store the actual name like `"ACTIVE"`, `"BANNED"`.

> 📝 **Beginner Note — Why `@NoArgsConstructor`?**  
> JPA creates entity objects using reflection, which requires an **empty constructor** (no arguments). If you don't have Lombok, you must write `public User() {}` yourself. Without it, your app crashes at startup with a confusing error.

> 📝 **Beginner Note — `@Table` naming tips:**
> - Always set `name` explicitly. Don't rely on defaults.
> - Use **lowercase, plural, snake_case** names: `users`, `order_items`, `user_profiles`
> - Avoid reserved SQL words: `user`, `order`, `group`, `table`, `index` — these cause cryptic SQL errors if used as table names

---

### 3.2 @GeneratedValue Strategies

When you use `@Id` to mark a primary key, you usually want the database (or Hibernate) to **auto-generate** the ID for you. The `@GeneratedValue` annotation controls *how* that ID is generated.

> 🧠 **Analogy — Getting a Token Number:**  
> When you walk into a bank, you get a token number. There are different ways the bank can assign it:
> - **Counter machine (IDENTITY):** You take the next number from the counter display — simple and sequential
> - **Pre-printed ticket book (SEQUENCE):** The bank has a book of pre-printed numbers and tears out the next one — faster because they can prepare batches
> - **Random lottery number (UUID):** You get a unique random number — guaranteed unique even across branches
> - **Handwritten ledger (TABLE):** Someone manually writes the next number in a notebook — slow and unreliable

```java
// ── Strategy 1: IDENTITY ──────────────────────────────────────────────
// The database auto-increments: 1, 2, 3, 4...
// Simplest and most common for MySQL
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
// Behind the scenes: INSERT INTO users (name, email) VALUES (?, ?)
// The database assigns the next available ID automatically

// ── Strategy 2: SEQUENCE ──────────────────────────────────────────────
// Uses a database "sequence" object to generate IDs
// Best performance for PostgreSQL and Oracle
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 50)
private Long id;
// allocationSize=50 → Hibernate fetches 50 IDs at once from the database,
// then assigns them in memory without extra DB calls. Much faster for bulk inserts!

// ── Strategy 3: UUID ──────────────────────────────────────────────────
// Generates a universally unique identifier like "550e8400-e29b-41d4-a716-446655440000"
// Best for microservices where multiple services create records independently
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private UUID id;
// Advantage: No risk of ID collisions across different databases/services
// Disadvantage: Larger column size, slightly slower indexing

// ── Strategy 4: TABLE ─────────────────────────────────────────────────
// Uses a separate database table to track the next ID value
// SLOWEST strategy — requires a separate table query and lock for each ID
@Id
@GeneratedValue(strategy = GenerationType.TABLE)
private Long id;
// Avoid this in production. It exists for compatibility, not performance.
```

#### Which strategy should you use?

| Strategy | Best For | Database | Performance | Trade-off |
|---|---|---|---|---|
| `IDENTITY` | Simple apps, getting started | MySQL, PostgreSQL, SQL Server | Good | Disables Hibernate batch inserts |
| `SEQUENCE` | High-performance apps, bulk operations | PostgreSQL, Oracle | **Best** | Not supported by all databases |
| `UUID` | Microservices, distributed systems | All databases | Good | Larger IDs, bigger indexes |
| `TABLE` | Legacy compatibility only | All databases | **Worst** | Avoid in new projects |

> 💡 **Important to Remember:**  
> - **MySQL users:** Use `IDENTITY` — it's the most natural fit.
> - **PostgreSQL users:** Use `SEQUENCE` — it gives the best performance, especially for batch inserts.
> - **Building microservices?** Use `UUID` — no two services will ever generate the same ID.
> - **Need batch inserts?** Avoid `IDENTITY` — it forces Hibernate to insert one row at a time (because it needs the DB-generated ID back immediately). Use `SEQUENCE` with a high `allocationSize`.

---

### 3.3 Relationships

In real-world applications, tables don't exist in isolation — they're connected. A user has orders. An order has items. An item belongs to a product category. JPA lets you model these connections using relationship annotations.

> 🧠 **Analogy — A Family Tree:**  
> Think of database relationships like family relationships:
> - **@OneToOne** → A person has one passport (and that passport belongs to one person)
> - **@OneToMany** → A mother has many children
> - **@ManyToOne** → Many children have one mother
> - **@ManyToMany** → Many students attend many classes, and each class has many students

#### Key Concepts Before We Start

Before looking at the code, understand these three critical terms:

- **Owner side:** The entity whose table has the **foreign key column**. This is the side that "owns" the relationship in the database.
- **Inverse side:** The other entity. It uses `mappedBy` to say "the relationship is already defined on the other side; don't create another foreign key here."
- **`@JoinColumn`:** Specifies the name of the foreign key column on the owner's table.

> 🧠 **Analogy — Owner vs Inverse:**  
> Think of a **landlord and tenant** relationship. The **lease agreement** (foreign key) is stored in the **tenant's file** (owner side). The landlord (inverse side) knows about the relationship but doesn't keep the lease in their own file — they just reference the tenant's file (`mappedBy`).

---

#### @OneToOne

**Scenario:** Each `User` has exactly one `UserProfile`, and each `UserProfile` belongs to exactly one `User`.

```java
// User.java — OWNER SIDE (the foreign key column lives in the 'users' table)
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    @OneToOne(
        cascade = CascadeType.ALL,      // When you save/delete a User, do the same to its Profile
        fetch = FetchType.LAZY,          // Don't load the profile until you actually access it
        orphanRemoval = true             // If you set profile = null, delete the old Profile from DB
    )
    @JoinColumn(name = "profile_id")    // This creates a "profile_id" column in the 'users' table
    private UserProfile profile;
}
```

```java
// UserProfile.java — INVERSE SIDE (no foreign key here — it points back via mappedBy)
@Entity
@Table(name = "user_profiles")
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;
    private String avatarUrl;

    @OneToOne(mappedBy = "profile")     // "profile" = the field name in User class that owns this
    private User user;                  // This does NOT create a column — it's just a Java reference
}
```

**What happens in the database:**

```text
users table:           user_profiles table:
+----+-------+------------+    +----+------+
| id | email | profile_id |    | id | bio  |
+----+-------+------------+    +----+------+
|  1 | a@b.c |          1 |    |  1 | Hi!  |
+----+-------+------------+    +----+------+
```

The `profile_id` foreign key lives in the `users` table because `User` is the owner side.

---

#### @OneToMany and @ManyToOne (Most Common Relationship)

**Scenario:** One `Department` has many `Employees`. Each `Employee` belongs to one `Department`.

> 🧠 **Analogy — A Manager and Their Team:**  
> A department manager (Department) manages a team of employees (List<Employee>). The employee's badge (Employee table) has a "department_id" printed on it — that's the foreign key. The manager doesn't carry a list of employee IDs; instead, they know to look for everyone whose badge says their department.

```java
// Department.java — the "ONE" side (one department has many employees)
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(
        mappedBy = "department",        // "department" = the field name in Employee that points back
        cascade = CascadeType.ALL,      // Operations cascade to employees
        fetch = FetchType.LAZY,          // ✅ ALWAYS use LAZY for collections! (see section 3.4)
        orphanRemoval = true             // Removing an employee from the list deletes it from DB
    )
    private List<Employee> employees = new ArrayList<>();  // Initialize to avoid NullPointerException

    // ── Convenience methods ──────────────────────────────────
    // These keep BOTH sides of the relationship in sync.
    // Without them, you'd need to manually set department on the employee AND
    // add the employee to the list — easy to forget and causes bugs!
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);    // Sync the other side
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);    // Sync the other side
    }
}
```

```java
// Employee.java — the "MANY" side (many employees belong to one department)
// This is the OWNER SIDE — the foreign key column lives here
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @ManyToOne(fetch = FetchType.LAZY)  // ✅ Override default EAGER to LAZY!
    @JoinColumn(
        name = "department_id",         // Foreign key column in the 'employees' table
        nullable = false                // Every employee MUST belong to a department
    )
    private Department department;
}
```

**What happens in the database:**

```text
departments table:         employees table:
+----+-------------+      +----+-------+---------------+
| id | name        |      | id | name  | department_id |
+----+-------------+      +----+-------+---------------+
|  1 | Engineering |      |  1 | Alice |             1 |
|  2 | Marketing   |      |  2 | Bob   |             1 |
+----+-------------+      |  3 | Carol |             2 |
                           +----+-------+---------------+
```

> 📝 **Beginner Note — Who is the owner in @OneToMany/@ManyToOne?**  
> The **@ManyToOne side is always the owner**. That's because the foreign key naturally lives in the "many" table. Think about it: it makes sense for each employee row to store which department it belongs to — not for the department row to store a list of all employee IDs.

> ⚠️ **Common Mistake — Forgetting sync methods:**  
> If you do `department.getEmployees().add(employee)` but forget `employee.setDepartment(department)`, the relationship might not be saved correctly. Always sync both sides, or use the convenience methods shown above.

---

#### @ManyToMany

**Scenario:** `Students` can enroll in many `Courses`, and each `Course` has many `Students`.

> 🧠 **Analogy — A Library Membership:**  
> Many members can borrow from many libraries, and each library serves many members. Neither the member's card nor the library's system can store this cleanly — you need a **separate sign-up registry** (join table) that records "Member X is registered at Library Y."

A `@ManyToMany` relationship requires a **join table** — a third table that connects the two entities:

```java
// Student.java — OWNER SIDE (defines the join table)
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(
        cascade = {CascadeType.PERSIST, CascadeType.MERGE}, // Save/update cascade — but NOT delete!
        fetch = FetchType.LAZY
    )
    @JoinTable(
        name = "student_courses",                               // Name of the join table
        joinColumns = @JoinColumn(name = "student_id"),         // FK pointing to Student
        inverseJoinColumns = @JoinColumn(name = "course_id")    // FK pointing to Course
    )
    private Set<Course> courses = new HashSet<>();  // Use Set (not List) to avoid duplicates

    // Convenience method — keeps both sides in sync
    public void enrollIn(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
}
```

```java
// Course.java — INVERSE SIDE (uses mappedBy, no @JoinTable needed)
@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany(mappedBy = "courses", fetch = FetchType.LAZY)  // Points to Student.courses field
    private Set<Student> students = new HashSet<>();
}
```

**What happens in the database:**

```text
students table:        student_courses (JOIN TABLE):     courses table:
+----+-------+        +------------+-----------+        +----+---------+
| id | name  |        | student_id | course_id |        | id | title   |
+----+-------+        +------------+-----------+        +----+---------+
|  1 | Alice |        |          1 |         1 |        |  1 | Math    |
|  2 | Bob   |        |          1 |         2 |        |  2 | Science |
+----+-------+        |          2 |         1 |        +----+---------+
                       +------------+-----------+
```

Alice is enrolled in Math and Science. Bob is enrolled in Math. The join table stores these many-to-many connections.

> ⚠️ **Common Mistake — `CascadeType.ALL` on @ManyToMany:**  
> Never use `CascadeType.REMOVE` or `CascadeType.ALL` on a `@ManyToMany` relationship. Why? If you delete Student "Alice", cascade would try to delete Course "Math" — but "Bob" is still enrolled in Math! You'd be deleting shared data. Only use `PERSIST` and `MERGE`.

> 📝 **Beginner Note — Why `Set` instead of `List`?**  
> `Set` prevents duplicate entries (a student can't enroll in the same course twice). With `List`, Hibernate can also generate less efficient SQL for many-to-many deletions. Use `HashSet` as the default implementation.

> 💡 **Important to Remember — When @ManyToMany isn't enough:**  
> If your join table needs extra columns (e.g., `enrollment_date`, `grade`), you can't use `@ManyToMany`. Instead, create a separate `Enrollment` entity with `@ManyToOne` to both `Student` and `Course`. This is common in real projects.

---

### 3.4 Fetch Types

Fetch type controls **when** JPA loads related data from the database. This is one of the most important performance concepts in JPA.

> 🧠 **Analogy — Online Shopping Cart:**  
> Imagine you're browsing an online store:
> - **EAGER loading** = Every time you open a product page, the website also loads all reviews, related products, seller info, warranty details, and shipping options — even if you just wanted to see the price. Slow and wasteful.
> - **LAZY loading** = The product page loads just the basic info (name, price, image). Reviews only load when you scroll down and click "Show Reviews." Fast and efficient.

#### How It Works

```java
// EAGER: Related data is loaded IMMEDIATELY in the same SQL query
// When you load an Employee, their Department is fetched RIGHT AWAY (via SQL JOIN)
@ManyToOne(fetch = FetchType.EAGER)
private Department department;
// SQL: SELECT e.*, d.* FROM employees e JOIN departments d ON e.department_id = d.id

// LAZY: Related data is loaded ONLY when you explicitly access it
// When you load a Department, its employees are NOT fetched until you call getEmployees()
@OneToMany(fetch = FetchType.LAZY)
private List<Employee> employees;
// SQL: SELECT * FROM departments WHERE id = ?   ← only this runs initially
// SQL: SELECT * FROM employees WHERE department_id = ?  ← runs only when you access employees
```

#### Step-by-step: What happens with LAZY loading

1. You call `departmentRepository.findById(1L)` → Hibernate runs `SELECT * FROM departments WHERE id = 1`
2. You get back a `Department` object. The `employees` list is **not loaded yet** — it's a special Hibernate **proxy** (placeholder)
3. You call `department.getEmployees()` → **Now** Hibernate runs `SELECT * FROM employees WHERE department_id = 1`
4. The employees are loaded and returned

This means: if you never call `getEmployees()`, that second SQL query **never runs** — saving time and memory.

#### Default Fetch Types (memorize this!)

| Annotation | Default | Why | Your Action |
|---|---|---|---|
| `@ManyToOne` | `EAGER` ⚠️ | JPA spec assumes loading one parent is cheap | **Override to LAZY!** |
| `@OneToOne` | `EAGER` ⚠️ | JPA spec assumes loading one related item is cheap | **Override to LAZY!** |
| `@OneToMany` | `LAZY` ✅ | Loading a collection could be huge | Keep as LAZY |
| `@ManyToMany` | `LAZY` ✅ | Loading a collection could be huge | Keep as LAZY |

> ⚠️ **Common Mistake — Leaving @ManyToOne as default EAGER:**  
> If an `Employee` has an EAGER `Department`, and a `Department` has an EAGER `Company`, and a `Company` has EAGER `Addresses`... loading one Employee triggers a chain reaction that loads half your database! This is why EAGER loading is the #1 cause of unexpected performance problems.

> ✅ **Best Practice: Always use `FetchType.LAZY` on every relationship.** When you actually need the related data, use `JOIN FETCH` in your JPQL query (covered in Section 5) to load it efficiently in one query.

> 📝 **Beginner Note — LazyInitializationException:**  
> If you load an entity with LAZY relationships, then try to access the lazy field **after the database session is closed** (e.g., outside a `@Transactional` method), you'll get a `LazyInitializationException`. This is one of the most common JPA errors. The fix: load the data you need inside the transaction, either by accessing the field or using `JOIN FETCH`.

---

### 3.5 Cascade Types

Cascade types define what happens to **child entities** when you perform an operation on the **parent entity**.

> 🧠 **Analogy — A Filing Cabinet:**  
> Think of a `Department` as a folder in a filing cabinet. Each `Employee` document is inside that folder.
> - **CascadeType.PERSIST** = When you put the folder in the cabinet (save department), all documents inside are also filed (employees are saved)
> - **CascadeType.REMOVE** = When you throw away the folder (delete department), all documents inside go in the trash too (employees are deleted)
> - **CascadeType.ALL** = Every action you do to the folder happens to all documents inside

#### Cascade Type Reference

| CascadeType | What Happens | Real-World Example |
|---|---|---|
| `PERSIST` | Saving parent also saves new children | Saving an Order also saves its OrderItems |
| `MERGE` | Updating parent also updates changed children | Updating a blog Post also updates edited Comments |
| `REMOVE` | Deleting parent also deletes children | Deleting a Department deletes its Employees |
| `REFRESH` | Reloading parent from DB also reloads children | Refreshing an Order also refreshes its Items |
| `DETACH` | Detaching parent from JPA also detaches children | (Rarely used directly) |
| `ALL` | All of the above combined | Use when parent fully "owns" the children |

#### When to Use Each

```java
// ── @OneToMany: Parent fully owns children ────────────────────────────
// An Order owns its OrderItems. If the order is deleted, the items should go too.
// Using CascadeType.ALL + orphanRemoval = full lifecycle control
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items = new ArrayList<>();
// ✅ Save order → items are saved
// ✅ Delete order → items are deleted
// ✅ Remove item from list → item is deleted from DB (orphanRemoval)

// ── @ManyToMany: Shared entities — BE CAREFUL ────────────────────────
// A Student enrolls in Courses. Deleting a Student should NOT delete Courses!
// Only cascade PERSIST and MERGE — NEVER REMOVE
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private Set<Course> courses = new HashSet<>();
// ✅ Save student with new courses → courses are saved
// ✅ Update student → course changes are merged
// ❌ Delete student → courses are NOT deleted (correct behavior!)
```

#### What is `orphanRemoval`?

`orphanRemoval = true` means: if a child entity is **removed from the parent's collection**, delete it from the database. This is different from `CascadeType.REMOVE`, which only deletes children when the **parent itself** is deleted.

```java
// With orphanRemoval = true:
department.getEmployees().remove(employee);  // Remove from list
// → Hibernate automatically runs: DELETE FROM employees WHERE id = ?

// Without orphanRemoval (only cascade REMOVE):
department.getEmployees().remove(employee);  // Remove from list
// → The employee still exists in the database! It's just no longer in the Java list.
// → This creates an "orphan" — a record with no parent.
```

> ⚠️ **Common Mistake — `CascadeType.ALL` on @ManyToMany:**  
> This is a **dangerous mistake** beginners often make. If Student A and Student B are both enrolled in Course "Math", and you delete Student A with `CascadeType.ALL`, Hibernate will try to **delete the Math course** — which breaks Student B's enrollment! Only use `PERSIST` and `MERGE` on `@ManyToMany`.

> 📝 **Beginner Note — When to use `CascadeType.ALL`:**  
> Use it only when the parent **completely owns** the children and the children **can't exist without the parent**. Examples: Order → OrderItems, Invoice → InvoiceLines, Blog Post → Comments. Don't use it when entities are shared (like Courses that many Students use).

---

## 4. Repository Layer

> This section covers how Spring Data JPA lets you talk to the database without writing any implementation code. You just declare an interface, and Spring generates all the CRUD operations, queries, pagination, and sorting for you at runtime.

### 4.1 Repository Hierarchy — Understanding the Family Tree

#### What is a Repository?

A **repository** is a Java **interface** that acts as a bridge between your service/business logic and the database. Instead of writing SQL or managing `EntityManager` yourself, you declare a repository interface and Spring Data JPA **auto-generates the implementation** at startup.

> 🧠 **Analogy — A Bank Teller Window:**  
> Imagine a bank. You (your service class) don't walk into the vault to get money yourself. You go to a **teller window** (the repository). You say "I need the account with ID 42" and the teller handles everything — finding the record, returning the result. The repository is that teller window for your database.

Spring Data JPA provides a **hierarchy** of repository interfaces, each one adding more features on top of the previous one:

```text
Repository (marker interface — empty, does nothing by itself)
    └── CrudRepository<T, ID>                          → Basic CRUD (save, find, delete, count)
            └── ListCrudRepository<T, ID>              → Same as CrudRepository but returns List instead of Iterable
            └── PagingAndSortingRepository<T, ID>      → + Pagination & Sorting
                    └── JpaRepository<T, ID>           → + flush(), batch ops, JPA-specific extras
```

#### What does each level give you?

| Interface | What It Adds | When to Use |
|---|---|---|
| `CrudRepository` | `save()`, `findById()`, `findAll()`, `delete()`, `count()`, `existsById()` | When you only need basic CRUD — no pagination, no sorting |
| `PagingAndSortingRepository` | `findAll(Pageable)`, `findAll(Sort)` | When you need paginated or sorted results |
| `JpaRepository` | `flush()`, `saveAllAndFlush()`, `deleteInBatch()`, `getReferenceById()` | **Most common choice** — gives you everything above + JPA-specific optimizations |

> 📝 **Beginner Note:** In almost every real-world project, you'll extend `JpaRepository`. It includes everything from `CrudRepository` and `PagingAndSortingRepository`, plus extra JPA features. There's rarely a reason to use the others directly.

> 💡 **Important to Remember:** You **never write a class that implements** these interfaces. Spring Data JPA creates the implementation **automatically at runtime** using proxies. You just declare the interface — that's it!

---

### 4.2 Creating a Repository

#### What is this?

A repository interface is a plain Java interface that extends `JpaRepository`. By doing this, you instantly get dozens of ready-to-use database methods — without writing a single line of implementation code.

#### Why does it exist?

Without repositories, you'd have to write repetitive `EntityManager` code for every entity — `persist()`, `find()`, `remove()`, manual transaction handling, etc. Repositories eliminate all of that boilerplate.

> 🧠 **Analogy — A TV Remote:**  
> Think of `JpaRepository` as a universal TV remote that comes with **all the standard buttons pre-programmed** — power, volume, channel up/down, mute. You don't have to build these buttons yourself. You just pick up the remote (extend the interface) and start pressing buttons (calling methods). If you need a special button (a custom query), you can add it — but the basics are already there.

```java
// File: src/main/java/com/example/repository/UserRepository.java

package com.example.repository;

import com.example.entity.User;
import com.example.entity.UserStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;
import java.util.Optional;

@Repository // Optional — Spring auto-detects JpaRepository interfaces, but this makes it explicit
public interface UserRepository extends JpaRepository<User, Long> {
    //                                                  ^^^^  ^^^^
    //                                               Entity  Primary Key
    //                                               Type    Type
    
    // That's it! No implementation class needed.
    // Spring auto-generates all the code behind the scenes.
}
```

#### What happens behind the scenes when Spring starts up?

1. Spring Boot scans your project for interfaces that extend `JpaRepository`
2. For each one, it **dynamically creates a class** (using a Java proxy) that implements all the methods
3. It registers that class as a Spring Bean in the application context
4. When your service class injects `UserRepository`, it gets the auto-generated implementation

You never see this generated class — it happens at runtime. But it's a real, fully functional implementation.

#### Built-in Methods You Get for FREE

Once you extend `JpaRepository<User, Long>`, you can immediately use all of these methods without writing any code:

```java
// ─── CREATE / UPDATE ──────────────────────────────────────────────────
userRepository.save(user);                    // Insert new OR update existing user
userRepository.saveAll(listOfUsers);          // Save multiple users at once
userRepository.saveAndFlush(user);            // Save and immediately push to DB (not wait for transaction end)

// ─── READ ─────────────────────────────────────────────────────────────
userRepository.findById(1L);                  // Returns Optional<User> — might be empty!
userRepository.findAll();                     // Returns List<User> — all users in the table
userRepository.findAllById(List.of(1L, 2L));  // Find multiple users by their IDs
userRepository.getReferenceById(1L);          // Returns a lazy proxy (doesn't hit DB until you access a field)

// ─── DELETE ───────────────────────────────────────────────────────────
userRepository.deleteById(1L);                // Delete user with ID 1
userRepository.delete(user);                  // Delete this specific user object
userRepository.deleteAll();                   // Delete ALL users (be careful!)
userRepository.deleteAllInBatch();            // Faster bulk delete (single DELETE SQL, no entity loading)

// ─── UTILITY ──────────────────────────────────────────────────────────
userRepository.count();                       // Total number of users in the table
userRepository.existsById(1L);                // Does a user with ID 1 exist? (returns boolean)

// ─── PAGINATION & SORTING ─────────────────────────────────────────────
userRepository.findAll(PageRequest.of(0, 10));            // First page, 10 records per page
userRepository.findAll(Sort.by("firstName").ascending()); // All users sorted by name A→Z
```

> 📝 **Beginner Note — `save()` does double duty!**  
> `save()` is smart. If the entity's `@Id` field is `null` (new entity), it runs an `INSERT`. If the `@Id` already has a value (existing entity), it runs an `UPDATE`. You don't need separate "create" and "update" methods!

> ⚠️ **Common Mistake — `findById()` returns `Optional`, not the entity directly!**  
> You can't do `User user = userRepository.findById(1L);` — that won't compile. You need to handle the possibility that no user exists:
> ```java
> // ✅ Correct — handle the Optional
> User user = userRepository.findById(1L)
>     .orElseThrow(() -> new RuntimeException("User not found with ID: 1"));
> ```

> 💡 **Important to Remember — `deleteById()` vs `deleteAllInBatch()`:**  
> - `deleteById()` first loads the entity, then deletes it (2 SQL queries). This is needed if you have cascade/lifecycle callbacks.
> - `deleteAllInBatch()` runs a single `DELETE FROM users` SQL — much faster for bulk operations, but skips cascade and lifecycle callbacks.

---

### 4.3 Query Methods by Naming Convention (Derived Queries)

#### What is this?

Spring Data JPA can **read your method name** and automatically generate the SQL query from it. You just write a method signature following a naming convention, and Spring figures out the WHERE clause, ORDER BY, LIMIT, etc.

#### Why does it exist?

For simple queries like "find users by email" or "find active users sorted by name," writing full `@Query` annotations or SQL is overkill. Derived query methods let you express these in one line — the method name IS the query.

#### What problem does it solve?

Without this feature, you'd need to write a `@Query` annotation (or native SQL) for every single query, even trivial ones. Derived queries handle 70–80% of simple queries with zero SQL.

> 🧠 **Analogy — Voice Commands:**  
> Think of it like a smart assistant. You don't say "Execute SQL: SELECT star FROM users WHERE email equals john@gmail.com." You just say **"Find by email: john@gmail.com"** and the assistant knows exactly what to do. Spring Data JPA works the same way — your method name is the voice command, and Spring translates it into the right SQL.

#### How does Spring parse the method name?

Spring breaks your method name into parts and maps each part to SQL:

```text
findByFirstNameAndStatusOrderByCreatedAtDesc
│     │         │  │      │       │         │
│     │         │  │      │       │         └── DESC = descending order
│     │         │  │      │       └── CreatedAt = ORDER BY created_at
│     │         │  │      └── OrderBy = ORDER BY clause
│     │         │  └── Status = AND status = ?
│     │         └── And = AND keyword
│     └── FirstName = WHERE first_name = ?
└── findBy = SELECT ... WHERE ...

Generated SQL: SELECT * FROM users WHERE first_name = ? AND status = ? ORDER BY created_at DESC
```

#### Step-by-Step: What happens when you call a derived query method

1. At **application startup**, Spring reads every method in your repository interface
2. For methods that follow the naming convention, it **parses the method name** into a query tree
3. It validates that the field names in the method name actually exist on your entity class
4. At **runtime**, when your code calls the method, Spring generates the SQL, sets the parameters, executes it, and maps the results back to Java objects

> 📝 **Beginner Note — Validation at startup:**  
> If you misspell a field name (e.g., `findByEmali` instead of `findByEmail`), your application **won't start**. Spring checks at startup and throws an error like: *"No property 'emali' found for type 'User'."* This is a good thing — it catches bugs early!

#### Complete Example with All Common Patterns

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // ═══════════════════════════════════════════════════════════
    // FIND BY SINGLE FIELD
    // ═══════════════════════════════════════════════════════════
    
    // Find one user by their email address
    // Returns Optional because there might be no user with that email
    Optional<User> findByEmail(String email);
    // SQL: SELECT * FROM users WHERE email = ?

    // Find all users with a specific status
    // Returns a List because multiple users can have the same status
    List<User> findByStatus(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ?

    // ═══════════════════════════════════════════════════════════
    // FIND BY MULTIPLE FIELDS (AND / OR)
    // ═══════════════════════════════════════════════════════════
    
    // Both conditions must be true (AND)
    Optional<User> findByEmailAndStatus(String email, UserStatus status);
    // SQL: SELECT * FROM users WHERE email = ? AND status = ?

    // Either condition can be true (OR)
    List<User> findByFirstNameOrLastName(String firstName, String lastName);
    // SQL: SELECT * FROM users WHERE first_name = ? OR last_name = ?

    // ═══════════════════════════════════════════════════════════
    // COMPARISON OPERATORS — Numbers and Dates
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByAgeLessThan(int age);                   // WHERE age < ?
    List<User> findByAgeGreaterThanEqual(int age);           // WHERE age >= ?
    List<User> findByAgeBetween(int min, int max);           // WHERE age BETWEEN ? AND ?
    List<User> findByCreatedAtAfter(LocalDateTime date);     // WHERE created_at > ?
    List<User> findByCreatedAtBefore(LocalDateTime date);    // WHERE created_at < ?

    // ═══════════════════════════════════════════════════════════
    // STRING MATCHING — Searching Text
    // ═══════════════════════════════════════════════════════════
    
    // Contains — matches anywhere in the string (most common for search)
    List<User> findByFirstNameContaining(String keyword);
    // SQL: WHERE first_name LIKE '%keyword%'

    // Starts with — matches beginning of string
    List<User> findByEmailStartingWith(String prefix);
    // SQL: WHERE email LIKE 'prefix%'

    // Ends with — matches end of string
    List<User> findByLastNameEndingWith(String suffix);
    // SQL: WHERE last_name LIKE '%suffix'

    // Case-insensitive — "JOHN", "john", "John" all match
    List<User> findByFirstNameIgnoreCase(String name);
    // SQL: WHERE LOWER(first_name) = LOWER(?)

    // ═══════════════════════════════════════════════════════════
    // NULL CHECKS
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByBioIsNull();          // WHERE bio IS NULL     (users who haven't written a bio)
    List<User> findByBioIsNotNull();       // WHERE bio IS NOT NULL (users who have written a bio)

    // ═══════════════════════════════════════════════════════════
    // BOOLEAN CHECKS
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByVerifiedTrue();       // WHERE verified = true
    List<User> findByVerifiedFalse();      // WHERE verified = false

    // ═══════════════════════════════════════════════════════════
    // COLLECTION CHECKS — IN
    // ═══════════════════════════════════════════════════════════
    
    // Find users whose status is in a given list
    List<User> findByStatusIn(List<UserStatus> statuses);
    // SQL: WHERE status IN ('ACTIVE', 'PENDING_VERIFICATION')

    List<User> findByStatusNotIn(List<UserStatus> statuses);
    // SQL: WHERE status NOT IN (...)

    // ═══════════════════════════════════════════════════════════
    // ORDERING — Sort Results
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByStatusOrderByCreatedAtDesc(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ? ORDER BY created_at DESC
    // "Desc" = newest first. Use "Asc" for oldest first.

    List<User> findByStatusOrderByLastNameAscFirstNameAsc(UserStatus status);
    // SQL: ... ORDER BY last_name ASC, first_name ASC (multi-column sort)

    // ═══════════════════════════════════════════════════════════
    // LIMITING RESULTS — Top / First
    // ═══════════════════════════════════════════════════════════
    
    // Get only the most recent user (single result)
    User findFirstByOrderByCreatedAtDesc();
    // SQL: SELECT * FROM users ORDER BY created_at DESC LIMIT 1

    // Get the 5 most recently created active users
    List<User> findTop5ByStatusOrderByCreatedAtDesc(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ? ORDER BY created_at DESC LIMIT 5

    // ═══════════════════════════════════════════════════════════
    // COUNT AND EXISTENCE — Quick Checks
    // ═══════════════════════════════════════════════════════════
    
    // How many active users are there?
    long countByStatus(UserStatus status);
    // SQL: SELECT COUNT(*) FROM users WHERE status = ?

    // Does a user with this email exist? (returns true/false — very fast)
    boolean existsByEmail(String email);
    // SQL: SELECT CASE WHEN COUNT(*) > 0 THEN true ELSE false END FROM users WHERE email = ?

    // ═══════════════════════════════════════════════════════════
    // DELETE — Remove by Condition
    // ═══════════════════════════════════════════════════════════
    
    void deleteByStatus(UserStatus status);
    // SQL: DELETE FROM users WHERE status = ?

    long deleteByEmail(String email);
    // Returns count of deleted rows — useful for confirming deletion

    // ═══════════════════════════════════════════════════════════
    // NESTED PROPERTY ACCESS — Query Through Relationships
    // ═══════════════════════════════════════════════════════════
    
    // If User has a @ManyToOne Department, you can query through it!
    // Spring automatically joins the tables for you.
    List<User> findByDepartmentName(String departmentName);
    // SQL: SELECT u.* FROM users u JOIN departments d ON u.department_id = d.id WHERE d.name = ?
}
```

> ⚠️ **Common Mistake — Ambiguous property paths:**  
> If your `User` entity has both a field `departmentName` AND a relationship `department` with a `name` field, Spring gets confused. To be explicit, use an underscore: `findByDepartment_Name` (means `department.name`) vs `findByDepartmentName` (could be either). In practice, avoid naming fields that collide with relationship paths.

> 📝 **Beginner Note — When do derived queries become impractical?**  
> Once your method name gets very long (e.g., `findByStatusAndDepartmentNameAndCreatedAtAfterOrderByLastNameAsc`), it becomes hard to read. At that point, switch to `@Query` with JPQL (covered in Section 5). A good rule of thumb: if the method name has more than 3–4 keywords, use `@Query` instead.

#### Keyword Quick-Reference Table

| Keyword | SQL Equivalent | Example Method |
|---|---|---|
| `And` | `AND` | `findByFirstNameAndLastName` |
| `Or` | `OR` | `findByStatusOrEmail` |
| `Is`, `Equals` | `=` | `findByStatusIs` / `findByStatusEquals` |
| `Not` | `!=` | `findByStatusNot` |
| `Between` | `BETWEEN` | `findByAgeBetween` |
| `LessThan` | `<` | `findByAgeLessThan` |
| `LessThanEqual` | `<=` | `findByAgeLessThanEqual` |
| `GreaterThan` | `>` | `findByAgeGreaterThan` |
| `GreaterThanEqual` | `>=` | `findByAgeGreaterThanEqual` |
| `After` | `>` (for dates) | `findByCreatedAtAfter` |
| `Before` | `<` (for dates) | `findByCreatedAtBefore` |
| `IsNull` | `IS NULL` | `findByEmailIsNull` |
| `IsNotNull` / `NotNull` | `IS NOT NULL` | `findByEmailIsNotNull` |
| `Like` | `LIKE` (you provide the `%`) | `findByNameLike("%john%")` |
| `Containing` | `LIKE %x%` (auto-wraps with `%`) | `findByNameContaining("john")` |
| `StartingWith` | `LIKE x%` | `findByNameStartingWith("J")` |
| `EndingWith` | `LIKE %x` | `findByNameEndingWith("son")` |
| `In` | `IN (...)` | `findByStatusIn(List.of(ACTIVE, PENDING))` |
| `NotIn` | `NOT IN (...)` | `findByStatusNotIn(...)` |
| `True` / `False` | `= true` / `= false` | `findByVerifiedTrue` |
| `IgnoreCase` | `LOWER()` comparison | `findByEmailIgnoreCase` |
| `OrderBy` | `ORDER BY` | `findByStatusOrderByCreatedAtDesc` |
| `Top` / `First` | `LIMIT` | `findTop10ByStatus` / `findFirstByOrderByIdAsc` |

---

### 4.4 Pagination & Sorting

#### What is Pagination?

When your database has thousands (or millions) of records, loading all of them at once is **slow**, **memory-intensive**, and **bad UX**. Pagination lets you load data in small chunks — one "page" at a time.

#### What is Sorting?

Sorting lets you control the **order** in which results are returned — newest first, alphabetical, highest price, etc.

> 🧠 **Analogy — Google Search Results:**  
> When you search on Google, it doesn't dump all 10 million results on your screen. It shows you **10 results per page** (that's pagination), sorted by **relevance** (that's sorting). You click "Next" to see the next page. That's exactly what `Pageable` does for your database queries.

#### Key Concepts

| Concept | What It Is |
|---|---|
| `Pageable` | An object that holds the **page number**, **page size**, and **sort order** — you pass it to repository methods |
| `PageRequest` | The most common way to **create** a `Pageable` instance |
| `Page<T>` | The result object — contains the data **plus** metadata (total pages, total elements, etc.) |
| `Slice<T>` | A lighter result object — contains the data and knows if there's a **next page**, but does NOT count total records (faster!) |
| `Sort` | Defines how to order results — by which field(s), ascending or descending |

> 📝 **Beginner Note — Page vs Slice:**  
> - `Page` runs **two SQL queries**: one for the data, one for `COUNT(*)` to calculate total pages/elements. Use this when you need to show "Page 3 of 15" in the UI.
> - `Slice` runs only **one SQL query** (fetches one extra record to check if there's a next page). Use this for "Load More" buttons or infinite scroll — faster because it skips the count.

#### Step-by-Step Example: Repository → Service → Controller

**Step 1: Repository — Accept `Pageable` as a parameter**

Just add a `Pageable` parameter to any query method. Spring Data JPA handles the rest:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Paginated query — returns a Page with metadata (total count, total pages, etc.)
    Page<User> findByStatus(UserStatus status, Pageable pageable);

    // Slice — lighter alternative, no count query (good for "Load More" UIs)
    Slice<User> findByDepartmentId(Long deptId, Pageable pageable);

    // You can also paginate custom @Query methods:
    @Query("SELECT u FROM User u WHERE u.firstName LIKE %:name%")
    Page<User> searchByName(@Param("name") String name, Pageable pageable);
}
```

**Step 2: Service — Build the `Pageable` object**

The service layer creates a `Pageable` using `PageRequest.of()`:

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    // Simple pagination — no sorting
    public Page<User> getActiveUsers(int page, int size) {
        // page = 0-indexed! Page 0 = first page, Page 1 = second page, etc.
        // size = number of records per page
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findByStatus(UserStatus.ACTIVE, pageable);
    }

    // Pagination WITH sorting
    public Page<User> getActiveUsersSorted(int page, int size) {
        // Sort by createdAt descending (newest first), then by firstName ascending (A→Z)
        Sort sort = Sort.by(Sort.Direction.DESC, "createdAt")
                        .and(Sort.by(Sort.Direction.ASC, "firstName"));

        Pageable pageable = PageRequest.of(page, size, sort);
        return userRepository.findByStatus(UserStatus.ACTIVE, pageable);
    }
}
```

> ⚠️ **Common Mistake — Page numbers are 0-indexed!**  
> `PageRequest.of(0, 10)` = first page (records 1–10). `PageRequest.of(1, 10)` = second page (records 11–20). If your frontend sends `page=1` meaning "first page," you'll need to subtract 1!

**Step 3: Controller — Expose pagination via REST API**

The controller receives pagination parameters from the client (query params) and passes them to the service:

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    // GET /api/users?page=0&size=10&sortBy=createdAt&direction=desc
    @GetMapping
    public ResponseEntity<Page<UserDto>> getUsers(
            @RequestParam(defaultValue = "0") int page,       // Default: first page
            @RequestParam(defaultValue = "10") int size,      // Default: 10 per page
            @RequestParam(defaultValue = "createdAt") String sortBy,   // Default: sort by creation date
            @RequestParam(defaultValue = "desc") String direction) {   // Default: newest first

        // Build Sort object from query params
        Sort sort = direction.equalsIgnoreCase("desc")
                ? Sort.by(sortBy).descending()
                : Sort.by(sortBy).ascending();

        Pageable pageable = PageRequest.of(page, size, sort);
        Page<User> users = userService.findAll(pageable);

        // Convert entities to DTOs (never expose entities directly in API!)
        Page<UserDto> dtoPage = users.map(UserMapper::toDto);
        return ResponseEntity.ok(dtoPage);
    }
}
```

#### What does the `Page` response look like?

When your REST API returns a `Page`, Spring automatically serializes it to JSON with this structure:

```json
{
  "content": [                   // ← The actual data (list of users on this page)
    { "id": 1, "firstName": "Alice", "email": "alice@example.com" },
    { "id": 2, "firstName": "Bob", "email": "bob@example.com" }
  ],
  "totalElements": 150,          // ← Total records matching the query in the entire DB
  "totalPages": 15,              // ← Total number of pages (totalElements ÷ size)
  "number": 0,                   // ← Current page number (0-indexed)
  "size": 10,                    // ← Page size (records per page)
  "first": true,                 // ← Is this the first page?
  "last": false,                 // ← Is this the last page?
  "numberOfElements": 10,        // ← How many elements are on THIS page
  "empty": false                 // ← Is this page empty?
}
```

> 📝 **Beginner Note — Using Sort with multiple fields:**  
> You can sort by multiple fields. For example, to sort by status first, then by name:
> ```java
> Sort sort = Sort.by("status").ascending()
>                 .and(Sort.by("firstName").ascending());
> ```
> This produces: `ORDER BY status ASC, first_name ASC`

> 💡 **Important to Remember:** You can add `Pageable` as a parameter to **any** repository method — built-in methods, derived query methods, and even `@Query` methods. Spring Data JPA handles the `LIMIT` and `OFFSET` SQL for you.

---

## 5. Query Mechanisms in Spring Data JPA

Spring Data JPA gives us different ways to fetch, search, update, and delete database data.

### 5.1 Which query method should I use?

| Method | Simple meaning | Use it when |
|---|---|---|
| **Derived query** | Spring creates a query from the method name | The query is simple, such as finding a user by email |
| **JPQL (`@Query`)** | Write a query using Java entity and field names | You need joins, counts, filters, or selected fields |
| **Native SQL** | Write regular database SQL | You need a database-specific feature or complex SQL |
| **Named query** | Define a query on an entity and call it by name | An existing project uses them or queries are intentionally defined there |

---

### 5.2 JPQL — Query Using Java Class and Field Names

**JPQL looks like SQL, but uses entity class names and Java field names instead of database table and column names.** Hibernate converts JPQL into SQL for the database.

Imagine your entity is `User`, mapped to the database table `users`:

```java
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);
```

**What this means:** Find the `User` whose `email` matches the value passed to the method.

- `User` is the Java entity class.
- `u.email` is the Java field.
- `:email` is a named placeholder for the method argument.
- `@Param("email")` connects the method argument to that placeholder.

**JPQL vs SQL**

```sql
-- JPQL: entity and field names
SELECT u FROM User u WHERE u.email = :email

-- SQL: table and column names
SELECT * FROM users WHERE email = ?
```

Use named parameters such as `:email` instead of joining user input into a query string. This is easier to read and helps avoid SQL injection.

#### 5.2.1 Common JPQL examples

**Find users in a department** — `JOIN` lets the query use a related entity:

```java
@Query("SELECT u FROM User u JOIN u.department d WHERE d.name = :deptName")
List<User> findByDepartmentName(@Param("deptName") String deptName);
```

**Load users and their departments together** — `JOIN FETCH` can prevent the N+1 problem when you need the related data:

```java
@Query("SELECT u FROM User u JOIN FETCH u.department WHERE u.status = :status")
List<User> findWithDepartmentByStatus(@Param("status") UserStatus status);
```

**Search names without case sensitivity:**

```java
@Query("SELECT u FROM User u WHERE LOWER(u.firstName) LIKE LOWER(CONCAT('%', :name, '%'))")
List<User> searchByName(@Param("name") String name);
```

For example, searching for `john` can match `John`, `JOHN`, or `Johnson`.

**Count users with a given status:**

```java
@Query("SELECT COUNT(u) FROM User u WHERE u.status = :status")
long countByStatus(@Param("status") UserStatus status);
```

**Return only selected fields using a DTO:**

```java
@Query("SELECT new com.example.dto.UserSummaryDto(u.id, u.firstName, u.email) FROM User u WHERE u.status = 'ACTIVE'")
List<UserSummaryDto> findActiveUserSummaries();
```

The DTO must have a matching constructor. This fetches only the fields needed instead of loading the full entity.

**Update or delete many rows:**

```java
@Modifying
@Transactional
@Query("UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
int updateStatus(@Param("oldStatus") UserStatus oldStatus,
                 @Param("newStatus") UserStatus newStatus);
```

- Use `@Modifying` for JPQL `UPDATE` and `DELETE` queries.
- Use a transaction for these data changes.
- The returned `int` is the number of rows changed.
- After a bulk update, entities already loaded in the current persistence context may still have old values. `@Modifying(clearAutomatically = true)` can clear that context.

**Remember:** JPQL is usually the best choice for custom queries because it uses Java entity names and is not tied to one database.

---

### 5.3 Native SQL — Write Regular SQL

A **native query** is normal SQL sent directly to the database. Use database table and column names.

Use native SQL when you need features JPQL cannot easily express, such as window functions, CTEs, or database-specific functions. For normal searches and joins, prefer a derived query or JPQL.

```java
@Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
Optional<User> findByEmailNative(@Param("email") String email);
```

Here, `users` is the database table name. `nativeQuery = true` tells Spring this is SQL, not JPQL.

#### 5.3.1 Native query with pagination

```java
@Query(
    value = "SELECT * FROM users WHERE status = :status",
    countQuery = "SELECT COUNT(*) FROM users WHERE status = :status",
    nativeQuery = true
)
Page<User> findByStatusNative(@Param("status") String status, Pageable pageable);
```

The `countQuery` tells Spring how many matching rows exist so it can build the `Page` result.

#### 5.3.2 Things to remember

- Native SQL uses table and column names, not entity and field names.
- It may rely on features specific to your database, so changing databases may require changes to the query.
- Errors in native SQL are generally found when the query runs, so test it carefully.
- When selecting joined or partial columns, the result may need a projection or result mapping rather than mapping directly to an entity.

**Remember:** Use native SQL when you need direct control or a database feature that JPQL does not support.

---

### 5.4 Named Queries — Define a Query Once and Reuse It by Name

A **named query** is defined on an entity class and used by a repository method with a matching name.

```java
@Entity
@Table(name = "users")
@NamedQuery(
    name = "User.findActive",
    query = "SELECT u FROM User u WHERE u.status = 'ACTIVE'"
)
public class User {
    // entity fields
}
```

Repository:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findActive();
}
```

Spring can look for the named query `User.findActive` based on the entity name and method name.

Named queries can be checked when the application starts, which can help reveal query mistakes early. However, they put query definitions on the entity, so many modern projects prefer `@Query` directly beside the repository method.

**Remember:** Named queries are useful to know, but `@Query` is often simpler for new code.

---

### 5.5 Quick Comparison: JPQL vs Native SQL

| | JPQL | Native SQL |
|---|---|---|
| Uses | Entity and field names | Table and column names |
| Converted by Hibernate? | Yes | SQL is sent directly to the database |
| Works across databases? | Usually more portable | May depend on the database |
| Best for | Most custom queries | Database-specific or advanced SQL |

### 5.6 Interview Revision

- **Derived query:** Spring creates the query from the repository method name.
- **JPQL:** Write queries using Java entity and field names; Hibernate converts them to SQL.
- **Native SQL:** Write the database SQL yourself using table and column names.
- **Named query:** Define a query on the entity and call it by name.
- **`@Param`:** Connects a method argument to a named query parameter such as `:email`.
- **`@Modifying`:** Marks a JPQL query that updates or deletes data.
- **`JOIN FETCH`:** Loads related data in the same query when needed, which can help avoid N+1 queries.
- **DTO projection:** Fetches only the fields needed for a response.

---

## 6. Advanced Topics

> This section covers the concepts that separate a beginner JPA developer from a confident, production-ready one. These topics — the N+1 problem, entity lifecycle, dirty checking, transactions, locking, and caching — are the areas where real-world bugs and performance issues live. Master these, and you'll debug JPA problems faster than most senior developers.

### 6.1 The N+1 Problem

> ⚠️ **CRITICAL — Most Common Performance Issue**

#### What is the N+1 Problem?

The N+1 problem is the **#1 performance killer** in JPA applications. It happens when your code triggers **1 query** to load a list of entities, and then **N additional queries** — one for each entity — to load a related object. If you have 1,000 employees, that's 1,001 database round-trips instead of 1.

#### Why does it happen?

It happens because of **lazy loading** (Section 3.4). When you load a list of entities, their related objects aren't fetched immediately — they're loaded one-by-one as you access them in a loop. Each access triggers a separate SQL query.

#### What problem does solving it fix?

A page that takes 10 seconds to load (because of 1,001 queries) can load in 50 milliseconds (with 1 query) once you fix the N+1 problem. This is often the single biggest performance improvement you can make in a JPA application.

> 🧠 **Analogy — Grocery Shopping:**  
> Imagine you need 20 items from the grocery store.
> - **N+1 way (terrible):** Drive to the store, buy 1 item, drive home. Then drive to the store again, buy the 2nd item, drive home. Repeat 20 times. That's **21 trips** (1 to make the list + 20 to buy each item).
> - **Join fetch way (smart):** Drive to the store once, put all 20 items in your cart, drive home. **1 trip.**
>
> Every SQL query is a "trip" to the database. The network round-trip cost adds up fast — even if each individual query is fast, 1,000 of them are slow.

#### Seeing the Problem in Action

Here's exactly how the N+1 problem looks in code and what SQL is generated:

```java
// ❌ BAD — N+1 Problem in action
// Step 1: Hibernate runs 1 query to get all employees
List<Employee> employees = employeeRepository.findAll();
// SQL: SELECT * FROM employees                          ← 1 query

// Step 2: For EACH employee, you access the department (which is LAZY)
// Hibernate runs a SEPARATE query for each employee's department!
for (Employee e : employees) {
    System.out.println(e.getDepartment().getName());
    // SQL: SELECT * FROM departments WHERE id = 1       ← query 2
    // SQL: SELECT * FROM departments WHERE id = 2       ← query 3
    // SQL: SELECT * FROM departments WHERE id = 3       ← query 4
    // ... and so on for EVERY employee!
}

// If you have 1000 employees → 1 + 1000 = 1001 SQL queries!
// Even if each query takes just 5ms, that's 5 SECONDS of just waiting for the database.
```

> 📝 **Beginner Note — How to detect the N+1 problem:**  
> Turn on `spring.jpa.show-sql=true` in your `application.properties` and watch the console. If you see the same `SELECT * FROM departments WHERE id = ?` repeated dozens (or hundreds) of times, you have an N+1 problem. In production, tools like **p6spy** or **datasource-proxy** can count queries automatically in tests.

#### Solution 1: JOIN FETCH in JPQL (Most Common Fix)

`JOIN FETCH` tells Hibernate: *"Load the related data in the SAME query — don't wait for me to access it."*

```java
// ✅ SOLUTION 1: JOIN FETCH — load employees AND departments in ONE query
@Query("SELECT e FROM Employee e JOIN FETCH e.department")
List<Employee> findAllWithDepartment();
// Generated SQL: SELECT e.*, d.* FROM employees e 
//                JOIN departments d ON e.department_id = d.id
// ✅ Just 1 query — no matter how many employees!
```

**Step-by-step: What happens behind the scenes:**

1. Hibernate generates a single SQL `JOIN` query that fetches employees AND departments together
2. For each row in the result, Hibernate builds an `Employee` object AND its associated `Department` object
3. Since the department is already loaded, accessing `employee.getDepartment()` later does NOT trigger another query
4. Result: 1 query instead of N+1

> 📝 **Beginner Note — `JOIN FETCH` vs regular `JOIN`:**  
> A regular `JOIN` only uses the joined table for filtering (WHERE clause). `JOIN FETCH` goes further — it **loads the joined data into the entity objects**. Always use `JOIN FETCH` when you want to avoid lazy loading queries.

#### Solution 2: @EntityGraph (Declarative Approach — No JPQL Needed)

`@EntityGraph` is a cleaner, annotation-based way to say *"when you run this query, also load these relationships."* You don't need to write any JPQL.

```java
// ✅ SOLUTION 2a: @EntityGraph directly on the repository method
// Just list the relationships you want loaded eagerly for THIS specific query
@EntityGraph(attributePaths = {"department", "address"})
List<Employee> findByStatus(EmployeeStatus status);
// Spring Data JPA generates: SELECT e.*, d.*, a.* FROM employees e
//   LEFT JOIN departments d ON ... LEFT JOIN addresses a ON ...
//   WHERE e.status = ?
// ✅ One query loads employees + departments + addresses
```

You can also define a **named entity graph** on the entity class and reuse it across multiple repository methods:

```java
// Define the graph on the entity — reusable across multiple queries
@Entity
@NamedEntityGraph(
    name = "Employee.withDepartment",
    attributeNodes = @NamedAttributeNode("department")
)
public class Employee { ... }

// Use the named graph in any repository method
@EntityGraph("Employee.withDepartment")
List<Employee> findAll();
// ✅ Uses the pre-defined graph to load department data
```

> 💡 **Important to Remember — When to use `@EntityGraph` vs `JOIN FETCH`:**
> - Use `@EntityGraph` when you want to **add eager loading to an existing derived query method** without rewriting it as JPQL
> - Use `JOIN FETCH` in JPQL when you're already writing a custom `@Query` and need full control over the join conditions
> - Both solve the N+1 problem equally well — choose based on readability and convenience

#### Solution 3: @BatchSize (Reduce N+1 to N/batch+1)

`@BatchSize` doesn't eliminate extra queries entirely, but it **dramatically reduces them** by loading related entities in batches instead of one-by-one.

```java
// ✅ SOLUTION 3: @BatchSize — load related entities in batches
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@BatchSize(size = 30)  // Load up to 30 departments' employees at a time
private List<Employee> employees;
// Instead of 100 separate queries, Hibernate runs:
// SELECT * FROM employees WHERE department_id IN (1, 2, 3, ..., 30)  ← batch 1
// SELECT * FROM employees WHERE department_id IN (31, 32, ..., 60)   ← batch 2
// ... etc.
// 100 queries → ~4 queries (100 / 30 ≈ 4 batches)
```

> 🧠 **Analogy — Batch Loading = Carpooling:**  
> Without `@BatchSize`, each department sends its own car to pick up its employees (one query each = many cars on the road). With `@BatchSize(size = 30)`, you send a bus that picks up employees from 30 departments at once. Fewer trips, same result.

> 📝 **Beginner Note — When to use `@BatchSize`:**  
> Use it when `JOIN FETCH` isn't practical — for example, when you have multiple collections that can't all be fetched in one join (Hibernate limits multi-collection fetch joins to avoid Cartesian products). `@BatchSize` is a quick, low-effort improvement you can add to any `@OneToMany` or `@ManyToMany` field.

#### Solution 4: @Fetch(FetchMode.SUBSELECT) (Hibernate-Specific)

This loads **all** related children in a single subselect query — regardless of how many parents there are.

```java
// ✅ SOLUTION 4: Fetch subselect — one query loads ALL children
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT)
private List<Employee> employees;
// When you access employees on ANY department, Hibernate runs:
// SELECT * FROM employees WHERE department_id IN 
//   (SELECT id FROM departments)   ← one subselect gets them ALL
// ✅ Exactly 2 queries total: 1 for departments + 1 for ALL employees
```

> 📝 **Beginner Note — Choosing the right N+1 solution:**

| Solution | Queries | Effort | Best For |
|---|---|---|---|
| `JOIN FETCH` | 1 | Write JPQL | Most cases — your default go-to |
| `@EntityGraph` | 1 | Add annotation | When you want to keep derived query methods |
| `@BatchSize` | N/batch + 1 | Add annotation | Multiple collections, quick fixes |
| `@Fetch(SUBSELECT)` | 2 | Add annotation | When you always load all parents |

> ⚠️ **Common Mistake — Not detecting N+1 in development:**  
> The N+1 problem is silent — your app works correctly, it's just slow. Always enable `spring.jpa.show-sql=true` during development and **count your queries**. If a single page load generates 50+ queries, you almost certainly have an N+1 problem.

---

### 6.2 JPA Entity Lifecycle

An **entity** is a Java object that represents a row in a database table.

JPA entities can be in four states. The state tells us whether JPA is tracking the object and whether changes can be saved automatically.

#### 6.2.1 The Four States

| State | Simple meaning | Example |
|---|---|---|
| **Transient** | A new Java object that JPA is not tracking. | `new User()` |
| **Managed** | JPA is tracking the object. Changes are saved when the transaction commits. | An entity loaded inside a transaction |
| **Detached** | The object was tracked before, but JPA is no longer tracking it. Changes are not saved automatically. | An entity after its persistence context ends |
| **Removed** | The entity is marked for deletion. The database row is deleted when the transaction commits. | An entity passed to `remove()` or deleted through a repository in a transaction |

#### 6.2.2 Simple Lifecycle Flow

```text
Transient --persist/save--> Managed --remove/delete--> Removed
                               |
                     transaction ends, detach, or clear
                               v
                            Detached
                               |
                         merge/save
                               v
                            Managed
```

> **Remember:** Managed = tracked. Detached = not tracked.

#### 6.2.3 Example of Each State

##### A. Transient — just a Java object

```java
User user = new User();
user.setEmail("alice@example.com");
```

JPA is not tracking this object, so creating or changing it does not save anything to the database.

##### B. Managed — JPA tracks changes

```java
@Transactional
public void updateEmail(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    user.setEmail("new@example.com");
}
```

The entity is managed inside the transaction. When the method finishes successfully, Hibernate detects the change and updates the database. This is called **dirty checking**; you do not need to call `save()` for this change.

##### C. Detached — changes are not tracked

```java
User user = userRepository.findById(1L).orElseThrow();
user.setEmail("new@example.com");
```

If the repository call's transaction has ended, the returned entity is typically detached. Changing its email does **not** automatically update the database.

To save changes to a detached entity, call `save()` in a suitable transaction and use the returned object if you need the managed instance:

```java
@Transactional
public User updateDetachedUser(User user) {
    user.setEmail("new@example.com");
    return userRepository.save(user);
}
```

Spring Data JPA uses `merge` when it needs to copy changes from an existing detached entity into a managed entity. The returned object may be a different Java object from the one passed in.

##### D. Removed — marked for deletion

```java
@Transactional
public void deleteUser(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    userRepository.delete(user);
}
```

The entity is marked for deletion. The SQL `DELETE` is executed as the transaction is flushed or committed.

#### 6.2.4 What Is `LazyInitializationException`?

A relationship may be **lazy-loaded**, meaning JPA fetches its data only when you first access it.

If you access that relationship after the entity's persistence context has closed, JPA may no longer be able to load it and can throw `LazyInitializationException`.

**Simple fix:** Load the required relationship while the transaction is active, for example with `JOIN FETCH` or `@EntityGraph`.

#### 6.2.5 Common Mistakes

- **Changing a detached entity:** the change is not automatically saved. Save/merge it in a transaction, or load it again inside a transaction and update it there.
- **Assuming `save()` always returns the same object:** for an existing detached entity, `save()` may return a different managed object. Use the returned value when needed.
- **Accessing lazy data too late:** fetch the data before the persistence context closes.
- **Calling `save()` for every managed change:** inside a transaction, changes to a managed entity are detected automatically at commit.

#### 6.2.6 Interview Answer

“JPA entities have four states: transient, managed, detached, and removed. A transient object is not tracked by JPA. A managed entity is tracked, so its changes can be saved automatically when the transaction commits. A detached entity is no longer tracked, so its changes are not saved automatically. A removed entity is marked for deletion.”

---

### 6.3 Dirty Checking

#### 6.3.1 What Is Dirty Checking?

**Dirty checking means Hibernate notices changes to a managed entity and saves those changes to the database automatically.** You usually do not need to call `save()` again.

Think of it like auto-save in a document: change the value, and Hibernate detects the change.

#### 6.3.2 How It Works

1. Load an entity inside a `@Transactional` method.
2. Hibernate tracks the entity and remembers its original values.
3. Change a field, such as the user's email.
4. When Hibernate flushes changes (often at transaction commit), it detects the difference and runs an `UPDATE` query.

#### 6.3.3 Example — Dirty Checking Works

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    @Transactional
    public void updateEmail(Long userId, String newEmail) {
        User user = userRepository.findById(userId).orElseThrow();
        user.setEmail(newEmail);
        // No save() needed: Hibernate detects the change.
    }
}
```

**What happens:** The user is loaded and tracked in the transaction. When the transaction commits, Hibernate detects the changed email and updates the database.

#### 6.3.4 When Dirty Checking Does Not Work

Dirty checking does not automatically save changes to an entity that is **detached** (no longer tracked by Hibernate).

```java
public void updateEmail(Long userId) {
    User user = userRepository.findById(userId).orElseThrow();
    // The repository transaction has ended; this entity may now be detached.

    user.setEmail("new@email.com");
    // This change is not automatically saved.
}
```

**Fix:** Make the update inside a service method annotated with `@Transactional`, as shown in the previous example.

#### 6.3.5 Important Points

- **Managed entity:** Hibernate tracks it; changes can be saved automatically.
- **Detached entity:** Hibernate no longer tracks it; changes are not automatically saved.
- **Transient entity:** A new object that has not been saved; changing it does not update the database.
- `save()` is usually unnecessary when changing an already managed entity inside the same transaction.
- **Dirty checking** detects changes. **Flush** sends pending SQL to the database. **Commit** completes the transaction.
- By default, an `UPDATE` may include more columns than just the changed field. Hibernate's `@DynamicUpdate` can make the SQL include only changed columns:

```java
@Entity
@DynamicUpdate
public class User {
    // fields...
}
```

#### 6.3.6 Interview Answer

“Dirty checking is a Hibernate feature that detects changes to managed entities and updates the database automatically when changes are flushed. If the entity is managed inside a `@Transactional` method, I usually don't need to call `save()` after changing its fields.”

---

### 6.4 JPA Transactions (`@Transactional`)

#### 6.4.1 What is a transaction?

A **transaction** is a group of database operations that should **all succeed or all fail together**. 🎯

Example: transferring ₹500 between two bank accounts 🏦:

1. Subtract ₹500 from **Account A**.
2. Add ₹500 to **Account B**.

If step 2 fails, step 1 should **also be undone**.

- ✅ **Commit:** Keep the database changes.
- ↩️ **Rollback:** Undo the database changes made by the transaction.

#### 6.4.2 ACID properties

ACID describes four important rules that help database transactions stay reliable.

| Property | Simple meaning | Example |
|---|---|---|
| ⚛️ **Atomicity** | **All** operations happen, or **none** do. | Money is not taken from A unless it is added to B. |
| ⚖️ **Consistency** | The data follows the database rules before and after the transaction. | An account balance must follow the application's allowed rules. |
| 🔐 **Isolation** | Transactions running at the same time should not interfere in unsafe ways. | Two transfers should not accidentally overwrite each other's updates. |
| 💾 **Durability** | Once **committed**, the changes **remain saved**, even after a restart. | A completed transfer remains recorded after the application restarts. |

> 🎯 **Interview tip:** **Atomicity** = all or nothing; **Consistency** = valid data; **Isolation** = safe concurrent work; **Durability** = committed data stays saved.

#### 6.4.3 Spring transaction management and `@Transactional`

Spring transaction management handles starting, committing, and rolling back database transactions for us.

`@Transactional` is an annotation in Spring Boot that **manages database transactions automatically**. It helps ensure that a group of database operations either **completes successfully as a whole** or is **rolled back** when a qualifying failure occurs.

##### Simplified example

```java
@Service
public class BankService {

    private final AccountRepository repo;

    public BankService(AccountRepository repo) {
        this.repo = repo;
    }

    @Transactional
    public void transferMoney(Long fromId, Long toId, BigDecimal amount) {

        Account from = repo.findById(fromId).orElseThrow();
        Account to = repo.findById(toId).orElseThrow();

        if (from.getBalance().compareTo(amount) < 0) {
            throw new RuntimeException("Insufficient balance");
        }

        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
    }
}
```

##### Simple explanation

`@Transactional` ensures both account updates happen as one transaction.

- ✅ **Success:** Money is deducted from one account and added to the other. Spring **commits** the changes to the database.
- ❌ **Failure:** If an **unchecked exception** occurs, Spring **rolls back** the transaction.
- 💡 **No `save()` needed:** JPA automatically **detects changes** to the loaded accounts and saves them when the transaction commits.

> 💡 **Remember:** Either both account changes succeed, or the database changes are rolled back.

#### 6.4.4 Transaction boundaries

A **transaction boundary** is where a transaction **starts** and where it **finishes**.

With Spring's usual proxy-based setup:

1. A call comes into a Spring-managed transactional method through its proxy.
2. Spring starts or joins a transaction.
3. The method performs database work.
4. If it completes successfully, Spring commits; if a rollback-triggering exception escapes, Spring rolls back.

Keep related database changes inside one service method when they must succeed or fail together. ⚠️ A database rollback **cannot automatically undo an external action**, such as an email or a payment already sent to another system.

#### 6.4.5 Rollback rules

Rollback means **undoing database changes**.

| Rule              | Easy meaning                                          |
| ----------------- | ----------------------------------------------------- |
| ✅ Default         | Rollback for `RuntimeException` and `Error`.          |
| Checked exception | No rollback by default, e.g. `IOException`.           |
| `rollbackFor`     | Tell Spring when to rollback.                         |
| `noRollbackFor`   | Tell Spring not to rollback for a specific exception. |

**🧩 Example: Rollback for all exceptions**

```java
@Transactional(rollbackFor = Exception.class)
public void importData() throws Exception {
    // Update database
    throw new Exception("Something went wrong");
}
```

Here, Spring rolls back the database changes if the exception escapes the method.

> 💡 **Remember:**
>
> - `RuntimeException` → ✅ **rollback** by default.
> - Checked exception → ❌ **no rollback** by default.
> - `rollbackFor` → rollback for the **specified** exception.
> - ⚠️ If you **catch** an exception and **do not rethrow** it, Spring may **commit** the changes.

#### 6.4.6 Transaction Propagation

**Propagation** decides what happens when a transactional method calls another method that also uses `@Transactional`.

There are 7 types, but the most important ones are `REQUIRED` and `REQUIRES_NEW`.

##### 1. Types of propagation

| Type                 | Easiest meaning                                                      |
| -------------------- | -------------------------------------------------------------------- |
| `REQUIRED` (default) | **Use the existing** transaction, or create one if needed.               |
| `REQUIRES_NEW`       | Start a **separate** transaction.                                        |
| `NESTED`             | Create a savepoint inside the current transaction, if supported.     |
| `SUPPORTS`           | Use a transaction if one already exists; otherwise, run without one. |
| `NOT_SUPPORTED`      | Run without a transaction.                                           |
| `MANDATORY`          | Must have an existing transaction, or throw an exception.            |
| `NEVER`              | Must not have an existing transaction, or throw an exception.        |

##### 2. Easy example: `REQUIRED`

Imagine you are placing an order 🛒.

```java
@Transactional
public void placeOrder() {
    saveOrder();
    updateStock();
}
```

Both operations use the **same transaction**.

- ✅ If both succeed, the transaction **commits**.
- ❌ If a qualifying exception causes rollback, **both** database changes **roll back**.

💡 `REQUIRED` is the **default**, so it is usually what you need.

##### 3. 🆕 Easy example: `REQUIRES_NEW`

Imagine you want to save an audit record even if placing the order fails 📝.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit() {
    auditRepository.save(new AuditRecord());
}
```

`REQUIRES_NEW` starts a **separate transaction**.

**🧩 For example:**

- ✅ The audit record is saved successfully.
- ❌ The order later fails and rolls back.
- 💾 The audit record can still remain saved because it has its own transaction.

> 💡 **Remember:**
>
> - `REQUIRED` = **Same** transaction.
> - `REQUIRES_NEW` = **Separate** transaction.

For most normal service methods, start with `REQUIRED`. Use `REQUIRES_NEW` when a task must **commit independently** of the main transaction.

> 🎯 **Interview answer:** `REQUIRES_NEW` is a Spring transaction propagation type that **always starts a new transaction**. If a transaction is already active, Spring **suspends** it temporarily. The new transaction commits or rolls back independently of the suspended transaction.

#### 6.4.7 Isolation Levels in Spring Boot

**Isolation** controls what one transaction can see while other transactions are running at the same time.

##### 1. Analogy: Two people using the same spreadsheet

Imagine two bank employees, A and B, checking and updating the same account balance at the same time.

Isolation levels control what each employee can see while the other is making changes.

**Example:** Account balance is ₹1,000

| Employee | Action |
|---|---|
| **Employee A** | Updates balance |
| **Employee B** | Reads balance |

The database decides whether Employee B can see A's uncommitted changes, and whether B sees different data when reading again.

##### 2. Types of isolation levels

| Level              | Easiest meaning                                                          |
| ------------------ | ------------------------------------------------------------------------ |
| `DEFAULT`          | Use the **database's default** setting.                                  |
| `READ_UNCOMMITTED` | May read changes that are not saved permanently yet.                     |
| `READ_COMMITTED`   | Read only changes that have been committed.                              |
| `REPEATABLE_READ`  | Reading the same row again gives a stable result during the transaction. |
| `SERIALIZABLE`     | **Strongest** isolation; transactions behave as if they run one at a time.   |

##### 3. Understand with simple examples

###### `READ_UNCOMMITTED` — Reading unfinished work

Employee A changes the balance to ₹500 but hasn't committed the change yet.

Employee B sees ₹500. Then Employee A cancels the change.

⚠️ Employee B saw data that was never committed. This is called a **dirty read**.

###### `READ_COMMITTED` — Reading only saved work

Employee A changes the balance and commits it.

Employee B can see the new balance **only after A commits**.

However, if B reads the balance again later, it may be different because another transaction has updated it.

###### `REPEATABLE_READ` — Keeping the same row stable

Employee B reads an account balance of ₹1,000.

Even if Employee A updates that account, B's repeated read of the same row **remains stable** for the transaction under this isolation level. The exact behavior can depend on the database.

###### `SERIALIZABLE` — Taking turns

Imagine only one employee is allowed to perform conflicting work at a time.

This provides **stronger protection** against concurrent transaction problems, but can make the application **slower** or cause transactions to retry.

##### 4. What are dirty reads, non-repeatable reads and phantom reads?

- **Dirty read:** You read another transaction's changes before they are committed.
- **Non-repeatable read:** You read the same row twice and get different values.
- **Phantom read:** You run the same search twice and get a different set of rows, because matching rows were added or removed.

##### 5. Simple Spring Boot example

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public User getUser(Long id) {
    return userRepository.findById(id).orElseThrow();
}
```

Here, `READ_COMMITTED` tells Spring to use that isolation level for the transaction, assuming the transaction manager and database support it.

> 💡 **Remember:**
>
> - ✅ `READ_COMMITTED` is a **common choice**.
> - ⚖️ **Higher isolation** generally provides stronger protection against concurrency problems, but can **reduce performance**.
> - `DEFAULT` uses the **database's configured default**.
> - Exact behavior depends on the database, and isolation settings generally take effect when a new transaction starts.

#### 6.4.8 Read-only transactions

**`readOnly = true`** means: “This method is only supposed to read data, not change it.”

##### Analogy: Library

Imagine you visit a library.

You can read books, but you are not supposed to change the books.

Similarly, a read-only transaction is meant for fetching data from the database without modifying it.

##### Simple Spring Boot example

```java
@Transactional(readOnly = true)
public List<User> getUsers() {
    return userRepository.findAll();
}
```

**🔎 What does this do?**

- `@Transactional` — manages the database transaction.
- `readOnly = true` — tells Spring that this method is only meant to read data.
- `findAll()` — fetches all users from the database.

Spring and the database framework may use this information to **improve performance** ⚡.

##### Important to remember

⚠️ `readOnly = true` **does not guarantee** that database changes are blocked. It is mainly a **hint for optimization**.

Use it for methods that only fetch data, such as `getUsers()` or `getUserById()`.

> 💡 **Easy rule:** Reading data → `readOnly = true`. Changing data → use a normal `@Transactional` method when a transaction is needed.

#### 6.4.9 Proxy-based transaction management and self-invocation

Spring usually applies `@Transactional` using a **proxy** 🎭. A proxy is a Spring-created wrapper that intercepts a method call and applies transaction behavior.

For this to work, the call **must go through the proxy**. Calling another method in the same class directly with `this` **bypasses** it.

```java
@Service
public class ReportService {

    public void generateReport() {
        this.loadData(); // Direct call inside this class: bypasses the proxy
    }

    @Transactional(readOnly = true)
    public void loadData() {
        // Read data
    }
}
```

Here, calling `generateReport()` does **not** activate a new transaction just because `loadData()` has `@Transactional`.

> ✅ **Simple fix:** Move `loadData()` to another Spring-managed service and call that service through its injected dependency. Then the call passes through the proxy.

**⭐ Best practices:**

- ✅ Put `@Transactional` on **service-layer** methods that define the unit of work.
- ✅ Prefer **public** methods called from another Spring bean.
- ❌ Avoid relying on `@Transactional` on private methods or on self-invocation (`this.someMethod()`).
- 📌 A class-level `@Transactional` provides defaults for methods in that class; method-level settings can override those defaults.

#### 6.4.11 Quick revision

- 🔄 **Transaction:** A group of database operations that succeed or fail together.
- 🧪 **ACID:** Atomicity, Consistency, Isolation, Durability.
- 🏷️ **`@Transactional`:** Lets Spring manage transaction boundaries.
- ✅ **Commit:** Keep changes. ↩️ **Rollback:** Undo database changes in the transaction.
- 🔀 **Propagation:** Decides whether a method joins or starts a transaction; `REQUIRED` is the default.
- 🔐 **Isolation:** Controls what concurrent transactions can see.
- 📖 **`readOnly = true`:** Marks a read-only operation and may allow optimizations.
- 🎭 **Proxy/self-invocation:** The call must pass through Spring's proxy for its transaction settings to apply.

#### 6.4.12 Interview answer

> “`@Transactional` is a Spring annotation that lets Spring manage a database transaction. It defines a boundary around related database operations: the changes are committed if the method succeeds, and Spring rolls them back for unchecked exceptions by default. I usually apply it at the service layer. I also consider propagation, isolation, and rollback rules when the business case requires them.”

---

### 6.5 JPA Locking: Optimistic vs Pessimistic

#### 6.5.1 What is locking?

Locking helps prevent problems when **two users change the same database record at the same time**.

Example: Two users see that a product has 10 items in stock. Both try to buy 7. Without protection, the stock could be updated incorrectly.

There are two common approaches:

- **Optimistic locking:** Let users work, then check for a conflict when saving.
- **Pessimistic locking:** Lock the database row while it is being changed, so competing updates must wait.

#### 6.5.2 Optimistic Locking

**Idea:** Assume conflicts are rare. Check whether someone else changed the record before saving.

##### How it works

1. User A and User B both read the product at version `1`.
2. User A updates it. Hibernate saves the change and increases the version to `2`.
3. User B tries to save using the old version `1`.
4. The version no longer matches, so the update fails instead of overwriting User A's change.

##### Example

Add `@Version` to the entity. Hibernate manages this field; do not change it yourself.

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int stock;

    @Version
    private int version;
}
```

**Remember:** `@Version` lets Hibernate detect if another transaction changed the same record.

When a conflict happens, the application can ask the user to refresh and try again, or safely retry the operation.

**Use it when:** Conflicts are uncommon. It is a good starting choice for most applications.

#### 6.5.3 Pessimistic Locking

**Idea:** Lock the database row before changing it. Other transactions trying to make conflicting changes must wait until the lock is released.

This is useful when several requests may update the same record at the same time, such as product stock.

##### Repository example

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdForUpdate(@Param("id") Long id);
}
```

##### Service example

```java
@Service
@RequiredArgsConstructor
public class InventoryService {

    private final ProductRepository productRepository;

    @Transactional
    public void decreaseStock(Long id, int quantity) {
        Product product = productRepository.findByIdForUpdate(id)
                .orElseThrow();

        if (product.getStock() < quantity) {
            throw new IllegalArgumentException("Not enough stock");
        }

        product.setStock(product.getStock() - quantity);
    }
}
```

The row stays locked during the transaction. When the transaction finishes, the database releases the lock.

**Remember:** `PESSIMISTIC_WRITE` is used when you need to protect a row from competing updates. Exact behavior for other reads depends on the database.

#### 6.5.4 Quick Comparison

| | Optimistic locking | Pessimistic locking |
|---|---|---|
| Main idea | Check the version when saving | Lock the row before changing it |
| Do other updates wait? | Usually no; a conflicting update fails | Yes; conflicting updates usually wait |
| Best for | Occasional conflicts | Frequent conflicts on the same record |
| Main downside | May need a retry | Can cause waiting or deadlocks |
| How to use it | Add `@Version` | Use `@Lock(LockModeType.PESSIMISTIC_WRITE)` |

#### 6.5.5 Important points

- Start with **optimistic locking** for most cases.
- Use **pessimistic locking** only where competing updates are common or must be carefully controlled.
- Keep pessimistic-lock transactions short so other requests do not wait too long.
- Pessimistic locks can contribute to a **deadlock** if transactions lock rows in different orders. Lock rows in a consistent order where possible.
- Both approaches can be used in the same application.

#### 6.5.6 Interview answer

**Optimistic locking** uses `@Version` to detect whether another transaction changed a record. If the version has changed, the update fails. **Pessimistic locking** locks the database row so competing updates have to wait. I would usually start with optimistic locking and use pessimistic locking for specific high-contention operations, such as updating inventory.

---

### 6.6 Caching in JPA and Spring Boot

#### 6.6.1 What Is Caching?

**Caching means keeping frequently used data in memory so the application can return it faster instead of querying the database every time.**

**Example:** The list of countries rarely changes. Fetch it once, cache it, and reuse it for later requests.

**Why use it?**

- Faster responses
- Fewer database queries
- Less load on the database

Cache data that is read often and does not change too frequently. Remember that cached data can become outdated, so it must be refreshed or removed when needed.

---

#### 6.6.2 Types of Cache

| Cache | Where it works | Setup needed? | Simple meaning |
|---|---|---|---|
| **First-level cache** | One Hibernate persistence context | No | Reuses entities already loaded in the current context |
| **Second-level cache** | Shared across persistence contexts | Yes | Reuses cached entities across transactions |
| **Spring cache** | At the method/service level | Enable caching and choose a cache provider | Stores the result of a method call |

#### 6.6.3 First-Level Cache

Hibernate provides this cache automatically. It works inside the current persistence context, usually during a transaction.

When an entity is loaded by its ID, Hibernate keeps track of it. Asking for the same entity by ID again in the same context usually does not run another SQL query.

```java
@Transactional
public void checkCache() {
    User first = userRepository.findById(1L).orElseThrow();  // Reads from DB
    User second = userRepository.findById(1L).orElseThrow(); // Reuses cached entity

    System.out.println(first == second); // true
}
```

**Remember:** First-level cache is automatic, but it does not generally cache the results of queries such as `findByEmail()`.

---

#### 6.6.4 Second-Level Cache

The second-level cache is optional. It can share cached entities between different persistence contexts. You must configure a supported cache provider and mark the entities you want to cache.

Use it for data that is read often and changes rarely, such as categories or reference data.

**Remember:** It needs extra setup. Do not enable it everywhere without measuring whether it helps.

> Exact setup depends on your Hibernate version and chosen cache provider. Learn the idea first; configure it only when your project needs it.

---

#### 6.6.5 Spring Boot Cache — Most Useful to Know

Spring caching can store the result of a method, such as a service method that fetches roles or categories.

##### Step 1: Add the dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

##### Step 2: Enable caching

```java
@SpringBootApplication
@EnableCaching
public class MyApplication {
}
```

##### Step 3: Use the caching annotations

| Annotation | Simple meaning | Common use |
|---|---|---|
| `@Cacheable` | Return cached data if available; otherwise run the method and cache its result | Read methods |
| `@CachePut` | Always run the method, then update the cache with its result | Save/update methods |
| `@CacheEvict` | Remove data from the cache | Delete or invalidate cached data |

##### Example: `@Cacheable`

```java
@Service
public class RoleService {

    @Cacheable("roles")
    public Role getRole(String name) {
        return roleRepository.findByName(name).orElseThrow();
    }
}
```

**How it works:**

1. First call for `ADMIN`: the method runs and the result is cached.
2. Next call for `ADMIN`: Spring returns the cached result, so the method is skipped.
3. A call for `USER` uses a different cache key and runs separately.

##### Example: Update and remove cached data

```java
@CachePut(value = "roles", key = "#role.name")
public Role updateRole(Role role) {
    return roleRepository.save(role);
}

@CacheEvict(value = "roles", key = "#name")
public void deleteRole(String name) {
    roleRepository.deleteByName(name);
}
```

**Remember:**

- `@Cacheable` — reuse a result.
- `@CachePut` — run the method and refresh the cached result.
- `@CacheEvict` — remove a cached result.
- Do not use `@Cacheable` on a method that must always save or update data, because a cache hit can skip the method.

---

#### 6.6.6 Cache Keys

A **cache key** identifies a stored result.

By default, Spring builds the key from the method arguments. Different arguments usually mean different cache entries.

```java
@Cacheable(value = "users", key = "#id")
public User getUser(Long id) {
    return userRepository.findById(id).orElseThrow();
}
```

Here, `getUser(1L)` and `getUser(2L)` use different cache entries.

Common key expressions:

| Expression | Meaning |
|---|---|
| `#id` | The `id` method argument |
| `#email` | The `email` method argument |
| `#user.id` | The ID field of the `user` argument |

Use stable values such as IDs or unique names as keys.

---

#### 6.6.7 Cache Expiry (TTL)

**TTL (Time To Live)** is how long cached data remains **valid** before it **expires**.

Cached data stays available for a **limited time**. After it expires, the application may **fetch fresh data** again.

**Example:** If a job listing is cached for **10 minutes**, then once it expires, the next request can fetch fresh data and **cache it again**.

---

##### Caffeine Setup (Simple In-Memory Cache)

Add the dependency:

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

Example configuration in `application.properties`:

```properties
spring.cache.type=caffeine
spring.cache.cache-names=jobs,roles
spring.cache.caffeine.spec=maximumSize=5000,expireAfterWrite=10m
```

- **`maximumSize=5000`**: limits the approximate **number of entries** kept in the cache.
- **`expireAfterWrite=10m`**: entries **expire 10 minutes** after they are written or refreshed.

---

##### Simple Example

Suppose you cache **user details for 5 minutes**:

1. **First request** → Data is fetched from the **database** and stored in the **cache**.
2. **Next request within 5 minutes** → Data comes from the **cache**.
3. **After 5 minutes** → The cached entry **expires**. The next request fetches **fresh data** from the database and caches it again.

> **Remember:** TTL controls **how long cached data remains valid**. It helps prevent your application from using **stale data** for too long.


---

### 6.6.8 Cache-Aside Pattern

**Cache-aside** means: first check the **cache**. If the data is **not there**, fetch it from the **database** and put it in the cache.

---

### Analogy: Looking for a Book 📚

Imagine you need a book:

1. First, check your **desk**.
2. If the book is there, **use it**.
3. If it's not there, get it from the **library** and keep it on your desk for **next time**.

Your **desk** is the **cache**, and the **library** is the **database**.

---

### How It Works

1. **User requests data**
2. **Check the cache**
   - **Cache hit** → Return data from the cache
   - **Cache miss** → Fetch from the **database**, then **store it in the cache**

---

### Simple Example

Suppose you request user `1`:

- **First request:** The cache is **empty** (cache miss), so the application fetches the user from the **database** and stores the result in the **cache**.
- **Second request:** The user is **already cached** (cache hit), so the application returns the **cached data** without querying the database again.

---

> **Remember:** Cache-aside = **Check cache** → If missing, **get data from database** → **Store it in cache**.
>
> The **application manages** this process itself. When database data changes, the application should also **update or invalidate** the relevant cached entry to avoid serving **stale data**.


### 6.6.9 Cache Invalidation
Cache invalidation means **removing or updating old data** in the cache when the original data changes.

### 6.6.10 Cache Consistency
Cache consistency means **keeping the cache and database in sync** so that users don't get outdated data.


#### 6.6.11 Quick Revision

- **Caching:** Keeps frequently used data ready in memory to reduce repeated work.
- **First-level cache:** Automatic; works within one persistence context.
- **Second-level cache:** Optional; can share cached entities across contexts and needs setup.
- **`@Cacheable`:** Use a cached result when available.
- **`@CachePut`:** Run the method and update the cache.
- **`@CacheEvict`:** Remove data from the cache.
- **TTL:** How long a cached entry remains valid.
- **Caffeine:** A common in-memory cache provider for a single application instance.

##### Interview Answer

“Caching stores frequently used data so the application does not have to fetch it from the database every time. Hibernate has a first-level cache by default and an optional second-level cache. Spring Boot also provides `@Cacheable`, `@CachePut`, and `@CacheEvict` to cache method results, update cached data, and remove outdated entries.”

---

## 7. Quick Revision Cheat Sheet

### 7.1 The Stack

```text
Your Code → Spring Data JPA → JPA (spec) → Hibernate (engine) → JDBC → Database
```

- **JPA** = specification (the menu) — defines rules, can't run alone
- **Hibernate** = implementation (the kitchen) — generates & executes SQL
- **Spring Data JPA** = convenience layer (the waiter) — auto-generates repositories, eliminates boilerplate

---

### 7.2 Entity Essentials

| Annotation | Purpose |
|---|---|
| `@Entity` | Marks class as a DB table (required) |
| `@Table(name = "users")` | Custom table name — use lowercase, plural, snake_case |
| `@Id` | Primary key (every entity must have one) |
| `@GeneratedValue` | Auto-generate PK — see strategy table below |
| `@Column(nullable, unique, length, columnDefinition)` | Customize column mapping |
| `@Enumerated(EnumType.STRING)` | Store enum as text — **never use ORDINAL** |
| `@Transient` | Field not mapped to DB |
| `@Version` | Optimistic locking version field |
| `@DynamicUpdate` | UPDATE only changed columns (not all) |
| `@NoArgsConstructor` | **Required** by JPA — empty constructor for reflection |

**@GeneratedValue Strategies**

| Strategy | Best For | Notes |
|---|---|---|
| `IDENTITY` | MySQL — simplest | Disables Hibernate batch inserts |
| `SEQUENCE` | PostgreSQL/Oracle — best perf | Use `allocationSize` for bulk inserts |
| `UUID` | Microservices / distributed | No ID collisions; larger index size |
| `TABLE` | Legacy only | Slowest — avoid in new projects |

---

### 7.3 Relationships

| Annotation | Default Fetch | Your Action | Owner Side? |
|---|---|---|---|
| `@OneToOne` | EAGER ⚠️ | **Override → LAZY** | Side with `@JoinColumn` |
| `@ManyToOne` | EAGER ⚠️ | **Override → LAZY** | **Always the owner** (has FK) |
| `@OneToMany` | LAZY ✅ | Keep LAZY, use `mappedBy` | Never — use `mappedBy` |
| `@ManyToMany` | LAZY ✅ | Keep LAZY, use `Set` not `List` | Side with `@JoinTable` |

> **Rule:** `@ManyToOne` side is always the owner (FK lives in the "many" table).

**Key terms:**

- **Owner side** → has `@JoinColumn`, holds the FK column
- **Inverse side** → uses `mappedBy`, no FK created
- Always sync both sides in bidirectional relationships

---

### 7.4 Cascade Types

| CascadeType | Effect | Use When |
|---|---|---|
| `PERSIST` | Save parent → saves children | Parent creates children (Order → Items) |
| `MERGE` | Update parent → updates children | Parent modifies children |
| `REMOVE` | Delete parent → deletes children | Children can't exist without parent |
| `ALL` | All of the above | Parent **fully owns** children |

- `orphanRemoval = true` → removing child from collection deletes it from DB
- **@ManyToMany** → only use `PERSIST` + `MERGE`, **never** `REMOVE` or `ALL`

---

### 7.5 DDL-Auto Strategies

| Value | Behavior | Environment |
|---|---|---|
| `create` | Drop & recreate all tables — **data lost** | Local dev (throwaway) |
| `create-drop` | Create on start, drop on shutdown | Automated tests |
| `update` | Add new columns/tables, never drops | Development |
| `validate` | Check schema match — crash if mismatch | Staging / pre-prod |
| `none` | Do nothing — you manage schema | **Production** (use Flyway/Liquibase) |

---

### 7.6 Repositories

**Hierarchy:** `CrudRepository` → `PagingAndSortingRepository` → `JpaRepository` (use this one)

```java
public interface UserRepository extends JpaRepository<User, Long> { }
```

**Built-in methods (free):**

| Operation | Methods |
|---|---|
| Create/Update | `save()`, `saveAll()`, `saveAndFlush()` |
| Read | `findById()` → `Optional<T>`, `findAll()`, `existsById()`, `count()` |
| Delete | `deleteById()`, `delete()`, `deleteAllInBatch()` (faster bulk) |
| Page/Sort | `findAll(Pageable)`, `findAll(Sort)` |

- `save()` = INSERT if ID is null, UPDATE if ID exists
- `findById()` returns `Optional` — use `.orElseThrow()`

---

### 7.7 Derived Query Keywords (Top 15)

| Keyword | SQL | Example |
|---|---|---|
| `And` / `Or` | `AND` / `OR` | `findByNameAndStatus` |
| `Is` / `Equals` | `=` | `findByStatusIs` |
| `Not` | `!=` | `findByStatusNot` |
| `Between` | `BETWEEN` | `findByAgeBetween` |
| `LessThan` / `GreaterThan` | `<` / `>` | `findByAgeLessThan` |
| `Before` / `After` | `<` / `>` (dates) | `findByCreatedAtAfter` |
| `IsNull` / `IsNotNull` | `IS NULL` / `IS NOT NULL` | `findByEmailIsNull` |
| `Containing` | `LIKE %x%` | `findByNameContaining` |
| `StartingWith` / `EndingWith` | `LIKE x%` / `LIKE %x` | `findByNameStartingWith` |
| `In` | `IN (...)` | `findByStatusIn(List)` |
| `True` / `False` | `= true` / `= false` | `findByVerifiedTrue` |
| `IgnoreCase` | `LOWER()` | `findByEmailIgnoreCase` |
| `OrderBy` | `ORDER BY` | `findByStatusOrderByNameAsc` |
| `Top` / `First` | `LIMIT` | `findTop5ByStatus` |

> If method name > 3–4 keywords long → switch to `@Query` with JPQL.

---

### 7.8 Query Resolution Priority

When Spring finds a repository method, it checks in this order:

1. **@NamedQuery** (`User.methodName` on entity) → oldest, rarely used now
2. **@Query annotation** (JPQL or native SQL on the method) → most flexible
3. **Derived query** (parsed from method name) → simplest, auto-generated

**JPQL vs Native SQL:**

| | JPQL | Native SQL |
|---|---|---|
| References | Java class/field names | Table/column names |
| DB-independent? | ✅ Yes | ❌ No |
| Validated at startup? | ✅ Yes | ❌ No (runtime only) |
| Supports window functions, CTEs? | ❌ No | ✅ Yes |

- `@Modifying` + `@Transactional` required for UPDATE/DELETE `@Query`
- Add `@Modifying(clearAutomatically = true)` to avoid stale persistence context

**Projections** — fetch only needed columns:

- **Interface-based** (recommended): define interface with getters → Spring auto-implements
- **DTO-based**: JPQL `new` constructor expression
- **Dynamic**: pass projection type as parameter

---

### 7.9 Pagination & Sorting

```java
PageRequest.of(page, size, Sort.by("field").descending())  // page is 0-indexed!
```

| Concept | Purpose |
|---|---|
| `Pageable` / `PageRequest` | Holds page number, size, sort |
| `Page<T>` | Data + metadata (totalPages, totalElements) — runs COUNT query |
| `Slice<T>` | Data + hasNext — **no COUNT** (faster for infinite scroll) |
| `Sort` | `Sort.by("field").ascending().and(Sort.by("other"))` |

> Add `Pageable` to **any** repository method — built-in, derived, or `@Query`.

---

### 7.10 N+1 Problem — Solutions

**Problem:** 1 query loads N entities + N extra queries for each entity's lazy relationship.

| Solution | Queries | How | Best For |
|---|---|---|---|
| `JOIN FETCH` (JPQL) | **1** | `SELECT e FROM Employee e JOIN FETCH e.department` | Default go-to |
| `@EntityGraph` | **1** | `@EntityGraph(attributePaths = {"department"})` on method | Keep derived methods clean |
| `@BatchSize(size=N)` | N/batch + 1 | Loads children in `IN (...)` batches | Multiple collections |
| `@Fetch(SUBSELECT)` | **2** | Loads all children via subselect | Always loading all parents |

> **Detection:** Enable `spring.jpa.show-sql=true` — if same SELECT repeats dozens of times, it's N+1.

---

### 7.11 Entity Lifecycle States

```text
new Object()          save()/persist()          transaction ends
     │                      │                         │
 TRANSIENT ──────────→ MANAGED ──────────────→ DETACHED
                         │    ↑                       │
                  delete()/   save()/merge()    save() inside
                  remove()    (re-attach)       new @Transactional
                         ↓                            │
                      REMOVED                    MANAGED again
```

| State | JPA Watching? | Changes Auto-Saved? | How You Get Here |
|---|---|---|---|
| **Transient** | ❌ No | ❌ No | `new Entity()` |
| **Managed** | ✅ Yes | ✅ Yes (dirty checking) | After `save()`, `findById()` |
| **Detached** | ❌ No | ❌ No | Transaction ends, `detach()`, `clear()` |
| **Removed** | ✅ Scheduled | DELETE on commit | After `delete()` / `remove()` |

---

### 7.12 Dirty Checking

- Hibernate **snapshots** every managed entity when loaded
- At commit, compares current state vs snapshot → auto-generates UPDATE for changed fields
- **No `save()` needed** inside `@Transactional` for managed entities
- Does **NOT** work on detached or transient entities
- Use `@DynamicUpdate` to UPDATE only changed columns (not all)

---

### 7.13 @Transactional

| Attribute | Default | Purpose |
|---|---|---|
| `propagation` | `REQUIRED` | Join existing tx, or start new |
| `readOnly` | `false` | Set `true` for reads → skips dirty checking |
| `rollbackFor` | `RuntimeException` only | Add `Exception.class` for checked exceptions |
| `timeout` | none | Max seconds before rollback |

**Propagation quick reference:**

| Type | Behavior |
|---|---|
| `REQUIRED` | Join existing tx or create new (99% of cases) |
| `REQUIRES_NEW` | Always create new tx; outer tx pauses |
| `MANDATORY` | Must be called within existing tx, else exception |
| `NEVER` | Must NOT be within a tx, else exception |

**Critical rules:**

- ✅ Only on **public** methods (proxy can't intercept private)
- ✅ Place on **service layer** (not controllers or repositories)
- ❌ **Self-invocation** bypasses proxy → `@Transactional` ignored (fix: inject `self`)
- ✅ Use `readOnly = true` on all read-only methods

---

### 7.14 Locking

**Optimistic (recommended for most apps):**

- Add `@Version` field → Hibernate checks version on every UPDATE
- Conflict → `OptimisticLockException` → retry or return 409
- No DB locks, no blocking, no deadlocks
- Best when: conflicts are **rare** (read-heavy apps)

**Pessimistic (high-contention scenarios):**

- `@Lock(LockModeType.PESSIMISTIC_WRITE)` → `SELECT ... FOR UPDATE`
- Row is locked — other transactions **wait**
- Risk: deadlocks (always lock rows in consistent order)
- Best when: concurrent writes to **same row** are frequent (inventory, payments)

---

### 7.15 Caching

**First-Level Cache (always on):**

- Per-transaction, per-session — automatic
- Same `findById()` twice in one `@Transactional` → 1 SQL query
- Cleared when transaction ends

**Second-Level Cache (L2 — needs config):**

- Shared across transactions, application-wide
- Add `@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)` on entity
- Requires: dependency (Caffeine/EhCache) + `hibernate.cache.use_second_level_cache=true`

| Strategy | Staleness | Use For |
|---|---|---|
| `READ_ONLY` | None (never changes) | Countries, currencies |
| `NONSTRICT_READ_WRITE` | Brief staleness OK | Categories, preferences |
| `READ_WRITE` | Consistent | User profiles, products |

**Spring Boot Caching (`@Cacheable`):**

- Works on **any Spring Bean method**, not just DB queries
- Requires `@EnableCaching` on config class

| Annotation | Purpose |
|---|---|
| `@Cacheable("name")` | Return from cache if present; else execute & cache |
| `@CachePut("name")` | Always execute, then update cache |
| `@CacheEvict("name")` | Remove entry from cache |

- **Never** use `@Cacheable` on write methods (would skip the actual save!)
- Use Caffeine for TTL: `expireAfterWrite` (freshness) vs `expireAfterAccess` (popularity)

---

### 7.16 Common Gotchas

| Gotcha | Fix |
|---|---|
| `LazyInitializationException` | Load data inside `@Transactional`, or use `JOIN FETCH` |
| `@Transactional` on private method | Make it `public` — proxy can't intercept private |
| Self-invocation skips `@Transactional` | Inject `self` via `@Autowired` and call through proxy |
| `@Enumerated` default is ORDINAL | Always use `@Enumerated(EnumType.STRING)` |
| `findById()` returns `Optional` | Use `.orElseThrow()`, not direct assignment |
| Editing detached entity — changes lost | Re-attach via `save()` inside new `@Transactional` |
| `@Modifying` query → stale context | Add `clearAutomatically = true` |
| `CascadeType.ALL` on `@ManyToMany` | Use only `PERSIST` + `MERGE` — never `REMOVE` |
| N+1 silently slow | Enable `show-sql=true`, count repeated queries |
| `ddl-auto=update` in production | Use `none` + Flyway/Liquibase for migrations |
| Native query typo not caught at startup | Test all native queries — only validated at runtime |
| Page numbers are 0-indexed | `PageRequest.of(0, 10)` = first page |

---

## 8. Interview Questions & Answers

### 8.1 Fundamentals

#### Q1: What is the difference between JPA, Hibernate, and Spring Data JPA?

**A:** JPA is a **specification** (a set of interfaces and rules defined by Jakarta EE). Hibernate is an **implementation** of that specification — it's the actual code that translates Java operations to SQL. Spring Data JPA is an **abstraction layer on top of JPA** that eliminates boilerplate code by auto-generating repository implementations, supporting method-name-based query generation, and providing built-in pagination. The relationship is:  
`Spring Data JPA → JPA (interfaces) → Hibernate (implementation) → JDBC → Database`

#### Q2: What is an EntityManager and what is its role?

**A:** The `EntityManager` is the core JPA interface for interacting with the **persistence context** (first-level cache). It manages entity lifecycle operations: `persist()` (insert), `merge()` (update detached), `remove()` (delete), `find()` (select by PK), and `createQuery()` (JPQL queries). Spring Data JPA hides this behind repositories, but it's used internally and can be injected with `@PersistenceContext` for custom repository implementations.

#### Q3: What is the persistence context?

**A:** The persistence context is a **cache that holds managed entities** during a unit of work (usually one transaction). It's Hibernate's first-level cache — every entity loaded or saved within a transaction is stored here. This enables dirty checking (auto-detecting and flushing changes) and ensures identity guarantee (same DB row = same Java object instance within one session). The persistence context is flushed (SQL sent to DB) before commits or explicit `flush()` calls.

---

### 8.2 Entity Mapping

#### Q4: What is the difference between `@JoinColumn` and `mappedBy`?

**A:** `@JoinColumn` is placed on the **owner side** of the relationship (the entity whose table contains the foreign key column). `mappedBy` is placed on the **inverse/non-owner side** and tells JPA "don't create a foreign key here — the relationship is owned by the field named X in the other entity." If you forget `mappedBy`, JPA creates a join table or extra columns.

#### Q5: Why should you use `@Enumerated(EnumType.STRING)` instead of the default `ORDINAL`?

**A:** `ORDINAL` stores the enum's position (0, 1, 2...). If you reorder enum constants or add new ones in the middle, all stored data becomes invalid/corrupted. `STRING` stores the enum name ("ACTIVE", "INACTIVE"), which is stable — reordering or adding constants doesn't affect stored data. Always use `EnumType.STRING`.

#### Q6: What is the difference between `CascadeType.ALL` and `orphanRemoval = true`?

**A:** `CascadeType.ALL` means all JPA operations (persist, merge, remove, refresh, detach) on the parent are cascaded to children. `orphanRemoval = true` specifically handles the case where you **remove a child from the parent's collection** — the child is automatically deleted from the database. `CascadeType.REMOVE` only deletes children when the parent itself is deleted. `orphanRemoval` handles collection removals. Both are often used together on `@OneToMany` relationships.

---

### 8.3 N+1 & Performance

#### Q7: What is the N+1 problem? How do you detect and fix it?

**A:** The N+1 problem occurs when fetching N entities triggers N additional queries to load their related entities (e.g., fetching 100 orders then 100 separate queries for each order's customer = 101 queries total).

**Detection:** Enable `show-sql=true` and count queries, or use `datasource-proxy` / `p6spy` in tests. Hibernate also logs warnings.

**Fixes:**

1. **JOIN FETCH** in JPQL: `SELECT e FROM Employee e JOIN FETCH e.department`
2. **@EntityGraph**: `@EntityGraph(attributePaths = {"department"})`
3. **@BatchSize**: `@BatchSize(size = 30)` on the collection — loads in batches
4. **FetchMode.SUBSELECT**: Loads all children in one subquery

#### Q8: When would you use `PESSIMISTIC_WRITE` vs `@Version` (optimistic locking)?

**A:** Use **optimistic locking (`@Version`)** when conflicts are rare — it allows concurrent reads, and only fails at commit if a concurrent update happened. It's lighter weight and scales better. Use **`PESSIMISTIC_WRITE`** when conflicts are frequent or the cost of retrying is high — it locks the row immediately (other transactions wait or fail). For example, inventory reduction in flash sales should use pessimistic locking to prevent overselling; a user profile update can use optimistic locking.

---

### 8.4 Transactions

#### Q9: `@Transactional` on a private method — does it work?

**A:** No. Spring's transaction management uses **AOP proxies**. The proxy wraps the bean and intercepts method calls to apply transactional behavior. Private methods are not visible to the proxy (and are not overridable), so `@Transactional` is completely ignored. The fix: make the method `public`, or refactor the logic into a separate Spring bean where the proxy can intercept it.

#### Q10: What is the difference between transaction propagation `REQUIRED` and `REQUIRES_NEW`?

**A:**

- `REQUIRED` (default): If a transaction already exists, join it. If not, create a new one. All operations share one transaction — one rollback affects everything.
- `REQUIRES_NEW`: Always start a brand new transaction, suspending any existing one. The new transaction has its own commit/rollback lifecycle. Use case: **audit logging** — you want the audit record saved even if the main business transaction rolls back.

#### Q11: What is dirty checking?

**A:** Hibernate takes a **snapshot** of managed entities when they're loaded. At flush time (before commit), Hibernate compares the current state with the snapshot. If anything changed, it automatically generates and executes the `UPDATE` SQL — without you calling `save()`. This only works for **managed entities** (inside a `@Transactional` method). After the transaction ends, entities are detached and changes are no longer tracked.

---

### 8.5 Queries

#### Q12: What's the difference between JPQL and native SQL queries in Spring Data JPA?

**A:**

| | JPQL | Native SQL |
|---|---|---|
| Syntax | Entity/field names | Table/column names |
| Portability | Works on any DB | DB-specific |
| JPA features | Full (caching, 2nd level) | Limited |
| Complex queries | Limited | Full SQL power |
| Use for | Standard queries | Window functions, CTEs, specific DB features |

#### Q13: How do projections improve performance?

**A:** When you return a full entity, Hibernate loads all columns from the table. If you only need 3 out of 30 columns, you're wasting 90% of the data transfer. **Interface projections** and **DTO projections** let you define exactly which fields to fetch. Spring Data JPA generates `SELECT id, first_name, email FROM users` instead of `SELECT *`. This reduces: data transfer, memory usage, and deserialization time. For read-heavy endpoints, this can be a 3-10x performance improvement.

#### Q14: What is the difference between `Page` and `Slice` in Spring Data JPA?

**A:**

- `Page<T>`: Executes TWO queries — one for the data and one for `COUNT(*)` to know the total records and pages. More expensive but gives you `totalElements` and `totalPages`.
- `Slice<T>`: Executes ONE query — just fetches page+1 records to know if there's a next page. No total count. Use `Slice` for infinite scroll UIs where you don't need total count. Use `Page` for traditional pagination with page numbers.

---

### 8.6 Advanced

#### Q15: Explain the first-level and second-level cache in Hibernate.

**A:**

- **First-level cache** (Session cache): Always on, scoped to a single `Session`/transaction. Every entity you load or save is stored here. Within the same transaction, fetching the same entity twice returns the same object without an extra SQL query. Cleared when the session closes.
- **Second-level cache**: Shared across sessions/transactions. Stores entity data by ID. Needs explicit configuration (Ehcache, Caffeine, Redis). Annotate entities with `@Cache`. Great for frequently read, rarely changed data (countries, categories, etc.). Query cache is an extension that caches query results.

#### Q16: How do you handle `LazyInitializationException`?

**A:** This happens when you access a lazy-loaded relationship outside an open transaction (the session is closed). Solutions:

1. **Best:** Load the data within the transaction using `JOIN FETCH` or `@EntityGraph`
2. **Use DTO projection**: Don't pass entities outside the service layer
3. **`@Transactional` on service method**: Keeps session open during the call
4. **`spring.jpa.open-in-view=false`** (disable OSIV): Forces you to solve the problem properly at the design level
5. **Avoid:** `FetchType.EAGER` — solves LazyInitException but creates N+1

#### Q17: How does the `Specification` pattern help with dynamic queries?

**A:** When you have multiple optional filters (name, status, date range, department), combining them with method-name queries leads to combinatorial explosion (2^n methods). The `Specification` pattern wraps each filter as a predicate and lets you compose them with `.and()`, `.or()`, `.not()`. You extend `JpaSpecificationExecutor` in the repository and build `Specification<T>` objects in the service. Only non-null predicates are applied, giving you clean, dynamic, composable SQL without if-else chains.

#### Q18: What is `@MappedSuperclass` and when do you use it?

**A:** `@MappedSuperclass` marks a class whose **mapped fields are inherited by entity subclasses** — but the class itself is not an entity and has no table. Used for sharing common fields across multiple entities (e.g., `id`, `createdAt`, `updatedAt`, `createdBy`). Unlike `@Inheritance`, there's no hierarchy table — each entity gets its own table with the inherited columns. The base auditing class pattern uses `@MappedSuperclass` with `@EntityListeners(AuditingEntityListener.class)`.

#### Q19: What are the `GenerationType` strategies and which should you use?

**A:**

- `IDENTITY`: DB auto-increment. Simple, works everywhere. **Disables Hibernate batch inserts** (DB generates ID after insert, so Hibernate can't batch them). Use for simple apps.
- `SEQUENCE`: DB sequence object. Hibernate pre-allocates IDs in batches (`allocationSize`). **Best for performance and batch inserts**. Best for PostgreSQL/Oracle.
- `UUID`: Generates random UUID. Good for microservices and distributed systems. Slightly larger index size.
- `TABLE`: Uses a separate table for ID tracking. Slowest, avoid in production.

**Recommendation:** `SEQUENCE` for PostgreSQL apps needing performance; `IDENTITY` for simple MySQL apps; `UUID` for distributed systems.

#### Q20: How do you implement soft deletes with Spring Data JPA?

**A:** Instead of physically deleting records, add a `deletedAt` or `deleted` flag:

```java
@Entity
@Where(clause = "deleted = false")          // Hibernate filter — always adds WHERE deleted=false
@SQLDelete(sql = "UPDATE users SET deleted = true WHERE id = ?") // Override DELETE SQL
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private boolean deleted = false;
    private LocalDateTime deletedAt;
}

// Now repository.delete(user) executes UPDATE instead of DELETE
// And all findBy methods automatically exclude deleted records
```

---