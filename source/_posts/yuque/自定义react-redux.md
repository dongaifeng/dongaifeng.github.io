---
title: 自定义react-redux
urlname: oh50ry
date: '2020-09-08 18:59:45 +0800'
tags: []
categories: []
---

1. 用户通过 connect 传入两个参数  mapStateToProps， mapDispatchToProps
1. mapStateToProps 是一个函数   在 connect 中被调用   返回组件要用到的 state
1. mapDispatchToProps  返回一个对象   在 connect 中会被遍历   并加上 dispatch 方法
1. 在 connect 里  store 通过 useContext 获取到的由 createContext 产生的
1. 在 useEffect 里挂载  store.subscribe 方法   组件挂载，更新时会调用

```javascript
import React, { useContext, useState, useEffect } from "react";

// 定义context 用于传递store
const ReduxContext = React.createContext();

export function Provider({ store, children }) {
  return (
    // children 组件复合
    <ReduxContext.Provider value={store}>{children}</ReduxContext.Provider>
  );
}

// 用户传入两个参数mapStateToProps， mapDispatchToProps
export const connect = function (
  mapStateToProps = (state) => state,
  mapDispatchToProps = {}
) {
  // 返回一个函数 处理用户传入的组件
  return (Cmp) => {
    // 返回的函数组件
    return (props) => {
      // 获取Provider传入的store
      const store = useContext(ReduxContext);

      // 处理用户传入的参数 返回真正的state，dispatch
      const getMoreProps = () => {
        // 这里对应 用户写的mapDispatchToProps
        const stateProps = mapStateToProps(store.getState());

        // mapDispatchToProps是用户传入的对象{add： {type: 'add}} {type: 'add}}是action 循环这个对象给add 便规定dispatch({type: 'add}})
        const dispatchToProps = bindActionCreators(
          mapDispatchToProps,
          store.dispatch
        );
        return {
          ...stateProps,
          ...dispatchToProps,
        };
      };

      const [moreProps, setMoreProps] = useState(getMoreProps());

      useEffect(() => {
        store.subscribe(() => {
          setMoreProps({
            ...moreProps,
            ...getMoreProps(),
          });
        });
      });
      return <Cmp {...props} {...moreProps} />;
    };
  };
};

function bindActionCreators(actionCreators, dispatch) {
  let obj = {};
  for (let key in actionCreators) {
    obj[key] = bindActionCreator(actionCreators[key], dispatch);
  }
  return obj;
}
// 给actionCreators所有的方法绑定上dispatch
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args));
}
```
