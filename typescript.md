# TypeScript related conventions

<!-- TOC -->
* [TypeScript related conventions](#typescript-related-conventions)
  * [Types](#types)
    * [Utility types](#utility-types)
      * [Record](#record)
        * [Example](#example)
    * [Casting number to string](#casting-number-to-string)
    * [Enums](#enums)
    * [String manipulation](#string-manipulation)
  * [Naming & order](#naming--order)
    * [Subject as Observable notation](#subject-as-observable-notation)
    * [Class member ordering](#class-member-ordering)
    * [Read-only constant values](#read-only-constant-values)
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
1. **Limited Flexibility**: TypeScript enums are limited in functionality compared to other constructs like objects or union types. For example, you can't have enums with computed or string values.
2. **No Type Safety Across Enums**: Enums in TypeScript are not type-safe across different enum types. This means you can mistakenly assign a value of one enum type to a variable of another enum type, which might lead to unexpected behavior or errors.
3. **Runtime Overhead**: Enums are compiled into JavaScript, where they're represented as objects. If you have a large enum, this could introduce some overhead in terms of memory usage and performance.

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
