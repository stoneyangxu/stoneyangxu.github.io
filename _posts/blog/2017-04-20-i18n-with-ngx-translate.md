---
layout: post
title:  "i18n with ngx-translate"
date:   2017-04-20 00:32:12
categories: Angular
---

# i18n with ngx-translate

`ngx-translate` is i18n library for Angular2+
> https://github.com/ngx-translate/core#additional-framework-support

## Install npm package as runtime dependency

```shell
$ npm install @ngx-translate/core --save
```

## Import TranslateModule

### In root module

```ts
TranslateModule.forRoot()
```
- The forRoot static method is a convention that provides and configures `services` at the same time

```ts
    /**
     * Use this method in your root module to provide the TranslateService
     * @param {TranslateModuleConfig} config
     * @returns {ModuleWithProviders}
     */
    static forRoot(config: TranslateModuleConfig = {}): ModuleWithProviders {
        return {
            ngModule: TranslateModule,
            providers: [
                config.loader || {provide: TranslateLoader, useClass: TranslateFakeLoader},
                config.parser || {provide: TranslateParser, useClass: TranslateDefaultParser},
                config.missingTranslationHandler || {provide: MissingTranslationHandler, useClass: FakeMissingTranslationHandler},
                TranslateStore,
                {provide: USE_STORE, useValue: config.isolate},
                TranslateService
            ]
        };
    }
```

### In shared module

```ts
@NgModule({
    exports: [
        CommonModule,
        TranslateModule
    ]
})
export class SharedModule { }
```

- import `TranslateModule` to avoid importing it everywhere.

### In Lazy loaded modules

Use for child method.

```ts
TranslateModule.forChild()
```

- lazy loaded modules use a different injector from the rest of your application
- you can configure them separately with a different loader/parser/missing translations handler
- You can also isolate the service by using isolate: true. In which case the service is a completely isolated instance

```ts
    /**
     * Use this method in your other (non root) modules to import the directive/pipe
     * @param {TranslateModuleConfig} config
     * @returns {ModuleWithProviders}
     */
    static forChild(config: TranslateModuleConfig = {}): ModuleWithProviders {
        return {
            ngModule: TranslateModule,
            providers: [
                config.loader || {provide: TranslateLoader, useClass: TranslateFakeLoader},
                config.parser || {provide: TranslateParser, useClass: TranslateDefaultParser},
                config.missingTranslationHandler || {provide: MissingTranslationHandler, useClass: FakeMissingTranslationHandler},
                {provide: USE_STORE, useValue: config.isolate},
                TranslateService
            ]
        };
    }

```

## Configuration

By default, there is no loader available, we should install `@ngx-translate/http-loader` to load resource by http.

> https://github.com/ngx-translate/http-loader

```shell
$ npm install @ngx-translate/http-loader --save
```

Then add configuration to forRoot and forChild method.

```ts
@NgModule({
    imports: [
        BrowserModule,
        HttpModule,
        TranslateModule.forRoot({
            loader: {
                provide: TranslateLoader,
                useFactory: HttpLoaderFactory,
                deps: [Http]
            }
        })
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

And we need to create a `HttpLoaderFactory` method to build the loader.

```ts
export function HttpLoaderFactory(http: Http) {
    return new TranslateHttpLoader(http, "/public/lang-files/", "-lang.json");
}
```

The TranslateHttpLoader also has two optional parameters:

- prefix: string = "/assets/i18n/"
- suffix: string = ".json"


