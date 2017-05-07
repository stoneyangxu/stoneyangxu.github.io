---
layout: post
title:  "Developing a Tab component"
date:   2017-04-12 00:30:56
categories: Angular
---
# Developing a Tab component
- The consumer should use it like this:

```html
<tabs>
  <tab tabTitle="Tab 1">
    Here's some content.
  </tab>
  <tab tabTitle="Tab 2">
    And here's more in another tab.
  </tab>
</tabs>
```

# Create `tabs` component 

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'my-tabs',
  templateUrl: './tabs.component.html',
  styleUrls: ['./tabs.component.scss']
})
export class TabsComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }
}
```

```html
<ul>
  <li>Tab 1</li>
  <li>Tab 2</li>
</ul>
```
- Use the component with `my-tabs` tag

```html
<my-tabs></my-tabs>
```
# We need a place to show the tab content with `ng-content`
- Add ng-content in the template

```html
<ul>
  <li>Tab 1</li>
  <li>Tab 2</li>
</ul>
<ng-content></ng-content>
```
- When using the component, put tab contents in `<tabs>` tag

```html
<my-tabs>
  <p>Tab content</p>
</my-tabs>
```
# Create a tab element to replace `<li>Tab 1</li>` with the property `tabTitle`

```ts
import { Component, OnInit, Input } from '@angular/core';

@Component({
  selector: 'my-tab',
  templateUrl: './tab.component.html',
  styleUrls: ['./tab.component.scss']
})
export class TabComponent implements OnInit {

  @Input() tabTitle: string;

  constructor() { }

  ngOnInit() {
  }

}
```
```html
<div>
  <ng-content></ng-content>
</div>
```
- Use the component like this:

```html
<my-tabs>
  <my-tab tabTitle="tab1">
    Tab 1 content
  </my-tab>
  <my-tab tabTitle="tab2">
    Tab 2 content
  </my-tab>
</my-tabs>
```
# Generate tabs dynamic
- Using `*ngFor` directive to generate li element

```html
<ul>
  <li *ngFor="let tab of tabs">{{ tab.tabTitle }}</li>
</ul>
```
- Create `tabs` property in TabsComponent, get children component with `ContentChildren`
```ts
  @ContentChildren(TabComponent) tabs: QueryList<TabComponent>;
```
# And `how to test` component with ViewChildren

- Create test component in the spec file as the host

```ts
@Component({
  selector: 'test-my-tabs',
  template: ''
})
export class TestTabsComponent {
  constructor() { }
}
```
- Create html template in spec file

```ts
  const html = `
    <my-tabs>
      <my-tab tabTitle="tab1">
        Tab 1 content
      </my-tab>
      <my-tab tabTitle="tab2">
        Tab 2 content
      </my-tab>
    </my-tabs>
  `;
```

- Init Angular test component with the `TestTabsComponent`

```ts
  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TestTabsComponent, TabsComponent, TabComponent]
    });
  }));

  beforeEach(() => {

    TestBed.overrideComponent(TestTabsComponent, { set: { template: html } });

    fixture = TestBed.createComponent(TestTabsComponent);

    component = fixture.componentInstance;
    fixture.detectChanges();
  });
```

- Override the component template with `TestBed.overrideComponent` before creating the test component and `do not compile the component before override`

```ts
  beforeEach(() => {

    TestBed.overrideComponent(TabsComponent, { set: { template: html } });

    fixture = TestBed.createComponent(TabsComponent);

    component = fixture.componentInstance;
    fixture.detectChanges();
  });
```
- Then the children components will be created

```ts
  it('should display childrens components with the property tabs', () => {
    const tabs = fixture.debugElement.queryAll(By.css('li'));
    expect(tabs.length).toBe(2);
  });
```
- And we can get `TabsComponent instance` with `@ViewChild`

```ts
@Component({
  selector: 'test-my-tabs',
  template: ''
})
export class TestTabsComponent {
  @ViewChild(TabsComponent) tabs: TabsComponent;
  constructor() { }
}

  it('should active the first tab as default', () => {
    const tabsComponent = component.tabs;
    expect(tabsComponent.tabs.first.active).toBeTruthy();
    expect(tabsComponent.tabs.last.active).toBeFalsy();
  });
```
# Switch the tab content when clicking the `tab title`
## Test first 

```ts
  it('should siwtch tab when clicking the title', () => {
    const tabTitles = fixture.debugElement.queryAll(By.css('li'));
    const secondTabTitle = tabTitles[tabTitles.length - 1].nativeElement;

    secondTabTitle.click();

    expect(component.tabs.tabs.first.active).toBeFalsy('the first tab is unactive');
    expect(component.tabs.tabs.last.active).toBeTruthy('the tab clicked is active');
  });
```
## Then bind click event to selectTab method

```html
  <li *ngFor="let tab of tabs" (click)="selectTab(tab)">
    {{ tab.tabTitle }}
  </li>
```
```ts
  selectTab(tab: TabComponent) {
    this.tabs.forEach(x => x.active = false);
    tab.active = true;
  }
```