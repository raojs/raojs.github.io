---
layout: post
title:  'Web Workers 要点总结'
date:   2019-07-18
updated-date: 2019-07-18
categories: workers
---

Javascript 是一门单线程语言，单线程在执行复杂任务时很容易阻塞用户界面，Web workers 生成了真正的操作系统级别线程，提供了多任务并行执行的能力。

本文主要为要点总结，了解详细内容可以查看文末参考链接。

关键字：专用 worker (Dedicated worker)、共享 worker (Shared worker)、对象转移 (Transferring Object)、内联方式创建 worker (Inline worker)、广播通道 (Broadcast Channel)

## Dedicated worker

Dedicated worker 仅能被生成它的脚本所使用。创建一个 Dedicated worker 并通过 postMessage 进行通信，通信数据在消息的 data 字段中：

```js
var worker = new Worker('./worker.js');
worker.addEventListener('message', function(e) {
  console.log(e.data);
});
worker.addEventListener('error', function(e) {
  console.log(e.data);
});
worker.postMessage({cmd: 'pow', data: [10, 2]});
```

worker.js

```js
self.addEventListener('message', function(e) {
  var data = e.data;
  switch (data.cmd) {
    case 'pow':
      var res = Math.pow(data.data[0], data.data[1]);
      self.postMessage(res);
      break;
    default:
      self.postMessage('Unknown');
      break;
  }
  self.close();
});
```

Tips:

- worker 可以访问大多数的 javascript API，比如 Navigator、WebSocket、IndexedDB，但是不能操作 DOM，不能访问 window、document、parent。
- 在 worker 中，self 和 this 都指向全局作用域。
- 终止 worker 有两种方式：
  1. 主线程中调用 `worker.terminate()`，此时 worker 线程会被立刻杀死。
  2. 在 worker 线程中调用 `self.close()`。
- 可以通过 importScripts() 加载其他脚本。

## Shared worker

Shared worker 可以被多个脚本使用，即使在不同的 tab、iframe 或 worker 中。与 Shared worker 通信必须通过确切的端口：

主线程

```js
if (!!window.SharedWorker) {
  var myWorker = new SharedWorker('shared-worker.js');

  myWorker.port.onmessage = function(e) {
    console.log('Message received from worker, data:', e.data);
  }

  myWorker.port.postMessage([1, 1]);
}
```

shared-worker.js

```js
onconnect = function(e) {
  var port = e.ports[0];

  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data[0] + e.data[1]);
    port.postMessage(workerResult);
  }
}
```

Tips:

- 通信前需要调用 start 方法启用端口，为端口添加 onmessage 处理函数会默认启用端口，采用 addEventListener 监听时才需要手动调用 start 方法启用。
- Shared worker 遵循同源策略。
- Shared worker 默认是无法在开发工具中查看的，进行调试需要访问 `chrome://inspect/#workers`，点击 `inspect`，弹出单独的面板进行调试。
- Shared worker 重用了 [MessageEvent](https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent) 接口，worker 线程中 e.ports 虽然是数组形式但是长度始终为 1，在别的场景可以大于 1 比如通过 window.postMessage 将多个 MessageChannel port 转移所有权 (transfer) 给 iframe 页面。

## Transferring Object

在通过 postMessage 方法与 worker 进行通信时，支持传递复杂类型的消息（类似JSON对象、文件、ArrayBuffer），它们在传递时会进行拷贝 ([structured cloning](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm))，有一个序列化和反序列化的过程，会有额外的开销，且数据越大耗时越长。

有一种基本零成本的传递方式，通过传递引用将数据转移到 worker，原执行上下文无法再访问该对象。不过只有 [Transferable Object](https://developer.mozilla.org/en-US/docs/Web/API/Transferable) 可以支持。postMessage 调用时传递引用即可，写法如下：

```js
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```

需要注意的是 postMessage 方法 window 和 worker 的 API 有些许不同，window 间通信存在跨域问题所以需要传递 targeOrigin 参数，接收方也有必要进行来源检测。

```js
worker.postMessage(arrayBuffer, [transferableList]);
window.postMessage(arrayBuffer, targetOrigin, [transferableList]);
```

## Inline worker

Web Workers 遵循同源策略，当资源部署在不同的域下面时（常见场景为使用 CDN 服务），workers 的执行很可能会被浏览器阻止。

通过内联的方式可以避过这一问题，调用 URL.createObjectURL 这个 API 将数据块生成全局唯一标识符，然后创建 worker。

创建一个 Inline worker:

```js
var blob = new Blob(['onmessage = function(e) { postMessage('msg from worker'); }']);
var worker = new Worker(window.URL.createObjectURL(blob));
```

Tips:

- URL.createObjectURL 不支持 Shared worker。
- 如果代码通过 webpack 打包，[worker-loader](https://www.npmjs.com/package/worker-loader) 可以帮忙提取 worker 代码，通过 inline 配置可以选择 worker 是以独立文件还是 Blob 的形式存在。
- 当 worker 依赖较多或者文件较大时，分离成单独的文件可以享受缓存福利。

## Broadcast Channel

通过 [Broadcast Channel](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel) 可以在不同的资源上下文（tabs、iframes、workers）中传递消息，消息会发送到所有监听者（不包括自身），Broadcast Channel 遵循同源策略。

```js
var channel = new BroadcastChannel('myChannel');

channel.onmessage = function(e) {
    console.log(e.data);
};

channel.postMessage('test');

channel.close();
```

## 参考资料

- [Web Workers](https://html.spec.whatwg.org/multipage/workers.html) specification
- [Using Web Workers (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)
- [Transferable Objects: Lightning Fast!](https://developers.google.com/web/updates/2011/12/Transferable-Objects-Lightning-Fast)
- [The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)
- [BroadcastChannel API: A Message Bus for the Web](https://developers.google.com/web/updates/2016/09/broadcastchannel)


