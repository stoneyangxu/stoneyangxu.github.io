---
layout: post
title:  "Developing attribute directive"
date:   2017-04-13 22:07:44
categories: Angular
---

# Developing attribute directive

## The basic

### Feature - highlight an element with attribute directive

```ts
  it('should highlight element', () => {
    const fixture = createGenericTestComponent(`
      <div myHighlight>Content</div>
    `, TestComponent);

    const elemnt = fixture.debugElement.query(By.css('div')).nativeElement;

    expect(elemnt.style['background-color']).toBe('yellow');
  });
```
### Create directive with selector `[myHighlight]`

```ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[myHighlight]'
})
export class MyHighlightDirective {
  constructor(private elementRef: ElementRef) {
    elementRef.nativeElement.style['background-color'] = 'yellow';
  }
}
```
- Use `@Directive` to define the class as a Directive.
- `selector` defines a `CSS selector` to identify the HTML element that is associated with the directive.
- `elementRef` is an injected reference for accessing the element in our directive.
- We change the `background-color` with `nativeElement` property.

### Why we use `myHighlight` instead of `highlight`?
- To ensure they don't conflict with `standard HTML attribute`.
- To reduce the risk of colliding with `third-party directives`.
- Do not use `ng` prefix because it belongs to Angular

### How to use our directive?

- First of all, we need to add `MyHighlightDirective` to the module's `declarations`.
- And then, associate with an element like `<p myHighlight>Content</p>`.
- Element `<p>` is the `host` of the directive.

## Adding event listener to the host element associated with directive

### Feature - when mouseenter, background-color will be green. when mouseleave, it recovers

```ts
  it('should change the background-color when mouse enter', () => {
    const fixture = createGenericTestComponent(`
      <div myHighlight>Content</div>
    `, TestComponent);

    const elemnt: HTMLDivElement = fixture.debugElement.query(By.css('div')).nativeElement;

    elemnt.dispatchEvent(new Event('mouseenter'));
    expect(elemnt.style['background-color']).toBe('green');

    elemnt.dispatchEvent(new Event('mouseleave'));
    expect(elemnt.style['background-color']).toBe('yellow');
  });
```

### Listen to mouse event with `@HostListener` decorator

```ts
  @HostListener('mouseenter') onMouseEnter() {
    this.changeBackgroundColor('green');
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.changeBackgroundColor('yellow');
  }

  private changeBackgroundColor(color: string) {
    this.elementRef.nativeElement.style.backgroundColor = color;
  }
```

- `@HostListener` listen to the event of `host element`

## Bind a color with @Input()
### Feature - define the highlight color with an element attribute 

```ts
  it('should define the highlight color by attribute', () => {
    const fixture = createGenericTestComponent(`
      <div myHighlight highlightColor="pink">Content</div>
    `, TestComponent);

    const elemnt = fixture.debugElement.query(By.css('div')).nativeElement;
    expect(elemnt.style['background-color']).toBe('pink');
  });
```

### Get highlightColor with @Input() decorator

```ts
  @Input() highlightColor = 'yellow';

  constructor(private elementRef: ElementRef) {
  }

  ngOnInit(): void {
    this.changeBackgroundColor(this.highlightColor);
  }
```
- We can't set highlight color with `constructor` any more, binding @Input value `runs after` constructor.
- We move the code to `ngOnInit` method which runs after constructor and input binding

> OnNgInit - Initialize the directive/component after Angular first displays the data-bound properties and sets the directive/component's input properties.

