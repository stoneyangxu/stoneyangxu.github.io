---
layout: post
title:  "PopupService in ng-bootstrap"
date:   2017-04-15 12:13:46
categories: Angular
---

# PopupService in ng-bootstrap

It is used to load the component dynamically, resolve string content or templateRef.

Use the content to replace `<ng-content>` in the target component.

In the end, returns the new created component's reference.

## Constructor

```ts
  private _windowFactory: ComponentFactory<T>;

  constructor(
      type: any, private _injector: Injector, private _viewContainerRef: ViewContainerRef, private _renderer: Renderer,
      componentFactoryResolver: ComponentFactoryResolver) {
    this._windowFactory = componentFactoryResolver.resolveComponentFactory<T>(type);
  }
```

- `type` is a `Component Type` that will be rendered in the pop-up window.
- `injector` is provided by the host component.
- `viewContainerRef` is provided by the caller that refer to the host component.
- `renderer` is used to manage DOM elements in UI
> [Renderer2 in Angular API](https://angular.io/docs/ts/latest/api/core/index/Renderer2-class.html)
> [Rendering in Angular2](https://www.yearofmoo.com/2016/02/rendering-in-angular2.html)

- `componentFactoryResolver` is provided by host component that is used to load component dynamically

- `windowFactory` is created for loading component later.

## ContentRef
A container to hold `elements nodes` and `needed reference`

### ContentRef Class

```ts
export class ContentRef {
  constructor(public nodes: any[], public viewRef?: ViewRef, public componentRef?: ComponentRef<any>) {}
}
```
> todo

### ContentRef will be created when opening a pop-up window

```ts
  private _getContentRef(content: string | TemplateRef<any>, context?: any): ContentRef {
    if (!content) {
      return new ContentRef([]);
    } else if (content instanceof TemplateRef) {
      const viewRef = this._viewContainerRef.createEmbeddedView(<TemplateRef<T>>content, context);
      return new ContentRef([viewRef.rootNodes], viewRef);
    } else {
      return new ContentRef([[this._renderer.createText(null, `${content}`)]]);
    }
  }
```

- Depends on the type of `content`, create `ContentRef instance` in different ways.
- When content is empty, create an empty ContentRef
- When content is a `TemplateRef`, insert the template into host view container with `_viewContainerRef.createEmbeddedView`. And save the `element nodes` and `view reference` in ContentRef

![](/images/2017-04-15-12-58-51.jpg)

- When content is just a string, create standalone test element, and surround the element to match the needed format.

## Opening the pop-up window

```ts
  open(content?: string | TemplateRef<any>, context?: any): ComponentRef<T> {
    if (!this._windowRef) {
      this._contentRef = this._getContentRef(content, context);
      this._windowRef =
          this._viewContainerRef.createComponent(this._windowFactory, 0, this._injector, this._contentRef.nodes);
    }

    return this._windowRef;
  }
```

### If window haven't been created, create the ContentRef instance and then load component dynamically with `_viewContainerRef.createComponent`

```ts
this._viewContainerRef.createComponent(this._windowFactory, 0, this._injector, this._contentRef.nodes);
```

- `nodes` parameter will replace <ng-content> in the dynamically loaded component.
- It returns the reference to the loaded component. 
- Create the component next to the component where we got `_viewContainerRef`

![](/images/2017-04-15-15-19-52.jpg)

## If the content is a templateRef, PopupService only copy contents in template into `ng-conent`(without ng-content, nodes will exist behind new loaded component)

```html
<button class="btn" (click)="clickButton()">Button</button>

<ng-template>
  This is content in template!
</ng-template>
```

```ts
  @ViewChild(TemplateRef) templateRef: TemplateRef<any>;
  clickButton() {
    this.popupService.open(this.templateRef);
  }
```

![](/images/2017-04-15-15-28-44.jpg)



## When closing the pop-up window, remove created view from host container.

```ts
  close() {
    if (this._windowRef) {
      this._viewContainerRef.remove(this._viewContainerRef.indexOf(this._windowRef.hostView));
      this._windowRef = null;

      if (this._contentRef.viewRef) { // when creating pop-up with template
        this._viewContainerRef.remove(this._viewContainerRef.indexOf(this._contentRef.viewRef));
        this._contentRef = null;
      }
    }
  }
```

# The source

```ts
import {
  Injector,
  TemplateRef,
  ViewRef,
  ViewContainerRef,
  Renderer,
  ComponentRef,
  ComponentFactory,
  ComponentFactoryResolver
} from '@angular/core';

export class ContentRef {
  constructor(public nodes: any[], public viewRef?: ViewRef, public componentRef?: ComponentRef<any>) {}
}

export class PopupService<T> {
  private _windowFactory: ComponentFactory<T>;
  private _windowRef: ComponentRef<T>;
  private _contentRef: ContentRef;

  constructor(
      type: any, private _injector: Injector, private _viewContainerRef: ViewContainerRef, private _renderer: Renderer,
      componentFactoryResolver: ComponentFactoryResolver) {
    this._windowFactory = componentFactoryResolver.resolveComponentFactory<T>(type);
  }

  open(content?: string | TemplateRef<any>, context?: any): ComponentRef<T> {
    if (!this._windowRef) {
      this._contentRef = this._getContentRef(content, context);
      this._windowRef =
          this._viewContainerRef.createComponent(this._windowFactory, 0, this._injector, this._contentRef.nodes);
    }

    return this._windowRef;
  }

  close() {
    if (this._windowRef) {
      this._viewContainerRef.remove(this._viewContainerRef.indexOf(this._windowRef.hostView));
      this._windowRef = null;

      if (this._contentRef.viewRef) {
        this._viewContainerRef.remove(this._viewContainerRef.indexOf(this._contentRef.viewRef));
        this._contentRef = null;
      }
    }
  }

  private _getContentRef(content: string | TemplateRef<any>, context?: any): ContentRef {
    if (!content) {
      return new ContentRef([]);
    } else if (content instanceof TemplateRef) {
      const viewRef = this._viewContainerRef.createEmbeddedView(<TemplateRef<T>>content, context);
      return new ContentRef([viewRef.rootNodes], viewRef);
    } else {
      return new ContentRef([[this._renderer.createText(null, `${content}`)]]);
    }
  }
}

```
