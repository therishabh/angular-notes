# Angular Template Control Flow — `@if`, `@else`, `@switch`, `@for`

Modern Angular provides built-in **control flow syntax** for conditional rendering and loops.

Since you're learning modern Angular, the main syntax you should learn is:

```text
@if
@else if
@else
@switch / @case / @default
@for / @empty
```

> Older Angular code commonly uses `*ngIf` and `*ngFor`. You should know them for interviews and existing projects, but **modern Angular recommends the built-in `@` control flow syntax**.

---

# 1. `@if` — Conditional Rendering

## Core Concept

`@if` conditionally renders part of a template.

Syntax:

```html
@if (condition) {
  <!-- template -->
}
```

Example:

```ts
export class UserComponent {
  isLoggedIn = true;
}
```

```html
@if (isLoggedIn) {
  <p>Welcome back!</p>
}
```

If `isLoggedIn` is `true`, Angular renders the `<p>`.

If `false`, Angular doesn't render that block.

### Mental model

```text
condition === true
       ↓
render block

condition === false
       ↓
don't render block
```

This is **conditional rendering**, not just CSS hiding.

---

# 2. `@if ... @else`

Use `@else` when you have two possible UI states.

```html
@if (isLoggedIn) {
  <p>Welcome back!</p>
} @else {
  <p>Please login.</p>
}
```

Equivalent logic:

```text
if logged in
    → Welcome
else
    → Login
```

### Important

`@else` must immediately follow the `@if` block.

Correct:

```html
@if (condition) {
  ...
} @else {
  ...
}
```

---

# 3. `@else if`

For multiple conditions:

```html
@if (status === 'success') {
  <p>Payment successful</p>
} @else if (status === 'pending') {
  <p>Payment is processing</p>
} @else if (status === 'failed') {
  <p>Payment failed</p>
} @else {
  <p>Unknown status</p>
}
```

This is the Angular equivalent of normal JavaScript:

```ts
if (...) {
} else if (...) {
} else {
}
```

### Real-world example

```ts
status: 'loading' | 'success' | 'error' = 'loading';
```

```html
@if (status === 'loading') {
  <app-spinner />
} @else if (status === 'success') {
  <app-user-list />
} @else {
  <app-error />
}
```

This pattern is very common in real Angular applications.

---

# 4. `@if` with variable assignment

A useful Angular-specific feature is assigning the result of an expression.

```html
@if (user(); as user) {
  <h2>{{ user.name }}</h2>
  <p>{{ user.email }}</p>
}
```

This is particularly useful with signals/observables or expressions where you want to avoid repeatedly evaluating the same expression.

For example, if:

```ts
user = signal<User | null>({
  name: 'Rishabh',
  email: 'rishabh@example.com'
});
```

then:

```html
@if (user(); as currentUser) {
  <h2>{{ currentUser.name }}</h2>
}
```

Here:

```text
user()
   ↓
result assigned to currentUser
   ↓
currentUser.name
```

### Interview point

The `as` syntax gives the evaluated result a local template variable.

---

# 5. `@switch`

Use `@switch` when you're comparing **one expression against multiple possible values**.

Syntax:

```html
@switch (expression) {
  @case (value) {
    ...
  }
  @case (value) {
    ...
  }
  @default {
    ...
  }
}
```

Example:

```ts
status = 'pending';
```

```html
@switch (status) {
  @case ('success') {
    <p>Payment successful</p>
  }

  @case ('pending') {
    <p>Payment processing...</p>
  }

  @case ('failed') {
    <p>Payment failed</p>
  }

  @default {
    <p>Unknown status</p>
  }
}
```

---

# 6. `@switch` vs `@if`

This is an important interview comparison.

### Use `@if`

When conditions are different:

```html
@if (user && user.age >= 18) {
  ...
} @else if (isAdmin) {
  ...
}
```

### Use `@switch`

When you're comparing **one value** against multiple known values:

```html
@switch (status) {
  @case ('loading') { ... }
  @case ('success') { ... }
  @case ('error') { ... }
}
```

### Simple rule

> **Different conditions → `@if`**
> **One value, many possible values → `@switch`**

---

# 7. `@default`

`@default` is similar to JavaScript's `default` in a switch statement.

```html
@switch (role) {
  @case ('admin') {
    <p>Admin Dashboard</p>
  }

  @case ('user') {
    <p>User Dashboard</p>
  }

  @default {
    <p>Access denied</p>
  }
}
```

If none of the cases match, Angular renders `@default`.

`@default` is optional.

---

# 8. Important `@switch` Gotcha

Angular's `@switch` uses **strict equality (`===`)** for matching.

So conceptually:

```text
1 !== '1'
```

Therefore:

```html
@switch (value) {
  @case (1) {
    ...
  }
}
```

will not match:

```ts
value = '1';
```

This is a useful interview detail.

---

# 9. `@for` — Loops

`@for` is used to render a template for every item in a collection.

Syntax:

```html
@for (item of items; track item.id) {
  ...
}
```

Example:

```ts
users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Sarah' },
  { id: 3, name: 'Mike' }
];
```

```html
<ul>
  @for (user of users; track user.id) {
    <li>{{ user.name }}</li>
  }
</ul>
```

Output:

```text
John
Sarah
Mike
```

---

# 10. `track` — Very Important

This is one of the most important things to understand about modern Angular loops.

```html
@for (user of users; track user.id) {
  <li>{{ user.name }}</li>
}
```

`track` tells Angular how to identify each item.

For example:

```text
User 1 → id = 101
User 2 → id = 102
User 3 → id = 103
```

If the array changes, Angular can determine which DOM elements correspond to which users.

### Why?

Suppose:

```text
Before:
A
B
C

After:
A
C
D
```

With:

```html
track user.id
```

Angular knows:

```text
A → same
B → removed
C → same
D → new
```

So Angular can update the DOM efficiently instead of unnecessarily recreating everything.

### Best practice

Use a **stable unique identifier**:

```html
track user.id
```

rather than:

```html
track $index
```

when your collection has stable IDs.

---

# 11. `$index`

`@for` provides several contextual variables.

The most commonly used is:

```html
$index
```

Example:

```html
@for (user of users; track user.id) {
  <p>{{ $index + 1 }}. {{ user.name }}</p>
}
```

Output:

```text
1. John
2. Sarah
3. Mike
```

`$index` is zero-based.

```text
first item  → 0
second item → 1
third item  → 2
```

---

# 12. Other `@for` Context Variables

Angular provides:

| Variable | Meaning               |
| -------- | --------------------- |
| `$index` | Current index         |
| `$first` | `true` for first item |
| `$last`  | `true` for last item  |
| `$even`  | `true` for even index |
| `$odd`   | `true` for odd index  |
| `$count` | Total number of items |

Example:

```html
@for (user of users; track user.id) {
  @if ($first) {
    <strong>First user</strong>
  }

  <p>{{ $index }} - {{ user.name }}</p>
}
```

You can also alias them:

```html
@for (
  user of users;
  track user.id;
  let i = $index;
  let first = $first
) {
  <p>{{ i }} - {{ user.name }}</p>
}
```

This can make complex templates easier to read.

---

# 13. `@empty`

A very useful feature of `@for` is `@empty`.

```html
@for (user of users; track user.id) {
  <p>{{ user.name }}</p>
} @empty {
  <p>No users found.</p>
}
```

If:

```ts
users = [];
```

Angular renders:

```text
No users found.
```

This is cleaner than writing a separate:

```html
@if (users.length === 0)
```

in many situations.

---

# 14. `@for` with Objects

Usually you'll receive arrays from APIs:

```ts
products = [
  {
    id: 101,
    name: 'Laptop',
    price: 80000
  },
  {
    id: 102,
    name: 'Monitor',
    price: 20000
  }
];
```

Template:

```html
@for (product of products; track product.id) {
  <div>
    <h3>{{ product.name }}</h3>
    <p>₹{{ product.price }}</p>
  </div>
} @empty {
  <p>No products available.</p>
}
```

This is the pattern you'll use constantly in real applications.

---

# 15. Nested Control Flow

You can combine these constructs.

```html
@for (user of users; track user.id) {
  <div>
    <h3>{{ user.name }}</h3>

    @if (user.isActive) {
      <span>Active</span>
    } @else {
      <span>Inactive</span>
    }
  </div>
}
```

This is completely normal Angular code.

---

# 16. `@if` vs CSS Hiding

This distinction is important.

### `@if`

```html
@if (isVisible) {
  <app-user />
}
```

When false, Angular does not render that block.

### CSS

```html
<app-user [class.hidden]="!isVisible" />
```

The element still exists in the DOM; you're just controlling its presentation.

So:

```text
@if
→ controls whether the view exists

CSS/display
→ controls presentation of an existing element
```

This distinction can matter for:

* component lifecycle
* DOM size
* performance
* accessibility
* state preservation

---

# 17. Older Angular Syntax

Before built-in control flow, Angular commonly used structural directives.

### Modern

```html
@if (isLoggedIn) {
  <p>Welcome</p>
}
```

### Older

```html
<p *ngIf="isLoggedIn">
  Welcome
</p>
```

---

### Modern loop

```html
@for (user of users; track user.id) {
  <p>{{ user.name }}</p>
}
```

### Older loop

```html
<p *ngFor="let user of users; trackBy: trackById">
  {{ user.name }}
</p>
```

### Modern → Older

| Modern Angular  | Older/common         |
| --------------- | -------------------- |
| `@if`           | `*ngIf`              |
| `@for`          | `*ngFor`             |
| `@switch`       | `*ngSwitch`          |
| `track user.id` | `trackBy: trackByFn` |

You should **learn the modern syntax first**, but definitely recognize the older syntax when working on existing Angular applications.

---

# 18. Why Did Angular Introduce `@if` / `@for`?

This is a good senior-level interview question.

The built-in control-flow syntax provides a cleaner template syntax and allows Angular to implement control flow directly in the template compiler/runtime rather than expressing it through the older structural-directive syntax.

The new syntax is also easier to compose:

```html
@if (...) {
  @for (...) {
    ...
  }
}
```

And `@for` has explicit `track` semantics.

---

# 19. Common Mistakes / Gotchas

### Mistake 1 — Forgetting `track`

Prefer:

```html
@for (user of users; track user.id) {
```

Don't blindly use:

```html
@for (user of users; track $index) {
```

`$index` can be problematic when items are inserted, removed, or reordered because the index isn't stable identity.

---

### Mistake 2 — Using `$index` as identity

Suppose:

```text
Before:
A → index 0
B → index 1
C → index 2
```

Remove A:

```text
B → index 0
C → index 1
```

The indices changed even though B and C are the same logical entities.

That's why:

```html
track user.id
```

is usually better.

---

### Mistake 3 — Calling expensive functions in conditions

Avoid:

```html
@if (calculateUserPermissions()) {
```

if `calculateUserPermissions()` performs expensive work and can be evaluated frequently.

Prefer deriving the state appropriately.

---

### Mistake 4 — Using `@switch` for unrelated conditions

Don't force this:

```html
@switch (true) {
  @case (user && user.isAdmin) { ... }
  @case (user && user.isActive) { ... }
}
```

Use `@if / @else if` when conditions are logically different.

---

# 20. Interview Questions

### Q1. What is Angular's modern conditional rendering syntax?

**Answer:**

```html
@if (condition) {
  ...
} @else if (...) {
  ...
} @else {
  ...
}
```

---

### Q2. `@if` vs `*ngIf`?

**Answer:**

`@if` is the modern built-in control-flow syntax. `*ngIf` is the older structural-directive approach still found in many existing Angular applications.

---

### Q3. How does `@for` differ from JavaScript `for`?

**Answer:**

`@for` is Angular template control flow used to render UI for each item. It isn't a general-purpose TypeScript loop.

```html
@for (user of users; track user.id) {
  <app-user-card [user]="user" />
}
```

---

### Q4. Why is `track` important?

**Answer:**

It gives Angular an identity for each collection item so Angular can efficiently determine which DOM nodes need to be created, reused, moved, or removed.

---

### Q5. Why is `track $index` usually not ideal?

**Answer:**

The index represents a position, not the identity of the item. When items are inserted, removed, or reordered, indexes can change and lead to inefficient DOM updates or incorrect preservation of DOM state.

---

### Q6. What is `@empty`?

**Answer:**

It defines the UI that should render when the `@for` collection contains no items.

```html
@for (user of users; track user.id) {
  <p>{{ user.name }}</p>
} @empty {
  <p>No users found.</p>
}
```

---

### Q7. `@if` vs `@switch`?

**Answer:**

Use `@if` when evaluating different boolean/logical conditions.

Use `@switch` when comparing one expression against multiple known values.

---

### Q8. What does `$index` represent?

**Answer:**

The zero-based index of the current item in an `@for` loop.

---

### Q9. Name the contextual variables available in `@for`.

**Answer:**

```text
$index
$count
$first
$last
$even
$odd
```

---

### Q10. Does `@if(false)` simply hide the element?

**Answer:**

No. The block is conditionally rendered; it isn't equivalent to merely applying `display: none`.

---

### Q11. Scenario: API returns 10,000 records. What would you consider?

**Answer:**

Don't blindly render all 10,000 items.

Consider:

* pagination
* virtual scrolling
* server-side filtering
* server-side sorting
* efficient `track`
* minimizing expensive template expressions
* avoiding unnecessary DOM creation

`track` helps Angular reconcile the list efficiently, but it **doesn't make rendering 10,000 DOM elements cheap**.

---

# 21. Quick Revision

### Conditions

```html
@if (condition) {
  ...
} @else if (condition) {
  ...
} @else {
  ...
}
```

### Switch

```html
@switch (status) {
  @case ('success') {
    ...
  }
  @case ('error') {
    ...
  }
  @default {
    ...
  }
}
```

### Loop

```html
@for (user of users; track user.id) {
  {{ user.name }}
} @empty {
  No users
}
```

### Loop variables

```text
$index  → current index
$count  → total count
$first  → first item?
$last   → last item?
$even   → even index?
$odd    → odd index?
```

### Modern → Older

```text
@if      → *ngIf
@for     → *ngFor
@switch  → *ngSwitch
```

### Most important interview point

> **Always understand `track`.** For dynamic lists, use a stable unique identity such as `user.id`, not `$index`, unless you have a specific reason that index-based tracking is appropriate.

### Mental model

```text
@if / @else
    ↓
Conditional UI

@switch / @case
    ↓
Multiple values → one matching UI branch

@for
    ↓
Collection → repeated UI

@empty
    ↓
Empty collection → fallback UI

track
    ↓
Item identity → efficient DOM reconciliation
```
