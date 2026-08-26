# Angular Signals

## 1. Core Concept

### What is a Signal?

A **Signal is a reactive wrapper around a value**.

It allows Angular to know:

> **Which code/template is using this value, and when that value changes.**

Basic example:

```ts
import { signal } from '@angular/core';

export class CounterComponent {
  count = signal(0);
}
```

A normal property:

```ts
count = 0;
```

A signal:

```ts
count = signal(0);
```

The important difference is that a signal is **reactive**.

### Reading a signal

You read a signal by calling it:

```ts
count()
```

not:

```ts
count
```

In a template:

```html
<p>{{ count() }}</p>
```

Angular tracks this signal read and can react when the signal changes. ([angular.dev][1])

---

# 2. How Signals Work

Think of a signal as:

```text
             signal
                │
        ┌───────┴────────┐
        │                │
      value          dependency
        │                │
        ↓                ↓
       10          who is using it?
```

When the value changes:

```text
signal changes
      ↓
Angular knows its consumers
      ↓
affected consumers are notified
      ↓
UI/reactive computation can update
```

This is why Signals are useful for **fine-grained reactive state**. Angular tracks where signals are read and can optimize rendering updates accordingly. ([angular.dev][1])

---

# 3. Creating a Signal

Syntax:

```ts
signal(initialValue)
```

Example:

```ts
import { signal } from '@angular/core';

export class UserComponent {
  name = signal('Rishabh');
  age = signal(30);
  isLoggedIn = signal(false);
}
```

Signals can contain different types:

```ts
count = signal(10);

name = signal('Rishabh');

isLoggedIn = signal(true);

user = signal({
  id: 1,
  name: 'Rishabh'
});

users = signal([
  { id: 1, name: 'John' },
  { id: 2, name: 'Sarah' }
]);
```

A signal can hold primitives as well as objects/arrays. ([angular.dev][1])

---

# 4. Reading a Signal

This is one of the most important rules.

### TypeScript

```ts
count()
```

### Template

```html
<p>{{ count() }}</p>
```

You **call** a signal because the signal itself behaves like a getter function.

Example:

```ts
count = signal(10);

console.log(count()); // 10
```

Not:

```ts
console.log(count); // Signal object/function, not the value
```

### Mental model

```text
count
  ↓
Signal

count()
  ↓
Current value
```

---

# 5. Updating a Signal

There are two primary ways to update a writable signal:

```text
set()
update()
```

---

## `set()`

Use `set()` when you already know the new value.

```ts
count = signal(0);

increment() {
  this.count.set(10);
}
```

This changes:

```text
0 → 10
```

Another example:

```ts
name = signal('John');

changeName() {
  this.name.set('Rishabh');
}
```

---

## `update()`

Use `update()` when the new value depends on the current value.

```ts
count = signal(0);

increment() {
  this.count.update(current => current + 1);
}
```

Flow:

```text
current value
     ↓
update(current => ...)
     ↓
new value
```

### `set()` vs `update()`

| `set()`                   | `update()`                                 |
| ------------------------- | ------------------------------------------ |
| You provide the new value | You calculate new value from current value |
| `count.set(10)`           | `count.update(v => v + 1)`                 |
| Good for replacement      | Good for transformation                    |

Official Angular examples use `set()` for direct replacement and `update()` when the new value depends on the current value. ([angular.dev][2])

---

# 6. Signals in Templates

Example:

```ts
export class CounterComponent {

  count = signal(0);

  increment() {
    this.count.update(value => value + 1);
  }
}
```

Template:

```html
<h2>{{ count() }}</h2>

<button (click)="increment()">
  Increment
</button>
```

Flow:

```text
User clicks button
       ↓
increment()
       ↓
count.update(...)
       ↓
Signal changes
       ↓
Angular knows template reads count()
       ↓
UI updates
```

This is one of the most common real-world uses of Signals.

---

# 7. Signals vs Normal Properties

This is important because you shouldn't convert **every property** into a signal.

### Normal property

```ts
name = 'Rishabh';
```

### Signal

```ts
name = signal('Rishabh');
```

With normal property:

```ts
this.name = 'John';
```

With signal:

```ts
this.name.set('John');
```

Read:

```ts
this.name()
```

### When is a signal useful?

Use a signal when the value is **reactive state** and other Angular code needs to react to changes.

Examples:

```text
UI state
Loading state
Selected item
Filter state
Pagination state
Modal visibility
Form-related UI state
Derived application state
```

Don't blindly turn constants or values that don't need reactivity into signals.

---

# 8. Writable Signals

`signal()` creates a **writable signal**.

```ts
count = signal(0);
```

You can:

```ts
count.set(10);

count.update(value => value + 1);
```

Conceptually:

```text
signal()
   ↓
WritableSignal
   ├── read()
   ├── set()
   └── update()
```

---

# 9. `computed()` — Derived State

This is the second major Signals concept.

Sometimes you don't want to store a value separately.

Instead, you want to **derive it from other signals**.

Example:

```ts
count = signal(10);

doubleCount = computed(() => this.count() * 2);
```

Now:

```ts
count()
```

is:

```text
10
```

and:

```ts
doubleCount()
```

is:

```text
20
```

If:

```ts
count.set(20);
```

then:

```ts
doubleCount()
```

automatically becomes:

```text
40
```

Angular automatically tracks the dependency between `doubleCount` and `count`. ([angular.dev][1])

---

# 10. `computed()` is Read-Only

This is extremely important.

```ts
count = signal(10);

doubleCount = computed(() => this.count() * 2);
```

You can:

```ts
console.log(this.doubleCount());
```

But you cannot:

```ts
this.doubleCount.set(100);
```

A computed signal is **read-only**.

Why?

Because it represents **derived state**.

```text
count
  ↓
computed
  ↓
doubleCount
```

The source of truth is `count`, not `doubleCount`.

---

# 11. `computed()` is Lazy and Memoized

This is an important interview topic.

Consider:

```ts
count = signal(10);

doubleCount = computed(() => {
  console.log('computed executed');
  return this.count() * 2;
});
```

The computation doesn't execute simply because `computed()` was declared.

It is **lazy**.

It runs when:

```ts
doubleCount()
```

is first read.

The result is then **cached/memoized** until a dependency changes. ([angular.dev][1])

Conceptually:

```text
computed()
   ↓
No calculation yet

doubleCount()
   ↓
Calculate
   ↓
Cache result

doubleCount()
   ↓
Return cached result

count changes
   ↓
Cache invalidated

doubleCount()
   ↓
Recalculate
```

This makes `computed()` useful for expensive derived calculations.

---

# 12. Dynamic Dependencies in `computed()`

This is a more advanced but important concept.

```ts
showCount = signal(false);

count = signal(10);

message = computed(() => {
  if (this.showCount()) {
    return `Count: ${this.count()}`;
  }

  return 'Count hidden';
});
```

When:

```ts
showCount() === false
```

the computation does **not read `count()`**.

Therefore `count` is not currently a dependency.

If:

```ts
showCount.set(true);
```

then Angular re-evaluates the computation, reads `count()`, and `count` becomes a dependency.

Dependencies are therefore tracked dynamically based on which signals are actually read. ([angular.dev][1])

---

# 13. `computed()` — Real-World Example

Suppose:

```ts
products = signal([
  { id: 1, name: 'Laptop', price: 80000 },
  { id: 2, name: 'Mouse', price: 2000 },
  { id: 3, name: 'Keyboard', price: 5000 }
]);

searchTerm = signal('');
```

Instead of storing:

```ts
filteredProducts = ...
```

and manually synchronizing it, use:

```ts
filteredProducts = computed(() => {
  const term = this.searchTerm().toLowerCase();

  return this.products().filter(product =>
    product.name.toLowerCase().includes(term)
  );
});
```

Template:

```html
<input
  [value]="searchTerm()"
  (input)="searchTerm.set($any($event.target).value)"
>

@for (product of filteredProducts(); track product.id) {
  <p>{{ product.name }}</p>
}
```

Now:

```text
searchTerm changes
       ↓
filteredProducts invalidated
       ↓
next read recalculates
       ↓
UI gets new filtered result
```

This is a much cleaner model than manually keeping two pieces of state synchronized.

---

# 14. `effect()` — React to Signal Changes

`effect()` is different from `computed()`.

### `computed()`

Used for:

> **Deriving a value**

### `effect()`

Used for:

> **Performing a side effect when signals change**

Example:

```ts
import { effect, signal } from '@angular/core';

export class UserComponent {

  name = signal('Rishabh');

  constructor() {

    effect(() => {
      console.log('Name:', this.name());
    });
  }
}
```

The effect:

1. Runs initially.
2. Reads `name()`.
3. Angular tracks `name` as a dependency.
4. When `name` changes, the effect runs again.

Effects always run at least once and track the signals read during their execution. ([angular.dev][3])

---

# 15. `computed()` vs `effect()`

This is **one of the most important Signals interview questions**.

| `computed()`             | `effect()`                                   |
| ------------------------ | -------------------------------------------- |
| Derives a value          | Performs a side effect                       |
| Returns a signal         | Returns an `EffectRef`                       |
| Read-only                | Executes code                                |
| Lazy                     | Runs at least once                           |
| Memoized                 | Not a derived-value cache                    |
| Prefer for derived state | Use for imperative/non-reactive side effects |

### Use `computed()`

```ts
fullName = computed(() =>
  `${this.firstName()} ${this.lastName()}`
);
```

### Use `effect()`

```ts
effect(() => {
  localStorage.setItem('theme', this.theme());
});
```

Angular explicitly recommends preferring `computed()` for derived state and using effects primarily to synchronize signal state with imperative/non-reactive APIs. ([angular.dev][3])

---

# 16. Don't Use `effect()` to Propagate State

This is a **very important gotcha**.

Avoid:

```ts
count = signal(10);

doubleCount = signal(0);

constructor() {
  effect(() => {
    this.doubleCount.set(this.count() * 2);
  });
}
```

This is usually a bad design.

Why?

You now have two pieces of state:

```text
count
doubleCount
```

and you're manually synchronizing them.

Instead:

```ts
count = signal(10);

doubleCount = computed(() => this.count() * 2);
```

Now:

```text
count
  ↓
computed
  ↓
doubleCount
```

One source of truth.

Angular warns against using effects for state propagation because it can cause unnecessary change-detection cycles, circular updates, or `ExpressionChangedAfterItHasBeenChecked` errors. ([angular.dev][3])

---


Question ko thoda more clear aur professional way me aise likh sakte ho:

> **Angular Signals me `effect()` ke regarding do questions hain:**
>
> 1. Kya `effect()` ke andar kisi signal ki **previous/old value** aur **current/new value** dono access kar sakte hain? Agar haan, to old value kaise retrieve karte hain?
>
> 2. React ke `useEffect()` me hum dependency array pass karte hain, jaise `useEffect(() => {}, [count])`, jisse effect specifically `count` ko track karta hai aur `count` change hone par hi execute hota hai. **Kya Angular ke `effect()` me bhi similar dependency tracking hoti hai?** Agar ek component me multiple signals hain, to kya Angular automatically sirf unhi signals ko track karta hai jo `effect()` ke andar read kiye gaye hain, ya `effect()` component ke sabhi signals ko track karta hai?


## 1. Kya `effect()` me old value directly milti hai?

**Nahi.** Angular ka `effect()` callback React ke `useEffect` ki tarah previous value argument ke form me provide nahi karta.

```ts
effect(() => {
  console.log(this.count());
});
```

Yahan `effect()` ko directly `oldValue` / `previousValue` nahi milta.

Agar old value chahiye, to manually track karna padega:

```ts
count = signal(0);

private previousCount = this.count();

constructor() {
  effect(() => {
    const currentCount = this.count();

    console.log('Previous:', this.previousCount);
    console.log('Current:', currentCount);

    this.previousCount = currentCount;
  });
}
```

Lekin **important:** pehli execution me old/current distinction carefully handle karna padega, kyunki effect **at least once initially run hota hai**.

---

# 2. Ab tumhara main question — kya Angular me `effect` sab signals ko track karta hai?

**Nahi!**

Ye bahut important concept hai:

> `effect()` sirf un signals ko track karta hai **jinhe effect ke andar read kiya gaya hai**.

Example:

```ts
count = signal(0);
name = signal('Rishabh');
age = signal(35);

constructor() {
  effect(() => {
    console.log(this.count());
    console.log(this.name());
  });
}
```

Angular automatically dependencies identify karega:

```text
effect
 ├── count ✅ tracked
 ├── name  ✅ tracked
 └── age   ❌ not tracked
```

So:

```ts
this.count.set(10);
```

➡️ effect runs

```ts
this.name.set('Rahul');
```

➡️ effect runs

```ts
this.age.set(36);
```

➡️ **effect nahi chalega**

Because `age()` effect ke andar read hi nahi hua.

Angular officially isi behavior ko **dynamic dependency tracking** kehta hai.

---

# 3. React `useEffect` vs Angular `effect()`

Tumhare React background ke hisaab se ye comparison yaad rakhna best rahega:

### React

```ts
useEffect(() => {
  console.log(count);
}, [count]);
```

Tum explicitly bolte ho:

> `count` meri dependency hai.

### Angular

```ts
effect(() => {
  console.log(count());
});
```

Tum dependency list **nahi dete**.

Angular khud dekhta hai:

> Effect ke andar kaun-kaun se signals read hue?

Aur unhe dependency bana deta hai.

So roughly:

```text
React

useEffect(() => {
   ...
}, [count, name])


Angular

effect(() => {
   count();
   name();
})
```

Conceptually dono ka result similar hai, **but Angular dependency list explicitly nahi leta.**

---

# 4. Ye aur interesting ho jata hai dynamic conditions ke saath

Suppose:

```ts
showCount = signal(true);
count = signal(0);
name = signal('Rishabh');

effect(() => {
  if (this.showCount()) {
    console.log(this.count());
  } else {
    console.log(this.name());
  }
});
```

Initially:

```text
showCount = true
```

Angular dependencies:

```text
showCount ✅
count     ✅
name      ❌
```

Ab:

```ts
this.count.set(10);
```

➡️ effect runs.

Lekin:

```ts
this.name.set('Rahul');
```

➡️ effect **nahi** chalega.

Now:

```ts
this.showCount.set(false);
```

Effect reruns and dependency tracking becomes:

```text
showCount ✅
count     ❌
name      ✅
```

Ab:

```ts
this.name.set('Amit');
```

➡️ effect runs.

And:

```ts
this.count.set(20);
```

➡️ effect **nahi** chalega.

Ye dependency tracking **dynamic** hai. Angular har execution me dekhta hai ki kaunse signals actually read hue.

---

# 5. Aur agar effect ke andar kisi signal ko track nahi karna ho?

Angular me `untracked()` hai.

Example:

```ts
effect(() => {
  console.log(this.user());
  console.log(untracked(this.counter));
});
```

Ab dependency:

```text
user      ✅ tracked
counter   ❌ NOT tracked
```

Even though `counter()` read ho raha hai, Angular usko dependency nahi banayega.

Ye kuch situations me kaafi useful hota hai.

---

# 6. Ek important correction regarding "old value"

Agar tumhara actual requirement hai:

> "Jab signal change ho to mujhe previous aur current value compare karni hai"

to har baar `effect()` best solution nahi hota.

For example:

```ts
count = signal(0);

effect(() => {
  const current = this.count();
  // compare with previous...
});
```

Manually previous value maintain karna possible hai, but Angular Signals ke ecosystem me **`effect()` primarily side effects ke liye hai**, state transformation/derived state ke liye nahi. Angular itself recommends avoiding effects for propagating state changes and using `computed()` for derived state instead.

---

## Tumhare liye simple mental model

React se Angular Signals par shift karte waqt ye yaad rakho:

```text
signal()
   ↓
reactive state

computed()
   ↓
derived state

effect()
   ↓
side effect
```

Aur `effect()` ka dependency model:

```text
effect(() => {

   signalA();  ← tracked
   signalB();  ← tracked

   // signalC() is not read
   // so NOT tracked

});
```

Therefore:

> **Angular me ek global effect nahi hota jo component ke saare signals ko watch kare. Har `effect()` ka apna dynamically discovered dependency set hota hai.**

Aur haan, **ek component me multiple effects bhi bana sakte ho**, exactly different concerns ke liye:

```ts
effect(() => {
  // user-related side effect
  console.log(this.user());
});

effect(() => {
  // theme-related side effect
  console.log(this.theme());
});
```

Pehla `user` ko track karega, doosra `theme` ko.

**One-liner interview answer:**

> *Angular `effect()` doesn't take an explicit dependency array like React's `useEffect`. It automatically tracks every signal read during its execution, and dependencies are dynamically updated on subsequent executions.*
