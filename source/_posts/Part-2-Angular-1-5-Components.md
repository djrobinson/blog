---
title: 'Part 2: Angular 1.5 Components'
date: 2016-05-19 11:30:03
tags:
---

Now that we understand the discrepancies in Angular and the Web Components standards, let’s dive into the changes in 1.5 that begin to pave the way to Angular 2. I’ve built the standard Todo List example in bother versions to compare and contrast. To play around with the examples below, checkout out this <a href="https://plnkr.co/edit/GogmZNIC4QvsRWtrhlqq?p=info">Plunker</a> for Angular 1.5 Component structure, and this <a href="https://plnkr.co/edit/9sLQI44HlXDa1j6EztVo?p=info">Plunker</a> for Angular Directive structure.

Here’s a component for creating an item in our todo list written out in both styles:

#### Angular 1.5 Component:

```javascript
/* ANGULAR 1.5 COMPONENT */
(function(){
  'use strict';

  angular
    .module('exampleApp')
    .component('createTodo', {
      template:
      `
        <h1>Create Todo</h1>
        <form>
          <input ng-model="newTodo" type="text" />
          <button ng-click="$ctrl.createTodo(newTodo)">Create</button>
        </form>
      `,
      bindings: {
        onCreate: '&'
      },
      controller: createController
    });

  function createController() {
    var ctrl = this;

    ctrl.createTodo = function(todoName){
      var todo = {
        name: todoName,
        done: false
      };
      ctrl.onCreate({todo: todo});
    };
  }
})();
```

#### Angular 1.x Directive:

```javascript
(function(){
  'use strict';
  angular
    .module('exampleApp')
    .directive('createTodo', function(){
      return {
        template: `
          <h1>Create Todo</h1>
          <form>
            <input ng-model="ctrl.newTodo" type="text" />
            <button ng-click="ctrl.createChildTodo(ctrl.newTodo)">Create</button>
          </form>
        `,
        scope: {
          todos: '='
        },
        controllerAs: 'ctrl',
        controller: createController,
        bindToController: true
      };
    });
  function createController() {
    var ctrl = this;
    ctrl.createChildTodo = function(todoName){
      var todo = {
        name: todoName,
        done: false
      };
      ctrl.todos.push(todo);
      ctrl.newTodo = "";
    };
  }
})();
```

Not terribly different at first glance, but let’s take a look at major deletions and additions to see how things are more component-ized. If you take a look at the component docs, you'll find a quick glance table of a comparison of the component method versus the directive method. Here's some of the highlights.

## Deletions

### `scope`
The most important thing to recognize is that components are defaulted to isolate scope with the component method.  This means that the scope option will be unnecessary, and any inherited variable definitions will be handled in the bindings option.

### `controllerAs` & `bindToController`:
Both of these options have been eliminated due to the component method’s automatic creation of isolate scope.  Now that each component should be limited to controlling their own view and data, `bindToController` makes sense to be defaulted to `true`.There also shouldn’t be concerns of intermingling scope variables with default isolate scope, so `controllerAs` is defaulted to `$ctrl`.

### `link`
As I mentioned in my previous post, the `link` function’s ability to update & listen to the DOM steps beyond the limited DOM manipulation that components aim to provide.

## Additions:

### `bindings {}`
This is the replacement for the directive scope option. Will be used the same way, but with additional options for one-way binding.

### One-way data bindings with `<` and `&`
While the component defaults to isolate scope, this only goes so far as controlling then internal variables of a component. With any directive that uses two-way data binding on a scope variable with the `"="` option, inherited variables can be altered in the child scope and it will be updated in the parent.  The `"<"` option signifies one-way data binding, which means the property in the child scope is not watched.  When used in conjunction with the ‘&’ option on bindings, you can run functions defined by the parent.

You can see an example of this in the createTodo function.  This inherits from the parent via the on-create property passed in on the create-todo element. Note that bindings for parent expressions must be passed into the element as snake-cased attributes, and must be camel cased in the component binding.  Also, the child component must pass in an object mapping the child scope variables to the parameter accepted by the parent expression.

Relevant code for updating parent scope with bound expression:


```javascript
// On button click, call the createTodo function defined in the controller
// passing in the newTodo defined in the input
<input ng-model="newTodo" type="text" />
<button ng-click="$ctrl.createTodo(newTodo)">Create</button>

```
------

```javascript
//This expression binding references the on-create attr defined on the parent component
bindings: {
        onCreate: '&'
      },
```
------

```javascript
//Here we map todo from the parent component function to the todo passed in by
ctrl.createTodo = function(todoName){
      var todo = {
        name: todoName,
        done: false
      };
      ctrl.onCreate({todo: todo});
      ctrl.newTodo = "";
    };
```

One exception to one way data binding is when you bind to a reference type from the parent.  Any change to an Object or Array in the child component will update the parent. Here’s an example of how this would work on the todo.done attribute:


```javascript
  <button ng-click="$ctrl.complete()">Complete</button>

```
------
```javascript
    bindings: {
      todo: "<",
    },
```
------
```javascript
    ctrl.complete = function(){
      ctrl.todo.done = !ctrl.todo.done;
    };
```

While this does work, it should be avoided. Parent scope changes should be made explicit by passing expressions as attributes to the child component like the first example.

