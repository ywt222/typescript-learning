## Basic Types

### Boolean  
`let isDone: boolean = false;`

### Number

```
let decimal: number = 6;  
let hex: number = 0xf00d;  
let binary: number = 0b1010;  
let octal: number = 0o744;
```

### String

```
let color: string = "blue";  
color = 'red';

let fullName: string = `Bob Bobbington`;  
let age: number = 37;  
let sentence: string = `Hello, my name is ${ fullName }. I'll be ${ age + 1 } years old next month.`;  
let sentence: string = "Hello, my name is " + fullName + ". I'll be " + (age + 1) + " years old next month.";
```

### Array

```
let list: number[] = [1, 2, 3];  
let list: Array<number> = [1, 2, 3];
```

### Tuple

```
// Declare a tuple type  
let x: [string, number];  
// Initialize it  
x = ["hello", 10]; // OK  
// Initialize it incorrectly  
x = [10, "hello"]; // Error
```

* push & pop

```
let mytuple = [10, "Hello", "World", "typeScript"];
mytuple.push(12);
mytuple.push(true);  // Error
console.log(mytuple);
console.log(mytuple.pop() + " popped from the tuple");
console.log(mytuple.pop() + " popped from the tuple");
console.log(mytuple);
```

### Enum

```
enum Color {Red, Green, Blue}  
let c: Color = Color.Green;
```

--

By default, enums begin numbering their members starting at 0.  
You can change this by manually setting the value of one of its members.

```
enum Color {Red = 1, Green, Blue}  
let c: Color = Color.Green;
```

Or, even manually set all the values in the enum:

```
enum Color {Red = 1, Green = 2, Blue = 4}  
let c: Color = Color.Green;  
console.log(Color[2]);    // Green
```

### Any  
Opt-out of type-checking, let the values pass through compile-time checks.  
The `any` type is also handy if you know some part of the type, but perhaps not all of it.

```
let list: any[] = [1, true, "free"];  
list[1] = 100;
```

### Void

```
function warnUser(): void {
  console.log("This is my warning message");  
}
```

Declaring variables of type `void` is not useful because you can only assign `undefined` or `null` to them:
`let unusable: void = undefined;`

### Null & Undefined

1. not extremely useful on their own
      
  ```
  let u: undefined = undefined;
  let n: null = null;
  ```
  
2. By default `null` and `undefined` are subtypes of all other types. That means you can assign `null` and `undefined` to something like `number`.  

  **N.B.:  When using the `--strictNullChecks` flag, `null` and `undefined` are only assignable to `void` and their respective types.  
  Use the union type `string | null | undefined` instead.**

### Never

> The `never` type represents the type of values that never occur.  
> For instance, `never` is the return type for a function expression or an arrow function expression that always throws an exception or one that never returns;  
> Variables also acquire the type `never` when narrowed by any type guards that can never be true.

Examples:

```
// Function returning 'never' must have unreachable end point
function error(message: string): never {
    throw new Error(message);
}

// Inferred return type is 'never'
function fail() {
    return error("Something failed");
}

// Function returning 'never' must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}
```

### Object

`object` is a type that represents the non-primitive type, i.e. any thing that is not `number`, `string`, `boolean`, `symbol`, `null`, or `undefined`.

Example:

```
declare function create(o: object | null): void;

create({ prop: 0 }); // OK
create(null); // OK

create(42); // Error
create("string"); // Error
create(false); // Error
create(undefined); // Error
```

### Type assertions

* `angle-bracket` syntax:

  ```  
  let someValue: any = "this is a string";  
  let strLength: number = (<string>someValue).length;
  ```

* `as` syntax:

  ```  
  let someValue: any = "this is a string";
  let strLength: number = (someValue as string).length;
  ```

