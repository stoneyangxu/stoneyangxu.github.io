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
`TestBed.configureTestingModule` creates a test module and detach the tested component from its real application module.
The `metadata` object passed to the method declares the component we are going to test, and we need to `imports` other modules and `provides` that the component needs.
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

### Automatic change detection

Detect the changes automatically with `ComponentFixtureAutoDetect` provided in the test module.

```ts
TestBed.configureTestingModule({
  declarations: [ BannerComponent ],
  providers: [
    { provide: ComponentFixtureAutoDetect, useValue: true }
  ]
})

it('should display original title', () => {
  // Hooray! No `fixture.detectChanges()` needed
  expect(el.textContent).toContain(comp.title);
});

```

But when we change the component's property, the testing environment `does not know` about that, we need to call `detectChanges`.
> The ComponentFixtureAutoDetect service responds to asynchronous activities such as promise resolution, timers, and DOM events. But a direct, synchronous update of the component property is invisible.

```ts
it('should still see original title after comp.title change', () => {
  const oldTitle = comp.title;
  comp.title = 'Test Title';
  // Displayed title is old because Angular didn't hear the change :(
  expect(el.textContent).toContain(oldTitle);
});
```

Testing with `ComponentFixtureAutoDetect` is `un-certain`, we'd better always call detectChanges() explicitly.

## Test a component with an external template
In real worlds, components always word with external templates and style files. The `createComponent` method is synchronous, but the component must get and compile external templates `in asynchronous way`.
We must give complier time to load files and `split` the single beforeEach into two parts. An asynchronous to load files and a synchronous one to compile.

Creating the test module with `async` function.

```ts
// async beforeEach
beforeEach(async(() => {
  TestBed.configureTestingModule({
    declarations: [ BannerComponent ], // declare the test component
  })
  .compileComponents();  // compile template and css
}));
```

And then create the component synchronously.

```ts
// synchronous beforeEach
beforeEach(() => {
  fixture = TestBed.createComponent(BannerComponent);
  comp = fixture.componentInstance; // BannerComponent test instance
});
```

## Test a component with a dependency
We provide a test double in the test module not a real service. We will get a lot of trouble to use the real one and out purpose is testing the component.

```ts
    TestBed.configureTestingModule({
       declarations: [ WelcomeComponent ],
    // providers:    [ UserService ]  // NO! Don't provide the real service!
                                      // Provide a test-double instead
       providers:    [ {provide: UserService, useValue: userServiceStub } ]
    });
```

There are two ways to get the injected service.

### Getting it by DebugElement
The safest way to get the injected service, the way that always works, is to get it from the injector of the component-under-test. 

```ts
// UserService actually injected into the component
userService = fixture.debugElement.injector.get(UserService);
```

### Gettting it by TestBed.get
But it only works when Angular injects the component with the service instance in the test's root injector.

```ts
// UserService from the root injector
userService = TestBed.get(UserService);
```

Do not reference the userServiceStub object that's provided to the testing module. `It does not work!` The injected service is a cloned one.

## Test a component with an async service

Services always work with Http requests in asynchronous way.

```ts
export class TwainComponent  implements OnInit {
  intervalId: number;
  quote = '...';
  constructor(private twainService: TwainService) { }

  ngOnInit(): void {
    this.twainService.getQuote().then(quote => this.quote = quote);
  }
}
```
We can inject the real service, but replace the method with a spy.

```ts
spy = spyOn(twainService, 'getQuote')
      .and.returnValue(Promise.resolve(testQuote));
```

> Spying on the real service isn't always easy, especially when the real service has injected dependencies.

Then testing the asynchronous methods with `async` or `fakeAsync`

```ts
  it('should show quote after getQuote promise (async)', async(() => {
    fixture.detectChanges();
    fixture.whenStable().then(() => { // wait for async getQuote
      fixture.detectChanges();        // update view with quote
      expect(el.textContent).toBe(testQuote);
    });
  }));
  it('should show quote after getQuote promise (fakeAsync)', fakeAsync(() => {
    fixture.detectChanges();
    tick();                  // wait for async getQuote
    fixture.detectChanges(); // update view with quote
    expect(el.textContent).toBe(testQuote);
  }));
```
Notice: we need to `detectChanges` after `whenStable()` or `tick()`.

> The fakeAsync function enables a `linear coding style` by running the test body in a special fakeAsync test zone.

Calling `tick()` simulates the passage of time until all pending asynchronous activities finish

## Test a component with inputs and outputs
The testing goal is to verify the `property binding` and `event binding`.

We can:
- Test it with a `real` host component.
- Test it as a standalone component.
- Test it with a `substitute` host component.

Real components always has injected services, they will make much trouble.

### As a standalone component

Set input property directly.

```ts
    comp.hero = expectedHero;
    fixture.detectChanges(); // trigger initial data binding
```
And then check the result in DOM.

```ts
expect(heroEl.nativeElement.textContent).toContain(expectedPipedName);
```

Subscribe output property and emit DOM events, check the expected result.

```ts
    let selectedHero: Hero;
    comp.selected.subscribe((hero: Hero) => selectedHero = hero);

    heroEl.triggerEventHandler('click', null);
    expect(selectedHero).toBe(expectedHero);
```

## Test a component a test host component

```ts
@Component({
  template: `
    <dashboard-hero  [hero]="hero"  (selected)="onSelected($event)"></dashboard-hero>`
})
class TestHostComponent {
  hero = new Hero(42, 'Test Name');
  selectedHero: Hero;
  onSelected(hero: Hero) { this.selectedHero = hero; }
}
beforeEach( async(() => {
  TestBed.configureTestingModule({
    declarations: [ DashboardHeroComponent, TestHostComponent ], // declare both
  }).compileComponents();
}));

beforeEach(() => {
  // create TestHostComponent instead of DashboardHeroComponent
  fixture  = TestBed.createComponent(TestHostComponent);
  testHost = fixture.componentInstance;
  heroEl   = fixture.debugElement.query(By.css('.hero')); // find hero
  fixture.detectChanges(); // trigger initial data binding
});
```
- `fixture` holds an instance of `TestHostComponet` with a `DashboardHeroComponent` instance inside.
- The query for hero element still works well.

### A more useful way from `ng-bootstrap`

```ts
export function createGenericTestComponent<T>(html: string, type: {new (...args: any[]): T}): ComponentFixture<T> {
  TestBed.overrideComponent(type, {set: {template: html}});
  const fixture = TestBed.createComponent(type);
  fixture.detectChanges();
  return fixture as ComponentFixture<T>;
}

@Component({
  selector: 'app-test-component',
  template: ''
})
export class TestComponent {
  @ViewChild(MyRatingComponent) instance: MyRatingComponent;

  currentRate: number;
}
describe('MyRatingComponent', () => {
  let component: TestComponent;
  let fixture: ComponentFixture<TestComponent>;
  let instance: MyRatingComponent;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TestComponent, MyRatingComponent]
    });
  }));

  it('should create', () => {
    fixture = createGenericTestComponent(`
      <my-rating></my-rating>
    `, TestComponent);
    component = fixture.componentInstance;
    instance = component.instance;
    expect(instance).toBeTruthy();
  });
);
```
- Specify how to use the tested component by `overrideComponent` inside `createGenericTestComponent` helper function.
- Get the reference of the tested component's instance with `@ViewChild`
- We can fully describe how to use the component in each `it`.

- - - - -

## Test a routed component
A component with injected `Router` and `services` is hard to test.

```ts
constructor(
  private router: Router,
  private heroService: HeroService) {
}

gotoDetail(hero: Hero) {
  let url = `/heroes/${hero.id}`;
  this.router.navigateByUrl(url);
}
```
As often, we are not testing the router, we only care whether a suitable function is `called with expected parameter`.
So we can create stub class with mocked method.

```ts
class RouterStub {
  navigateByUrl(url: string) { return url; }
}
```

Provide router with RouterStub class.

```ts
{ provide: Router,      useClass: RouterStub }
```

And get the injected router with `inject` function.

```ts
it('should tell ROUTER to navigate when hero clicked',
  inject([Router], (router: Router) => { // ...
}));
```

> The inject function uses the current TestBed injector and can only return services provided at that level. It does not return services from component providers.
> Use `fixture.debugElement.injector.get` instead.
> The inject function closes the current TestBed instance to further configuration. You cannot call any more TestBed configuration methods.

## Test a routed component with parameter

```ts
constructor(
  private heroDetailService: HeroDetailService,
  private route:  ActivatedRoute,
  private router: Router) {
}
ngOnInit(): void {
  // get hero when `id` param changes
  this.route.params.subscribe(p => this.getHero(p && p['id']));
}
```
- The `params` property is an `Observable`
- Explore how to response different `id` values by injected `ActivatedRoute`

```ts
import { BehaviorSubject } from 'rxjs/BehaviorSubject';

@Injectable()
export class ActivatedRouteStub {

  // ActivatedRoute.params is Observable
  private subject = new BehaviorSubject(this.testParams);
  params = this.subject.asObservable();

  // Test parameters
  private _testParams: {};
  get testParams() { return this._testParams; }
  set testParams(params: {}) {
    this._testParams = params;
    this.subject.next(params);
  }

  // ActivatedRoute.snapshot.params
  get snapshot() {
    return { params: this.testParams };
  }
}

```
- - - - -
## Use a page object to simplify setup
When there is a complex template:

```html
<div *ngIf="hero">
  <h2><span>{{hero.name | titlecase}}</span> Details</h2>
  <div>
    <label>id: </label>{{hero.id}}</div>
  <div>
    <label for="name">name: </label>
    <input id="name" [(ngModel)]="hero.name" placeholder="name" />
  </div>
  <button (click)="save()">Save</button>
  <button (click)="cancel()">Cancel</button>
</div>
```
Creating a Page object to simplify access to component properties.

```ts
class Page {
  gotoSpy:      jasmine.Spy;
  navSpy:       jasmine.Spy;

  saveBtn:      DebugElement;
  cancelBtn:    DebugElement;
  nameDisplay:  HTMLElement;
  nameInput:    HTMLInputElement;

  constructor() {
    const router = TestBed.get(Router); // get router from root injector
    this.gotoSpy = spyOn(comp, 'gotoList').and.callThrough();
    this.navSpy  = spyOn(router, 'navigate');
  }

  /** Add page elements after hero arrives */
  addPageElements() {
    if (comp.hero) {
      // have a hero so these elements are now in the DOM
      const buttons    = fixture.debugElement.queryAll(By.css('button'));
      this.saveBtn     = buttons[0];
      this.cancelBtn   = buttons[1];
      this.nameDisplay = fixture.debugElement.query(By.css('span')).nativeElement;
      this.nameInput   = fixture.debugElement.query(By.css('input')).nativeElement;
    }
  }
}
```

- - - - -
## Setup with module imports

Merge multiple dependencies into a module and import it at once.

```ts
beforeEach( async(() => {
   TestBed.configureTestingModule({
    imports:   [ HeroModule ],
    providers: [
      { provide: ActivatedRoute, useValue: activatedRoute },
      { provide: HeroService,    useClass: FakeHeroService },
      { provide: Router,         useClass: RouterStub},
    ]
  })
  .compileComponents();
}));
```

> Importing the component's feature module is often the easiest way to configure the tests, especially when the feature module is small and mostly self-contained, as feature modules should be.

- - - - -

## Override a component's providers
When a component has its own providers, we can't stub it with `TestBed.configureTestingModule`.
Angular create component with a `fixture level` injector, it always use the real service.

We can override the component's providers with `TestBed.overrideComponent` function.

```ts
  beforeEach( async(() => {
    TestBed.configureTestingModule({
      imports:   [ HeroModule ],
      providers: [
        { provide: ActivatedRoute, useValue: activatedRoute },
        { provide: Router,         useClass: RouterStub},
      ]
    })

    // Override component's own provider
    .overrideComponent(HeroDetailComponent, {
      set: {
        providers: [
          { provide: HeroDetailService, useClass: HeroDetailServiceSpy }
        ]
      }
    })

    .compileComponents();
  }));
```

It takes two arguments: the component type to override (HeroDetailComponent) and an override metadata object

```ts
type MetadataOverride = {
  add?: T;
  remove?: T;
  set?: T;
};
```
> The TestBed.overrideComponent method can be called multiple times for the same or different components. The TestBed offers similar `overrideDirective`, `overrideModule`, and `overridePipe` methods for digging into and replacing parts of these other classes.
- - - - -
## Test a RouterOutlet component

```html
<app-banner></app-banner>
<app-welcome></app-welcome>

<nav>
  <a routerLink="/dashboard">Dashboard</a>
  <a routerLink="/heroes">Heroes</a>
  <a routerLink="/about">About</a>
</nav>

<router-outlet></router-outlet>
```

Test it with a stub `RouterLink` directive.

```ts
@Directive({
  selector: '[routerLink]',
  host: {
    '(click)': 'onClick()'
  }
})
export class RouterLinkStubDirective {
  @Input('routerLink') linkParams: any;
  navigatedTo: any = null;

  onClick() {
    this.navigatedTo = this.linkParams;
  }
}
```
Then get the directives: 

```ts
beforeEach(() => {
  // trigger initial data binding
  fixture.detectChanges();

  // find DebugElements with an attached RouterLinkStubDirective
  linkDes = fixture.debugElement
    .queryAll(By.directive(RouterLinkStubDirective));

  // get the attached link directive instances using the DebugElement injectors
  links = linkDes
    .map(de => de.injector.get(RouterLinkStubDirective) as RouterLinkStubDirective);
});
```

> Stubbed RouterLink tests can confirm that a component with links and an outlet is setup properly, that the component has the links it should have, and that they are all pointing in the expected direction. 

- - - - -

## "Shallow component tests" with NO_ERRORS_SCHEMA

When create stub components with `no reason more than to avoid a compile error`, Add `NO_ERRORS_SCHEMA` to the testing module's schemas metadata to tell the compiler to ignore unrecognized elements and attributes.

```ts
  TestBed.configureTestingModule({
    declarations: [ AppComponent, RouterLinkStubDirective ],
    schemas:      [ NO_ERRORS_SCHEMA ]
  })
```