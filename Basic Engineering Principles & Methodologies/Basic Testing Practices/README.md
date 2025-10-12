# Basic Testing Practices

This directory covers fundamental software testing practices that every developer should know and implement in their projects.

## Overview

Basic testing practices are essential skills that help ensure code quality, catch bugs early, and make code maintenance easier. This guide covers fundamental testing concepts and practical examples.

## Key Concepts

1. **Types of Testing**
   - Unit Testing
   - Integration Testing
   - Functional Testing
   - Manual Testing

2. **Testing Principles**
   - Test Early and Often
   - Test One Thing at a Time
   - Keep Tests Simple
   - Make Tests Repeatable
   - Tests Should Be Independent

## Example Code

### Unit Testing Example (Go)

```go
// calculator.go
package calculator

import "errors"

type Calculator struct{}

func (c *Calculator) Add(a, b float64) float64 {
    return a + b
}

func (c *Calculator) Subtract(a, b float64) float64 {
    return a - b
}

func (c *Calculator) Multiply(a, b float64) float64 {
    return a * b
}

func (c *Calculator) Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// calculator_test.go
package calculator

import (
    "testing"
)

func TestCalculator(t *testing.T) {
    calc := &Calculator{}

    t.Run("adds two numbers correctly", func(t *testing.T) {
        if got := calc.Add(2, 3); got != 5 {
            t.Errorf("Add(2, 3) = %v; want 5", got)
        }
        if got := calc.Add(-1, 1); got != 0 {
            t.Errorf("Add(-1, 1) = %v; want 0", got)
        }
    })

    t.Run("subtracts two numbers correctly", func(t *testing.T) {
        if got := calc.Subtract(5, 3); got != 2 {
            t.Errorf("Subtract(5, 3) = %v; want 2", got)
        }
        if got := calc.Subtract(1, 1); got != 0 {
            t.Errorf("Subtract(1, 1) = %v; want 0", got)
        }
    })

    t.Run("multiplies two numbers correctly", func(t *testing.T) {
        if got := calc.Multiply(2, 3); got != 6 {
            t.Errorf("Multiply(2, 3) = %v; want 6", got)
        }
        if got := calc.Multiply(-2, 3); got != -6 {
            t.Errorf("Multiply(-2, 3) = %v; want -6", got)
        }
    })

    t.Run("divides two numbers correctly", func(t *testing.T) {
        got, err := calc.Divide(6, 2)
        if err != nil {
            t.Errorf("Divide(6, 2) unexpected error: %v", err)
        }
        if got != 3 {
            t.Errorf("Divide(6, 2) = %v; want 3", got)
        }

        got, err = calc.Divide(5, 2)
        if err != nil {
            t.Errorf("Divide(5, 2) unexpected error: %v", err)
        }
        if got != 2.5 {
            t.Errorf("Divide(5, 2) = %v; want 2.5", got)
        }
    })

    t.Run("returns error when dividing by zero", func(t *testing.T) {
        _, err := calc.Divide(5, 0)
        if err == nil {
            t.Error("Divide(5, 0) expected error; got nil")
        }
        if err.Error() != "division by zero" {
            t.Errorf("Divide(5, 0) error = %v; want 'division by zero'", err)
        }
    })
})
```

### Integration Testing Example (Go)

```go
// user.go
package user

type User struct {
    Username string
    Email    string
}

// database.go
package user

type Database struct {
    users []User
}

func NewDatabase() *Database {
    return &Database{
        users: make([]User, 0),
    }
}

func (db *Database) FindUserByEmail(email string) *User {
    for _, user := range db.users {
        if user.Email == email {
            return &user
        }
    }
    return nil
}

func (db *Database) CreateUser(user User) User {
    db.users = append(db.users, user)
    return user
}

// service.go
package user

import "errors"

type UserService struct {
    db *Database
}

func NewUserService(db *Database) *UserService {
    return &UserService{db: db}
}

func (s *UserService) CreateUser(username, email string) (User, error) {
    if username == "" || email == "" {
        return User{}, errors.New("username and email are required")
    }

    if s.db.FindUserByEmail(email) != nil {
        return User{}, errors.New("email already exists")
    }

    return s.db.CreateUser(User{
        Username: username,
        Email:    email,
    }), nil
}

// service_test.go
package user

import (
    "testing"
)

func TestCreateUser(t *testing.T) {
    t.Run("creates user successfully", func(t *testing.T) {
        db := NewDatabase()
        service := NewUserService(db)

        user, err := service.CreateUser("john_doe", "john@example.com")
        if err != nil {
            t.Errorf("unexpected error: %v", err)
        }

        if user.Username != "john_doe" {
            t.Errorf("username = %v; want john_doe", user.Username)
        }
        if user.Email != "john@example.com" {
            t.Errorf("email = %v; want john@example.com", user.Email)
        }
        if len(db.users) != 1 {
            t.Errorf("db.users length = %v; want 1", len(db.users))
        }
    })

    t.Run("fails with duplicate email", func(t *testing.T) {
        db := NewDatabase()
        service := NewUserService(db)

        _, err := service.CreateUser("john_doe", "john@example.com")
        if err != nil {
            t.Errorf("unexpected error on first create: %v", err)
        }

        _, err = service.CreateUser("jane_doe", "john@example.com")
        if err == nil {
            t.Error("expected error for duplicate email; got nil")
        }
        if err.Error() != "email already exists" {
            t.Errorf("error = %v; want 'email already exists'", err)
        }
    })
}
```

### Functional Testing Example (Go/chromedp)

```go
// login_test.go
package test

import (
    "context"
    "testing"
    "time"

    "github.com/chromedp/chromedp"
)

func TestSuccessfulLogin(t *testing.T) {
    // Create a new context
    ctx, cancel := chromedp.NewContext(context.Background())
    defer cancel()

    // Create a timeout context
    ctx, cancel = context.WithTimeout(ctx, 15*time.Second)
    defer cancel()

    // Run the test
    err := chromedp.Run(ctx,
        // Navigate to the login page
        chromedp.Navigate("http://example.com/login"),

        // Wait for and find username field, then type into it
        chromedp.WaitVisible("#username"),
        chromedp.SendKeys("#username", "testuser"),

        // Wait for and find password field, then type into it
        chromedp.WaitVisible("#password"),
        chromedp.SendKeys("#password", "password123"),

        // Wait for and click the login button
        chromedp.WaitVisible("#login-button"),
        chromedp.Click("#login-button"),

        // Wait for dashboard to appear, confirming successful login
        chromedp.WaitVisible("#dashboard"),
    )

    if err != nil {
        t.Fatalf("Failed to complete login test: %v", err)
    }
}
```

## Best Practices

1. **Write Tests First (TDD)**
   - Define expected behavior before implementation
   - Write failing tests first
   - Implement code to make tests pass
   - Refactor while keeping tests green

2. **Test Structure (AAA Pattern)**
   - Arrange: Set up test conditions
   - Act: Execute the code being tested
   - Assert: Verify the results

3. **Test Coverage**
   - Aim for comprehensive test coverage
   - Focus on critical business logic
   - Test edge cases and error conditions
   - Don't just test happy paths

4. **Test Maintenance**
   - Keep tests simple and readable
   - Use descriptive test names
   - Update tests when requirements change
   - Remove obsolete tests

## Common Testing Tools

### JavaScript
- Jest
- Mocha
- Cypress
- Testing Library

### Python
- pytest
- unittest
- Selenium
- Robot Framework

### Java
- JUnit
- TestNG
- Mockito
- Selenium WebDriver

## Further Learning

- Test-Driven Development (TDD)
- Behavior-Driven Development (BDD)
- Continuous Integration/Continuous Deployment (CI/CD)
- Automated Testing Frameworks
- Performance Testing Basics
- Security Testing Fundamentals

---

*Part of the Basic Engineering Principles & Methodologies collection*
