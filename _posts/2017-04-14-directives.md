---
layout: post
title:  "Directives"
date:   2017-04-14 23:44:19
categories: Angular
---

# Directives

> Component is the sub class of Directive, it can be treated as a directive with template.

## There are three kinds of directive
- Component - directives with template 
- Attribute Directives - change the behaver or appearance of an existing DOM element
- Structural Directives - change the DOM layout

## Lifefycle of directives

### Some hooks exist in both component and directive

- ngOnInit 
- ngOnChanges
- ngDoCheck
- ngOnDestroy

### Some hooks only exist in component
- ngAfterContentInit
- ngAfterContentChecked
- ngAfterViewInit
- ngAfterViewChecked

## Native Directives

There are directives offered by Angular including `Common Directives`, `Router Directives` and `Form Directives`.

## Make a Attribute Directive

- A class with a `@Directuve` defined.
- Match DOM elements with `selector`.
- Get the reference of the element with `ElementRef` injected in the constructor.

> see [Developing attribute directive](https://stoneyangxu.github.io/angular/2017/04/13/developing-attribute-directive.html) for details.

## Make a Structural Directive

### Feature - an `unless` directive which will show the element when its value is false

```ts
  it('should show element when `myUnless` is false', () => {
    const fixture = createGenericTestComponent(`
      <div class="content" *myUnless="false">Content</div>
    `, TestComponent);

    const el = fixture.debugElement.query(By.css('.content'));
    expect(el).toBeTruthy();
  });

  it('should remove element when `myUnless` is true', () => {
    const fixture = createGenericTestComponent(`
      <div class="content" *myUnless="true">Content</div>
    `, TestComponent);

    const el = fixture.debugElement.query(By.css('.content'));
    expect(el).toBeFalsy();
  });
```

> Error: No provider for TemplateRef!
> You missed the * in front of structural directive.

### Create a directive with `TemplateRef and ViewContainerRef` injected

```ts
import { Directive, Input, ViewContainerRef, TemplateRef } from '@angular/core';

@Directive({
  selector: '[myUnless]'
})
export class MyUnlessDirective {

  constructor(
    private tempalteRef: TemplateRef<any>,
    private viewContainerRef: ViewContainerRef
  ) { }

  @Input() set myUnless(unless: boolean) {
    if (unless) {
      this.viewContainerRef.clear();
    } else {
      this.viewContainerRef.createEmbeddedView(this.tempalteRef);
    }
  }
}
```

- tempalteRef is used to access the template 
- viewContainerRef is the renderer

## Structural directives, like ngIf, do their magic by using the HTML 5 template tag.
> Outside of an Angular app, the <template> tag's default CSS display property is none. It's contents are invisible within a hidden document fragment.
> Inside of an app, Angular removes the<template> tags and their children. The contents are gone — but not forgotten as we'll see soon.

![](/images/2017-04-15-00-43-12.jpg)

![](/images/2017-04-15-00-59-48.jpg)

## The asterisk (*) effect
The asterisk is "syntactic sugar"

```html
<!-- Examples (A) and (B) are the same -->
<!-- (A) *ngIf paragraph -->
<p *ngIf="condition">
  Our heroes are true!
</p>

<!-- (B) [ngIf] with template -->
<template [ngIf]="condition">
  <p>
    Our heroes are true!
  </p>
</template>
```

# Concepts

## <template>

> The HTML <template> element is a mechanism for holding client-side content that is not to be rendered when a page is loaded but may subsequently be instantiated during runtime using JavaScript.

### We use <script> tag to do this before it is supported with HTML5

```html
<script id="tpl-mock" type="text/template">
   <span>I am span in mock template</span>
</script>
```

### tempalte tag in HTML5

```html
<template id="tpl">
    <span>I am span in template</span>
</template>
```

### <ng-template> is suggested in angular


![](/images/2017-04-15-01-05-19.jpg)


```html
  <ng-template>
    <span>I am span in template</span>
  </ng-template>
```

### We fill a container with templte

```html
<!-- Template Container -->
<div class="tpl-container"></div>
<!-- Template -->
<template id="tpl">
    <span>I am span in template</span>
</template>
<!-- Script -->
<script type="text/javascript">
    (function renderTpl() {
        if ('content' in document.createElement('template')) {
            var tpl = document.querySelector('#tpl'); // --> templateRef
            var tplContainer = document.querySelector('.tpl-container'); // --> viewContainerRef
            var tplNode = document.importNode(tpl.content, true); 
            tplContainer.appendChild(tplNode); // --> viewContainerRef.createEmbeddedView
        } else {
            throw  new Error("Current browser doesn't support template element");
        }
    })();
</script>
```

## TemplateRef, ViewContainerRef and EmbeddedViewRef
### Results in browser when using structural directive

![](/images/2017-04-15-01-14-19.jpg)

The displayed element existed in nodes property.

![](/images/2017-04-15-01-16-08.jpg)

## ViewContainerRef

```ts
/**
 * @license
 * Copyright Google Inc. All Rights Reserved.
 *
 * Use of this source code is governed by an MIT-style license that can be
 * found in the LICENSE file at https://angular.io/license
 */
import { Injector } from '../di/injector';
import { ComponentFactory, ComponentRef } from './component_factory';
import { ElementRef } from './element_ref';
import { NgModuleRef } from './ng_module_factory';
import { TemplateRef } from './template_ref';
import { EmbeddedViewRef, ViewRef } from './view_ref';
/**
 * Represents a container where one or more Views can be attached.
 *
 * The container can contain two kinds of Views. Host Views, created by instantiating a
 * {@link Component} via {@link #createComponent}, and Embedded Views, created by instantiating an
 * {@link TemplateRef Embedded Template} via {@link #createEmbeddedView}.
 *
 * The location of the View Container within the containing View is specified by the Anchor
 * `element`. Each View Container can have only one Anchor Element and each Anchor Element can only
 * have a single View Container.
 *
 * Root elements of Views attached to this container become siblings of the Anchor Element in
 * the Rendered View.
 *
 * To access a `ViewContainerRef` of an Element, you can either place a {@link Directive} injected
 * with `ViewContainerRef` on the Element, or you obtain it via a {@link ViewChild} query.
 * @stable
 */
export declare abstract class ViewContainerRef {
    /**
     * Anchor element that specifies the location of this container in the containing View.
     * <!-- TODO: rename to anchorElement -->
     */
    readonly abstract element: ElementRef;
    readonly abstract injector: Injector;
    readonly abstract parentInjector: Injector;
    /**
     * Destroys all Views in this container.
     */
    abstract clear(): void;
    /**
     * Returns the {@link ViewRef} for the View located in this container at the specified index.
     */
    abstract get(index: number): ViewRef | null;
    /**
     * Returns the number of Views currently attached to this container.
     */
    readonly abstract length: number;
    /**
     * Instantiates an Embedded View based on the {@link TemplateRef `templateRef`} and inserts it
     * into this container at the specified `index`.
     *
     * If `index` is not specified, the new View will be inserted as the last View in the container.
     *
     * Returns the {@link ViewRef} for the newly created View.
     */
    abstract createEmbeddedView<C>(templateRef: TemplateRef<C>, context?: C, index?: number): EmbeddedViewRef<C>;
    /**
     * Instantiates a single {@link Component} and inserts its Host View into this container at the
     * specified `index`.
     *
     * The component is instantiated using its {@link ComponentFactory} which can be
     * obtained via {@link ComponentFactoryResolver#resolveComponentFactory}.
     *
     * If `index` is not specified, the new View will be inserted as the last View in the container.
     *
     * You can optionally specify the {@link Injector} that will be used as parent for the Component.
     *
     * Returns the {@link ComponentRef} of the Host View created for the newly instantiated Component.
     */
    abstract createComponent<C>(componentFactory: ComponentFactory<C>, index?: number, injector?: Injector, projectableNodes?: any[][], ngModule?: NgModuleRef<any>): ComponentRef<C>;
    /**
     * Inserts a View identified by a {@link ViewRef} into the container at the specified `index`.
     *
     * If `index` is not specified, the new View will be inserted as the last View in the container.
     *
     * Returns the inserted {@link ViewRef}.
     */
    abstract insert(viewRef: ViewRef, index?: number): ViewRef;
    /**
     * Moves a View identified by a {@link ViewRef} into the container at the specified `index`.
     *
     * Returns the inserted {@link ViewRef}.
     */
    abstract move(viewRef: ViewRef, currentIndex: number): ViewRef;
    /**
     * Returns the index of the View, specified via {@link ViewRef}, within the current container or
     * `-1` if this container doesn't contain the View.
     */
    abstract indexOf(viewRef: ViewRef): number;
    /**
     * Destroys a View attached to this container at the specified `index`.
     *
     * If `index` is not specified, the last View in the container will be removed.
     */
    abstract remove(index?: number): void;
    /**
     * Use along with {@link #insert} to move a View within the current container.
     *
     * If the `index` param is omitted, the last {@link ViewRef} is detached.
     */
    abstract detach(index?: number): ViewRef | null;
}

```
