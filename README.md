# Task Board Component

Used Angular features in below code:

- **Property binding** — `[src]`, `[alt]`, `[title]`, `[value]`, `[disabled]`, `[class.xxx]`
- **Event binding** — `(input)`, `(change)`, `(click)` with `$event`
- **`@if / @else if / @else`**
- **`@switch / @case / @default`**
- **`@for` with `track`** — plus `$index`, `$first`, `$last`, `$odd`, `$count`
- **`@empty`** — for empty list case

#### Output

<img width="1192" height="795" alt="Screenshot 2026-08-25 at 8 42 09 PM" src="https://github.com/user-attachments/assets/a107d8dc-e20f-4db8-856d-62e855393ed3" />

LINK : [Task Board Component](https://github.com/therishabh/angular-notes/blob/main/Code__Task-Board-Component.md)


-------
-------
-------

# Shopping Cart Component

Ek realistic cart component jo Angular ke `signal`, `computed` aur `effect` ka
proper use dikhata hai.

## Kya use hua hai

- **Writable signals** — `cartItems`, `stockWarning` (aur `currentUserId`) actual
  state hold karte hain, `.set()` / `.update()` se change hote hain.
- **`computed()`** — derived, read-only values: `isCartEmpty`, `totalItemsCount`,
  `subtotal`, `discountPercent`, `discountAmount`, `grandTotal`, `cheapestItem`.
  Ye lazy + memoized hote hain aur ek dusre pe chain (diamond dependency) bhi
  karte hain — `subtotal → discountPercent → discountAmount → grandTotal`.
- **`effect()`** — sirf side effects ke liye:
  - Effect #1: `cartItems` change hote hi localStorage me save karta hai, aur
    `untracked()` se `currentUserId` sirf logging ke liye padhta hai (dependency
    banaye bina).
  - Effect #2: stock full hone par `setTimeout` se warning show karta hai, aur
    `onCleanup` se purana timer clear karta hai (subscription cleanup jaisa).
- **Immutable updates** — array kabhi mutate nahi kiya, hamesha naya array
  (`map`/`filter`) `.update()` ko diya, kyunki signals reference se compare
  karte hain.
- **Custom `equal` option** — `cheapestItem` computed me, taaki same id hone par
  downstream unnecessary re-run na ho.

## Files

- `shopping-cart.ts` — state, computed values, effects, logic
- `shopping-cart.html` — table + summary UI (`@if`, `@for`, property/event binding)
- `shopping-cart.css` — styling

## Screenshot : 
<img width="1042" height="730" alt="Screenshot 2026-08-27 at 1 11 52 PM" src="https://github.com/user-attachments/assets/472522d4-b2ed-4514-bc4d-4626a40e4212" />

LINK : [Shopping Cart Component](https://github.com/therishabh/angular-notes/blob/main/Code__Shopping-Cart-Component.md)

