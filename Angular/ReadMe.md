# Angular Cheatsheet

Welcome to the Angular cheatsheet! This guide will help you quickly get up to speed with Angular development.

## Overview

Angular is a comprehensive TypeScript-based web application framework developed and maintained by Google. It provides a complete solution for building scalable single-page applications with a powerful set of tools including routing, forms, HTTP client, and more. Angular uses a component-based architecture (often compared to MVVM-style patterns) and emphasizes testability, maintainability, and reusability.

## Contents

- [Beginner.md](Beginner.md) - Components, templates, data binding, directives, and services
- [Intermediate.md](Intermediate.md) - Forms, routing, HTTP, RxJS, lifecycle hooks, and modules
- [Advanced.md](Advanced.md) - Advanced patterns, state management, change detection, performance, and testing

## Quick Reference

### Basic Component
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-welcome',
  template: `<h1>Hello, {{ name }}!</h1>`,
  styles: [`h1 { color: blue; }`]
})
export class WelcomeComponent {
  name = 'Angular';
}
```

### Component with Service
```typescript
import { Component, inject } from '@angular/core';
import { DataService } from './data.service';

@Component({
  selector: 'app-user-list',
  template: `
    <ul>
      <li *ngFor="let user of users">{{ user.name }}</li>
    </ul>
  `
})
export class UserListComponent {
  private dataService = inject(DataService);
  users = this.dataService.getUsers();
}
```

### Key Features

- **Component-Based Architecture**: Build encapsulated components with their own template, styles, and logic
- **TypeScript**: Strongly typed language with enhanced IDE support and error detection
- **Dependency Injection**: Powerful DI system for managing service dependencies
- **Two-Way Data Binding**: Synchronize data between model and view automatically
- **RxJS Integration**: Built-in reactive programming with Observables
- **Angular CLI**: Powerful command-line interface for scaffolding and development
- **Complete Framework**: Includes routing, forms, HTTP client, animations, and more

## Getting Started

### Install Angular CLI
```bash
npm install -g @angular/cli
```

### Create New Application
```bash
ng new my-app
cd my-app
ng serve
```

### Navigate to Application
```
http://localhost:4200
```

## Project Structure
```
my-app/
├── src/
│   ├── app/
│   │   ├── components/
│   │   ├── services/
│   │   ├── models/
│   │   ├── app.component.ts
│   │   ├── app.component.html
│   │   ├── app.component.css
│   │   └── app.module.ts
│   ├── assets/
│   ├── environments/
│   └── index.html
├── angular.json
├── package.json
└── tsconfig.json
```

## Essential CLI Commands

```bash
# Generate component
ng generate component my-component
ng g c my-component

# Generate service
ng generate service my-service
ng g s my-service

# Generate module
ng generate module my-module
ng g m my-module

# Build for production
ng build --configuration production

# Run tests
ng test

# Run linter
ng lint
```

## Angular Versions

This cheatsheet focuses on **Angular v15+** which includes:
- Standalone components (module-free)
- Improved dependency injection with `inject()`
- Better TypeScript support
- Enhanced performance
- Simplified APIs

## Learning Path

1. **Start with [Beginner.md](Beginner.md)** - Learn the fundamentals of components, templates, and data binding
2. **Progress to [Intermediate.md](Intermediate.md)** - Master forms, routing, HTTP, and RxJS
3. **Advance to [Advanced.md](Advanced.md)** - Explore advanced patterns, optimization, and testing

## Additional Resources

### Official Documentation
- [Angular Official Docs](https://angular.io/docs)
- [Angular CLI](https://angular.io/cli)
- [Angular Style Guide](https://angular.io/guide/styleguide)

### Tools & Libraries
- [Angular Material](https://material.angular.io/) - Material Design components
- [NgRx](https://ngrx.io/) - State management
- [RxJS](https://rxjs.dev/) - Reactive programming

### Community
- [Angular Blog](https://blog.angular.io/)
- [Angular on GitHub](https://github.com/angular/angular)
- [Angular Discord](https://discord.gg/angular)

## Quick Tips

✅ **Do:**
- Use Angular CLI for consistent code generation
- Follow the official style guide
- Embrace TypeScript's type system
- Use standalone components for new projects (v15+)
- Implement proper error handling
- Write unit tests for components and services

❌ **Don't:**
- Manipulate DOM directly (use Angular's template syntax)
- Ignore TypeScript errors or use `any` everywhere
- Overuse two-way binding for complex forms
- Forget to unsubscribe from Observables
- Skip learning RxJS fundamentals
- Neglect accessibility features

---

Happy coding with Angular! 🅰️
