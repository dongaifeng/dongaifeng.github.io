---
title: antd-pro学习笔记
urlname: ers3sz
date: '2020-09-08 18:59:45 +0800'
tags: []
categories: []
---

#### 1 全局样式

```less
.card {
  margin-bottom: 24px;

  :global {
    .ant-legacy-form-item .ant-legacy-form-item-control-wrapper {
      width: 100%;
    }
  }
}

/* 定义全局样式 */
:global(.text) {
  font-size: 16px;
}
```

:globa 表示这是定义的全局样式。他被 .card 包裹表示 只有在这个 class 下才生效。

#### 2 dva-loading 的使用

```javascript
export default connect(
  // mapStateToProps ：(state) => ({ })
  ({ loading }: P) => ({
    submitting: loading.effects["gaojiForm/submit"],
  })
)(GaojiForm);
```

在异步请求是 loading 变量会为 true，完成后会变成 false，用来控制加载动画。
[参考链接](https://blog.csdn.net/sinat_42338962/article/details/98340667?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

#### 3 antd 的 Form

设置了  `name`  属性的  `Form.Item`  包装的控件，表单控件会自动添加  `value`（或  `valuePropName`  指定的其他属性） `onChange`（或  `trigger`  指定的其他属性），数据同步将被 Form 接管。
使用方式：
你不再需要也不应该用 onChange 来做数据收集同步（你可以使用 Form 的 onValuesChange），但还是可以继续监听 onChange 事件。
你不能用控件的 value 或 defaultValue 等属性来设置表单域的值，默认值可以用 Form 里的 initialValues 来设置。注意 initialValues 不能被 setState 动态更新，你需要用 setFieldsValue 来更新。
你不应该用 setState，可以使用 form.setFieldsValue 来动态改变表单值。

#### 4 newData?.filter()

这里的  ?.  意思是：newData 存在不为空才执行 filter  相当于  if(newData) { newData.filter }

#### 5 <Table<TableFormDateType>

```typescript
// TypeScript 2.9 之后也可以这样写
// https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-9.html#generic-type-arguments-in-jsx-elements
<Table<User> columns={columns} dataSource={data} />
<Table<User> dataSource={data}>
  <Table.Column<User> key="name" title="Name" dataIndex="name" />
</Table>

其实就是给这个类加类型
相当于下面
// 使用 JSX 风格的 API
class NameColumn extends Table.Column<User> {}

<UserTable dataSource={data}>
  <NameColumn key="name" title="Name" dataIndex="name" />
</UserTable>
```

#### 6 类型断言 as

意思是告诉编译器，这个变量我知道是什么类型，我推断他的类型为 as 后面的类型。它只在编译时候起作用。

```typescript
let val: any = "hello word";
// 两种方式类型断言
let len = (<string>val).length;
let len = (val as string).length; // tsx中只有as被允许
```

#### 7 event.persist()

如果在 react 中想异步访问事件属性（如在 setTimeout 内），应该在是处理事件时调用 event.persist()，这会从事件池中移除该合成函数并允许对该合成事件的引用被保留下来.

#### 删除对象属性

```typescript
// 删除对象属性
delete target.isNew;
```

#### 8 model 的写法

1 写 ModelState 的 interface。
2 写 ModelType 的 interface。包括 namespace， state 的类型就是 ModelState。effects 中每一个的类型用从 umi 中引入的 Effect，reducer 也是从 umi 中引入 Reducer。但是要用泛型方式传入 ModelState。
3 写 model，model 里无论添加了 state，reducer，effects 都要去 interface 里添加类型。effects 里的每个函数参数为：action 和 effects(dva 封装的 saga 方法)。reducer 里也是函数参数为 state， action。

```typescript
import { Reducer, Effect } from "umi";
import { CurrentUser } from "./data.d";

export interface ModelState {
  currentUser: CurrentUser;
}

interface ModelType {
  namespace: string;
  state: ModelState;
  effects: {
    fetchCurrent: Effect;
  };
  reducers: {
    saveCurrentUser: Reducer<ModelState>;
  };
}

const Model: ModelType = {
  namespace: "center",
  state: {
    currentUser: {},
  },
  effects: {
    *fetchCurrent(action, { call, put }) {
      const res = yield call("/api", { params: action.payload });
      put({
        type: "saveCurrentUser",
        payload: res,
      });
    },
  },
  reducers: {
    saveCurrentUser(state, action) {
      return {
        ...state,
        currentUser: action.payload,
      };
    },
  },
};

export default Model;
```

#### 9 用函数封装部分 jsx

```typescript
  // 如果逻辑比较复杂 可以把这部分拿出来 用函数封装
  renderUserInfo = (currentUser: CurrentUser) => (
    <div className={styles.detail}>
      <p>
          {
            // (currentUser.geographic || { province: { label: '' } }).province.label
            currentUser.geographic?.province?.label
          }
      </p>
    </div>
	)

<div>
	{ this.renderUserInfo(currentUser) }
</div>
```

#### 10 接口类型也可以使用某个接口里面某一个属性

```typescript
type CurrentUser = {
  tags: TagType[];
};

interface TagProps {
  tags: CurrentUser["tags"];
}
```

#### 11 useEffect

```javascript
useEffect(
  () => {
    // 做一些事
    console.log("执行effect");

    // return的函数用于清楚effect
    return () => {
      clear("effect");
    };
  },
  // 依赖项
  []
);
```

1 使用 useEffect，useEffect 里面的所有东西都是每次 render 都是独立的，里面有独立的 state、effects、function。
2 react-hooks 的作用是一种同步的作用，同步函数 hooks 函数内的内容与外部的 props 以及 state，
3 所以才会在每次 render 之后执行 useEffect 里面的函数，这时可以获取到当前 render 结束后的 props 和 state，来保持一种同步
4 依赖项的比较是浅比较，如果 state 是一个对象，那么对象只要指向不发生变化，那么就不会执行 effect 里面的函数
