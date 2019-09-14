## Type Compatibility
### Prerequisites
* 名义类型(Nominal Typing), eg.: C++, Java, Swift...

```
class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: string) { /* ... */ } }

let foo: Foo = new Bar(); // Error!
```
* 结构类型(Structural Typing), eg.: OCaml, Haskell, Elm...

```
class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: string) { /* ... */ } }

let foo: Foo = new Bar(); // Works!
```

### Introduction
TypeScript里的类型兼容性基于结构子类型的。

```
interface Named {
    name: string;
}

class Person {
    name: string;
}

let p: Named;
// OK, because of structural typing
p = new Person();
```

### Start
* TypeScript结构化类型系统的基本规则是，如果x要兼容y，那么y至少具有与x相同的属性。比如：

```
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: "Alice", location: "Seattle" };
x = y;
```
要检查y是否能赋值给x，编译器检查x中的每个属性，看是否能在y中也找到对应属性。

* 检查函数参数时使用相同的规则：

```
function greet(n: Named) {
    console.log("Hello, " + n.name);
}
greet(y); // OK
```
注意，y有个额外的`location`属性，但这不会引发错误。 只有目标类型（这里是Named）的成员会被一一检查是否兼容。  
这个比较过程是递归进行的，检查每个成员及子成员。

### Comparing two functions
```
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```
要查看x是否能赋值给y，首先看它们的参数列表。 x的每个参数必须能在y里找到对应类型的参数。 注意的是参数的名字相同与否无所谓，只看它们的类型。 这里，x的每个参数在y中都能找到对应的参数，所以允许赋值。  
第二个赋值错误，因为y有个必需的第二个参数，但是x并没有，所以不允许赋值。

**why allow 'discarding' parameters? 允许忽略参数**

忽略额外的参数在JavaScript里是很常见的。例如，`Array#forEach`给回调函数传3个参数：数组元素，索引和整个数组。 尽管如此，传入一个只使用第一个参数的回调函数也是很有用的：

```
let items = [1, 2, 3];

// Don't force these extra parameters
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach(item => console.log(item));
```

--

如何处理返回值类型：  
创建两个仅是返回值类型不同的函数：

```
let x = () => ({name: "Alice"});
let y = () => ({name: "Alice", location: "Seattle"});

x = y; // OK
y = x; // Error, because x() lacks a location property
```
类型系统强制源函数的返回值类型必须是目标函数返回值类型的子类型。

### Enums
枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的。例如：

```
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Green;  // Error
```

### Classes
类与对象字面量和接口差不多，但有一点不同：类有静态部分和实例部分的类型。  
比较两个类类型的对象时，只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内。

```
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```

### Generics
因为TypeScript是结构性的类型系统，类型参数只影响使用其做为类型一部分的结果类型。比如：

```
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // OK, because y matches structure of x
```

上面代码里，x和y是兼容的，因为它们的结构使用类型参数时并没有什么不同。  
把这个例子改变一下，增加一个成员，就能看出是如何工作的了：

```
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // Error, because x and y are not compatible
```
在这里，泛型类型在使用时就好比不是一个泛型类型。  
对于没指定泛型类型的泛型参数时，会把所有泛型参数当成any比较。 然后用结果类型进行比较，就像上面第一个例子。例如：

```
let identity = function<T>(x: T): T {
    // ...
};

let reverse = function<U>(y: U): U {
    // ...
};

identity = reverse;  // OK, because (x: any) => any matches (y: any) => any
```