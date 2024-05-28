# Functions

### Function Type Expressions

The simplest way to describe a function is with a function type expression. These types are syntactically similar to arrow functions:

```typescript
function sayHello(fn: (a: string) => void) {
    fn('Hello world');
}

const printToConsole = (s: string) => {
    console.log(s);
};

sayHello(printToConsole);
```

The syntax (a: string) => void means “a function with one parameter, named a, of type string, that doesn’t have a return value”. Just like with function declarations, if a parameter type isn’t specified, it’s implicitly any.

> [!NOTE]
> Note that the parameter name is required. The function type (string) => void means “a function with a parameter named string of type any“!
>
> Of course, we can use a type alias to name a function type:

```typescript
type PrintToConsole = (s: string) => void;

function sayHello(fn: PrintToConsole) {
    // ...
}
```

### Call Signatures

In JavaScript, functions can have properties in addition to being callable. However, the function type expression syntax doesn’t allow for declaring properties. If we want to describe something callable with properties, we can write a call signature in an object type:

```typescript
type FunctionSignature = {
    description: string;
    (args: number): boolean;
};
function doSomething(fn: FunctionSignature) {
    console.log(fn.description + ' returned ' + fn(6));
}

function myFunc(args: number) {
    return args > 3;
}

myFunc.description = 'Function description';

doSomething(myFunc);
```

> [!NOTE]
> Note that the syntax is slightly different compared to a function type expression - use : between the parameter list and the return type rather than =>.

### Construct Signatures

JavaScript functions can also be invoked with the new operator. TypeScript refers to these as constructors because they usually create a new object. You can write a construct signature by adding the new keyword in front of a call signature:

```typescript
type SomeConstructor = {
    new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
    return new ctor('hello');
}
```

Some objects, like JavaScript’s Date object, can be called with or without new. You can combine call and construct signatures in the same type arbitrarily:

```typescript
interface CallOrConstruct {
    (n?: number): string;
    new (s: string): Date;
}
```

### Generic Functions

It’s common to write a function where the types of the input relate to the type of the output, or where the types of two inputs are related in some way. Let’s consider for a moment a function that returns the first element of an array:

```typescript
function firstElem(arr: any[]) {
    return arr[0];
}
```

This function does its job, but unfortunately has the return type any. It’d be better if the function returned the type of the array element.

In TypeScript, generics are used when we want to describe a correspondence between two values. We do this by declaring a type parameter in the function signature:

```typescript
function firstElement<Type>(arr: Type[]): Type | undefined {
    return arr[0];
}
```

By adding a type parameter Type to this function and using it in two places, we’ve created a link between the input of the function (the array) and the output (the return value). Now when we call it, a more specific type comes out:

#### Inference

> [!NOTE]
> Note that we didn’t have to specify Type in this sample. The type was inferred - chosen automatically - by TypeScript.

We can use multiple type parameters as well. For example, a standalone version of map would look like this:

```typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
    return arr.map(func);
}

// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(['1', '2', '3'], n => parseInt(n));
```

> [!NOTE]
> Note that in this example, TypeScript could infer both the type of the Input type parameter (from the given string array), as well as the Output type parameter based on the return value of the function expression (number).

#### Constraints

We’ve written some generic functions that can work on any kind of value. Sometimes we want to relate two values, but can only operate on a certain subset of values. In this case, we can use a constraint to limit the kinds of types that a type parameter can accept.

Let’s write a function that returns the longer of two values. To do this, we need a length property that’s a number. We constrain the type parameter to that type by writing an extends clause:

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
    if (a.length >= b.length) {
        return a;
    } else {
        return b;
    }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest('alice', 'bob');
```

> [!CAUTION]
> // Error! Numbers don't have a 'length' property  
> const notOK = longest(10, 100);

There are a few interesting things to note in this example. We allowed TypeScript to infer the return type of longest. Return type inference also works on generic functions.

Because we constrained Type to { length: number }, we were allowed to access the .length property of the a and b parameters. Without the type constraint, we wouldn’t be able to access those properties because the values might have been some other type without a length property.

The types of longerArray and longerString were inferred based on the arguments. Remember, generics are all about relating two or more values with the same type!

Finally, just as we’d like, the call to longest(10, 100) is rejected because the number type doesn’t have a .length property.

#### Specifying Type Arguments

TypeScript can usually infer the intended type arguments in a generic call, but not always. For example, let’s say you wrote a function to combine two arrays:

```typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
    return arr1.concat(arr2);
}
```

Normally it would be an error to call this function with mismatched arrays:

> [!CAUTION]
> const arr = combine([1, 2, 3], ["hello"]);  
> Type 'string' is not assignable to type 'number'.

If you intended to do this, however, you could manually specify Type:

```typescript
const arr = combine<string | number>([1, 2, 3], ['hello']);
```

### Function Overloads

Some JavaScript functions can be called in a variety of argument counts and types. For example, you might write a function to produce a Date that takes either a timestamp (one argument) or a month/day/year specification (three arguments).

In TypeScript, we can specify a function that can be called in different ways by writing overload signatures. To do this, write some number of function signatures (usually two or more), followed by the body of the function:

```typescript
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
    if (d !== undefined && y !== undefined) {
        return new Date(y, mOrTimestamp, d);
    } else {
        return new Date(mOrTimestamp);
    }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3); // ERROR : No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

In this example, we wrote two overloads: one accepting one argument, and another accepting three arguments. These first two signatures are called the overload signatures.

Then, we wrote a function implementation with a compatible signature. Functions have an implementation signature, but this signature can’t be called directly. Even though we wrote a function with two optional parameters after the required one, it can’t be called with two parameters!

#### Overload Signatures and the Implementation Signature

This is a common source of confusion. Often people will write code like this and not understand why there is an error:

```typescript
function fn(x: string): void;
function fn() {
    // ...
}
// Expected to be able to call with zero arguments
fn();
```

Again, the signature used to write the function body can’t be “seen” from the outside.

> [!NOTE]
> The signature of the implementation is not visible from the outside. When writing an overloaded function, you should always have two or more signatures above the implementation of the function.

The implementation signature must also be compatible with the overload signatures. For example, these functions have errors because the implementation signature doesn’t match the overloads in a correct way:

```typescript
function fn(x: boolean): void;
// Argument type isn't right
function fn(x: string): void;
// This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}
```

```typescript
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
// This overload signature is not compatible with its implementation signature.
function fn(x: string | number) {
    return 'oops';
}
```

#### Writing Good Overloads

Like generics, there are a few guidelines you should follow when using function overloads. Following these principles will make your function easier to call, easier to understand, and easier to implement.

Let’s consider a function that returns the length of a string or an array:

```typescript
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
    return x.length;
}
```

#### Declaring `this` in a Function

TypeScript will infer what the `this` should be in a function via code flow analysis, for example in the following:

```typescript
const user = {
    id: 123,

    admin: false,
    becomeAdmin: function () {
        this.admin = true;
    },
};
```

TypeScript understands that the function user.becomeAdmin has a corresponding this which is the outer object user. this, heh, can be enough for a lot of cases, but there are a lot of cases where you need more control over what object this represents. The JavaScript specification states that you cannot have a parameter called this, and so TypeScript uses that syntax space to let you declare the type for this in the function body.

```typescript
interface DB {
    filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
    return this.admin;
});
```
