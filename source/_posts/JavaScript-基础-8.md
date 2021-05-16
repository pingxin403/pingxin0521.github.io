---
title: JavaScript 常用对象方法（二）
date: 2019-05-02 20:18:59
tags:
 - 前端
 - JavaScript
categories:
 - 前端
 - JavaScript
---

### Object

**Object.assign(target,source1,source2,...)**

该方法主要用于对象的合并，将源对象source的所有可枚举属性合并到目标对象target上,此方法只拷贝源对象的自身属性，不拷贝继承的属性。

<!--more-->

`Object.assign`方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。同名属性会替换。

`Object.assign`只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制。

`Object.assign`可以用来处理数组，但是会把数组视为对象。

```
const target = {
    x : 0,
    y : 1
};
const source = {
    x : 1,
    z : 2 ,
    fn : {
        number : 1
    }
};
Object.assign(target, source);  
// target  {x : 1, y : 1, z : 2, fn : {number : 1}}    // 同名属性会被覆盖
// source  {x : 1, z : 2, fn : {number : 1}}
target.fn.number = 2;                                  // 拷贝为对象引用
// source  {x : 1, z : 2, fn : {number : 2}}
 
 
function Person(){
    this.name = 1
};
Person.prototype.country = 'china';
let student = new Person();
student.age = 29 ;
const young = {insterst : 'sport'};
Object.assign(young,student);
// young {instest : 'sport' , age : 29, name: 1}               // 只能拷贝自身的属性，不能拷贝prototype
 
 
Object.assign([1, 2, 3], [4, 5])                      // 把数组当作对象来处理
// [4, 5, 3]
```

**Object.create(prototype[,propertiesObject])**

使用指定的原型对象及其属性去创建一个新的对象

```
var parent = {
    x : 1,
    y : 1
}
var child = Object.create(parent,{
    z : {                           // z会成为创建对象的属性
        writable:true,
        configurable:true,
        value: "newAdd"
    }
});
console.log(child)
```

**Object.defineProperties(obj,props)**

直接在一个对象上定义新的属性或修改现有属性，并返回该对象。

```
var obj = {};
Object.defineProperties(obj, {
  'property1': {
    value: true,
    writable: true
  },
  'property2': {
    value: 'Hello',
    writable: false
  }
  // etc. etc.
});
console.log(obj)   // {property1: true, property2: "Hello"}
```

**Object.defineProperty(obj,prop,descriptor)**

在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

```
Object.defineProperty(Object, 'is', {
  value: function(x, y) {
    if (x === y) {
      // 针对+0 不等于 -0的情况
      return x !== 0 || 1 / x === 1 / y;
    }
    // 针对NaN的情况
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true 
}); 
 
// 注意不能同时设置(writable，value) 和 get，set方法，否则浏览器会报错 ： Invalid property descriptor. Cannot both specify accessors and a value or writable attribute
```

**Object.keys(obj)**

 返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和使用 for...in 循环遍历该对象时返回的顺序一致 （两者的主要区别是 一个 for-in 循环还会枚举其原型链上的属性）。

```
let arr = ["a", "b", "c"];
console.log(Object.keys(arr));
// ['0', '1', '2']
 
/* Object 对象 */
let obj = { foo: "bar", baz: 42 },
    keys = Object.keys(obj);
console.log(keys);
// ["foo","baz"] 
```

**Object.values()**

方法返回一个给定对象自己的所有可枚举属性值的数组，值的顺序与使用[`for...in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in)循环的顺序相同 ( 区别在于 for-in 循环枚举原型链中的属性 )。

`Object.values`会过滤属性名为 Symbol 值的属性。

```
var an_obj = { 100: 'a', 2: 'b', 7: 'c' };
console.log(Object.values(an_obj)); // ['b', 'c', 'a']
 
var obj = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.values(obj)); // ['a', 'b', 'c']
```

**Object.entries()**

返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 [`for...in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in) 循环遍历该对象时返回的顺序一致（区别在于 for-in 循环也枚举原型链中的属性）

```
const obj = { foo: 'bar', baz: 42 };
console.log(Object.entries(obj)); // [ ['foo', 'bar'], ['baz', 42] ]
 
const simuArray = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.entries(simuArray)); // [ ['0', 'a'], ['1', 'b'],
```

**hasOwnProperty()**

判断对象**自身**属性中是否具有指定的属性。

```
obj.hasOwnProperty('name')
```

**Object.getOwnPropertyDescriptor(obj,prop)**

返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）.

如果指定的属性存在于对象上，则返回其属性描述符对象（property descriptor），否则返回 undefined。

```
var arr = ['name','age'] ;
arr.forEach(val => console.log(Object.getOwnPropertyDescriptor(obj,val)))
 
 
// {value: "js", writable: true, enumerable: true, configurable: true}
// undefined
```

**Object.getOwnPropertyDescriptors(obj)**

获取一个对象的所有自身属性的描述符。

```
var obj = {
    name : 'js',
    age : 20
}
console.log(Object.getOwnPropertyDescriptors(obj))
const source = {
  set foo(value) {
    console.log(value);
  }
};
 
const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
 
 
const obj = Object.create(
  some_obj,
  Object.getOwnPropertyDescriptors({
    foo: 123,
  })
);
```

**Object.getOwnPropertyNames()**

返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组。

```
var obj = { 0: "a", 1: "b", 2: "c"};
 
Object.getOwnPropertyNames(obj).forEach(function(val) {
  console.log(val);
});
 
 
var obj = {
    x : 1,
    y : 2
}
 
Object.defineProperty(obj,'z',{
    enumerable : false
})
console.log(Object.getOwnPropertyNames(obj))  // ["x", "y", "z"] 包含不可枚举属性 。
console.log(Object.keys(obj))                 // ["x", "y"]      只包含可枚举属性 。
```

**Object.getOwnPropertySymbols()** 

返回一个给定对象自身的所有 Symbol 属性的数组。 

**Object.getPrototypeOf()** 

返回指定对象的原型（内部[[Prototype]]属性的值，即__proto__，而非对象的prototype）。 

**isPrototypeOf()**

判断一个对象是否存在于另一个对象的原型链上。 

**Object.setPrototypeOf(obj,prototype)**

设置对象的原型对象 

**Object.is()**

判断两个值是否相同。

如果下列任何一项成立，则两个值相同：

- 两个值都是 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)
- 两个值都是 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)
- 两个值都是 `true` 或者都是 `false`
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且 
  - 都是正零 `+0`
  - 都是负零 `-0`
  - 都是 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN)
  - 都是除零和 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN) 外的其它同一个数字

```
Object.is('foo', 'foo');     // true
Object.is(window, window);   // true
 
Object.is('foo', 'bar');     // false
Object.is([], []);           // false
 
var test = { a: 1 };
Object.is(test, test);       // true
 
Object.is(null, null);       // true
 
// 特例
Object.is(0, -0);            // false
Object.is(-0, -0);           // true
Object.is(NaN, 0/0);         // true
```

**Object.freeze()**

冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。

```
var obj = {
  prop: function() {},
  foo: 'bar'
};
 
// 新的属性会被添加, 已存在的属性可能
// 会被修改或移除
obj.foo = 'baz';
obj.lumpy = 'woof';
delete obj.prop;
 
// 作为参数传递的对象与返回的对象都被冻结
// 所以不必保存返回的对象（因为两个对象全等）
var o = Object.freeze(obj);
 
o === obj; // true
Object.isFrozen(obj); // === true
 
// 现在任何改变都会失效
obj.foo = 'quux'; // 静默地不做任何事
// 静默地不添加此属性
obj.quaxxor = 'the friendly duck';
console.log(obj)
```

**Object.isFrozen()**

判断一个对象是否被冻结 .

**Object.preventExtensions()**

对象不能再添加新的属性。可修改，删除现有属性，不能添加新属性。

```
var obj = {
    name :'lilei',
    age : 30 ,
    sex : 'male'
}
 
obj = Object.preventExtensions(obj);
console.log(obj);    // {name: "lilei", age: 30, sex: "male"}
obj.name = 'haha';
console.log(obj)     // {name: "haha", age: 30, sex: "male"}
delete obj.sex ;
console.log(obj);    // {name: "haha", age: 30}
obj.address  = 'china';
console.log(obj)     // {name: "haha", age: 30}
```

**Object.isExtensible()**

 判断对象是否是可扩展的，[`Object.preventExtensions`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)，[`Object.seal`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal) 或 [`Object.freeze`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) 方法都可以标记一个对象为不可扩展（non-extensible）

**Object.seal()**

`**Object.seal()**` 方法可以让一个对象密封，并返回被密封后的对象。密封一个对象会让这个对象变的不能添加新属性，且所有已有属性会变的不可配置。属性不可配置的效果就是属性变的不可删除，以及一个数据属性不能被重新定义成为访问器属性，或者反之。但属性的值仍然可以修改。尝试删除一个密封对象的属性或者将某个密封对象的属性从数据属性转换成访问器属性，结果会静默失败或抛出`TypeError` `异常. 不会影响从原型链上继承的属性。但 __proto__ (  ) 属性的值也会不能修改。`

```javascript
var obj = {
    prop: function () {},
    foo: "bar"
  };
 
// 可以添加新的属性,已有属性的值可以修改,可以删除
obj.foo = "baz";
obj.lumpy = "woof";
delete obj.prop;
 
var o = Object.seal(obj);
 
assert(o === obj);
assert(Object.isSealed(obj) === true);
 
// 仍然可以修改密封对象上的属性的值.
obj.foo = "quux";
 
// 但你不能把一个数据属性重定义成访问器属性.
Object.defineProperty(obj, "foo", { get: function() { return "g"; } }); // 抛出TypeError异常
 
// 现在,任何属性值以外的修改操作都会失败.
obj.quaxxor = "the friendly duck"; // 静默失败,新属性没有成功添加
delete obj.foo; // 静默失败,属性没有删除成功
 
// ...在严格模式中,会抛出TypeError异常
function fail() {
  "use strict";
  delete obj.foo; // 抛出TypeError异常
  obj.sparky = "arf"; // 抛出TypeError异常
}
fail();
 
// 使用Object.defineProperty方法同样会抛出异常
Object.defineProperty(obj, "ohai", { value: 17 }); // 抛出TypeError异常
Object.defineProperty(obj, "foo", { value: "eit" }); // 成功将原有值改变
```

**Object.isSealed()**

判断一个对象是否被密封

### Array数组方法

`length` 属性的值是一个 0 到 232-1 的整数。

你可以设置 `length` 属性的值来截断任何数组。当通过改变`length`属性值来扩展数组时，实际元素的数目将会增加。例如：将一个拥有 2 个元素的数组的 `length` 属性值设为 3 时，那么这个数组将会包含3个元素，并且，第三个元素的值将会是 `undefined` 。

但是， `length` 属性不一定表示数组中定义值的个数。

```
var fruits = [];
fruits.push('banana', 'apple', 'peach');

console.log(fruits.length); // 3

fruits[5] = 'mango';
console.log(fruits[5]); // 'mango'
console.log(Object.keys(fruits));  // ['0', '1', '2', '5']
console.log(fruits.length); // 6

fruits.length = 10;
console.log(Object.keys(fruits)); // ['0', '1', '2', '5']
console.log(fruits.length); // 10

//Decreasing the length property does, however, delete elements.
fruits.length = 2;
console.log(Object.keys(fruits)); // ['0', '1']
console.log(fruits.length); // 2
```

**from**

`Array.from()` 方法从一个类似数组或可迭代对象中创建一个新的数组实例。

```
Array.from(arrayLike[, mapFn[, thisArg]])
```

- `arrayLike`

  想要转换成数组的伪数组对象或可迭代对象。

- `mapFn (可选参数) `

  如果指定了该参数，新数组中的每个元素会执行该回调函数。

- `thisArg (可选参数)`

  可选参数，执行回调函数 `mapFn` 时 `this` 对象。

`Array.from()` 可以通过以下方式来创建数组对象：

- 伪数组对象（拥有一个 `length` 属性和若干索引属性的任意对象）
- [可迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/iterable)（可以获取对象中的元素,如 Map和 Set 等）

`Array.from()` 方法有一个可选参数 `mapFn`，让你可以在最后生成的数组上再执行一次 [`map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 方法后再返回。也就是说` Array.from(obj, mapFn, thisArg) `就相当于` Array.from(obj).map(mapFn, thisArg),` 除非创建的不是可用的中间数组。 这对一些数组的子类`,`如  [typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays) 来说很重要, 因为中间数组的值在调用 map() 时需要是适当的类型。

`from()` 的 `length` 属性为 1 ，即`Array.from.length = 1`。

```
//数组去重合并
function combine(){ 
    let arr = [].concat.apply([], arguments);  //没有去重复的新数组 
    return Array.from(new Set(arr));
} 

var m = [1, 2, 2], n = [2,3,3]; 
console.log(combine(m,n));                     // [1, 2, 3]
```

**entries**

返回一个新的**Array Iterator**对象，该对象包含数组中每个索引的键/值对

```
var arr = ["a", "b", "c"];
var iterator = arr.entries();
console.log(iterator);
/*Array Iterator {}
         __proto__:Array Iterator
         next:ƒ next()
         Symbol(Symbol.toStringTag):"Array Iterator"
         __proto__:Object
*/

console.log(iterator.next());

/*{value: Array(2), done: false}
          done:false
          value:(2) [0, "a"]
           __proto__: Object
*/
// iterator.next()返回一个对象，对于有元素的数组，
// 是next{ value: Array(2), done: false }；
// next.done 用于指示迭代器是否完成：在每次迭代时进行更新而且都是false，
// 直到迭代器结束done才是true。
// next.value是一个["key":"value"]的数组，是返回的迭代器中的元素值。

for (let e of iterator) {
    console.log(e);
}

// [0, "a"] 
// [1, "b"] 
// [2, "c"]
```

 **keys()** 方法返回一个包含数组中每个索引键的Array Iterator对象。

**values()** 方法返回一个新的 Array Iterator 对象，该对象包含数组每个索引的值

**every**
对数组中每一项进行给定函数，如果**每一项**都满足条件则返回true，否则返回false；

```
arr.every(callback[, thisArg])
```

- `callback`

  用来测试每个元素的函数，它可以接收三个参数：     

  - `element`   用于测试的当前值。   

  - `index`可选   用于测试的当前值的索引。

  - `array`可选   调用 `every` 的当前数组。    

- `thisArg`

  执行 `callback` 时使用的 `this` 值。

```
var numbers = [1,2,3,4,5,4,3,2,1]; 
var result = numbers.every((item,index,array) => {
	return item >2 ;
});
 alert(result ); //false
```

**fill**

用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。

```
arr.fill(value[, start[, end]])
```

- `value`

  用来填充数组元素的值。

- `start` 可选

  起始索引，默认值为0。

- `end` 可选

  终止索引，默认值为 `this.length`。

fill 方法接受三个参数 `value`, `start` 以及 `end`. `start` 和 `end` 参数是可选的, 其默认值分别为 `0` 和 `this` 对象的 `length `属性值。

如果 `start` 是个负数, 则开始索引会被自动计算成为 `length+start`, 其中 `length` 是 `this` 对象的 `length `属性值。如果 `end` 是个负数, 则结束索引会被自动计算成为 `length+end`。

`fill` 方法故意被设计成通用方法, 该方法不要求 `this` 是数组对象。

`fill` 方法是个可变方法, 它会改变调用它的 `this` 对象本身, 然后返回它, 而并不是返回一个副本。

当一个对象被传递给 **fill**方法的时候, 填充数组的是这个对象的引用。

```
[1, 2, 3].fill(4);               // [4, 4, 4]
[1, 2, 3].fill(4, 1);            // [1, 4, 4]
[1, 2, 3].fill(4, 1, 2);         // [1, 4, 3]
[1, 2, 3].fill(4, 1, 1);         // [1, 2, 3]
[1, 2, 3].fill(4, 3, 3);         // [1, 2, 3]
[1, 2, 3].fill(4, -3, -2);       // [4, 2, 3]
[1, 2, 3].fill(4, NaN, NaN);     // [1, 2, 3]
[1, 2, 3].fill(4, 3, 5);         // [1, 2, 3]
Array(3).fill(4);                // [4, 4, 4]
[].fill.call({ length: 3 }, 4);  // {0: 4, 1: 4, 2: 4, length: 3}

// Objects by reference.
var arr = Array(3).fill({}) // [{}, {}, {}];
arr[0].hi = "hi"; // [{ hi: "hi" }, { hi: "hi" }, { hi: "hi" }]
```

**some**
对数组中每一项进行给定函数，如果**任意一项**满足条件则返回true，否则返回false；

```
var numbers = [1,2,3,4,5,4,3,2,1];
var everyResult = numbers.some(function(item,index,array){
    return item>2;
}); 
alert(everyResult); //true

```

**flat**

`flat()` 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。

```
var newArray = arr.flat(depth)
```

- `depth` 可选

  指定要提取嵌套数组的结构深度，默认值为 1。

返回一个包含将数组与子数组中所有元素的新数组。

```
var arr1 = [1, 2, [3, 4]];
arr1.flat(); 
// [1, 2, 3, 4]

var arr2 = [1, 2, [3, 4, [5, 6]]];
arr2.flat();
// [1, 2, 3, 4, [5, 6]]

var arr3 = [1, 2, [3, 4, [5, 6]]];
arr3.flat(2);
// [1, 2, 3, 4, 5, 6]

//使用 Infinity 作为深度，展开任意深度的嵌套数组
arr3.flat(Infinity); 
// [1, 2, 3, 4, 5, 6]

flat() 方法会移除数组中的空项:
var arr4 = [1, 2, , 4, 5];
arr4.flat();
// [1, 2, 4, 5]
```

**flatMap**

`flatMap()` 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。它与 [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 和 深度值1的 [flat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) 几乎相同，但 `flatMap` 通常在合并成一种方法的效率稍微高一些。

```
var new_array = arr.flatMap(function callback(currentValue[, index[, array]]) {
    // 返回新数组的元素
}[, thisArg])
```

- `callback`

  可以生成一个新数组中的元素的函数，可以传入三个参数：        

  - `currentValue`   当前正在数组中处理的元素   

  - `index`可选   可选的。数组中正在处理的当前元素的索引。 

  - `array`可选   可选的。被调用的 `map` 数组    

- `thisArg`可选

  可选的。执行 `callback` 函数时 使用的`this` 值。

返回 一个新的数组，其中每个元素都是回调函数的结果，并且结构深度 `depth` 值为1。

```
var arr1 = [1, 2, 3, 4];

arr1.map(x => [x * 2]); 
// [[2], [4], [6], [8]]

arr1.flatMap(x => [x * 2]);
// [2, 4, 6, 8]
// 等价于
arr1.reduce((acc, x) => acc.concat([x * 2]), []);
// [2, 4, 6, 8]

// 只会将 flatMap 中的函数返回的数组 “压平” 一层
arr1.flatMap(x => [[x * 2]]);
// [[2], [4], [6], [8]]

let arr = ["今天天气不错", "", "早上好"]

arr.map(s => s.split(""))
// [["今", "天", "天", "气", "不", "错"],[""],["早", "上", "好"]]

arr.flatMap(s => s.split(''));
// ["今", "天", "天", "气", "不", "错", "", "早", "上", "好"]
```

**filter**
对数组中每一项进行给定函数判断，返回满足函数条件的项组成的数组

```
var newArray = arr.filter(callback(element[, index[, array]])[, thisArg])
```

- `callback`

  用来测试数组的每个元素的函数。返回 `true` 表示该元素通过测试，保留该元素，`false` 则不保留。它接受以下三个参数：

  -  `element`   数组中当前正在处理的元素。  

  -  `index`可选   正在处理的元素在数组中的索引。  

  -  `array`可选   调用了 `filter` 的数组本身。    

- `thisArg`可选

  执行 `callback` 时，用于 `this` 的值。

返回一个新的、由通过测试的元素组成的数组，如果没有任何数组元素通过测试，则返回空数组。

`filter` 遍历的元素范围在第一次调用 `callback` 之前就已经确定了。在调用 `filter` 之后被添加到数组中的元素不会被 `filter` 遍历到。如果已经存在的元素被改变了，则他们传入 `callback` 的值是 `filter` 遍历到它们那一刻的值。被删除或从来未被赋值的元素不会被遍历到。

```
var numbers = [1,2,3,4,5,4,3,2,1];
var everyResult = numbers.filter(function(item,index,array){
    return item>2;
}); 
alert(everyResult); //  [3,4,5,4,3]

var arr = [
  { id: 15 },
  { id: -1 },
  { id: 0 },
  { id: 3 },
  { id: 12.2 },
  { },
  { id: null },
  { id: NaN },
  { id: 'undefined' }
];

var invalidEntries = 0;

function isNumber(obj) {
  return obj !== undefined && typeof(obj) === 'number' && !isNaN(obj);
}

function filterByID(item) {
  if (isNumber(item.id) && item.id !== 0) {
    return true;
  } 
  invalidEntries++;
  return false; 
}

var arrByID = arr.filter(filterByID);

console.log('Filtered Array\n', arrByID); 
// Filtered Array
// [{ id: 15 }, { id: -1 }, { id: 3 }, { id: 12.2 }]

console.log('Number of Invalid Entries = ', invalidEntries); 
// Number of Invalid Entries = 5
```

**map**
对数组中每一项进行给定函数，返回执行后的结果组成的数组

```
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
 // Return element for new_array 
}[, thisArg])
```

- `callback`

  生成新数组元素的函数，使用三个参数：   

  - `currentValue`   `callback` 数组中正在处理的当前元素。   

  - `index`可选   `callback` 数组中正在处理的当前元素的索引。

  - `array`可选   `callback`  `map` 方法被调用的数组。    

- `thisArg`可选

  执行 `callback` 函数时使用的`this` 值。

```
var numbers = [1,2,3,4,5,4,3,2,1]; 
var everyResult = numbers.map(function(item,index,array){
    return item*2;
}); 
alert(everyResult); //  [2, 4, 6, 8, 10, 8, 6, 4, 2]

```

**forEach**
遍历数组，类似for循环

- `callback`

  为数组中每个元素执行的函数，该函数接收三个参数：    

  -  `currentValue`   数组中正在处理的当前元素。  

  - `index`可选   数组中正在处理的当前元素的索引。   

  - `array`可选   `forEach()` 方法正在操作的数组。    

- `thisArg`可选

  可选参数。当执行回调函数时用作 `this` 的值(参考对象)。

返回undefined。

```
var numbers = [1,2,3,4,5,4,3,2,1]; 
numbers.forEach(function(item,index,array){ 
	if(item!=2){ //如果不是2
		numbers.splice(index,1,2); //替换成2
	} 
}); 
alert(numbers); //  [2, 2, 2, 2, 2, 2, 2, 2, 2]

```

**注意：** 没有办法中止或者跳出 `forEach()` 循环，除了抛出一个异常。如果你需要这样，使用 `forEach()` 方法是错误的。

若你需要提前终止循环，你可以使用：

- 简单循环
- [for...of](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of) 循环
- [`Array.prototype.every()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
- [`Array.prototype.some()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
- [`Array.prototype.find()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
- [`Array.prototype.findIndex()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)

这些数组方法可以对数组元素判断，以便确定是否需要继续遍历：[`every()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every)，[`some()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)，[`find()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find)，[`findIndex()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)

若条件允许，也可以使用 [`filter()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) 提前过滤出需要遍历的部分，再用 `forEach()` 处理。

如果数组在迭代时被修改了，则其他元素会被跳过。

**reduce && reduceRight**
这两个方法迭代数组所有项，然后构建一个最终返回的值。

reducer 函数接收4个参数:

1. Accumulator (acc) (累计器)
2. Current Value (cur) (当前值)
3. Current Index (idx) (当前索引)
4. Source Array (src) (源数组)

reduce从左到右，reduceRight从右到左。

```
//语法 
arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])

var values = [1,2,3,4,5];
var sum = values.reduce(function(prev,cur,index,array){
    return prev + cur;
});
alert(sum); //15

```

**检测数组instanceof && array.isArray**

```
var values = [1,2,3];
if(values instanceof Array){
    //对数组进行某些操作
}
if(Array.isArray(values)){
    //对数组进行某些操作
}

// 下面的函数调用都返回 true
Array.isArray([]);
Array.isArray([1]);
Array.isArray(new Array());
// 鲜为人知的事实：其实 Array.prototype 也是一个数组。
Array.isArray(Array.prototype); 

// 下面的函数调用都返回 false
Array.isArray();
Array.isArray({});
Array.isArray(null);
Array.isArray(undefined);
Array.isArray(17);
Array.isArray('Array');
Array.isArray(true);
Array.isArray(false);
Array.isArray({ __proto__: Array.prototype });
```

**join**
将数组转换成字符串，且用分隔符分隔

```
var colors = [1,2,3];
alert(colors.join("|"));  // 1|2|3

```

**find**

返回数组中满足提供的测试函数的第一个元素的值。否则返回 undefined。

```
arr.find(callback[, thisArg])
```

- `callback`

  在数组每一项上执行的函数，接收 3 个参数：  

  -  `element`   当前遍历到的元素。

  -  `index`可选   当前遍历到的索引。 

  -  `array`可选   数组本身。    

- `thisArg`可选

  执行回调时用作`this` 的对象。

`find`方法不会改变数组。

在第一次调用 `callback`函数时会确定元素的索引范围，因此在 `find`方法开始执行之后添加到数组的新元素将不会被 `callback`函数访问到。如果数组中一个尚未被`callback`函数访问到的元素的值被`callback`函数所改变，那么当`callback`函数访问到它时，它的值是将是根据它在数组中的索引所访问到的当前值。被删除的元素仍旧会被访问到。

```
// Declare array with no element at index 2, 3 and 4
var a = [0,1,,,,5,6];

// Shows all indexes, not just those that have been assigned values
a.find(function(value, index) {
  console.log('Visited index ' + index + ' with value ' + value); 
});

// Shows all indexes, including deleted
a.find(function(value, index) {

  // Delete element 5 on first iteration
  if (index == 0) {
    console.log('Deleting a[5] with value ' + a[5]);
    delete a[5];  // 注：这里只是将a[5]设置为undefined，可以试试用a.pop()删除最后一项，依然会遍历到被删的那一项
  }
  // Element 5 is still visited even though deleted
  console.log('Visited index ' + index + ' with value ' + value); 
});
```

`findIndex()`方法返回数组中满足提供的测试函数的第一个元素的**索引**。否则返回-1。

**toString**
将数组转换成字符串

```
var color = [1,2,3];
console.log(color.toString());// 1,2,3
```

**valueOf**
将数组转换成字符串，返回数组值

`Array.of()` 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

 `Array.of()` 和 `Array` 构造函数之间的区别在于处理整数参数：`Array.of(7)` 创建一个具有单个元素 **7** 的数组，而 **Array(7)** 创建一个长度为7的空数组（**注意：**这是指一个有7个空位(empty)的数组，而不是由7个`undefined`组成的数组）

```
var color = [1,2,3,4];
console.log(color.valueOf());//1,2,3,4

Array.of(7);       // [7] 
Array.of(1, 2, 3); // [1, 2, 3]

Array(7);          // [ , , , , , , ]
Array(1, 2, 3);    // [1, 2, 3]
```

**push && pop && shift && unshift**

```
push() 从数组末尾添加
pop()  从数组末尾移除
shift()    从数组前端移除
unshift()  从数组前端添加
```

**reverse**
反转数组，返回数组

```
var color = [1,2,3,4,5];
console.log(color.reverse());//[5, 4, 3, 2, 1]
console.log(Array.isArray(color.reverse()));//true
```

**sort**
排序返回数组，默认是升序，在排序时，sort()方法会调用每个数组项的 toString()转型方法，然后比较得到的字符串，以确定如何排序。即使数组中的每一项都是数值， sort()方法比较的也是字符串。

sort()方法可以接收一个比较函数作为参数，以便我们指定哪个值位于哪个值的前面。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个之后则返回一个正数。

```
var color = [1,2,3,4,5];
var color2 = color.sort((a,b) => {
	return b-a;	//降序
	//return a-b；升序
	//retun 是 负数；
});
console.log(color2);
// [5, 4, 3, 2, 1]

```

**concat**

`concat`方法创建一个新的数组，它由被调用的对象中的元素组成，每个参数的顺序依次是该参数的元素（如果参数是数组）或参数本身（如果参数不是数组）。它不会递归到嵌套数组参数中。

```
var new_array = old_array.concat(value1[, value2[, ...[, valueN]]])
```

`concat`方法不会改变`this`或任何作为参数提供的数组，而是返回一个浅拷贝，它包含与原始数组相结合的相同元素的副本。 原始数组的元素将复制到新数组中，如下所示：

- 对象引用（而不是实际对象）：`concat`将对象引用复制到新数组中。 原始数组和新数组都引用相同的对象。 也就是说，如果引用的对象被修改，则更改对于新数组和原始数组都是可见的。 这包括也是数组的数组参数的元素。

- 数据类型如字符串，数字和布尔（不是[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)，[`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 和 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean) 对象）：`concat`将字符串和数字的值复制到新数组中。

```
var values = [1,2,3]; 
var v1 = values.concat();
var v2 = values.concat(4); 
console.log(values); //[1,2,3];
console.log(v1); //[1,2,3] 
console.log(v2); //[1,2,3,4]
```

**注意：**数组/值在连接时保持不变。此外，对于新数组的任何操作（仅当元素不是对象引用时）都不会对原始数组产生影响，反之亦然。

**copyWithin**

浅复制数组的一部分到同一数组中的另一个位置，并返回它，不会改变原数组的长度。

```
arr.copyWithin(target[, start[, end]])
```

- `target`

  0 为基底的索引，复制序列到该位置。如果是负数，`target` 将从末尾开始计算。

  如果 `target` 大于等于 `arr.length`，将会不发生拷贝。如果 `target` 在 `start` 之后，复制的序列将被修改以符合 `arr.length`。

- `start`

  0 为基底的索引，开始复制元素的起始位置。如果是负数，`start` 将从末尾开始计算。

  如果 `start` 被忽略，`copyWithin` 将会从0开始复制。

- `end`

  0 为基底的索引，开始复制元素的结束位置。`copyWithin` 将会拷贝到该位置，但不包括 `end` 这个位置的元素。如果是负数， `end` 将从末尾开始计算。

  如果 `end` 被忽略，`copyWithin` 方法将会一直复制至数组结尾（默认为 `arr.length`）。

```
let numbers = [1, 2, 3, 4, 5];

numbers.copyWithin(-2);
// [1, 2, 3, 1, 2]

numbers.copyWithin(0, 3);
// [4, 5, 3, 4, 5]

numbers.copyWithin(0, 3, 4);
// [4, 2, 3, 4, 5]

numbers.copyWithin(-2, -3, -1);
// [1, 2, 3, 3, 4]

[].copyWithin.call({length: 5, 3: 1}, 0, 3);
// {0: 1, 3: 1, length: 5}

// ES2015 Typed Arrays are subclasses of Array
var i32a = new Int32Array([1, 2, 3, 4, 5]);

i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// On platforms that are not yet ES2015 compliant: 
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```

**splice**
splice() 方法用于插入、删除或替换数组的元素。

删除：可以删除任意数量的项，只需指定 2 个参数：要删除的第一项的位置和要删除的项数。例如， splice(0,2)会删除数组中的前两项。

插入：可以向指定位置插入任意数量的项，只需提供 3 个参数：起始位置、 0（要删除的项数）和要插入的项。例如，splice(2,0,4,6)会从当前数组的位置 2 开始插入4和6。
替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定  3 个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice  (2,1,4,6)会删除当前数组位置 2 的项，然后再从位置 2 开始插入4和6。

splice()方法始终都会返回一个数组，该数组中包含从原始数组中删除的项，如果没有删除任何项，则返回一个空数组。

```
array.splice(index,num,arr);
返回被删除的元素
/**
*index：修改的位置即下标数字
*num：删除的长度，默认是至结尾
*arr：添加进去的数组
**/
删除demo：
var values = [1,2,3,4,5,6];
var v = values.splice(0,2);
console.log(values);  //[3,4,5,6]
console.log(v);       //[1,2]

插入demo： 
var values = [1,2,3,4,5,6]; 
var v1 = values.splice(1,0,1,1,1); 
console.log(values); //[1,1,1,1,2,3,4,5,6] 
console.log(v1); //[] 

替换demo： 
var values = [1,2,3,4,5,6]; 
var v1 = values.splice(1,2,1,1,1); 
console.log(values); //[1,1,1,1,4,5,6] 
console.log(v1); //[2,3]
```

**slice**
用于复制或截取数组–>创建新数组，截取当前数组的一部分创建一个新数组。可以接受一个或者两个参数，只有一个参数时返回指定位置到尾部的数组。两个参数时，返回指定位置到结束位置之前但不包括结束位置的数组。

slice()方法可以接受一或两个参数，即要返回项的起始和结束位置。在只有一个参数的情况下， slice()方法返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项——但不包括结束位置的项。

```
var values = [1,2,3]; 
var v1 = values.slice(); 
var v2 = values.slice(1); 
var v3 = values.slice(1,3); 
console.log(values); //[1,2,3] 
console.log(v1); //[1,2,3] 
console.log(v2); //[2,3] 
console.log(v3); //[2,3]
```

**indexOf && lastIndexOf**
返回元素在数组的位置

```
arr.indexOf(searchElement)
arr.indexOf(searchElement[, fromIndex = 0])
arr.lastIndexOf(searchElement[, fromIndex = arr.length - 1])
```

- `searchElement`

  要查找的元素

- `fromIndex`

  开始查找的位置。如果该索引值大于或等于数组长度，意味着不会在数组里查找，返回-1。如果参数中提供的索引值是一个负值，则将其作为数组末尾的一个抵消，即-1表示从最后一个元素开始查找，-2表示从倒数第二个元素开始查找  ，以此类推。  注意：如果参数中提供的索引值是一个负值，并不改变其查找顺序，查找顺序仍然是从前向后查询数组。如果抵消后的索引值仍小于0，则整个数组都将会被查询。其默认值为0.

```
var values = [1,2,3,4,5];
indexOf() 从头找指定项的位置
var v1 = values.indexOf(3);

lastIndexOf() 从后往前查位置
var v2 = values.lastIndexOf(3);

两者如果没查到都返回-1

判断一个元素是否在数组里，不在则更新数组
function updateVegetablesCollection (veggies, veggie) {
    if (veggies.indexOf(veggie) === -1) {
        veggies.push(veggie);
        console.log('New veggies collection is : ' + veggies);
    } else if (veggies.indexOf(veggie) > -1) {
        console.log(veggie + ' already exists in the veggies collection.');
    }
}

var veggies = ['potato', 'tomato', 'chillies', 'green-pepper'];

// New veggies collection is : potato,tomato,chillies,green-papper,spinach
updateVegetablesCollection(veggies, 'spinach'); 
// spinach already exists in the veggies collection.
updateVegetablesCollection(veggies, 'spinach');
```

**includes**

判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回false。

```
arr.includes(valueToFind[, fromIndex])
```

- `valueToFind`

    需要查找的元素值。     

  **Note:**  使用 `includes()`比较字符串和字符时是区分大小写。    

- `fromIndex` 可选

  从`fromIndex` 索引处开始查找 `valueToFind`。如果为负值，则按升序从 `array.length + fromIndex` 的索引开始搜 （即使从末尾开始往前跳 `fromIndex` 的绝对值个索引，然后往后搜寻）。默认为 0。

返回一个布尔值 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean) ，如果在数组中找到了（如果传入了 `fromIndex` ，表示在 `fromIndex` 指定的索引范围中找到了）则返回 `true` 。

如果 `fromIndex` 大于等于数组的长度，则会返回 `false`，且该数组不会被搜索。

如果 `fromIndex `为负值，计算出的索引将作为开始搜索`searchElement`的位置。如果计算出的索引小于 0，则整个数组都会被搜索。

`includes()` 方法有意设计为通用方法。它不要求`this`值是数组对象，所以它可以被用于其他类型的对象 (比如类数组对象)。

```
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
[1, 2, NaN].includes(NaN); // true

var arr = ['a', 'b', 'c'];

arr.includes('c', 3);   // false
arr.includes('c', 100); // false

// array length is 3
// fromIndex is -100
// computed index is 3 + (-100) = -97

var arr = ['a', 'b', 'c'];

arr.includes('a', -100); // true
arr.includes('b', -100); // true
arr.includes('c', -100); // true
arr.includes('a', -2); // false

(function() {
  console.log([].includes.call(arguments, 'a')); // true
  console.log([].includes.call(arguments, 'd')); // false
})('a','b','c');
```



### Map

Map映射是ES6里面新增的一个对象，是一组键值对的结构，具有极快的查找速度。

定义  键/值对的集合。

集合中的键和值可以是任何类型。如果使用现有密钥向集合添加值，则新值会替换旧值

```
mapObj = new Map()
```

#### 属性

下表列出了 Map 对象的属性和描述

```
构造函数
    指定创建映射的函数。
Prototype — 原型
    为映射返回对原型的引用。
size
    返回映射中的元素数。
```

#### 方法

下表列出了 Map 对象的方法和描述。

```
clear
    从映射中移除所有元素。
delete
    从映射中移除指定的元素。
forEach
    对映射中的每个元素执行指定操作。
get
    返回映射中的指定元素。
has
    如果映射包含指定元素，则返回 true。
set
    添加一个新建元素到映射。
toString
    返回映射的字符串表示形式。
valueOf
    返回指定对象的原始值。
```

下面的示例演示如何将成员添加到 Map，然后检索它们。

```javascript
    // 初始化Map需要一个二维数组，或者直接初始化一个空Map
    var m1 = new Map([['a', 'a1'], ['b', 'b2'], ['c', 'c3']]);
    var m11 = new Map([['a', 'a1'], ['b', 'b2'], ['c', 'c3']]);
    var m2 = new Map();
	
    console.log(m1);			// 返回Map  {"a" => "a1", "b" => "b2", "c" => "c3"}
    console.log(typeof(m1));	// object, Map仍属于 object
    console.log(m1 == m11)		// flase  虽然两个Map里面的值一样，但是是属于不同的object

    // 1. size属性，返回 Map的元素数
    console.log(m1.size);		// 3

    // 2. keys()	获取Map的所有key
    console.log(m1.keys());		// 返回 MapIterator {"a", "b", "c"}

    // 3. values()	获取Map的所有value
    console.log(m1.values());	// 返回 MapIterator {"a1", "b2", "c3"}

    // 4. entries()	获取Map所有成员  
    console.log(m1.entries());	// 返回 MapIterator {"a" => "a1", "b" => "b2", "c" => "c3"}

    // 5. forEach()	循环操作映射元素
    m1.forEach(function(value, key, map) {
  	  // value:  key对应的值，  
	  // key: Map的key，(map参数已省略情况下，key可省略)
	  // map:  Map本身，(该参数是可省略参数)
	  console.log(value);			// key对应的值   a1  b2  c3
	  console.log(key);			// key 			a   b   c
	  console.log(map);			// Map本身      Map Map Map
    });

    // 6. set()		给Map添加数据，  返回添加后的Map
    console.log(m2.set('a', 1));	// 返回Map  {"a" => 1}
    console.log(m2.set('b', 2));	// {"a" => 1, "b" => 2}
    console.log(m2.set('a', 11));	// {"a" => 11, "b" => 2} 给已存在的键赋值会覆盖掉之前的值，  

    // 7. has()		检测是否存在某个key， 返回布尔值，有：true； 没有：false
    console.log(m2.has('a'));		// true
    console.log(m2.has('c'));		// false

    // 8. get()		获取某个key的值，返回key对应的值，没有则返回undefined	
    console.log(m2.get('a')); 		// 11
    console.log(m2.get('c'));		// undefined

    // 9. delete()	删除某个key及其对应的value，返回布尔值，成功：true； 失败：false
    console.log(m2.delete('b'));	// true
    console.log(m2.delete('d'));	// false
	
    console.log(m2.get('b'));		// undefined， 因为b已经删除

    // 10. clear()	清除所有的值，返回 undefined
    console.log(m1.clear());		// undefined
    console.log(m1);				// {} 

```

#### 封装类 map.js

```javascript
Array.prototype.remove = function(s) {     
    for (var i = 0; i < this.length; i++) {     
        if (s == this[i])     
            this.splice(i, 1);     
    }     
}     
    
/**   
 * Simple Map   
 *    
 *    
 * var m = new Map();   
 * m.put('key','value');   
 * ...   
 * var s = "";   
 * m.each(function(key,value,index){   
 *      s += index+":"+ key+"="+value+"/n";   
 * });   
 * alert(s);   
 *    
 * @author dewitt   
 * @date 2008-05-24   
 */    
function Map() {     
    /** 存放键的数组(遍历用到) */    
    this.keys = new Array();     
    /** 存放数据 */    
    this.data = new Object();     
         
    /**   
     * 放入一个键值对   
     * @param {String} key   
     * @param {Object} value   
     */    
    this.put = function(key, value) {     
        if(this.data[key] == null){     
            this.keys.push(key);     
        }     
        this.data[key] = value;     
    };     
         
    /**   
     * 获取某键对应的值   
     * @param {String} key   
     * @return {Object} value   
     */    
    this.get = function(key) {     
        return this.data[key];     
    };     
         
    /**   
     * 删除一个键值对   
     * @param {String} key   
     */    
    this.remove = function(key) {     
        this.keys.remove(key);     
        this.data[key] = null;     
    };     
         
    /**   
     * 遍历Map,执行处理函数   
     *    
     * @param {Function} 回调函数 function(key,value,index){..}   
     */    
    this.each = function(fn){     
        if(typeof fn != 'function'){     
            return;     
        }     
        var len = this.keys.length;     
        for(var i=0;i<len;i++){     
            var k = this.keys[i];     
            fn(k,this.data[k],i);     
        }     
    };     
         
    /**   
     * 获取键值数组(类似Java的entrySet())   
     * @return 键值对象{key,value}的数组   
     */    
    this.entrys = function() {     
        var len = this.keys.length;     
        var entrys = new Array(len);     
        for (var i = 0; i < len; i++) {     
            entrys[i] = {     
                key : this.keys[i],     
                value : this.data[i]     
            };     
        }     
        return entrys;     
    };     
         
    /**   
     * 判断Map是否为空   
     */    
    this.isEmpty = function() {     
        return this.keys.length == 0;     
    };     
         
    /**   
     * 获取键值对数量   
     */    
    this.size = function(){     
        return this.keys.length;     
    };     
         
    /**   
     * 重写toString    
     */    
    this.toString = function(){     
        var s = "{";     
        for(var i=0;i<this.keys.length;i++,s+=','){     
            var k = this.keys[i];     
            s += k+"="+this.data[k];     
        }     
        s+="}";     
        return s;     
    };     
}     
    
    
function testMap(){     
    var m = new Map();     
    m.put('key1','Comtop');     
    m.put('key2','南方电网');     
    m.put('key3','景新花园');     
    alert("init:"+m);     
         
    m.put('key1','康拓普');     
    alert("set key1:"+m);     
         
    m.remove("key2");     
    alert("remove key2: "+m);     
         
    var s ="";     
    m.each(function(key,value,index){     
        s += index+":"+ key+"="+value+"/n";     
    });     
    alert(s);     
}    
```

### Set

Set也是ES6新增的对象，Set是一组key的集合，但不存储value, 而且key不重复，可自动排重

语法：new Set([iterable])
参数：
iterable 如果传递一个可迭代对象，它的所有元素将被添加到新的 Set中；如果不指定此参数或其值为null，则新的 Set为空

```
    let arr = [1,2,2,3];
    let mySet = new Set(arr);
    console.log(mySet); // Set(3) {1, 2, 3}
```

size属性将会返回Set对象中元素的个数

```javascript
    let mySet = new Set();
    mySet.add(1);
    mySet.add(5);
    mySet.add("some text");
    console.log(mySet.size); // 3
```

Set实例方法

```javascript
    // 初始化Map需要提供一个Array作为输入，或者直接创建一个空Set
    var s1 = new Set(['a', 'b', 'c']);
    var s11 = new Set(['a', 'b', 'c']);
    var s2 = new Set(['a', 'a', 'b', 'b', 'c', 'c']);
    var s3 = new Set();
	
    console.log(s1);				// 返回 Set(3) {"a", "b", "c"}
    console.log(s2);				// 返回 Set(3) {"a", "b", "c"}
    console.log(typeof(s1));		// object
    console.log(s1 == s11);		// false
    console.log(s1 == s2);		// false
	
    // 1. size属性  返回Set的元素数
    console.log(s1.size);			// 3
	
    // 2. keys() 获取Set的所有key	
    console.log(s1.keys());		// 返回 SetIterator {"a", "b", "c"}
	
    // 3. values()  获取Set的值，返回结果和 keys()一样
    console.log(s1.values());		// 返回 SetIterator {"a", "b", "c"}
	
    // 4. entries() 获取Set所有成员，返回同keys()
    console.log(s1.entries());	// 返回 SetIterator {"a", "b", "c"}
	
    // 5. forEach() 循环操作集合元素	
    s1.forEach(function(v, k, s){	// v、k是集合的键，s是集合本身
	  console.log(v);				//  a   b   c
	  console.log(k);				//  a   b   c
	  console.log(s);				// Set Set Set
    });
    // 6. add()   给集合添加数据,末尾添加一个指定的值	返回添加后的Set
    console.log(s3.add('aa'));		// Set(1) {"aa"}
    console.log(s3.add('bb'));		// Set(2) {"aa", "bb"}
    console.log(s3.add('aa'));		// Set(2) {"aa", "bb"}	添加重复的值，会被排重掉，
	
    // 7. has() 查询集合中是否包含某个元素  返回布尔值 有：true； 没有：false
    console.log(s3.has('aa'));		// true
    console.log(s3.has('ff'));		// false	

    // 8. delete() 删除集合中的某个元素,删除指定的元素  返回布尔值
    console.log(s3.delete('aa'));	// true
    console.log(s3.delete('ee'));	// false

    console.log(s3);				// Set(1) {"bb"}
	
    // 9. clear()  清除集合的所有值	返回undefined
    console.log(s1.clear());		// undefined
	
    console.log(s1);				// Set(0) {}
```

#### 数组去重之利用set数据结构去重

```javascript
function dedupe(array){
 return Array.from(new Set(array));
}
dedupe([1,1,2,3]) //[1,2,3]
```

解释： 

1. 先新建个dedupe函数，传入数据是数组 
2. 传入的数组通过new set()转化为set数据格式，此时就已经把重复值给去掉了。 
3. 通过Array.form方法，把set数据结构转换为数组即可。

### String

**1. charCodeAt方法返回一个整数，代表指定位置字符的Unicode编码。** strObj.charCodeAt(index) 

说明： 

index将被处理字符的从零开始计数的编号。有效值为0到字符串长度减1的数字。 
如果指定位置没有字符，将返回NaN。 

例如： 

```
var str = "ABC"; 
str.charCodeAt(0); 
```

结果：65 
**2. fromCharCode方法从一些Unicode字符串中返回一个字符串。** 
`String.fromCharCode([code1[,code2...]]) `

说明： 
code1，code2...是要转换为字符串的Unicode字符串序列。如果没有参数，结果为空字符串。 

例如： 

```
String.fromCharCode(65,66,112); 
```

结果：ABp 

**3.charAt方法返回指定索引位置处的字符。如果超出有效范围的索引值返回空字符串。** 

`strObj.charAt(index)` 

说明： 

index想得到的字符的基于零的索引。有效值是0与字符串长度减一之间的值。 

例如：

```
var str = "ABC"; 
str.charAt(1); 
```

结果：B 

**4. slice方法返回字符串的片段。** 

`strObj.slice(start[,end])` 

说明： 

start下标从0开始的strObj指定部分其实索引。如果start为负，将它作为length+start处理，此处length为字符串的长度。 

end小标从0开始的strObj指定部分结束索引。如果end为负，将它作为length+end处理，此处length为字符串的长度。 

例如：

```
 var str = "ABCDEF"; 
str.slice(2,4); 
```

结果：CD 

**5. substring方法返回位于String对象中指定位置的子字符串。** 

`strObj.substring(start,end)` 

说明：

start指明子字符串的起始位置，该索引从0开始起算。 
end指明子字符串的结束位置，该索引从0开始起算。 
substring方法使用start和end两者中的较小值作为子字符串的起始点。如果start或end为NaN或者为负数，那么将其替换为0。 

例如： 

```
var str = "ABCDEF"; 
str.substring(2,4); // 或 str.substring(4,2); 
```

结果：CD 

**6. substr方法返回一个从指定位置开始的指定长度的子字符串。** 

`strObj.substr(start[,length])` 

说明：


start所需的子字符串的起始位置。字符串中的第一个字符的索引为0。 
length在返回的子字符串中应包括的字符个数。 

例如： 

```
var str = "ABCDEF"; 
str.substr(2,4); 
```

结果：CDEF 

**7. indexOf方法放回String对象内第一次出现子字符串位置。如果没有找到子字符串，则返回-1。** 

`strObj.indexOf(substr[,startIndex])` 

说明： 

substr要在String对象中查找的子字符串。 
startIndex该整数值指出在String对象内开始查找的索引。如果省略，则从字符串的开始处查找。 

例如：

```
var str = "ABCDECDF"; 
str.indexOf("CD"，1); // 由1位置从左向右查找 123... 
```

结果：2 

**8. lastIndexOf方法返回String对象中字符串最后出现的位置。如果没有匹配到子字符串，则返回-1。** 

`strObj.lastIndexOf(substr[,startindex])` 

说明： 

substr要在String对象内查找的子字符串。 
startindex该整数值指出在String对象内进行查找的开始索引位置。如果省略，则查找从字符串的末尾开始。 

例如： 

```
var str = "ABCDECDF"; 
str.lastIndexOf("CD",6); // 由6位置从右向左查找 ...456 
```

结果：5 

**9. search方法返回与正则表达式查找内容匹配的第一个字符串的位置。** 

`strObj.search(reExp)` 

说明：


reExp包含正则表达式模式和可用标志的正则表达式对象。 

例如： 

```
var str = "ABCDECDF"; 
str.search("CD"); // 或 str.search(/CD/i); 
```

结果：2 

**10. concat方法返回字符串值，该值包含了两个或多个提供的字符串的连接。** 

`str.concat([string1[,string2...]])` 

说明： 

string1，string2要和所有其他指定的字符串进行连接的String对象或文字。 

例如： 

```
var str = "ABCDEF"; 
str.concat("ABCDEF","ABC"); 
```

结果：ABCDEFABCDEFABC 

**11. 将一个字符串分割为子字符串，然后将结果作为字符串数组返回。** 

`strObj.split([separator[,limit]])` 

说明： 

separator字符串或 正则表达式 对象，它标识了分隔字符串时使用的是一个还是多个字符。如果忽略该选项，返回包含整个字符串的单一元素数组。 
limit该值用来限制返回数组中的元素个数。 

例如： 

```
var str = "AA BB CC DD EE FF"; 
alert(str.split(" "，3)); 
```

结果： AA,BB,CC 

**12. toLowerCase方法返回一个字符串，该字符串中的字母被转换成小写。** 

例如： 

```
var str = "ABCabc"; 
str.toLowerCase(); 
```

结果：abcabc 

**13. toUpperCase方法返回一个字符串，该字符串中的所有字母都被转换为大写字母。** 

例如： 

```
var str = "ABCabc"; 
str.toUpperCase(); 
```

结果：ABCABC

