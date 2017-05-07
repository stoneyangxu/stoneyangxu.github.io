---
layout: post
title:  "rebirth-ng源码学习"
date:   2017-05-06 23:51:44
categories: rebirth-ng
---
# rebirth-ng源码学习
## AppModule
在AppModule中，导入各个模块demo的module，实际的组件是按照实际使用场景的导入方式引入：

![](/images/2017-05-07-00-46-43.jpg)

## AppComponent

### 作用
- 设置主题

```ts
this.themeService.setupTheme('', this.renderer, this.elementRef);
```

- 读取应用级的README.md
```ts
this.gettingStarted = this.domSanitizer.bypassSecurityTrustHtml(this.demoConfigService.gettingStarted.readMe);
```
- 解析各个模块的README.md，对菜单按字母顺序进行排序

```ts
    this.components = this.demoConfigService.components
      .map(cmp => {
        cmp.readMe = this.domSanitizer.bypassSecurityTrustHtml(cmp.readMe);
        cmp.ts = this.fixTSModuleImport(cmp.ts);
        return cmp;
      })
      .sort((a, b) => a.name.localeCompare(b.name));
```
- 解析模版对过程中，使用`fixTSModuleImport`方法将demo中对相对路径引用修改为模块引用，使得页面展示时更贴近实际使用场景

```ts
  private fixTSModuleImport(code): string {
    console.log(code);
    return code.replace(/\.\.\/\.\.\/exports(\/.*)?/, 'rebirth-ng');
  }
```

![](/images/2017-05-07-00-28-29.jpg)

- 调用`this.setupMenus();`方法构造菜单数据

### setupMenu()

- 将组件列表转化为菜单数据，组件名称作为标题，并且生成跳转锚点

```ts
    const cmpMenus = this.components.map(item => {
      return {
        text: item.name,
        url: `#${item.name}`
      };
    });
```

- 根据MenuBar所需要对数据结构，生成菜单数据，组件列表作为`Components`的子菜单，`changeThemeHandler`作为切换主题的处理函数

### 模板中使用MenuBar组件加载菜单，主界面循环`components`，使用`DocComponent`加载各个组件的样例

```html
  <ng-container *ngFor="let item of components">
    <st-doc [component]="item"></st-doc>
    <hr/>
  </ng-container>
```
- ng-container - 作为虚拟容器，在实际的dom中不会显示

## DocComponent
DocComponent的实现类相对简单，实际上只监听html内容的变更，对代码进行高亮处理。

```ts
export class DocComponent implements AfterViewInit {

  @ViewChildren('html') html: QueryList<ElementRef>;
  @ViewChildren('typescript') typescript: QueryList<ElementRef>;

  @Input() component: any;

  constructor() {
  }

  activeTabChange(id) {
    setTimeout(() => {
      hljs.highlightBlock(this.html.last.nativeElement);
    }, 0);
  }

  ngAfterViewInit(): void {

    this.html.changes.subscribe((html) => {
      hljs.highlightBlock(html.last.nativeElement);
    });

    this.typescript.changes.subscribe((typescript) => {
      hljs.highlightBlock(typescript.last.nativeElement);
    });
  }
}
```
- activeTabChange - 用来绑定tab页签对切换事件，借助了自定义对tab组件
- ngAfterViewInit - html文本内容是通过tab组件显示的，所以需要在子组件加载完成后，再监听内容变化，重新进行高亮。
### 模板分为四个部分：组件标题、demo显示效果、demo源码、组件的说明文件

#### 组件标题：包含组件名称、api文档路径、github源码路径

```html
  <h2><a href="#{{component.name}}">{{component.name}}</a>
    <small class="meta-data">
      (<a class="" target="_blank"
          [href]="'/rebirth-ng/compodocs/modules/' + component.name +'Module.html'">API Docs
    </a>, <a class="" target="_blank"
             [href]="'https://github.com/greengerong/rebirth-ng/tree/master/src/app/exports/' + component.directory">
      Github Source</a>)
    </small>
  </h2>
```

#### demo显示效果：使用DocContentComponent加载demo

```html
  <st-doc-content [component]="component.cmp"></st-doc-content>
```
- component.cmp - 组件的demo实现类引用

#### demo源码：借助tab组件显示html和ts文件内容

```html
  <st-tabs class="code-tabs" (activeTabChange)="activeTabChange($event)">
    <st-tab id="html" title="HTML">
      <ng-template stTabContent>
        <div class="html" #html>
          <pre>{{component.html}}</pre>
        </div>
      </ng-template>
    </st-tab>
    <st-tab id="ts" title="TypeScript">
      <ng-template stTabContent>
        <div class="typescript" #typescript>
          <pre>{{component.ts}}</pre>
        </div>
      </ng-template>
    </st-tab>
  </st-tabs>
```
#### 描述文件内容

```html
  <div class="markdown" [innerHTML]="component.readMe"></div>
```
#### 组件配置实例：

```ts
    {
      name: 'Alert',
      directory: 'alert',
      cmp: AlertDemoComponent,
      readMe: require('!html-loader!markdown-loader!../../exports/alert/README.md'),
      html: require('!raw-loader!../../demo/alert/alert-demo.component.html'),
      ts: require('!raw-loader!../../demo/alert/alert-demo.component.ts'),
    },
```

## DocContentComponent

DocContentComponent的模板为空，作用是根据传入的demo组件类，动态加载组件示例。

```ts
  onComponentChange() {
    const factory = this.componentFactoryResolver.resolveComponentFactory(this.cmp);
    this.cmpRef = this.viewContainerRef.createComponent(factory, this.viewContainerRef.length, this.injector);
  }
```
- this.viewContainerRef.length - 保证组件加载位置紧邻host之下
- cmp: Type<any>; - 获取任意类的引用
- this.cmpRef.destroy(); - 动态加载的组件需要手动销毁

## ThemeService
用来切换主题样式

```ts
@Injectable()
export class ThemeService {
  static THEME_KEY = 'stone-ui:theme';

  setupTheme(theme: string, renderer: Renderer2, parentElementRef: ElementRef) {
    theme = theme || localStorage.getItem(ThemeService.THEME_KEY);
    if (theme) {
      const link = renderer.createElement('link');
      link.rel = 'stylesheet';
      link.href = `./assets/themes/bootstrap.${theme.toLowerCase()}.css`;
      renderer.appendChild(parentElementRef.nativeElement, link);
      localStorage.setItem(ThemeService.THEME_KEY, theme);
    }
  }
}
```

- localStorage - 使用本地存储保存当前主题
- renderer.createElement('link'); - 使用Renderer2创建新的样式引用节点
- link.href - 使用样式名称拼接自定义样式文件路径
- appendChild - 加载自定义样式文件，覆盖默认样式

## SharedModule

### forRoot方法
应该在AppModule中通过forRoot方法引入，保证单例服务的引入

```ts
  static forRoot(): ModuleWithProviders {
    return {
      ngModule: SharedModule,
      providers: [
        { provide: DemoConfigService, useClass: DemoConfigService },
        { provide: ThemeService, useClass: ThemeService }
      ]
    };
  }
```
子模块的引用只需要引入SharedModule

## StoneUIModule
组件对外发布的module，客户端使用的时候通过forRoot方法或直接引入module来使用组件。

![](/images/2017-05-07-01-42-43.jpg)


# 疑问

## DomSanitizer的作用？屏蔽Angular自身对安全转译？

```ts
cmp.readMe = this.domSanitizer.bypassSecurityTrustHtml(cmp.readMe);
```
## highlightBlock为何没有生效？为什么要用last？

```ts
  hljs.highlightBlock(typescript.last.nativeElement);
```
## demo-config.service.ts中的require是在编译时解析？

```ts
      readMe: require('!html-loader!markdown-loader!../../exports/alert/README.md'),
      html: require('!raw-loader!../../demo/alert/alert-demo.component.html'),
      ts: require('!raw-loader!../../demo/alert/alert-demo.component.ts'),
```
## DocContentComponent中组件动态加载，为什么要用get/set方法设置，而不是简单的属性绑定，通过init加载？

```ts
  @Input() set component(cmp: Type<any>) {
    this.cmp = cmp;
    this.onComponentChange();
  }

  get component() {
    return this.cmp;
  }
```
## SharedModule中为何要将Angular提供的module导出？

![](/images/2017-05-07-01-34-11.jpg)

## asset/fonts中各种格式文件的作用是什么？

![](/images/2017-05-07-01-35-00.jpg)

## tsconfig-es2016.json相对构建配置的区别是什么？

![](/images/2017-05-07-01-41-29.jpg)

## OnPush与Immutable.js的作用是什么？
