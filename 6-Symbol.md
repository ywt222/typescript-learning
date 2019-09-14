## Symbol
> ref:  
> http://es6.ruanyifeng.com/#docs/symbol#Symbol-isConcatSpreadable
> https://www.jianshu.com/p/d44261924ef3

### Symbol.hasInstance

> 概述：该方法确定构造函数对象是否将对象识别为构造函数实例之一。 由 instanceof 操作符进行调用。

#### 类的`Symbol.hasInstance`属性

对象的`Symbol.hasInstance`属性，指向一个内部方法。当其他对象使用`instanceof`运算符，判断是否为该对象的实例时，会调用这个方法。比如，`foo instanceof Foo`在语言内部，实际调用的是`Foo[Symbol.hasInstance](foo)`。

```
class MyClass {
}
var x = new MyClass()
console.log(x instanceof MyClass); // true
console.log(MyClass[Symbol.hasInstance](x)); // true
```
--

```
class Even {
}
const x = new Even();
console.log(x instanceof Even); // true
console.log(Even[Symbol.hasInstance](x)); // true
```

重写`Even[Symbol.hasInstance]`的静态方法：

```
class Even {
  static [Symbol.hasInstance](obj: any) {
    return Number(obj) % 2 === 0;
  }
}
const a: any = 1;
const b: any = 2;
const x = new Even();
console.log(a instanceof Even); // false
console.log(b instanceof Even); // true
console.log(x instanceof Even); // false 原本判断x是否为Even的实例的方法，被改成了传入的数字%2===0。所以此刻是false。
```

#### 实例的`Symbol.hasInstance`属性

`MyClass`是一个类，`new MyClass()`会返回一个实例。该实例的`Symbol.hasInstance`方法，会在进行`instanceof`运算时自动调用，判断左侧的运算子是否为Array的实例。

```
class MyClass {
  [Symbol.hasInstance](foo: any) {
    return foo instanceof Array;
  }
}
const x: any = new MyClass();

// instance
console.log([1, 2, 3] instanceof x); // true
console.log(x[Symbol.hasInstance]([0, 0, 0])); // true
// class
console.log([1, 2, 3] instanceof MyClass); // false
console.log(x instanceof MyClass); // true
console.log(MyClass[Symbol.hasInstance](x)); // true
```

```
class MyClass {
  [Symbol.hasInstance](foo: any) {
    return foo instanceof Array;
  }

  static [Symbol.hasInstance](foo: any) {
    return foo instanceof Array;
  }
}
const x: any = new MyClass();

// instance
console.log([1, 2, 3] instanceof x); // true
console.log(x[Symbol.hasInstance]([0, 0, 0])); // true
// class
console.log([1, 2, 3] instanceof MyClass); // true
console.log(x instanceof MyClass); // false
console.log(MyClass[Symbol.hasInstance](x)); // false
```

### Symbol.isConcatSpreadable

> 概述：该值为Boolean属性，如果为`true`，则表示对象可以通过`Array.prototype.concat`展开（flatten）到其数组元素上。

#### 数组

```
let arr1 = ['c', 'd'];
console.log(arr1[Symbol.isConcatSpreadable]);   // undefined
console.log(['a', 'b'].concat(arr1, 'e'));   // [ 'a', 'b', 'c', 'd', 'e' ]
```

```
let arr2 = ['c', 'd'];
arr2[Symbol.isConcatSpreadable] = false;
console.log(arr2[Symbol.isConcatSpreadable]);   // false
console.log(['a', 'b'].concat(arr2, 'e'));   // ['a', 'b', ['c','d'], 'e']
```

```
let arr3 = ['c', 'd'];
arr3[Symbol.isConcatSpreadable] = true;
console.log(arr3[Symbol.isConcatSpreadable]);   // true
console.log(['a', 'b'].concat(arr3, 'e'));   // [ 'a', 'b', 'c', 'd', 'e' ]
```
**数组的默认行为是可以展开**，`Symbol.isConcatSpreadable`默认等于`undefined`。该属性等于`true`时，也有展开的效果。

#### 对象

```
const obj: any = { length: 2, 0: 'c', 1: 'd' };
console.log(['a', 'b'].concat(obj, 'e'));  // [ 'a', 'b', { '0': 'c', '1': 'd', length: 2 }, 'e' ]


obj[Symbol.isConcatSpreadable] = false;
console.log(['a', 'b'].concat(obj, 'e'));   // [ 'a', 'b', { '0': 'c', '1': 'd', length: 2 }, 'e' ]


obj[Symbol.isConcatSpreadable] = true;
console.log(['a', 'b'].concat(obj, 'e'));   // ['a', 'b', 'c', 'd', 'e']
```
**对象的默认行为是不展开**。

#### `Symbol.isConcatSpreadable`属性也可以定义在类里面

```
class A1 extends Array {
  constructor() {
    super();
    this[Symbol.isConcatSpreadable] = true;
  }
}
class A2 extends Array {
  constructor() {
    super();
  }
  get [Symbol.isConcatSpreadable] () {
    return false;
  }
}
let a1 = new A1();
a1[0] = 3;
a1[1] = 4;
let a2 = new A2();
a2[0] = 5;
a2[1] = 6;
console.log([1, 2].concat(a1).concat(a2));  // [1, 2, 3, 4, [5, 6]]
```

### Symbol.iterator

> 概述：返回对象的默认迭代器的方法。 由 for-of 语句进行调用。

#### 对象的`Symbol.iterator`属性，指向该对象的默认遍历器方法。

```
const myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```
**note:** `function*` 这种声明方式(function关键字后跟一个星号）会定义一个生成器函数 (generator function)，它返回一个 `Generator` 对象。  
<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*>  
<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator>

#### 对象进行`for...of`循环时，会调用`Symbol.iterator`方法，返回该对象的默认遍历器。

```
class Collection {
  *[Symbol.iterator]() {
    let i = 0;
    while(this[i] !== undefined) {
      yield this[i];
      ++i;
    }
  }
}

let myCollection = new Collection();
myCollection[0] = 1;
myCollection[1] = 2;

for(let value of myCollection) {
  console.log(value);
}
// 1
// 2
```

### Symbol.match
> 概述：将正则表达式与字符串匹配的正则表达式方法。 由 String.prototype.match 方法调用。

对象的`Symbol.match`属性，指向一个函数。当执行`str.match(myObject)`时，如果该属性存在，会调用它，返回该方法的返回值。

```
String.prototype.match(regexp)
// 等同于
regexp[Symbol.match](this)

'abc'.match(/a/);
// 等同于
/a/[Symbol.match]('abc');

class MyMatcher {
  [Symbol.match](string) {
    return 'hello world'.indexOf(string);
  }
}

'e'.match(new MyMatcher());  // 1
// 等同于
(new MyMatcher())[Symbol.match]('e');  //1
```

### Symbol.replace

> 概述：该方法是替换字符串的匹配子字符串的正则表达式方法。 由 String.prototype.replace 方法调用。

对象的`Symbol.replace`属性，指向一个方法，当该对象被`String.prototype.replace`方法调用时，会返回该方法的返回值。

```
String.prototype.replace(searchValue, replaceValue)
// 等同于
searchValue[Symbol.replace](this, replaceValue)
```

```
const x = {};
x[Symbol.replace] = (...s) => console.log(s);

'Hello'.replace(x, 'World'); // ["Hello", "World"]
```
`Symbol.replace`方法会收到两个参数，第一个参数是`replace`方法正在作用的对象(`this`)，上面例子是`Hello`，第二个参数是替换后的值，上面例子是`World`。

### Symbol.search

> 概述：该方法是正则表达式方法，返回与正则表达式匹配的字符串中的索引。 由 String.prototype.search 方法调用。

对象的`Symbol.search`属性，指向一个方法，当该对象被`String.prototype.search`方法调用时，会返回该方法的返回值。

```
String.prototype.search(regexp)
// 等同于
regexp[Symbol.search](this)

class MySearch {
  constructor(value) {
    this.value = value;
  }
  [Symbol.search](string) {
    return string.indexOf(this.value);
  }
}

'foobar'.search(new MySearch('foo'));  // 0
// 等同于
(new MySearch('foo'))[Symbol.search]('foobar');  // 0
```

### Symbol.species

> 概述：用于创建扩展对象的构造函数的函数值的属性。

对象的`Symbol.species`属性，指向一个构造函数。创建衍生对象时，会使用该属性。

```
class MyArray extends Array {
}

const a = new MyArray(1, 2, 3);
const b = a.map(x => x);
const c = a.filter(x => x > 1);

b instanceof MyArray;  // true
c instanceof MyArray;  // true
```

上面代码中，子类`MyArray`继承了父类`Array`，a 是`MyArray`的实例，b 和 c是 a 的衍生对象。你可能会认为，b 和 c 都是调用数组方法生成的，所以应该是数组（`Array`的实例），但实际上它们也是`MyArray`的实例。

`Symbol.species`属性就是为了解决这个问题而提供的。现在，我们可以为`MyArray`设置`Symbol.species`属性。

```
class MyArray extends Array {
  static get [Symbol.species]() { return Array; }
}
```

上面代码中，由于定义了`Symbol.species`属性，创建衍生对象时就会使用这个属性返回的函数，作为构造函数。这个例子也说明，定义`Symbol.species`属性要采用`get`取值器。默认的`Symbol.species`属性等同于下面的写法。

```
static get [Symbol.species]() {
  return this;
}
```

现在，再来看前面的例子。

```
class MyArray extends Array {
  static get [Symbol.species]() { return Array; }
}

const a = new MyArray();
const b = a.map(x => x);

b instanceof MyArray;  // false
b instanceof Array;  // true
```
上面代码中，`a.map(x => x)`生成的衍生对象，就不是`MyArray`的实例，而直接就是`Array`的实例。

再看一个例子。

```
class T1 extends Promise {
}

class T2 extends Promise {
  static get [Symbol.species]() {
    return Promise;
  }
}

new T1(r => r()).then(v => v) instanceof T1;  // true
new T2(r => r()).then(v => v) instanceof T2;  // false
```
上面代码中，T2定义了`Symbol.species`属性，T1没有。结果就导致了创建衍生对象时（`then`方法），T1调用的是自身的构造方法，而T2调用的是`Promise`的构造方法。

总之，`Symbol.species`的作用在于，实例对象在运行过程中，需要再次调用自身的构造函数时，会调用该属性指定的构造函数。它主要的用途是，有些类库是在基类的基础上修改的，那么子类使用继承的方法时，作者可能希望返回基类的实例，而不是子类的实例。

### Symbol.split

> 概述：该方法是正则表达式方法，用于在匹配正则表达式的索引处拆分字符串。 由 String.prototype.split 方法调用。

对象的`Symbol.split`属性，指向一个方法，当该对象被`String.prototype.split`方法调用时，会返回该方法的返回值。

```
String.prototype.split(separator, limit)
// 等同于
separator[Symbol.split](this, limit)
```
例子：

```
class MySplitter {
  constructor(value) {
    this.value = value;
  }
  [Symbol.split](string) {
    let index = string.indexOf(this.value);       // 0       // 3          // -1
    if (index === -1) {
      return string;                                                      // 'foobar'
    }
    return [
      string.substr(0, index),                   // ''      // 'foo'
      string.substr(index + this.value.length)   // 'bar'   // ''
    ];
  }
}

'foobar'.split(new MySplitter('foo'));
// ['', 'bar']

'foobar'.split(new MySplitter('bar'));
// ['foo', '']

'foobar'.split(new MySplitter('baz'));
// 'foobar'
```

### Symbol.toPrimitive

> 概述：将对象转换为相应原始值的方法。 由 ToPrimitive 抽象操作调用。

对象的`Symbol.toPrimitive`属性，指向一个方法。该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。

`Symbol.toPrimitive`被调用时，会接受一个字符串参数，表示当前运算的模式，一共有三种模式。

- Number：该场合需要转成数值
- String：该场合需要转成字符串
- Default：该场合可以转成数值，也可以转成字符串

```
let obj = {
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case 'number':
        return 123;
      case 'string':
        return 'str';
      case 'default':
        return 'default';
      default:
        throw new Error();
     }
   }
};

2 * obj;  // 246
3 + obj;  // '3default'
obj == 'default';  // true
String(obj);  // 'str'
```

### Symbol.toStringTag

> 概述：该方法是 String 值属性，用于创建对象的默认字符串描述。 通过内置方法 Object.prototype.toString 访问。

对象的`Symbol.toStringTag`属性，指向一个方法。在该对象上面调用`Object.prototype.toString`方法时，如果这个属性存在，它的返回值会出现在`toString`方法返回的字符串之中，表示对象的类型。也就是说，这个属性可以用来定制`[object Object]`或`[object Array]`中`object`后面的那个字符串。

```
let a = 'name';
console.log(Object.prototype.toString(a)); // [object Object]

let b = Symbol('name');
console.log(Object.prototype.toString(b)); // [object Object]
```

ECMAScript 语言中一切都是对象！我们使用对象的原型方法，它们的内在最终都是指向 Object，所以该方法就会被覆盖。

```
let a = [1,2,3];
console.log( Object.prototype.toString.call(a) );  // [object Array]

let b = Symbol('name'); 
console.log( Object.prototype.toString.call(b) );  // [object Symbol]
```

```
// 例一
({[Symbol.toStringTag]: 'Foo'}.toString());  // "[object Foo]"

// 例二
class Collection {
  get [Symbol.toStringTag]() {
    return 'xxx';
  }
}
let x = new Collection();
Object.prototype.toString.call(x);  // "[object xxx]"
```

### Symbol.unscopables (官方不推荐)

> 概述：该方法是对象值属性，其自身和继承的属性名称是从 with 关联对象的环境绑定中排除的属性名称。

对象的`Symbol.unscopables`属性，指向一个对象。该对象指定了使用`with`关键字时，哪些属性会被`with`环境排除。

```
// 没有 unscopables 时
class MyClass {
  foo() { return 1; }
}

let foo = function () { return 2; };

with (MyClass.prototype) {
  foo();  // 1
}


// 有 unscopables 时
class MyClass {
  foo() { return 1; }
  get [Symbol.unscopables]() {
    return { foo: true };
  }
}

let foo = function () { return 2; };

with (MyClass.prototype) {
  foo();  // 2
}
```
下面代码说明，数组有 7 个属性，会被`with`命令排除。

```
Array.prototype[Symbol.unscopables]
// {
//   copyWithin: true,
//   entries: true,
//   fill: true,
//   find: true,
//   findIndex: true,
//   includes: true,
//   keys: true,
//   values: true
// }

Object.keys(Array.prototype[Symbol.unscopables])
// ['copyWithin', 'entries', 'fill', 'find', 'findIndex', 'includes', 'keys', 'values']
```