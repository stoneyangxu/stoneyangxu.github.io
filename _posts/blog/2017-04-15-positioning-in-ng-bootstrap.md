---
layout: post
title:  "Positioning in ng-bootstrap"
date:   2017-04-15 12:13:46
categories: Angular
---

# Positioning in ng-bootstrap
> A set of utility methods that can be use to retrieve position of DOM elements.It is meant to be used where we need to absolute-position DOM elements in relation to other, existing elements (this is the case for tooltips, popovers, typeahead suggestions etc.).

## There is only one public function exported for the consumer

The `positionElements` function is used set the position of `targetElement` depends on the position of `hostElement`.
Specify the position with `placement` parameter, which can be `"top" | "bottom" | "left" | "right"` or mixed value like `top-center`.
The `appendToBody` is a boolean value that tells Positioning to calculate position relative to body.
After calculation, set the target element's `top and left`.

```ts
const positionService = new Positioning();
export function positionElements(
    hostElement: HTMLElement, targetElement: HTMLElement, placement: string, appendToBody?: boolean): void {
  const pos = positionService.positionElements(hostElement, targetElement, placement, appendToBody);

  targetElement.style.top = `${pos.top}px`;
  targetElement.style.left = `${pos.left}px`;
}
```

## The `positionElements` function returns a `ClientRect` instance to describe the position in the screen.

```ts
positionElements(hostElement: HTMLElement, targetElement: HTMLElement, placement: string, appendToBody?: boolean):
      ClientRect
```

> `ClientRect` is an Object that representing the area of the screen occupied by the range.

```ts
interface ClientRect {
    bottom: number;
    readonly height: number;
    left: number;
    right: number;
    top: number;
    readonly width: number;
}
```

## If `appendToBody` is true

### Use `this.offset(hostElement, false)` to get the offset of the HostElement first.

```ts
    const hostElPosition = appendToBody ? this.offset(hostElement, false) : this.position(hostElement, false);
```

### Calculate the `shiftWidth` and `shiftHeight` that will be used to move the target element depends on the `placementSecondary`

```ts
    const shiftWidth: any = {
      left: hostElPosition.left, 
      center: hostElPosition.left + hostElPosition.width / 2 - targetElement.offsetWidth / 2,
      right: hostElPosition.left + hostElPosition.width
    };
    const shiftHeight: any = {
      top: hostElPosition.top,
      center: hostElPosition.top + hostElPosition.height / 2 - targetElement.offsetHeight / 2,
      bottom: hostElPosition.top + hostElPosition.height
    };
```

#### Horizontal
- left - the left position of the host element.
- center - the target element and the host element align by `center line`. 
- right - the right position of the hose element.

#### Vertical
- top - the top position of the host element.
- center - the target element and the host element align by `middle line`.
- bottom: the bottom position of the host element. 

### Get the placement by splitting the parameter

```ts
    const placementPrimary = placement.split('-')[0] || 'top';
    const placementSecondary = placement.split('-')[1] || 'center';
```
### Init the target element's position, just to the top-left of body or parent element.

```ts
    const targetElPosition: ClientRect = {
      height: targetElBCR.height || targetElement.offsetHeight,
      width: targetElBCR.width || targetElement.offsetWidth,
      top: 0,
      bottom: targetElBCR.height || targetElement.offsetHeight,
      left: 0,
      right: targetElBCR.width || targetElement.offsetWidth
    };
```

### Then calculate the final position by `placement`

```ts
    switch (placementPrimary) {
      case 'top':
        targetElPosition.top = hostElPosition.top - targetElement.offsetHeight;
        targetElPosition.bottom += hostElPosition.top - targetElement.offsetHeight;
        targetElPosition.left = shiftWidth[placementSecondary];
        targetElPosition.right += shiftWidth[placementSecondary];
        break;
      case 'bottom':
        targetElPosition.top = shiftHeight[placementPrimary];
        targetElPosition.bottom += shiftHeight[placementPrimary];
        targetElPosition.left = shiftWidth[placementSecondary];
        targetElPosition.right += shiftWidth[placementSecondary];
        break;
      case 'left':
        targetElPosition.top = shiftHeight[placementSecondary];
        targetElPosition.bottom += shiftHeight[placementSecondary];
        targetElPosition.left = hostElPosition.left - targetElement.offsetWidth;
        targetElPosition.right += hostElPosition.left - targetElement.offsetWidth;
        break;
      case 'right':
        targetElPosition.top = shiftHeight[placementSecondary];
        targetElPosition.bottom += shiftHeight[placementSecondary];
        targetElPosition.left = shiftWidth[placementPrimary];
        targetElPosition.right += shiftWidth[placementPrimary];
        break;
    }
```
### Round the offset value and returns.

```ts
    targetElPosition.top = Math.round(targetElPosition.top);
    targetElPosition.bottom = Math.round(targetElPosition.bottom);
    targetElPosition.left = Math.round(targetElPosition.left);
    targetElPosition.right = Math.round(targetElPosition.right);

    return targetElPosition;
```
## How to use it?

### There is a button and a tooltip content

```html
<button #hostButton class="btn">A button</button>

<div #toptipWin class="tooltip show">
  <div class="tooltip-inner">
    Content in tooltip!
  </div>
</div>
```

### Get element reference in component and use positioning to set the position

```ts
  @ViewChild('hostButton') host: ElementRef;
  @ViewChild('toptipWin') target: ElementRef;
  ngOnInit() {
    positionElements(
      this.host.nativeElement,
      this.target.nativeElement,
      'top-right',
    );
  }
```

![](/images/2017-04-16-15-12-20.jpg)

