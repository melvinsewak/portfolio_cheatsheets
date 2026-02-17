# Angular Beginner Cheatsheet

## Components Basics

### What is a Component?
A component controls a section of the screen called a view. It consists of a TypeScript class, an HTML template, and CSS styles.

```typescript
import { Component } from '@angular/core';

// Basic component
@Component({
  selector: 'app-hello',
  template: `<h1>Hello World!</h1>`,
  styles: [`h1 { color: blue; }`]
})
export class HelloComponent {}

// Component with external files
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent {
  username = 'John Doe';
  age = 25;
}
```

### Standalone Components (Angular 15+)
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-counter',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div>
      <h2>Counter: {{ count }}</h2>
      <button (click)="increment()">Increment</button>
    </div>
  `
})
export class CounterComponent {
  count = 0;
  
  increment() {
    this.count++;
  }
}
```

### Component Decorator Properties
```typescript
@Component({
  selector: 'app-example',        // CSS selector for the component
  templateUrl: './example.html',  // Path to template file
  styleUrls: ['./example.css'],   // Array of style files
  template: `<h1>Inline</h1>`,    // Inline template (alternative)
  styles: [`h1 { color: red; }`], // Inline styles (alternative)
  standalone: true,               // Standalone component (v15+)
  imports: []                     // Dependencies for standalone
})
```

## Templates and Data Binding

### Interpolation
Display component data in the template using double curly braces.

```typescript
// Component
export class ProfileComponent {
  firstName = 'Jane';
  lastName = 'Smith';
  age = 28;
  
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

```html
<!-- Template -->
<h1>{{ firstName }} {{ lastName }}</h1>
<p>Age: {{ age }}</p>
<p>Full Name: {{ getFullName() }}</p>
<p>Next Year: {{ age + 1 }}</p>
<p>Uppercase: {{ firstName.toUpperCase() }}</p>
```

### Property Binding
Bind component properties to element properties using square brackets.

```typescript
export class ImageComponent {
  imageUrl = 'assets/photo.jpg';
  imageAlt = 'Profile Photo';
  isDisabled = false;
  userEmail = 'user@example.com';
}
```

```html
<!-- Bind to attributes -->
<img [src]="imageUrl" [alt]="imageAlt">
<button [disabled]="isDisabled">Click Me</button>
<input [value]="userEmail" [type]="'email'">

<!-- Bind to classes -->
<div [class.active]="isActive">Content</div>
<div [class]="'card primary'">Card</div>

<!-- Bind to styles -->
<p [style.color]="textColor">Colored text</p>
<p [style.font-size.px]="fontSize">Sized text</p>
```

### Event Binding
Handle DOM events using parentheses.

```typescript
export class ButtonComponent {
  message = '';
  
  handleClick() {
    this.message = 'Button clicked!';
    console.log('Click handled');
  }
  
  handleInput(event: Event) {
    const target = event.target as HTMLInputElement;
    this.message = target.value;
  }
  
  handleKeyPress(event: KeyboardEvent) {
    if (event.key === 'Enter') {
      console.log('Enter pressed');
    }
  }
}
```

```html
<!-- Basic event binding -->
<button (click)="handleClick()">Click Me</button>

<!-- Event with $event object -->
<input (input)="handleInput($event)">
<input (keypress)="handleKeyPress($event)">

<!-- Multiple events -->
<button 
  (click)="handleClick()"
  (mouseenter)="onMouseEnter()"
  (mouseleave)="onMouseLeave()">
  Hover Me
</button>

<!-- Prevent default behavior -->
<form (submit)="onSubmit($event)">
  <button type="submit">Submit</button>
</form>
```

### Two-Way Data Binding
Combine property and event binding using `[(ngModel)]`.

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-form',
  standalone: true,
  imports: [FormsModule],
  template: `
    <input [(ngModel)]="username">
    <p>Hello, {{ username }}!</p>
  `
})
export class FormComponent {
  username = '';
}
```

```html
<!-- Two-way binding with ngModel -->
<input [(ngModel)]="email" type="email">
<textarea [(ngModel)]="description"></textarea>
<select [(ngModel)]="selectedOption">
  <option value="1">Option 1</option>
  <option value="2">Option 2</option>
</select>

<!-- Manual two-way binding -->
<input 
  [value]="searchTerm"
  (input)="searchTerm = $event.target.value">
```

## Directives

### Structural Directives
Change the DOM structure by adding or removing elements.

#### *ngIf - Conditional Rendering
```typescript
export class ConditionalComponent {
  isLoggedIn = true;
  userRole = 'admin';
  items: string[] = [];
}
```

```html
<!-- Basic ngIf -->
<div *ngIf="isLoggedIn">
  <p>Welcome back!</p>
</div>

<!-- ngIf with else -->
<div *ngIf="isLoggedIn; else loggedOut">
  <p>Welcome back!</p>
</div>
<ng-template #loggedOut>
  <p>Please log in.</p>
</ng-template>

<!-- ngIf with then and else -->
<div *ngIf="isLoggedIn; then loggedIn; else loggedOut"></div>
<ng-template #loggedIn>
  <p>Welcome!</p>
</ng-template>
<ng-template #loggedOut>
  <p>Please log in.</p>
</ng-template>

<!-- ngIf with condition -->
<div *ngIf="userRole === 'admin'">
  <p>Admin panel</p>
</div>

<!-- ngIf checking for empty array -->
<div *ngIf="items.length > 0; else noItems">
  <p>Items found!</p>
</div>
<ng-template #noItems>
  <p>No items available.</p>
</ng-template>
```

#### *ngFor - Loop Through Arrays
```typescript
export class ListComponent {
  users = [
    { id: 1, name: 'Alice', age: 25 },
    { id: 2, name: 'Bob', age: 30 },
    { id: 3, name: 'Charlie', age: 35 }
  ];
  
  numbers = [1, 2, 3, 4, 5];
  colors = ['red', 'blue', 'green'];
}
```

```html
<!-- Basic ngFor -->
<ul>
  <li *ngFor="let user of users">
    {{ user.name }} - {{ user.age }}
  </li>
</ul>

<!-- ngFor with index -->
<div *ngFor="let color of colors; let i = index">
  {{ i + 1 }}. {{ color }}
</div>

<!-- ngFor with trackBy for performance -->
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>

<!-- Component method -->
trackByUserId(index: number, user: any) {
  return user.id;
}

<!-- ngFor with additional variables -->
<div *ngFor="let item of items; 
              let i = index;
              let isFirst = first;
              let isLast = last;
              let isEven = even;
              let isOdd = odd">
  <p>{{ i }}: {{ item }}</p>
  <span *ngIf="isFirst">First!</span>
  <span *ngIf="isLast">Last!</span>
</div>
```

#### *ngSwitch - Multiple Conditions
```typescript
export class SwitchComponent {
  viewMode = 'list'; // 'list', 'grid', 'table'
  status = 'pending'; // 'pending', 'approved', 'rejected'
}
```

```html
<!-- ngSwitch example -->
<div [ngSwitch]="viewMode">
  <div *ngSwitchCase="'list'">
    <p>List View</p>
  </div>
  <div *ngSwitchCase="'grid'">
    <p>Grid View</p>
  </div>
  <div *ngSwitchCase="'table'">
    <p>Table View</p>
  </div>
  <div *ngSwitchDefault>
    <p>Default View</p>
  </div>
</div>

<!-- ngSwitch with status -->
<div [ngSwitch]="status">
  <p *ngSwitchCase="'pending'" class="warning">
    Pending approval
  </p>
  <p *ngSwitchCase="'approved'" class="success">
    Approved
  </p>
  <p *ngSwitchCase="'rejected'" class="error">
    Rejected
  </p>
</div>
```

### Attribute Directives
Modify the appearance or behavior of elements.

```typescript
export class StyleComponent {
  isActive = true;
  isHighlighted = false;
  currentColor = 'blue';
  fontSize = 16;
  
  currentStyles = {
    'font-size': '20px',
    'color': 'green',
    'font-weight': 'bold'
  };
  
  currentClasses = {
    'active': this.isActive,
    'highlighted': this.isHighlighted
  };
}
```

```html
<!-- ngClass - Dynamic classes -->
<div [ngClass]="'active'">Single class</div>
<div [ngClass]="['active', 'highlighted']">Multiple classes</div>
<div [ngClass]="{ 'active': isActive, 'highlighted': isHighlighted }">
  Conditional classes
</div>
<div [ngClass]="currentClasses">Object binding</div>

<!-- ngStyle - Dynamic styles -->
<div [ngStyle]="{ 'color': currentColor, 'font-size.px': fontSize }">
  Styled text
</div>
<div [ngStyle]="currentStyles">Object binding</div>

<!-- Individual class and style binding -->
<div [class.active]="isActive">Toggle active class</div>
<div [style.color]="currentColor">Colored text</div>
```

## Pipes

### Built-in Pipes
Transform data in templates without changing the component data.

```typescript
export class PipeComponent {
  text = 'hello angular';
  price = 1234.56;
  today = new Date();
  user = { name: 'John', age: 30 };
  progress = 0.75;
  largeNumber = 1234567.89;
}
```

```html
<!-- String pipes -->
<p>{{ text | uppercase }}</p>           <!-- HELLO ANGULAR -->
<p>{{ text | lowercase }}</p>           <!-- hello angular -->
<p>{{ text | titlecase }}</p>           <!-- Hello Angular -->

<!-- Number pipes -->
<p>{{ price | number }}</p>             <!-- 1,234.56 -->
<p>{{ price | number:'1.0-0' }}</p>     <!-- 1,235 -->
<p>{{ price | number:'3.1-2' }}</p>     <!-- 001,234.56 -->
<p>{{ price | currency }}</p>           <!-- $1,234.56 -->
<p>{{ price | currency:'EUR' }}</p>     <!-- €1,234.56 -->
<p>{{ progress | percent }}</p>         <!-- 75% -->

<!-- Date pipes -->
<p>{{ today | date }}</p>               <!-- Dec 15, 2023 -->
<p>{{ today | date:'short' }}</p>       <!-- 12/15/23, 3:45 PM -->
<p>{{ today | date:'fullDate' }}</p>    <!-- Friday, December 15, 2023 -->
<p>{{ today | date:'MM/dd/yyyy' }}</p>  <!-- 12/15/2023 -->
<p>{{ today | date:'h:mm a' }}</p>      <!-- 3:45 PM -->

<!-- JSON pipe (for debugging) -->
<pre>{{ user | json }}</pre>

<!-- Slice pipe -->
<p>{{ text | slice:0:5 }}</p>           <!-- hello -->

<!-- Chain multiple pipes -->
<p>{{ text | uppercase | slice:0:5 }}</p> <!-- HELLO -->
<p>{{ price | currency:'USD':'symbol':'1.0-0' }}</p> <!-- $1,235 -->
```

### Async Pipe
Automatically subscribe and unsubscribe from Observables.

```typescript
import { Component } from '@angular/core';
import { Observable, interval } from 'rxjs';
import { map } from 'rxjs/operators';

export class AsyncComponent {
  // Observable that emits every second
  time$: Observable<Date> = interval(1000).pipe(
    map(() => new Date())
  );
  
  // Promise example
  promise: Promise<string> = new Promise((resolve) => {
    setTimeout(() => resolve('Promise resolved!'), 2000);
  });
}
```

```html
<!-- Async pipe with Observable -->
<p>Current time: {{ time$ | async | date:'medium' }}</p>

<!-- Async pipe with Promise -->
<p>{{ promise | async }}</p>

<!-- Using async with ngIf -->
<div *ngIf="time$ | async as currentTime">
  {{ currentTime | date:'short' }}
</div>
```

## Component Communication

### @Input - Parent to Child
Pass data from parent to child component.

```typescript
// child.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `
    <div class="child">
      <h3>{{ title }}</h3>
      <p>{{ message }}</p>
      <p>Count: {{ count }}</p>
    </div>
  `
})
export class ChildComponent {
  @Input() title = 'Default Title';
  @Input() message = '';
  @Input() count = 0;
  
  // Input with alias
  @Input('userName') name = '';
  
  // Required input (Angular 16+)
  @Input({ required: true }) userId!: number;
}

// parent.component.ts
@Component({
  selector: 'app-parent',
  template: `
    <app-child 
      [title]="parentTitle"
      [message]="parentMessage"
      [count]="counter">
    </app-child>
  `
})
export class ParentComponent {
  parentTitle = 'Hello from Parent';
  parentMessage = 'This is a message';
  counter = 42;
}
```

### @Output - Child to Parent
Emit events from child to parent component.

```typescript
// child.component.ts
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `
    <button (click)="sendMessage()">Send to Parent</button>
    <button (click)="increment()">Increment</button>
  `
})
export class ChildComponent {
  @Output() messageEvent = new EventEmitter<string>();
  @Output() countChange = new EventEmitter<number>();
  
  count = 0;
  
  sendMessage() {
    this.messageEvent.emit('Hello from child!');
  }
  
  increment() {
    this.count++;
    this.countChange.emit(this.count);
  }
}

// parent.component.ts
@Component({
  selector: 'app-parent',
  template: `
    <app-child 
      (messageEvent)="receiveMessage($event)"
      (countChange)="onCountChange($event)">
    </app-child>
    <p>{{ receivedMessage }}</p>
    <p>Count: {{ childCount }}</p>
  `
})
export class ParentComponent {
  receivedMessage = '';
  childCount = 0;
  
  receiveMessage(message: string) {
    this.receivedMessage = message;
  }
  
  onCountChange(count: number) {
    this.childCount = count;
  }
}
```

### Two-Way Binding Between Components
```typescript
// child.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <button (click)="decrement()">-</button>
    <span>{{ count }}</span>
    <button (click)="increment()">+</button>
  `
})
export class CounterComponent {
  @Input() count = 0;
  @Output() countChange = new EventEmitter<number>();
  
  increment() {
    this.count++;
    this.countChange.emit(this.count);
  }
  
  decrement() {
    this.count--;
    this.countChange.emit(this.count);
  }
}

// parent.component.ts
@Component({
  selector: 'app-parent',
  template: `
    <app-counter [(count)]="myCount"></app-counter>
    <p>Parent count: {{ myCount }}</p>
  `
})
export class ParentComponent {
  myCount = 10;
}
```

## Basic Services and Dependency Injection

### Creating a Service
Services provide shared logic and data across components.

```typescript
// user.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // Available app-wide
})
export class UserService {
  private users = [
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' }
  ];
  
  getUsers() {
    return this.users;
  }
  
  getUserById(id: number) {
    return this.users.find(user => user.id === id);
  }
  
  addUser(user: any) {
    this.users.push(user);
  }
  
  deleteUser(id: number) {
    this.users = this.users.filter(user => user.id !== id);
  }
}
```

### Injecting a Service (Constructor Injection)
```typescript
import { Component } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `
    <ul>
      <li *ngFor="let user of users">
        {{ user.name }} - {{ user.email }}
      </li>
    </ul>
  `
})
export class UserListComponent {
  users: any[] = [];
  
  constructor(private userService: UserService) {
    this.users = this.userService.getUsers();
  }
}
```

### Injecting a Service (inject() Function - Angular 14+)
```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `
    <ul>
      <li *ngFor="let user of users">
        {{ user.name }}
      </li>
    </ul>
  `
})
export class UserListComponent {
  private userService = inject(UserService);
  users = this.userService.getUsers();
}
```

### Service with Methods
```typescript
// logger.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class LoggerService {
  log(message: string) {
    console.log(`LOG: ${message}`);
  }
  
  error(message: string) {
    console.error(`ERROR: ${message}`);
  }
  
  warn(message: string) {
    console.warn(`WARN: ${message}`);
  }
}

// Using the service
@Component({
  selector: 'app-example',
  template: `<button (click)="doSomething()">Click</button>`
})
export class ExampleComponent {
  constructor(private logger: LoggerService) {}
  
  doSomething() {
    this.logger.log('Button clicked');
    try {
      // Some operation
    } catch (error) {
      this.logger.error('Operation failed');
    }
  }
}
```

## Angular CLI Commands

### Project Commands
```bash
# Create new Angular project
ng new my-app
ng new my-app --routing --style=scss

# Serve application (default: http://localhost:4200)
ng serve
ng serve --open          # Opens browser automatically
ng serve --port 4300     # Custom port

# Build application
ng build
ng build --configuration production
ng build --prod          # Production build (optimized)

# Run tests
ng test                  # Unit tests
ng e2e                   # End-to-end tests

# Run linter
ng lint
```

### Generate Commands
```bash
# Generate component
ng generate component my-component
ng g c my-component
ng g c my-component --skip-tests        # Skip test file
ng g c my-component --inline-style      # Inline styles
ng g c my-component --inline-template   # Inline template
ng g c components/my-component          # In subdirectory

# Generate service
ng generate service services/my-service
ng g s services/my-service
ng g s services/my-service --skip-tests

# Generate module
ng generate module my-module
ng g m my-module
ng g m my-module --routing              # With routing

# Generate directive
ng generate directive my-directive
ng g d my-directive

# Generate pipe
ng generate pipe my-pipe
ng g p my-pipe

# Generate guard
ng generate guard guards/auth
ng g g guards/auth

# Generate interface
ng generate interface models/user
ng g i models/user

# Generate enum
ng generate enum models/status
ng g e models/status

# Generate class
ng generate class models/product
ng g cl models/product
```

### Update and Add Commands
```bash
# Update Angular and dependencies
ng update
ng update @angular/core @angular/cli

# Add packages
ng add @angular/material        # Angular Material
ng add @angular/pwa            # Progressive Web App
ng add @ngrx/store             # NgRx state management
```

## Do's and Don'ts

### ✅ Do's

#### Component Best Practices
```typescript
// ✅ DO: Use meaningful component names
@Component({
  selector: 'app-user-profile',  // Good
})

// ✅ DO: Keep components focused and small
@Component({
  selector: 'app-user-card',
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  `
})
export class UserCardComponent {
  @Input() user!: User;  // Single responsibility
}

// ✅ DO: Use OnInit for initialization
export class MyComponent implements OnInit {
  data: any[] = [];
  
  constructor(private service: DataService) {
    // Keep constructor simple - only for DI
  }
  
  ngOnInit() {
    // Initialization logic here
    this.data = this.service.getData();
  }
}

// ✅ DO: Use strong typing
export class UserComponent {
  user: User;              // Typed
  users: User[] = [];      // Array type
  age: number = 25;        // Primitive type
}

// ✅ DO: Use trackBy with ngFor for performance
trackByUserId(index: number, user: User) {
  return user.id;
}
```

```html
<!-- ✅ DO: Use trackBy with large lists -->
<div *ngFor="let item of items; trackBy: trackById">
  {{ item.name }}
</div>

<!-- ✅ DO: Use async pipe to auto-unsubscribe -->
<p>{{ data$ | async }}</p>

<!-- ✅ DO: Use safe navigation operator -->
<p>{{ user?.address?.city }}</p>
```

#### Service Best Practices
```typescript
// ✅ DO: Use providedIn: 'root' for singleton services
@Injectable({
  providedIn: 'root'
})
export class UserService {}

// ✅ DO: Keep business logic in services
@Injectable({
  providedIn: 'root'
})
export class CalculatorService {
  calculate(a: number, b: number): number {
    return a + b;
  }
}

// ✅ DO: Use readonly for properties that shouldn't change
export class ConfigService {
  private readonly apiUrl = 'https://api.example.com';
}
```

### ❌ Don'ts

#### Component Anti-Patterns
```typescript
// ❌ DON'T: Use any type unnecessarily
export class BadComponent {
  data: any;  // Bad - loses type safety
}

// ❌ DON'T: Manipulate DOM directly
export class BadComponent {
  ngAfterViewInit() {
    // Bad - use Angular's template binding instead
    document.getElementById('myDiv').innerHTML = 'text';
  }
}

// ❌ DON'T: Put business logic in components
export class BadComponent {
  calculateTotal() {
    // Bad - this should be in a service
    let total = 0;
    // Complex business logic here
    return total;
  }
}

// ❌ DON'T: Forget to initialize required inputs
export class BadComponent {
  @Input() user: User;  // Bad - might be undefined
  // Should be: @Input() user!: User; or provide default
}

// ❌ DON'T: Use complex logic in templates
```

```html
<!-- ❌ DON'T: Complex expressions in templates -->
<p>{{ users.filter(u => u.age > 18).map(u => u.name).join(', ') }}</p>
<!-- Bad - do this in component instead -->

<!-- ❌ DON'T: Call methods in templates unnecessarily -->
<div *ngFor="let item of getItems()">  <!-- Bad - called on every change detection -->
  {{ item }}
</div>

<!-- ❌ DON'T: Forget else template with ngIf when needed -->
<div *ngIf="condition">Yes</div>
<div *ngIf="!condition">No</div>  <!-- Bad - use else instead -->
```

#### Service Anti-Patterns
```typescript
// ❌ DON'T: Store component-specific logic in services
@Injectable()
export class BadService {
  updateComponentView() {  // Bad - too specific to component
    // View logic here
  }
}

// ❌ DON'T: Create services without @Injectable
export class BadService {  // Bad - missing @Injectable()
  getData() {}
}
```

### Summary Tips

✅ **Always Do:**
- Use Angular CLI for code generation
- Follow naming conventions (kebab-case for files)
- Implement OnInit for initialization logic
- Use type annotations
- Unsubscribe from observables (or use async pipe)
- Keep components simple and focused
- Put business logic in services
- Use trackBy with ngFor for lists

❌ **Never Do:**
- Manipulate DOM directly
- Use `any` type everywhere
- Put complex logic in templates
- Call methods unnecessarily in templates
- Forget to unsubscribe from subscriptions
- Mix business logic with presentation logic
- Ignore TypeScript errors
- Skip error handling

---

Ready for more? Continue to [Intermediate.md](Intermediate.md)! 🚀
