# Angular Form Validation: A Comprehensive Guide

*Published on February 18, 2024*

## Table of Contents
1. [Template-Driven Forms](#template-driven-forms)
2. [Reactive Forms](#reactive-forms)
3. [Built-in Validators](#built-in-validators)
4. [Custom Validators](#custom-validators)
5. [Cross-Field Validation](#cross-field-validation)
6. [Async Validators](#async-validators)
7. [Error Messages](#error-messages)
8. [Best Practices](#best-practices)

## Template-Driven Forms

Template-driven forms are simpler to implement and are suitable for basic forms. Here's how to set them up:

1. Import FormsModule in your app.module.ts:

```typescript
import { FormsModule } from '@angular/forms';

@NgModule({
  imports: [FormsModule],
  // ...
})
export class AppModule { }
```

2. Create a form in your template:

```html
<form #myForm="ngForm" (ngSubmit)="onSubmit(myForm)">
  <div>
    <label for="name">Name:</label>
    <input 
      type="text" 
      id="name" 
      name="name" 
      [(ngModel)]="user.name"
      required
      minlength="3"
      #name="ngModel"
    >
    <div *ngIf="name.invalid && (name.dirty || name.touched)">
      <div *ngIf="name.errors?.['required']">Name is required</div>
      <div *ngIf="name.errors?.['minlength']">Name must be at least 3 characters</div>
    </div>
  </div>
  <button type="submit" [disabled]="myForm.invalid">Submit</button>
</form>
```

## Reactive Forms

Reactive forms provide more control and are better for complex forms:

1. Import ReactiveFormsModule:

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [ReactiveFormsModule],
  // ...
})
export class AppModule { }
```

2. Create a form in your component:

```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

export class MyComponent {
  myForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.myForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8)]]
    });
  }

  onSubmit() {
    if (this.myForm.valid) {
      console.log(this.myForm.value);
    }
  }
}
```

3. Bind the form in your template:

```html
<form [formGroup]="myForm" (ngSubmit)="onSubmit()">
  <div>
    <label for="name">Name:</label>
    <input 
      type="text" 
      id="name" 
      formControlName="name"
    >
    <div *ngIf="myForm.get('name')?.invalid && myForm.get('name')?.touched">
      <div *ngIf="myForm.get('name')?.errors?.['required']">Name is required</div>
      <div *ngIf="myForm.get('name')?.errors?.['minlength']">Name must be at least 3 characters</div>
    </div>
  </div>
  <button type="submit" [disabled]="myForm.invalid">Submit</button>
</form>
```

## Built-in Validators

Angular provides several built-in validators:

```typescript
this.myForm = this.fb.group({
  requiredField: ['', Validators.required],
  emailField: ['', Validators.email],
  minLengthField: ['', Validators.minLength(3)],
  maxLengthField: ['', Validators.maxLength(10)],
  patternField: ['', Validators.pattern('[a-zA-Z ]*')],
  minField: ['', Validators.min(18)],
  maxField: ['', Validators.max(100)]
});
```

## Custom Validators

Create custom validators for specific validation requirements:

```typescript
export function forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = nameRe.test(control.value);
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Usage in component
this.myForm = this.fb.group({
  name: ['', [
    Validators.required,
    forbiddenNameValidator(/bob/i)
  ]]
});
```

## Cross-Field Validation

Validate fields that depend on each other:

```typescript
export function passwordMatchValidator(g: FormGroup): ValidationErrors | null {
  return g.get('password')?.value === g.get('confirmPassword')?.value
    ? null
    : { mismatch: true };
}

// Usage
this.myForm = this.fb.group({
  password: ['', Validators.required],
  confirmPassword: ['', Validators.required]
}, { validator: passwordMatchValidator });
```

## Async Validators

Use async validators for server-side validation:

```typescript
export class UniqueEmailValidator implements AsyncValidator {
  constructor(private userService: UserService) {}

  validate(control: AbstractControl): Observable<ValidationErrors | null> {
    return this.userService.checkEmail(control.value).pipe(
      map(isTaken => (isTaken ? { uniqueEmail: true } : null)),
      catchError(() => of(null))
    );
  }
}

// Usage
this.myForm = this.fb.group({
  email: ['', {
    validators: [Validators.required, Validators.email],
    asyncValidators: [this.uniqueEmailValidator.validate.bind(this.uniqueEmailValidator)],
    updateOn: 'blur'
  }]
});
```

## Error Messages

Create reusable error message components:

```typescript
@Component({
  selector: 'app-error-message',
  template: `
    <div class="error-message" *ngIf="control?.invalid && control?.touched">
      <div *ngIf="control?.errors?.['required']">This field is required</div>
      <div *ngIf="control?.errors?.['email']">Please enter a valid email</div>
      <div *ngIf="control?.errors?.['minlength']">
        Minimum length is {{control?.errors?.['minlength'].requiredLength}}
      </div>
    </div>
  `
})
export class ErrorMessageComponent {
  @Input() control: AbstractControl | null = null;
}
```

## Best Practices

1. **Choose the Right Approach**: Use template-driven forms for simple forms and reactive forms for complex ones
2. **Validation Timing**: Use `updateOn: 'blur'` for better performance with async validators
3. **Error Messages**: Create reusable error message components
4. **Custom Validators**: Implement custom validators for specific business rules
5. **Async Validation**: Use async validators for server-side validation
6. **Form State**: Track form state (pristine, touched, dirty) for better UX
7. **Accessibility**: Ensure form controls are properly labeled and error messages are accessible
8. **Testing**: Test both form validation and submission logic

Remember to always provide clear feedback to users when validation fails and ensure your forms are accessible to all users.
