# DRY Principle (Don't Repeat Yourself)

This directory covers the DRY (Don't Repeat Yourself) principle, a fundamental concept in software development that aims to reduce repetition in code.

## Overview

The DRY principle states that "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system." This means avoiding code duplication and instead creating reusable components.

## Key Concepts

1. **Code Reusability**
   - Functions and Methods
   - Classes and Objects
   - Modules and Libraries
   - Templates and Generics

2. **Knowledge Representation**
   - Business Logic
   - Configuration
   - Documentation
   - Database Schema

## Example Code

### Bad Practice (Not DRY)

```go
// Example of code that violates DRY principle
type UserManager struct{}

type User struct {
    Email    string
    Password string
}

func (um *UserManager) validateUserRegistration(user User) error {
    if user.Email == "" {
        return fmt.Errorf("email is required")
    }
    if !strings.Contains(user.Email, "@") {
        return fmt.Errorf("invalid email format")
    }
    if user.Password == "" {
        return fmt.Errorf("password is required")
    }
    if len(user.Password) < 8 {
        return fmt.Errorf("password must be at least 8 characters")
    }
    return nil
}

func (um *UserManager) validateUserUpdate(user User) error {
    // Duplicated validation logic
    if user.Email == "" {
        return fmt.Errorf("email is required")
    }
    if !strings.Contains(user.Email, "@") {
        return fmt.Errorf("invalid email format")
    }
    if user.Password == "" {
        return fmt.Errorf("password is required")
    }
    if len(user.Password) < 8 {
        return fmt.Errorf("password must be at least 8 characters")
    }
    return nil
}
```

### Good Practice (DRY)

```go
package validation

import (
    "fmt"
    "strings"
)

type Validator struct{}

func (v *Validator) ValidateEmail(email string) error {
    if email == "" {
        return fmt.Errorf("email is required")
    }
    if !strings.Contains(email, "@") {
        return fmt.Errorf("invalid email format")
    }
    return nil
}

func (v *Validator) ValidatePassword(password string) error {
    if password == "" {
        return fmt.Errorf("password is required")
    }
    if len(password) < 8 {
        return fmt.Errorf("password must be at least 8 characters")
    }
    return nil
}

type UserManager struct {
    validator *Validator
}

func NewUserManager() *UserManager {
    return &UserManager{
        validator: &Validator{},
    }
}

func (um *UserManager) ValidateUserRegistration(user User) error {
    if err := um.validator.ValidateEmail(user.Email); err != nil {
        return err
    }
    if err := um.validator.ValidatePassword(user.Password); err != nil {
        return err
    }
    return nil
}

func (um *UserManager) ValidateUserUpdate(user User) error {
    return um.ValidateUserRegistration(user) // Reuse the same validation
}
```

### Using Generics (Go)

```go
package repository

import (
    "context"
    "fmt"
)

// Entity represents a generic entity with an ID
type Entity interface {
    GetID() string
}

// Repository provides generic CRUD operations
type Repository[T Entity] struct {
    items map[string]T
}

// NewRepository creates a new repository instance
func NewRepository[T Entity]() *Repository[T] {
    return &Repository[T]{
        items: make(map[string]T),
    }
}

// Create adds a new item to the repository
func (r *Repository[T]) Create(ctx context.Context, item T) (T, error) {
    if r.items[item.GetID()] != nil {
        var empty T
        return empty, fmt.Errorf("item with ID %s already exists", item.GetID())
    }
    r.items[item.GetID()] = item
    return item, nil
}

// FindAll returns all items in the repository
func (r *Repository[T]) FindAll(ctx context.Context) ([]T, error) {
    items := make([]T, 0, len(r.items))
    for _, item := range r.items {
        items = append(items, item)
    }
    return items, nil
}

// FindByID returns an item by its ID
func (r *Repository[T]) FindByID(ctx context.Context, id string) (T, error) {
    if item, exists := r.items[id]; exists {
        return item, nil
    }
    var empty T
    return empty, fmt.Errorf("item with ID %s not found", id)
}

// Update updates an existing item
func (r *Repository[T]) Update(ctx context.Context, item T) (T, error) {
    if _, exists := r.items[item.GetID()]; !exists {
        var empty T
        return empty, fmt.Errorf("item with ID %s not found", item.GetID())
    }
    r.items[item.GetID()] = item
    return item, nil
}

// Delete removes an item from the repository
func (r *Repository[T]) Delete(ctx context.Context, id string) error {
    if _, exists := r.items[id]; !exists {
        return fmt.Errorf("item with ID %s not found", id)
    }
    delete(r.items, id)
    return nil
}

// Specific implementations
type User struct {
    ID    string
    Name  string
    Email string
}

func (u User) GetID() string { return u.ID }

type UserRepository struct {
    *Repository[User]
}

func NewUserRepository() *UserRepository {
    return &UserRepository{
        Repository: NewRepository[User](),
    }
}

// FindByEmail is a user-specific method
func (r *UserRepository) FindByEmail(ctx context.Context, email string) (User, error) {
    for _, user := range r.items {
        if user.Email == email {
            return user, nil
        }
    }
    return User{}, fmt.Errorf("user with email %s not found", email)
}

type Product struct {
    ID    string
    Name  string
    Price float64
}

func (p Product) GetID() string { return p.ID }

type ProductRepository struct {
    *Repository[Product]
}

func NewProductRepository() *ProductRepository {
    return &ProductRepository{
        Repository: NewRepository[Product](),
    }
}

// FindByPriceRange is a product-specific method
func (r *ProductRepository) FindByPriceRange(ctx context.Context, min, max float64) ([]Product, error) {
    var products []Product
    for _, product := range r.items {
        if product.Price >= min && product.Price <= max {
            products = append(products, product)
        }
    }
    return products, nil
}
```

### Configuration Management Example

```go
package config

import (
    "os"
    "strconv"
)

// Config holds the application configuration
type Config struct {
    Database DatabaseConfig
    API      APIConfig
}

type DatabaseConfig struct {
    Host string
    Port int
    Name string
}

type APIConfig struct {
    Port  int
    CORS  CORSConfig
}

type CORSConfig struct {
    Origin string
}

// GetConfig returns the configuration for the current environment
func GetConfig() *Config {
    if os.Getenv("GO_ENV") == "production" {
        return getProductionConfig()
    }
    return getDevelopmentConfig()
}

func getDevelopmentConfig() *Config {
    return &Config{
        Database: DatabaseConfig{
            Host: "localhost",
            Port: 5432,
            Name: "dev_db",
        },
        API: APIConfig{
            Port: 3000,
            CORS: CORSConfig{
                Origin: "*",
            },
        },
    }
}

func getProductionConfig() *Config {
    port, _ := strconv.Atoi(os.Getenv("DB_PORT"))
    apiPort, _ := strconv.Atoi(os.Getenv("API_PORT"))

    return &Config{
        Database: DatabaseConfig{
            Host: os.Getenv("DB_HOST"),
            Port: port,
            Name: os.Getenv("DB_NAME"),
        },
        API: APIConfig{
            Port: apiPort,
            CORS: CORSConfig{
                Origin: os.Getenv("CORS_ORIGIN"),
            },
        },
    }
}

// Usage example:
type Database struct {
    config DatabaseConfig
}

func NewDatabase(config DatabaseConfig) *Database {
    return &Database{config: config}
}

func (db *Database) Connect() error {
    // Uses centralized configuration
    connStr := fmt.Sprintf(
        "host=%s port=%d dbname=%s",
        db.config.Host,
        db.config.Port,
        db.config.Name,
    )
    // Connection logic here
    return nil
}

type APIServer struct {
    config APIConfig
}

func NewAPIServer(config APIConfig) *APIServer {
    return &APIServer{config: config}
}

func (s *APIServer) Start() error {
    // Uses same configuration
    addr := fmt.Sprintf(":%d", s.config.Port)
    // Server setup logic here
    return nil
}

// Main application setup
func main() {
    config := GetConfig()
    
    db := NewDatabase(config.Database)
    if err := db.Connect(); err != nil {
        log.Fatal(err)
    }
    
    server := NewAPIServer(config.API)
    if err := server.Start(); err != nil {
        log.Fatal(err)
    }
}
```

## Best Practices

1. **Identify Common Patterns**
   - Look for repeated code blocks
   - Identify similar business logic
   - Spot repeated configurations
   - Notice duplicate validation rules

2. **Create Reusable Components**
   - Extract common functionality into functions
   - Use classes for related functionality
   - Create utility modules
   - Implement generic solutions

3. **Use Design Patterns**
   - Template Method Pattern
   - Strategy Pattern
   - Factory Pattern
   - Observer Pattern

4. **Maintain Single Source of Truth**
   - Centralize configuration
   - Use constants and enums
   - Create shared utilities
   - Implement service layers

## Common Anti-patterns to Avoid

1. **Copy-Paste Programming**
   - Copying code instead of reusing
   - Duplicating business logic
   - Repeating validation rules
   - Copying configuration values

2. **Premature Abstraction**
   - Over-generalizing simple code
   - Creating unnecessary complexity
   - Abstracting before patterns emerge
   - Making code too generic

## Tools and Techniques

1. **Code Analysis**
   - Linters
   - Code duplication detectors
   - Static analysis tools
   - Complexity analyzers

2. **Refactoring Tools**
   - IDE refactoring features
   - Code cleanup tools
   - Dead code eliminators
   - Dependency analyzers

## Further Learning

- Design Patterns
- Code Refactoring
- Clean Code Principles
- Modular Programming
- Software Architecture
- Code Quality Metrics

---

*Part of the Basic Engineering Principles & Methodologies collection*
