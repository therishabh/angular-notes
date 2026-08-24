# Property Binding in Angular

## 1. Core Concept

**Property binding** is used to bind a value from the component's TypeScript class to a **DOM property** or a property/input of an Angular component.

Syntax:

```html
[property]="expression"
```

Example:

```ts
export class UserComponent {
  imageUrl = '/images/user.png';
  isDisabled = true;
}
```

```html
<img [src]="imageUrl">

<button [disabled]="isDisabled">
  Submit
</button>
```

Here:

```text
[disabled]  → target property
isDisabled  → Angular expression
```

The important idea:

> **Property binding makes the DOM/component property dynamically follow your component state.**

---

## 2. How It Works

Consider:

```html
<button [disabled]="isDisabled">
  Submit
</button>
```

Angular evaluates:

```ts
isDisabled
```

and assigns the result to the button's **DOM `disabled` property**.

If:

```ts
isDisabled = true;
```

the button becomes disabled.

If later:

```ts
isDisabled = false;
```

Angular updates the button.

So the flow is:

```text
Component state
      ↓
  [property]
      ↓
DOM / Component property
```

### Important: Property vs Attribute

This is one of the most important things to understand.

```html
<button disabled="false">
```

This is **not the same** as:

```html
<button [disabled]="false">
```

Why?

HTML attributes are strings/presence-based, while DOM properties have actual runtime values.

With property binding:

```html
<button [disabled]="false">
```

Angular assigns:

```ts
button.disabled = false;
```

So the button is enabled.

---

## 3. Practical Example

### Basic DOM Property Binding

```ts
export class ProfileComponent {
  username = 'Rishabh';
  imageUrl = '/assets/profile.png';
  isDisabled = true;
}
```

```html
<h2>{{ username }}</h2>

<img [src]="imageUrl">

<button [disabled]="isDisabled">
  Edit Profile
</button>
```

Different properties:

```text
[src]       → img.src
[disabled]  → button.disabled
[value]     → input.value
[checked]   → checkbox.checked
```

---

### Binding an Expression

The right-hand side doesn't have to be a simple property.

```html
<button [disabled]="users.length === 0">
  Continue
</button>
```

Angular evaluates:

```ts
users.length === 0
```

and assigns the resulting boolean to `disabled`.

You can also call a method:

```html
<img [src]="getProfileImage()">
```

But be careful with methods that are expensive or have side effects because Angular may evaluate template expressions frequently.

---

### Binding Component Inputs

Property binding isn't limited to native DOM properties.

Suppose:

```ts
@Component({
  selector: 'app-user-card',
  template: `{{ name }}`
})
export class UserCardComponent {
  name = input.required<string>();
}
```

Parent:

```ts
export class AppComponent {
  userName = 'Rishabh';
}
```

```html
<app-user-card [name]="userName" />
```

Here:

```text
[name]
   ↓
UserCardComponent's input
   ↓
userName
```

This is still property-style binding, but the target is an **Angular component input** rather than a native DOM property.

---

## 4. Important Points

### Property binding vs interpolation

Both can display values:

```html
<img src="{{ imageUrl }}">
```

and:

```html
<img [src]="imageUrl">
```

But they are conceptually different.

| Interpolation                     | Property Binding                  |
| --------------------------------- | --------------------------------- |
| `{{ value }}`                     | `[property]="value"`              |
| Primarily converts result to text | Assigns value to property         |
| Commonly used for text            | Used for DOM/component properties |
| `src="{{ imageUrl }}"`            | `[src]="imageUrl"`                |

### Rule of thumb

For text:

```html
<h2>{{ username }}</h2>
```

For properties:

```html
<img [src]="imageUrl">
<button [disabled]="isDisabled">
```

Don't think of interpolation as simply "old property binding." They are different template mechanisms.

---

### Property Binding vs Attribute Binding

This is **very important for interviews**.

Property:

```html
<button [disabled]="isDisabled">
```

Attribute:

```html
<button [attr.aria-label]="label">
```

Property binding targets a DOM/component **property**.

Attribute binding manipulates an HTML **attribute**.

Use attribute binding when you specifically need an attribute:

```html
<td [attr.colspan]="columnSpan"></td>

<button [attr.aria-label]="label"></button>
```

---

### `attr.` is not property binding

```html
[attr.disabled]="isDisabled"
```

is **attribute binding**, not property binding.

Compare:

```html
[disabled]="isDisabled"
[attr.disabled]="isDisabled"
```

They have different semantics.

For normal interactive DOM behavior, prefer the actual DOM property:

```html
[disabled]="isDisabled"
```

---

### Binding static values vs dynamic values

This:

```html
<button disabled="true">
```

is a static HTML attribute.

This:

```html
<button [disabled]="true">
```

is Angular property binding.

And:

```html
<button [disabled]="isDisabled">
```

is dynamic property binding.

---

### Property binding is one-way

Property binding is:

```text
Component → View
```

Example:

```ts
isDisabled = true;
```

```html
<button [disabled]="isDisabled">
```

Changing the component state updates the view.

But changing the DOM property does **not automatically update your component property**.

## 5. Interview Questions

### Q1. What is property binding?

**Answer:**

> Property binding allows Angular to dynamically assign a component expression's value to a DOM property or Angular component/directive input using `[property]="expression"` syntax.

---

### Q2. What is the syntax?

```html
[property]="expression"
```

Example:

```html
<button [disabled]="isDisabled">
```

---

### Q3. Property binding vs interpolation?

**Answer:**

> Interpolation is primarily used to render expression results as text, while property binding assigns the expression result to a DOM/component property.

Example:

```html
<h2>{{ username }}</h2>

<button [disabled]="isDisabled">
```

---

### Q4. Property binding vs attribute binding?

**Answer:**

> Property binding targets a DOM/component property, while attribute binding targets an HTML attribute.

```html
[disabled]="isDisabled"
[attr.aria-label]="label"
```

