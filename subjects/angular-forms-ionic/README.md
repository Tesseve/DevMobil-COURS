# Angular Forms

<!-- slide-front-matter class: center, middle -->

## Summary

Get started with and understand the basics of forms in [Angular][angular], using Ionic components.

This subject is an adaptation of the [Angular Forms][angular-forms-subject] subject to the context of an Ionic application.

It is a condensed version of Angular's [Tour of Heroes][angular-tour-of-heroes] tutorial and some of its [Developer Guide][angular-guide],
which you should both read to gain a deeper understanding of Angular.

<!-- slide-include ../../BANNER.md -->

**You will need**

- [Google Chrome][chrome] (recommended, any browser with developer tools will do)
- [Ionic CLI][ionic-cli] (to generate the blank app)

**Recommended reading**

- [Angular][angular-subject]

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Getting started](#getting-started)
- [Forms](#forms)
  - [HTML validations](#html-validations)
  - [Creating a form](#creating-a-form)
    - [Updating the component](#updating-the-component)
    - [Listening to form submit events](#listening-to-form-submit-events)
  - [Checking the validation state](#checking-the-validation-state)
    - [Prevent the form's submission](#prevent-the-forms-submission)
    - [Disable the submit button](#disable-the-submit-button)
    - [Display an error message](#display-an-error-message)
      - [Dirty, pristine, touched, untouched](#dirty-pristine-touched-untouched)
    - [Set the input field background color to red](#set-the-input-field-background-color-to-red)
  - [Angular validators](#angular-validators)
  - [Custom validators](#custom-validators)
    - [Registering a custom validator](#registering-a-custom-validator)
    - [Using a custom validator](#using-a-custom-validator)
  - [Displaying different messages for different errors](#displaying-different-messages-for-different-errors)
  - [Asynchronous validators](#asynchronous-validators)
- [Reactive forms](#reactive-forms)
  - [Using reactive forms in the component](#using-reactive-forms-in-the-component)
    - [Form submition](#form-submition)
    - [Reacting to form value changes](#reacting-to-form-value-changes)
  - [Using reactive forms in the template](#using-reactive-forms-in-the-template)
  - [Reactive form validations](#reactive-form-validations)
    - [Custom validators in reactive forms](#custom-validators-in-reactive-forms)
  - [Which is better, reactive or template-driven?](#which-is-better-reactive-or-template-driven)
- [Resources](#resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Getting started

To follow along this tutorial, you'll need a blank Ionic application.

Generate one using the following command, somewhere on your file system:

```bash
$> ionic start comem-ionic-angular-forms blank
```
Wait for it to finish (this can take several minutes), then head to the newly created folder and start the app:

```bash
$> cd ./comem-ionic-angular-forms
$> ionic serve
```

## Forms

Angular provides **validation** services for forms and controls.
These validations are performed **client-side** for a better user experience: the user gets **instant feedback**.

However, keep in mind that although this provides a good user experience, it can easily be circumvented and thus **cannot be trusted**.

> **Server-side validation is still necessary** for a secure application.


### HTML validations

HTML 5 has [built-in validation attributes][html-input] to define validations on your HTML form inputs (e.g. `<input>`, `<textarea>`, etc), or Ionic form components ([`<ion-checkbox>`][ion-checkbox], [`<ion-input>`][ion-input], etc):

Attribute   | Description
:---        | :---
`min`       | Minimum value for a number
`max`       | Maximum value for a number
`minlength` | Minimum length for a string
`maxlength` | Maximum length for a string
`pattern`   | Regular expression for a string
`required`  | Required field

You simply add them to the HTML tag:

```html
<ion-input type="text" `required minlength="2"` />
```

Usually the **browser** performs these validations.
But Angular **overrides** these and provide its own implementation.
This allows you to add **more complex validations and interaction**.



### Creating a form

Let's add a very simple greeting form to our app:

Open the `home.page.html` file, and replace the `<ion-content>` tags and everything inbetween by:

```html
<ion-content class="ion-padding">

* <form>
    <ion-input type="text" placeholder="Who are you?" [(ngModel)]="greeting"
      `name="greeting" required`></ion-input>
    `<ion-button type="submit">Submit</ion-button>`
* </form>

  <p>Hello {{ greeting }}</p>

</ion-content>
```

Add a `greeting` property to the `HomePage` component in the `home.page.ts` file:

```ts
@Component({ /* ... */ })
export class HomePage {

  `greeting: string;`

}
```

#### Updating the component

Let's now make it so that the greeting will only be displayed if submitted through the form.
We need to add a separate property to our component:

- The `greeting` property will represent the internal value of the input field.
- The `displayedGreeting` property will represent the submitted value (which will no longer be bound to the input field).

We also need a new `displayGreeting()` method which will take the current value of `greeting` and copy it to `displayedGreeting`:

```ts
// ...
export class HomePage {
  // ...
  greeting: string;
* displayedGreeting: string;
  // ...

* displayGreeting() {
*   this.displayedGreeting = this.greeting;
*   console.log('Greeting displayed');
* }
}
```

#### Listening to form submit events

Update the component's template to reflect the fact that we now want to display `displayedGreeting` instead of `greeting`:

```html
<p *ngIf="displayedGreeting">
  Hello {{ displayedGreeting }}
</p>
```

Bind the new `displayGreeting()` method to the form's `submit` event to make it work:

```html
<form `(submit)="displayGreeting()"`>
```



### Checking the validation state

You might have noticed that we have marked the input field as **required**,
but that the user can **still submit the form** when it is invalid (i.e. the input field is empty).

That's not very user-friendly.
We're going to make the following improvements:

- **Prevent the form's submission** if it has invalid fields.
- **Disable the submit button** if the form has invalid fields.
- **Display an error message** when the greeting input field contains an invalid value.
- **Set the input field background color to red** if it contains an invalid value.

#### Prevent the form's submission

In Angular, any `<form>` tag is enriched by the [`NgForm`][angular-docs-ng-form] directive.
You can retrieve the instance of the directive attached to the form by using a [**template reference variable**][angular-template-reference-variable]
(`#greetingForm` in this example):

```html
<form `#greetingForm="ngForm"` (submit)="displayGreeting(`greetingForm`)">
```

We can now update the implementation of `displayGreeting()` to add this new argument.

[`NgForm`][angular-docs-ng-form] must be imported from `@angular/forms`.
This class provides, among other things, a `valid` (or `invalid`) attribute to check whether all the fields are valid or not:

```ts
*import { NgForm } from '@angular/forms';
// ...

displayGreeting(`form: NgForm`) {
  `if (form.valid) {`
    this.displayedGreeting = this.greeting;
    console.log('Greeting displayed');
  `}`
}
```

#### Disable the submit button

The `#greetingForm` template reference variable is already available in the template from the previous modification,
since we passed it to `displayGreeting()` as an argument.

You can also bind it to DOM elements or their attributes elsewhere in the template.

This time, we'll use [`NgForm`][angular-docs-ng-form]'s `invalid` attribute.
We simply have to bind the value of the `<button>` tag's `disabled` attribute to it:

- When the form is **invalid** (`greetingForm.invalid` is true), the button should be **disabled** (`disabled` should be true).
- When the form is **valid** (`greetingForm.invalid` is false), the button should **not be disabled** (`disabled` should be false).

```html
<ion-button `[disabled]="greetingForm.invalid"` type="submit">Submit</ion-button>
```

#### Display an error message

We've seen that the `<form>` tag is **enriched** with the [`NgForm`][angular-docs-ng-form] **directive**,
and that it's possible to **retrieve that directive** to gain access to the **form's validation state**.

You can do the same with the `<input>` tag, by retrieving the field's [`NgModel`][angular-docs-ng-model]'s directive
(which you applied by using `[(ngModel)]='expression'`):

```html
<input type="text" placeholder="Who are you?" [(ngModel)]="greeting"
  name="greeting" required `#greetingInput="ngModel"` />
```

The `NgModel` directive also has the `valid` and `invalid` attributes,
indicating whether that particular field is valid.
All we have to do is add an error message to the form, and, through judicious use of our old friend the `NgIf` directive,
only display the message when the field is invalid.

```html
<form #greetingForm="ngForm" (submit)="displayGreeting(greetingForm)">
  <!-- input -->
* <p *ngIf="greetingInput.invalid">
*   Name is required
* </p>
  <!-- button -->
</form>
```

##### Dirty, pristine, touched, untouched

That's nice, but the error message is displayed right away.
It makes sense, since the **initial value** of the input field's **is actually invalid**.
But that's not very user-friendly.
Ideally, we would want the error message to be displayed **only once the user has interacted with the form**.

Enter the following `NgForm` and `NgModel` attributes:

- `dirty` - A control is **dirty** if the user has **changed its value**.
- `pristine` - A control is **pristine** if the user has **not yet changed its value** (the opposite of `dirty`).
- `touched` - A control is **touched** if the user has triggered a **[`blur`][blur-event] event** on it.
- `untouched` - A control is **untouched** if the user has **not yet** triggered a **[`blur`][blur-event] event** on it (the opposite of `touched`).

Make the following change to only display the error message after the user has started typing:

```html
<p *ngIf="greetingInput.invalid` && greetingInput.dirty`">
  Name is required
</p>
```

#### Set the input field background color to red

Angular automatically **mirrors** many `NgModel` **properties** onto the `<ion-input>` tag as **CSS classes**.
You can use these classes to **style** form elements according to the state of the form.

These are some of the supported classes ([full list][angular-form-control-status-classes]):

- `.ng-valid` or `.ng-invalid` is applied depending on whether the value is valid
- `.ng-pristine` or `.ng-dirty` is applied depending on whether the user has changed the value
- `.ng-untouched` or `.ng-touched` is applied depending on whether the user has triggered `blur` event

So when our field is invalid and dirty, it will have both the `.ng-invalid` and `.ng-dirty` CSS classes added to it.
All you need to do is modify `src/app/home/home.page.css` to add a background color to input fields with this combination of classes:

```css
ion-input.ng-invalid.ng-dirty {
  background-color: #ffc0c0;
}
```



### Angular validators

These are some of the validators provided **out of the box** by Angular:

- [`email`][angular-docs-email-validator] - Validates that a string is a valid e-mail address.
- [`min`][angular-docs-min-validator] & [`max`][angular-docs-max-validator] - Validate that a number is within the specified bound(s).
- [`min-length`][angular-docs-min-length-validator] & [`max-length`][angular-docs-max-length-validator] - Validate that a string's length is within the specified bound(s).
- [`pattern`][angular-docs-pattern-validator] - Validates that a value matches a regular expression.
- [`required`][angular-docs-required-validator] - Validates that a value is present.

Here's a few usage examples:

```html
<ion-input `type="email"` name="email"></ion-input>
<ion-input type="number" name="age" `min="3" max="10"`></ion-input>
<ion-input type="text" name="firstName" `min-length="1" max-length="50"`></ion-input>
<ion-input type="text" name="lastName" `pattern="[a-zA-Z ]*"`></ion-input>
<ion-input type="text" name="occupation" `required`></ion-input>
```



### Custom validators

These validators are nice, but you might need **more complex validations**.

That's why Angular allows you to implement [custom validators][angular-custom-validators].
Here's an example of a validator that ensures a string is not in a list of forbidden values:

Create a new `validators.ts` file in `src/app` with this content:

```ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

export function notInValidator(notIn: string[]): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {

    // Check if the value is invalid.
    if (notIn.indexOf(control.value) >= 0) {
      // Return an error named after the validator if that is the case.
      return {
        notIn: { value: control.value }
      };
    }

    // Otherwise, all is well, there is no error.
    return null;
  };
}
```

#### Registering a custom validator

To use your validation function in a form, you need to wrap it in a directive. Create another file in `src/app`, called for example `not-in.directive.ts` with this content:

```ts
import { Directive, Input } from '@angular/core';
import { AbstractControl, NG_VALIDATORS,
         ValidationErrors, Validator } from '@angular/forms';

import { notInValidator } from './validators';

@Directive({
  selector: "[notIn]",
  providers: [
    {
      provide: NG_VALIDATORS,
      useExisting: NotInValidatorDirective,
      multi: true
    }
  ]
})
export class NotInValidatorDirective implements Validator {
  @Input() notIn: string[];

  validate(control: AbstractControl): ValidationErrors | null {
    return notInValidator(this.notIn)(control);
  }
}
```

#### Using a custom validator

You must register the new directive in the `declarations` array of the module that needs to use this directive, in our case, it's the `home.module.ts`:

> In a real life scenario, and in your application, you should instead create a `SharedModule` that will declare and export all shared directives and components throughout your application (see [this tutorial][angular-shared-module], in French)

```ts
// Other imports...
*import { NotInValidatorDirective } from '../not-in.directive';

@NgModule({
  imports: [/* ... */],
  declarations: [HomePage, `NotInValidatorDirective`]
})
export class HomePageModule {}
```

You can then finally use it in the template:

```html
<ion-input #greetingInput="ngModel" type="text" placeholder="Who are you?"
  [(ngModel)]="greeting" name="greeting" `[notIn]="['Bob']"` required>
</ion-input>
```
> Notice that the `notIn` attribute must be bound to an array of string


### Displaying different messages for different errors

There's a little problem now.
Since our error message is displayed as soon as there's an error on the `greetingInput` field,
it always displays `Name is required`, even if the error is due to our new custom validator.

```html
<p *ngIf="`greetingInput.invalid` && greetingInput.dirty">
  Name is required
</p>
```

To fix that, use `greetingInput.hasError(<errorName: string>)` which is a method that checks if the given `errorName` is present in the `errors` array of the input.

That way you can react separately to the `required` validator's error and to the custom validator's `notIn` error:

```html
<p *ngIf="`greetingInput.hasError('required')` && greetingInput.dirty">
  Name is required
</p>
*<p *ngIf="greetingInput.hasError('notIn') && greetingInput.dirty">
* Name is forbidden
*</p>
```
> `notIn` is, in this case, the name of the `ValidationError`'s property returned by the `notInValidator` function

### Asynchronous validators

The validator we implemented is actually a function that matches Angular's [`ValidatorFn`][angular-docs-validator-fn] interface.
It is **synchronous**, i.e. it performs no I/O operation to validate its value and immediately returns its errors (or `null`):

```ts
(control: AbstractControl): `ValidationErrors | null`
```

It's also possible to create **asynchronous validators**.
For example, to check whether a **username is already taken**, you might have to **call your API**, which is an asynchronous operation.

In that case, your validator function must match the [`AsyncValidatorFn`][angular-docs-async-validator-fn] interface instead.
That is, instead of returning an object of validation errors, it must return either a **Promise or an Observable** of that object:

```ts
(control: AbstractControl): `Promise<ValidationErrors | null>`
                            | `Observable<ValidationErrors | null>`
```

That way, Angular will wait for the Promise to be resolved or for the Observable to emit the errors before updating the template.



## Reactive forms

The form we have seen so far is a [**template-driven form**][angular-template-driven-form].
In contrast, **reactive forms** are an Angular technique for creating forms in a **reactive programming** style.
They are provided by a separate module, the [`ReactiveFormsModule`][angular-docs-reactive-forms-module]. Let's add it to our `HomePageModule`'s `imports` array:

```ts
// Other imports
import { FormsModule, `ReactiveFormsModule` } from '@angular/forms';

@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    IonicModule,
    RouterModule.forChild([
      {
        path: '',
        component: HomePage
      }
    ]),
    `ReactiveFormsModule`
  ],
  // ...
})
export class HomePageModule {}
```



### Using reactive forms in the component

With reactive forms, the form structure is also defined with code in the component

Replace all the content of `home.page.ts` with the following:

```ts
import { Component } from '@angular/core';
import { `FormBuilder, FormGroup` } from '@angular/forms';

@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss']
})
export class HomePage {
  greeting: string;
  displayedGreeting: string;
  `greetingForm: FormGroup;`

  constructor(`private formBuilder: FormBuilder`) {
*   this.greetingForm = this.formBuilder.group({
*     // Define the greeting input and its default value
*     greeting: ['']
*   });
  }
}
```

#### Form submition

Since the form is built directly into the component, the method that is executed when it's submitted doesn't need an argument anymore.

It can be rewritten as follows:

```ts
// ...
export class HomePage {
  // ...

  displayGreeting`()` {
    if (`this.greetingForm`.valid) {
      this.displayedGreeting = this.greeting;
    }
  }
}
```


#### Reacting to form value changes

Reactive forms do not use `ngModel`, so there won't be two-way binding with the template any longer.
There will only be **one-way binding** from the form group in the component to the template.

To be notified of changes in the form group, you can subscribe to its `valueChanges` property, which is an Observable:

```ts
// ...
export class HomePage {
  // ...
  constructor(private formBuilder: FormBuilder) {
    // ...
*   this.greetingForm.valueChanges.subscribe(formValues => {
*     console.log('Greeting changed to', formValues.greeting);
*     this.greeting = formValues.greeting;
*   });
  }
}
```



### Using reactive forms in the template

With reactive forms, your **template** is somewhat **simplified**.
Remove `ngModel`, the validations, `#greetingForm` and `#greetingInput`.
You simply have to pass the form group created in the component to the `<form>` tag with the `formGroup` directive,
and use the `formControlName` directive on the `<input>` tag:

```html
<form `[formGroup]="greetingForm"` (submit)="`displayGreeting()`">
  <ion-input type="text" placeholder="Who are you?" `formControlName="greeting"`></ion-input>
  <!-- ... -->
</form>
```

You must also update your error messages to get the input field from the form group with `greetingForm.get("greeting")`
instead of using the `#greetingInput` template reference variable you just removed:

> Since `get()` can return `undefined`, we need the `?` after the call

```html
<p *ngIf="`greetingForm.get('greeting')?`.hasError('required')
          && `greetingForm.get('greeting')`.dirty">
  Name is required
</p>
<p *ngIf="`greetingForm.get('greeting')?`.hasError('notIn')
          && `greetingForm.get('greeting')`.dirty">
  Name is forbidden
</p>
```



### Reactive form validations

The validations are no longer applied since we removed them from the template.
With reactive forms, **validation is configured in the component**:

```ts
// Other imports...
import { FormBuilder, FormGroup, `Validators` } from '@angular/forms';
*import { notInValidator } from '../validators';

@Component({/* ... */})
export class HomePage {
  // ...
  constructor(private formBuilder: FormBuilder) {
    this.greetingForm = this.formBuilder.group({
*     greeting: [
*       '',
*       // Add the built-in "required" validator and our custom function
*       [ Validators.required, notInValidator([ 'Bob' ]) ]
*     ]
    });
    // ...
  }
  // ...
}
```

#### Custom validators in reactive forms

Note that we use our custom validator function (`notInValidator`) directly,
instead of using the `NotInValidatorDirective` wrapper like before:

```ts
[ // Add validators to the field.
  Validators.required, // Use the built-in "required" validator.
  `notInValidator([ 'Bob' ])` // Use our custom validator function.
]
```

This is an advantage of reactive forms over template-driven forms:
validators can be **simple functions** that do not require an additional directive to be applied in the template.

You can remove the directive (e.g. delete `src/app/not-in.directive.ts` and remove it from `declarations` in `src/app/home/home.module.ts`),
and the form will keep working.



### Which is better, reactive or template-driven?

In [**template-driven forms**][angular-forms], form structure and validation are specified and handled **in the template**:

- Creation of form controls is delegated to directives and asynchronous.
- Angular handles data updates with two-way binding.
- Hard to test with automated tests.

In [**reactive forms**][angular-reactive-forms], a tree of form control objects and validations is managed **in the component** and bound to elements in the template:

- The component class has immediate access to both the data model and the form control structure.
- Changes can be subscribed to in the form of Observables.
- Data and validity updates are synchronous and under your control.
- Easier to test with automated tests.

Neither is "better".
They're two different architectural paradigms, with their own strengths and weaknesses.
You may even use both in the same application.

**Read the documentation to learn more.**



## Resources

**Documentation**

- [Angular Tour of Heroes Tutorial][angular-tour-of-heroes]
- [Angular Developer Guide][angular-guide]
  * [Template-driven Forms][angular-forms]
  * [Reactive Forms][angular-reactive-forms]
- [Angular API reference][angular-api]

[angular]: https://angular.io
[angular-api]: https://angular.io/api
[angular-cli-subject]: ../angular-cli
[angular-custom-validators]: https://angular.io/guide/form-validation#custom-validators
[angular-docs-async-validator-fn]: https://angular.io/api/forms/AsyncValidatorFn
[angular-docs-email-validator]: https://angular.io/api/forms/EmailValidator
[angular-docs-max-length-validator]: https://angular.io/api/forms/MaxLengthValidator
[angular-docs-min-length-validator]: https://angular.io/api/forms/MinLengthValidator
[angular-docs-max-validator]: https://angular.io/api/forms/Validators#max
[angular-docs-min-validator]: https://angular.io/api/forms/Validators#min
[angular-docs-ng-form]: https://angular.io/api/forms/NgForm
[angular-docs-ng-model]: https://angular.io/api/forms/NgModel
[angular-docs-pattern-validator]: https://angular.io/api/forms/PatternValidator
[angular-docs-reactive-forms-module]: https://angular.io/api/forms/ReactiveFormsModule
[angular-docs-required-validator]: https://angular.io/api/forms/RequiredValidator
[angular-docs-validator-fn]: https://angular.io/api/forms/ValidatorFn
[angular-form-control-status-classes]: https://angular.io/guide/form-validation#control-status-css-classes
[angular-forms]: https://angular.io/guide/forms
[angular-forms-subject]: ../angular-forms
[angular-guide]: https://angular.io/guide/architecture
[angular-subject]: ../angular/
[angular-template-driven-form]: https://angular.io/guide/forms
[angular-template-reference-variable]: https://angular.io/guide/template-syntax#ref-vars
[angular-tour-of-heroes]: https://angular.io/tutorial
[angular-reactive-forms]: https://angular.io/guide/reactive-forms
[angular-shared-module]: https://guide-angular.wishtack.io/angular/project-structure-and-modules/shared-module
[blur-event]: https://developer.mozilla.org/en-US/docs/Web/Events/blur
[chrome]: https://www.google.com/chrome/
[html-input]: https://www.w3schools.com/tags/tag_input.asp
[ionic-cli]: https://ionicframework.com/docs/cli
[ion-input]: https://ionicframework.com/docs/api/input
[ion-checkbox]: https://ionicframework.com/docs/api/checkbox
