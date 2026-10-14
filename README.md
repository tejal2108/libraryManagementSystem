# Library Management System

A Java and Spring Boot REST API for managing library authors, books, students, library cards, and book issue/return transactions. The application uses Spring Data JPA/Hibernate to persist data in MySQL.


## Features

- **Author management:** create, update, and delete author records.
- **Book management:** create, view one or all books, update, and delete books.
- **Book search/filtering:** filter the book list by genre, availability, and author name.
- **Flexible book request:** book creation/update accepts either `authorId` or `author: { "id": ... }`; the book name can be supplied as `name` or `title`.
- **Student management:** register, update, and delete students.
- **Library cards:** student registration creates a library card; cards have activation status.
- **Issue and return workflow:** issue books to a library card, track transactions, and adjust available-copy counts.
- **Overdue fines:** return processing calculates fines using the configured loan duration and per-day fine.
- **API documentation support:** Springdoc OpenAPI UI dependency is included.

## Technology stack

- Java 11
- Spring Boot 2.4.3
- Spring Web
- Spring Data JPA / Hibernate
- MySQL
- Maven (Maven Wrapper included)
- Springdoc OpenAPI UI 1.7.0
- Spring Boot Test / JUnit 5 dependency

## Project structure

```text
src/main/java/com/StudentLibrary/Studentlibrary/
├── Controllers/    # HTTP endpoints
├── DTO/            # Request DTOs, including BookRequest
├── Model/          # JPA entities and enums
├── Repositories/   # Spring Data JPA repositories and queries
├── Services/       # Business logic
└── StudentLibraryApplication.java

src/main/resources/
└── application.properties

src/test/java/      # Spring application context-load test
```

## Prerequisites

- JDK 11
- MySQL Server
- Maven 3.6+ (optional if you use the included Maven Wrapper)

## Configure the database

1. Create a local MySQL database:

   ```sql
   CREATE DATABASE library_management;
   ```

2. Configure the database connection. The supplied `application.properties` uses environment-variable overrides and defaults to a local MySQL instance:

   ```properties
   spring.datasource.url=${DB_URL:jdbc:mysql://localhost:3306/library_management}
   spring.datasource.username=${DB_USERNAME:root}
   spring.datasource.password=${DB_PASSWORD:}
   ```

   Set `DB_URL`, `DB_USERNAME`, and `DB_PASSWORD` in your environment if your local MySQL setup differs. Do not commit real credentials.

3. Hibernate is configured with `spring.jpa.hibernate.ddl-auto=update` for local development. It creates/updates schema based on the JPA entities; use a suitable migration strategy and database configuration for production.

### Issue/return configuration

These properties are read by the transaction service:

| Property | Default | Meaning |
|---|---:|---|
| `books.max_allowed` | `3` | Maximum books allowed per card |
| `books.max_allowed_days` | `14` | Allowed loan duration in days |
| `books.fine.per_day` | `5` | Fine per overdue day |

## Run the application

Open a terminal in the directory containing `pom.xml` and run one of the following commands.

**Windows**

```bat
mvnw.cmd spring-boot:run
```

**macOS/Linux**

```bash
./mvnw spring-boot:run
```

Or use an installed Maven version:

```bash
mvn spring-boot:run
```

The default server port is `8080` unless changed in configuration.

Verify that the application responds:

```http
GET http://localhost:8080/test
```

Expected response:

```text
Test route is working!
```

## API endpoints

Unless noted otherwise, endpoints accept JSON request bodies. The examples assume the application is running at `http://localhost:8080`.

### Authors

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/createAuthor` | Create an author |
| `PUT` | `/updateAuthor/{id}` | Update an author |
| `DELETE` | `/deleteAuthor?id={id}` | Delete an author by ID |

Example `POST /createAuthor` body (fields correspond to the `Author` model):

```json
{
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "age": 45,
  "country": "India"
}
```

### Books

The controller exposes both the original endpoint names and REST-style `/books` aliases.

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/createBook` | Create a book |
| `GET` | `/getBooks` | List books, optionally filtered |
| `GET` | `/getBook/{id}` | Get a book by ID |
| `PUT` | `/updateBook/{id}`  | Update a book |
| `DELETE` | `/deleteBook/{id}`  | Delete a book |

Example `POST /books` body:

```json
{
  "name": "Clean Code",
  "genre": "NON_FICTIONAL",
  "authorId": 1,
  "availableCopies": 5
}
```

The author must already exist. The alternative author format is also accepted:

```json
{
  "title": "Clean Code",
  "genre": "NON_FICTIONAL",
  "author": { "id": 1 },
  "availableCopies": 5
}
```

Optional query parameters for `GET /getBooks` or `GET /books`:

- `genre` — genre enum value
- `available` — `true` or `false`
- `author` — author name

Example:

```http
GET http://localhost:8080/books?genre=HISTORY&available=true
```

Available genre values in the current enum: `FICTIONAL`, `NON_FICTIONAL`, `GEOGRAPHY`, `HISTORY`, `POLITICAL_SCIENCE`, `BOTANY`, `CHEMISTRY`, `MATHEMATICS`, and `PHYSICS`.

### Students and library cards

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/test` | Check that the application is responding |
| `POST` | `/createStudent` | Register a student; service also creates a card |
| `PUT` | `/updateStudent/{id}` | Update student details |
| `DELETE` | `/deleteStudent?id={id}` | Delete a student by ID |
| `POST` | `/reset-student-table` | Reset the student table and auto-increment counter |

Example `POST /createStudent` body (use fields defined by the `Student` model):

```json
{
  "emailId": "student@example.com",
  "name": "Alex Kumar",
  "age": 21,
  "country": "India"
}
```

**Warning:** `/reset-student-table` is a destructive development endpoint. It executes a table reset operation and can remove student data; foreign-key constraints may also prevent it from running. Do not expose or call it against data you need to preserve.

### Book issue and return

| Method | Endpoint | Parameters | Purpose |
|---|---|---|---|
| `POST` | `/issueBook?cardId={cardId}&bookId={bookId}` | `cardId`, `bookId` | Issue a book to a card |
| `POST` | `/returnBook?cardId={cardId}&bookId={bookId}` | `cardId`, `bookId` | Return a book and calculate any overdue fine |

Example requests:

```http
POST http://localhost:8080/issueBook?cardId=1&bookId=1
POST http://localhost:8080/returnBook?cardId=1&bookId=1
```

Use IDs that exist in your local database. Issue/return requests can be rejected when business rules are not satisfied, such as card/book validity, copy availability, or transaction state.

