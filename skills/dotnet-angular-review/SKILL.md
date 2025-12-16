---
name: .NET and Angular Code Review Best Practices
description: |
  Use this skill when reviewing .NET 8/10 or Angular code for bugs, security issues, and best practice violations. Triggers on: "review .NET code", "check Angular component", "code review best practices", "clean architecture review", "SOLID principles check", "async await issues", "RxJS review", "security audit", or when analyzing code changes in .NET solutions or Angular projects.
version: 1.0.0
---

Apply these best practices when reviewing .NET and Angular code. Focus on issues that matter - bugs, security vulnerabilities, and significant design problems. Avoid nitpicks and style preferences.

## .NET Best Practices

### Clean Architecture

**Layer Dependencies (Strict):**
- Domain: No dependencies (pure business logic)
- Application: Depends only on Domain
- Infrastructure: Depends on Application and Domain
- Presentation/API: Depends on Application (not Infrastructure directly)

**Violations to flag:**
- Domain layer importing Infrastructure namespaces
- Application layer directly instantiating Infrastructure classes
- Circular dependencies between layers
- Business logic in controllers or Infrastructure

### SOLID Principles

**Single Responsibility:**
- Classes doing multiple unrelated things
- Methods longer than 30-50 lines
- Services mixing business logic with I/O operations

**Open/Closed:**
- Switch statements on type that could use polymorphism
- Modifying existing classes instead of extending

**Liskov Substitution:**
- Derived classes throwing `NotImplementedException`
- Overrides that don't honor base class contracts

**Interface Segregation:**
- Large interfaces forcing implementers to stub methods
- Classes depending on interfaces they don't fully use

**Dependency Inversion:**
- High-level modules instantiating low-level modules with `new`
- Depending on concrete classes instead of abstractions

### Async/Await Patterns

**Critical Issues (Always flag):**
```csharp
// BAD: Blocking on async
var result = GetDataAsync().Result;  // Deadlock risk
var result = GetDataAsync().Wait();  // Deadlock risk

// BAD: async void (except event handlers)
public async void ProcessData() { }  // Exceptions lost

// BAD: Missing await
public async Task ProcessAsync() {
    GetDataAsync();  // Fire and forget - bugs!
}
```

**Correct Patterns:**
```csharp
// GOOD: Proper async/await
public async Task<Result> ProcessAsync() {
    var data = await GetDataAsync();
    return await TransformAsync(data);
}

// GOOD: ConfigureAwait in library code
var data = await GetDataAsync().ConfigureAwait(false);
```

### Exception Handling

**Bad Patterns:**
```csharp
// BAD: Swallowing exceptions
catch (Exception) { }

// BAD: Catching and only logging
catch (Exception ex) { _logger.Log(ex); }  // Should re-throw or handle

// BAD: Hardcoded error messages (TOM project rule)
throw new UserFriendlyException("License not found");
```

**Correct Patterns:**
```csharp
// GOOD: Using translation keys (TOM project)
throw new UserFriendlyException(ApplicationMessages.Err_LicenseNotFound);

// GOOD: Proper exception handling
catch (SpecificException ex) {
    _logger.LogError(ex, "Context message");
    throw new DomainException("Meaningful message", ex);
}
```

### Entity Framework

**N+1 Query Detection:**
```csharp
// BAD: N+1 queries
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders) {
    var items = order.Items;  // Lazy load per iteration!
}

// GOOD: Eager loading
var orders = await _context.Orders
    .Include(o => o.Items)
    .ToListAsync();
```

**Read-Only Queries:**
```csharp
// BAD: Tracking when not needed
var users = await _context.Users.ToListAsync();

// GOOD: No tracking for read-only
var users = await _context.Users.AsNoTracking().ToListAsync();
```

### Security Checklist

- [ ] No hardcoded connection strings or secrets
- [ ] Input validation on all public endpoints
- [ ] Parameterized queries (no string concatenation for SQL)
- [ ] Authorization attributes on controllers/actions
- [ ] No sensitive data in logs
- [ ] HTTPS enforced
- [ ] CORS properly configured

## Angular Best Practices

### Component Design

**Change Detection:**
```typescript
// GOOD: OnPush for performance
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

**TrackBy for ngFor:**
```html
<!-- BAD: No trackBy -->
<li *ngFor="let item of items">{{item.name}}</li>

<!-- GOOD: With trackBy -->
<li *ngFor="let item of items; trackBy: trackById">{{item.name}}</li>
```

### RxJS Memory Leaks

**Critical - Always flag unsubscribed subscriptions:**

```typescript
// BAD: Memory leak
ngOnInit() {
  this.dataService.getData().subscribe(data => {
    this.data = data;
  });
}

// GOOD: Using takeUntilDestroyed (Angular 16+)
private destroyRef = inject(DestroyRef);

ngOnInit() {
  this.dataService.getData()
    .pipe(takeUntilDestroyed(this.destroyRef))
    .subscribe(data => this.data = data);
}

// GOOD: Using async pipe (preferred)
// In component:
data$ = this.dataService.getData();

// In template:
<div *ngIf="data$ | async as data">{{data.name}}</div>
```

**Nested Subscriptions:**
```typescript
// BAD: Nested subscriptions
this.user$.subscribe(user => {
  this.orderService.getOrders(user.id).subscribe(orders => {
    this.orders = orders;
  });
});

// GOOD: Using switchMap
this.user$.pipe(
  switchMap(user => this.orderService.getOrders(user.id)),
  takeUntilDestroyed(this.destroyRef)
).subscribe(orders => this.orders = orders);
```

### Security

**Template Security:**
```html
<!-- BAD: Bypassing sanitization without reason -->
<div [innerHTML]="userContent | safeHtml"></div>

<!-- Review carefully any use of: -->
<!-- bypassSecurityTrustHtml -->
<!-- bypassSecurityTrustScript -->
<!-- bypassSecurityTrustUrl -->
```

**Storage Security:**
```typescript
// BAD: Sensitive data in localStorage
localStorage.setItem('authToken', token);
localStorage.setItem('userPassword', password);  // NEVER do this

// GOOD: Use HttpOnly cookies for tokens (server-side)
// Or use memory-only storage for sensitive data
```

### Performance

**Lazy Loading:**
```typescript
// GOOD: Lazy loaded routes
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
];
```

**Barrel File Imports:**
```typescript
// BAD: Imports entire barrel (large bundle)
import { OneComponent } from '@shared';

// GOOD: Direct imports
import { OneComponent } from '@shared/components/one/one.component';
```

## Confidence Scoring Guidelines

**Score 80-100 (Report these):**
- Missing `await` on async calls
- `async void` methods
- Unsubscribed observables
- SQL injection vulnerabilities
- Hardcoded secrets
- N+1 queries
- Memory leaks
- Authorization bypass

**Score 50-79 (Usually filter out):**
- Missing `AsNoTracking()` on simple queries
- Missing `OnPush` change detection
- Could use `async` pipe but subscription is properly cleaned up
- Minor SOLID violations

**Score 0-49 (Always filter out):**
- Style preferences
- Could be slightly more efficient
- Missing optional optimizations
- Pre-existing issues

## TOM Project Specific Rules

When reviewing TOM project code:

1. **UserFriendlyException**: Must use `ApplicationMessages` constants, never hardcoded strings
2. **Microservice patterns**: Follow accounting API patterns for consistency
3. **Proto-shared interfaces**: Check that implementations match interface definitions
4. **Translation keys**: All user-facing messages must be translatable
