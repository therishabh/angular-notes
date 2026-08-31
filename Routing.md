
# Angular Routing — Complete Guide

---

# 1. Core Concept

Angular Router is responsible for deciding:

> **"Based on the current URL, which component(s) should be displayed?"**

Angular routing is primarily built around three things:

```text
URL
 ↓
Route Configuration
 ↓
Router
 ↓
RouterOutlet
 ↓
Component
```

For example:

```text
/user/123
```

matches:

```ts
{
  path: 'user/:id',
  component: UserComponent
}
```

and Angular renders `UserComponent` inside:

```html
<router-outlet />
```

Angular Router is the official routing library for Angular SPAs. ([Angular][1])

---

# 2. Basic Routing

## Route Configuration

Modern standalone Angular commonly uses:

```ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  },
  {
    path: 'users',
    component: UsersComponent
  },
  {
    path: 'about',
    component: AboutComponent
  }
];
```

And provide the router:

```ts
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes)
  ]
});
```

Then your root template needs:

```html
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/users">Users</a>
  <a routerLink="/about">About</a>
</nav>

<router-outlet />
```

### Important pieces

| Concept          | Purpose                                    |
| ---------------- | ------------------------------------------ |
| `Routes`         | Defines routes                             |
| `Route`          | Defines one route                          |
| `Router`         | Programmatic navigation                    |
| `RouterLink`     | Template navigation                        |
| `RouterOutlet`   | Placeholder where routed component renders |
| `ActivatedRoute` | Access current route information           |

---

# 3. How Routing Actually Works

Suppose user clicks:

```html
<a routerLink="/users">Users</a>
```

Flow:

```text
User clicks link
       ↓
RouterLink
       ↓
Angular Router
       ↓
Match URL against Routes
       ↓
Run guards
       ↓
Run resolvers if configured
       ↓
Activate route
       ↓
Create component
       ↓
Render inside router-outlet
       ↓
NavigationEnd
```

This lifecycle becomes very important for senior-level interviews.

Angular exposes router events such as:

```text
NavigationStart
RoutesRecognized
GuardsCheckStart
GuardsCheckEnd
ResolveStart
ResolveEnd
ActivationStart
ActivationEnd
NavigationEnd
```

and failure-related events such as `NavigationCancel` and `NavigationError`. ([Angular][2])

---

# 4. `routerLink` — Navigation from Template

Basic:

```html
<a routerLink="/users">Users</a>
```

You can also use an array:

```html
<a [routerLink]="['/users', user.id]">
  View User
</a>
```

For:

```text
/users/123
```

---

## Relative Navigation

Suppose current URL:

```text
/users/123
```

You want:

```text
/users/123/orders
```

You can use:

```html
<a [routerLink]="['orders']">
  Orders
</a>
```

Relative routing becomes especially important with nested routes.

---

# 5. Programmatic Navigation

Inject `Router`:

```ts
private readonly router = inject(Router);
```

Then:

```ts
this.router.navigate(['/users']);
```

With parameters:

```ts
this.router.navigate(['/users', user.id]);
```

With query parameters:

```ts
this.router.navigate(['/users'], {
  queryParams: {
    page: 2,
    sort: 'name'
  }
});
```

Result:

```text
/users?page=2&sort=name
```

### `navigate()` vs `navigateByUrl()`

```ts
router.navigate(['/users', 123]);
```

works with URL segments.

```ts
router.navigateByUrl('/users/123');
```

works directly with a URL.

In normal application code, `navigate()` is often more convenient when constructing URLs from segments/parameters.

---

# 6. Route Parameters

Suppose:

```ts
{
  path: 'users/:id',
  component: UserDetailComponent
}
```

URL:

```text
/users/123
```

Here:

```text
id = 123
```

---

## Reading Route Parameter

Modern Angular:

```ts
private readonly route = inject(ActivatedRoute);

ngOnInit() {
  const id = this.route.snapshot.paramMap.get('id');
}
```

### `snapshot`

Useful when you only care about the value at that moment.

```ts
const id = this.route.snapshot.paramMap.get('id');
```

---

## Reactive Parameter

If the same component can remain active while the parameter changes:

```ts
this.route.paramMap.subscribe(params => {
  const id = params.get('id');
});
```

Or with signals:

```ts
readonly id = toSignal(
  this.route.paramMap.pipe(
    map(params => params.get('id'))
  )
);
```

The important concept:

> **Don't assume `ngOnInit()` will run again every time only a route parameter changes.**

The router can reuse the same component instance, so reacting to route parameter changes is important when the component remains active.

---

# 7. Query Parameters

Route:

```text
/products?page=2&sort=price
```

Query parameters are different from path parameters.

```text
/products/123
         ↑
    path parameter
```

versus:

```text
/products?page=2
         ↑
   query parameter
```

Read:

```ts
const page = this.route.snapshot.queryParamMap.get('page');
```

Reactive:

```ts
this.route.queryParamMap.subscribe(params => {
  const page = params.get('page');
});
```

### Typical use cases

Query parameters are ideal for:

* Pagination
* Search
* Sorting
* Filtering
* Tabs
* Optional UI state

Example:

```text
/products?page=3&sort=price&category=mobile
```

---

# 8. Path Params vs Query Params

|                   | Path Parameter | Query Parameter  |
| ----------------- | -------------- | ---------------- |
| Example           | `/users/123`   | `/users?page=2`  |
| Required identity | Usually        | Usually optional |
| Common use        | Resource ID    | Filters/options  |
| Defined in route  | `:id`          | Not required     |
| Access            | `paramMap`     | `queryParamMap`  |

A good rule:

> **Path params identify the resource; query params usually modify how the resource is displayed/fetched.**

---

# 9. Nested Routing

This is extremely important.

Suppose your application has:

```text
/dashboard
/dashboard/profile
/dashboard/orders
/dashboard/settings
```

You can define:

```ts
{
  path: 'dashboard',
  component: DashboardComponent,
  children: [
    {
      path: 'profile',
      component: ProfileComponent
    },
    {
      path: 'orders',
      component: OrdersComponent
    },
    {
      path: 'settings',
      component: SettingsComponent
    }
  ]
}
```

But there's an important detail.

`DashboardComponent` must have its **own** outlet:

```html
<!-- dashboard.component.html -->

<h1>Dashboard</h1>

<nav>
  <a routerLink="profile">Profile</a>
  <a routerLink="orders">Orders</a>
  <a routerLink="settings">Settings</a>
</nav>

<router-outlet />
```

The rendering becomes:

```text
AppComponent
    │
    └── DashboardComponent
            │
            └── router-outlet
                    │
                    ├── ProfileComponent
                    ├── OrdersComponent
                    └── SettingsComponent
```

This is the core concept behind nested routing. Angular's `children` configuration defines child routes. ([Angular][3])

---

# 10. Why Nested Routing?

Imagine a dashboard layout:

```text
┌─────────────────────────────┐
│ Header                      │
├──────────┬──────────────────┤
│ Sidebar  │ Content          │
│          │                  │
│ Profile  │ OrdersComponent  │
│ Orders   │                  │
│ Settings │                  │
└──────────┴──────────────────┘
```

You don't want the entire dashboard layout recreated for every page.

Instead:

```text
DashboardComponent
 ├── Header
 ├── Sidebar
 └── router-outlet
       └── active child
```

This is a very common real-world pattern.

---

# 11. Default Child Route

Suppose:

```text
/dashboard
```

should automatically show:

```text
/dashboard/overview
```

Use:

```ts
{
  path: 'dashboard',
  component: DashboardComponent,
  children: [
    {
      path: '',
      redirectTo: 'overview',
      pathMatch: 'full'
    },
    {
      path: 'overview',
      component: OverviewComponent
    }
  ]
}
```

### Why `pathMatch: 'full'`?

Because:

```ts
path: ''
```

with the default:

```ts
pathMatch: 'prefix'
```

can match too broadly.

For empty-path redirects, use:

```ts
pathMatch: 'full'
```

Angular specifically warns about this for root/empty-path redirects. ([Angular][4])

---

# 12. Lazy Loading

Lazy loading means:

> **Don't download a route's JavaScript until the user actually needs that route.**

Without lazy loading:

```text
Initial application
 ↓
Download
 ├── Home
 ├── Login
 ├── Admin
 ├── Reports
 ├── Settings
 └── huge feature modules
```

With lazy loading:

```text
Initial load
 ↓
Home + essential code

User opens Admin
 ↓
Download Admin chunk
```

This can significantly reduce initial JavaScript and improve initial loading performance. ([Angular][5])

---

# 13. Lazy Loading a Component

Modern Angular:

```ts
{
  path: 'login',
  loadComponent: () =>
    import('./login/login.component')
      .then(m => m.LoginComponent)
}
```

Angular uses the JavaScript `import()` mechanism to create a separate chunk that can be loaded when the route becomes active. ([Angular][5])

---

# 14. Lazy Loading a Route Tree

For a feature with multiple pages:

```text
/admin
/admin/users
/admin/reports
/admin/settings
```

Create:

```ts
// admin.routes.ts

export const adminRoutes: Routes = [
  {
    path: '',
    component: AdminComponent,
    children: [
      {
        path: 'users',
        loadComponent: () =>
          import('./users/users.component')
            .then(m => m.UsersComponent)
      },
      {
        path: 'reports',
        loadComponent: () =>
          import('./reports/reports.component')
            .then(m => m.ReportsComponent)
      }
    ]
  }
];
```

Then:

```ts
{
  path: 'admin',
  loadChildren: () =>
    import('./admin/admin.routes')
      .then(m => m.adminRoutes)
}
```

Conceptually:

```text
Main bundle
    │
    ├── Home
    ├── Login
    │
    └── Admin route config
             ↓
       lazy loaded
             ↓
       Admin chunk
             ↓
       Users / Reports
```

`loadChildren` lazy-loads child route configurations, while `loadComponent` lazy-loads an individual component. ([Angular][5])

---

# 15. `loadComponent` vs `loadChildren`

|          | `loadComponent` | `loadChildren`      |
| -------- | --------------- | ------------------- |
| Loads    | Component       | Route configuration |
| Good for | Single page     | Feature/route tree  |
| Lazy?    | Yes             | Yes                 |
| Example  | Login           | Admin feature       |

Think:

```text
One page
→ loadComponent

Whole feature
→ loadChildren
```

---

# 16. Eager vs Lazy Loading

Don't blindly lazy-load everything.

A reasonable strategy:

```text
Frequently visited landing page
→ Eager

Large/infrequently visited feature
→ Lazy
```

Angular's current guidance generally recommends eager loading for primary landing pages and lazy loading other pages, depending on application needs. Excessive nested lazy loading can introduce additional network requests. ([Angular][5])

---

# 17. Preloading

Lazy loading has a downside:

```text
User clicks Reports
       ↓
Browser downloads Reports chunk
       ↓
Reports opens
```

There can be a delay.

Preloading solves this by loading lazy routes **in the background after initial navigation**.

Angular provides:

```text
NoPreloading
PreloadAllModules
```

`NoPreloading` is the default. `PreloadAllModules` loads lazy route configurations after initial navigation. ([Angular][6])

Conceptually:

```text
Initial page
    ↓
Application loaded
    ↓
Background
    ↓
Preload likely-needed features
```

### Interview question

> Why use lazy loading + preloading?

**Answer:**

> Lazy loading reduces initial bundle size, while preloading can reduce the delay when the user later navigates to a lazy feature.

---

# 18. Route Guards

Guards control whether navigation is allowed.

Typical use cases:

```text
Authentication
Authorization
Unsaved form
Feature flags
Conditional routes
```

Angular currently provides:

```text
CanActivate
CanActivateChild
CanDeactivate
CanMatch
```

([Angular][7])

---

# 19. `CanActivate` — Protect a Route

Example:

```ts
export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService);

  return authService.isLoggedIn();
};
```

Route:

```ts
{
  path: 'dashboard',
  component: DashboardComponent,
  canActivate: [authGuard]
}
```

Flow:

```text
/user/dashboard
       ↓
authGuard
       ↓
Logged in?
   /       \
 yes       no
 ↓          ↓
Dashboard   Login
```

---

# 20. Don't Return `false` + Navigate

Avoid:

```ts
if (!loggedIn) {
  router.navigate(['/login']);
  return false;
}
```

Prefer returning a redirect:

```ts
return router.parseUrl('/login');
```

or:

```ts
return new RedirectCommand(
  router.createUrlTree(['/login'])
);
```

Angular recommends returning a `UrlTree` or `RedirectCommand` when redirecting rather than returning `false` and separately navigating. ([Angular][7])

---

# 21. `CanActivateChild`

Suppose:

```text
/admin
/admin/users
/admin/reports
/admin/settings
```

and every child requires admin access.

Instead of:

```ts
users → guard
reports → guard
settings → guard
```

you can guard the children:

```ts
{
  path: 'admin',
  component: AdminComponent,
  canActivateChild: [adminGuard],

  children: [
    {
      path: 'users',
      component: UsersComponent
    },
    {
      path: 'reports',
      component: ReportsComponent
    }
  ]
}
```

Useful for protecting an entire route subtree.

---

# 22. `CanDeactivate` — Unsaved Changes

Very common real-world scenario.

User is editing:

```text
/edit-profile
```

They type:

```text
New phone number
```

Then click:

```text
Dashboard
```

You want:

```text
"You have unsaved changes. Leave?"
```

A `CanDeactivate` guard is designed for this.

Conceptually:

```ts
export const unsavedChangesGuard: CanDeactivateFn<EditComponent> =
  (component) => {
    return component.hasUnsavedChanges()
      ? confirm('Discard changes?')
      : true;
  };
```

Route:

```ts
{
  path: 'edit',
  component: EditComponent,
  canDeactivate: [unsavedChangesGuard]
}
```

---

# 23. `CanMatch` — Important Modern Guard

`CanMatch` controls whether a route is considered a match in the first place.

This is useful for:

* Feature flags
* A/B testing
* Conditional route loading
* Different components for the same URL

Example:

```ts
{
  path: 'dashboard',
  component: AdminDashboardComponent,
  canMatch: [adminGuard]
},
{
  path: 'dashboard',
  component: UserDashboardComponent,
  canMatch: [userGuard]
}
```

Angular tries the first route. If `canMatch` returns `false`, the router can try another matching route. This is a key difference from `CanActivate`. ([Angular][7])

---

# 24. `CanMatch` vs `CanActivate`

This is a **very good interview question**.

|                            | CanMatch                                | CanActivate         |
| -------------------------- | --------------------------------------- | ------------------- |
| Runs                       | During route matching                   | During activation   |
| Can prevent route matching | Yes                                     | No                  |
| `false` behavior           | Router can try another route            | Navigation blocked  |
| Useful for                 | Feature flags, conditional lazy loading | Auth/access control |

Simple mental model:

```text
CanMatch
"Should this route match?"

CanActivate
"Now that it matched, can I enter?"
```

---

# 25. Important Security Rule

A route guard is **not a security boundary**.

For example:

```ts
canActivate: [adminGuard]
```

doesn't secure your backend API.

A malicious user can modify browser JavaScript.

Real security must be enforced server-side:

```text
Angular Guard
     ↓
UX / navigation protection

Backend Authorization
     ↓
Actual security
```

Angular explicitly warns that client-side guards must not be the sole access-control mechanism. ([Angular][7])

This is a **must-remember interview point**.

---

# 26. 404 / Page Not Found

Create:

```ts
{
  path: '**',
  component: NotFoundComponent
}
```

Example complete routes:

```ts
export const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  },
  {
    path: 'users',
    component: UsersComponent
  },
  {
    path: 'about',
    component: AboutComponent
  },
  {
    path: '**',
    component: NotFoundComponent
  }
];
```

`**` means:

> Match anything that hasn't matched an earlier route.

The wildcard route should normally be **last** because Angular uses a first-match-wins strategy. ([Angular][8])

---

# 27. Redirects

Example:

```ts
{
  path: '',
  redirectTo: 'home',
  pathMatch: 'full'
}
```

Another:

```ts
{
  path: 'old-dashboard',
  redirectTo: 'dashboard'
}
```

Useful for:

* Default routes
* Legacy URLs
* Renamed pages
* Migration

---

# 28. Route Order Matters

Angular uses **first-match wins**.

Consider:

```ts
{
  path: 'users/:id',
  component: UserComponent
},
{
  path: 'users/new',
  component: NewUserComponent
}
```

Visit:

```text
/users/new
```

The first route:

```text
users/:id
```

can interpret:

```text
id = "new"
```

So `NewUserComponent` may never be reached.

Correct:

```ts
{
  path: 'users/new',
  component: NewUserComponent
},
{
  path: 'users/:id',
  component: UserComponent
}
```

General rule:

```text
Specific routes
      ↓
Dynamic routes
      ↓
Wildcard
```

Angular documents this first-match-wins behavior explicitly. ([Angular][8])

---

# 29. Route Data

You can attach static metadata to routes:

```ts
{
  path: 'admin',
  component: AdminComponent,
  data: {
    title: 'Administration',
    requiredRole: 'admin'
  }
}
```

Then access it through `ActivatedRoute`.

This is useful for:

* Breadcrumbs
* Analytics
* Authorization metadata
* UI configuration
* Titles
* Feature metadata

Angular routes support custom `data`. ([Angular][3])

---

# 30. Route Titles

Modern Angular allows:

```ts
{
  path: 'products',
  component: ProductsComponent,
  title: 'Products'
}
```

Angular automatically updates the document title when the route activates. ([Angular][8])

This is better than manually doing:

```ts
document.title = 'Products';
```

for normal route-based titles.

---

# 31. Route Resolver

Sometimes you don't want the component to render until required data is available.

Example:

```text
/users/123
```

Before displaying the page:

```text
Router
 ↓
Fetch user 123
 ↓
Data available
 ↓
Activate UserComponent
```

That's where **route resolvers** come in.

Example:

```ts
export const userResolver: ResolveFn<User> = (route) => {
  const userService = inject(UserService);

  const id = route.paramMap.get('id')!;

  return userService.getUser(id);
};
```

Route:

```ts
{
  path: 'users/:id',
  component: UserComponent,
  resolve: {
    user: userResolver
  }
}
```

Then the component can access resolved data through route state.

Resolvers are part of the router's activation flow, after guards. ([Angular][2])

---

# 32. Guard vs Resolver

Very common interview question.

|         | Guard                     | Resolver                               |
| ------- | ------------------------- | -------------------------------------- |
| Purpose | Should navigation happen? | What data is needed before activation? |
| Example | User authenticated?       | Load user details                      |
| Returns | Permission/redirect       | Data                                   |
| Runs    | Before activation         | After guards                           |

Mental model:

```text
URL
 ↓
CanMatch?
 ↓
CanActivate?
 ↓
Resolver
 ↓
Component
```

---

# 33. Route Lifecycle

A useful simplified lifecycle:

```text
NavigationStart
      ↓
Route recognition
      ↓
CanMatch
      ↓
Guards
      ↓
Resolve
      ↓
Route activation
      ↓
Component rendered
      ↓
NavigationEnd
```

For lazy-loaded routes, configuration loading also happens during navigation.

Angular exposes events for these stages, including `RouteConfigLoadStart/End`, `GuardsCheckStart/End`, `ResolveStart/End`, and `ActivationStart/End`. ([Angular][2])

---

# 34. Router Events

You can listen to navigation events:

```ts
private readonly router = inject(Router);

constructor() {
  this.router.events.subscribe(event => {
    console.log(event);
  });
}
```

Useful for:

* Global loading indicator
* Analytics
* Navigation logging
* Debugging
* Performance monitoring

Example:

```text
NavigationStart
    ↓
show spinner

NavigationEnd
    ↓
hide spinner
```

Don't forget to handle canceled/error navigation appropriately in real applications.

---

# 35. Named / Auxiliary Routes

This is an advanced routing feature.

Normally:

```html
<router-outlet />
```

is the primary outlet.

You can have another:

```html
<router-outlet name="chat" />
```

Route:

```ts
{
  path: 'chat',
  component: ChatComponent,
  outlet: 'chat'
}
```

This allows multiple independently routed UI areas.

Conceptually:

```text
URL
 ├── primary route
 │      ↓
 │   Main content
 │
 └── chat route
        ↓
     Chat panel
```

Angular calls these **named outlets / auxiliary routes**. ([Angular][3])

This is useful for things like:

* Chat drawer
* Modal routes
* Side panels

You don't need to use this in every application, but it's worth knowing for senior interviews.

---

# 36. Route-Level Providers

Modern Angular also allows providers directly on a route:

```ts
{
  path: 'admin',
  providers: [
    AdminService
  ],
  loadChildren: () =>
    import('./admin/admin.routes')
      .then(m => m.adminRoutes)
}
```

Angular creates a route-level `EnvironmentInjector` for the route and its children. ([Angular][3])

This is useful when a dependency should be scoped to a feature rather than the entire application.

---

# 37. Lazy Loading + Guards

A common architecture:

```ts
{
  path: 'admin',
  canMatch: [adminGuard],
  loadChildren: () =>
    import('./admin/admin.routes')
      .then(m => m.adminRoutes)
}
```

This is particularly useful because `CanMatch` participates in route matching and can be used to conditionally load route configurations. ([Angular][7])

Conceptually:

```text
User → /admin
       ↓
CanMatch
       ↓
Admin?
 ┌─────┴─────┐
No          Yes
 ↓            ↓
Don't       Load
match       admin chunk
              ↓
          Admin routes
```

---

# 38. A Real-World Route Structure

For a reasonably large application:

```text
/
├── login
├── register
│
├── dashboard
│   ├── overview
│   ├── profile
│   └── notifications
│
├── products
│   ├── list
│   └── :id
│
├── admin
│   ├── users
│   ├── reports
│   └── settings
│
└── **
```

Possible route configuration:

```ts
export const routes: Routes = [
  {
    path: '',
    redirectTo: 'dashboard',
    pathMatch: 'full'
  },

  {
    path: 'login',
    loadComponent: () =>
      import('./auth/login.component')
        .then(m => m.LoginComponent)
  },

  {
    path: 'dashboard',
    canActivate: [authGuard],
    loadChildren: () =>
      import('./dashboard/dashboard.routes')
        .then(m => m.dashboardRoutes)
  },

  {
    path: 'products',
    loadChildren: () =>
      import('./products/product.routes')
        .then(m => m.productRoutes)
  },

  {
    path: 'admin',
    canMatch: [adminGuard],
    loadChildren: () =>
      import('./admin/admin.routes')
        .then(m => m.adminRoutes)
  },

  {
    path: '**',
    component: NotFoundComponent
  }
];
```

This gives you:

```text
                App
                 │
          ┌──────┴──────┐
          │             │
       Eager         Lazy routes
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Dashboard     Products      Admin
          │            │            │
       Guard         Routes       Guard
```

---

# 39. Common Mistakes / Gotchas

### ❌ Putting `**` before real routes

Bad:

```ts
{
  path: '**',
  component: NotFoundComponent
},
{
  path: 'users',
  component: UsersComponent
}
```

`users` will never be reached.

---

### ❌ Forgetting `pathMatch: 'full'`

Especially:

```ts
{
  path: '',
  redirectTo: 'dashboard'
}
```

Use:

```ts
{
  path: '',
  redirectTo: 'dashboard',
  pathMatch: 'full'
}
```

---

### ❌ Putting dynamic route before static route

Bad:

```ts
users/:id
users/new
```

Better:

```ts
users/new
users/:id
```

---

### ❌ Forgetting `<router-outlet>`

Route configuration alone doesn't tell Angular **where** to render the component.

You need:

```html
<router-outlet />
```

For nested routing, the parent component needs its own outlet.

---

### ❌ Using guards as backend security

Wrong assumption:

```text
Angular guard = secure API
```

Correct:

```text
Angular guard
→ navigation/UX

Backend authorization
→ actual security
```

---

### ❌ Lazy-loading everything

Lazy loading reduces initial bundle size but creates additional network requests later.

Overusing nested lazy loading can hurt navigation performance. ([Angular][5])

---

### ❌ Using `effect()` for routing state unnecessarily

If route state is simply being transformed into derived UI state, prefer Angular's reactive APIs rather than creating unnecessary side effects.

---

# 40. Important Routing Performance Concepts

For interviews, remember these:

### 1. Lazy loading

Reduces initial JS.

```text
Initial bundle ↓
Initial load ↓
```

### 2. Preloading

Reduces delay when navigating later.

```text
Lazy chunk
    ↓
background download
    ↓
instant-ish later navigation
```

### 3. Avoid excessive lazy nesting

```text
Lazy
 ↓
Lazy
 ↓
Lazy
 ↓
Lazy
```

can cause multiple sequential network requests.

### 4. Route guards shouldn't perform unnecessarily expensive work

If authentication state is already available locally, don't make unnecessary API requests on every navigation.

### 5. Resolvers can delay navigation

Resolvers are useful, but if you resolve huge/slow data before every route activation, navigation can feel blocked.

---

# 41. Modern Angular vs Older Angular Routing

You'll see both styles in interviews and existing projects.

| Modern Angular                    | Older Angular                                           |
| --------------------------------- | ------------------------------------------------------- |
| `provideRouter(routes)`           | `RouterModule.forRoot(routes)`                          |
| Standalone components             | NgModules                                               |
| `loadComponent()`                 | Module-based lazy loading                               |
| `loadChildren()` with route files | `loadChildren: () => import(...).then(...)` for modules |
| Functional guards                 | Class-based guards                                      |
| `CanActivateFn`                   | `CanActivate` class                                     |
| `CanMatchFn`                      | Older code may use `CanLoad`                            |
| `inject()`                        | Constructor injection                                   |

### Important

Don't assume old code is "wrong."

You'll encounter:

```ts
RouterModule.forRoot()
RouterModule.forChild()
@Injectable()
class AuthGuard implements CanActivate
```

in many production Angular applications.

You should be able to **read and maintain** it, while knowing modern Angular prefers standalone APIs and functional guards.

---

# 42. `CanLoad` vs `CanMatch`

You'll encounter `CanLoad` in older Angular applications.

Modern Angular generally favors:

```ts
CanMatch
```

for controlling route matching and conditional loading.

`CanMatch` also allows the router to try another matching route when the guard returns `false`, which makes it particularly useful for feature flags and alternate implementations. ([Angular][7])

For interviews:

> **Know `CanLoad` because you'll see it in existing projects, but understand `CanMatch` as the modern mechanism to reason about conditional route matching/loading.**



### Quick Revision

If you remember only this diagram, you're in good shape:

```text
                         Angular Router
                              │
                              ↓
                         URL changes
                              │
                              ↓
                       Route Recognition
                              │
                     ┌────────┴────────┐
                     ↓                 ↓
                  CanMatch          Lazy Load
                     │                 │
                     └────────┬────────┘
                              ↓
                           Guards
                              │
                     ┌────────┴────────┐
                     ↓                 ↓
                 CanActivate       CanActivateChild
                              │
                              ↓
                          Resolvers
                              │
                              ↓
                       Route Activation
                              │
                              ↓
                       Component Created
                              │
                              ↓
                       router-outlet
                              │
                              ↓
                         NavigationEnd
```

### Routing Cheat Sheet

| Requirement                      | Angular solution                 |
| -------------------------------- | -------------------------------- |
| Basic route                      | `path + component`               |
| Template navigation              | `routerLink`                     |
| Programmatic navigation          | `Router.navigate()`              |
| Route ID                         | `:id`                            |
| Query string                     | `queryParams`                    |
| Nested pages                     | `children`                       |
| Render child route               | Nested `router-outlet`           |
| Lazy component                   | `loadComponent`                  |
| Lazy feature                     | `loadChildren`                   |
| Authentication                   | `CanActivate`                    |
| Protect children                 | `CanActivateChild`               |
| Unsaved changes                  | `CanDeactivate`                  |
| Feature flag / conditional route | `CanMatch`                       |
| 404                              | `path: '**'`                     |
| Default route                    | `redirectTo + pathMatch: 'full'` |
| Route metadata                   | `data`                           |
| Page title                       | `title`                          |
| Preload lazy routes              | `withPreloading()`               |
| Fetch data before activation     | `resolve`                        |
| Navigation monitoring            | `Router.events`                  |
| Multiple routed areas            | Named `router-outlet`            |
| Feature-scoped DI                | Route `providers`                |
