---
layout: post
title:  "Developing a Alert Component"
date:   2017-04-12 22:39:10
categories: Angular
---

# Developing a [Alert Componnent](https://ng-bootstrap.github.io/#/components/alert)

Looks like this

![](/images/2017-04-12-22-40-11.jpg)

# Features
- Base on `bootstrap`
- Use `type` to select a displaying style, including `success`, `info`, `warning` and `danger`
- Use `dismissible` to specify whether to show `close button`
- When clicking close button, emit an `close event` that will be subscribed by parent component

# Codes
## Component

```ts
import { Component, OnInit, Input, EventEmitter, Output } from '@angular/core';

@Component({
  selector: 'my-alert',
  templateUrl: './my-alert.component.html',
  styleUrls: ['./my-alert.component.scss']
})
export class MyAlertComponent {

  @Input() type: string;
  @Input() dismissible: boolean;

  constructor() {
    this.dismissible = true;
  }
}
```

## Template

```html
<div class="alert-content" [class]="'alert alert-' + type" [ngClass]="{'alert-dismissible': dismissible}">
  <button *ngIf="dismissible" type="button" class="close" aria-label="Close">
    <span aria-hidden="true">&times;</span>
  </button>
  <ng-content></ng-content>
</div>
```

# And the most important is the `spec`

## First of all, instead of testing a standalone component, test the component with a mocked component as the host of the `MyAlertComponent` is much more useful. It will show `how the component will be used`

Create an empty component with the reference of MyAlertComponent to be tested.

```ts
@Component({
  selector: 'test-component',
  template: ''
})
export class TestComponent {
  @ViewChild(MyAlertComponent) myAlertComponent: MyAlertComponent;

  closeAlert() {
  }
}
```
- `@ViewChild` will get the child component with the component type `MyAlertComponent`.
- `template` is empty and we will override it with html in each spec to show `how the component is used`.
- `closeAlert` will be used to process close event later.

## Init the test enviroment with the mocked host component and MyAlertCompoennt to be tested

```ts
  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TestComponent, MyAlertComponent]
    });
  }));
```
- Remember that `do not` compile the component with `compileComponents` method, otherwise we can't override the host component with specified template.

## In a specified test case, we create a html which shows how the component is used and override the host's template

```ts
  const html = `
    <my-alert>
      <strong>Warning!</strong>Alert message!
    </my-alert>
  `;

fixture = createGenericTestComponent(html, TestComponent);

const contentElement = fixture.debugElement.query(By.css('.alert-content')).nativeElement;
expect(contentElement.textContent).toContain(`Alert message!`);
```

- `createGenericTestComponent` is a common method that `override` the empty template in the host which will return a fixture.

```ts
export function createGenericTestComponent<T>(html: string, type: {new (...args: any[]): T}): ComponentFixture<T> {
  TestBed.overrideComponent(type, {set: {template: html}});
  const fixture = TestBed.createComponent(type);
  fixture.detectChanges();
  return fixture as ComponentFixture<T>;
}
```

- We get the element with `fixture.debugElement.query`.
- And then check whether the result element contains a correct content that we create in `html`.

# Then we develop the event emitter when clicking close button with TDD approach

## Create a spec first to show `how we will process with the event` and `when the event will be emitted`

```ts
  it('should emit close event when clicking close button', () => {
    const customFixture = createGenericTestComponent(`
      <my-alert type='success' (close)="closeAlert()">
        Message
      </my-alert>
    `, TestComponent);

    const spy = spyOn(customFixture.componentInstance, 'closeAlert');

    const button = customFixture.debugElement.query(By.css('button.close')).nativeElement;
    button.click();

    expect(spy).toHaveBeenCalled();
  });
```
- `(close)="closeAlert()"` shows that when a `close` event is emitted by MyAlertComponent, `closeAlert` method in the host component will be called 
- We create a spy to check whether closeAlert is called in a suitable time
- Then get the button element displayed in MyAlertCompnent instance and click the button with `button.click();`
- In the end, we check whether the event is processed with `toHaveBeenCalled` that is offered by Jasmine

## We run the test and get a fealure because we haven't implement anything
## Bind clicking event on the close button to the method in component

```html
  <button *ngIf="dismissible" type="button" class="close" aria-label="Close" (click)="onClose()">
    <span aria-hidden="true">&times;</span>
  </button>
```

## Create an `onClick` method in the component

```ts
  onClose() {
    // process with the clicking event
  }
```

## Create an `output property` that will be cached by the parent

```ts
  @Output() close = new EventEmitter();
```

## In the end, fill `onClose` to emit an event

```ts
  onClose() {
    this.close.emit(null);
  }
```

## And then the spec is passed


![](/images/2017-04-12-23-33-20.jpg)

# Gains

## Bootstrap supports

- supports alert style with `alert alert-{type}` class
- supports close button with `close` class inside `alert`

## Component test

- A `mocked host component` is much more helpful, it is more convenient for `building a test scene` and helps the code consumer to know `how the component is used`
- Build the host component with `a specified html template` and override the host component's with `TestBed.overrideComponent` method which is extracted to a common file
- `Do not compile component` before overriding, otherwise jasmine will throw a runtime error
- TDD helps us to figure out what we are going to do before we are caught in details

