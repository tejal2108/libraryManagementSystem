# Library Management System

A Java-based REST API built with Spring Boot and MySQL to manage library operations, including authors, books, students, library cards, and book issue/return transactions.

## Features

- CRUD operations for authors, books, and students.
- Search and filter books by genre, availability, and author.
- Automatic library card creation during student registration.
- Book issue and return workflows with availability tracking.
- Overdue fine calculation based on loan duration.
- Database persistence using Spring Data JPA and Hibernate.
- Request handling and validation for library operations.

## Technology Stack

- **Language:** Java 11
- **Framework:** Spring Boot 2.4.3
- **Backend:** Spring Web, Spring Data JPA
- **Database:** MySQL
- **ORM:** Hibernate
- **Build Tool:** Maven
- **Testing Dependencies:** JUnit 5, Spring Boot Test

## Project Structure

```text
src/main/java/com/StudentLibrary/Studentlibrary/
├── Controllers/
├── DTO/
├── Model/
├── Repositories/
├── Services/
└── StudentLibraryApplication.java

src/main/resources/
└── application.properties

src/test/java/
```

## Prerequisites

- JDK 11
- MySQL Server
- Maven 3.6+ (optional if using the included Maven Wrapper)

## Setup and Installation

### 1. Clone the Repository

```bash
git clone https://github.com/tejal2108/libraryManagementSystem.git
cd libraryManagementSystem
```

### 2. Create the Database

Open MySQL and execute:

```sql
CREATE DATABASE library_management;
```

### 3. Configure Database Connection

Update `src/main/resources/application.properties` with your MySQL credentials.

Example:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/library_management
spring.datasource.username=${DB_USERNAME:root}
spring.datasource.password=${DB_PASSWORD:}
spring.jpa.hibernate.ddl-auto=update
```

Set the appropriate database username and password for your environment. Do not commit real credentials to GitHub.

### 4. Run the Application

**Windows**

```bat
mvnw.cmd spring-boot:run
```

**macOS/Linux**

```bash
./mvnw spring-boot:run
```

Alternatively, use an installed Maven version:

```bash
mvn spring-boot:run
```

The application runs at `http://localhost:8080` by default.

## API Endpoints

### Authors

| Method | Endpoint | Description |
|---|---|---|
| POST | `/createAuthor` | Create an author |
| PUT | `/updateAuthor/{id}` | Update author details |
| DELETE | `/deleteAuthor?id={id}` | Delete an author |

### Books

| Method | Endpoint | Description |
|---|---|---|
| POST | `/createBook` | Create a book |
| GET | `/getBooks` | Retrieve and filter books |
| GET | `/getBook/{id}` | Retrieve a book by ID |
| PUT | `/updateBook/{id}` | Update book details |
| DELETE | `/deleteBook/{id}` | Delete a book |

Book listings support filtering by genre, availability, and author name where configured.

### Students and Library Cards

| Method | Endpoint | Description |
|---|---|---|
| POST | `/createStudent` | Register a student and create a library card |
| PUT | `/updateStudent/{id}` | Update student details |
| DELETE | `/deleteStudent?id={id}` | Delete a student |

### Book Issue and Return

| Method | Endpoint | Description |
|---|---|---|
| POST | `/issueBook?cardId={cardId}&bookId={bookId}` | Issue a book |
| POST | `/returnBook?cardId={cardId}&bookId={bookId}` | Return a book and calculate any applicable fine |

## Example Request

### Create a Book

**POST** `/createBook`

```json
{
  "name": "Clean Code",
  "genre": "NON_FICTIONAL",
  "authorId": 1,
  "availableCopies": 5
}
```

The referenced author must already exist in the database.

## Business Rules

- Limits the number of books that can be issued to a library card.
- Tracks book availability during issue and return operations.
- Calculates overdue fines based on the configured loan duration and daily fine.
- Uses database entities and repositories to manage persistent records.

## API Testing

Endpoints can be tested using Postman. The project also includes Spring Boot testing dependencies for automated testing.
