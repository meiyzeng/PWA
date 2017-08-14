### PWA
> Progressive Web App, 简称 PWA，是提升 Web App 的体验的一种新方法，能给用户原生应用的体验。

PWA(Progressive Web Apps) 不是一项技术，不是一个框架,可以把她理解为一种模式
一种通过应用一些技术将 Web App 在安全、性能和体验等方面带来渐进式的提升的一种 Web App 模式。


* 可靠 - 即使在不稳定的网络环境下，也能瞬间加载并展现
* 体验 - 快速响应，并且有平滑的动画响应用户的操作
* 粘性 - 像设备上的原生应用，具有沉浸式的用户体验，用户可以添加到桌面

> 渐进式改造步骤：

- 第一步，应该是安全，将全站 HTTPS 化，因为这是 PWA 的基础，没有 HTTPS，就没有 Service Worker
- 第二步，应该是 Service Worker 来提升基础性能，离线提供静态文件，把用户首屏体验提升上来
- 第三步，App Manifest，这一步可以和第二步同时进行
后续，再考虑其他的特性，离线消息推送等


Service Worker 具有离线缓存（Offline Cache），消息推送（Notification Push），后端同步（Background sync）的能力

Service Worker 在 Web Worker 的基础上加上了持久离线缓存能力。

Service Worker 有以下功能和特性

- 一个独立的 worker 线程，独立于当前网页进程，有自己独立的 worker context。
- 一旦被 install，就永远存在，除非被 uninstall
- 需要的时候可以直接唤醒，不需要的时候自动睡眠（有效利用资源，此处有坑）
- 可编程拦截代理请求和返回，缓存文件，缓存的文件可以被网页进程取到（包括网络离线状态）
- 离线内容开发者可控
- 能向客户端推送消息
- 不能直接操作 DOM
- 出于安全的考虑，必须在 HTTPS 环境下才能工作
- 异步实现，内部大都是通过 Promise 实现

前提条件


- 由于 Service Worker 要求 HTTPS 的环境，我们通常可以借助于 [github page](https://pages.github.com/) 进行学习调试。当然一般浏览器允许调试 Service Worker 的时候 host 为 localhost 或者 127.0.0.1 也是 ok 的

- Service Worker 的缓存机制是依赖  [Cache API](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 实现的
- 依赖 [HTML5 fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API)
- 依赖 [Promise](https://developer.mozilla.org/zh-CN/docs/Web/javaScript/Reference/Global_Objects/Promise) 实现

注册
```
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function () {
        navigator.serviceWorker.register('/sw.js', {scope: '/'})
            .then(function (registration) {

                // 注册成功
                console.log('ServiceWorker registration successful with scope: ', registration.scope);
            })
            .catch(function (err) {

                // 注册失败:(
                console.log('ServiceWorker registration failed: ', err);
            });
    });
}
```

可以通过 chrome://serviceworker-internals 来查看服务工作线程详情
 (chrome://inspect/#service-workers 目前好像看不到)

 我们可以在 install 的时候进行静态资源缓存，也可以通过 fetch 事件处理回调来代理页面请求从而实现动态资源缓存。

 - on install 的优点是第二次访问即可离线，缺点是需要将需要缓存的 URL 在编译时插入到脚本中，增加代码量和降低可维护性；
 - on fetch 的优点是无需更改编译过程，也不会产生额外的流量，缺点是需要多一次访问才能离线可用。

> Service Worker 版本更新

/sw.js 控制着页面资源和请求的缓存，那么如果缓存策略需要更新呢？也就是如果 /sw.js 有更新怎么办？/sw.js 自身该如何更新？

如果 /sw.js 内容有更新，当访问网站页面时浏览器获取了新的文件，逐字节比对 /sw.js 文件发现不同时它会认为有更新启动 更新算法open_in_new，
于是会安装新的文件并触发 install 事件。但是此时已经处于激活状态的旧的 Service Worker 还在运行，新的 Service Worker 完成安装后会进入 waiting 状态。
直到所有已打开的页面都关闭，旧的 Service Worker 自动停止，新的 Service Worker 才会在接下来重新打开的页面里生效。


对于网址可寻址的资源，使用 [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) （服务工作线程的一部分）。 对于所有其他数据，使用 [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) （具有一个 Promise 包装器）。

IndexedDB 基于事件的，而 Cache API 基于 Promise

Promise
https://developer.mozilla.org/zh-CN/docs/Web/javaScript/Reference/Global_Objects/Promise

Fetch API
https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API

Cache

https://developer.mozilla.org/zh-CN/docs/Web/API/Cache