# Angular Signals and Effects

This guide explains how to use Signals and Effects in Angular, which are powerful features introduced to handle reactive state management.

## What are Signals?

Signals are a new reactive primitive in Angular that can contain any value. They notify interested consumers when that value changes.

## Basic Signal Usage

### Creating a Signal

```typescript
import { signal } from '@angular/core';

// Create a signal with an initial value
const count = signal(0);

// Create a signal with a complex object
const user = signal({
  name: 'John',
  age: 30,
  email: 'john@example.com'
});
```

### Reading Signal Values

```typescript
// Get the current value
console.log(count()); // Output: 0

// Get the current value of a complex object
console.log(user().name); // Output: 'John'
```

### Updating Signal Values

```typescript
// Set a new value
count.set(5);

// Update using the current value
count.update(value => value + 1);

// Update a complex object
user.update(user => ({
  ...user,
  age: user.age + 1
}));
```

## Computed Signals

Computed signals derive their value from other signals.

```typescript
import { signal, computed } from '@angular/core';

const firstName = signal('John');
const lastName = signal('Doe');

// Create a computed signal
const fullName = computed(() => `${firstName()} ${lastName()}`);

console.log(fullName()); // Output: 'John Doe'

// When firstName or lastName changes, fullName automatically updates
firstName.set('Jane');
console.log(fullName()); // Output: 'Jane Doe'
```

## Effects

Effects are operations that run when one or more signal values change.

```typescript
import { signal, effect } from '@angular/core';

const count = signal(0);

// Create an effect
effect(() => {
  console.log(`The count is: ${count()}`);
});

// The effect will run immediately and whenever count changes
count.set(1); // Logs: "The count is: 1"
count.set(2); // Logs: "The count is: 2"
```

## Practical Examples

### Example 1: Shopping Cart

```typescript
import { Component, signal, computed, effect } from '@angular/core';

interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Component({
  selector: 'app-cart',
  template: `
    <div *ngFor="let item of cartItems()">
      {{ item.name }} - ${{ item.price }} x {{ item.quantity }}
    </div>
    <div>Total: ${{ total() }}</div>
  `
})
export class CartComponent {
  // Signal for cart items
  cartItems = signal<CartItem[]>([
    { id: 1, name: 'Product 1', price: 10, quantity: 2 },
    { id: 2, name: 'Product 2', price: 20, quantity: 1 }
  ]);

  // Computed signal for total
  total = computed(() => 
    this.cartItems().reduce((sum, item) => sum + (item.price * item.quantity), 0)
  );

  // Effect to save cart to localStorage
  constructor() {
    effect(() => {
      localStorage.setItem('cart', JSON.stringify(this.cartItems()));
    });
  }

  // Methods to modify cart
  addItem(item: CartItem) {
    this.cartItems.update(items => [...items, item]);
  }

  updateQuantity(id: number, quantity: number) {
    this.cartItems.update(items =>
      items.map(item => 
        item.id === id ? { ...item, quantity } : item
      )
    );
  }
}
```

### Example 2: Theme Switcher

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-theme-switcher',
  template: `
    <button (click)="toggleTheme()">
      Switch to {{ isDarkTheme() ? 'Light' : 'Dark' }} Theme
    </button>
  `
})
export class ThemeSwitcherComponent {
  isDarkTheme = signal(false);

  constructor() {
    // Initialize theme from localStorage
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      this.isDarkTheme.set(savedTheme === 'dark');
    }

    // Effect to apply theme changes
    effect(() => {
      document.body.classList.toggle('dark-theme', this.isDarkTheme());
      localStorage.setItem('theme', this.isDarkTheme() ? 'dark' : 'light');
    });
  }

  toggleTheme() {
    this.isDarkTheme.update(value => !value);
  }
}
```

### Example 3: Form Validation

```typescript
import { Component, signal, computed } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-signup-form',
  template: `
    <form (ngSubmit)="onSubmit()">
      <input [(ngModel)]="email" name="email" />
      <div *ngIf="!isEmailValid()">Invalid email format</div>
      
      <input [(ngModel)]="password" name="password" type="password" />
      <div *ngIf="!isPasswordValid()">
        Password must be at least 8 characters
      </div>
      
      <button [disabled]="!isFormValid()">Submit</button>
    </form>
  `
})
export class SignupFormComponent {
  email = signal('');
  password = signal('');

  // Computed signals for validation
  isEmailValid = computed(() => 
    /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email())
  );

  isPasswordValid = computed(() => 
    this.password().length >= 8
  );

  isFormValid = computed(() => 
    this.isEmailValid() && this.isPasswordValid()
  );

  onSubmit() {
    if (this.isFormValid()) {
      // Handle form submission
      console.log('Form submitted:', {
        email: this.email(),
        password: this.password()
      });
    }
  }
}
```

## Best Practices

1. **Use Signals for State Management**
   - Replace simple state variables with signals
   - Use computed signals for derived state
   - Use effects for side effects

2. **Performance Considerations**
   - Keep effects minimal and focused
   - Use computed signals to avoid unnecessary recalculations
   - Be careful with effects that trigger other effects

3. **Cleanup**
   - Effects automatically clean up when the component is destroyed
   - For manual cleanup, store the effect in a variable:
   ```typescript
   const effectRef = effect(() => {
     // effect code
   });
   
   // Later, to cleanup
   effectRef.destroy();
   ```

## Additional Resources

- [Angular Signals Documentation](https://angular.io/guide/signals)
- [Angular Change Detection](https://angular.io/guide/change-detection)
- [Reactive Programming with Signals](https://angular.io/guide/reactive-programming) 