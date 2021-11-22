## 手动上报错误
* `Sentry`的`SDK`会自动上报错误、未捕获的异常和`unhandled rejections`以及其他类型的错误。
* 对于未捕获的异常，可以手动上传，[官方文档](https://docs.sentry.io/platforms/javascript/guides/react/usage/)
```
import * as Sentry from "@sentry/react";
Sentry.captureException(err);
```
* 可以在issues里查看
* 可以上报纯文本信息
```
Sentry.captureMessage("xxx");
```
## 设置上报等级
* 可以对上报的信息定义等级，主要包括:
```
export declare enum Severity {
  Fatal = "fatal",
  Error = "error",
  Warning = "warning",
  Log = "log",
  Info = "info",
  Debug = "debug",
  Critical = "critical"
}
```
* 手动上报错误定义`level`
```
Sentry.captureException(err, {
  level: Sentry.Severity.Warning
});
```
* 手动上报纯文本定义`level`
```
Sentry.captureMessage("xxx",Sentry.Severity.Warning);
```
* 定义一个范围，在范围内容统一定义`level`
```
Sentry.configureScope(function(scope) {
  scope.setLevel(Sentry.Severity.Warning);
  aFunctionThatMightFail();
  bFunctionThatMightFail();
});
```
## `Sentry`错误边界组件
* 使用`Sentry.ErrorBoundary`组件
```
onst App = () => {
  const [error,setError] = useState(false);
  return (
    <>
      { error && new Error('error')}
      <button onClick={() => setError(true)}>set error</button>
    </>
  );
}
const WithSentryErrorBoundaryApp = () => {
  return (
    <Sentry.ErrorBoundary fallback={<div>error boundary error</div>} showDialog>
      <App />
    </Sentry.ErrorBoundary>
  );
}

ReactDOM.render(
  <WithSentryErrorBoundaryApp />,
  document.getElementById('root')
);
```
  * 点击按钮，页面会展示`fallback`内容，并且弹出反馈弹窗，向`Sentry`提交反馈，反馈内容可以在`Sentry`的`User Feedback`中查看
  * [React Error Boundary](https://docs.sentry.io/platforms/javascript/guides/react/components/errorboundary/)
## 集成`redux`
* 安装`redux`
```
npm i redux
```
* [集成redux](https://docs.sentry.io/platforms/javascript/guides/react/configuration/integrations/redux/)
  * 通过`sentryReduxEnhancer`中的`actionTransformer`和`stateTransformer`过滤不需要在sentry管理界面中展示的`action`和`state`
  * 依次点击四个按钮，会报错
  * 在issues里找到这条错误记录
  * 在`BREADCRUMBS`中，可以看到有三条`redux`相关的`CATEGORY`，分别表示`redux init`、`dispatch SET_A`、`dispatch SET_C`。`dispatch SET_B`没有，是因为在`actionTransformer`中将其过滤掉了
![image](https://github.com/JX-Zhuang/sentry/blob/master/06/imgs/redux.png)
  * 页面往下翻，可以看到`REDIX.STATE`中记录了报错前最后的`state`，有`a`、`b`值，但没有c的值，是因为在`stateTransformer`中过滤了c的值
![image](https://github.com/JX-Zhuang/sentry/blob/master/06/imgs/state.png)
## rrweb重播
* `rrweb`是一个开源的`Web`会话回放库，它提供易于使用的`API`来记录用户的交互并远程回放。可以记录鼠标移动轨迹、交互动作和页面变化，并播放。
* `@sentry/rrweb`是`rrweb`的`sentry`插件。
```
npm i @sentry/rrweb rrweb
```
* 使用
```
import SentryRRWeb from '@sentry/rrweb';
Sentry.init({
  integrations: [
    new SentryRRWeb({
      checkoutEveryNms: 10 * 1000, // 每10秒重新制作快照
      checkoutEveryNth: 200, // 每 200 个 event 重新制作快照
      maskAllInputs: false, // 将所有输入内容记录为 *
    }),
  ],
});
```
* 在页面中点击各个按钮，直到报错，在issues中找到相关错误详情，在页面最底部有一个`REPLAY`模块，点击播放可以回放错误发生前的一系列操作