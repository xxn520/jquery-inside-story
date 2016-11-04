## 异步请求 Ajax

这可能是最后一章了，其余的暂时不看。

### 核心方法

###### `jQuery.ajax([url,]options)`

1. 修正参数。
2. 一堆局部变量，要疯了要疯了。
 - `parts`：存放协议、域名、端口等用来判断是否跨域。
 - `i`：循环变量。
 - `cacheURL`
 - `responseHeadersString`：响应头的字符串形式。
 - `timeoutTimer`：超时计时器，设置了 `timeout` 后，超时会调用 `jQuery.abort` 来取消本次请求。
 - `fireGlobals`：表示是否触发全局 Ajax 事件。
 - `transport`：指向为当前请求分配的一个请求发送器，拥有 `send`、`abort` 方法。
 - `responseHeaders`：响应头。
 - `s = jQuery.ajaxSetup( {}, options )`：构造完整的请求选项集，它会合并默认选项 `ajaxSettings` 和用户传入的 `options`。
 - `callbackContext`：回调函数的上下文，默认为 `s`，如果选项中有 `context`，那么使用 `s.context`。
 - `globalEventContext`：Ajax 全局事件的上下文，默认为 `jQuery.event`，通过 `jQuery.event.trigger` 触发全局事件传播到所有元素。如果指定了 `context`，并且它是 DOM 元素或 jQuery 集合，那么只会在指定的元素上触发 `ajaxSuccess` 等事件，通过 `.trigger` 调用。
 - `deferred`：延迟对象，用于异步。
 - `completeDeferred`：回调函数列表。
 - `statusCode`：存放依赖于状态码的回调函数。
 - `requestHeaders`、`requestHeadersNames`：记录请求头。
 - `state`：`jqHXR` 的状态，0、1、2 分别表示初始状态、处理中、响应成功。
 - `strAbort`：默认的取消消息。
 - `jqXHR`
3. 构造 `jqHXR` 对象，它是浏览器原生 `XMLHttpRequest` 对象的超集，如果请求发送的不是 `XMLHttpRequest`，会尽量模拟 `XMLHttpRequest` 的行为。并且还拥有异步队列的行为。
 - `readyState`：`XMLHttpRequest` 的同名属性，初始为 0，发送前为 1，响应完成后为 4。
 - `setRequestHeader`：`XMLHttpRequest` 的同名方法，但仅仅是缓存请求头。真正设置请求头是在 `XMLHttpRequest` 的同名方法被调用时。
 - `getAllResponseHeaders`：`XMLHttpRequest` 的同名方法，当 `state` 为 2，响应成功以后，可获取响应头，被存储在 `responseHeadersString` 中。
 - `getResponseHeader`：扩展方法，根据 `key`，获取响应头的字段。
 - `overrideMimeType`：`XMLHttpRequest` 的同名方法，用于覆盖响应内容的 `ContentType`。不过真正生效是在 `XMLHttpRequest` 的同名方法被调用时。
 - `statusCode`：`state` 小于 2，向 `statusCode` 中添加回调，否则的话执行回调。
 - `abort`：取消请求，并回调 `done(0,finalText)` 来触发失败的回调。
4. 构造回调函数。
 1. 如果 `state` 已经为 2，直接返回。 
 2. 设置 `state` 为 2。 
 3. 清空定时器、缓存的响应头字符串、请求发送器。
 4. `readyState` 改为 4，根据状态得到 `isSuccess`。
 5. 读取响应数据 `ajaxHandleResponses`。
 6. 数据类型转换 `ajaxConvert`