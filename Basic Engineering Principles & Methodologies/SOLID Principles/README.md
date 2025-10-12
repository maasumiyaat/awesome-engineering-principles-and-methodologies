# SOLID Principles

This directory covers the five SOLID principles of object-oriented design, which are fundamental guidelines for writing maintainable and scalable software.

## Overview

SOLID is an acronym for five design principles that help create more maintainable, flexible, and scalable software:

- **S**ingle Responsibility Principle (SRP)
- **O**pen/Closed Principle (OCP)
- **L**iskov Substitution Principle (LSP)
- **I**nterface Segregation Principle (ISP)
- **D**ependency Inversion Principle (DIP)

## Example Code

### Single Responsibility Principle (SRP)

```go
// Bad Example - Multiple responsibilities
type User struct {
    name     string
    email    string
    password string
}

func NewUser(name, email, password string) *User {
    return &User{
        name:     name,
        email:    email,
        password: password,
    }
}

// User properties management
func (u *User) UpdateProfile(name, email string) {
    u.name = name
    u.email = email
}

// Authentication logic
func (u *User) ValidatePassword(password string) bool {
    return u.password == password
}

// Database operations
func (u *User) Save() error {
    // Save user to database
    return database.Save(u)
}

// Email notifications
func (u *User) SendWelcomeEmail() error {
    // Send welcome email
    return emailService.Send(u.email, "Welcome!")
}

// Good Example - Separated responsibilities
type User struct {
    name     string
    email    string
    password string
}

func NewUser(name, email, password string) *User {
    return &User{
        name:     name,
        email:    email,
        password: password,
    }
}

func (u *User) GetName() string { return u.name }
func (u *User) GetEmail() string { return u.email }
func (u *User) UpdateProfile(name, email string) {
    u.name = name
    u.email = email
}

type UserAuthentication struct {
    // Dependencies could be injected here
}

func (auth *UserAuthentication) ValidatePassword(user *User, password string) bool {
    // Password validation logic
    return false
}

type UserRepository struct {
    // Database connection could be injected here
}

func (repo *UserRepository) Save(user *User) error {
    // Database operations
    return nil
}

type UserNotificationService struct {
    // Email service could be injected here
}

func (notif *UserNotificationService) SendWelcomeEmail(user *User) error {
    // Email sending logic
    return nil
}
```

### Open/Closed Principle (OCP)

```go
// Bad Example - Not extensible
type Payment struct {
    Type   string
    Amount float64
}

type PaymentProcessor struct{}

func (p *PaymentProcessor) ProcessPayment(payment Payment) error {
    switch payment.Type {
    case "credit-card":
        // Process credit card payment
        return nil
    case "paypal":
        // Process PayPal payment
        return nil
    case "bitcoin":
        // Process Bitcoin payment
        return nil
    default:
        return fmt.Errorf("unknown payment type: %s", payment.Type)
    }
}

// Good Example - Open for extension, closed for modification
type PaymentMethod interface {
    Process(amount float64) error
}

type CreditCardPayment struct {
    // Credit card specific fields
}

func (p *CreditCardPayment) Process(amount float64) error {
    // Credit card processing logic
    return nil
}

type PayPalPayment struct {
    // PayPal specific fields
}

func (p *PayPalPayment) Process(amount float64) error {
    // PayPal processing logic
    return nil
}

type BitcoinPayment struct {
    // Bitcoin specific fields
}

func (p *BitcoinPayment) Process(amount float64) error {
    // Bitcoin processing logic
    return nil
}

type PaymentProcessor struct {}

func (p *PaymentProcessor) ProcessPayment(method PaymentMethod, amount float64) error {
    return method.Process(amount)
}
```

### Liskov Substitution Principle (LSP)

```go
// Bad Example - Violates LSP
type Bird interface {
    Fly() error
}

type Sparrow struct{}

func (s *Sparrow) Fly() error {
    // Flying implementation
    return nil
}

type Penguin struct{}

func (p *Penguin) Fly() error {
    return fmt.Errorf("penguins can't fly")
}

// Good Example - Follows LSP
type Flyer interface {
    Fly() error
}

type Swimmer interface {
    Swim() error
}

type Walker interface {
    Walk() error
}

// Base bird type with common behavior
type Bird struct {
    Name string
}

func (b *Bird) Walk() error {
    // Walking implementation
    return nil
}

// FlyingBird combines walking and flying
type FlyingBird struct {
    Bird
}

func (fb *FlyingBird) Fly() error {
    // Flying implementation
    return nil
}

// Penguin combines walking and swimming
type Penguin struct {
    Bird
}

func (p *Penguin) Swim() error {
    // Swimming implementation
    return nil
}
```

### Interface Segregation Principle (ISP)

```go
// Bad Example - Fat interface
type Worker interface {
    Work() error
    Eat() error
    Sleep() error
}

type Human struct{}

func (h *Human) Work() error {
    // Working implementation
    return nil
}

func (h *Human) Eat() error {
    // Eating implementation
    return nil
}

func (h *Human) Sleep() error {
    // Sleeping implementation
    return nil
}

type Robot struct{}

func (r *Robot) Work() error {
    // Working implementation
    return nil
}

func (r *Robot) Eat() error {
    return fmt.Errorf("robots don't eat")
}

func (r *Robot) Sleep() error {
    return fmt.Errorf("robots don't sleep")
}

// Good Example - Segregated interfaces
type Workable interface {
    Work() error
}

type Eatable interface {
    Eat() error
}

type Sleepable interface {
    Sleep() error
}

// Human implements all interfaces
type Human struct{}

func (h *Human) Work() error {
    // Working implementation
    return nil
}

func (h *Human) Eat() error {
    // Eating implementation
    return nil
}

func (h *Human) Sleep() error {
    // Sleeping implementation
    return nil
}

// Robot only implements Workable
type Robot struct{}

func (r *Robot) Work() error {
    // Working implementation
    return nil
}

// Now we can have functions that only require the specific interface they need
func RunWorkProcess(w Workable) error {
    return w.Work()
}

func FeedWorker(e Eatable) error {
    return e.Eat()
}

func RestWorker(s Sleepable) error {
    return s.Sleep()
}
```

### Dependency Inversion Principle (DIP)

```typescript
// Bad Example - High-level module depends on low-level module
class MySQLDatabase {
    connect() { /* ... */ }
    query(sql: string) { /* ... */ }
}

class UserService {
    private database: MySQLDatabase;

    constructor() {
        this.database = new MySQLDatabase();
    }

    getUser(id: string) {
        return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
    }
}

// Good Example - Depends on abstractions
interface IDatabase {
    connect(): void;
    query(sql: string): any;
}

class MySQLDatabase implements IDatabase {
    connect() { /* ... */ }
    query(sql: string) { /* ... */ }
}

class PostgreSQLDatabase implements IDatabase {
    connect() { /* ... */ }
    query(sql: string) { /* ... */ }
}

class UserService {
    constructor(private database: IDatabase) {}

    getUser(id: string) {
        return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
    }
}
```

## Real-World Application Example

```typescript
// A complete example showing all SOLID principles in action

// Interfaces (ISP)
interface IUserRepository {
    findById(id: string): Promise<User>;
    save(user: User): Promise<void>;
}

interface IEmailService {
    sendEmail(to: string, subject: string, body: string): Promise<void>;
}

interface IPasswordHasher {
    hash(password: string): Promise<string>;
    verify(password: string, hash: string): Promise<boolean>;
}

// Domain Entity (SRP)
class User {
    constructor(
        private id: string,
        private email: string,
        private passwordHash: string,
        private lastLoginDate?: Date
    ) {}

    getId(): string { return this.id; }
    getEmail(): string { return this.email; }
    getPasswordHash(): string { return this.passwordHash; }
    getLastLoginDate(): Date | undefined { return this.lastLoginDate; }

    updateLastLoginDate(): void {
        this.lastLoginDate = new Date();
    }
}

// Repository Implementation (OCP & DIP)
class PostgresUserRepository implements IUserRepository {
    async findById(id: string): Promise<User> {
        // Implementation
        return new User(id, "email@example.com", "hash");
    }

    async save(user: User): Promise<void> {
        // Implementation
    }
}

// Service Implementation (SRP & DIP)
class SendGridEmailService implements IEmailService {
    async sendEmail(to: string, subject: string, body: string): Promise<void> {
        // Implementation
    }
}

class BcryptPasswordHasher implements IPasswordHasher {
    async hash(password: string): Promise<string> {
        // Implementation
        return "hashed_password";
    }

    async verify(password: string, hash: string): Promise<boolean> {
        // Implementation
        return true;
    }
}

// Use Case / Service Layer (LSP & DIP)
class AuthenticationService {
    constructor(
        private userRepository: IUserRepository,
        private emailService: IEmailService,
        private passwordHasher: IPasswordHasher
    ) {}

    async authenticate(email: string, password: string): Promise<User> {
        const user = await this.userRepository.findById(email);
        if (!user) {
            throw new Error('User not found');
        }

        const isValid = await this.passwordHasher.verify(
            password,
            user.getPasswordHash()
        );

        if (!isValid) {
            throw new Error('Invalid password');
        }

        user.updateLastLoginDate();
        await this.userRepository.save(user);

        return user;
    }

    async registerUser(email: string, password: string): Promise<User> {
        const passwordHash = await this.passwordHasher.hash(password);
        const user = new User(crypto.randomUUID(), email, passwordHash);
        
        await this.userRepository.save(user);
        await this.emailService.sendEmail(
            email,
            'Welcome!',
            'Thank you for registering.'
        );

        return user;
    }
}

// Usage
const authService = new AuthenticationService(
    new PostgresUserRepository(),
    new SendGridEmailService(),
    new BcryptPasswordHasher()
);
```

## Best Practices

1. **Single Responsibility Principle**
   - Each class should have one reason to change
   - Separate concerns into different classes
   - Keep classes focused and cohesive

2. **Open/Closed Principle**
   - Use interfaces and abstract classes
   - Design for extensibility
   - Avoid modifying existing code

3. **Liskov Substitution Principle**
   - Subtypes must be substitutable for base types
   - Maintain contract of base class
   - Use composition over inheritance when appropriate

4. **Interface Segregation Principle**
   - Keep interfaces small and focused
   - Don't force clients to implement unnecessary methods
   - Split large interfaces into smaller ones

5. **Dependency Inversion Principle**
   - Depend on abstractions, not concrete implementations
   - Use dependency injection
   - Implement inversion of control

## Benefits of SOLID

1. **Maintainability**
   - Easier to understand and modify code
   - Reduced coupling between components
   - Better organization of code

2. **Flexibility**
   - Easier to extend functionality
   - Better adaptability to changes
   - Improved reusability

3. **Testability**
   - Easier to write unit tests
   - Better isolation of components
   - Improved test coverage

## Further Learning

- Design Patterns
- Clean Architecture
- Domain-Driven Design
- Test-Driven Development
- Dependency Injection
- Clean Code Principles

---

*Part of the Basic Engineering Principles & Methodologies collection*
