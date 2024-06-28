# TypeScript related conventions

<!-- TOC -->
* [TypeScript related conventions](#typescript-related-conventions)
  * [Types](#types)
    * [Utility types](#utility-types)
      * [Record](#record)
    * [Casting number to string](#casting-number-to-string)
    * [Enums](#enums)
    * [String manipulation](#string-manipulation)
  * [Naming & order](#naming--order)
    * [Subject as Observable notation](#subject-as-observable-notation)
    * [Class member ordering](#class-member-ordering)
    * [Read-only constant values](#read-only-constant-values)
  * [Methods](#methods)
    * [Maximum method parameters and options object](#maximum-method-parameters-and-options-object)
<!-- TOC -->

## Types

### Utility types
Make use of these utility types:
* Record

#### Record
**Why a Record and not an Object?**
Readability. Creating a key-value object yourself can be unclear for a beginner.
It also roughly corresponds to the existing Record in Java.

**Why not a map?**
Map has additional functions that you might not need. When you make an API request, you often already receive an object back that you can then create as a Record."

```typescript
// Bad
getViews(): Observable<{ [key: string]: View }> {
    return this.http.get<{ [key: string]: View }>('api/testtool/views');
}

// Good
getViews(): Observable<Record<string, View>> {
    return this.http.get<Record<string, View>>('api/testtool/views');
}
```

### Casting number to string
Even though the class String must not be used as a type, it's constructor should be used when casting a number to string.
Why not use the class string as type?
Because it's bad practice to use a class as type if there's a primitive type available.

```typescript
// Bad
function numberToString(value: number): string {
    return `${value}`;
}

// Bad
function numberToString(value: number): string {
    return '' + value;
}

// Bad
function numberToString(value: number): string {
    return value.toString();
}

// Good
function numberToString(value: number): string {
    return String(value);
}
```

### Enums

Enums in TypeScript can be a useful tool for defining a set of constant values. However, there are several reasons why their use can be considered problematic:

1. **Unexpected Behavior**: TypeScript enums are essentially objects with keys and values. When enum variant values are not explicitly assigned, they are automatically assigned numeric values starting from 0. This means that the first enum value variant is falsy, while every other value is truthy. This can lead to unexpected behavior in your code.

2. **Loose Type Safety**: Enums in TypeScript are not as type-safe as you might expect. For example, if you pass a number to a function that expects an enum, TypeScript will not raise a compile-time error. This can lead to bugs that are hard to track down.

3. **Code Size and Performance**: Excessive use of regular enums can lead to code size issues, security issues, scalability issues, and maintainability issues.

4. **Confusing Output**: The compiled JavaScript for an enum can be confusing because it includes both the named properties you defined and number keys with a string value representing the named constant.

As a result, some developers recommend using alternatives to enums, such as objects or types. These alternatives can provide similar functionality to enums but without some of the drawbacks. However, like any tool, enums have their place and can be useful in certain situations. It's important to understand their behavior and trade-offs to make an informed decision about when to use them.

```typescript
// Bad
export enum appInitState {
    UN_INIT = -1,
    PRE_INIT = 0,
    INIT = 1,
    POST_INIT = 2,
};

// Good
export const appInitState = {
    UN_INIT: -1,
    PRE_INIT: 0,
    INIT: 1,
    POST_INIT: 2,
} as const;

export const state = ["state1", "state2"] as const;
export type State = typeof state[number];
```

### String manipulation
In most cases we find that string concatenation is not readable, for example: 'string' + variable + 'string'. It's better to use a string template literal in these cases, like: =`string${variable}string`.

```typescript
// Bad
function manipulatePath(path: string): string {
    if (...) {
        path = `hostname/${path}/blah`;
    }
    if (...) {
        path += '/domain';
    }
    return path;
}

// Bad (prohibited by eslint)
function manipulatePath(path: string): string {
    if (...) {
      path = 'hostname/' + path;
    }
    if (...) {
      path = path + '/domain';
    }
    return path;
}



// Good
function manipulatePath(path: string): string {
    if (...) {
        path = `hostname/${path}`;
    }
    if (...) {
        path = `${path}/domain`;
    }
    return path;    
}

// Good
function makePath(id: number): string {
return `//.../getById${id}`;
}
```

The only exception to use string concatenation is when string building. If that is the case make sure that the string being built is not the input parameter but a new string.

```typescript
// Good (only exception)
function htmlPage(options: PageOptions): string {
    let page = '<html><body>'
    if (options.header) {
        page += `<div class="header">${options.header}</div>`
    }    
    if (options.content) {
        page += `<div class="content">${options.content}</div>`
    }    
    if (options.footer) {
        page += `<div class="footer">${options.footer}</div>`
    }    
    page += '</body></html>'
    return page;
}
```


## Naming & order

### Subject as Observable notation
Like in the RxJS docs we abbreviate 'observer' with a dollar sign.
Why not put observer in the name?
There are a lot of observers, tho minimize the reading that a person needs to do, we use this symbol.
The name of the observer should preferably have the same name as the variable and subject.

```typescript
// Bad
export class FileService {
    private loadedSubject: Subject = new Subject<boolean>();  
    loadedObservable: Observable<boolean> = loadedSubject.asObservable();
    
    private loaded = false;  
}
    
// Good
export class FileService {
    private loaded = false;
    private loadedSubject: Subject = new Subject<boolean>();  
    loaded$: Observable<boolean> = loadedSubject.asObservable();  
}
    
// Good
export class FileService {
    private loadedSubject: Subject = new Subject<boolean>();  
    loaded$: Observable<boolean> = loadedSubject.asObservable();

    private loaded = false;  
}
```

### Class member ordering
The order is decided based on how often the type of access modifier is needed, with addition of RxJS specific members
First comes all public properties because this is relevant for everyone, then comes protected properties because they are only accessible in the same namespace. Lastly comes all private properties because they are only relevant for people working on the same class.
At the very top of the file, even above the public properties, we define the (commonly private) subjects first and after the public "asObservable" properties that expose the read-only Observables.

```typescript
// Bad
export class FileService {
    // private
    // public
}

// Bad
export class FileService {
    // public
    // protected
    // private
    
    // asObservable
}

// Good
export class FileService {
    // subjects
    
    // public observables
    
    // public
    
    // protected
    
    // private
}
```

### Read-only constant values
Variables that are read-only and used to name a constant value have to be uppercase. A lower dash _ should be used as word separator.
This makes it easier to identify a constant against a reassignable variable

```typescript
// Bad
const DEFAULTLOADED = true;

// Bad
const defaultLoaded = true;

// Bad
const default_loaded = true;

// Good
export const DEFAULT_LOADED = true;

// Good
private readonly DEFAULT_LOADED = true;

// Good
readonly DEFAULT_LOADED = true;
```

## Methods

### Maximum method parameters and options object

When using a method with more than 3 parameters, it's mandatory to use an options object to group the parameters.
This is to avoid the problem of having to change the order of the parameters when adding a new parameter. 
And it's also to make it easier to read the method signature.

When multiple parameters are used, it's good practice to place the options object as the last parameter.

```typescript
// Bad
function compare(a: number, b: number, reversed: true, absolute: true): number {
    // implementation
}

compare(1, 2, true, true);

// Good
function compareGood(a: number, b: number, options: { reversed: true, absolute: true }): number {
    // implementation
}

compare(1, 2, { reversed: true, absolute: true });
```
