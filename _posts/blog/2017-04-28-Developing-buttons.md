---
layout: post
title:  "Developing Radio Buttons Directive"
date:   2017-04-28 00:25:02
categories: Angular
---

# Developing Radio Buttons Directive
## Basic styles provided by Bootstrap

Bootstrap provides basic styles for radio buttons with `btn-group` and `btn` class.

```html
<div class="btn-group" role="group">
  <button type="button" class="btn btn-primary">Left</button>
  <button type="button" class="btn btn-primary">Middle</button>
  <button type="button" class="btn btn-primary">Right</button>
</div>
```
It looks like this:

![](/images/2017-04-28-00-28-14.jpg)

Then we are going to complete a radio buttons directive named `myRadioGroup` with Angular. 
It will manage radio status and bind value for Template-driven form and Reactive form.

## First of all, create directive files with `@angular/cli`

![](/images/2017-04-28-00-35-22.jpg)

Create our first test to make sure that the directive can be create.

```ts
  it('should create', () => {
    fixture = createGenericTestComponent(`
      <div class="btn-group" myRadioGroup>
        <button type="button" class="btn btn-primary">Left</button>
        <button type="button" class="btn btn-primary">Middle</button>
        <button type="button" class="btn btn-primary">Right</button>
      </div>
    `, TestComponent);
    component = fixture.componentInstance;
    expect(component).toBeTruthy('test component is created');
    expect(component.instance).toBeTruthy('directive is created');
  });
```
## The RadioGroup should contains children directives named `myRadio` that manage status of single button.

![](/images/2017-04-28-00-49-38.jpg)

```ts
  it('should create', () => {
    fixture = createGenericTestComponent(`
      <button type="button" myRadio class="btn btn-primary">Radio</button>
    `, TestComponent);
    component = fixture.componentInstance;
    instance = component.instance;

    expect(component).toBeTruthy('component is created');
    expect(instance).toBeTruthy('myRadio is created');
  });
```

### When the button is clicked, it will be `active`

```ts
  it('should switch `active` when clicking', () => {
    fixture = createGenericTestComponent(`
      <button type="button" myRadio class="btn btn-primary">Radio</button>
    `, TestComponent);
    component = fixture.componentInstance;
    instance = component.instance;

    const btn = getButton(fixture);
    btn.nativeElement.click();

    expect(btn.nativeElement.classList).toContain('active');

    btn.nativeElement.click();
    expect(btn.nativeElement.classList).not.toContain('active');
  });
```
Implements: 

```ts
@Directive({
  selector: '[myRadio]'
})
export class MyRadioDirective {

  private isActive: boolean;

  constructor(
    private elementRef: ElementRef,
    private renderer: Renderer
  ) { }

  @HostListener('click') onclick() {
    this.isActive = !this.isActive;
    this.renderer.setElementClass(this.elementRef.nativeElement, 'active', this.isActive);
  }
}
```

- isActive - save current button status.
- elementRef: ElementRef - injected by Angular, reference to the button element.
- renderer: Renderer - injected by Angular, used to manage DOM element.
- setElementClass - add/remove class to specified DOM element depends on the third parameter.


