# Angular Intermediate Cheatsheet

## Forms

### Template-Driven Forms
Forms driven by directives in the template.

#### Basic Setup
```typescript
// app.module.ts or component (standalone)
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [FormsModule, CommonModule],
  templateUrl: './user-form.component.html'
})
export class UserFormComponent {
  user = {
    name: '',
    email: '',
    age: null
  };
  
  onSubmit() {
    console.log('Form submitted:', this.user);
  }
}
```

```html
<!-- user-form.component.html -->
<form #userForm="ngForm" (ngSubmit)="onSubmit()">
  <div>
    <label for="name">Name:</label>
    <input 
      type="text" 
      id="name"
      name="name"
      [(ngModel)]="user.name"
      required
      #nameInput="ngModel">
    <div *ngIf="nameInput.invalid && nameInput.touched">
      Name is required
    </div>
  </div>
  
  <div>
    <label for="email">Email:</label>
    <input 
      type="email" 
      id="email"
      name="email"
      [(ngModel)]="user.email"
      required
      email
      #emailInput="ngModel">
    <div *ngIf="emailInput.invalid && emailInput.touched">
      <span *ngIf="emailInput.errors?.['required']">Email is required</span>
      <span *ngIf="emailInput.errors?.['email']">Invalid email format</span>
    </div>
  </div>
  
  <button type="submit" [disabled]="userForm.invalid">Submit</button>
</form>

<!-- Form state information -->
<p>Form Valid: {{ userForm.valid }}</p>
<p>Form Submitted: {{ userForm.submitted }}</p>
```

### Reactive Forms
Forms built programmatically with more control and testability.

#### Basic Setup
```typescript
import { Component, OnInit } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-registration',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  templateUrl: './registration.component.html'
})
export class RegistrationComponent implements OnInit {
  registrationForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.registrationForm = this.fb.group({
      username: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8)]],
      age: [null, [Validators.required, Validators.min(18), Validators.max(100)]]
    });
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log(this.registrationForm.value);
    } else {
      this.registrationForm.markAllAsTouched();
    }
  }
  
  // Getter for easy access in template
  get username() {
    return this.registrationForm.get('username');
  }
  
  get email() {
    return this.registrationForm.get('email');
  }
}
```

```html
<!-- registration.component.html -->
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
  <div>
    <label>Username:</label>
    <input type="text" formControlName="username">
    <div *ngIf="username?.invalid && username?.touched">
      <span *ngIf="username?.errors?.['required']">Username is required</span>
      <span *ngIf="username?.errors?.['minlength']">
        Username must be at least 3 characters
      </span>
    </div>
  </div>
  
  <div>
    <label>Email:</label>
    <input type="email" formControlName="email">
    <div *ngIf="email?.invalid && email?.touched">
      <span *ngIf="email?.errors?.['required']">Email is required</span>
      <span *ngIf="email?.errors?.['email']">Invalid email format</span>
    </div>
  </div>
  
  <div>
    <label>Password:</label>
    <input type="password" formControlName="password">
  </div>
  
  <div>
    <label>Age:</label>
    <input type="number" formControlName="age">
  </div>
  
  <button type="submit" [disabled]="registrationForm.invalid">
    Register
  </button>
</form>
```

### FormArray - Dynamic Form Controls
```typescript
import { FormArray, FormBuilder, FormGroup } from '@angular/forms';

@Component({
  selector: 'app-skills-form',
  template: `
    <form [formGroup]="skillsForm" (ngSubmit)="onSubmit()">
      <div formArrayName="skills">
        <div *ngFor="let skill of skills.controls; let i = index">
          <input [formControlName]="i" placeholder="Skill {{ i + 1 }}">
          <button type="button" (click)="removeSkill(i)">Remove</button>
        </div>
      </div>
      <button type="button" (click)="addSkill()">Add Skill</button>
      <button type="submit">Submit</button>
    </form>
  `
})
export class SkillsFormComponent {
  skillsForm: FormGroup;
  
  constructor(private fb: FormBuilder) {
    this.skillsForm = this.fb.group({
      skills: this.fb.array([
        this.fb.control('')
      ])
    });
  }
  
  get skills() {
    return this.skillsForm.get('skills') as FormArray;
  }
  
  addSkill() {
    this.skills.push(this.fb.control(''));
  }
  
  removeSkill(index: number) {
    this.skills.removeAt(index);
  }
  
  onSubmit() {
    console.log(this.skillsForm.value);
  }
}
```

### Custom Validators
```typescript
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Custom validator function
export function passwordMatchValidator(control: AbstractControl): ValidationErrors | null {
  const password = control.get('password');
  const confirmPassword = control.get('confirmPassword');
  
  if (!password || !confirmPassword) {
    return null;
  }
  
  return password.value === confirmPassword.value ? null : { passwordMismatch: true };
}

// Usage in component
export class SignupComponent {
  signupForm = this.fb.group({
    password: ['', Validators.required],
    confirmPassword: ['', Validators.required]
  }, { validators: passwordMatchValidator });
  
  constructor(private fb: FormBuilder) {}
}

// Reusable validator factory
export function minArrayLength(min: number): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    if (control.value && control.value.length >= min) {
      return null;
    }
    return { minArrayLength: { required: min, actual: control.value?.length } };
  };
}
```

## Routing and Navigation

### Basic Router Setup
```typescript
// app.routes.ts (Angular 15+ standalone)
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { ContactComponent } from './contact/contact.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent },
  { path: '**', redirectTo: '' }  // Wildcard route
];

// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes)
  ]
});
```

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  template: `
    <nav>
      <a routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">
        Home
      </a>
      <a routerLink="/about" routerLinkActive="active">About</a>
      <a routerLink="/contact" routerLinkActive="active">Contact</a>
    </nav>
    <router-outlet></router-outlet>
  `
})
export class AppComponent {}
```

### Route Parameters
```typescript
// routes
const routes: Routes = [
  { path: 'user/:id', component: UserDetailComponent },
  { path: 'product/:id/review/:reviewId', component: ReviewComponent }
];

// user-detail.component.ts
import { Component, OnInit, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user-detail',
  template: `
    <h2>User ID: {{ userId }}</h2>
    <p>User data here...</p>
  `
})
export class UserDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);
  userId: string | null = null;
  
  ngOnInit() {
    // Snapshot (for single read)
    this.userId = this.route.snapshot.paramMap.get('id');
    
    // Observable (for reactive updates)
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
      this.loadUserData(this.userId);
    });
  }
  
  loadUserData(id: string | null) {
    // Load user data based on ID
  }
}
```

### Query Parameters
```typescript
// Navigate with query params
import { Router } from '@angular/router';

export class SearchComponent {
  private router = inject(Router);
  
  search(term: string) {
    this.router.navigate(['/results'], {
      queryParams: { q: term, page: 1 }
    });
  }
}

// Read query params
import { ActivatedRoute } from '@angular/router';

export class ResultsComponent implements OnInit {
  private route = inject(ActivatedRoute);
  searchTerm = '';
  page = 1;
  
  ngOnInit() {
    this.route.queryParamMap.subscribe(params => {
      this.searchTerm = params.get('q') || '';
      this.page = Number(params.get('page')) || 1;
    });
  }
}
```

```html
<!-- Navigate with query params in template -->
<a [routerLink]="['/results']" [queryParams]="{q: 'angular', page: 1}">
  Search Angular
</a>
```

### Programmatic Navigation
```typescript
import { Router } from '@angular/router';

export class NavigationComponent {
  private router = inject(Router);
  
  goToUser(userId: number) {
    this.router.navigate(['/user', userId]);
  }
  
  goToSearch(query: string) {
    this.router.navigate(['/search'], {
      queryParams: { q: query }
    });
  }
  
  goBack() {
    this.router.navigate(['..'], { relativeTo: this.route });
  }
  
  replaceUrl() {
    this.router.navigate(['/home'], { replaceUrl: true });
  }
}
```

### Child Routes
```typescript
const routes: Routes = [
  {
    path: 'products',
    component: ProductsComponent,
    children: [
      { path: '', component: ProductListComponent },
      { path: ':id', component: ProductDetailComponent },
      { path: ':id/edit', component: ProductEditComponent }
    ]
  }
];
```

```html
<!-- products.component.html -->
<div class="products-container">
  <h1>Products</h1>
  <router-outlet></router-outlet>  <!-- Child components render here -->
</div>
```

## HTTP Client and Observables

### HTTP Client Setup
```typescript
// app.config.ts (Angular 15+ standalone)
import { provideHttpClient } from '@angular/common/http';

export const appConfig = {
  providers: [
    provideHttpClient()
  ]
};

// Or in main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient()
  ]
});
```

### Basic HTTP Requests
```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

interface User {
  id: number;
  name: string;
  email: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private http = inject(HttpClient);
  private apiUrl = 'https://api.example.com/users';
  
  // GET request
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
  
  // GET by ID
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
  
  // POST request
  createUser(user: User): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
  
  // PUT request
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }
  
  // PATCH request
  patchUser(id: number, updates: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, updates);
  }
  
  // DELETE request
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

### Using HTTP in Components
```typescript
import { Component, OnInit, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `
    <div *ngIf="loading">Loading...</div>
    <div *ngIf="error">{{ error }}</div>
    <ul *ngIf="!loading && !error">
      <li *ngFor="let user of users">
        {{ user.name }} - {{ user.email }}
      </li>
    </ul>
  `
})
export class UserListComponent implements OnInit {
  private userService = inject(UserService);
  users: User[] = [];
  loading = false;
  error = '';
  
  ngOnInit() {
    this.loadUsers();
  }
  
  loadUsers() {
    this.loading = true;
    this.userService.getUsers().subscribe({
      next: (users) => {
        this.users = users;
        this.loading = false;
      },
      error: (err) => {
        this.error = 'Failed to load users';
        this.loading = false;
        console.error(err);
      }
    });
  }
}
```

### HTTP Headers and Options
```typescript
import { HttpHeaders, HttpParams } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private http = inject(HttpClient);
  
  // With custom headers
  getUsersWithAuth(): Observable<User[]> {
    const headers = new HttpHeaders({
      'Authorization': 'Bearer ' + this.getToken(),
      'Content-Type': 'application/json'
    });
    
    return this.http.get<User[]>(`${this.apiUrl}/users`, { headers });
  }
  
  // With query parameters
  searchUsers(searchTerm: string, page: number): Observable<User[]> {
    const params = new HttpParams()
      .set('q', searchTerm)
      .set('page', page.toString())
      .set('limit', '10');
    
    return this.http.get<User[]>(`${this.apiUrl}/users/search`, { params });
  }
  
  // With full response
  getUserWithFullResponse(id: number) {
    return this.http.get<User>(`${this.apiUrl}/users/${id}`, {
      observe: 'response'
    });
  }
  
  private getToken(): string {
    return localStorage.getItem('token') || '';
  }
}
```

## RxJS Operators

### Basic Operators
```typescript
import { map, filter, tap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  private http = inject(HttpClient);
  
  // map - Transform data
  getUserNames(): Observable<string[]> {
    return this.http.get<User[]>('/api/users').pipe(
      map(users => users.map(user => user.name))
    );
  }
  
  // filter - Filter items
  getActiveUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      filter(users => users.every(user => user.active))
    );
  }
  
  // tap - Side effects (logging, debugging)
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => console.log('Users loaded:', users)),
      tap(users => this.cacheUsers(users))
    );
  }
}
```

### Error Handling Operators
```typescript
import { catchError, retry, throwError } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private http = inject(HttpClient);
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      retry(3),  // Retry failed request 3 times
      catchError(error => {
        console.error('Error loading users:', error);
        return throwError(() => new Error('Failed to load users'));
      })
    );
  }
  
  // Return default value on error
  getUsersWithDefault(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      catchError(() => of([]))  // Return empty array on error
    );
  }
}
```

### Combination Operators
```typescript
import { forkJoin, combineLatest, merge } from 'rxjs';
import { switchMap, mergeMap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  private http = inject(HttpClient);
  
  // forkJoin - Wait for all to complete
  loadDashboardData(): Observable<any> {
    return forkJoin({
      users: this.http.get<User[]>('/api/users'),
      posts: this.http.get<Post[]>('/api/posts'),
      comments: this.http.get<Comment[]>('/api/comments')
    });
  }
  
  // switchMap - Cancel previous, switch to new
  searchUsers(searchTerm$: Observable<string>): Observable<User[]> {
    return searchTerm$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => this.http.get<User[]>(`/api/users?q=${term}`))
    );
  }
  
  // mergeMap - Run in parallel
  getUsersWithPosts(): Observable<UserWithPosts[]> {
    return this.http.get<User[]>('/api/users').pipe(
      mergeMap(users => 
        forkJoin(
          users.map(user => 
            this.http.get<Post[]>(`/api/users/${user.id}/posts`).pipe(
              map(posts => ({ ...user, posts }))
            )
          )
        )
      )
    );
  }
}
```

### Common RxJS Patterns
```typescript
import { Subject, BehaviorSubject, debounceTime, distinctUntilChanged } from 'rxjs';

export class SearchComponent implements OnInit, OnDestroy {
  private searchSubject = new Subject<string>();
  private destroy$ = new Subject<void>();
  results: any[] = [];
  
  constructor(private searchService: SearchService) {}
  
  ngOnInit() {
    // Debounced search
    this.searchSubject.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => this.searchService.search(term)),
      takeUntil(this.destroy$)
    ).subscribe(results => {
      this.results = results;
    });
  }
  
  onSearchInput(term: string) {
    this.searchSubject.next(term);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

## Lifecycle Hooks

### All Lifecycle Hooks
```typescript
import { 
  Component, 
  OnInit, 
  OnChanges, 
  DoCheck,
  AfterContentInit,
  AfterContentChecked,
  AfterViewInit,
  AfterViewChecked,
  OnDestroy,
  SimpleChanges,
  Input
} from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  template: `<p>{{ data }}</p>`
})
export class LifecycleDemoComponent implements 
  OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked, OnDestroy {
  
  @Input() data: string = '';
  
  // 1. Called when input properties change
  ngOnChanges(changes: SimpleChanges) {
    console.log('ngOnChanges', changes);
  }
  
  // 2. Called once after first ngOnChanges
  ngOnInit() {
    console.log('ngOnInit - Component initialized');
    // Initialize component, fetch data
  }
  
  // 3. Called on every change detection run
  ngDoCheck() {
    console.log('ngDoCheck - Change detection');
  }
  
  // 4. Called after content projection initialized
  ngAfterContentInit() {
    console.log('ngAfterContentInit');
  }
  
  // 5. Called after every content check
  ngAfterContentChecked() {
    console.log('ngAfterContentChecked');
  }
  
  // 6. Called after component view initialized
  ngAfterViewInit() {
    console.log('ngAfterViewInit');
    // Safe to access ViewChild here
  }
  
  // 7. Called after every view check
  ngAfterViewChecked() {
    console.log('ngAfterViewChecked');
  }
  
  // 8. Called before component is destroyed
  ngOnDestroy() {
    console.log('ngOnDestroy - Cleanup');
    // Cleanup: unsubscribe, detach event handlers
  }
}
```

### Practical Lifecycle Examples
```typescript
// OnInit - Data fetching
export class UserComponent implements OnInit {
  user: User | null = null;
  
  ngOnInit() {
    this.userService.getUser(1).subscribe(user => {
      this.user = user;
    });
  }
}

// OnChanges - React to input changes
export class ChildComponent implements OnChanges {
  @Input() userId: number = 0;
  
  ngOnChanges(changes: SimpleChanges) {
    if (changes['userId'] && !changes['userId'].firstChange) {
      this.loadUserData(changes['userId'].currentValue);
    }
  }
}

// OnDestroy - Cleanup
export class SubscriptionComponent implements OnInit, OnDestroy {
  private subscription = new Subscription();
  
  ngOnInit() {
    this.subscription.add(
      this.dataService.getData().subscribe(data => {
        // Handle data
      })
    );
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

## Custom Directives

### Attribute Directive
```typescript
import { Directive, ElementRef, HostListener, Input, Renderer2 } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  @Input() appHighlight = 'yellow';
  @Input() defaultColor = 'transparent';
  
  constructor(
    private el: ElementRef,
    private renderer: Renderer2
  ) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.appHighlight);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(this.defaultColor);
  }
  
  private highlight(color: string) {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
  }
}
```

```html
<!-- Usage -->
<p appHighlight="lightblue">Hover over me!</p>
<p [appHighlight]="'pink'" defaultColor="lightgray">Custom colors</p>
```

### Structural Directive
```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]',
  standalone: true
})
export class UnlessDirective {
  private hasView = false;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
  
  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
```

```html
<!-- Usage (opposite of ngIf) -->
<p *appUnless="isHidden">This is shown when isHidden is false</p>
```

## Custom Pipes

### Basic Custom Pipe
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'exponential',
  standalone: true
})
export class ExponentialPipe implements PipeTransform {
  transform(value: number, exponent: number = 1): number {
    return Math.pow(value, exponent);
  }
}
```

```html
<!-- Usage -->
<p>{{ 2 | exponential:3 }}</p>  <!-- Output: 8 -->
```

### Advanced Custom Pipe
```typescript
@Pipe({
  name: 'filter',
  standalone: true,
  pure: false  // Impure pipe (use carefully)
})
export class FilterPipe implements PipeTransform {
  transform(items: any[], searchText: string, property: string): any[] {
    if (!items || !searchText) {
      return items;
    }
    
    return items.filter(item => 
      item[property].toLowerCase().includes(searchText.toLowerCase())
    );
  }
}
```

```html
<!-- Usage -->
<div *ngFor="let user of users | filter:searchTerm:'name'">
  {{ user.name }}
</div>
```

## Modules and Lazy Loading

### Feature Module
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Routes } from '@angular/router';
import { ProductListComponent } from './product-list/product-list.component';
import { ProductDetailComponent } from './product-detail/product-detail.component';

const routes: Routes = [
  { path: '', component: ProductListComponent },
  { path: ':id', component: ProductDetailComponent }
];

@NgModule({
  declarations: [
    ProductListComponent,
    ProductDetailComponent
  ],
  imports: [
    CommonModule,
    RouterModule.forChild(routes)
  ]
})
export class ProductsModule {}
```

### Lazy Loading
```typescript
// app.routes.ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  {
    path: 'products',
    loadChildren: () => import('./products/products.module')
      .then(m => m.ProductsModule)
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes')
      .then(m => m.ADMIN_ROUTES)
  }
];
```

### Lazy Loading with Standalone Components
```typescript
// app.routes.ts
export const routes: Routes = [
  { path: '', component: HomeComponent },
  {
    path: 'products',
    loadComponent: () => import('./products/product-list.component')
      .then(m => m.ProductListComponent)
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes')
  }
];
```

## Do's and Don'ts

### ✅ Do's

```typescript
// ✅ DO: Unsubscribe from observables
export class MyComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.service.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => this.data = data);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// ✅ DO: Use async pipe (auto-unsubscribe)
export class MyComponent {
  users$ = this.userService.getUsers();
}
```

```html
<!-- Template -->
<div *ngFor="let user of users$ | async">
  {{ user.name }}
</div>
```

```typescript
// ✅ DO: Use reactive forms for complex forms
// ✅ DO: Handle HTTP errors properly
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users').pipe(
    retry(2),
    catchError(this.handleError)
  );
}

// ✅ DO: Use lazy loading for feature modules
// ✅ DO: Use trackBy with ngFor
trackByUserId(index: number, user: User) {
  return user.id;
}
```

### ❌ Don'ts

```typescript
// ❌ DON'T: Subscribe in subscribe (pyramid of doom)
// Bad
this.service1.getData().subscribe(data1 => {
  this.service2.getData(data1).subscribe(data2 => {
    this.service3.getData(data2).subscribe(data3 => {
      // Use data3
    });
  });
});

// Good - Use RxJS operators
this.service1.getData().pipe(
  switchMap(data1 => this.service2.getData(data1)),
  switchMap(data2 => this.service3.getData(data2))
).subscribe(data3 => {
  // Use data3
});

// ❌ DON'T: Forget to unsubscribe
// Bad
ngOnInit() {
  this.service.getData().subscribe(data => {
    this.data = data;
  });
  // No unsubscribe - memory leak!
}

// ❌ DON'T: Use any type with HTTP requests
// Bad
getData(): Observable<any> { }

// Good
getData(): Observable<User[]> { }

// ❌ DON'T: Mutate form values directly
// Bad
this.form.value.name = 'New Name';

// Good
this.form.patchValue({ name: 'New Name' });
```

### Summary Tips

✅ **Always Do:**
- Use reactive forms for complex scenarios
- Implement proper error handling for HTTP requests
- Unsubscribe from observables (or use async pipe)
- Use lazy loading for large applications
- Validate forms properly (both client and server)
- Use RxJS operators instead of nested subscribes
- Type your observables and HTTP responses
- Use proper HTTP interceptors for auth and errors

❌ **Never Do:**
- Forget to unsubscribe from manual subscriptions
- Nest subscribe calls (use operators instead)
- Ignore HTTP error handling
- Use impure pipes without understanding performance impact
- Mutate form values directly
- Create memory leaks with lifecycle hooks
- Skip lazy loading for large feature modules

---

Ready for advanced topics? Continue to [Advanced.md](Advanced.md)! 🚀
