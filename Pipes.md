Absolutely. Pipes are a very important Angular topic, especially **pure vs impure pipes**, **custom pipes**, and understanding when a pipe is appropriate versus `computed()` or a normal TypeScript function.

I’ve also checked the current Angular documentation so the notes below follow modern Angular behavior. ([Angular][1])

# Angular Pipes

## 1. Core Concept

### What is a Pipe?

A **pipe is a reusable transformation mechanism used in Angular templates**.

It takes a value, transforms it, and returns another value for display.

Basic syntax:

```html
{{ value | pipeName }}
```

Example:

```ts
price = 12500;
```

```html
<p>{{ price | currency }}</p>
```

The pipe transforms:

```text
12500
  ↓
currency pipe
  ↓
₹12,500.00   // depending on locale/configuration
```

The key idea:

> **Pipe = declarative data transformation in the template.**

Angular provides many built-in pipes, and you can create your own custom pipes. ([Angular][1])

---

# 2. How It Works

A pipe conceptually behaves like a transformation function:

```text
Input
  ↓
Pipe
  ↓
Transformed Output
```

For example:

```html
{{ username | uppercase }}
```

Conceptually:

```text
"rishabh"
    ↓
uppercase
    ↓
"RISHABH"
```

The original value is **not modified**. The pipe produces a transformed value for the template.

---

## Basic syntax

```html
{{ value | pipeName }}
```

With parameters:

```html
{{ value | pipeName:parameter }}
```

Multiple parameters:

```html
{{ value | pipeName:param1:param2 }}
```

Multiple pipes can be chained:

```html
{{ value | pipe1 | pipe2 }}
```

Angular evaluates chained pipes **left to right**. ([Angular][1])

Example:

```html
{{ name | lowercase | titlecase }}
```

Flow:

```text
"RISHABH"
    ↓ lowercase
"rishabh"
    ↓ titlecase
"Rishabh"
```

---

# 3. Built-in Pipes

Angular provides built-in pipes mainly through `@angular/common`. ([Angular][1])

Some important ones:

| Pipe        | Purpose                                             |
| ----------- | --------------------------------------------------- |
| `uppercase` | Converts text to uppercase                          |
| `lowercase` | Converts text to lowercase                          |
| `titlecase` | Converts text to title case                         |
| `date`      | Formats dates                                       |
| `currency`  | Formats currency                                    |
| `number`    | Formats numbers                                     |
| `percent`   | Formats percentages                                 |
| `json`      | Converts value to JSON string, mainly for debugging |
| `slice`     | Returns a slice of an array/string                  |
| `keyvalue`  | Converts object/Map entries into key-value pairs    |
| `async`     | Reads values from `Observable`/`Promise`            |

---

# 4. Using Built-in Pipes

## `uppercase`

```html
<p>{{ name | uppercase }}</p>
```

```ts
name = 'rishabh';
```

Output:

```text
RISHABH
```

---

## `lowercase`

```html
<p>{{ name | lowercase }}</p>
```

Output:

```text
rishabh
```

---

## `titlecase`

```html
<p>{{ name | titlecase }}</p>
```

Example:

```text
"hello angular developer"
          ↓
"Hello Angular Developer"
```

---

# 5. Pipe Parameters

Pipes can accept additional parameters.

Syntax:

```html
{{ value | pipeName: parameter }}
```

Example:

```html
{{ birthday | date:'medium' }}
```

Another example:

```html
{{ price | currency:'INR' }}
```

Conceptually:

```text
price
  ↓
currency
  ↓
INR configuration
  ↓
formatted currency
```

Angular uses `:` to pass parameters to a pipe. ([Angular][1])

---

# 6. Multiple Parameters

Example:

```html
{{ dateValue | date:'hh:mm':'UTC' }}
```

Here:

```text
dateValue
   ↓
date pipe
   ↓
'hh:mm'   → format
'UTC'     → timezone
```

The exact parameters depend on the pipe.

---

# 7. Pipe Chaining

You can use multiple pipes:

```html
{{ name | lowercase | titlecase }}
```

Execution:

```text
name
 ↓
lowercase
 ↓
titlecase
 ↓
final output
```

Another example:

```html
{{ scheduledOn | date | uppercase }}
```

The result of `date` becomes the input of `uppercase`. ([Angular][1])

---

# 8. Pipes Are Template-Level Transformations

Consider:

```ts
fullName = 'rishabh sharma';
```

Instead of:

```ts
formattedName = this.formatName(fullName);
```

you can write:

```html
{{ fullName | titlecase }}
```

This makes the transformation declarative:

```text
Template
   ↓
"This value should be displayed as title case"
   ↓
titlecase pipe
```

That's one of the main reasons Angular has pipes.

---

# 9. `AsyncPipe` — Very Important

`AsyncPipe` is especially important for Angular interviews.

It can consume:

* `Observable`
* `Promise`

Example:

```ts
users$ = this.userService.getUsers();
```

Template:

```html
@for (user of users$ | async; track user.id) {
  <p>{{ user.name }}</p>
}
```

The `async` pipe:

* subscribes to an Observable / waits for a Promise
* provides the latest emitted/resolved value to the template
* manages subscription cleanup when the component is destroyed

This is one of the most useful built-in pipes in real Angular applications. ([Angular][1])

> Note: Modern Angular also has signal-based APIs for working with reactive state, but `AsyncPipe` remains important when working with Observables/Promises.

---

# 10. `JsonPipe`

Useful while debugging:

```html
<pre>{{ user | json }}</pre>
```

If:

```ts
user = {
  id: 1,
  name: 'Rishabh'
};
```

you get a readable JSON representation.

It is intended mainly for debugging, not normal production formatting. ([Angular][2])

---

# 11. Creating a Custom Pipe

Now the important part.

A custom pipe is simply a TypeScript class decorated with `@Pipe`.

Example:

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'capitalize'
})
export class CapitalizePipe implements PipeTransform {

  transform(value: string): string {
    if (!value) {
      return '';
    }

    return value.charAt(0).toUpperCase() + value.slice(1);
  }
}
```

Usage:

```html
<p>{{ name | capitalize }}</p>
```

If:

```ts
name = 'rishabh';
```

Output:

```text
Rishabh
```

A custom pipe requires:

1. `@Pipe()` decorator
2. `name`
3. `transform()` method

Angular recommends implementing `PipeTransform` for the expected pipe structure. ([Angular][1])

---

# 12. Understanding `@Pipe()`

```ts
@Pipe({
  name: 'capitalize'
})
```

The `name` is extremely important.

It determines what you write in the template:

```html
{{ name | capitalize }}
```

Not:

```html
{{ name | CapitalizePipe }}
```

Naming convention:

```text
Pipe name:
capitalize

Class name:
CapitalizePipe
```

Angular recommends camelCase for the pipe name and a PascalCase class name ending in `Pipe`. ([Angular][1])

---

# 13. `transform()` Method

This is where your transformation logic lives.

```ts
transform(value: string): string {
  return value.toUpperCase();
}
```

Conceptually:

```text
Template
   ↓
{{ value | capitalize }}
   ↓
transform(value)
   ↓
returned value
   ↓
rendered UI
```

---

# 14. Custom Pipe with Parameters

Suppose we want:

```html
{{ username | truncate:20 }}
```

Custom pipe:

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate'
})
export class TruncatePipe implements PipeTransform {

  transform(value: string, limit: number): string {

    if (!value) {
      return '';
    }

    if (value.length <= limit) {
      return value;
    }

    return value.slice(0, limit) + '...';
  }
}
```

Usage:

```html
<p>
  {{ description | truncate:30 }}
</p>
```

The mapping is:

```text
{{ description | truncate:30 }}

description
     ↓
transform(
    description,
    30
)
```

So the first `transform()` parameter is always the value flowing into the pipe, and additional parameters correspond to values after `:`. ([Angular][1])

---

# 15. Custom Pipe with Multiple Parameters

Example:

```html
{{ price | priceFormat:'INR':2 }}
```

Pipe:

```ts
transform(
  value: number,
  currency: string,
  decimals: number
): string {

  // transformation logic
}
```

Mapping:

```text
value     → price
currency  → 'INR'
decimals  → 2
```

---

# 16. Standalone Custom Pipe — Modern Angular

Modern Angular applications commonly use standalone pipes.

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'capitalize',
  standalone: true
})
export class CapitalizePipe implements PipeTransform {

  transform(value: string): string {
    return value
      ? value.charAt(0).toUpperCase() + value.slice(1)
      : '';
  }
}
```

Then import it into the component:

```ts
@Component({
  selector: 'app-user',
  imports: [CapitalizePipe],
  template: `
    <p>{{ name | capitalize }}</p>
  `
})
export class UserComponent {
  name = 'rishabh';
}
```

Modern Angular's CLI generates standalone pipes by default. ([Angular][3])

---

# 17. Pipe vs Function in Template

This is a very important practical decision.

You could write:

```html
{{ formatName(name) }}
```

or:

```html
{{ name | capitalize }}
```

For a reusable **template transformation**, a pipe is generally more expressive:

```html
{{ name | capitalize }}
```

It communicates:

> "This is presentation formatting."

But don't automatically turn every function into a pipe.

Use a pipe when:

* transformation is presentation-oriented
* reusable across templates
* preferably stateless/pure
* easy to understand as a template transformation

Use a normal TypeScript function/service when the logic is business logic or needs to be reused outside templates.

---

# 18. Pipe vs `computed()`

This is particularly relevant because you've just learned Signals.

Suppose:

```ts
price = signal(100);
quantity = signal(2);
```

You want total:

```ts
total = computed(() =>
  this.price() * this.quantity()
);
```

**Don't create a pipe for this.**

Why?

`total` is **application/component derived state**, not merely display formatting.

Use:

```text
computed()
```

for:

> Derived reactive state.

Use:

```text
pipe
```

for:

> Template-oriented transformation/formatting.

### Example

Good `computed()`:

```ts
total = computed(() =>
  this.price() * this.quantity()
);
```

Good pipe:

```html
{{ total() | currency }}
```

Here:

```text
computed()
   ↓
business/application derived value

currency pipe
   ↓
display formatting
```

That's a very clean separation.

---

# 19. Pure Pipes — Extremely Important

By default, Angular pipes are **pure**.

```ts
@Pipe({
  name: 'capitalize'
})
```

is effectively:

```ts
@Pipe({
  name: 'capitalize',
  pure: true
})
```

A pure pipe runs when its input changes according to Angular's pure-pipe checking.

For primitive values, this means the value changes.

For objects/arrays/functions/dates, Angular checks the **reference**. ([Angular][1])

---

# 20. Why Pure Pipes Are Fast

Suppose:

```html
{{ name | capitalize }}
```

If `name` hasn't changed, Angular doesn't need to repeatedly execute the pipe transformation.

Conceptually:

```text
Input unchanged
      ↓
Don't recalculate
      ↓
Reuse previous result
```

This can provide performance benefits. ([Angular][1])

---

# 21. Pure Pipe + Object Mutation Gotcha

This is one of the most important pipe interview questions.

Suppose:

```ts
user = {
  name: 'John'
};
```

Pipe:

```html
{{ user | myPipe }}
```

Now you mutate:

```ts
this.user.name = 'Rishabh';
```

The object reference is still:

```text
same object
```

So a pure pipe may **not run again**, because Angular sees the same object reference. ([Angular][1])

Instead, create a new object:

```ts
this.user = {
  ...this.user,
  name: 'Rishabh'
};
```

Now:

```text
old object reference
        ≠
new object reference
```

and Angular can detect the input reference change.

---

# 22. Pure vs Impure Pipes

You can explicitly create an impure pipe:

```ts
@Pipe({
  name: 'myPipe',
  pure: false
})
```

### Pure

```text
pure: true
```

Runs when Angular detects a relevant input/reference change.

### Impure

```text
pure: false
```

Angular can run it much more frequently during change detection.

This makes impure pipes potentially expensive.

Angular explicitly recommends avoiding impure pipes unless they are truly necessary. ([Angular][1])

---

# 23. Example of an Impure Pipe

Suppose:

```ts
users = [
  { name: 'John' },
  { name: 'Sarah' }
];
```

You mutate the array:

```ts
this.users.push({
  name: 'Mike'
});
```

The array reference hasn't changed.

A pure pipe receiving that array may not detect the internal mutation.

An impure pipe could detect changes within the array.

But:

> **Don't immediately choose an impure pipe as the solution.**

Prefer immutable updates when possible:

```ts
this.users = [
  ...this.users,
  { name: 'Mike' }
];
```

Now the array reference changes and a pure pipe can react appropriately.

---

# 24. Why Impure Pipes Can Hurt Performance

Imagine:

```html
@for (user of users | expensiveFilter; track user.id) {
  ...
}
```

If `expensiveFilter` is impure, Angular may execute it frequently.

For a large array:

```text
10,000 users
      ↓
expensive transformation
      ↓
repeated many times
      ↓
performance problem
```

Therefore:

> **Pure pipes should be the default choice.**

And for expensive derived collections, consider whether the logic belongs in:

* `computed()`
* component state
* a service
* server-side filtering
* another appropriate architecture

depending on the use case.

---

# 25. Pipe Precedence

Angular's pipe operator has specific precedence rules.

For example:

```html
{{ firstName + lastName | uppercase }}
```

The concatenation happens before the pipe:

```text
firstName + lastName
        ↓
uppercase
```

If the expression is complex, use parentheses to make the intention obvious:

```html
{{ (isAdmin ? 'Access granted' : 'Access denied') | uppercase }}
```

This is especially important with ternary expressions because pipe precedence can be surprising. ([Angular][1])

---

# 26. Pipes Should Usually Be Side-Effect Free

A good pipe:

```ts
transform(value: string): string {
  return value.toUpperCase();
}
```

A problematic pipe:

```ts
transform(value: string): string {
  this.someService.save(value);
  return value.toUpperCase();
}
```

Why?

Pipes are intended to **transform values**, not perform application side effects.

Think:

```text
Input
 ↓
Transformation
 ↓
Output
```

not:

```text
Input
 ↓
API call
 ↓
database update
 ↓
global state mutation
 ↓
Output
```

For side effects, use appropriate Angular services/effects/application logic instead.

---

# 27. Don't Use Pipes for Business Logic

Bad:

```html
{{ order | calculateTax | calculateDiscount | calculateFinalPrice }}
```

This makes important business rules hidden inside presentation logic.

Prefer:

```ts
finalPrice = computed(() => {
  // business/domain calculation
});
```

Then:

```html
{{ finalPrice() | currency }}
```

Now:

```text
Business logic
     ↓
computed/service
     ↓
final value
     ↓
pipe
     ↓
display formatting
```

Much cleaner.

---

# 28. Reusing Custom Pipe Logic Outside Templates

Suppose you create:

```ts
@Pipe({
  name: 'kebabCase'
})
export class KebabCasePipe implements PipeTransform {
  transform(value: string): string {
    return value.toLowerCase().replace(/ /g, '-');
  }
}
```

Don't make a service depend on this pipe just to reuse the transformation.

Angular recommends extracting reusable transformation logic into a standalone function and using that function both from the pipe and elsewhere. ([Angular][1])

Example:

```ts
export function toKebabCase(value: string): string {
  return value.toLowerCase().replace(/ /g, '-');
}
```

Pipe:

```ts
@Pipe({
  name: 'kebabCase'
})
export class KebabCasePipe implements PipeTransform {

  transform(value: string): string {
    return toKebabCase(value);
  }
}
```

Service:

```ts
export class FormatterService {

  formatSlug(title: string): string {
    return toKebabCase(title);
  }
}
```

This gives you:

```text
Reusable logic
      ↓
Standalone function
   ↙       ↘
Pipe      Service
```

This is a good architectural pattern. ([Angular][1])

---

# 29. Built-in Pipe vs Custom Pipe

| Built-in Pipe       | Custom Pipe                 |
| ------------------- | --------------------------- |
| Provided by Angular | Created by your application |
| `currency`          | `truncate`                  |
| `date`              | `capitalize`                |
| `uppercase`         | `fileSize`                  |
| `async`             | `statusLabel`               |
| `json`              | `phoneFormat`               |

Use a built-in pipe whenever it already solves your requirement.

Don't create:

```text
MyCurrencyPipe
MyDatePipe
MyUppercasePipe
```

if Angular's built-in pipe already handles the requirement.

---

# 30. Testing Custom Pipes

Pipes are usually very easy to unit test because the core logic is in:

```ts
transform()
```

Example:

```ts
describe('CapitalizePipe', () => {

  const pipe = new CapitalizePipe();

  it('should capitalize the first character', () => {
    expect(pipe.transform('rishabh'))
      .toBe('Rishabh');
  });
});
```

You don't necessarily need Angular's full testing environment for a simple pure pipe. Angular's documentation also recommends testing simple pipes directly because most pipes have little/no DOM dependency. ([Angular][4])

---

# 31. Common Mistakes / Gotchas

### 1. Forgetting to import the pipe

With standalone components:

```ts
@Component({
  imports: [CapitalizePipe]
})
```

Without making the pipe available to the template, Angular can't use it.

---

### 2. Creating an impure pipe unnecessarily

Avoid:

```ts
@Pipe({
  name: 'filter',
  pure: false
})
```

unless you genuinely need impure behavior.

For large collections, this can become expensive.

---

### 3. Mutating arrays/objects used by pure pipes

Avoid:

```ts
this.users.push(user);
```

when relying on reference-based change detection.

Prefer:

```ts
this.users = [
  ...this.users,
  user
];
```

---

### 4. Putting business logic inside pipes

A pipe should generally be presentation-oriented.

---

### 5. Using a pipe for expensive application state derivation

If you have:

```ts
products = signal(...);
searchTerm = signal(...);
```

prefer:

```ts
filteredProducts = computed(() => ...);
```

and then optionally use a pipe for final formatting.

---

### 6. Calling an expensive function from the template instead of using an appropriate abstraction

For repeated template transformations, a pure pipe can sometimes be a better fit than repeatedly calling a method.

But don't use a pipe just to hide expensive business logic.

Choose based on whether the transformation is **presentation-oriented and reusable**.

---

# 32. Interview Questions

### Q1. What is a pipe in Angular?

**Answer:**

> A pipe is a declarative template transformation mechanism used to transform a value for display. Angular provides built-in pipes and allows developers to create custom pipes.

---

### Q2. What is the syntax for using a pipe?

```html
{{ value | pipeName }}
```

With parameters:

```html
{{ value | pipeName:param1:param2 }}
```

---

### Q3. What is a custom pipe?

**Answer:**

> A custom pipe is a TypeScript class decorated with `@Pipe`, containing a `transform()` method that implements the desired value transformation.

---

### Q4. What is `PipeTransform`?

**Answer:**

`PipeTransform` is an Angular interface that defines the expected `transform()` method structure for a pipe.

```ts
export class MyPipe implements PipeTransform {
  transform(value: string): string {
    return value;
  }
}
```

---

### Q5. What does `@Pipe({ name: 'capitalize' })` do?

**Answer:**

It registers the pipe name used in templates:

```html
{{ name | capitalize }}
```

The TypeScript class name can be:

```ts
CapitalizePipe
```

but the template uses the configured `name`.

---

### Q6. What is a pure pipe?

**Answer:**

> A pure pipe is the default Angular pipe behavior. Angular re-executes it when the pipe's input value changes or, for objects/arrays, when the input reference changes.

Pure pipes allow Angular to avoid unnecessary transformation calls and therefore generally provide better performance. ([Angular][1])

---

### Q7. What is an impure pipe?

**Answer:**

A pipe configured with:

```ts
pure: false
```

can be executed much more frequently during change detection, allowing it to react to internal mutations that a pure pipe would not detect.

The trade-off is potentially significant performance cost. ([Angular][1])

---

### Q8. Why doesn't a pure pipe detect this?

```ts
this.users.push(newUser);
```

**Answer:**

Because the array reference remains the same.

Prefer:

```ts
this.users = [...this.users, newUser];
```

Now Angular sees a new array reference.

---

### Q9. Pure vs impure pipe?

| Pure                              | Impure                          |
| --------------------------------- | ------------------------------- |
| Default                           | `pure: false`                   |
| More performant                   | Potentially expensive           |
| Reacts to input/reference changes | Can react to internal mutations |
| Prefer this                       | Use only when necessary         |

---

### Q10. Can a pipe modify the original input?

Technically a pipe can mutate objects because TypeScript/JavaScript allows it, but this is generally a **bad practice**.

Prefer pipes to be pure transformations:

```text
Input
 ↓
Transform
 ↓
New/display value
```

rather than mutating application state.

---

### Q11. Pipe vs `computed()`?

**Answer:**

> Use `computed()` for derived reactive application state. Use a pipe for declarative presentation transformation in the template.

Example:

```ts
total = computed(() => this.price() * this.quantity());
```

Then:

```html
{{ total() | currency }}
```

---

### Q12. Pipe vs method in template?

**Answer:**

> A pipe is a reusable, declarative template transformation and pure pipes have Angular's optimized execution semantics. A method is general-purpose application code and can be evaluated repeatedly during rendering. Use the abstraction that matches the responsibility rather than blindly replacing every method with a pipe.

---

### Q13. Why is `AsyncPipe` important?

**Answer:**

> `AsyncPipe` lets templates consume Observable or Promise values and manages the subscription lifecycle, including cleanup when the component is destroyed.

---

### Q14. Can a pipe accept multiple arguments?

Yes:

```html
{{ value | myPipe:arg1:arg2 }}
```

Angular passes them to:

```ts
transform(value, arg1, arg2)
```

---

### Q15. Can you use a pipe inside TypeScript?

Technically a pipe class can be instantiated/called, but Angular recommends **not using pipe classes as general-purpose injectable services**. Extract reusable transformation logic into a standalone function and use that function from both the pipe and non-template code. ([Angular][1])

---

### Q16. Scenario: You need to filter 50,000 products based on a search term. Would you use an impure pipe?

**Answer:**

Usually no.

An impure pipe can execute frequently and become expensive.

Better options depend on the architecture:

```text
Search state
    ↓
computed() / appropriate state derivation
    ↓
filtered collection
    ↓
@for
```

For very large datasets, also consider server-side filtering/pagination or virtual scrolling.

---

### Q17. Scenario: You need to display `₹12,500` instead of `12500`. Pipe or service?

**Answer:**

Pipe.

This is a **presentation formatting** requirement:

```html
{{ price | currency:'INR' }}
```

---

### Q18. Scenario: You need to calculate order tax, discount and final payable amount. Pipe or computed/service?

**Answer:**

Prefer application/domain logic such as a service or `computed()`, depending on where the state lives.

Then use a pipe only for final display formatting.

---

# 33. Quick Revision

### Basic pipe

```html
{{ value | pipe }}
```

### Pipe with parameter

```html
{{ value | pipe:param }}
```

### Multiple parameters

```html
{{ value | pipe:param1:param2 }}
```

### Chaining

```html
{{ value | pipe1 | pipe2 }}
```

Executed:

```text
value
 ↓
pipe1
 ↓
pipe2
 ↓
output
```

---

### Custom pipe

```ts
@Pipe({
  name: 'capitalize'
})
export class CapitalizePipe implements PipeTransform {

  transform(value: string): string {
    return value.charAt(0).toUpperCase() + value.slice(1);
  }
}
```

Usage:

```html
{{ name | capitalize }}
```

---

### Most important concept: Pure vs Impure

```text
                 PIPE
                   |
          +--------+--------+
          |                 |
        Pure              Impure
      (default)         pure: false
          |                 |
  input/reference       can react to
     changes             mutations
          |                 |
      efficient          potentially
                         expensive
```

### The decision rule

```text
Is this presentation formatting?
        ↓
      PIPE

Is this derived reactive state?
        ↓
    computed()

Is this business/domain logic?
        ↓
   service / domain logic

Is this imperative side effect?
        ↓
    effect() / appropriate API
```

### The 5 things I'd definitely remember for interviews

1. **Pipe = declarative transformation in Angular templates.**
2. **Custom pipe = `@Pipe()` + `transform()`.**
3. **Pipes are pure by default.**
4. **Pure pipes depend on input/reference changes; object/array mutation can therefore be missed.**
5. **Avoid impure pipes unless genuinely necessary because they can have significant performance implications.** ([Angular][1])

[1]: https://angular.dev/guide/templates/pipes?utm_source=chatgpt.com "Pipes • Angular"
[2]: https://angular.dev/api/common/JsonPipe?utm_source=chatgpt.com "JsonPipe • Angular"
[3]: https://angular.dev/cli/generate/pipe?utm_source=chatgpt.com "pipe • Angular"
[4]: https://angular.dev/guide/testing/pipes?utm_source=chatgpt.com "Testing pipes • Angular"
