---
title: react-redux简单使用
urlname: ew4pp8
date: '2020-09-08 18:59:45 +0800'
tags: []
categories: []
---

## 1  注入 store。在 index.js

```javascript
import { Provider } from "react-redux";
import store from "./store";

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

## 2 配置 store.js

```javascript
import { createStore, combineReducers, applyMiddleware } from "redux";
import thunk from "redux-thunk";

let dafaultState = {
  num: "1",
};

let home = (state = dafaultState, action) => {
  switch (action.type) {
    case "add":
      return { ...state, ...{ num: action.data } };
    default:
      return state;
  }
};

let reducers = combineReducers({
  home,
});

let store = createStore(reducers, applyMiddleware(thunk));

export default store;
```

## 3  配置 action

```javascript
export const addAction = () => {
  return {
    type: "add",
    data: "2",
  };
};
```

## 4  配置组件

```javascript
import React, { Component } from "react";
import { connect } from "react-redux";
import { addAction } from "../actions";

class Test extends Component {
  render() {
    console.log(this.props);
    return (
      <div>
        <h1>{this.props.num}</h1>
        <button onClick={this.props.addAction}>按钮</button>
      </div>
    );
  }
}

const mapStateToProps = (state, ownProps) => ({
  num: state.home.num,
});

const mapDispatchToProps = {
  addAction,
};

export default connect(mapStateToProps, mapDispatchToProps)(Test);
```
