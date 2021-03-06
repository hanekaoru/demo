
Node.js 文档学习笔记

基于 @6.9.5

----

## Buffer.from(), Buffer.alloc(), Buffer.allocUnsafe()

为了使 ```Buffer``` 实例的创建更可靠、更不容易出错，各种 ```new Buffer()``` 构造函数已被**废弃**，并由```Buffer.from()```、```Buffer.alloc()```、和 ```Buffer.allocUnsafe()``` 方法替代

* ```Buffer.from(array)``` 返回一个新建的包含所提供的字节数组的副本的 ```Buffer```

* ```Buffer.from(arrayBuffer[, byteOffset [, length]])``` 返回一个新建的与给定的 ```ArrayBuffer``` 共享同一内存的 ```Buffer```

* ```Buffer.from(buffer)``` 返回一个新建的包含所提供的 ```Buffer``` 的内容的副本的 ```Buffer```

* ```Buffer.from(string[, encoding])``` 返回一个新建的包含所提供的字符串的副本的 ```Buffer```

* ```Buffer.alloc(size[, fill[, encoding]])``` 返回一个指定大小的被填满的 ```Buffer``` 实例。 这个方法会明显地比 ```Buffer.allocUnsafe(size)``` 慢，但可确保新创建的 ```Buffer``` 实例绝不会包含旧的和潜在的敏感数据

```Buffer.allocUnsafe(size)``` 与 Buffer.allocUnsafeSlow(size) 返回一个新建的指定 ```size``` 的 ```Buffer```，但它的内容必须被初始化，可以使用 ```buf.fill(0)``` 或完全写满


## Buffer 与 ES6 迭代器

Buffer 实例可以使用 ES6 的 for..of 语法进行遍历

```js
const buf = Buffer.from([1, 2, 3]);

for (var b of buf) {
    console.log(b);
}
// 输出:
//   1
//   2
//   3
```



## 类方法：Buffer.alloc(size[, fill[, encoding]])

分配一个大小为 ```size``` 字节的新建的 ```Buffer``` 。 如果 ```fill``` 为 ```undefined``` ，则该 ```Buffer``` 会用 ```0``` 填充

```js
const buf = Buffer.alloc(5);

console.log(buf);
// 输出: <Buffer 00 00 00 00 00>
```

```size``` 必须小于或等于 ```buffer.kMaxLength```（分配给单个 ```Buffer``` 实例的最大内存） 的值，否则会抛出错误。 如果 ```size``` 小于或等于 ```0```，则创建一个长度为 ```0``` 的 ```Buffer```

```js
const buf = Buffer.alloc(5, 'a');

// 输出: <Buffer 61 61 61 61 61>
console.log(buf);
```

如果同时指定了 ```fill``` 和 ```encoding``` ，则会调用 ```buf.fill(fill, encoding)``` 初始化分配的 ```Buffer``` 

```js
const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

// 输出: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
console.log(buf);
```

调用 ```Buffer.alloc()``` 会明显地比另一个方法 ```Buffer.allocUnsafe()``` 慢，但是能确保新建的 ```Buffer``` 实例的内容不会包含敏感数据。

需要注意的是：如果 ```size``` 不是一个数值，则抛出错误


## 类方法：Buffer.allocUnsafe(size)

同 ```Buffer.alloc``` 类似，不同之处在于，以这种方式创建的 ```Buffer``` 实例的底层内存是未初始化的，内容是未知的，且可能包含敏感数据

## 类方法：Buffer.concat(list[, totalLength])

返回一个合并了 ```list``` 中所有 Buffer 实例的新建的 Buffer 

如果 ```list``` 中没有元素、或 ```totalLength``` 为 ```0``` ，则返回一个新建的长度为 ```0``` 的 ```Buffer```

## 类方法：Buffer.from(array)

通过一个八位字节的 ```array``` 创建一个新的 Buffer 

如果 ```array``` 不是一个数组，则抛出 ```TypeError``` 错误

## 类方法：Buffer.isBuffer(obj)

如果 ```obj``` 是一个 ```Buffer``` 则返回 ```true``` ，否则返回 ```false```

## 类方法：Buffer.isEncoding(encoding)

如果 ```encoding``` 是一个支持的字符编码则返回 ```true```，否则返回 ```false``` 



## File System (文件系统)

通过 ```require('fs')``` 使用该模块。 所有的方法都有异步和同步的形式。

异步形式始终以完成回调作为它最后一个参数

传给完成回调的参数取决于具体方法，但第一个参数总是留给异常。 如果操作成功完成，则第一个参数会是 ```null``` 或 ```undefined```

当使用同步形式时，任何异常都会被立即抛出。 可以使用 ```try/catch``` 来处理异常，或让它们往上冒泡。

异步版本的例子

```js
const fs = require("fs");

fs.unlink("/tmp/hello", (err) => {
    if (err) throw err;
    console.log("successfully deleted /tmp/hello");
})
```

同步版本的例子

```js
const fs = require("fs");

fs.unlinkSync("/tmp/hello");
console.log("successfully deleted /tmp/hello");
```

异步方法不保证执行顺序。 所以下面的例子容易出错：

```js
fs.rename('/tmp/hello', '/tmp/world', (err) => {
    if (err) throw err;
    console.log('renamed complete');
});
fs.stat('/tmp/world', (err, stats) => {
    if (err) throw err;
    console.log(`stats: ${JSON.stringify(stats)}`);
});
```

正确的方法是把回调链起来

```js
fs.rename('/tmp/hello', '/tmp/world', (err) => {
    if (err) throw err;
    fs.stat('/tmp/world', (err, stats) => {
        if (err) throw err;
        console.log(`stats: ${JSON.stringify(stats)}`);
    });
});
```
