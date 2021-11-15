## 一、下载sentry docker 并部署
### 1. 下载
```
git clone https://github.com/getsentry/onpremise
```

### 2. 安装
* 进入目录
```
cd onpremise
```

* 执行 ./install.sh 报错
```
./install/_lib.sh: line 15: realpath: command not found
```

* 解决方案https://github.com/getsentry/onpremise/issues/941
```
brew install coreutils
```
* 重新 ./install.sh，如果有以下报错，说明内存分配的不够
```
▶ Parsing command line ...
▶ Setting up error handling ...
▶ Checking minimum requirements ...
FAIL: Required minimum RAM available to Docker is 3800 MB, found 1988 MB
```
* docker内存分配8G
* 重新 ./install.sh，成功
```
------------------------------------------------------
You're all done! Run the following command to get Sentry running:

  docker compose up -d
------------------------------------------------------
```

### 3. 启动
* 在 onpremise 文件中中启动服务
```
docker compose up -d
```
* 访问 http://localhost:9000/

### 4. 注册
* 创建超级管理员账号
```
docker-compose run --rm web createuser --superuser
```
* 如果还使用上述命令，还可以创建新的账号，但数据是共享的
* 覆盖原账号：如果想修改密码可以使用
```
docker-compose run --rm web createuser --superuser --force-update
```

### 5. 登录
* 登录成功

### 6. 停服
* 在onpremise目录
```
docker-compose down
```

## 二、接入sentry sdk
### 1. 创建sentry项目
* 创建project
* 选择react，把project name改为cra-test，点击创建
* 创建后会进入Configure React页，介绍前端如何接入sentry
### 2. 前端接入 sentry sdk
* 创建前端项目
```
npx create-react-app sentry-sdk-demo
```
* 安装sdk
```
# Using yarn
yarn add @sentry/react @sentry/tracing

# Using npm
npm install --save @sentry/react @sentry/tracing
```
* 引入sdk
```
//index.js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import * as Sentry from "@sentry/react";
import { Integrations } from "@sentry/tracing";
Sentry.init({
  dsn: "http://xxxxx@localhost:9000/2",
  integrations: [new Integrations.BrowserTracing()],
  tracesSampleRate: 1.0,
});
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

reportWebVitals();
```
* Sentry.init相关文档 https://docs.sentry.io/platforms/javascript/configuration/options/
* 接入后遇到跨域的问题，请检查 relay/credentials.json，如果是空，请在sentry目录执行
```
docker-compose run --rm --no-deps -v $(pwd)/relay/config.yml:/tmp/config.yml relay --config /tmp credentials generate --stdout
```

### 3. 前端错误
* 绑定一个执行出错的方法
```
function App() {
  const errorClick = () => window.a();
  return (
    <div className="App">
      <button onClick={errorClick}>click me</button>
    </div>
  );
}
export default App;
```
### 4. 在sentry查看报错