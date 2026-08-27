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

### File : shopping-cart.html
```HTML
<section class="cart">
  <h2>Shopping Cart</h2>

  <!-- stockWarning is a plain signal, set from inside effect #2 -->
  @if (stockWarning()) {
    <p class="warning-banner">{{ stockWarning() }}</p>
  }

  <!-- isCartEmpty is a computed signal, note the () call, same as reading any signal -->
  @if (isCartEmpty()) {
    <p class="empty-state">Your cart is empty. Go add something!</p>
  } @else {
    <table class="cart-table">
      <thead>
        <tr>
          <th>Item</th>
          <th>Price</th>
          <th>Quantity</th>
          <th>Line Total</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        @for (item of cartItems(); track item.id) {
          <tr [class.cheapest-row]="item.id === cheapestItem().id">
            <td>
              {{ item.name }}
              @if (item.id === cheapestItem().id) {
                <span class="tag">Best Price</span>
              }
            </td>
            <td>₹{{ item.price }}</td>
            <td class="qty-cell">
              <button type="button" (click)="decreaseQuantity(item.id)" [disabled]="item.quantity <= 1">-</button>
              <span>{{ item.quantity }}</span>
              <!-- disabled once quantity hits available stock, computed in the component logic -->
              <button type="button" (click)="increaseQuantity(item.id)" [disabled]="item.quantity >= item.stock">+</button>
            </td>
            <td>₹{{ item.price * item.quantity }}</td>
            <td>
              <button type="button" class="remove-btn" (click)="removeItem(item.id)">Remove</button>
            </td>
          </tr>
        }
      </tbody>
    </table>

    <!-- Every value below is a computed signal, all derived from cartItems -->
    <div class="summary">
      <p>Total items: {{ totalItemsCount() }}</p>
      <p>Subtotal: ₹{{ subtotal() }}</p>
      <p>Discount ({{ discountPercent() }}%): -₹{{ discountAmount() }}</p>
      <p class="grand-total">Grand Total: ₹{{ grandTotal() }}</p>
    </div>

    <button type="button" class="clear-btn" (click)="clearCart()">Clear Cart</button>
  }
</section>

```

### File : shopping-cart.ts
```ts
import { Component, computed, effect, signal, untracked } from '@angular/core';

// Real-world shape: one product sitting inside the cart.
interface CartItem {
  id: number;
  name: string;
  price: number; // price per unit, in rupees
  quantity: number; // how many the user has added
  stock: number; // how many units are available in warehouse
}

@Component({
  selector: 'app-shopping-cart',
  templateUrl: './shopping-cart.html',
  styleUrl: './shopping-cart.css',
})
export class ShoppingCartComponent {
  // ------------------------------------------------------------------
  // 1) WRITABLE SIGNALS — these hold the actual state.
  //    Only writable signals have .set() / .update(), computed signals do not.
  // ------------------------------------------------------------------

  // Not shown in the UI on purpose. Used later only for a console log,
  // so we can demonstrate untracked() below without adding a fake dependency.
  private currentUserId = signal(101);

  // IMPORTANT interview point: arrays/objects inside a signal must be replaced
  // with a NEW reference on every change (spread/map/filter), never mutated in
  // place (e.g. `this.cartItems().push(x)`). Signals compare by reference by
  // default, so mutating the same array will NOT notify any subscriber.
  cartItems = signal<CartItem[]>([
    { id: 1, name: 'Wireless Mouse', price: 799, quantity: 1, stock: 5 },
    { id: 2, name: 'Mechanical Keyboard', price: 2999, quantity: 1, stock: 3 },
    { id: 3, name: 'USB-C Hub', price: 1499, quantity: 1, stock: 2 },
  ]);

  // Plain writable signal that a template banner reads. Set inside an effect below.
  stockWarning = signal<string | null>(null);

  // ------------------------------------------------------------------
  // 2) COMPUTED SIGNALS — derived, read-only state.
  //    Computed is LAZY + MEMOIZED: the callback only re-runs when it is
  //    actually read AND one of the signals it touched has changed.
  //    Reading the same computed twice without a dependency change returns
  //    the cached value instead of re-running the function.
  // ------------------------------------------------------------------

  isCartEmpty = computed(() => this.cartItems().length === 0);

  totalItemsCount = computed(() =>
    this.cartItems().reduce((sum, item) => sum + item.quantity, 0),
  );

  subtotal = computed(() =>
    this.cartItems().reduce((sum, item) => sum + item.price * item.quantity, 0),
  );

  // "Diamond dependency" example: this computed depends on ANOTHER computed
  // (subtotal), not directly on the cartItems signal. Angular still tracks
  // this correctly and only recomputes discountPercent when subtotal's
  // actual VALUE changes (not merely when cartItems changes internally).
  discountPercent = computed(() => {
    const amount = this.subtotal();
    if (amount >= 5000) return 15;
    if (amount >= 2000) return 10;
    if (amount >= 1000) return 5;
    return 0;
  });

  discountAmount = computed(() =>
    Math.round((this.subtotal() * this.discountPercent()) / 100),
  );

  // Depends on two computeds at once, proves computed chains compose cleanly.
  grandTotal = computed(() => this.subtotal() - this.discountAmount());

  // Custom equality function: without this, every recompute of "cheapestItem"
  // would produce a brand new object reference and could trigger unnecessary
  // work downstream (e.g. re-render of anything comparing by reference).
  // With `equal`, Angular treats the new result as "unchanged" if the id is
  // the same, so dependents of THIS computed do not re-run needlessly.
  cheapestItem = computed(
    () =>
      this.cartItems().reduce((cheapest, item) =>
        item.price < cheapest.price ? item : cheapest,
      ),
    { equal: (a, b) => a.id === b.id },
  );

  constructor() {
    // ------------------------------------------------------------------
    // 3) EFFECT — for SIDE EFFECTS ONLY (logging, localStorage, DOM APIs,
    //    analytics), never for deriving state. If you find yourself calling
    //    `.set()` on another signal purely to mirror a value, that should
    //    almost always be a computed() instead. Using effect() for that is
    //    a very common interview anti-pattern question.
    //
    //    Effects must be created inside an "injection context" — normally
    //    the constructor (like here) or a field initializer. Calling
    //    effect() later, e.g. inside a click handler, needs an explicit
    //    `{ injector }` option passed in.
    // ------------------------------------------------------------------

    // Effect #1: persist the cart to localStorage whenever it changes.
    effect(() => {
      const items = this.cartItems(); // <-- reading this makes it a dependency

      // Guard for SSR: localStorage does not exist on the server.
      // (In a real SSR app, prefer `afterNextRender` for browser-only work.)
      if (typeof window !== 'undefined') {
        window.localStorage.setItem('cart-items', JSON.stringify(items));
      }

      // untracked() lets us READ a signal inside the effect WITHOUT
      // subscribing to it. Without untracked, reading currentUserId() here
      // would make the effect re-run every time the user id changes too,
      // even though we only want it for the log line, not as a trigger.
      untracked(() => {
        console.log(`[user ${this.currentUserId()}] cart saved, ${items.length} line item(s)`);
      });
    });

    // Effect #2: warn when any item has reached max available stock.
    // Demonstrates the onCleanup callback — it runs before the NEXT
    // execution of the effect, and also when the component is destroyed.
    // This is exactly like clearing a timer/subscription in ngOnDestroy,
    // except Angular does it automatically per-run.
    effect((onCleanup) => {
      const items = this.cartItems();
      const lowStockItem = items.find((item) => item.quantity >= item.stock);

      if (!lowStockItem) {
        this.stockWarning.set(null);
        return;
      }

      // Simulate a debounced warning (e.g. waiting for an animation to finish).
      const timerId = setTimeout(() => {
        this.stockWarning.set(`Only ${lowStockItem.stock} unit(s) of "${lowStockItem.name}" left in stock!`);
      }, 300);

      onCleanup(() => clearTimeout(timerId));
    });
  }

  // ------------------------------------------------------------------
  // 4) STATE UPDATES — always immutable, using .update() with a new array.
  // ------------------------------------------------------------------

  increaseQuantity(id: number): void {
    this.cartItems.update((items) =>
      items.map((item) =>
        // Never let quantity exceed the available stock.
        item.id === id && item.quantity < item.stock
          ? { ...item, quantity: item.quantity + 1 }
          : item,
      ),
    );
  }

  decreaseQuantity(id: number): void {
    this.cartItems.update((items) =>
      items.map((item) =>
        item.id === id && item.quantity > 1
          ? { ...item, quantity: item.quantity - 1 }
          : item,
      ),
    );
  }

  removeItem(id: number): void {
    this.cartItems.update((items) => items.filter((item) => item.id !== id));
  }

  clearCart(): void {
    this.cartItems.set([]);
  }

  // NOTE for interview: the line below would NOT compile if uncommented,
  // because computed() signals are read-only (no .set/.update on them).
  // this.subtotal.set(0);
}

```

### File : shopping-cart.css
```css
.cart {
  font-family: Arial, sans-serif;
  max-width: 700px;
  margin: 20px auto;
  padding: 16px;
}

.warning-banner {
  background-color: #fff3e0;
  color: #e65100;
  padding: 10px 14px;
  border-radius: 6px;
  font-weight: 600;
}

.empty-state {
  color: #999;
  font-style: italic;
}

.cart-table {
  width: 100%;
  border-collapse: collapse;
}

.cart-table th,
.cart-table td {
  padding: 10px;
  border-bottom: 1px solid #eee;
  text-align: left;
}

.cheapest-row {
  background-color: #f1f8e9;
}

.tag {
  margin-left: 6px;
  padding: 2px 6px;
  font-size: 10px;
  background-color: #4caf50;
  color: #fff;
  border-radius: 4px;
}

.qty-cell {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty-cell button {
  width: 26px;
  height: 26px;
  border: 1px solid #ccc;
  background-color: #fff;
  border-radius: 4px;
  cursor: pointer;
}
.qty-cell button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.remove-btn {
  border: 0;
  background: none;
  color: #c62828;
  cursor: pointer;
  text-decoration: underline;
}

.summary {
  margin-top: 16px;
  text-align: right;
}

.grand-total {
  font-size: 18px;
  font-weight: 700;
}

.clear-btn {
  margin-top: 10px;
  padding: 8px 14px;
  border: 0;
  border-radius: 6px;
  background-color: #3f51b5;
  color: #fff;
  cursor: pointer;
}

```
