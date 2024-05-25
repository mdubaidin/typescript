# Everyday Types

### The primitives: string, number and boolean.

### Type Annotations on Variables

-   Automatically typscript detects the type of the following variables no need to be defined explicitly.

```typescript
const text = 'hello world';
const num = 123;
const bol = false;
```

-   Explicitly defined the types of a variables

```typescript
const textWithType: string = 'hello world';
const numWithType: number = 123;
const bolWithType: boolean = false;
```

### Functions

Functions are the primary means of passing data around in JavaScript. TypeScript allows you to specify the types of both the input and output values of functions.

-   Parameter type anotation

```typescript
const getValue = (v: number) => v * 2;
getValue(10); // 20
```

-   Return type anotation

```typescript
const getFavNumber = (): number => 42;
console.log(typeof getFavNumber()); // number
```

### Object types

It refers to any JavaScript value with properties, which is almost all of them! To define an object type, we simply list its properties and their types.

```typescript
function printCard(card: { name: String; address: String; date: Date }) {
    console.log(card.name, card.address, card.date.toLocaleDateString());
}

printCard({ name: 'Ubaid', address: 'Gonda', date: new Date('2017-10-10') });
```

Optional types

```typescript
function printName(obj: { first: string; last?: string }) {
    console.log(obj.first, obj?.last);
}
// Both OK
printName({ first: 'Bob' });
printName({ first: 'Aiden', last: 'Markram' });
```

### Union Types

A union type is a type formed from two or more other types, representing values that may be any one of those types.

```typescript
function userId(id: number | string) {
    console.log('A User id is ', id);
}

userId(123);
userId('1654dfasd');
```

#### Working with Union Types

It’s easy to provide a value matching a union type - simply provide a type matching any of the union’s members. If you have a value of a union type, how do you work with it?

TypeScript will only allow an operation if it is valid for every member of the union. For example, if you have the union string | number, you can’t use methods that are only available on string:

```typescript
function printId(id: number | string) {
    console.log(id.toUpperCase());
}
```

The following example will throw an error if it is not handled properly

> [!CAUTION]
> Property 'toUpperCase' does not exist on type 'string | number'. <br />
> Property 'toUpperCase' does not exist on type 'number'.

The solution is to narrow the union with code, the same as you would in JavaScript without type annotations.

```typescript
function printId(id: number | string) {
    if (typeof id === 'string') {
        // In this branch, id is of type 'string'
        console.log(id.toUpperCase());
    } else {
        // Here, id is of type 'number'
        console.log(id);
    }
}
```

### Type Aliases

We’ve been using object types and union types by writing them directly in type annotations. This is convenient, but it’s common to want to use the same type more than once and refer to it by a single name.

```typescript
type Card = {
    name: string;
    address: string;
    date: Date;
};

function printCard(card: Card) {
    console.log(card.name, card.address, card.date.toLocaleDateString());
}

printCard({ name: 'Ubaid', address: 'Gonda', date: new Date('2017-10-10') });
```

You can actually use a type alias to give a name to any type at all, not just an object type. For example, a type alias can name a union type:

```typescript
type ID = string | number;
```

### Interfaces

An interface declaration is another way to name an object type:

```typescript
interface Card {
    name: string;
    address: string;
    date: Date;
}

function printCard(card: Card) {
    console.log(card.name, card.address, card.date.toLocaleDateString());
}

printCard({ name: 'Ubaid', address: 'Gonda', date: new Date('2017-10-10') });
```

### Type Aliases v/s Interfaces

Type aliases and interfaces are very similar, and in many cases you can choose between them freely. Almost all features of an interface are available in type, the key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable.

<table>
<tbody>
<tr><th>Type Aliases</th><th>Interfaces</th></tr>
<tr><td>Extending a type via intersections <br /><br />
<pre><code> type Animal = {
    name: string;
} <br />
type Bear = Animal & { 
    honey: boolean;
} <br />
const bear = getBear();
bear.name;
bear.honey;</code></pre></td>
<td>Extending an Interfaces <br /><br />
<pre><code> interface Animal {
  name: string;
} <br />
interface Bear extends Animal {
honey: boolean;
} <br />
const bear = getBear();
bear.name;
bear.honey;</code></pre>
</td></tr>
<tr><td>Extending a type via intersections <br /><br />
<pre><code>type Window = {
  title: string;
} <br />
type Window = {
  ts: TypeScriptAPI;
} <br />
<p style='color:#D70040; font-weight:300'>// Error: Duplicate identifier 'Window'.</p></code></pre></td>
<td>Adding new fields to an existing interface<br /><br />
<pre><code> interface Window {
  title: string;
} <br />
interface Window {
  ts: TypeScriptAPI;
} <br />
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});</code></pre>
</td></tr>

</tbody>
</table>

### Type Assertions

Sometimes you will have information about the type of a value that TypeScript can’t know about.

```typescript
const myCanvas = document.getElementById('main_canvas') as HTMLCanvasElement;
```

### Literal Types

In addition to the general types string and number, we can refer to specific strings and numbers in type positions.

```typescript
let x: 'hello' = 'hello';
// OK
x = 'hello';
```

> [!WARNING]
> x = "world";  
> Type '"howdy"' is not assignable to type '"hello"'.
