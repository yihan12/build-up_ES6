# 预览

> ES6中实现的一个语法糖，用于简化基于原型集成实现类定义的场景。  
> 提供更便利的方法去创建老式的构造器函数。  

类和子类  
```javascript
class Point {
  constructor(x, y){
    this.x = x;
    this.y = y;
  }
  toString() {
    return `(${this.x}, ${this.y})`
  }
}

class ColorPoint extends Point{
  constructor(x, y, color){
    super(x, y);
    this.color = color;
  }
  toString(){
    return super.toString() + ' in ' + this.color
  }
}

// 使用： 
const cp = new ColorPoint(25, 8, 'green');

console.log(cp.toString()); // '(25, 8) in green'
console.log(cp instanceof ColorPoint); // true
console.log(cp instanceof Point); // true
console.log(typeof Point); // 'function'
```

### 要点  

* **一、只能通过`new`调用类，不能通过函数调用。**  
```javascript
class Point{
  constructor(x, y){
    this.x = x;
    this.y = y;
  }
  toString(){
    return `(${this.x}, ${this.y})`
  }
}
const p = new Point(25, 8);
console.log(p.toString()); // '(25, 8)'

console.log(typeof Point); // 'function'

Point(); // Uncaught TypeError: Class constructor Point cannot be invoked without 'new'
```

* **二、类声明不能被提升。**  
```javascript
new Foo(); // Uncaught ReferenceError: Cannot access 'Foo' before initialization
class Foo{}
```

* **三、类的表达式定义。**  
```javascript
const MyClass  = class Me{
  getClassName(){
    return Me.name
  }
}
const init = new MyClass();
console.log(init.getClassName()); // Me
console.log(MyClass.name); // Me
console.log(Me.name); // Uncaught ReferenceError: Me is not defined
```

* **四、类的构造器、静态方法、原型方法**  
```javascript
class Foo{
  constructor(prop){
    this.prop = prop;
  }
  static staticMethod(){
    return 'classy';
  }
  prototypeMethod(){
    return 'prototypical'
  }
}
const foo = new Foo(123);
console.log(Foo === Foo.prototype.constructor); // true
console.log(typeof Foo); // 'function'

// 静态方法
console.log(typeof Foo.staticMethod); // 'function'
console.log(Foo.staticMethod()); // 'classy'
console.log(Foo.prototype.prototypeMethod()); // 'prototypical'
console.log(Foo.prototypeMethod()); // Uncaught TypeError: Foo.prototypeMethod is not a function

// 原型方法
console.log(typeof Foo.prototype.prototypeMethod); // 'function'
console.log(foo.prototypeMethod()); // 'prototypical'
console.log(foo.staticMethod()); // Uncaught TypeError: foo.staticMethod is not a function
```
父类的静态方法可以被子类继承  
```javascript
class Foo{
  static classMethod(){
    return 'hello'
  }
}
class Bar extends Foo{}
console.log(Bar.classMethod()); // 'hello'
```
静态方法也可以从`super`对象上调用  
```javascript
class Foo{
  static classMethod(){
    return 'hello'
  }
}
class Bar extends Foo{
  static classMethod(){
    return super.classMethod() + ', too';
  }
}
console.log(Bar.classMethod()); // 'hello, too'
```

* **五、类的静态属性和实例属性。**   
静态属性  
```javascript
// 旧方法
class Foo{

}
Foo.prop = 1;

// 新方法
class MyClass {
  static myStaticProp = 1;
  constructor(){
    console.log(MyClass.myStaticProp); // 1
  }
}
console.log(MyClass.props); // 1
```
实例属性  
```javascript
class MyClass{
  myProp = 42;
  static props = 1;
  constructor(){
    console.log(MyClass.props); // 1
    console.log(this.myProp); // 42
  }
}
console.log(MyClass.myProp); // undefined
console.log(MyClass.props); // 1  
const p = new MyClass();
console.log(p.myProp); // 42
console.log(p.props); // undefined
```
实例属性的使用
```javascript
// 以前定义实例属性时，只能写在类的`constructor`方法里
class ReactCounter extends React.Component{
  constructor(props){
    super(props);
    this.state = {
      count:0
    }
  }
}

// 新写法
class ReactCounter extends React.Component{
  state = {
    count: 0
  }
}
// 增强可读性
class ReactCounter extends React.Component{
  state;
  constructor(props){
    super(props);
    this.state = {
      count: 0 
    }
  }
}
```

* **六、类的取值函数（getter）和存值函数（setter）**   
prop属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了。
```javascript
class MyClass{
  constructor(){}
  get prop(){
    return 'getter';
  }
  set prop(value){
    console.log('setter:' + value);
  }
}
let inst = new MyClass();
inst.prop = 123; // 'setter:123'
console.log(inst.prop); // 'getter'
```

存值函数和取值函数是设置在属性的Descriptor对象上的。  
```javascript
class CustomHTMLElement{
  constructor(element){
    this.element = element;
  }
  get html(){
    return this.element.innerHTML;
  }
  set html(value){
    this.element.innerHTML = value
  }
}
const descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype,'html'
);
console.log('get' in descriptor); // true
console.log('set' in descriptor); // true
```

* **七、类的Generator方法**  
```javascript
class IterableArguments{
  constructor(...args){
    this.args = args;
  }
  *[Symbol.iterator](){
    for(let arg of this.args){
      yield arg;
    }
  }
}
for(let x of new IterableArguments('hello','world')){
  console.log(x);
}
// 'hello'
// 'world'
```

### Class的继承  

> Class可以通过extends关键字实现继承  

```javascript
class Point{
  constructor(x, y ){
    this.x = x;
    this.y = y;
  }
  toString() {
    return `(${this.x}||${this.y})`
  }
} 
class ColorPoint extends Point{
  constructor(x, y, color){
    super(x, y); // 调用父类的constructor(x,y)
    this.color = color; // 子类的constructor方法没有调用super之前不能使用this关键字
  }

  toString(){
    return this.color + ' ' + super.toString(); // 调用父类的toString
  }
}

const cp = new ColorPoint(25, 8, 'green');
console.log(cp.toString()); // 'green (25||8)'
console.log(cp instanceof ColorPoint); // true
console.log(cp instanceof Point); // true
// Object.getPrototypeof()可以用来从子类获取父类
console.log(Object.getPrototypeOf(ColorPoint) === Point); // true
```

**`super`关键字**  
* 第一种情况，super作为函数调用时代表父类的构造函数。  
```javascript
class A {
  constructor(){
    console.log(new.target.name);
  }
}
class B extends A{
  constructor(){
    super() // 相当于A.prototype.constructor.call(this)
  }
}

new A() // A
new B() // B
```

* 第二种情况，super作为对象时，在普通方法中指向父类的原型对象；在静态方法中指向父类。  
```javascript
class A{
  p(){
    return 2;
  }
}
class B extends A{
  constructor(){
    super();
    console.log(super.p()); // 2
    // super.p() 相当于A.prototype.p()
  }
}
let b = new B();
```

由于`super`指向父类的原型对象，所以定义在父类实例上的方法或属性是无法通过`super`调用的。  
```javascript
class A{
  constructor(){
    this.p = 2;
  }
}
class B extends A{
  get m(){
    return super.p;
  }
}
let b = new B();
console.log(b.m); // undefined
```

上面的代码中，p是父类A的实例的属性，因此`super.p`就引用不到它  

如果属性定义在父类的原型对象上，`super`就可以取到。  
```javascript
class A{}
A.prototype.x = 2;
class B extends A{
  constructor(){
    super();
    console.log(super.x); // 2
  }
}
let b = new B();
```

ES6规定，通过`super`调用父类的方法时，`super`会绑定子类的`this`。  
```javascript
class A{
  constructor(){
    this.x = 1;
  }
  print(){
    console.log(this.x); // 2
  }
}
class B extends A{
  constructor(){
    super();
    this.x = 2;
  }
  m(){
    super.print();
  }
}
let b = new B();
b.m()
```

由于绑定子类的this，因此如果通过`super`对某个属性赋值，这是`super`就是`this`，赋值的属性会变成子类属性的实例。  
```javascript
class A{
  constructor(){
    this.x = 1;
  }
}
class B extends A{
  constructor(){
    super();
    this.x = 2;
    super.x = 3;
    // super.x 相当于读取的时A.prototype.x
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}
let b = new B();
```

如果`super`作为对象用在静态方法之中，这时`super`将指向父类，而不是父类的原型对象。  
```javascript
class Parent{
  static myMethod(msg){
    console.log('static',msg);
  }

  myMethod(msg){
    console.log('instance', msg);
  }
}
class Child extends Parent{
  static myMethod(msg){
  // super在静态方法中指向父类
    super.myMethod(msg);
  }

  myMethod(msg){
  // super在普通方法中指向父类的原型对象
    super.myMethod(msg);
  }
}
Child.myMethod(1); // static 1

const child = new Child(); 
child.myMethod(2); // instance 2
```

使用super的时候，必须显式指定是作为函数还是作为对象使用，否则会报错。  
```javascript
class A{}

class B extends A{
  constructor(){
    super();
    console.log(super); // 报错
  }
}

// 正确的使用
class C extends A{
  constructor(){
    super();
    console.log(super.valueOf() instanceof C); // true
  }
}
let c = new C();
```

由于对象总是继承其他对象，所以可以在任意一个对象中使用`super`关键字
```javascript
const obj = {
  toString(){
    return 'MYobject:' + super.toString();
  }
}
console.log(obj.toString()); // 'MYobject:[object Object]'
```
