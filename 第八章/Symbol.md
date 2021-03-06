# 概览  
```javascript
const mySymbol = Symbol('mySymbol');
console.log(mySymbol); // Symbol(mySymbol)
console.log(mySymbol === Symbol('mySymbol')); // false
console.log(typeof mySymbol); // 'symbol'
```

### 基本数据类型`Symbol`

> ES6 六种基本数据类型: `String`、`Number`、`Boolean`、`Null`、`Undefined`、`Symbol`  
> ES6 七种数据类型：`String`、`Number`、`Boolean`、`Null`、`Undefined`、`Object`、`Symbol`

### 类型转换 

* 转字符串（不支持强制转换成字符串）  
```javascript
// 转String
console.log(mySymbol.toString()); // 'Symbol(mySymbol)'
console.log(String(mySymbol)); // 'Symbol(mySymbol)'
// console.log(mySymbol + ''); // Cannot convert a Symbol value to a string
// console.log(`${mySymbol}`); // Cannot convert a Symbol value to a string
```  

* 转Boolean  
```javascript
// 转Boolean  
console.log(Boolean(mySymbol)); // true
console.log(!mySymbol); // false
if(mySymbol){
  console.log(123); // 123
}
```  

* 转Number(不支持)
```javascript
// console.log(Number(mySymbol)); //Uncaught TypeError: Cannot convert a Symbol value to a number
// console.log(+mySymbol); //Uncaught TypeError: Cannot convert a Symbol value to a number
// console.log(1+mySymbol); //Uncaught TypeError: Cannot convert a Symbol value to a number
```  

### 自定义`Symbol`属性键  

* 第一种方法
```javascript
let ourSymbol = Symbol();
let a = {};
a[ourSymbol] = 'hello'; 
console.log(a); // {Symbol(): "hello"}
```

* 第二种方法  
```javascript
let ourSymbol = Symbol();
let b = {
  [ourSymbol]: 'world'
};
console.log(b); // {Symbol(): "world"}
```  

* 第三种方法  
```javascript
let ourSymbol = Symbol();
let c = {};
Object.defineProperty(c,ourSymbol,{value: '!'});
console.log(c); // {Symbol(): "!"}
```  

**注意：不支持点运算符** 
```javascript
let ourSymbol = Symbol();
a.ourSymbol = '点运算符';
console.log(a[ourSymbol]); // undefined
console.log(a['ourSymbol']); // '点运算符'
```

**注意：在对象内部，使用Symbol值定义属性时，Symbol值必须放在方括号中。**  
```javascript
let s = Symbol();
let objNew = {
  [s]:function(arg){
    console.log(arg);
  }
};
let objNew1 = {
  [s](arg){
    console.log(arg);
  }
};
objNew[s](123)
objNew1[s](234)
```

### `Symbol`作为属性键  

```javascript
const MY_KEY = Symbol();
const MY = Symbol();
const FOO = Symbol();
const objkEY = {
  [MY]:333,
  [FOO](fo){
    return fo
  }
};
objkEY[MY_KEY] = 123;
console.log(objkEY[MY_KEY]); // 123
console.log(objkEY[MY]); // 333
console.log(objkEY[FOO](11)); // 11
```  

### 枚举属性键  

```javascript
const keyObj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
}
Object.defineProperty(keyObj, 'nonEnum', {enumerable: false});
console.log(keyObj); // {enum: 2, nonEnum: 3, Symbol(my_key): 1}
console.log(Object.getOwnPropertyNames(keyObj)); // 忽略Symbol属性键：["enum", "nonEnum"]
console.log(Object.getOwnPropertySymbols(keyObj)); // 忽略字符串属性键：[Symbol(my_key)]
console.log(Reflect.ownKeys(keyObj)); // 遍历所有属性键：["enum", "nonEnum", Symbol(my_key)]
console.log(Object.keys(keyObj)); // 遍历可枚举字符串属性键：["enum"]
```

### `Symbol`用来表示概念

用于定义一种常量，保证这组常量不相等  
```javascript
let levels = {
  DEBUG:Symbol('debug'),
  INFO:Symbol('info'),
  WARN:Symbol('warn')
}
console.log(levels.DEBUG, 'debug message');
console.log(levels.INFO, 'info message');
```  

其他任何值都不能有相同的值————保证switch语句设计的方式工作  
```javascript
const COLOR_RED = Symbol();
const COLOR_GREEN = Symbol();
function getComplement(color) {
  switch(color) {
    case COLOR_RED:
      return COLOR_GREEN;
    case COLOR_GREEN:
      return COLOR_RED;
    default: 
      throw new Error('undefined')
  }
}
console.log(getComplement(COLOR_RED));
console.log(getComplement(COLOR_GREEN));
// console.log(getComplement('21321'));
```  

### `Symbol`的方法和属性  

```
console.dir(Symbol);
// ƒ Symbol()
// arguments: (...)
// asyncIterator: Symbol(Symbol.asyncIterator)
// caller: (...)
// for: ƒ for()
// hasInstance: Symbol(Symbol.hasInstance)
// isConcatSpreadable: Symbol(Symbol.isConcatSpreadable)
// iterator: Symbol(Symbol.iterator)
// keyFor: ƒ keyFor()
// length: 0
// match: Symbol(Symbol.match)
// matchAll: Symbol(Symbol.matchAll)
// name: "Symbol"
// prototype: Symbol {constructor: ƒ Symbol()
//     description: (...)
//     toString: ƒ toString()
//     valueOf: ƒ valueOf()
//     Symbol(Symbol.toPrimitive): ƒ [Symbol.toPrimitive]()
//     Symbol(Symbol.toStringTag): "Symbol"
//     get description: ƒ description()
//     __proto__: Object
// }
// replace: Symbol(Symbol.replace)
// search: Symbol(Symbol.search)
// species: Symbol(Symbol.species)
// split: Symbol(Symbol.split)
// toPrimitive: Symbol(Symbol.toPrimitive)
// toStringTag: Symbol(Symbol.toStringTag)
// unscopables: Symbol(Symbol.unscopables)
```

> `Symbol.for()` 接受一个字符串作参数，然后搜索有没有以该参数作为名称的Symbol值。如果有就返回Symbol值，否则返回一个以该字符串为名称的Symbol值  
> `Symbol.keyFor()` 返回一个已登记的`Symbol`类型值的key  

```javascript
const s1 = Symbol.for('foo');
const s2 = Symbol.for('foo');
const s3 = Symbol('foo');
const s4 = Symbol('foo');
console.log(s1 === s2); // true
console.log(s3 === s4); // false
console.log(Symbol.keyFor(s1)); // 'foo'
console.log(Symbol.keyFor(s2)); // 'foo'
console.log(Symbol.keyFor(s3)); // undefined
console.log(Symbol.keyFor(s4)); // undefined
```

> `Symbol.prototype.toString()` 方法返回当前 symbol 对象的字符串表示。  
> `Symbol.prototype.valueOf()` 方法返回当前 symbol 对象所包含的 symbol 原始值。  

```javascript
// console.log(Symbol('foo') + 'bar'); // Uncaught TypeError: Cannot convert a Symbol value to a string
console.log(Symbol('foo').toString() + 'bar'); // Symbol(foo)bar
console.log(Object(Symbol('foo')).toString() + 'bar'); // Symbol(foo)bar
Symbol("desc").toString();   // "Symbol(desc)"
// well-known symbols
Symbol.iterator.toString();  // "Symbol(Symbol.iterator)
// global symbols
Symbol.for("foo").toString() // "Symbol(foo)"

// Object(Symbol("foo")) + "bar";
// TypeError: can't convert symbol object to primitive
// 无法隐式的调用 valueOf() 方法
console.log(Object(Symbol("foo")).valueOf()); // Symbol(foo)
console.log(Symbol("foo").valueOf()); // Symbol(foo)
// console.log(Object(Symbol("foo")).valueOf() + "bar");
// TypeError:  can't convert symbol to string
// 手动调用 valueOf() 方法，虽然转换成了原始值，但 symbol 原始值不能转换为字符串
console.log(Object(Symbol("foo")).toString() + "bar");
// "Symbol(foo)bar"，需要手动调用 toString() 方法才行
```

> `Symbol.asyncIterator` 符号指定了一个对象的默认异步迭代器。如果一个对象设置了这个属性，它就是异步可迭代对象，可用于`for await...of`循环。
>   
> `Symbol.hasInstance`用于判断某对象是否为某构造器的实例。因此你可以用它自定义`instanceof`操作符在某个类上的行为。  
> 
> `Symbol.isConcatSpreadable`符号等于一个布尔值；用于配置某对象作为`Array.prototype.concat()`方法的参数时是否展开其数组元素。  
> 
> `Symbol.iterator`为每一个对象定义了默认的迭代器。该迭代器可以被`for...of`循环使用。  
> 
> `Symbol.match`指定了匹配的是正则表达式而不是字符串。  
> 
> `Symbol.matchAll`返回一个迭代器，该迭代器根据字符串生成正则表达式的匹配项。  
> 
> `Symbol.replace`这个属性指定了当一个字符串替换所匹配字符串时所调用的方法。  
> 
> `Symbol.search` 指定了一个搜索方法，这个方法接受用户输入的正则表达式，返回该正则表达式在字符串中匹配到的下标。  
> 
> `Symbol.species`是个函数值属性，其被构造函数用以创建派生对象。  
> 
> `Symbol.split`指向 一个正则表达式的索引处分割字符串的方法。
> 
> `Symbol.toPrimitive` 是一个内置的 Symbol 值，它是作为对象的函数值属性存在的，当一个对象转换为对应的原始值时，会调用此函数。
> 
> `Symbol.toStringTag` 是一个内置 symbol，它通常作为对象的属性键使用，对应的属性值应该为字符串类型，这个字符串用来表示该对象的自定义类型标签，通常只有内置的 `Object.prototype.toString()` 方法会去读取这个标签并把它包含在自己的返回值里。  
> 
> `Symbol.unscopables` 指用于指定对象值，其对象自身和继承的从关联对象的 `with` 环境绑定中排除的属性名称。


