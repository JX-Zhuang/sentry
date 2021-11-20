# 导航操作检测
## 基本导航操作检测
* 在前端项目中加入路由
```
npm i -S react-router-dom
```
```
// App.js
import { BrowserRouter as Router, Switch, Link, Route } from 'react-router-dom';
function App(props) {
  const fn = () => {
    console.log(window.a.b);
  }
  return (
    <Router history={props.history}>
      <>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/user/1">User1</Link>
            </li>
            <li>
              <Link to="/user/2">User2</Link>
            </li>
          </ul>
        </nav>
        <Switch>
          <Route path="/user/:id" component={'user'} />
          <Route path="/about" component={'About'} />
          <Route path="/" component={<button onClick={fn}>Break the world</button>} />
        </Switch>
      </>
    </Router>
  );
}

export default App;
```
* 点击`About`跳转，在`sentry`后台可以发现多上报了一条`transaction`
![image](https://github.com/JX-Zhuang/sentry/blob/master/04/imgs/discover.png)
* 过一会儿，在`Sentry->Discover->All Events`中，我们可以看到刚才上报的`transaction: /about`，点击打开详情
![image](https://github.com/JX-Zhuang/sentry/blob/master/04/imgs/discover-detail.png)
## `React-Router`导航操作检测
* 对于`/user/:id`的路由，不同的`id`，上报的`transaction`名称是不同的，例如`/user/1`、`/user/2`。
* 我们可以分别在`/user/1`、`/user/2`路由下刷新页面，然后在Sentry里，会看到两条不同的 TRANSACTION: /user/1、/user/2
![image](https://github.com/JX-Zhuang/sentry/blob/master/04/imgs/users.png)
* 如果我们希望他们是一条，预期应该显示: /user/:id，在`React`项目中，我们可以通过`Sentry.reactRouterV5Instrumentation`配置。（仅支持react-router-dom v5)
* 安装history
```
npm i -S history@4
```
```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import { matchPath } from 'react-router-dom';
import * as Sentry from "@sentry/react";
import { Integrations } from "@sentry/tracing";
import { createBrowserHistory } from 'history';
const history = createBrowserHistory();
const routes = [{ path: '/about' }, { path: '/user/:id' }, { path: '/' }];
Sentry.init({
  dsn: "http://xxxx@localhost:9000/2",
  integrations: [new Integrations.BrowserTracing({
    routingInstrumentation: Sentry.reactRouterV5Instrumentation(history, routes, matchPath),
  })],
  tracesSampleRate: 1.0,
  environment: 'localhost',
  release: "0.0.1"
});
ReactDOM.render(
  <React.StrictMode>
    <App history={history}/>
  </React.StrictMode>,
  document.getElementById('root')
);
import { BrowserRouter as Router, Switch, Link, Route } from 'react-router-dom';
function App(props) {
  const fn = () => {
    console.log(window.a.b);
  }
  return (
    <Router history={props.history}>
      <>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/user/1">User1</Link>
            </li>
            <li>
              <Link to="/user/2">User2</Link>
            </li>
          </ul>
        </nav>
        <Switch>
          <Route path="/user/:id" component={'user'} />
          <Route path="/about" component={'About'} />
          <Route path="/" component={<button onClick={fn}>Break the world</button>} />
        </Switch>
      </>
    </Router>
  );
}

export default App;
```
![image](https://github.com/JX-Zhuang/sentry/blob/master/04/imgs/user.png)
![image](https://github.com/JX-Zhuang/sentry/blob/master/04/imgs/performance.png)
* 还有一种方式可以也达到这个目的：使用 Sentry.withSentryRouting 高阶组件包裹 Route，具体可以参考[文档](https://docs.sentry.io/platforms/javascript/guides/react/configuration/integrations/react-router/)