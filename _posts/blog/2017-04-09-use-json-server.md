---
layout: post
title:  "Use json-server as a REST service"
date:   2017-04-09 15:54:06
categories: json-server
---

# Install [json-server](https://github.com/typicode/json-server)
## Install
```shell
$ npm install json-server -g
```
## Create a mock-json.json file under project directory
```json
{
  "todos": [
    {
      "id": 1,
      "title": "todo 1",
      "createTime": 1234567890
    },
    {
      "id": 2,
      "title": "todo 2",
      "createTime": 1234567891
    }
  ]
}
```

## Start the json-server as port 3000
```shell
$ json-server --watch mock-data.json
```
## Access json-server from the browser

![](/images/2017-04-09-15-56-06.jpg)

# Get todo list from json-server with the injected http service
```ts
import { Http } from '@angular/http';
import 'rxjs/add/operator/toPromise';

  private todosUrl = 'api/todos';
  
  getTodoItemList(): Promise<TodoItem[]> {
    return this.http.get(this.todosUrl)
      .toPromise()
      .then((response) => response.json() as TodoItem[])
      .catch((error: any) => Promise.reject(error.message || error));
  }
```
- But we got an error when the application runs.

![](/images/2017-04-09-15-58-24.jpg)

- Our application runs on port 4200, but the json-server runs on port 3000
- And also, the url is not match, the app use `api/todos`, but the json-server applies `todos`

# Config the proxy supported by webpack-dev-server
## Create a `proxy.conf.json` file next to the package.json
```json
{
  "/api/todos": {
    "target": "http://localhost:3000/todos",
    "secure": false
  }
}
```
## Run `ng serve` with proxy argumament `--proxy-config proxy.conf.json`
```shell
$ ng serve --proxy-config proxy.config.json
```

![](/images/2017-04-09-16-08-42.jpg)

# Test
```ts
import { TestBed, inject } from '@angular/core/testing';

import { TodoService } from './todo.service';
import { HttpModule, XHRBackend, Response, ResponseOptions, Http, BaseRequestOptions } from '@angular/http';
import { MockBackend } from '@angular/http/testing';
import { async } from '@angular/core/testing';
import { TodoItem } from 'app/todo/model/todo-item';


describe('TodoService', () => {

  const todoItemList = [
    {
      'id': 1,
      'title': 'todo 1',
      'createTime': 1234567890,
      'status': 0
    },
    {
      'id': 2,
      'title': 'todo 2',
      'createTime': 1234567891,
      'status': 0
    }
  ];

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [],
      providers: [
        TodoService,
        MockBackend,
        BaseRequestOptions,
        {
          provide: Http,
          useFactory: (backend, options) => new Http(backend, options),
          deps: [MockBackend, BaseRequestOptions]
        }
      ]
    });
  });

  it('should return a promise with TodoItem list',
    async(inject([TodoService, MockBackend, Http], (service: TodoService, mockBackend: MockBackend, http: Http) => {

      mockBackend.connections.subscribe(conn => {
        conn.mockRespond(new Response(new ResponseOptions({ body: JSON.stringify(todoItemList) })));
      });

      service.getTodoItemList().then(list => {
        expect(list.length).toBe(3);
      });
    })));
});
```

- Test http request with `[MockBackend](https://angular.io/docs/ts/latest/api/http/testing/index/MockBackend-class.html)`
- [Angular 2 MockBackend Service Testing Template Using TestBed](https://kendaleiv.com/angular-2-mockbackend-service-testing-template-using-testbed/)
- `Notice` : Response need to be added in import statement 