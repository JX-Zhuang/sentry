# 面包屑Breadcrumbs
在错误的详情页，有`Breadcrumbs`信息。它可以追溯错误发生前的一些事情，例如点击、导航等。
* [Breadcrumbs官方文档](https://docs.sentry.io/platforms/javascript/enriching-events/breadcrumbs/)
![image](https://github.com/JX-Zhuang/sentry/blob/master/03/imgs/issues.png)
* 面包屑有两种上报方式：

  1. 自动生成：通过`SDK`及其相关的集成将自动记录多种类型的面包屑。例如，浏览器`JavaScript SDK`将自动记录`DOM`元素上的点击和按键、`XHR`请求、控制台`API`调用以及所有导航变更。
  2. 手动生成：[官方文档](https://getsentry.github.io/sentry-javascript/modules/minimal.html#addbreadcrumb)

```
Sentry.addBreadcrumb({
  category: "auth",
  message: "Authenticated user " + user.email,
  level: Sentry.Severity.Info,
});
```
## `Traces`,`Transactions`和`Spans`
![image](https://github.com/JX-Zhuang/sentry/blob/master/03/imgs/transaction-detail.png)
* 这个页面加载的过程，被称为一个`Transaction`
* 这个`Transaction`是一个树状结构，根节点是`pageload`，表示页面加载，根节点下有浏览器的缓存读取、`DNS`解析、请求、响应、卸载事件、`dom`内容加载事件等过程，还有各种资源加载过程
* 点击开任意一个节点，可以看到：
  * 每个节点都有一个`id`：`Span ID`
  * `Trace Id`:追踪id
  * 这个节点过程话费的时间等信息
* `Sentry`把这些树状结构的节点称为`Span`，他们归属于某一个`Transaction`
* 页面加载是一个`Transaction`，后端API接口逻辑是一个`Transaction`，操作数据库逻辑是一个`Transaction`，它们共同组成了一个`Trace`
* [官方文档](https://docs.sentry.io/product/sentry-basics/tracing/distributed-tracing/)
# 页面加载
## 页面加载与性能指标
* 如果把`SDK`的`Sentry.init()`中，将`new Integrations.BrowserTracing()`去掉，那么`Breadcrumbs`不会显示`pageload`信息
* `BrowserTracing`是用来将浏览器页面加载/导航操作检测为事务，并捕获请求、指标和错误作为跨度
* 在页面加载的`Transaction`中，可以看到页面加过程中不同阶段的所花费的时间，例如`FCP`、`FP`、`LCP`、`FID`等重要的性能指标信息
* 同一个`url`下会收集每一次上报的页面加载`Transaction`
* 在`Sentry`->`Performance`中，可以查看每个`url`下收集的综合性能指标信息
![image](https://github.com/JX-Zhuang/sentry/blob/master/03/imgs/url.png)
  * TPM:平均每分钟事务数
  * FCP: (First Contentful Paint) 首次内容绘制，标记浏览器渲染来自`DOM`第一位内容的时间点，该内容可能是文本、图像、`SVG`元素.
  * LCP: (Largest Contentful Paint) 最大内容渲染，代表viewport中最大的页面元素加载的时间. LCP的数据会通过PerformanceEntry对象记录, 每次出现更大的内容渲染, 则会产生一个新的PerformanceEntry对象
  * FID: (First Input Delay) 首次输入延迟，指标衡量的是从用户首次与您的网站进行交互（即当他们单击链接，点击按钮等）到浏览器实际能够访问之间的时间
  * CLS: (Cumulative Layout Shift) 累积布局偏移，总结起来就是一个元素初始时和其hidden之间的任何时间如果元素偏移了, 则会被计算进去, 具体的计算方法可看这篇文章 https://web.dev/cls/
  * FP: First Paint (FP) 首次绘制，测量第一个像素出现在视口中所花费的时间，呈现与先前显示内容相比的任何视觉变化。这可以是来自文档对象模型 (DOM) 的任何形式，例如`background color`、`canvas`或`image`。FP可帮助开发人员了解渲染页面是否发生了任何意外。
  * TTFB: Time To First Byte (TTFB) 首字节时间，测量用户浏览器接收页面内容的第一个字节所需的时间。TTFB 帮助开发人员了解他们的缓慢是由初始响应(initial response)引起的还是由于渲染阻塞内容(render-blocking content)引起的
  * USERS: UV数
  * USER MISERY: 对响应时间难以容忍度的用户数，User Misery 是一个用户加权的性能指标，用于评估应用程序性能的相对大小。虽然可以使用 Apdex 检查各种响应时间阈值级别的比率，但 User Misery 会根据满意响应时间阈值 (ms) 的四倍计算感到失望的唯一用户数。User Misery 突出显示对用户影响最大的事务。可以使用自定义阈值为每个项目设置令人满意的阈值。阈值设置在Settings -> sentry -> cra-test -> Performance
* [参考1](https://blog.csdn.net/c_kite/article/details/104237256)
* [参考2](https://cloud.tencent.com/developer/article/1878538?from=article.detail.1878539)
* [参考3](https://docs.sentry.io/product/performance/web-vitals/)
## Performance 面板
![image](https://github.com/JX-Zhuang/sentry/blob/master/03/imgs/performance.png)
![image](https://github.com/JX-Zhuang/sentry/blob/master/03/imgs/performance-url.png)
### [Apdex](https://docs.sentry.io/product/performance/metrics/#apdex)
`Apdex`是一种行业标准指标，用于根据应用程序响应时间(response time)跟踪和衡量用户满意度(satisfaction)。`Apdex`分数提供特定`transaction`或端点中满意(satisfactory)、可容忍(tolerable)和失败(frustrated)请求的比率。该指标提供了一个标准来比较 transaction 性能，了解哪些可能需要额外优化或排查，并为性能设定目标
### [Failure Rate 失败率](https://develop.sentry.dev/sdk/event-payloads/span/)
`failure_rate()`表示不成功`transaction`的百分比。`Sentry`将状态为 “ok”、“canceled” 和 “unknown” 以外的`transaction`视为失败。
