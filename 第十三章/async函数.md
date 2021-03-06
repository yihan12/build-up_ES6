# `async/await`函数  

> `Generator`函数的语法糖；  
> 返回一个`Promise` 对象，可以使用`then`方法添加回调函数；  
> 当函数执行的时候，一旦遇到`await`就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。  

async对Generator函数的改进：  
* 1、内置执行器：`async`函数自带执行器；  
* 2、更好的语义：`async...await`比起星号和`yield`，语义更加清楚；  
* 3、更广的适用性；  
* 4、返回值是Promise。  

### 使用  

```javascript
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}

async function asyncCall() {
  console.log('calling');
  const result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: "resolved"
}

asyncCall();
// 'calling'
// 'resolved'
```

`async`函数返回一个Promise对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。  
```javascript
async function getStockPriceByName(name){
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}
function getStockSymbol(e) {
  console.log(e, 'getStockSymbol');
  return e
}
function getStockPrice(e) {
  console.log(e, 'getStockPrice');
  return e
}
getStockPriceByName('goog').then(function(result){
  console.log(result, 'getStockPriceByName');
})
// 'goog' 'getStockSymbol'
// 'goog' 'getStockPrice'
// 'goog' 'getStockPriceByName'
```

指定多少毫秒后输出一个值  
```javascript
function timeout(ms){
  return new Promise(resolve=>{
    setTimeout(resolve, ms);
  })
}
async function asyncPrint(value, ms){
  await timeout(ms);
  console.log(value);
}
asyncPrint('hello', 3000);
// 'hello'

// 等同于  
async function timeout(ms){
  await new Promise(resolve=>{
    setTimeout(resolve, ms);
  })
}
async function asyncPrint(value, ms){
  await timeout(ms);
  console.log(value);
}
asyncPrint('hello', 3000);
```

`async`函数的多种形式：  
* 函数声明   
```javascript
async function foo(){}
```
* 函数表达式  
```javascript
const foo = async function(){};
```
* 对象的方法  
```javascript
let obj = { async foo() {}};
obj.foo().then(...)
```
* 箭头函数  
```javascript
const foo = async ()=>{};
```
* Class方法  
```javascript
class Strorage{
  constructor(){
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}
const storage = new Storage();
storage.getAvatar('jake').then(
  //...
);
```

返回promise对象  
```javascript
async function f(){
  return 'hello'
}
f().then(v=>console.log(v));
// 'hello'

// async函数抛出错误会导致返回的Promise对象变为reject状态，抛出的对象会被catch的回调函数接收到
async function f1(){
  throw new Error('出错了')
}
f1().then(v=>console.log(v))
.catch(e=>console.log(e));
// Error: 出错了

// 等同于
async function f2(){
  throw new Error('出错了')
}
f2().then(v=>console.log(v),e=>console.log(e))
```

`async`函数返回的Promise对象必须等到内部所有await命令后面的Promise对象执行完成才会发生状态改变，除非遇到return语句，或者抛出错误。  
```javascript
async function getTitle(url){
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1]
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
```

### `await` 命令  

> 正常情况下，await命令后面是一个Promise对象。如果不是，会被转成一个立即resolve的Promise对象。  
```javascript
async function f(){
  return await 123
}
f().then(v=>console.log(v)) // 123
```

`await`命令后面的Promise对象如果变为reject状态，则reject参数会被catch方法回调函数接收到。  
```javascript
async function f(){
  await Promise.reject('出错了')
}
f()
.then(v=>console.log(v))
.catch(e=>{console.log(e)}) // '出错了'
```

解决`await`语句后面的Promise变为reject，整个`async`函数中断执行：  
* 第一种  
```javascript
async function f(){
  try{
    await Promise.reject('出错了')
  }catch(e){
    console.log(e);
  }
  return await Promise.resolve('hello')
}
f()
.then(v=>console.log(v)) 
// '出错了'
// 'hello'
```

* 第二种  
```javascript
async function f(){
  await Promise.reject('出错了')
  .catch(e=>console.log(e))
  return await Promise.resolve('123')
}
f()
.then(v=>console.log(v))
// '出错了'
// '123'
```

错误的处理：  
```javascript
async function f(){
  await new Promise(function(resolve,reject){
    throw new Error('出错了')
  })
}
f()
.then(v=>console.log(v))
.catch(e=>console.log(e))
// Error: 出错了

// try...catch 防止出错
async function f1(){
  try{
    await new Promise(function(resolve,reject){
      throw new Error('出错了！')
    })
  }catch(e){

  }
  return await('hello')
}
f1()
.then(v=>console.log(v))
.catch(e=>console.log(e))
// 'hello'

// try...catch处理多个await命令
function F1(e){
  return new Promise(function(resolve,reject){
    resolve(e)
  })
}
function F2(e){
  return new Promise(function(resolve,reject){
    resolve(e)
  })
}
function F3(e){
  return new Promise(function(resolve,reject){
    resolve(e)
  })
}
async function f3(){
  try{
    const val1 = await F1('F1');
    const val2 = await F2(val1);
    const val3 = await F3(val1+val2);
    console.log('val3:'+val3); // 'val3:F1F1'
  }catch(e){
    console.log(e);
  }
  return await('end')
}
f3()
.then(v=>console.log(v)) // 'end'
.catch(e=>console.log(e))
```

### 按顺序完成一部操作  

```javascript
// Promise的写法
function logInOrder(urls){
  // 远程读取所有的url
  const textPromises = urls.map(url=>{
    return fetch(url).then(response=>response.text())
  })

  // 按次序输出
  textPromises.reduce((chain,textPromise)=>{
    return chain.then(()=>textPromise)
      .then(text=>console.log(text))
  }, Promise.resolve())
}

// async的写法
async function logInorder2(urls){
  for (const url of urls){
    const response = await fetch(url);
    console.log(await reponse.text());
  }
}

// 上面需要等待一个结果后才能执行后续
async function logInOrder3(urls){
  // 并发读取远程URL
  const textPromises = urls.map(async url=>{
    const response = await fetch(url)
    return response.text()
  })

  // 按次序输出
  for(const textPromise of textPromises){
    console.log(await textPromise);
  }
}
```
