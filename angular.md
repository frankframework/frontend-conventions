# Angular related conventions

<!-- TOC -->
* [Angular related conventions](#angular-related-conventions)
  * [Library Project](#library-project)
  * [App Project](#app-project)
    * [Folder structure](#folder-structure)
  * [Views (html files)](#views-html-files)
    * [String interpolation and binding](#string-interpolation-and-binding)
    * [Built-in control flow [preview]](#built-in-control-flow-preview)
      * [If statements](#if-statements)
      * [For loops](#for-loops)
<!-- TOC -->

## Library Project
[Angular Library Example](https://github.com/Matthbo/angular-lib-poc) is an example of how the project structure of an Angular Library project should be.
At a later date we'll add a template project that can be used as a starter for new Angular Library projects

## App Project

### Folder structure
A nested file structure is preferred over a flat file structure. Each component can be found in the folder of it's parent (the page). "Shared" components can be places in the `components`folder in the app directory. 
"Shared" services should go in the `services` folder in the app directory.
Component specific services should go into the app directory folder of the component that uses it.

<!-- To generate a new tree use: https://tree.nathanfriend.io/ -->

```
.
├── src
│   └── app
│       ├── components
│       │   └── button
│       ├── pages
│       │   └── dashboard
│       │       └── gauge
│       │           ├── component.ts
│       │           └── service.ts
│       ├── services
│       ├── app.component.ts
│       └── routing.module.ts
├── .editorconfig
└── eslint.rc
```


## Views (HTML files)

### String interpolation and binding
String interpolation can be used to enter a value inside some other text. This is mostly done in the content of an element.
For an attribute of an element this shouldn't be done, because it is already possible to use a one-way bind for this. Using the bind is a good practice because it will ensure consistency between the binding of a string and the binding of other kinds of objects.

```angular2html
// Bad
<option value="{{view.key}}px">
  {{ view.value }}
</option>

// Bad
<option [value]="'{{view.key}}px'">
  {{ view.value }}
</option>

// Good
<option [value]="view.key + 'px'">
  {{ view.value }}
</option>

// Good
<option value="15px">
  {{ view.value }}
</option>
```

### Built-in control flow [preview]
> [!IMPORTANT]
> This will only work in angular 17 and higher.

#### If statements
Why use the built-in control flow instead of the ngIf variant?
The new syntax for the built-in control flow, has a more readable if-else statement, for this reason, we opted to use it everywhere.
In addition to that, using the built-in flow control for loops and if statements are easier to recognize and don't have a long form or short form (*ngIf & let-var [ngIf] ...)

```angular2html
// Bad
<div *ngIf="amount > 0; else empty">
    Currently in progress: {{amount}}
</div>
<ng-template #empty>
    None in progress
</ng-template>

// Good
<div>
    @if (amount > 0) {
        Currently in progress: {{amount}}
    } @else {
        None in progress
    }
</div>
    
// Bad
<div *ngIf="user.isAdmin()">
    <button (click)="onEdit()">Edit</button>
</div>

// Good
@if (user.isAdmin) {
    <button (click)="onEdit()">Edit</button>
}
```

#### For loops
When using the built-in flow control for loops, it is important to set the track, the track is there for performance. It should always be set to track the unique elements in the list (for example element.id).  Link to docs: https://angular.io/api/core/for

```typescript
// Bad
<div @ngFor="let element of list">
    ...
</div>

// Good
@for (let element of list; track element) {
    ...
}
```

When looping over an array of strings.

```typescript
// Bad
@for (let element of list; track list) {
    ...
}

// Good
@for (let element of list; track element) {
    ...
}
```

When looping over an array of objects for example: `{ id: string, value: number }`.

```typescript
// Bad
@for (let element of list; track element) {
    ...
}

// Good
@for (let element of list; track element.id) {
    ...
}
```
