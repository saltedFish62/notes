# SuperTest

github地址：[github](https://github.com/visionmedia/supertest)

该模块提供了测试HTTP的高层次抽象，同时也向下兼容superagent提供的低层次API。

## 开始

```js
npm install supertest --save-dev
```

## 简单用法拆解

​	首先，需要有一个express应用。

```js
const app = express()
```

​	然后，引用SuperTest。

```js
const request = require('supertest')
```

### get()

​	假设app中有一个api是 `GET /user `，要测试该api，则：

```js
request(app)
  .get('/user')
	.end(function(err, res) {
  	if (err) throw err
})
```

​	以上模拟了一个对 `/user`的请求。如果要加入对于该请求的返回数据的验证，则使用 `.expect()`。

### expect()

```js
request(app)
  .get('/user')
	.expect('Content-Type', /json/)
  .expect('Content-Length', '15')
  .expect(200)
  .end(function(err, res) {
    if (err) throw err;
  })
```

​	expect()有如下用法：

|            用法             |                            解释                            |
| :-------------------------: | :--------------------------------------------------------: |
|    .expect(status[, fn])    |           断言返回值的status，可选传入回调函数fn           |
| .expect(status, body[, fn]) |        断言返回值的status和body，可选传入回调函数fn        |
|     .expect(body[, fn])     | 以字符串类型、正则表达式或已解析的对象体来断言返回值的body |
| .expect(field, value[, fn]) |          用字符或正则表达式断言header中的字段和值          |
|  .expect(function(res){})   |         传递一个定制的断言函数，通过抛出异常来报错         |

### end()

​	.end(fn)

​	执行请求，在请求结束的时候触发fn(err, res)

## 例子

​	你可以传递一个 `http.server` 或者一个 `Function` 给 `request()`。如果服务器还没有监听，那么它会被绑定到一个临时端口，所以也不需要跟踪。

​	SuperTest兼容任何测试框架，也可以不用测试框架。

- 没有使用测试框架的例子：

```js
const request = require('supertest');
const express = require('express');

const app = express();

app.get('/user', function(req, res) {
  res.status(200).json({ name: 'john' });
});

request(app)
  .get('/user')
  .expect('Content-Type', /json/)
  .expect('Content-Length', '15')
  .expect(200)
  .end(function(err, res) {
    if (err) throw err;
  });
```

- 使用mocha的例子：

```js
describe('GET /user', function() {
  it('responds with json', function(done) {
    request(app)
      .get('/user')
      .set('Accept', 'application/json')
      .expect('Content-Type', /json/)
      .expect(200, done);
  });
});
```

​	注意到`it()`的回调函数有传值 `done`。该值是mocha框架的机制，用于检测异步的代码。

- 如果没有调用 `done` ，mocha会等待直到超时。

- 如果调用了 `done` 而没有传递参数，mocha会直接返回成功，而不捕获到异步的断言失败。