---
layout: post
title:  "Angular Testing"
date:   2017-04-18 23:20:45
categories: Angular
---

# Angular Testing

Notes when reading [Angular's Testing Document](https://angular.io/docs/ts/latest/guide/testing.html)

What is tests used for?
- Guards against breaking existing codes.
- Declare how the code runs and how to use is.
- Reveal the mistakes in design and implementation.

## Testing Tools
- Jasmine - the basic framework to run tests in a browser.
- Karma - an ideal tool for running tests.
- Angular testing utilities - creating test environment for Angular application, tests control the application as they interact within real environment.
- Protractor - to write and run e2e tests. E2E test runs the real application and we explore the application as users experience.

## Isolated unit tests and Angular testing utilities
- Isolated unit tests examine an instance of class with `new`, supplying doubles with constructor parameters. 
- Isolated unit tests can't reveal how components interact with Angular. It's suitable for `Pipe` and `Service`.
- Angular testing utilities reveals how the component interact with its own template and other components.

## Test a component
### Name and position
If we have a BannerComponent in `banner.component.ts`, the test file should be a `banner.component.spec.ts` in same folder next to the component. We do this to against switching folders when writing tests, especially when working in TDD approach.

### Preparing test environment
Before running each test, we build the component with `TestBed` utility in `beforeEach` method.

```ts
    TestBed.configureTestingModule({
      declarations: [ BannerComponent ], // declare the test component
    });
    fixture = TestBed.createComponent(BannerComponent);
    comp = fixture.componentInstance; // BannerComponent test instance
```

### TestBed
`TestBed.configureTestingModule` creates a test module and detach the tested component form its real application module.
The metadata object passed to the method declares the component we are going to test, and we need to `imports` other modules and `provides` that the component needs.
`TestBed.createComponent` create component instance and returns a component test fixture.
> We can't call any methods of TestBed after createComponent, otherwise TestBed will throw an error.

The `fixture` surrounds the tested component instance and debugElement to handle the DOM element.
We query elements with debugElement and CSS selector, then get a different debugElement associated with the matching element.

- query - get the first matching element.
- queryAll - get an array of matching elements.

## The tests

```ts
it('should display original title', () => {
  fixture.detectChanges();
  expect(el.textContent).toContain(comp.title);
});
```

detectChanges - tells Angular to perform change detection, `triggers data binding and propagation` from component property to DOM elements.




