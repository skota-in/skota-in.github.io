# Angular Custom Form Controls: A Comprehensive Guide

*Published on April 17, 2024*

## Table of Contents
1. [Introduction](#introduction)
2. [Creating a Custom Form Control](#creating-a-custom-form-control)
3. [Implementing ControlValueAccessor](#implementing-controlvalueaccessor)
4. [Handling Validation](#handling-validation)
5. [Styling and Accessibility](#styling-and-accessibility)
6. [Best Practices](#best-practices)

## Introduction

Custom form controls in Angular allow you to create reusable form components that integrate seamlessly with Angular's reactive forms and template-driven forms. This guide will walk you through creating custom form controls using the `ControlValueAccessor` interface.

## Creating a Custom Form Control

Let's create a custom rating component as an example:

1. First, create a new component:

```typescript
// rating.component.ts
import { Component, forwardRef } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-rating',
  template: `
    <div class="rating">
      <span *ngFor="let star of stars" 
            (click)="setRating(star)"
            [class.active]="star <= value">
        ★
      </span>
    </div>
  `,
  styles: [`
    .rating {
      display: flex;
      gap: 4px;
    }
    .rating span {
      cursor: pointer;
      font-size: 24px;
      color: #ccc;
    }
    .rating span.active {
      color: #ffd700;
    }
  `],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => RatingComponent),
      multi: true
    }
  ]
})
export class RatingComponent implements ControlValueAccessor {
  value = 0;
  stars = [1, 2, 3, 4, 5];
  onChange: any = () => {};
  onTouched: any = () => {};

  setRating(rating: number) {
    this.value = rating;
    this.onChange(rating);
    this.onTouched();
  }

  writeValue(value: number): void {
    this.value = value;
  }

  registerOnChange(fn: any): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
}
```

## Implementing ControlValueAccessor

The `ControlValueAccessor` interface requires implementing four methods:

1. `writeValue`: Updates the value in the UI
2. `registerOnChange`: Registers a callback for when the value changes
3. `registerOnTouched`: Registers a callback for when the control is touched
4. `setDisabledState`: Optional method to handle disabled state

Here's how to use the custom rating component in a form:

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';

@Component({
  selector: 'app-root',
  template: `
    <form [formGroup]="form">
      <app-rating formControlName="rating"></app-rating>
      <p>Selected rating: {{ form.get('rating')?.value }}</p>
    </form>
  `
})
export class AppComponent {
  form: FormGroup;

  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      rating: [3]
    });
  }
}
```

## Handling Validation

To add validation to your custom form control:

```typescript
// rating.component.ts
import { Validator, NG_VALIDATORS } from '@angular/forms';

@Component({
  // ... other component metadata
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => RatingComponent),
      multi: true
    },
    {
      provide: NG_VALIDATORS,
      useExisting: forwardRef(() => RatingComponent),
      multi: true
    }
  ]
})
export class RatingComponent implements ControlValueAccessor, Validator {
  // ... existing code

  validate(control: AbstractControl): ValidationErrors | null {
    const value = control.value;
    if (value < 1 || value > 5) {
      return { invalidRating: true };
    }
    return null;
  }
}
```

## Styling and Accessibility

Make your custom form control accessible:

```typescript
// rating.component.ts
@Component({
  selector: 'app-rating',
  template: `
    <div class="rating" role="radiogroup" aria-label="Rating">
      <span *ngFor="let star of stars" 
            (click)="setRating(star)"
            [class.active]="star <= value"
            role="radio"
            [attr.aria-checked]="star <= value"
            [attr.aria-label]="'Rate ' + star + ' stars'"
            tabindex="0"
            (keydown.enter)="setRating(star)">
        ★
      </span>
    </div>
  `
})
```

## Best Practices

1. **Implement ControlValueAccessor**: Always implement this interface for custom form controls
2. **Handle Disabled State**: Implement `setDisabledState` to properly handle disabled state
3. **Accessibility**: Make your controls accessible with proper ARIA attributes
4. **Validation**: Implement the `Validator` interface for custom validation
5. **Error Handling**: Provide clear error messages and visual feedback
6. **Styling**: Use CSS custom properties for theming
7. **Documentation**: Document your component's inputs, outputs, and usage
8. **Testing**: Write comprehensive tests for your custom form control

Example of a more complex custom form control with validation and error messages:

```typescript
@Component({
  selector: 'app-custom-input',
  template: `
    <div class="form-group" [class.error]="hasError">
      <label [for]="id">{{ label }}</label>
      <input
        [id]="id"
        [type]="type"
        [value]="value"
        (input)="onInput($event)"
        (blur)="onBlur()"
        [disabled]="disabled"
      />
      <div class="error-message" *ngIf="hasError">
        {{ errorMessage }}
      </div>
    </div>
  `,
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CustomInputComponent),
      multi: true
    }
  ]
})
export class CustomInputComponent implements ControlValueAccessor {
  @Input() label: string;
  @Input() type = 'text';
  @Input() id: string;
  
  value: any;
  disabled = false;
  hasError = false;
  errorMessage = '';

  onChange: any = () => {};
  onTouched: any = () => {};

  onInput(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    this.value = value;
    this.onChange(value);
    this.validate(value);
  }

  onBlur() {
    this.onTouched();
    this.validate(this.value);
  }

  writeValue(value: any): void {
    this.value = value;
  }

  registerOnChange(fn: any): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }

  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }

  private validate(value: any) {
    // Add your validation logic here
    if (!value) {
      this.hasError = true;
      this.errorMessage = 'This field is required';
    } else {
      this.hasError = false;
      this.errorMessage = '';
    }
  }
}
```

Remember to always test your custom form controls thoroughly and ensure they work well with both reactive and template-driven forms.
