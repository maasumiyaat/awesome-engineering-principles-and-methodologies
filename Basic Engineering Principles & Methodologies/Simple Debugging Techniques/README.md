# Simple Debugging Techniques

This directory covers basic debugging techniques and practices that every developer should know to effectively troubleshoot code issues.

## Overview

Debugging is a critical skill in software development. This guide covers fundamental debugging techniques, tools, and methodologies to help identify and fix code problems efficiently.

## Key Concepts

1. **Debugging Basics**
   - Console logging
   - Breakpoints
   - Step-by-step execution
   - Variable inspection
   - Call stack analysis

2. **Common Debugging Tools**
   - Browser DevTools
   - IDE debuggers
   - Logging frameworks
   - Error tracking tools

## Example Code

### Basic Console Debugging

```javascript
// Using console methods effectively
function calculateTotal(items) {
    console.log('Starting calculation with items:', items);

    let total = 0;
    for (const item of items) {
        console.log('Processing item:', item);
        
        if (!item.price) {
            console.warn('Item missing price:', item);
            continue;
        }

        total += item.price;
        console.debug('Running total:', total);
    }

    console.log('Final total:', total);
    return total;
}

// Example usage with different console methods
function processOrder(order) {
    console.group('Processing Order:', order.id);
    
    console.time('Order Processing');
    
    try {
        // Log object structure
        console.dir(order);
        
        // Create table of order items
        console.table(order.items);
        
        const total = calculateTotal(order.items);
        
        // Assert expected conditions
        console.assert(total > 0, 'Total should be positive');
        
    } catch (error) {
        console.error('Error processing order:', error);
    }
    
    console.timeEnd('Order Processing');
    console.groupEnd();
}
```

### Using Try-Catch with Debug Information

```javascript
class DebugError extends Error {
    constructor(message, context) {
        super(message);
        this.name = 'DebugError';
        this.context = context;
        this.timestamp = new Date();
    }
}

async function fetchUserData(userId) {
    try {
        console.log(`Fetching user data for ID: ${userId}`);
        
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new DebugError('Failed to fetch user data', {
                userId,
                status: response.status,
                statusText: response.statusText
            });
        }

        const data = await response.json();
        console.log('Received user data:', data);
        
        return data;
    } catch (error) {
        console.error('Error details:', {
            message: error.message,
            context: error.context,
            stack: error.stack,
            timestamp: error.timestamp
        });
        throw error;
    }
}
```

### Debugging Asynchronous Code

```javascript
async function debugAsyncFlow() {
    console.log('Starting async operation...');
    
    try {
        const result1 = await step1();
        console.log('Step 1 complete:', result1);
        
        const result2 = await step2(result1);
        console.log('Step 2 complete:', result2);
        
        const result3 = await step3(result2);
        console.log('Step 3 complete:', result3);
        
        return result3;
    } catch (error) {
        console.error('Async operation failed:', error);
        throw error;
    }
}

// Using Promise.all with debugging
async function debugParallelOperations(items) {
    console.log('Starting parallel operations with:', items);
    
    try {
        const operations = items.map(async (item, index) => {
            console.log(`Starting operation ${index} for item:`, item);
            
            const result = await processItem(item);
            console.log(`Completed operation ${index}:`, result);
            
            return result;
        });

        const results = await Promise.all(operations);
        console.log('All parallel operations complete:', results);
        
        return results;
    } catch (error) {
        console.error('Parallel operations failed:', error);
        throw error;
    }
}
```

### Debug Helper Functions

```typescript
// TypeScript debug utilities
class DebugUtils {
    private static readonly DEBUG_MODE = process.env.NODE_ENV !== 'production';

    static log(message: string, ...args: any[]) {
        if (this.DEBUG_MODE) {
            console.log(`[DEBUG] ${message}`, ...args);
        }
    }

    static trace(label: string) {
        return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {
            const originalMethod = descriptor.value;

            descriptor.value = function (...args: any[]) {
                if (!DebugUtils.DEBUG_MODE) return originalMethod.apply(this, args);

                console.group(`${label} - ${propertyKey}`);
                console.log('Arguments:', args);
                
                const start = performance.now();
                const result = originalMethod.apply(this, args);
                const end = performance.now();
                
                console.log('Result:', result);
                console.log('Execution time:', end - start, 'ms');
                console.groupEnd();
                
                return result;
            };

            return descriptor;
        };
    }

    static measureTime<T>(operation: () => T, label: string): T {
        if (!this.DEBUG_MODE) return operation();

        console.time(label);
        const result = operation();
        console.timeEnd(label);
        return result;
    }

    static async measureAsyncTime<T>(
        operation: () => Promise<T>,
        label: string
    ): Promise<T> {
        if (!this.DEBUG_MODE) return operation();

        console.time(label);
        const result = await operation();
        console.timeEnd(label);
        return result;
    }
}

// Usage example
class UserService {
    @DebugUtils.trace('UserService')
    async findUser(id: string) {
        return await db.users.findById(id);
    }

    async processUsers(userIds: string[]) {
        return await DebugUtils.measureAsyncTime(
            async () => {
                const users = await Promise.all(
                    userIds.map(id => this.findUser(id))
                );
                return users.filter(user => user !== null);
            },
            'Process Users'
        );
    }
}
```

## Debugging Techniques

1. **Console Debugging**
   - `console.log()` for basic output
   - `console.warn()` for warnings
   - `console.error()` for errors
   - `console.table()` for structured data
   - `console.time()` for performance

2. **Breakpoint Debugging**
   - Setting breakpoints
   - Step over/into/out
   - Watch expressions
   - Call stack navigation
   - Conditional breakpoints

3. **Error Handling**
   - Try-catch blocks
   - Error objects
   - Stack traces
   - Custom error types
   - Error logging

4. **Network Debugging**
   - Network requests
   - Response status
   - Headers inspection
   - Payload analysis
   - Timing information

## Tools

### Browser Tools
- Chrome DevTools
- Firefox Developer Tools
- Safari Web Inspector
- Edge DevTools

### IDE Debugging
- VS Code Debugger
- WebStorm Debugger
- Eclipse Debug Perspective
- Visual Studio Debugger

### Logging Tools
- Winston
- Bunyan
- Log4j
- Debug.js

## Best Practices

1. **Systematic Approach**
   - Reproduce the issue
   - Isolate the problem
   - Check recent changes
   - Test assumptions
   - Document findings

2. **Effective Logging**
   - Use appropriate log levels
   - Include context information
   - Format logs for readability
   - Clean up debug logs
   - Use structured logging

3. **Error Handling**
   - Catch specific errors
   - Provide useful error messages
   - Include error context
   - Log stack traces
   - Handle async errors

4. **Performance Debugging**
   - Profile code execution
   - Measure timing
   - Monitor memory usage
   - Check resource usage
   - Identify bottlenecks

## Further Learning

- Advanced Debugging Techniques
- Remote Debugging
- Production Debugging
- Performance Profiling
- Memory Leak Detection
- Debug Tools and Extensions

---

*Part of the Basic Engineering Principles & Methodologies collection*
