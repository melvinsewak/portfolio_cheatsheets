# Angular Advanced Cheatsheet

## Advanced RxJS Patterns

### Subject Types
```typescript
import { Subject, BehaviorSubject, ReplaySubject, AsyncSubject } from 'rxjs';

export class SubjectExamples {
  // Subject - No initial value, new subscribers don't get previous values
  private messageSubject = new Subject<string>();
  message$ = this.messageSubject.asObservable();
  
  sendMessage(message: string) {
    this.messageSubject.next(message);
  }
  
  // BehaviorSubject - Has initial value, new subscribers get latest value
  private countSubject = new BehaviorSubject<number>(0);
  count$ = this.countSubject.asObservable();
  
  getCurrentCount(): number {
    return this.countSubject.value;
  }
  
  incrementCount() {
    this.countSubject.next(this.countSubject.value + 1);
  }
  
  // ReplaySubject - Replays last N values to new subscribers
  private historySubject = new ReplaySubject<string>(3);  // Replay last 3
  history$ = this.historySubject.asObservable();
  
  addToHistory(item: string) {
    this.historySubject.next(item);
  }
  
  // AsyncSubject - Only emits last value when complete
  private resultSubject = new AsyncSubject<any>();
  result$ = this.resultSubject.asObservable();
  
  completeOperation(result: any) {
    this.resultSubject.next(result);
    this.resultSubject.complete();
  }
}
```

### Advanced Operators
```typescript
import { 
  debounceTime, 
  distinctUntilChanged, 
  switchMap, 
  mergeMap,
  concatMap,
  exhaustMap,
  shareReplay,
  catchError,
  retry,
  tap,
  finalize
} from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AdvancedSearchService {
  private http = inject(HttpClient);
  
  // Debounced search with cancellation
  search(searchTerm$: Observable<string>): Observable<SearchResult[]> {
    return searchTerm$.pipe(
      debounceTime(300),           // Wait 300ms after typing stops
      distinctUntilChanged(),       // Only if value changed
      tap(term => console.log('Searching for:', term)),
      switchMap(term =>             // Cancel previous request
        this.http.get<SearchResult[]>(`/api/search?q=${term}`).pipe(
          catchError(error => {
            console.error('Search failed:', error);
            return of([]);
          })
        )
      ),
      shareReplay(1)                // Cache last result
    );
  }
  
  // Sequential operations with concatMap
  processItemsSequentially(items: Item[]): Observable<ProcessResult> {
    return from(items).pipe(
      concatMap(item => this.processItem(item)),  // Wait for each to complete
      tap(result => console.log('Processed:', result))
    );
  }
  
  // Parallel operations with mergeMap
  processItemsInParallel(items: Item[]): Observable<ProcessResult> {
    return from(items).pipe(
      mergeMap(item => this.processItem(item), 5),  // Max 5 concurrent
      tap(result => console.log('Processed:', result))
    );
  }
  
  // Ignore new requests while one is in progress (exhaustMap)
  saveData(data$: Observable<Data>): Observable<SaveResult> {
    return data$.pipe(
      exhaustMap(data =>
        this.http.post<SaveResult>('/api/save', data).pipe(
          retry(2),
          finalize(() => console.log('Save attempt finished'))
        )
      )
    );
  }
}
```

### Error Handling Strategies
```typescript
import { throwError, of, EMPTY, retry, catchError, retryWhen, delay, take } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ErrorHandlingService {
  private http = inject(HttpClient);
  
  // Strategy 1: Retry with exponential backoff
  getDataWithRetry<T>(url: string): Observable<T> {
    return this.http.get<T>(url).pipe(
      retryWhen(errors =>
        errors.pipe(
          mergeMap((error, index) => {
            const retryAttempt = index + 1;
            if (retryAttempt > 3) {
              return throwError(() => error);
            }
            console.log(`Retry attempt ${retryAttempt} after ${retryAttempt * 1000}ms`);
            return of(error).pipe(delay(retryAttempt * 1000));
          })
        )
      ),
      catchError(this.handleError)
    );
  }
  
  // Strategy 2: Fallback to cache or default
  getDataWithFallback<T>(url: string, fallback: T): Observable<T> {
    return this.http.get<T>(url).pipe(
      catchError(error => {
        console.error('Using fallback data:', error);
        return of(fallback);
      })
    );
  }
  
  // Strategy 3: Continue stream on error
  processMultipleRequests(urls: string[]): Observable<any> {
    return from(urls).pipe(
      mergeMap(url =>
        this.http.get(url).pipe(
          catchError(error => {
            console.error(`Request failed for ${url}:`, error);
            return EMPTY;  // Skip this item, continue with others
          })
        )
      )
    );
  }
  
  private handleError(error: HttpErrorResponse) {
    let errorMessage = 'An error occurred';
    
    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Client error: ${error.error.message}`;
    } else {
      // Server-side error
      errorMessage = `Server error: ${error.status} - ${error.message}`;
    }
    
    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

### Memory Management with RxJS
```typescript
import { Subject, Subscription, takeUntil, takeWhile } from 'rxjs';

// Pattern 1: takeUntil with Subject
export class ComponentWithDestroy implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  data: any[] = [];
  
  constructor(private dataService: DataService) {}
  
  ngOnInit() {
    // All subscriptions will auto-unsubscribe
    this.dataService.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => this.data = data);
    
    this.dataService.getUpdates()
      .pipe(takeUntil(this.destroy$))
      .subscribe(update => this.handleUpdate(update));
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Pattern 2: Subscription management
export class ComponentWithSubscription implements OnInit, OnDestroy {
  private subscriptions = new Subscription();
  
  ngOnInit() {
    this.subscriptions.add(
      this.service.getData().subscribe(data => this.data = data)
    );
    
    this.subscriptions.add(
      this.service.getUpdates().subscribe(update => this.handleUpdate(update))
    );
  }
  
  ngOnDestroy() {
    this.subscriptions.unsubscribe();
  }
}

// Pattern 3: takeWhile with condition
export class ConditionalComponent {
  private isActive = true;
  
  ngOnInit() {
    this.service.getStream()
      .pipe(takeWhile(() => this.isActive))
      .subscribe(data => this.handleData(data));
  }
  
  deactivate() {
    this.isActive = false;  // Stops subscription
  }
}
```

## State Management

### Service-Based State Management
```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { map } from 'rxjs/operators';

interface User {
  id: number;
  name: string;
  email: string;
}

interface AppState {
  users: User[];
  selectedUserId: number | null;
  loading: boolean;
  error: string | null;
}

@Injectable({
  providedIn: 'root'
})
export class StateService {
  private initialState: AppState = {
    users: [],
    selectedUserId: null,
    loading: false,
    error: null
  };
  
  private state$ = new BehaviorSubject<AppState>(this.initialState);
  
  // Selectors
  getState(): Observable<AppState> {
    return this.state$.asObservable();
  }
  
  getUsers(): Observable<User[]> {
    return this.state$.pipe(map(state => state.users));
  }
  
  getSelectedUser(): Observable<User | undefined> {
    return this.state$.pipe(
      map(state => state.users.find(u => u.id === state.selectedUserId))
    );
  }
  
  isLoading(): Observable<boolean> {
    return this.state$.pipe(map(state => state.loading));
  }
  
  // Actions
  setUsers(users: User[]) {
    this.updateState({ users, loading: false, error: null });
  }
  
  selectUser(userId: number) {
    this.updateState({ selectedUserId: userId });
  }
  
  setLoading(loading: boolean) {
    this.updateState({ loading });
  }
  
  setError(error: string) {
    this.updateState({ error, loading: false });
  }
  
  addUser(user: User) {
    const users = [...this.state$.value.users, user];
    this.updateState({ users });
  }
  
  updateUser(updatedUser: User) {
    const users = this.state$.value.users.map(user =>
      user.id === updatedUser.id ? updatedUser : user
    );
    this.updateState({ users });
  }
  
  deleteUser(userId: number) {
    const users = this.state$.value.users.filter(u => u.id !== userId);
    this.updateState({ users });
  }
  
  resetState() {
    this.state$.next(this.initialState);
  }
  
  private updateState(partial: Partial<AppState>) {
    this.state$.next({ ...this.state$.value, ...partial });
  }
}
```

### Using State Service in Component
```typescript
@Component({
  selector: 'app-user-management',
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <div *ngIf="error$ | async as error" class="error">{{ error }}</div>
    
    <div class="users">
      <div *ngFor="let user of users$ | async" 
           [class.selected]="user.id === (selectedUser$ | async)?.id"
           (click)="selectUser(user.id)">
        {{ user.name }} - {{ user.email }}
      </div>
    </div>
    
    <div *ngIf="selectedUser$ | async as user" class="details">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <button (click)="deleteUser(user.id)">Delete</button>
    </div>
  `
})
export class UserManagementComponent implements OnInit {
  users$ = this.stateService.getUsers();
  selectedUser$ = this.stateService.getSelectedUser();
  loading$ = this.stateService.isLoading();
  error$ = this.stateService.getState().pipe(map(state => state.error));
  
  constructor(
    private stateService: StateService,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.loadUsers();
  }
  
  loadUsers() {
    this.stateService.setLoading(true);
    this.userService.getUsers().subscribe({
      next: users => this.stateService.setUsers(users),
      error: err => this.stateService.setError(err.message)
    });
  }
  
  selectUser(userId: number) {
    this.stateService.selectUser(userId);
  }
  
  deleteUser(userId: number) {
    this.userService.deleteUser(userId).subscribe(() => {
      this.stateService.deleteUser(userId);
    });
  }
}
```

### Facade Pattern for Complex State
```typescript
@Injectable({
  providedIn: 'root'
})
export class UserFacade {
  // Observables
  users$ = this.stateService.getUsers();
  selectedUser$ = this.stateService.getSelectedUser();
  loading$ = this.stateService.isLoading();
  
  constructor(
    private stateService: StateService,
    private userService: UserService,
    private http: HttpClient
  ) {}
  
  // High-level operations
  loadUsers() {
    this.stateService.setLoading(true);
    this.userService.getUsers()
      .pipe(
        catchError(error => {
          this.stateService.setError(error.message);
          return of([]);
        })
      )
      .subscribe(users => this.stateService.setUsers(users));
  }
  
  selectUser(userId: number) {
    this.stateService.selectUser(userId);
  }
  
  createUser(user: Omit<User, 'id'>) {
    return this.userService.createUser(user).pipe(
      tap(newUser => this.stateService.addUser(newUser))
    );
  }
  
  updateUser(user: User) {
    return this.userService.updateUser(user).pipe(
      tap(updatedUser => this.stateService.updateUser(updatedUser))
    );
  }
  
  deleteUser(userId: number) {
    return this.userService.deleteUser(userId).pipe(
      tap(() => this.stateService.deleteUser(userId))
    );
  }
}
```

## Change Detection Strategies

### OnPush Change Detection
```typescript
import { Component, ChangeDetectionStrategy, Input } from '@angular/core';

// Default change detection - checks every time
@Component({
  selector: 'app-default',
  template: `<div>{{ data }}</div>`,
  changeDetection: ChangeDetectionStrategy.Default  // Default
})
export class DefaultComponent {
  @Input() data: any;
}

// OnPush - only checks when:
// 1. Input reference changes
// 2. Event is triggered in component
// 3. Async pipe receives new value
// 4. Manual change detection triggered
@Component({
  selector: 'app-optimized',
  template: `
    <div>{{ user.name }}</div>
    <button (click)="updateName()">Update</button>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent {
  @Input() user!: User;
  
  updateName() {
    // This triggers change detection (event handler)
    console.log('Name updated');
  }
}
```

### Manual Change Detection
```typescript
import { Component, ChangeDetectorRef, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-manual',
  template: `
    <div>{{ data }}</div>
    <button (click)="loadData()">Load Data</button>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ManualComponent {
  data: any;
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  loadData() {
    // Async operation
    setTimeout(() => {
      this.data = 'New data';
      // Manually trigger change detection
      this.cdr.markForCheck();  // Mark path to root as dirty
      // or
      // this.cdr.detectChanges();  // Run change detection immediately
    }, 1000);
  }
  
  ngOnDestroy() {
    // Detach component from change detection tree
    this.cdr.detach();
  }
}
```

### Immutable Data Patterns
```typescript
@Component({
  selector: 'app-list',
  template: `
    <app-item 
      *ngFor="let item of items; trackBy: trackById"
      [item]="item">
    </app-item>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ListComponent {
  items: Item[] = [];
  
  // ✅ Good - Create new reference
  addItem(item: Item) {
    this.items = [...this.items, item];
  }
  
  // ✅ Good - Create new reference with update
  updateItem(id: number, updates: Partial<Item>) {
    this.items = this.items.map(item =>
      item.id === id ? { ...item, ...updates } : item
    );
  }
  
  // ✅ Good - Create new reference for deletion
  deleteItem(id: number) {
    this.items = this.items.filter(item => item.id !== id);
  }
  
  // ❌ Bad - Mutates existing reference
  addItemBad(item: Item) {
    this.items.push(item);  // Change detection won't detect this
  }
  
  trackById(index: number, item: Item) {
    return item.id;
  }
}
```

## Advanced Routing

### Route Guards
```typescript
// Auth Guard
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isAuthenticated()) {
    return true;
  }
  
  // Redirect to login
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Role Guard
export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  const requiredRole = route.data['role'];
  
  if (authService.hasRole(requiredRole)) {
    return true;
  }
  
  return router.createUrlTree(['/unauthorized']);
};

// Can Deactivate Guard (unsaved changes)
export interface CanDeactivateComponent {
  canDeactivate: () => boolean | Observable<boolean>;
}

export const canDeactivateGuard: CanDeactivateFn<CanDeactivateComponent> = 
  (component) => {
    return component.canDeactivate ? component.canDeactivate() : true;
  };
```

### Using Guards in Routes
```typescript
const routes: Routes = [
  {
    path: 'admin',
    canActivate: [authGuard, roleGuard],
    data: { role: 'admin' },
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  },
  {
    path: 'profile',
    canActivate: [authGuard],
    component: ProfileComponent
  },
  {
    path: 'edit/:id',
    canActivate: [authGuard],
    canDeactivate: [canDeactivateGuard],
    component: EditComponent
  }
];
```

### Resolvers
```typescript
import { Injectable, inject } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot } from '@angular/router';
import { Observable } from 'rxjs';

// Class-based resolver
@Injectable({
  providedIn: 'root'
})
export class UserResolver implements Resolve<User> {
  private userService = inject(UserService);
  
  resolve(route: ActivatedRouteSnapshot): Observable<User> {
    const id = Number(route.paramMap.get('id'));
    return this.userService.getUser(id);
  }
}

// Function-based resolver (Angular 14+)
export const userResolver: ResolveFn<User> = (route, state) => {
  const userService = inject(UserService);
  const id = Number(route.params['id']);
  return userService.getUser(id);
};

// Routes with resolver
const routes: Routes = [
  {
    path: 'user/:id',
    component: UserDetailComponent,
    resolve: { user: userResolver }
  }
];

// Component accessing resolved data
@Component({
  selector: 'app-user-detail',
  template: `<div>{{ user.name }}</div>`
})
export class UserDetailComponent implements OnInit {
  user!: User;
  
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit() {
    this.user = this.route.snapshot.data['user'];
    // or reactive
    this.route.data.subscribe(data => {
      this.user = data['user'];
    });
  }
}
```

### Preloading Strategies
```typescript
// Custom preloading strategy
import { PreloadingStrategy, Route } from '@angular/router';
import { Observable, of, timer } from 'rxjs';
import { mergeMap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Preload if route data says so
    if (route.data && route.data['preload']) {
      const delay = route.data['preloadDelay'] || 0;
      return timer(delay).pipe(mergeMap(() => load()));
    }
    return of(null);
  }
}

// Routes with preloading
const routes: Routes = [
  {
    path: 'feature',
    loadChildren: () => import('./feature/feature.module'),
    data: { preload: true, preloadDelay: 2000 }
  }
];

// Provide in app config
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(
      routes,
      withPreloading(CustomPreloadingStrategy)
    )
  ]
});
```

## Dynamic Components

### Creating Components Dynamically
```typescript
import { 
  Component, 
  ViewChild, 
  ViewContainerRef, 
  ComponentRef,
  Type
} from '@angular/core';

@Component({
  selector: 'app-dynamic-container',
  template: `
    <div>
      <button (click)="loadComponent('alert')">Load Alert</button>
      <button (click)="loadComponent('form')">Load Form</button>
      <button (click)="clearComponent()">Clear</button>
    </div>
    <div #dynamicContainer></div>
  `
})
export class DynamicContainerComponent {
  @ViewChild('dynamicContainer', { read: ViewContainerRef }) 
  container!: ViewContainerRef;
  
  private componentRef: ComponentRef<any> | null = null;
  
  loadComponent(type: string) {
    this.clearComponent();
    
    const component = type === 'alert' ? AlertComponent : FormComponent;
    this.componentRef = this.container.createComponent(component);
    
    // Pass data to component
    if (this.componentRef.instance) {
      this.componentRef.instance.data = { message: 'Hello!' };
    }
    
    // Subscribe to component outputs
    if (this.componentRef.instance.onAction) {
      this.componentRef.instance.onAction.subscribe((action: any) => {
        console.log('Component action:', action);
      });
    }
  }
  
  clearComponent() {
    if (this.componentRef) {
      this.componentRef.destroy();
      this.componentRef = null;
    }
    this.container.clear();
  }
  
  ngOnDestroy() {
    this.clearComponent();
  }
}
```

### Modal Service with Dynamic Components
```typescript
@Injectable({
  providedIn: 'root'
})
export class ModalService {
  private modalContainer!: ViewContainerRef;
  
  setModalContainer(container: ViewContainerRef) {
    this.modalContainer = container;
  }
  
  open<T>(component: Type<T>, data?: any): ComponentRef<T> {
    const componentRef = this.modalContainer.createComponent(component);
    
    if (data && componentRef.instance) {
      Object.assign(componentRef.instance, data);
    }
    
    return componentRef;
  }
  
  close(componentRef: ComponentRef<any>) {
    componentRef.destroy();
  }
}

// Usage
export class AppComponent {
  @ViewChild('modalContainer', { read: ViewContainerRef })
  modalContainer!: ViewContainerRef;
  
  constructor(private modalService: ModalService) {}
  
  ngAfterViewInit() {
    this.modalService.setModalContainer(this.modalContainer);
  }
  
  openModal() {
    const modalRef = this.modalService.open(ModalComponent, {
      title: 'Confirm',
      message: 'Are you sure?'
    });
    
    // Handle modal output
    modalRef.instance.confirmed.subscribe(() => {
      console.log('Confirmed');
      this.modalService.close(modalRef);
    });
  }
}
```

## Angular Animations

### Basic Animations
```typescript
import { trigger, state, style, transition, animate } from '@angular/animations';

@Component({
  selector: 'app-animated',
  template: `
    <div [@openClose]="isOpen ? 'open' : 'closed'"
         (click)="toggle()">
      Click to toggle
    </div>
  `,
  animations: [
    trigger('openClose', [
      state('open', style({
        height: '200px',
        opacity: 1,
        backgroundColor: 'yellow'
      })),
      state('closed', style({
        height: '100px',
        opacity: 0.8,
        backgroundColor: 'blue'
      })),
      transition('open => closed', [
        animate('0.5s')
      ]),
      transition('closed => open', [
        animate('0.3s')
      ])
    ])
  ]
})
export class AnimatedComponent {
  isOpen = true;
  
  toggle() {
    this.isOpen = !this.isOpen;
  }
}
```

### Enter/Leave Animations
```typescript
import { trigger, transition, style, animate } from '@angular/animations';

@Component({
  selector: 'app-list',
  template: `
    <button (click)="addItem()">Add Item</button>
    <div *ngFor="let item of items" 
         @fadeInOut
         (click)="removeItem(item)">
      {{ item }}
    </div>
  `,
  animations: [
    trigger('fadeInOut', [
      transition(':enter', [
        style({ opacity: 0, transform: 'translateY(-20px)' }),
        animate('300ms ease-out', style({ opacity: 1, transform: 'translateY(0)' }))
      ]),
      transition(':leave', [
        animate('300ms ease-in', style({ opacity: 0, transform: 'translateY(20px)' }))
      ])
    ])
  ]
})
export class ListComponent {
  items = ['Item 1', 'Item 2', 'Item 3'];
  
  addItem() {
    this.items.push(`Item ${this.items.length + 1}`);
  }
  
  removeItem(item: string) {
    this.items = this.items.filter(i => i !== item);
  }
}
```

### Complex Animations
```typescript
import { 
  trigger, 
  transition, 
  style, 
  animate, 
  keyframes,
  query,
  stagger
} from '@angular/animations';

@Component({
  selector: 'app-advanced-animation',
  animations: [
    // Keyframe animation
    trigger('bounce', [
      transition('* => *', [
        animate('1s', keyframes([
          style({ transform: 'translateY(0)', offset: 0 }),
          style({ transform: 'translateY(-20px)', offset: 0.5 }),
          style({ transform: 'translateY(0)', offset: 1 })
        ]))
      ])
    ]),
    
    // Stagger animation for lists
    trigger('listAnimation', [
      transition('* => *', [
        query(':enter', [
          style({ opacity: 0, transform: 'translateX(-100%)' }),
          stagger('100ms', [
            animate('300ms ease-out', 
              style({ opacity: 1, transform: 'translateX(0)' })
            )
          ])
        ], { optional: true })
      ])
    ])
  ]
})
export class AdvancedAnimationComponent {}
```

## Performance Optimization

### TrackBy Function
```typescript
@Component({
  selector: 'app-list',
  template: `
    <div *ngFor="let item of items; trackBy: trackById">
      {{ item.name }}
    </div>
  `
})
export class ListComponent {
  items: Item[] = [];
  
  // Tells Angular how to identify items (prevents unnecessary re-renders)
  trackById(index: number, item: Item): number {
    return item.id;
  }
}
```

### Virtual Scrolling
```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  selector: 'app-virtual-scroll',
  standalone: true,
  imports: [ScrollingModule, CommonModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items" class="item">
        {{ item }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport {
      height: 400px;
      width: 100%;
    }
    .item {
      height: 50px;
    }
  `]
})
export class VirtualScrollComponent {
  items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);
}
```

### Lazy Loading Images
```typescript
@Directive({
  selector: 'img[appLazyLoad]',
  standalone: true
})
export class LazyLoadDirective implements OnInit {
  @Input() src!: string;
  
  constructor(private el: ElementRef<HTMLImageElement>) {}
  
  ngOnInit() {
    const observer = new IntersectionObserver(entries => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.loadImage();
          observer.unobserve(this.el.nativeElement);
        }
      });
    });
    
    observer.observe(this.el.nativeElement);
  }
  
  private loadImage() {
    this.el.nativeElement.src = this.src;
  }
}
```

### Memoization
```typescript
import { memo } from '@angular/core';

@Component({
  selector: 'app-expensive',
  template: `<div>{{ expensiveCalculation }}</div>`
})
export class ExpensiveComponent {
  @Input() data: number[] = [];
  
  // Cache expensive calculations
  private cache = new Map<string, number>();
  
  get expensiveCalculation(): number {
    const key = JSON.stringify(this.data);
    
    if (this.cache.has(key)) {
      return this.cache.get(key)!;
    }
    
    const result = this.calculateExpensiveValue(this.data);
    this.cache.set(key, result);
    return result;
  }
  
  private calculateExpensiveValue(data: number[]): number {
    // Expensive operation
    return data.reduce((sum, val) => sum + val, 0);
  }
}
```

## Testing

### Component Testing
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';
import { UserService } from './user.service';
import { of } from 'rxjs';

describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;
  let userService: jasmine.SpyObj<UserService>;
  
  beforeEach(async () => {
    const userServiceSpy = jasmine.createSpyObj('UserService', ['getUsers']);
    
    await TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [
        { provide: UserService, useValue: userServiceSpy }
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should load users on init', () => {
    const mockUsers = [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ];
    userService.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges(); // Trigger ngOnInit
    
    expect(component.users).toEqual(mockUsers);
    expect(userService.getUsers).toHaveBeenCalled();
  });
  
  it('should display user names', () => {
    const mockUsers = [{ id: 1, name: 'Alice' }];
    userService.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges();
    
    const compiled = fixture.nativeElement;
    expect(compiled.textContent).toContain('Alice');
  });
});
```

### Service Testing
```typescript
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify(); // Verify no outstanding requests
  });
  
  it('should retrieve users from API', () => {
    const mockUsers = [
      { id: 1, name: 'Alice', email: 'alice@example.com' },
      { id: 2, name: 'Bob', email: 'bob@example.com' }
    ];
    
    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
      expect(users).toEqual(mockUsers);
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
  
  it('should handle error when API fails', () => {
    service.getUsers().subscribe({
      next: () => fail('should have failed'),
      error: (error) => {
        expect(error).toBeTruthy();
      }
    });
    
    const req = httpMock.expectOne('/api/users');
    req.flush('Error', { status: 500, statusText: 'Server Error' });
  });
});
```

### E2E Testing Basics (Protractor/Cypress)
```typescript
// Cypress example
describe('User List', () => {
  beforeEach(() => {
    cy.visit('/users');
  });
  
  it('should display user list', () => {
    cy.get('.user-item').should('have.length.greaterThan', 0);
  });
  
  it('should add new user', () => {
    cy.get('[data-test="add-user-btn"]').click();
    cy.get('[data-test="name-input"]').type('New User');
    cy.get('[data-test="email-input"]').type('user@example.com');
    cy.get('[data-test="submit-btn"]').click();
    
    cy.contains('New User').should('be.visible');
  });
  
  it('should filter users by search', () => {
    cy.get('[data-test="search-input"]').type('Alice');
    cy.get('.user-item').should('have.length', 1);
    cy.get('.user-item').contains('Alice');
  });
});
```

## Do's and Don'ts

### ✅ Do's

```typescript
// ✅ DO: Use OnPush change detection for performance
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})

// ✅ DO: Use immutable patterns with OnPush
updateItems() {
  this.items = [...this.items, newItem];
}

// ✅ DO: Use proper unsubscription patterns
private destroy$ = new Subject<void>();

ngOnInit() {
  this.service.data$
    .pipe(takeUntil(this.destroy$))
    .subscribe(data => this.data = data);
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}

// ✅ DO: Use route guards for authorization
export const authGuard: CanActivateFn = () => {
  return inject(AuthService).isAuthenticated();
};

// ✅ DO: Use resolvers for data preloading
export const userResolver: ResolveFn<User> = (route) => {
  return inject(UserService).getUser(route.params['id']);
};

// ✅ DO: Use trackBy with ngFor
trackById(index: number, item: any) {
  return item.id;
}

// ✅ DO: Handle errors in observables
this.http.get('/api/data').pipe(
  retry(2),
  catchError(error => {
    this.handleError(error);
    return of([]);
  })
);

// ✅ DO: Use virtual scrolling for large lists
// ✅ DO: Lazy load images
// ✅ DO: Write unit tests for services and components
```

### ❌ Don'ts

```typescript
// ❌ DON'T: Subscribe in subscribe
// Bad
this.service1.getData().subscribe(data1 => {
  this.service2.getData(data1).subscribe(data2 => {
    // Nested subscriptions
  });
});

// ✅ Good
this.service1.getData().pipe(
  switchMap(data1 => this.service2.getData(data1))
).subscribe(data2 => {});

// ❌ DON'T: Mutate arrays/objects with OnPush
// Bad
this.items.push(newItem); // Won't trigger change detection

// ✅ Good
this.items = [...this.items, newItem];

// ❌ DON'T: Access DOM directly
// Bad
document.getElementById('myElement').innerHTML = 'text';

// ✅ Good - Use ViewChild and Renderer2
@ViewChild('myElement') element!: ElementRef;
this.renderer.setProperty(this.element.nativeElement, 'innerHTML', 'text');

// ❌ DON'T: Perform heavy computations in getters
// Bad
get expensiveValue() {
  return this.calculateExpensive(); // Called every change detection
}

// ✅ Good - Cache or use pipe
private cachedValue: number | null = null;
get expensiveValue() {
  if (this.cachedValue === null) {
    this.cachedValue = this.calculateExpensive();
  }
  return this.cachedValue;
}

// ❌ DON'T: Forget to cleanup in ngOnDestroy
// ❌ DON'T: Use any type in production code
// ❌ DON'T: Ignore TypeScript strict mode
```

### Performance Tips

✅ **Always Do:**
- Enable production mode for builds
- Use OnPush change detection
- Implement trackBy for lists
- Lazy load modules and routes
- Use virtual scrolling for long lists
- Optimize images (lazy loading, compression)
- Minimize bundle size (tree shaking)
- Use pure pipes when possible
- Profile with Angular DevTools

❌ **Never Do:**
- Perform heavy operations in change detection cycle
- Subscribe without unsubscribing
- Use large bundles without code splitting
- Ignore performance audits
- Skip testing
- Forget about accessibility

---

Congratulations! You've completed the Angular Advanced guide! 🎉

Now you have comprehensive knowledge of Angular from beginner to advanced concepts. Keep practicing and building amazing applications!
