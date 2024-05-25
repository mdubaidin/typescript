# Narrowing

Type narrowing in TypeScript refers to refining the type of a variable within a conditional block based on runtime checks. This is achieved through techniques like typeof guards, instance checks, or property existence checks, enabling more precise typing and avoiding type errors in subsequent code execution.

Imagine we have a function called padLeft.

```typescript
function padLeft(padding: number | string, input: string): string {
    return ' '.repeat(padding) + input;
}
```

> [!CAUTION]
> Argument of type 'string | number' is not assignable to parameter of type 'number'.  
> Type 'string' is not assignable to type 'number'.

we’re getting an error on padding. TypeScript is warning us that we’re passing a value with type `number | string` to the repeat function, which only accepts a `number`,

```typescript
function padLeft(padding: number | string, input: string): string {
    if (typeof padding === 'number') {
        return ' '.repeat(padding) + input;
    }
    return padding + input;
}
```

Within our `if` check, TypeScript sees typeof `padding === "number"` and understands that as a special form of code called a type guard.

### `typeof` type guards

As we’ve seen, JavaScript supports a typeof operator which can give very basic information about the type of values we have at runtime. TypeScript expects this to return a certain set of strings:

-   "string"
-   "number"
-   "bigint"
-   "boolean"
-   "symbol"
-   "undefined"
-   "object"
-   "function"

### The `in` operator narrowing

JavaScript has an operator for determining if an object or its prototype chain has a property with a name: the in operator. TypeScript takes this into account as a way to narrow down potential types.

For example, with the code: `"value" in x`. where `"value"` is a string literal and `x` is a union type. The “true” branch narrows `x`’s types which have either an optional or required property `value`, and the “false” branch narrows to types which have an optional or missing property `value`.

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
    if ('swim' in animal) {
        return animal.swim();
    }

    return animal.fly();
}
```

### `instanceof` narrowing

JavaScript has an operator for checking whether or not a value is an “instance” of another value. More specifically, in JavaScript `x instanceof Foo` checks whether the prototype chain of `x` contains `Foo.prototype`. As you might have guessed, `instanceof` is also a type guard, and TypeScript narrows in branches guarded by `instanceof s`.

```typescript
function logValue(x: Date | string) {
    if (x instanceof Date) {
        console.log(x.toUTCString());
    } else {
        console.log(x.toUpperCase());
    }
}
```
