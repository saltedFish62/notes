# SuperTest

github地址：[github](https://github.com/visionmedia/supertest)

该模块提供了测试HTTP的高层次抽象，同时也向下兼容superagent提供的低层次API。

## 开始

```js
npm install supertest --save-dev
```

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

