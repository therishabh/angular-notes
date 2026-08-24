# Components & Component Metadata

## 1. Core Concept

### What is a Component?

A **component** is the fundamental building block of an Angular application.

A component typically contains:

* **TypeScript class** → component logic/state
* **HTML template** → UI
* **CSS/SCSS** → component-specific styling
* **Metadata** → tells Angular how to treat the class as a component

Conceptually:

```text
Component
 ├── Class       → state + behavior
 ├── Template    → UI
 ├── Styles      → presentation
 └── Metadata    → Angular configuration
```

Example:

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-user',
  template: `
    <h2>{{ name }}</h2>
    <button (click)="changeName()">Change</button>
  `,
  styles: `
    h2 {
      color: blue;
    }
  `
})
export class UserComponent {
  name = 'Rishabh';

  changeName() {
    this.name = 'Angular Developer';
  }
}
```

Here:

* `UserComponent` → component class
* `@Component(...)` → component decorator
* `selector` → how Angular identifies the component in HTML
* `template` → UI
* `styles` → component styles

---

## 2. How It Works

### `@Component` Metadata

`@Component()` is a decorator that provides Angular with the metadata required to create and configure a component.

```ts
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss'
})
export class UserComponent {}
```

Think of metadata as:

> **"Angular, treat this class as a component and configure it this way."**

---

### Important Component Metadata

| Metadata          | Purpose                                                                                                                                             |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `selector`        | Defines how the component is used in templates                                                                                                      |
| `template`        | Inline HTML template                                                                                                                                |
| `templateUrl`     | External HTML template                                                                                                                              |
| `styles`          | Inline CSS                                                                                                                                          |
| `styleUrl`        | External stylesheet                                                                                                                                 |
| `styleUrls`       | External stylesheets; commonly seen in older/current codebases                                                                                      |
| `imports`         | Dependencies available to a standalone component                                                                                                    |
| `standalone`      | Explicitly marks a component as standalone in Angular versions where this metadata is relevant; modern Angular components are standalone by default |
| `changeDetection` | Configures change-detection strategy                                                                                                                |
| `encapsulation`   | Controls CSS encapsulation                                                                                                                          |
| `providers`       | Registers providers scoped to the component                                                                                                         |
| `host`            | Configures host element bindings/listeners                                                                                                          |
| `inputs`          | Input configuration; decorators or `input()` are generally preferred in modern code                                                                 |
| `outputs`         | Output configuration; decorators or `output()` are generally preferred in modern code                                                               |

Don't memorize every metadata property. For interviews, understand the **commonly used and architecturally important** ones.

---

### `selector`

Defines how the component is represented in another template.

#### Element selector — most common

```ts
@Component({
  selector: 'app-user'
})
export class UserComponent {}
```

Usage:

```html
<app-user></app-user>
```

#### Attribute selector

```ts
@Component({
  selector: '[appHighlight]'
})
export class HighlightComponent {}
```

Usage:

```html
<div appHighlight></div>
```

#### Class selector

```ts
@Component({
  selector: '.app-card'
})
```

Usage:

```html
<div class="app-card"></div>
```

**Best practice:** Prefer element selectors for components.

```html
<app-user></app-user>
```

Attribute selectors are more commonly appropriate for **directives**.

---

## 3. Practical Example

### Modern Standalone Component

Modern Angular applications generally use standalone components.

```ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-list',
  imports: [CommonModule],
  template: `
    @if (users.length) {
      <ul>
        @for (user of users; track user.id) {
          <li>{{ user.name }}</li>
        }
      </ul>
    } @else {
      <p>No users found.</p>
    }
  `
})
export class UserListComponent {
  users = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Sarah' }
  ];
}
```

Important Angular-specific points:

* `@Component()` defines the component.
* `imports` declares template dependencies for the standalone component.
* Modern Angular control flow uses `@if` / `@for`.
* `track user.id` helps Angular efficiently update the rendered list.

> Note: In modern Angular, standalone components are the default. Older Angular applications commonly use `NgModule`-based components.

---

### External Template and Styles

For larger components:

```ts
@Component({
  selector: 'app-user-profile',
  templateUrl: './user-profile.component.html',
  styleUrl: './user-profile.component.scss'
})
export class UserProfileComponent {}
```

This keeps the component class focused on logic while HTML/CSS live in separate files.

---

### Component Inputs

A component can receive data from its parent.

Modern Angular:

```ts
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <h3>{{ name() }}</h3>
  `
})
export class UserCardComponent {
  name = input.required<string>();
}
```

Parent:

```html
<app-user-card [name]="user.name" />
```

`input()` is the modern signal-based input API.

Older/common approach:

```ts
@Input() name!: string;
```

**Modern → Older → Key difference**

```text
input()       → @Input()
output()      → @Output()
```

Modern APIs integrate naturally with Angular's signal-based reactivity model.

---


## 4. Interview Questions

### Q1. What is an Angular component?

**Answer:**
A component is a UI building block consisting of a TypeScript class, template, styles, and Angular metadata. It controls a specific part of the application's UI.

---

### Q2. What does `@Component()` do?

**Answer:**
It is an Angular decorator that provides metadata describing how Angular should create, render, and configure the component.

---

### Q3. What is component metadata?

**Answer:**
Metadata is configuration supplied to Angular through `@Component()`, such as `selector`, template, styles, imports, providers, encapsulation, and change-detection strategy.

---

### Q4. What is the difference between `template` and `templateUrl`?

**Answer:**

```ts
template: `<h1>Hello</h1>`
```

uses an inline template.

```ts
templateUrl: './user.component.html'
```

loads the template from a separate file.

For non-trivial components, external templates are generally easier to maintain.

---

### Q5. What is the difference between `styles` and `styleUrl`?

**Answer:**

```ts
styles: `h1 { color: red; }`
```

defines inline styles.

```ts
styleUrl: './user.component.scss'
```

references an external stylesheet.

---

### Q6. What is the purpose of `selector`?

**Answer:**
It defines how the component is referenced in Angular templates.

```ts
selector: 'app-user'
```

allows:

```html
<app-user />
```

---

### Q7. Component vs Directive?

**Answer:**
A component is a directive that has its own template. A directive primarily adds behavior or modifies an existing element/component.

---

### Q8. What is standalone Angular?

**Answer:**
Standalone components allow Angular components to manage their own template dependencies without requiring an `NgModule` declaration. Modern Angular uses standalone components by default.

---

### Q9. Why is `imports` present inside `@Component()`?

**Answer:**
For standalone components, `imports` declares the directives, pipes, components, and other template dependencies that the component's template can use.

---

### Q10. What happens if a required template dependency isn't imported?

**Answer:**
Angular may report a template compilation/diagnostic error because the template doesn't know about the referenced directive, pipe, component, etc.

---

### Q11. What is view encapsulation?

**Answer:**
It controls how component styles are scoped. Angular supports `Emulated`, `None`, and `ShadowDom`.

---

### Q12. What is the default view encapsulation?

**Answer:**
`ViewEncapsulation.Emulated` is the default.

---

### Q13. When would you use component-level providers?

**Answer:**
When you want a service instance scoped to a component or component subtree rather than the application's root injector.

---

### Q14. Why use `ChangeDetectionStrategy.OnPush`?

**Answer:**
It allows Angular to use a more optimized change-detection strategy, reducing unnecessary checks when the component's relevant state hasn't changed.

---

### Q15. Scenario: A component has become 1,000 lines long. What would you do?

**Answer:**

* Move business logic into services/facades.
* Extract reusable UI into child components.
* Separate complex state management.
* Keep the component responsible primarily for UI orchestration.
* Avoid creating components purely for arbitrary code splitting; extract based on clear responsibilities.

---

### Q16. Scenario: Two component instances should not share service state. How would you achieve it?

**Answer:**
Provide the service at the component level:

```ts
@Component({
  providers: [MyService]
})
```

Each component instance gets its own service instance within its injector scope.

---

### Q17. Modern Angular uses signals. Does that mean components no longer need change detection?

**Answer:**
No. Signals provide reactive state and fine-grained dependency tracking, but Angular still has a rendering/change-detection system. Signals change how state changes can be tracked efficiently; they don't eliminate Angular's rendering mechanism.

---

### Must-remember interview points

* `@Component()` → component metadata.
* `selector` → how the component is used.
* `template/templateUrl` → component UI.
* `styles/styleUrl` → component styling.
* Modern Angular → **standalone components by default**.
* `imports` → template dependencies for standalone components.
* `providers` → dependency-injection scope.
* `encapsulation` → CSS isolation strategy.
* `changeDetection` → rendering/checking strategy.
* Components are UI-focused; business logic should generally live outside them.
* Know **modern standalone APIs** but recognize **NgModule-based code** in older projects.

