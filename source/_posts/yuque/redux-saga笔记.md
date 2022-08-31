---
title: redux-saga笔记
urlname: qszcig
date: '2020-09-08 18:59:45 +0800'
tags: []
categories: []
---

### 基本搭建

在 store.js  中

```javascript
import { createStore, applyMiddleware } from "redux";
import createSagaMiddleware from "redux-saga";

import { helloSaga } from "./sagas";

const store = createStore(
  reducer,
  applyMiddleware(createSagaMiddleware(helloSaga))
);
```

第二种方式:

```javascript
import { createStore, applyMiddleware } from "redux";
import createSagaMiddleware from "redux-saga";

import { helloSaga } from "./sagas";

const sagaMiddleware = createSagaMiddleware();
const store = createStore(reducer, applyMiddleware(createSagaMiddleware()));
sagaMiddleware.run(helloSaga);
```

### 使用 saga

```javascript
import { takeEvery } from "redux-saga";
import { call, put } from "redux-saga/effects";

// 监听 FETCH_REQUESTED action 每一次触发 都会执行 fetchData
function* watchFetchData() {
  yield* takeEvery("FETCH_REQUESTED", fetchData);
}

export function* fetchData(action) {
  try {
    const data = yield call(Api.fetchUser, action.payload.url);
    yield put({ type: "FETCH_SUCCEEDED", data });
  } catch (error) {
    yield put({ type: "FETCH_FAILED", error });
  }
}

// call：函数调用
// select：获取Store中的数据
// put：向Store发送action
// take：监听未来的action
// takeEvery 是每次发起 监听的action的时候都会执行
// takeLatest 只监听最后那个action 调用时
// fork：函数是用来调用其他函数的，但是fork函数是非阻塞函数 也就是说，程序执行完 yield fork(fn， args) 这一行代码后，会立即接着执行下一行代码语句。
```
