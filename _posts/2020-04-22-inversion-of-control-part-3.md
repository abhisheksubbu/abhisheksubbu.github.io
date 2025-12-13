---
title: Inversion of Control - Part 3
layout: blog
category: [Architecture, Deep Dive]
excerpt: In this blog post, we'll dive into advanced topics like sophisticated DI containers, the Service Locator pattern, and how popular JavaScript frameworks implement Inversion of Control. We'll explore real-world implementations with practical examples.
comments: true
---

In [Part 1](/2020/04/20/inversion-of-control-part-1), we learned what Inversion of Control is and why it matters. In [Part 2](/2020/04/21/inversion-of-control-part-2), we explored Dependency Injection patterns with practical JavaScript examples.

Now, let's dive into **advanced topics** that will help you understand how IOC is implemented in real-world applications and frameworks.

## Advanced Dependency Injection Container

In Part 2, we saw a simple DI container. Let's build a more sophisticated one that handles:
- Singleton instances
- Automatic dependency resolution
- Lifecycle management
- Circular dependency detection

```javascript
class AdvancedDIContainer {
    constructor() {
        this.services = new Map();
        this.instances = new Map();
        this.resolving = new Set(); // Track what's being resolved
    }

    // Register a service with its dependencies
    register(name, factory, options = {}) {
        this.services.set(name, {
            factory,
            singleton: options.singleton !== false, // Default to singleton
            dependencies: options.dependencies || []
        });
    }

    // Resolve a service and all its dependencies
    resolve(name) {
        // Check if already resolved (for singletons)
        if (this.instances.has(name)) {
            return this.instances.get(name);
        }

        // Detect circular dependencies
        if (this.resolving.has(name)) {
            throw new Error(`Circular dependency detected: ${name}`);
        }

        const service = this.services.get(name);
        if (!service) {
            throw new Error(`Service ${name} not registered`);
        }

        // Mark as resolving
        this.resolving.add(name);

        try {
            // Resolve dependencies first
            const dependencies = service.dependencies.map(dep => this.resolve(dep));
            
            // Create instance
            const instance = service.factory(...dependencies);

            // Store if singleton
            if (service.singleton) {
                this.instances.set(name, instance);
            }

            return instance;
        } finally {
            // Remove from resolving set
            this.resolving.delete(name);
        }
    }

    // Clear all instances (useful for testing)
    clear() {
        this.instances.clear();
    }
}

// Example usage
const container = new AdvancedDIContainer();

// Register services with their dependencies
container.register('logger', () => {
    return {
        log: (message) => console.log(`[LOG] ${message}`)
    };
}, { singleton: true });

container.register('database', () => {
    return {
        query: (sql) => console.log(`Executing: ${sql}`)
    };
}, { singleton: true });

container.register('userRepository', (database, logger) => {
    return {
        findById: (id) => {
            logger.log(`Finding user ${id}`);
            return database.query(`SELECT * FROM users WHERE id = ${id}`);
        }
    };
}, { dependencies: ['database', 'logger'] });

container.register('userService', (userRepository, logger) => {
    return {
        getUser: (id) => {
            logger.log(`Getting user ${id}`);
            return userRepository.findById(id);
        }
    };
}, { dependencies: ['userRepository', 'logger'] });

// Use the container - dependencies are automatically resolved!
const userService = container.resolve('userService');
userService.getUser(123);
```

**Key Features:**
- **Singleton support**: Services can be registered as singletons (created once) or transient (new instance each time)
- **Automatic resolution**: Dependencies are automatically resolved recursively
- **Circular dependency detection**: Prevents infinite loops
- **Lifecycle management**: Controls when instances are created and reused

## Service Locator Pattern

The **Service Locator** is another pattern that implements IOC, though it's considered less ideal than Dependency Injection. Let's understand it:

```javascript
class ServiceLocator {
    constructor() {
        this.services = new Map();
    }

    register(name, service) {
        this.services.set(name, service);
    }

    get(name) {
        const service = this.services.get(name);
        if (!service) {
            throw new Error(`Service ${name} not found`);
        }
        return service;
    }
}

// Global service locator (singleton)
const serviceLocator = new ServiceLocator();

// Register services
serviceLocator.register('emailService', new EmailService());
serviceLocator.register('logger', {
    log: (msg) => console.log(msg)
});

// Classes use the service locator to get dependencies
class UserService {
    constructor() {
        // Getting dependencies from the locator
        this.emailService = serviceLocator.get('emailService');
        this.logger = serviceLocator.get('logger');
    }

    registerUser(name, email) {
        this.logger.log(`Registering ${name}`);
        this.emailService.sendEmail(email, "Welcome!");
    }
}

// Usage
const userService = new UserService();
userService.registerUser("John", "john@example.com");
```

**Service Locator vs Dependency Injection:**

| Aspect | Service Locator | Dependency Injection |
|--------|----------------|---------------------|
| **Coupling** | Class knows about the locator | Class doesn't know about container |
| **Testability** | Harder (need to mock locator) | Easier (just inject mocks) |
| **Explicitness** | Dependencies are hidden | Dependencies are explicit |
| **Flexibility** | Less flexible | More flexible |

**Why Dependency Injection is preferred:**
- Dependencies are **explicit** (you can see them in the constructor)
- Easier to **test** (no need to set up a locator)
- Better **separation of concerns**

## How Popular Frameworks Implement IOC

Let's see how real-world frameworks implement IOC:

### 1. Angular (TypeScript/JavaScript)

Angular has a built-in Dependency Injection system:

```typescript
// In Angular, you use decorators and constructor injection
import { Injectable, Inject } from '@angular/core';

// Define a service
@Injectable({
  providedIn: 'root' // Makes it a singleton
})
export class EmailService {
  sendEmail(to: string, message: string) {
    console.log(`Email to ${to}: ${message}`);
  }
}

// Inject it into a component
@Component({
  selector: 'app-user',
  template: '<div>User Component</div>'
})
export class UserComponent {
  // Angular automatically injects EmailService
  constructor(private emailService: EmailService) {}
  
  registerUser(name: string, email: string) {
    this.emailService.sendEmail(email, "Welcome!");
  }
}
```

**How Angular's DI works:**
- Uses **decorators** (`@Injectable`, `@Inject`) to mark services
- **Constructor injection** is the primary method
- Angular's **injector** automatically resolves dependencies
- Supports **hierarchical injectors** (component-level, module-level, root-level)

### 2. React Context API (React Hooks)

React uses Context API for dependency injection:

```javascript
import React, { createContext, useContext } from 'react';

// Create a context (like a service container)
const ServicesContext = createContext();

// Service implementations
class EmailService {
    sendEmail(to, message) {
        console.log(`Email to ${to}: ${message}`);
    }
}

// Provider component (sets up dependencies)
export function ServicesProvider({ children }) {
    const emailService = new EmailService();
    
    return (
        <ServicesContext.Provider value={{ emailService }}>
            {children}
        </ServicesContext.Provider>
    );
}

// Hook to use services (inject dependencies)
export function useServices() {
    return useContext(ServicesContext);
}

// Component using the service
function UserComponent() {
    const { emailService } = useServices(); // Dependency injection!
    
    const handleRegister = (name, email) => {
        emailService.sendEmail(email, "Welcome!");
    };
    
    return <button onClick={() => handleRegister("John", "john@example.com")}>
        Register
    </button>;
}

// App setup
function App() {
    return (
        <ServicesProvider>
            <UserComponent />
        </ServicesProvider>
    );
}
```

**How React's approach works:**
- **Context API** acts as the DI container
- **Provider** component registers services
- **Hooks** (`useContext`) inject dependencies
- Works well with React's component tree

### 3. Node.js with InversifyJS

InversifyJS is a popular DI container for Node.js and TypeScript:

```javascript
const { Container, injectable, inject } = require('inversify');

// Define tokens (identifiers for dependencies)
const TYPES = {
    EmailService: Symbol.for('EmailService'),
    Logger: Symbol.for('Logger')
};

// Mark classes as injectable
@injectable()
class EmailService {
    sendEmail(to, message) {
        console.log(`Email to ${to}: ${message}`);
    }
}

@injectable()
class Logger {
    log(message) {
        console.log(`[LOG] ${message}`);
    }
}

@injectable()
class UserService {
    constructor(
        @inject(TYPES.EmailService) emailService,
        @inject(TYPES.Logger) logger
    ) {
        this.emailService = emailService;
        this.logger = logger;
    }

    registerUser(name, email) {
        this.logger.log(`Registering ${name}`);
        this.emailService.sendEmail(email, "Welcome!");
    }
}

// Setup container
const container = new Container();
container.bind(TYPES.EmailService).to(EmailService);
container.bind(TYPES.Logger).to(Logger);
container.bind(UserService).to(UserService);

// Resolve dependencies
const userService = container.get(UserService);
userService.registerUser("John", "john@example.com");
```

**InversifyJS features:**
- Uses **decorators** for metadata
- **Type-safe** with TypeScript
- Supports **interfaces** and **symbols** for binding
- Powerful **binding** system (singleton, transient, etc.)

### 4. Express.js Middleware Pattern

Express.js uses middleware, which is a form of IOC:

```javascript
// Express middleware (dependency injection for request/response)
const express = require('express');
const app = express();

// Service
class AuthService {
    authenticate(token) {
        return token === 'valid-token';
    }
}

// Middleware that injects services into requests
function injectServices(req, res, next) {
    req.services = {
        auth: new AuthService(),
        logger: {
            log: (msg) => console.log(msg)
        }
    };
    next();
}

// Use middleware
app.use(injectServices);

// Route handler receives services via request object
app.get('/user', (req, res) => {
    // Services are injected via req.services
    const { auth, logger } = req.services;
    
    if (auth.authenticate(req.headers.token)) {
        logger.log('User authenticated');
        res.json({ message: 'User data' });
    } else {
        res.status(401).json({ error: 'Unauthorized' });
    }
});
```

## Building a Framework-Agnostic DI Container

Let's create a production-ready DI container that you can use in any JavaScript project:

```javascript
class ProductionDIContainer {
    constructor() {
        this.registrations = new Map();
        this.instances = new Map();
        this.resolving = new Set();
    }

    // Register with various options
    register(name, config) {
        if (typeof config === 'function') {
            // Simple registration: just a factory function
            this.registrations.set(name, {
                factory: config,
                singleton: true,
                dependencies: []
            });
        } else {
            // Advanced registration: with options
            this.registrations.set(name, {
                factory: config.factory || config.implementation,
                singleton: config.singleton !== false,
                dependencies: config.dependencies || [],
                instance: config.instance // Pre-created instance
            });
        }
        return this; // Allow chaining
    }

    // Register an instance directly
    registerInstance(name, instance) {
        this.instances.set(name, instance);
        return this;
    }

    // Resolve a service
    resolve(name) {
        // Return pre-registered instance if exists
        if (this.instances.has(name)) {
            return this.instances.get(name);
        }

        // Return singleton if already created
        const registration = this.registrations.get(name);
        if (registration && registration.singleton && this.instances.has(name)) {
            return this.instances.get(name);
        }

        // Detect circular dependencies
        if (this.resolving.has(name)) {
            const cycle = Array.from(this.resolving).concat(name);
            throw new Error(`Circular dependency: ${cycle.join(' -> ')}`);
        }

        if (!registration) {
            throw new Error(`Service '${name}' is not registered`);
        }

        this.resolving.add(name);

        try {
            // Resolve dependencies
            const dependencies = registration.dependencies.map(dep => 
                this.resolve(dep)
            );

            // Create instance
            const instance = registration.factory(...dependencies);

            // Store if singleton
            if (registration.singleton) {
                this.instances.set(name, instance);
            }

            return instance;
        } finally {
            this.resolving.delete(name);
        }
    }

    // Check if service is registered
    isRegistered(name) {
        return this.registrations.has(name) || this.instances.has(name);
    }

    // Clear all instances (useful for testing)
    reset() {
        this.instances.clear();
        return this;
    }
}

// Usage example
const container = new ProductionDIContainer();

// Simple registration
container.register('config', () => ({
    apiUrl: 'https://api.example.com',
    timeout: 5000
}));

// Advanced registration with dependencies
container.register('httpClient', {
    factory: (config) => ({
        get: (url) => fetch(`${config.apiUrl}${url}`)
    }),
    dependencies: ['config'],
    singleton: true
});

container.register('userService', {
    factory: (httpClient) => ({
        getUsers: () => httpClient.get('/users')
    }),
    dependencies: ['httpClient']
});

// Use it
const userService = container.resolve('userService');
```

## Best Practices for IOC

1. **Prefer Constructor Injection**
   - Makes dependencies explicit
   - Easier to test
   - Required dependencies can't be forgotten

2. **Use Abstractions (Interfaces)**
   - Depend on contracts, not implementations
   - Makes code more flexible

3. **Keep Containers Simple**
   - Don't put business logic in containers
   - Containers should only manage object creation

4. **Avoid Service Locator Pattern**
   - Use Dependency Injection instead
   - Service Locator hides dependencies

5. **Register Dependencies at Application Startup**
   - Configure your container early
   - Makes dependencies clear and manageable

6. **Use Singleton Wisely**
   - Not everything should be a singleton
   - Use singletons for stateless services (loggers, configs)
   - Avoid singletons for stateful services

## Common Pitfalls and How to Avoid Them

### Pitfall 1: Circular Dependencies

```javascript
// BAD: Circular dependency
class ServiceA {
    constructor(serviceB) {
        this.serviceB = serviceB;
    }
}

class ServiceB {
    constructor(serviceA) {
        this.serviceA = serviceA;
    }
}

// Solution: Refactor to remove circular dependency
// Option 1: Extract common functionality
class CommonService {
    // Shared logic
}

class ServiceA {
    constructor(commonService) {
        this.common = commonService;
    }
}

class ServiceB {
    constructor(commonService) {
        this.common = commonService;
    }
}

// Option 2: Use lazy injection (inject a factory)
class ServiceA {
    constructor(serviceBFactory) {
        this.getServiceB = serviceBFactory;
    }
}
```

### Pitfall 2: Over-Engineering

```javascript
// BAD: Using DI for everything, even simple cases
class SimpleCalculator {
    constructor(mathUtils, logger, config) {
        // Too many dependencies for a simple class
    }
}

// GOOD: Keep it simple when appropriate
class SimpleCalculator {
    add(a, b) {
        return a + b; // No DI needed here
    }
}
```

### Pitfall 3: Hidden Dependencies

```javascript
// BAD: Hidden dependency via global
class UserService {
    registerUser(name) {
        // Hidden dependency on global EmailService
        EmailService.sendEmail(name, "Welcome");
    }
}

// GOOD: Explicit dependency
class UserService {
    constructor(emailService) {
        this.emailService = emailService;
    }
    
    registerUser(name) {
        this.emailService.sendEmail(name, "Welcome");
    }
}
```

## Summary

In this three-part series, we've covered:

1. **Part 1**: What IOC is and why it matters
2. **Part 2**: How to achieve IOC with Dependency Injection patterns
3. **Part 3**: Advanced topics including:
   - Sophisticated DI containers with lifecycle management
   - Service Locator pattern (and why DI is better)
   - How real frameworks implement IOC
   - Production-ready container implementation
   - Best practices and common pitfalls

**Key Takeaways:**
- IOC makes code more **testable**, **flexible**, and **maintainable**
- **Dependency Injection** is the preferred way to achieve IOC
- **Constructor Injection** is the most common and recommended pattern
- Modern frameworks have built-in IOC support
- Understanding IOC helps you write better, more professional code

Remember the golden rule: **"Don't call me, We will call you"** - let dependencies come to you, don't go and get them yourself!

Happy coding! 🚀
