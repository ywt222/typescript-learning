## Generics

```
function identity(arg: number): number {
    return arg;
}
```

```
function identity(arg: any): any {
    return arg;
}
```
We actually are losing the information about what that type was when the function returns.  
If we passed in a number, the only information we have is that any type could be returned.

```
function identity<T>(arg: T): T {
    return arg;
}
```

* Once we’ve written the generic identity function, we can call it in one of two ways.

1. Pass all of the arguments, including the type argument, to the function:

  ```
  let output = identity<string>("myString");  // type of output will be 'string'
  ```

2. Use `type argument inference`:

  ```
  let output = identity("myString");  // type of output will be 'string'
  ```

--

* You actually treat these parameters as if they could be any and all types.

Error:

```
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

Correct:

```
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

### Generic Classes

```
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

```
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

### Generic Constraints

```
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

We’d like to constrain this function to work with any and all types that also have the `.length` property.  
As long as the type has this member, we’ll allow it, but it’s required to have at least this member.  
To do so, we must list our requirement as a constraint on what `T` can be.

```
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}

loggingIdentity(3);  // Error, number doesn't have a .length property
loggingIdentity({length: 10, value: 3});

```

### Using Type Parameters in Generic Constraints

You can declare a type parameter that is constrained by another type parameter.  
For example, here we’d like to get a property from an object given its name.  
We’d like to ensure that we’re not accidentally grabbing a property that does not exist on the obj, so we’ll place a constraint between the two types:

```
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```
