# JavaScript验证输入的string的内容是不是数值

[TOC]

## JavaScript里面对Number的支持

[Number](https://developer.mozilla.org/en-US/docs
/Web/JavaScript/Reference/Global_Objects/Number)是JavaScript的一个内置的对象，有很多method可以直接使用。在使用JavaScript验证一个值是不是数值的问题中，我们可能用到的两个method是：

1. [`Number.parseInt()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/parseInt)
2. [`Number.parseFloat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/parseFloat)

以下是这两个method的几个简单的用法：

```javascript
// example from https://www.bennadel.com/blog/2012-exploring-javascript-s-parseint-and-parsefloat-functions.htm
// parseInt
Number.parseInt( "123" ); //123
Number.parseInt( "123.456" ); //123
Number.parseInt( "+123.456" ); //123
Number.parseInt( "-123.456" ); //-123
Number.parseInt( "123ABC" ); //123
Number.parseInt( "ABC" ); //NaN
Number.parseInt( "5px 5px" ); //5
Number.parseInt( "(123)" ); //NaN
Number.parseInt( "0xF" ); //15
Number.parseInt( "012" ); //10
Number.parseInt( " 123 " ); //123

// parseFloat
Number.parseFloat( "123" ); //123
Number.parseFloat( "123.456" ); //123.456
Number.parseFloat( "+123.456" ); //123.456
Number.parseFloat( "-123.456" ); //-123.456
Number.parseFloat( "123ABC" ); //123
Number.parseFloat( "ABC" ); //NaN
Number.parseFloat( "5px 5px" ); //5
Number.parseFloat( "(123)" ); //NaN
Number.parseFloat( "0xF" ); //0
Number.parseFloat( "012" ); //12
Number.parseFloat( " 123 " ); //123
```

从上面的例子可以看到，Number.parseInt()和Number.parseFloat()虽然可以把string转化成对就的数值，但是这个转化过程不是exactly的转化，所以如果我们在验证中使用会造成很大的迷惑。

## 考虑直接使用第三方库

函数式编程的流行，促使了很多优秀的第三方工具库的出现，这些大大地减少了我们在开发中的工作量，也一定程度上提高了我们代码的质量。在JavaScript中，我列举两个：

1. [Lodash](https://lodash.com/)
2. [Ramda](http://ramdajs.com/)

在Ramda中只有[`R.is`](http://ramdajs.com/docs/#is)来判断类型，没有想要的函数。

在lodash中有[`_.parseInt`](https://lodash.com/docs/4.17.4#parseInt)，使用与Number.parseInt基本相同。其中的[`_.isNumber`](https://lodash.com/docs/4.17.4#isNumber)也不能满足需求。

## 使用Regrex验证

最终,还是使用了Regrex解决了业务需求。以下是最后的Regrex:

**`/^0$|^[+-]?[1-9]\d*(\.\d+)?$/`**

```javascript
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("2"); //true
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("+2"); //true
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("-2"); //true
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("2.1"); //true
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("+2.1"); //true
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("-2.1"); //true
/^0$|^[+-]?[1-9]\d*(\.\d+)?$/.test("abc"); //false
```
