# YAGNI Principle (You Aren't Gonna Need It)

This directory covers the YAGNI principle, which advocates against adding functionality until it is necessary.

## Overview

YAGNI is a principle of extreme programming (XP) that states a programmer should not add functionality until deemed necessary. It helps prevent over-engineering and keeps code focused on current requirements.

## Key Concepts

1. **Code Simplicity**
   - Write only what you need now
   - Avoid speculative features
   - Focus on current requirements
   - Minimize complexity

2. **Development Efficiency**
   - Save development time
   - Reduce maintenance burden
   - Focus on essential features
   - Iterative development

## Example Code

### Over-Engineered Example (Not Following YAGNI)

```go
// Over-engineered, speculative generalization
type DataSource interface {
    Connect() error
    Disconnect() error
    Query(query string, args ...interface{}) ([]interface{}, error)
    BeginTransaction() error
    CommitTransaction() error
    RollbackTransaction() error
    Backup() error
    Restore(backupID string) error
    Migrate(version string) error
    Validate() (bool, error)
    Optimize() error
}

type DatabaseService struct {
    config              map[string]interface{}
    connection          interface{}
    transactionLevel   int
    backupHistory      []string
    migrationHistory   []string
    validationRules    []interface{}
    optimizationRules  []interface{}

    constructor(config: any) {
        this.config = config;
    }

    async connect(): Promise<void> {
        // Complex connection logic
    }

    async disconnect(): Promise<void> {
        // Disconnect logic
    }

    async query<T>(query: string): Promise<T[]> {
        // Query execution
        return [];
    }

    async beginTransaction(): Promise<void> {
        // Transaction management
    }

    async commitTransaction(): Promise<void> {
        // Commit logic
    }

    async rollbackTransaction(): Promise<void> {
        // Rollback logic
    }

    async backup(): Promise<void> {
        // Backup implementation
    }

    async restore(backupId: string): Promise<void> {
        // Restore implementation
    }

    async migrate(version: string): Promise<void> {
        // Migration logic
    }

    async validate(): Promise<boolean> {
        // Validation logic
        return true;
    }

    async optimize(): Promise<void> {
        // Optimization logic
    }
}
```

### YAGNI-Compliant Example

```typescript
// Simple, focused implementation
class DatabaseService {
    private connection: any;

    constructor(private connectionString: string) {}

    async connect(): Promise<void> {
        this.connection = await createConnection(this.connectionString);
    }

    async query<T>(sql: string, params: any[] = []): Promise<T[]> {
        return this.connection.query(sql, params);
    }
}

// Usage
async function getUserById(id: string): Promise<User> {
    const db = new DatabaseService('connection_string');
    await db.connect();
    
    const [user] = await db.query<User>(
        'SELECT * FROM users WHERE id = ?',
        [id]
    );
    
    return user;
}
```

### Real-World Example: User Management System

```typescript
// Before (Over-engineered)
interface IUserPreferences {
    theme: string;
    language: string;
    timezone: string;
    notifications: {
        email: boolean;
        push: boolean;
        sms: boolean;
        frequency: 'immediate' | 'daily' | 'weekly';
        quiet_hours: {
            start: number;
            end: number;
        };
    };
    accessibility: {
        high_contrast: boolean;
        font_size: number;
        screen_reader: boolean;
    };
    privacy: {
        profile_visibility: 'public' | 'private' | 'friends';
        activity_tracking: boolean;
        data_sharing: boolean;
    };
}

class UserManager {
    async createUser(userData: {
        username: string;
        email: string;
        password: string;
        preferences?: IUserPreferences;
    }) {
        // Complex user creation with all possible preferences
    }

    async updatePreferences(userId: string, preferences: Partial<IUserPreferences>) {
        // Complex preference update logic
    }
}

// After (YAGNI-compliant)
interface User {
    id: string;
    username: string;
    email: string;
    passwordHash: string;
}

class UserService {
    constructor(private db: DatabaseService) {}

    async createUser(username: string, email: string, password: string): Promise<User> {
        const passwordHash = await hashPassword(password);
        
        return this.db.query(
            'INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?) RETURNING *',
            [username, email, passwordHash]
        );
    }
}
```

### Feature Flag Example (When You Actually Need It)

```typescript
// Simple feature flag implementation when needed
interface FeatureFlags {
    newUserInterface: boolean;
    betaFeatures: boolean;
}

class FeatureService {
    private flags: FeatureFlags;

    constructor() {
        // Load flags from environment or configuration
        this.flags = {
            newUserInterface: process.env.ENABLE_NEW_UI === 'true',
            betaFeatures: process.env.ENABLE_BETA === 'true'
        };
    }

    isEnabled(feature: keyof FeatureFlags): boolean {
        return this.flags[feature] || false;
    }
}

// Usage
class UserInterface {
    constructor(private features: FeatureService) {}

    renderUI(): void {
        if (this.features.isEnabled('newUserInterface')) {
            this.renderNewUI();
        } else {
            this.renderClassicUI();
        }
    }

    private renderNewUI(): void {
        // New UI implementation
    }

    private renderClassicUI(): void {
        // Classic UI implementation
    }
}
```

### Simplified Example (Following YAGNI)

```go
// Simple, focused implementation based on current needs
type Database struct {
    db *sql.DB
}

func NewDatabase(connectionString string) (*Database, error) {
    db, err := sql.Open("postgres", connectionString)
    if err != nil {
        return nil, fmt.Errorf("failed to connect to database: %w", err)
    }
    return &Database{db: db}, nil
}

func (d *Database) Query(query string, args ...interface{}) ([]map[string]interface{}, error) {
    rows, err := d.db.Query(query, args...)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    defer rows.Close()

    // Simple result processing
    var results []map[string]interface{}
    columns, _ := rows.Columns()
    
    for rows.Next() {
        result := make(map[string]interface{})
        values := make([]interface{}, len(columns))
        for i := range values {
            values[i] = new(interface{})
        }
        
        if err := rows.Scan(values...); err != nil {
            return nil, fmt.Errorf("failed to scan row: %w", err)
        }
        
        for i, col := range columns {
            result[col] = *(values[i].(*interface{}))
        }
        results = append(results, result)
    }
    
    return results, nil
}
```

In this simplified example:
1. We only implemented what we currently need: database connection and querying
2. No speculative features like backup, migration, or optimization
3. Simple, straightforward interface that meets current requirements
4. Can be extended later when new features are actually needed

## Best Practices

1. **Start Simple**
   - Implement only required features
   - Add complexity when needed
   - Focus on current requirements
   - Use iterative development

2. **Avoid Speculation**
   - Don't add "future-proof" features
   - Wait for concrete requirements
   - Resist premature abstraction
   - Keep interfaces minimal

3. **Refactor When Needed**
   - Refactor when patterns emerge
   - Wait for multiple use cases
   - Keep code maintainable
   - Follow actual requirements

4. **Question Additions**
   - Is it needed right now?
   - Do we have clear requirements?
   - What's the cost of waiting?
   - Can we add it later?

## Common Anti-patterns to Avoid

1. **Speculative Generalization**
   - Adding unused interfaces
   - Creating unnecessary abstractions
   - Building "just in case" features
   - Over-parameterizing functions

2. **Premature Optimization**
   - Complex caching systems
   - Advanced scaling features
   - Unnecessary performance tweaks
   - Over-engineered architectures

3. **Framework Syndrome**
   - Building frameworks too early
   - Generic solutions for specific problems
   - Excessive abstraction layers
   - Over-configurable systems

## Benefits of YAGNI

1. **Reduced Complexity**
   - Simpler codebase
   - Easier maintenance
   - Better understanding
   - Fewer bugs

2. **Faster Development**
   - Less code to write
   - Quicker iterations
   - Focused development
   - Earlier delivery

3. **Better Resources Usage**
   - Efficient time management
   - Reduced technical debt
   - Lower maintenance costs
   - Focused testing effort

## Further Learning

- Extreme Programming (XP)
- Agile Development
- Lean Software Development
- Minimum Viable Product (MVP)
- Technical Debt Management
- Code Refactoring

---

*Part of the Basic Engineering Principles & Methodologies collection*
