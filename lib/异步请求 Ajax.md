## 异步请求 Ajax

这可能是最后一章了，其余的暂时不看。

### 请求参数

###### `jQuery.ajaxSetup(target[,settings])`

将设置的参数与默认的参数进行合并。

```
jQuery.ajaxSetup = function( target, settings ) {
    // 如果有参数
    return settings ?
        // 创建一个设置对象，先将jQuery.ajaxSettings的属性放进去，
        // 然后将参数也放进去
        ajaxExtend( ajaxExtend( target, jQuery.ajaxSettings ), settings ) :
        // 并将设置对象的属性放进jQuery.ajaxSettings对象里
        ajaxExtend( jQuery.ajaxSettings, target );
};
```

###### `jQuery.ajaxSettings`

默认的参数。

```
ajaxSettings: {
	url: ajaxLocation, // location.href，IE 下对于设置了 document.domain 的情况会抛错，此时通过 a.href 来读取
	type: "GET",
	isLocal: rlocalProtocol.test( ajaxLocParts[ 1 ] ), // 是否为本地的地址如:about
	global: true, // 无论怎么样这个请求将触发全局AJAX事件处理程序。
	processData: true, // 默认情况下，通过data选项传递进来的数据，如果是一个对象(技术上讲只要不是字符串)，都会处理转化成一个查询字符串，以配合默认内容类型 "application/x-www-form-urlencoded"。如果要发送 DOM 树信息或其它不希望转换的信息，请设置为 false。
	async: true, // 默认异步
	contentType: "application/x-www-form-urlencoded; charset=UTF-8", // 发送信息至服务器时内容编码类型。
	/*
	timeout: 0,
	data: null, // 发送到服务器的数据。将自动转换为请求字符串格式。
	dataType: null, // 预期服务器返回的数据类型。如果不指定，jQuery 将自动根据 HTTP 包 MIME 信息来智能判断，比如XML MIME类型就被识别为XML。
	username: null,
	password: null,
	cache: null,
	throws: false,
	traditional: false, // 如果你想要用传统的方式来序列化数据，那么就设置为true。
	headers: {}, // 一个额外的"{键:值}"对映射到请求一起发送。此设置会在beforeSend 函数调用之前被设置 ;因此，请求头中的设置值，会被beforeSend 函数内的设置覆盖 。
	// 内容类型发送请求头（Content-Type），用于通知服务器该请求需要接收何种类型的返回结果。
	accepts: {
		"*": allTypes,
		text: "text/plain",
		html: "text/html",
		xml: "application/xml, text/xml",
		json: "application/json, text/javascript"
	},
	// 一个以"{字符串/正则表达式}"配对的对象，根据给定的内容类型，解析请求的返回结果。
	contents: {
		xml: /xml/,
		html: /html/,
		json: /json/
	},
	responseFields: {
		xml: "responseXML",
		text: "responseText",
		json: "responseJSON"
	},
	// 一个数据类型到数据类型转换器的对象。
	converters: {
		"* text": String,
		"text html": true,
		"text json": jQuery.parseJSON,
		"text xml": jQuery.parseXML
	},
	// 不需要深拷贝的属性
	flatOptions: {
		url: true,
		context: true
	}
}
```

###### `ajaxExtend(target,src)`

将除了 `flatOptions` 中的参数以外的深拷贝到 `target` 对象上。

### 工具方法

###### `ajaxHandleResponses`

1. 删除掉通配 `dataType`，得到之前覆盖的的 `Content-Type` 或返回的 `Content-Type`。
2. 检查是否能处理，能处理并且匹配上一步求出的 `Content-Type` 那么将其推入 `dataTypes`。
3. 如果 `dataTypes` 是我们想要的，也就是 text、xml、json。那么就确定是它了。
4. 尝试转换成我们要的 `dataType`，如果 `dataTypes[0]` 不存在，则直接用 `type` 作为最终 `dataType`。
5. 否则，看看能不能转换，能的话就用 `type` 作为最终 `dataType`。
6. 保存第一个 `type`。
7. 用最终 `dataType` 或者用第一个 `type`。
8. 如果有最终 `dataType`，并且不是 `dataTypes[0]`，将其推入 `dataTypes`。
9. 返回 `responses[finalDataType]`。

###### `ajaxConvert`

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
4. 添加异步队列的行为。
5. 修正参数 
 - `url`
 - `type`
 - `dataTypes`
 - `crossDomain`
 - `data`
6. 应用前置过滤器。
7. 如果在前置过滤器中终止，那么直接返回 `jqXHR`。
8. 如果没有禁用全局事件且没有其他 Ajax 请求在执行，那么触发 `ajaxStart` 事件。
9. 继续修正参数。
 - `type`：改为大写。
 - `hasContent`：是否有请求内容，GET、HEAD 没有。
10. 缓存 `url`。
4. 构造回调函数。
 1. 如果 `state` 已经为 2，直接返回。 
 2. 设置 `state` 为 2。 
 3. 清空定时器、缓存的响应头字符串、请求发送器。
 4. `readyState` 改为 4，根据状态得到 `isSuccess`。
 5. 读取响应数据 `ajaxHandleResponses`。
 6. 数据类型转换 `ajaxConvert`