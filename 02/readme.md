## 上传`sourcemap`
有两种上传`sourcemap`的方式
* `sentry-cli`命令行工具
* `webpack`插件

## 通过`sentry-cli`上传`sourcemap`
* [sentry-cli官方文档](https://docs.sentry.io/product/cli/)
### 1.在开发的电脑安装`@sentry/cli`
```
npm install -g --unsafe-perm=true --allow-root @sentry/cli
```
### 2.配置`@sentry/cli`
* `url`:要上传的`sentry`的地址
* `org`:组织名称，可以在`sentry`的`Settings`中设置
* `project`:`sentry`的项目名，可以在`sentry`管理界面的`Projects`中查看、创建、设置项目
* `token`:`sentry`服务准入`sentry-cli`上传的鉴权`token`，可以在`sentry`管理界面的 `Settings -> Account -> API -> Auth Tokens`中创建和删除
![image](https://github.com/JX-Zhuang/sentry/blob/master/02/imgs/token.png)
### 3.执行命令创建`sentry-cli`配置文件
```
sentry-cli --url http://localhost:9000 login
```
### 4.命令行弹出
```
This helps you signing in your sentry-cli with an authentication token.
If you do not yet have a token ready we can bring up a browser for you
to create a token now.

Sentry server: localhost
Open browser now? [y/n] y
Enter your token:
```
输入token
### 5.会生成文件 ~/.sentryclirc
* 前端项目根目录中如果有`.sentryclirc`文件，`sentry-cli`命令会优先读取当前目录下的。如果没有这个文件，才会读取`~/.sentryclirc`
```
vim ~/.sentryclirc
```
```
[auth]
token=81fc556a0d1cxxxxx

[defaults]
url=http://localhost:9000
org=sentry
project=cra-test
```
### 6.手动上传`sourcemap`
* 编译前端项目，生成`build`目录，里面包含`sourcemap`文件
```
npm run build
```
* 在上传之前，先看下`sentry`管理界面`Settings -> sentry -> Projects -> cra-test -> Source Maps -> Archive`，会发现列表中有一个`0.0.1`版本，这个版本是我们在前端项目里通过`sentry.init`的`release`配置的，只要前端上传过报错信息，就会在管理界面生成这个版本。目前列表中应该没有任何文件。
* 上传`sourcemap`
```
sentry-cli releases files 0.0.1 upload-sourcemaps --url-prefix '~/' './build'
```

```
> Found 7 release files
> Analyzing 7 sources
> Analyzing completed in 0.073s
> Rewriting sources
> Rewriting completed in 0.046s
> Adding source map references
> Bundling files for upload... ~/static/js/2.3a8da821.chunk.js.map
> Bundling completed in 0.068s
> Optimizing completed in 0.001s
> Uploading completed in 0.172s
> Uploaded release files to Sentry
> Processing completed in 0.45s
> File upload complete (processing pending on server)
> Organization: sentry
> Project: cra-test
> Release: 0.0.1
> Dist: None

Source Map Upload Report
  Minified Scripts
    ~/static/js/2.3a8da821.chunk.js (sourcemap at 2.3a8da821.chunk.js.map)
    ~/static/js/main.12561afa.chunk.js (sourcemap at main.12561afa.chunk.js.map)
    ~/static/js/runtime-main.06bbfa03.js (sourcemap at runtime-main.06bbfa03.js.map)
  Source Maps
    ~/static/css/main.8c8b27cf.chunk.css.map
    ~/static/js/2.3a8da821.chunk.js.map
    ~/static/js/main.12561afa.chunk.js.map
    ~/static/js/runtime-main.06bbfa03.js.map
```
* 回到Archive页面，发现sourcemap已经上传
![image](https://github.com/JX-Zhuang/sentry/blob/master/02/imgs/archive.png)
### 7.查看效果
* `http-server`启动前端服务
* 访问`http://127.0.0.1:8080/`，手动制造错误
* 在`sentry`后台查看`issue`，可以看到源码
![image](https://github.com/JX-Zhuang/sentry/blob/master/02/imgs/issue.png)
* 删除`sentry`
```
sentry-cli releases files 0.0.1 delete --all
```
```
D ~/static/css/main.8c8b27cf.chunk.css.map
D ~/static/js/2.3a8da821.chunk.js
D ~/static/js/2.3a8da821.chunk.js.map
D ~/static/js/main.12561afa.chunk.js
D ~/static/js/main.12561afa.chunk.js.map
D ~/static/js/runtime-main.06bbfa03.js
D ~/static/js/runtime-main.06bbfa03.js.map
```
## 通过`webpack`插件上传`sourcemap`
`npm run build`时自动上传`sourcemap`
### 1.安装`@sentry/webpack-plugin`
```
npm i -D @sentry/webpack-plugin
```
* [@sentry/webpack-plugin官方文档](https://github.com/getsentry/sentry-webpack-plugin)
### 2.配置`webpack`
* [react-app-rewired](https://github.com/timarney/react-app-rewired)
```
// config-overrides.js
const SentryCliPlugin = require('@sentry/webpack-plugin');
module.exports = function override(config, env) {
  config.devtool = 'source-map';
  config.plugins.push(
    new SentryCliPlugin({
      release: '0.0.1',
      authToken: 'xxxxxxxx',
      url: 'http://localhost:9000',
      org: 'sentry',
      project: 'cra-test',
      urlPrefix: '~/',
      include: './build',
      ignore: ['node_modules'],
    })
  );
  return config;
}
```
* 配置中的`release`、`authToken`、`url`、`org`、`project`、`urlPrefix`、`include`同`.sentryclirc`
### 3.修改`package.json`
```
{
  "scripts": {
    "build": "react-app-rewired build && rm -rf dist/*.map",
  }
}
```
### 4.`npm run build`
```
Creating an optimized production build...
> Found 7 release files
> Analyzing 7 sources
> Analyzing completed in 0.055s
> Rewriting sources
> Rewriting completed in 0.033s
> Adding source map references
> Bundling files for upload...
> Bundling completed in 0.071s
> Optimizing completed in 0.001s
> Uploading completed in 0.116s
> Uploaded release files to Sentry
> Processing completed in 0.094s
> File upload complete (processing pending on server)
> Organization: sentry
> Project: cra-test
> Release: 0.0.1
> Dist: None

Source Map Upload Report
  Minified Scripts
    ~/static/js/2.eb79a9fc.chunk.js (sourcemap at 2.eb79a9fc.chunk.js.map)
    ~/static/js/main.6f5aaa33.chunk.js (sourcemap at main.6f5aaa33.chunk.js.map)
    ~/static/js/runtime-main.54cef60d.js (sourcemap at runtime-main.54cef60d.js.map)
  Source Maps
    ~/static/css/main.8c8b27cf.chunk.css.map
    ~/static/js/2.eb79a9fc.chunk.js.map
    ~/static/js/main.6f5aaa33.chunk.js.map
    ~/static/js/runtime-main.54cef60d.js.map
Compiled successfully.
```
### 5.回到`Archive`，`sourcemap`文件已经传到`sentry`