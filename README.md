# TestLang++ - Backend API Testing DSL

A Domain-Specific Language (DSL) for HTTP API testing that compiles `.test` files into executable JUnit 5 tests using Java's HttpClient.

## 🎯 Overview

TestLang++ allows you to write declarative HTTP API tests that are compiled into Java JUnit 5 test classes. Tests are executed against a local Spring Boot backend.

## 📁 Project Structure

```
.
├── ast/                    # Abstract Syntax Tree node classes
├── backend/                # Spring Boot backend (test target)
├── codegen/                # Code generation (AST → JUnit)
├── compiler/               # Main compiler entry point
├── examples/               # Sample .test files
├── lib/                    # External dependencies (JFlex, CUP, JUnit)
├── output/                 # Generated Java test files
├── parser/                 # CUP parser specification
├── scanner/                # JFlex lexer specification
└── scripts/                # Build and run scripts
```

## 🚀 Quick Start

### 1. Setup Dependencies

Download JFlex and CUP (Java parser generators):

```bash
./scripts/setup-deps.sh
```

### 2. Compile the Compiler

Build the scanner, parser, and code generator:

```bash
./scripts/compile.sh
```

### 3. Write Your Tests

Create a `.test` file (see examples below):

```testlang
config {
  base_url = "http://localhost:8080";
  header "Content-Type" = "application/json";
}

let user = "admin";

test Login {
  POST "/api/login" {
    body = "{ \"username\": \"$user\", \"password\": \"1234\" }";
  }
  expect status = 200;
  expect body contains "\"token\":";
}
```

### 4. Generate Tests

Compile your `.test` file to Java:

```bash
./scripts/run-compiler.sh examples/example.test output/GeneratedTests.java
```

### 5. Run Backend

Start the Spring Boot backend (in a separate terminal):

```bash
./scripts/start-backend.sh
```

The server will start at `http://localhost:8080`

### 6. Run Tests

Execute the generated JUnit tests:

```bash
./scripts/run-tests.sh output/GeneratedTests.java
```

## 📝 Language Syntax

### Config Block (Optional)

```testlang
config {
  base_url = "http://localhost:8080";
  header "Content-Type" = "application/json";
  header "X-App" = "TestLangDemo";
}
```

### Variables

```testlang
let user = "admin";
let id = 42;
```

Variables can be referenced in strings and paths using `$variableName`.

### Test Blocks

Each test becomes a `@Test` method in JUnit:

```testlang
test TestName {
  // HTTP requests
  // Assertions
}
```

### HTTP Requests

**GET/DELETE** (no body):
```testlang
GET "/api/users/42";
DELETE "/api/users/999";
```

**POST/PUT** (with optional body and headers):
```testlang
POST "/api/login" {
  header "Authorization" = "Bearer token";
  body = "{ \"username\": \"$user\" }";
}

PUT "/api/users/$id" {
  body = "{ \"role\": \"ADMIN\" }";
}
```

### Assertions

```testlang
expect status = 200;
expect header "Content-Type" = "application/json";
expect header "Content-Type" contains "json";
expect body contains "\"token\":";
```

**Requirements:**
- Each test must have ≥1 request
- Each test must have ≥2 assertions

## 🧪 Example Test Files

### Simple Login Test

```testlang
config {
  base_url = "http://localhost:8080";
}

test Login {
  POST "/api/login" {
    header "Content-Type" = "application/json";
    body = "{ \"username\": \"admin\", \"password\": \"1234\" }";
  }
  expect status = 200;
  expect body contains "\"token\":";
}
```

### CRUD Operations

```testlang
config {
  base_url = "http://localhost:8080";
  header "Content-Type" = "application/json";
}

let userId = 42;

test GetUser {
  GET "/api/users/$userId";
  expect status = 200;
  expect body contains "\"id\": 42";
}

test UpdateUser {
  PUT "/api/users/$userId" {
    body = "{ \"role\": \"ADMIN\" }";
  }
  expect status = 200;
  expect header "Content-Type" contains "json";
  expect body contains "\"updated\": true";
}
```

## 🔧 Backend API Endpoints

The provided Spring Boot backend supports:

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/login` | Login with username/password |
| GET | `/api/users/{id}` | Get user by ID |
| PUT | `/api/users/{id}` | Update user |
| DELETE | `/api/users/{id}` | Delete user |

### Manual Testing (cURL)

```bash
# Login
curl -X POST http://localhost:8080/api/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin","password":"1234"}'

# Get user
curl http://localhost:8080/api/users/42

# Update user
curl -X PUT http://localhost:8080/api/users/42 \
  -H 'Content-Type: application/json' \
  -d '{"role":"ADMIN"}'
```

## 🛠️ Development

### Modify Scanner (Lexer)

Edit `scanner/lexer.flex` and recompile:

```bash
./scripts/compile.sh
```

### Modify Parser

Edit `parser/parser.cup` and recompile:

```bash
./scripts/compile.sh
```

### Modify Code Generation

Edit `codegen/CodeGenerator.java` and recompile:

```bash
./scripts/compile.sh
```

## 📊 Generated Code Structure

The compiler generates JUnit 5 test classes like:

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;
import java.net.http.*;

public class GeneratedTests {
  static String BASE = "http://localhost:8080";
  static HttpClient client;

  @BeforeAll
  static void setup() {
    client = HttpClient.newBuilder().build();
  }

  @Test
  void test_Login() throws Exception {
    HttpRequest req = HttpRequest.newBuilder(URI.create(BASE + "/api/login"))
      .POST(HttpRequest.BodyPublishers.ofString(...))
      .build();
    HttpResponse<String> resp = client.send(req, ...);
    
    assertEquals(200, resp.statusCode());
    assertTrue(resp.body().contains("token"));
  }
}
```

## ⚠️ Limitations (By Design)

- No JSON parsing/JSONPath
- No loops, conditionals, or macros
- No response capture/assignment
- Single-line strings only (no multiline)
- One file → one test class

## 📚 Requirements

- **Java**: 11 or higher (tested with Java 21)
- **JFlex**: 1.9.1 (auto-downloaded)
- **CUP**: 11b (auto-downloaded)
- **JUnit**: 5.10.0 (auto-downloaded)
- **Maven**: For building backend (optional)

## 🐛 Troubleshooting

### "Dependencies not found"
Run `./scripts/setup-deps.sh` first

### "Build directory not found"
Run `./scripts/compile.sh` to compile the compiler

### Backend not starting
Check if port 8080 is available, or ensure Maven is installed to build the backend

### Compilation errors
Ensure Java 11+ is installed: `java -version`

## 📖 Grammar Summary

```
program       → config? variables* tests+
config        → 'config' '{' config_items '}'
config_items  → base_url | header_decl
variables     → 'let' IDENT '=' value ';'
tests         → 'test' IDENT '{' statements+ '}'
statements    → request | assertion
request       → method path ['{' request_items '}'] ';'
assertion     → 'expect' assertion_type ';'
```

## 📄 License

Educational project for SE2062 course.

## 👤 Author

**L-Jayawardhana**

Assignment: TestLang++ DSL Compiler  
Course: SE2062 - Programming Paradigms  
Due: October 25, 2025
