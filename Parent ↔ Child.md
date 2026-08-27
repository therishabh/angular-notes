
# Angular Component Communication — Parent ↔ Child

## 1. Core Concept

When Angular components have a parent-child relationship, they often need to exchange data.

The most important patterns are:

```text
Parent → Child
    ↓
Input

Child → Parent
    ↓
Output

Parent ↔ Child
    ↓
Two-way binding / model()
```

### Modern Angular

```text
Parent → Child
input()

Child → Parent
output()

Two-way
model()
```

### Older / traditional Angular

```text
Parent → Child
@Input()

Child → Parent
@Output() + EventEmitter
```

The older APIs are **still fully supported**, but Angular recommends the signal-based `input()` and `output()` APIs for new projects. ([Angular][1])

---

# 2. Parent → Child Data Transfer

## Modern Angular — `input()`

Suppose we have:

```text
ParentComponent
      |
      | user
      ↓
UserCardComponent
```

The parent owns the data.

### Child

```ts
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <h2>{{ name() }}</h2>
    <p>Age: {{ age() }}</p>
  `
})
export class UserCardComponent {

  // `name` is an input signal.
  // The parent can provide a value for this input.
  name = input.required<string>();

  // `age` is another required input.
  age = input.required<number>();
}
```

### Parent

```ts
export class ParentComponent {

  // Parent owns the actual state.
  userName = 'Rishabh';

  userAge = 30;
}
```

Parent template:

```html
<app-user-card
  [name]="userName"
  [age]="userAge"
/>
```

### Data flow

```text
Parent
  |
  | [name]="userName"
  | [age]="userAge"
  ↓
Child
  |
  | name()
  | age()
```

The important point is:

> **The child receives the value, but the `input()` signal itself is read-only inside the child.** Angular updates it when the parent binding changes. ([Angular][1])

---

# 3. Why `input()` Is Called a Signal Input

This:

```ts
name = input.required<string>();
```

creates an `InputSignal`.

Therefore:

```ts
name()
```

reads its current value.

You can also use it in:

```ts
computed(() => ...)
```

or:

```ts
effect(() => ...)
```

Example:

```ts
name = input.required<string>();

nameLabel = computed(() => {
  return `User: ${this.name()}`;
});
```

This is one of the major advantages over the traditional `@Input()` approach: the input participates naturally in Angular's signal-based reactivity. ([Angular][1])

---

# 4. Required vs Optional Input

### Required

```ts
name = input.required<string>();
```

Parent **must** provide it.

```html
<app-user-card [name]="userName" />
```

If the required input isn't provided, Angular reports an error at build time. ([Angular][1])

### Optional with default value

```ts
name = input('Unknown');
```

Now the child can be used without providing `name`.

```html
<app-user-card />
```

The value will be:

```text
Unknown
```

---

# 5. Older Angular — `@Input()`

Before signal inputs, the common approach was:

### Child

```ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <h2>{{ name }}</h2>
    <p>Age: {{ age }}</p>
  `
})
export class UserCardComponent {

  // Traditional Angular input.
  @Input() name!: string;

  // Another traditional input.
  @Input() age!: number;
}
```

Parent:

```html
<app-user-card
  [name]="userName"
  [age]="userAge"
/>
```

The binding syntax is the **same**:

```html
[name]="userName"
```

The major difference is how the child declares and reads the input:

```text
Old:
@Input() name
name

Modern:
name = input()
name()
```

Angular still fully supports `@Input()`. ([Angular][1])

---

# 6. `input()` vs `@Input()` — Interview Comparison

| Modern                            | Older                          |
| --------------------------------- | ------------------------------ |
| `input()`                         | `@Input()`                     |
| Signal-based                      | Property-based                 |
| Read using `name()`               | Read using `name`              |
| Naturally integrates with signals | Traditional Angular reactivity |
| `input.required()`                | `@Input({ required: true })`   |
| Recommended for new code          | Still fully supported          |

### Interview answer

> **`input()` is the modern signal-based API for component inputs. `@Input()` is the traditional decorator-based API and remains supported. The parent-to-child template binding syntax is the same, but `input()` returns an `InputSignal`, so the child reads it as a signal and can use it naturally with `computed()` and other reactive APIs.** ([Angular][1])

---

# 7. Child → Parent Data Transfer

Now reverse the direction:

```text
Parent
   ↑
   |
   | selected user
   |
Child
```

The child **cannot directly modify the parent's input state**.

Instead, the child raises an event.

---

# 8. Modern Angular — `output()`

### Child

```ts
import { Component, output } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <button (click)="selectUser()">
      Select User
    </button>
  `
})
export class UserCardComponent {

  // Defines a custom output event.
  // The event carries a number.
  userSelected = output<number>();

  selectUser() {

    // Emit the selected user's ID to the parent.
    this.userSelected.emit(101);
  }
}
```

### Parent

```html
<app-user-card
  (userSelected)="handleUserSelected($event)"
/>
```

Parent:

```ts
export class ParentComponent {

  handleUserSelected(userId: number) {

    console.log('Selected user:', userId);
  }
}
```

Flow:

```text
Child
  |
  | userSelected.emit(101)
  ↓
Parent
  |
  | (userSelected)
  ↓
handleUserSelected(101)
```

Angular's `output()` API creates an `OutputEmitterRef`, whose `emit()` method sends the event to consumers such as the parent. ([Angular][2])

---

# 9. `$event`

This is important.

Child:

```ts
this.userSelected.emit(101);
```

Parent:

```html
(userSelected)="handleUserSelected($event)"
```

Here:

```text
$event = 101
```

So:

```ts
handleUserSelected(userId: number) {
  console.log(userId);
}
```

receives:

```text
101
```

The child can emit any appropriate value:

```ts
this.userSelected.emit(user);
```

Then:

```html
(userSelected)="handleUserSelected($event)"
```

and:

```ts
handleUserSelected(user: User) {}
```

---

# 10. Child → Parent with Event Object

You don't have to emit only primitive values.

Example:

```ts
userDeleted = output<{
  id: number;
  reason: string;
}>();
```

Then:

```ts
deleteUser() {

  this.userDeleted.emit({
    id: 101,
    reason: 'User requested deletion'
  });
}
```

Parent:

```html
<app-user-card
  (userDeleted)="handleDelete($event)"
/>
```

Parent receives:

```ts
handleDelete(event: { id: number; reason: string }) {

  console.log(event.id);
  console.log(event.reason);
}
```

### Best practice

Emit a **meaningful event payload**, not an unnecessarily large object.

---

# 11. Older Angular — `@Output()` + `EventEmitter`

Traditional approach:

### Child

```ts
import {
  Component,
  EventEmitter,
  Output
} from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <button (click)="selectUser()">
      Select
    </button>
  `
})
export class UserCardComponent {

  // Traditional Angular custom event.
  @Output()
  userSelected = new EventEmitter<number>();

  selectUser() {

    // Send the value to the parent.
    this.userSelected.emit(101);
  }
}
```

### Parent

```html
<app-user-card
  (userSelected)="handleUserSelected($event)"
/>
```

Same communication flow:

```text
Child
  ↓
EventEmitter.emit()
  ↓
Parent event binding
  ↓
Handler
```

The modern equivalent is:

```ts
userSelected = output<number>();
```

instead of:

```ts
@Output()
userSelected = new EventEmitter<number>();
```

Angular recommends `output()` for new projects, while `@Output()` remains supported. ([Angular][3])

---

# 12. `output()` vs `@Output()` — Interview Comparison

| Modern                       | Older                   |
| ---------------------------- | ----------------------- |
| `output()`                   | `@Output()`             |
| `OutputEmitterRef`           | `EventEmitter`          |
| `.emit()`                    | `.emit()`               |
| Cleaner modern API           | Traditional Angular API |
| Recommended for new projects | Still supported         |

### Important interview point

Don't say:

> "`@Output()` is deprecated."

That's incorrect.

Better:

> **`output()` is the recommended modern API for new Angular code, while `@Output()` remains fully supported for existing applications.** ([Angular][4])

---

# 13. Parent → Child + Child → Parent Together

This is the **most common interview example**.

Imagine a parent displaying a product card.

```text
                  Parent
             /             \
            ↓               ↑
       [product]       (addToCart)
            ↓               ↑
             Child
```

### Child

```ts
import {
  Component,
  input,
  output
} from '@angular/core';

interface Product {
  id: number;
  name: string;
  price: number;
}

@Component({
  selector: 'app-product-card',
  template: `
    <h3>{{ product().name }}</h3>

    <p>
      ₹{{ product().price }}
    </p>

    <button (click)="addToCart()">
      Add to Cart
    </button>
  `
})
export class ProductCardComponent {

  // Data comes from parent.
  product = input.required<Product>();

  // Event goes back to parent.
  addToCartEvent = output<Product>();

  addToCart() {

    // Send the product back to the parent.
    this.addToCartEvent.emit(this.product());
  }
}
```

### Parent

```ts
export class ProductListComponent {

  products = [
    {
      id: 1,
      name: 'Laptop',
      price: 80000
    },
    {
      id: 2,
      name: 'Mouse',
      price: 2000
    }
  ];

  addToCart(product: Product) {

    console.log('Adding to cart:', product);
  }
}
```

Parent template:

```html
@for (product of products; track product.id) {

  <app-product-card
    [product]="product"
    (addToCartEvent)="addToCart($event)"
  />
}
```

This single example demonstrates:

```text
[product]
    ↓
Parent → Child

(addToCartEvent)
    ↑
Child → Parent
```

This is probably the **most useful pattern to memorize for interviews**.

---

# 14. Important Rule: Child Should Not Mutate Input

Suppose:

```ts
product = input.required<Product>();
```

Don't do this:

```ts
this.product().price = 500;
```

The child should generally treat an input as **data supplied by the parent**, not as state that the child owns.

Instead:

```text
Child needs change
      ↓
emit event
      ↓
Parent updates state
      ↓
Parent passes new value
      ↓
Child receives new input
```

This creates **unidirectional data flow**.

---

# 15. The Most Important Architecture Concept: Unidirectional Data Flow

Angular component communication generally follows:

```text
                 Parent
                /      \
               ↓        ↑
            Input      Output
               ↓        ↑
             Child
```

Data:

```text
Parent → Child
```

Events:

```text
Child → Parent
```

This is intentionally designed so that the parent owns the state and the child communicates requested changes.

### Interview answer

> **Angular encourages unidirectional data flow: inputs carry data down from parent to child, while outputs communicate events up from child to parent.**

---

# 16. What if Child Wants to Change Parent Data?

Don't do:

```text
Child directly modifies Parent property
```

Instead:

```text
Child
  ↓
emit event
  ↓
Parent handler
  ↓
Parent updates its state
  ↓
New input value
  ↓
Child
```

Example:

```ts
// Child
increment = output<void>();

increase() {
  this.increment.emit();
}
```

Parent:

```html
<app-counter
  (increment)="increaseCounter()"
/>
```

Parent:

```ts
count = 0;

increaseCounter() {
  this.count++;
}
```

This keeps ownership clear.

---

# 17. Two-Way Binding — `model()`

Now there is another scenario.

Suppose a child component represents a UI control:

```text
Slider
Checkbox
Date picker
Custom select
Toggle
```

The parent provides a value, but the child also needs to modify that value.

This is where modern Angular provides:

```ts
model()
```

Angular describes `model()` as a special input that allows the component to propagate changes back to the parent. ([Angular][1])

---

# 18. Modern Two-Way Binding with `model()`

### Child

```ts
import {
  Component,
  model
} from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <button (click)="increment()">
      +
    </button>

    <span>{{ count() }}</span>
  `
})
export class CounterComponent {

  // Model is both readable and writable.
  count = model(0);

  increment() {

    // Updating the model also propagates the change
    // to the parent's two-way binding.
    this.count.update(value => value + 1);
  }
}
```

### Parent

```ts
count = signal(10);
```

Template:

```html
<app-counter [(count)]="count" />
```

Flow:

```text
Parent count
     ↓
   [count]
     ↓
Child model
     ↓
Child updates model
     ↓
(countChange)
     ↓
Parent count
```

`model()` automatically creates a corresponding `countChange` output behind the scenes. ([Angular][1])

---

# 19. What Does `[(count)]` Actually Mean?

This:

```html
<app-counter [(count)]="count" />
```

is two-way binding.

Conceptually, it combines:

```html
<app-counter
  [count]="count"
  (countChange)="count = $event"
/>
```

So:

```text
[count]
    ↓
Parent → Child

(countChange)
    ↑
Child → Parent
```

This is why it's often called:

> **Banana in a box**

```text
[(count)]
```

---

# 20. `model()` vs `input()` + `output()`

This is an excellent interview question.

### `input()`

```ts
value = input(0);
```

Means:

```text
Parent → Child
```

The child receives the value but doesn't write to the input.

### `model()`

```ts
value = model(0);
```

Means:

```text
Parent ↔ Child
```

The child can update the model.

### `output()`

```ts
valueChanged = output<number>();
```

Means:

```text
Child → Parent event
```

### Comparison

| API        | Direction      | Writable in child? | Main use        |
| ---------- | -------------- | -----------------: | --------------- |
| `input()`  | Parent → Child |                  ❌ | Receive data    |
| `output()` | Child → Parent |                N/A | Emit events     |
| `model()`  | Parent ↔ Child |                  ✅ | Two-way binding |

---

# 21. Older Angular Two-Way Binding

Before `model()`, you commonly implemented this manually.

Child:

```ts
@Input()
value = 0;

@Output()
valueChange = new EventEmitter<number>();

increment() {

  this.value++;

  this.valueChange.emit(this.value);
}
```

Parent:

```html
<app-counter
  [(value)]="count"
/>
```

Angular recognizes:

```text
value
+
valueChange
```

as the two-way binding pair.

Conceptually:

```html
<app-counter
  [value]="count"
  (valueChange)="count = $event"
/>
```

This is still a very important interview pattern because you will see it in existing Angular code.

---

# 22. `model()` vs Old `@Input/@Output`

### Old

```ts
@Input()
value = 0;

@Output()
valueChange = new EventEmitter<number>();
```

### Modern

```ts
value = model(0);
```

Modern Angular handles the corresponding input/output relationship for the model.

This is much cleaner for components whose primary purpose is modifying a value, such as custom form controls. ([Angular][1])

---

# 23. `input()` + `output()` vs `model()`

Don't use `model()` everywhere.

### Use `input()` + `output()` when:

The child is receiving data and reporting an **event/action**.

Example:

```text
ProductCard
    ↓
[product]
    ↓
Child

Child
    ↓
(addToCart)
    ↓
Parent
```

### Use `model()` when:

The child is essentially a **value editor/control**.

Examples:

```text
Checkbox
Slider
DatePicker
CustomSelect
Toggle
```

Mental model:

```text
Input/output
→ "Here is data; tell me what happened."

Model
→ "Here is a value; you can edit it."
```

---

# 24. Can Parent Directly Access Child?

Yes, Angular also provides mechanisms such as:

```ts
@ViewChild()
```

or modern query APIs such as:

```ts
viewChild()
```

Example:

```ts
child = viewChild(UserComponent);
```

This allows the parent to obtain a reference to a child component/directive.

But this is **not the normal mechanism for data communication**.

Prefer:

```text
Input
Output
Model
```

for normal component communication.

Use queries when the parent genuinely needs to interact with a child instance—for example, invoking a public method or accessing a child instance for imperative coordination.

---

# 25. What About Service-Based Communication?

Suppose components aren't parent/child:

```text
Component A

      ↕
   Shared Service
      ↕

Component B
```

Using `input()`/`output()` doesn't make sense because there's no direct parent-child relationship.

You can use a shared service with:

```text
Signals
RxJS Subject/BehaviorSubject
```

depending on the architecture.

Example:

```ts
@Injectable({
  providedIn: 'root'
})
export class CartService {

  cartCount = signal(0);
}
```

Both components can use the service.

This is more appropriate for:

```text
Sibling components
Unrelated components
Cross-feature state
Shared application state
```

---

# 26. Communication Patterns — Interview Cheat Sheet

| Scenario                    | Recommended approach     |
| --------------------------- | ------------------------ |
| Parent → Child              | `input()`                |
| Child → Parent event        | `output()`               |
| Parent ↔ Child value        | `model()`                |
| Existing/older code         | `@Input()` / `@Output()` |
| Sibling communication       | Shared service/state     |
| Unrelated components        | Shared service/state     |
| Parent needs child instance | `viewChild()` / query    |
| Global/shared state         | Service/store/signals    |

---

# 27. Common Mistakes / Gotchas

### Mistake 1 — Child directly modifying parent's state

Avoid designing:

```text
Child → directly modify Parent
```

Prefer:

```text
Child
 ↓
output()
 ↓
Parent
 ↓
update state
```

---

### Mistake 2 — Using `output()` for data that is actually a value model

If you're building:

```text
Custom checkbox
Custom slider
Custom date picker
```

and the child needs to modify the value, `model()` may be a better fit than manually pairing `input()` and `output()`.

---

### Mistake 3 — Using `model()` for everything

`model()` is specifically useful for two-way binding.

For normal parent-owned data:

```ts
product = input.required<Product>();
```

is cleaner.

---

### Mistake 4 — Confusing `$event`

For:

```html
<app-user
  (userSelected)="selectUser($event)"
/>
```

`$event` is the value emitted by the child:

```ts
this.userSelected.emit(user);
```

It is **not necessarily a DOM `Event` object**.

This is a common interview trap.

---

### Mistake 5 — Thinking custom outputs bubble like DOM events

Angular custom component outputs **do not bubble through the DOM**. The parent must listen to the output on the component that emits it. ([Angular][4])

---

# 28. Interview Questions

## Q1. How do you pass data from parent to child?

**Modern:**

```ts
// Child
user = input.required<User>();
```

```html
<!-- Parent -->
<app-user [user]="selectedUser" />
```

**Older:**

```ts
@Input() user!: User;
```

---

## Q2. How do you pass data from child to parent?

**Modern:**

```ts
// Child
userSelected = output<User>();

this.userSelected.emit(user);
```

Parent:

```html
<app-user
  (userSelected)="handleUser($event)"
/>
```

**Older:**

```ts
@Output()
userSelected = new EventEmitter<User>();
```

---

## Q3. What is the difference between `input()` and `output()`?

**Answer:**

> `input()` receives data from the parent, while `output()` emits events/data from the child to the parent.

```text
input()
Parent → Child

output()
Child → Parent
```

---

## Q4. Can a child modify an `input()`?

**Answer:**

No. An `InputSignal` is read-only from the child's perspective.

If the child needs to request a change, use an output or, when appropriate, `model()` for two-way binding. ([Angular][1])

---

## Q5. What is `model()`?

**Answer:**

> `model()` creates a writable model input that supports two-way binding. The child can update it, and Angular propagates that change back to the parent's binding. ([Angular][1])

---

## Q6. What is the difference between `model()` and `input()`?

```text
input()
→ read-only input
→ Parent → Child

model()
→ writable input
→ Parent ↔ Child
```

---

## Q7. What is the older equivalent of `model()`?

Typically:

```ts
@Input() value = 0;

@Output()
valueChange = new EventEmitter<number>();
```

and:

```html
<app-counter [(value)]="count" />
```

---

## Q8. What does this mean?

```html
<app-counter [(value)]="count" />
```

**Answer:**

It is two-way binding and conceptually corresponds to:

```html
<app-counter
  [value]="count"
  (valueChange)="count = $event"
/>
```

With `model()`, Angular creates the corresponding change output automatically. ([Angular][1])

---

## Q9. Is `@Output()` deprecated?

**Answer:**

No.

`output()` is the modern recommended API for new code, but `@Output()` remains fully supported. ([Angular][4])

---

## Q10. Is `@Input()` deprecated?

**Answer:**

No.

`input()` is recommended for new code, but `@Input()` remains fully supported. ([Angular][1])

---

## Q11. What happens if the parent changes an `input()` signal?

**Answer:**

Angular updates the child's input signal automatically.

For example:

```ts
// Parent
name = signal('John');

this.name.set('Rishabh');
```

Child:

```ts
name = input.required<string>();
```

```html
<h2>{{ name() }}</h2>
```

The child sees the updated value reactively. ([Angular][5])

---

## Q12. Scenario: Child has a `product` input and user clicks "Delete". Should child delete the product directly?

**Answer:**

Usually no.

Child should emit:

```ts
productDeleted = output<number>();

deleteProduct() {
  this.productDeleted.emit(this.product().id);
}
```

Parent handles the actual state mutation:

```html
<app-product
  (productDeleted)="deleteProduct($event)"
/>
```

This keeps state ownership in the parent.

---

## Q13. Scenario: You are creating a reusable date picker. Parent provides selected date, but the date picker must update it when the user selects another date. What would you use?

**Answer:**

Modern Angular:

```ts
selectedDate = model<Date | null>(null);
```

Parent:

```html
<app-date-picker [(selectedDate)]="selectedDate" />
```

This is an appropriate use of `model()` because the component acts like a value-editing control. ([Angular][1])

---

## Q14. Scenario: Two sibling components need to communicate. Would you use `@Output()`?

**Answer:**

Not directly, because neither component is the other's parent.

Prefer a shared service/state mechanism:

```text
Sibling A
   ↓
Shared Service
   ↓
Sibling B
```

Signals or RxJS can be used depending on the requirements.

---

# 29. Quick Revision

### Parent → Child

Modern:

```ts
// Child
user = input.required<User>();
```

```html
<!-- Parent -->
<app-user [user]="user" />
```

---

### Child → Parent

Modern:

```ts
// Child
userSelected = output<User>();

this.userSelected.emit(user);
```

```html
<!-- Parent -->
<app-user
  (userSelected)="handleUser($event)"
/>
```

---

### Parent ↔ Child

Modern:

```ts
// Child
value = model(0);
```

```html
<!-- Parent -->
<app-counter [(value)]="count" />
```

---

### Old → Modern

```text
@Input()                    → input()

@Output() + EventEmitter   → output()

@Input() + @Output() pair  → model() for two-way value binding
```

---

# 30. The Interview Mental Model

This is the one diagram I'd keep in your notes:

```text
                         PARENT
                           │
                           │
                    ┌──────┴──────┐
                    │             │
                 input()       output()
                    │             ↑
                    │             │
                    ↓             │
                  CHILD ──────────┘


Parent → Child
    [value]
       ↓
    input()


Child → Parent
    output()
       ↓
    (valueChanged)


Parent ↔ Child
    [(value)]
       ↕
    model()
```

### The golden rule

> **Parent owns the state. Child receives state through inputs and communicates user actions/changes through outputs. Use `model()` when the child is intentionally a two-way value editor.**

For a senior interview, don't stop at memorizing `@Input` and `@Output`. Be ready to explain **why the data flow is unidirectional, why a child shouldn't mutate an input, when `model()` is better than input/output, and how the modern signal APIs differ from the legacy decorator APIs.**
