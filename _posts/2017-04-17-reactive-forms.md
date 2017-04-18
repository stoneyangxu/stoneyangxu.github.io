---
layout: post
title:  "Reactive Forms"
date:   2017-04-17 23:25:05
categories: Angular
---

# Reactive Forms

> Reactive forms is an Angular technique for creating forms in a reactive style.
> see: https://angular.cn/docs/ts/latest/guide/reactive-forms.html

Two different form-building ways: `template-driven` and `reactive (model-driven)`.
They belong to two module: `ReactiveFormsModule` and `FormsModule`.

> Reactive forms offer the ease of using reactive patterns, testing, and validation

> You create and manipulate form control objects directly in the component class.
> Rather than update the data model directly, the component extracts user changes and forwards them to an external component or service, which does something with them (such as saving them) and returns a new data model to the component that reflects the updated model state.

## Template-driven forms
- Don't create form control objects.
- Angular create them and we use them with `ngModel`.
- Template-driven forms as asynchronous, which may complicate development.

## Async vs. sync

- Reactive forms are synchronous. 
- Template-driven forms are asynchronous.

### In reactive forms, the whole entrie form control tree is created in code, we can update the value through the tree.

### In template forms, form controls are delegated to directives.

> To avoid "changed after checked" errors, these directives take more than one cycle to build the entire control tree.
> That means you must wait a tick before manipulating any of the controls from within the component class.

### When using unit tests, template-driven forms must wrap tests with `async` and `fakeAsync`. But in reactive forms, everything is available when you need.

## Reactive Form Demo

### Create a `data-model.ts` to hold the models.

We create models that we are going to use.

```ts
export class Hero {
  id = 0;
  name = '';
  addresses: Address[];
}

export class Address {
  street = '';
  city   = '';
  state  = '';
  zip    = '';
}

export const heroes: Hero[] = [
  {
    id: 1,
    name: 'Whirlwind',
    addresses: [
      {street: '123 Main',  city: 'Anywhere', state: 'CA',  zip: '94801'},
      {street: '456 Maple', city: 'Somewhere', state: 'VA', zip: '23226'},
    ]
  },
  {
    id: 2,
    name: 'Bombastic',
    addresses: [
      {street: '789 Elm',  city: 'Smallville', state: 'OH',  zip: '04501'},
    ]
  },
  {
    id: 3,
    name: 'Magneta',
    addresses: [ ]
  },
];

export const states = ['CA', 'MD', 'OH', 'VA'];
```

### Then we create a simple form component, import `ReactiveFomrsModule`

```ts
@NgModule({
  imports: [
    CommonModule,
    ReactiveFormsModule
  ],
  declarations: [
    HeroDetailComponent
  ]
})
export class DemoComponentsModule { }
```

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'hero-detail',
  templateUrl: './hero-detail.component.html',
  styleUrls: ['./hero-detail.component.scss']
})
export class HeroDetailComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}
```

### Create a FormControl object to bind to an `input` element.

```ts
  name = new FormControl();
```
> A FormControl constructor accepts three, optional arguments: the initial data value, an array of validators, and an array of async validators.
> see: https://angular.io/docs/ts/latest/cookbook/form-validation.html

```html
<div class="container form-group">
  <label class="center-block">Name:</label>
  <input class="form-control" [formControl]="name">
</div>
```

![](/images/2017-04-18-00-00-17.jpg)

### Usually, there are multiple `FormControls` in a form, we register them in a `FromGroup`

```ts
export class HeroDetailComponent {
  heroForm = new FormGroup({
    name: new FormControl()
  });
}
```

The FormGroup need to be reflected in the template.

```html
<div class="container">
  <form [formGroup]="heroForm" novalidate>
    <div class="form-group">
      <label class="center-block">Name:</label>
      <input class="form-control" formControlName="name">
    </div>
  </form>
</div>
```

- Bind the form to FormGroup with `[formGroup]="heroForm"`.
- `novalidate` means that the browser will skip native validations.
- When working with `formGroup`, we need to use `formControlName="name"` to associated with the `FormControl` in formGroup.

### We will check the form status and values with `heroForm` property.

```html
<p>Form value: {{ heroForm.value | json }}</p>
<p>Form status: {{ heroForm.status | json }}</p>
```
When entering the hero's name the form's value changed.

![](/images/2017-04-18-00-36-06.jpg)

## Reducing repetition and clutter with `FormBuilder`

```ts
export class HeroDetailComponent implements OnInit {

  heroForm: FormGroup;

  constructor(private fb: FormBuilder) { }

  ngOnInit(): void {
    this.heroForm = this.fb.group({
      name: ''
    });
  }
}
```

Creating FormGroup with `FormBuilder` is much more simple and do the same thing with the `new` statements.

We create FormControl with `the key and an empty string` without any validations.

### As a new feature, the hero's name should be required.
Check the input value with `Validators.required`.

```ts

```

With the `required` validator, the form's status will be INVALID it the name is empty.

![](/images/2017-04-18-00-49-10.jpg)

Status will be valid if we enter a name.

![](/images/2017-04-18-00-49-41.jpg)

> see Form Validations for details: https://angular.io/docs/ts/latest/cookbook/form-validation.html

## Fill the formGroup and template to show hero detail.

```ts
import { Component, OnInit } from '@angular/core';
import { FormControl, FormGroup, FormBuilder, Validators } from '@angular/forms';
import { states } from 'app/demo-components/react-form/data-model';

@Component({
  selector: 'hero-detail',
  templateUrl: './hero-detail.component.html',
  styleUrls: ['./hero-detail.component.scss']
})
export class HeroDetailComponent implements OnInit {

  heroForm: FormGroup;
  states = states;

  constructor(private fb: FormBuilder) { }

  ngOnInit(): void {
    this.heroForm = this.fb.group({
      name: ['', Validators.required],
      street: '',
      city: '',
      state: '',
      zip: '',
      power: '',
      sidekick: ''
    });
  }
}
```

```html
<div class="container">
  <form [formGroup]="heroForm" novalidate>
    <div class="form-group">
      <label class="center-block">Name:</label>
      <input class="form-control" formControlName="name">
    </div>
    <div class="form-group">
      <label class="center-block">Street:
        <input class="form-control" formControlName="street">
      </label>
    </div>
    <div class="form-group">
      <label class="center-block">City:
        <input class="form-control" formControlName="city">
      </label>
    </div>
    <div class="form-group">
      <label class="center-block">State:
        <select class="form-control" formControlName="state">
            <option *ngFor="let state of states" [value]="state">{{state}}</option>
        </select>
      </label>
    </div>
    <div class="form-group">
      <label class="center-block">Zip Code:
        <input class="form-control" formControlName="zip">
      </label>
    </div>
    <div class="form-group radio">
      <h4>Super power:</h4>
      <label class="center-block"><input type="radio" formControlName="power" value="flight">Flight</label>
      <label class="center-block"><input type="radio" formControlName="power" value="x-ray vision">X-ray vision</label>
      <label class="center-block"><input type="radio" formControlName="power" value="strength">Strength</label>
    </div>
    <div class="checkbox">
      <label class="center-block">
        <input type="checkbox" formControlName="sidekick">I have a sidekick.
      </label>
    </div>
  </form>
</div>

<p>Form value: {{ heroForm.value | json }}</p>
<p>Form status: {{ heroForm.status | json }}</p>
```
> Pay attention to the formGroupName and formControlName attributes. 
> They are the Angular directives that bind the HTML controls to the Angular FormGroup and FormControl properties in the component class.

## The form is too big and we should manage them with `Nasted FormGroup`

Create nested form group with FormBuilder.

```ts
    this.heroForm = this.fb.group({
      name: ['', Validators.required],
      address: this.fb.group({
        street: '',
        city: '',
        state: '',
        zip: ''
      }),
      power: '',
      sidekick: ''
    });
```

Wrap the address with div and add a `formGroupName` directive to bind the address.

```html
<div formGroupName="address" class="well well-lg">
  <!-- address list -->
</div>
```

## We can inspect the FormControl with `formGroup` and `get()` method.

```html
<p>Name value: {{ heroForm.get('name').value }}</p>
```

![](/images/2017-04-18-01-10-56.jpg)

## Populate the form model with setValue and patchValue
> Previously you created a control and initialized its value at the same time. You can also `initialize or reset the values` later with the setValue and patchValue methods.

### form.setValue

When using `form.setValue`, it will assign `every` control as once. It will check the parameter's structure and return helpful error message. 

But the `patchValue` will fail silently.

### form.patchValue

With patchValue you can assign specified control's value with an object of key/value pairs.

### When to set form model values (ngOnChanges)

As if the form is editing a hero which is an input property that binded by the parent component. When the parent component have changed the value, the `formGroup` need to be updated by `setValue` method inside `ngOnChanges()` hook.

### We can `reset` the form's status and value with `formGroup.reset()` method.

The reset method accepts an `optional` state parameter that specify the state to be set. It will use `setValue` method to change state and controls.

```ts
ngOnChanges() {
  this.heroForm.reset({
    name: this.hero.name,
    address: this.hero.addresses[0] || new Address()
  });
}
```

## Use FormArray to present an array of FormGroups

FormGroup and FormControl only hold fixed number of properties.
FormArray supports an arbitrary number of controls or groups.

A hero can have many lairs.
### Change the formGroup from address to secret lairs

```ts
this.heroForm = this.fb.group({
  name: ['', Validators.required],
  secretLairs: this.fb.array([])
});
```
- Changing the `form model's property name` is an important point: the form model doesn't have to match the data model.

### Set the secretLaires

```ts
const addresses = [
  {street: '123 Main',  city: 'Anywhere', state: 'CA',  zip: '94801'},
  {street: '456 Maple', city: 'Somewhere', state: 'VA', zip: '23226'},
];

// map model array to FormGroup array.
const addressesGroups = addresses.map(address => this.fb.group(address));

// create FormArray by build method and FormGroup[]
const addressFormArray = this.fb.array(addressesGroups);

// override the controller with setControl
this.heroForm.setControl('secretLairs', addressFormArray);
```

- Notice that setControl method will override the control, instead of setValue method which will update the control's value.
- Notice also that the secretLairs FormArray contains FormGroups, not Addresses.

### Displaying the FormArray

```html
  <div formArrayName="secretLairs" class="well well-lg">
    <div *ngFor="let address of secretLairs.controls; let i=index" [formGroupName]="i" >
      <!-- The repeated address template -->
    </div>
  </div>
```

- With `formArrayName` directive.
- `secretLairs` access in *ngFor directive will call a getter method.

```ts
  get secretLairs() {
    return this.heroForm.get('secretLairs') as FormArray;
  }
```
- `formGroupName` in the loop bind to the index in the array.


### When adding a new address, we need to push a FormGroup object.

```ts
addLair() {
  this.secretLairs.push(this.fb.group(new Address()));
}
```

## Observe control changes

Angular calls ngOnChanges when the input hero is changed, but Angular does not call it when the hero's property is changed.

We can learn about those changes with subscribing to one of the form control.

```ts
nameChangeLog: string[] = [];
logNameChange() {
  const nameControl = this.heroForm.get('name');
  nameControl.valueChanges.forEach(
    (value: string) => this.nameChangeLog.push(value)
  );
}
```

