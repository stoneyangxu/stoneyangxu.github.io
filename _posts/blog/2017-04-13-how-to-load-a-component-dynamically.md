---
layout: post
title:  "How to load a component dynamically"
date:   2017-04-13 23:59:29
categories: Angular
---

# How to load a component dynamically

> Component templates are not always fixed. An application may need to load new components at runtime.

# Build a component to be loaded dynamically

## Feature - creating a new component to display current time with a random background-color when clicking the button each time.

### A component to display current time

```ts
  it('should display current time', () => {
    fixture = createGenericTestComponent(`
      <display-time></display-time>
    `, TestComponent);

    const element = fixture.debugElement.query(By.css('.create-time')).nativeElement;
    expect(element.textContent).toBeTruthy();
  });
```

### Set background-color with `bgColor` property

```ts
  it('should set background-color with bgColor property', () => {
    fixture = createGenericTestComponent(`
      <display-time bgColor="green"></display-time>
    `, TestComponent);

    const element = fixture.debugElement.query(By.css('.create-time')).nativeElement;
    expect(element.style.backgroundColor).toBe('green');
  });
```

## Fix error specs and finish the component

```ts
import { Component, OnInit, Input } from '@angular/core';
import * as moment from 'moment';

@Component({
  selector: 'display-time',
  templateUrl: './display-time.component.html',
  styleUrls: ['./display-time.component.scss']
})
export class DisplayTimeComponent implements OnInit {

  createTime: string;
  @Input() bgColor: string;

  ngOnInit() {
    this.createTime = moment().format();
  }
}
```

```html
<p class="create-time" [style.backgroundColor]="bgColor">
  {{createTime}}
</p>
```

# Build a parent component with a button

## Feature - component with a button and a place holder to load `DisplayTimeComponent`

### Displaying a button

```ts
  it('should display with a button', () => {
    fixture = createGenericTestComponent(`
      <holder></holder>
    `, TestComponent);

    const buttonElement = fixture.debugElement.query(By.css('button'));
    expect(buttonElement).toBeTruthy('display a button');
  });
```

### Loading DisplayTimeComponent instance in the placeholder

```ts
  it('should load DisplayTimeComponent in the placeholder', () => {
    fixture = createGenericTestComponent(`
      <holder></holder>
    `, TestComponent);

    const placeholder = fixture.debugElement.query(By.css('.create-time'));
    expect(placeholder.nativeElement.textContent).toBeTruthy('display create time');
  });
```
## Fix error specs

### Fill the template file with a button and a placeholder

```html
<button type="button" class="btn btn-primary">button</button>
<ng-template></ng-template>
```

### How can we `get reference to the placeholder`? - We create a directive to do this.

The directive will be added to `ng-template` element.
It will be injected with a `ViewContainerRef` that reference to the placeholder element.

```ts
import { Directive, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[dtPlaceholder]'
})
export class DtPlaceholderDirective {

  constructor(viewContainerRef: ViewContainerRef) { }

}
```

Add the directive selector to `ng-template` element.

```html
<ng-template class="placeholder" dtPlaceholder></ng-template>
```

We get the reference of `DtPlaceholderDirective` with `@ViewChild`.

```ts
  @ViewChild(DtPlaceholderDirective) dtPlaceholder: DtPlaceholderDirective;
```

Then we can get the reference of `placeholder` with `dtPlaceholder.viewContainerRef`

### When the hold component is created, we use `ComponentFactoryResolver` to load a `DisplayTimeComponent` instance inside `placeholder`

Add `entryComponents` to the module decorator first.
> Error: No component factory found for DisplayTimeComponent. Did you add it to @NgModule.entryComponents
![](/images/2017-04-14-01-03-27.jpg)

A `TestModule` need to be create for define `entryComponents` for TestBed, because `TestBed.configureTestingModule` do not support entryComponents option.

```ts
@NgModule({
  declarations: [DisplayTimeComponent],
  entryComponents: [
    DisplayTimeComponent,
  ]
})
class TestModule {}

TestBed.configureTestingModule({
  imports: [TestModule],
  declarations: [TestComponent, HolderComponent]
});
```
Finally, tests passed and the whole implement is: 

```ts
import { Component, OnInit, ViewChild, ComponentFactoryResolver } from '@angular/core';
import { DtPlaceholderDirective } from 'app/demo-components/load-component/dt-placeholder.directive';
import { DisplayTimeComponent } from 'app/demo-components/load-component/display-time/display-time.component';

@Component({
  selector: 'holder',
  templateUrl: './holder.component.html',
  styleUrls: ['./holder.component.scss']
})
export class HolderComponent implements OnInit {

  @ViewChild(DtPlaceholderDirective) dtPlaceholder: DtPlaceholderDirective;

  constructor(private componentFactoryResolver: ComponentFactoryResolver) { }

  ngOnInit() {
    const componentFactory = this.componentFactoryResolver.resolveComponentFactory(DisplayTimeComponent);
    const placeholderReference = this.dtPlaceholder.viewContainerRef;

    const componentRef = placeholderReference.createComponent(componentFactory);
    componentRef.instance.bgColor = 'yellow';
  }
}
```
- Inject `ComponentFactoryResolver` for creating component instance.
- Get child `DtPlaceholderDirective` instance with `@ViewChild`.
- Get `componentFactory` with method `componentFactoryResolver.resolveComponentFactory`
- Get the reference to `ng-template` that contains the attribute directive through `dtPlaceholder.viewContainerRef`.
- Load component dynamically with `placeholderReference.createComponent(componentFactory)`
