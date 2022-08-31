---
title: react-router-dom 简单使用
urlname: qu2m41
date: '2020-09-08 18:59:45 +0800'
tags: []
categories: []
---

## Route 三种渲染方式

```javascript
<Route path="/children" children={() => <div>children user</div>} />
<Route path="/search/:id" component={Search} />
<Route render={() => <div>404</div>} />
```

## 引入组件

```javascript
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link,
  useRouteMatch,
  useParams,
  useHistory,
  useLocation,
} from "react-router-dom";
```

## 配置组件

```javascript
export default function App() {
  return (
    <Router
      getUserConfirmation={(message, callback) => {
        const allowTransition = window.confirm(message);
        callback(allowTransition);
      }}
    >
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>

            <li>
              <Link to="/about">about</Link>
            </li>

            <li>
              <Link to="/user">User</Link>
            </li>
          </ul>
        </nav>

        <Switch>
          <Route path="/about/:id"> id page</Route>
          <Route path="/about">
            {" "}
            <About />{" "}
          </Route>
          <Route path="/user">
            {" "}
            <User />{" "}
          </Route>
          <Route path="/">
            {" "}
            <Home />{" "}
          </Route>
        </Switch>
      </div>
    </Router>
  );
}
```
