---
layout: post
title:  "How to use ControlValueAccessor interface"
date:   2017-05-03 23:03:22
categories: Angular
---

# How to use ControlValueAccessor interface

Create a `switch-button` component to explore this interface.
There is only a button styled by bootstrap.

![](/images/2017-05-03-23-04-50.jpg)

When clicking the button, change the style and label.

![](/images/2017-05-03-23-05-25.jpg)

That's all.

## The template is simple enough, only contains a button that will style its class with `value` property and catch the `click` event.

```ts
<button type="button" (click)="switch()" class="btn" [ngClass]="{'btn-success': value, 'btn-warning': !value}">
  {{ getLabel() }}
</button>
```
## Create our component class that implements `ControlValueAccessor`

```ts
import {Component, Input, OnInit} from '@angular/core';
import {ControlValueAccessor} from '@angular/forms';

@Component({
  selector: 'switch-button',
  templateUrl: './switch-button.component.html',
  styleUrls: ['./switch-button.component.scss']
})
export class SwitchButtonComponent implements ControlValueAccessor {


  @Input() value = true;

  switch() {
    this.value = !this.value;
    this.onChange();
    this.onTouch();
  }

  getLabel() {
    return this.value ? 'On' : 'Off';
  }

  writeValue(obj: any): void {
    throw new Error('Method not implemented.');
  }

  private onChange() {
    console.log('onChanged');
  };
  private onTouch() {
    console.log('onTouched');
  };

  registerOnChange(fn: any): void {
    console.log('registerOnChange');
    this.onChange = fn;
  }

  registerOnTouched(fn: any): void {
    console.log('registerOnTouched');
    this.onTouch = fn;
  }

  setDisabledState(isDisabled: boolean): void {
    throw new Error('Method not implemented.');
  }
}
```

Four methods need to implement:
- registerOnChange
- registerOnTouched
- setDisabledState 
- writeValue
And we are going to find out how to work with them.

## registerOnChange & registerOnTouched

![](/images/2017-05-03-23-12-17.jpg)

As described in Angular API document, they give us functions when the control is touched and changed.
- When `registerOnChange & registerOnTouched` will be called?
- How to use the parameter we got from Angular?

We have add some logs and save the parameter to an exist method.

Then we try to load the component and click the button.

![](/images/2017-05-03-23-15-45.jpg)

We found that the register functions have never been called, the original method is used when clicking. What's wrong with it?

We guess that the component need to work with `ngModel`.

```html
<switch-button [(ngModel)]="switchButtonValue"></switch-button>
```
But when loading the component, we got an error.

![](/images/2017-05-03-23-34-07.jpg)

By checking some opensource project, we found that when working with `ControlValueAccessor` we need to `provide a NG_VALUE_ACCESSOR`

```ts
@Component({
  selector: 'switch-button',
  templateUrl: './switch-button.component.html',
  styleUrls: ['./switch-button.component.scss'],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => SwitchButtonComponent),
      multi: true
    }
  ]
})
```
This provider means that when the host component need to load `NG_VALUE_ACCESSOR`, a `SwitchButtonComponent` will be provided . `multi` allows multiple values for a single `token`.

![](/images/2017-05-03-23-45-12.jpg)

After that the methods will be called when loading the component. And the host's class will be changed when clicking.

![](/images/2017-05-03-23-47-23.jpg)
After clicking:
![](/images/2017-05-03-23-47-48.jpg)

## writeValue
Print the `switchButtonValue` in the host component for debugging.

```ts
<switch-button [(ngModel)]="switchButtonValue"></switch-button>

{{ switchButtonValue }}
```

And we will change the value to `false` in the host component after 5 seconds.

```ts
  switchButtonValue = true;

  ngOnInit() {
    setTimeout(() => {
      this.switchButtonValue = false;
    }, 5000);
  }
```

After 5 seconds, the value in host component is changed and `writeValue` will be called.

```ts
  writeValue(obj: any): void {
    console.log('writeValue', obj);
    this.value = obj;
  }
```

![](/images/2017-05-04-00-02-35.jpg)

But when clicking the button, the host component didn't discover the changing.

![](/images/2017-05-04-00-06-34.jpg)
 
## Emit value change
Add more log to print the function passed by `registerOnChange`.

```ts
  registerOnChange(fn: any): void {
    console.log('registerOnChange', fn);
    this.onChange = fn;
  }
```
We found that the function accept a parameter named `newValue`, maybe it's what we want.

![](/images/2017-05-04-00-11-42.jpg)

Pass the new value as a parameter when calling `onChange` method.

```ts
  switch() {
    this.value = !this.value;
    this.onChange(this.value);
    this.onTouch();
  }
```
Then it works well.

![](/images/2017-05-04-00-14-30.jpg)

![](/images/2017-05-04-00-14-42.jpg)

## The example above works well with `ngModel` and how about `FormControl`

Create a FormControl in the host component.

```ts

  switchButtonControl = new FormControl(true);

  ngOnInit() {
    setTimeout(() => {
      this.switchButtonControl.setValue(false);
    }, 5000);
  }
```
```html
<switch-button [formControl]="switchButtonControl"></switch-button>
<pre>
  {{ switchButtonControl | json }}
</pre>
```

It works, and the status of control will be updated when clicking.

![](/images/2017-05-04-00-23-20.jpg)

## At last, try `setDisabledState`

Enable and disable it with `switchButtonControl`.

```ts
    setTimeout(() => {
      // this.switchButtonValue = false;
      this.switchButtonControl.disable();

      setTimeout(() => {
        this.switchButtonControl.enable();
      }, 5000)
    }, 5000);
```
And change the button status when enabled or disabled.

```html
<button type="button"  (click)="switch()" [disabled]="disabled" class="btn" [ngClass]="{'btn-success': value, 'btn-warning': !value}">
  {{ getLabel() }}
</button>
```

```ts
  disabled = false;
  
  setDisabledState(isDisabled: boolean): void {
    console.log('setDisabledState', isDisabled);
    this.disabled = isDisabled;
  }
```
The `setDisabledState` is called when control's `enable/disable` method is called.

![](/images/2017-05-04-00-29-18.jpg)
