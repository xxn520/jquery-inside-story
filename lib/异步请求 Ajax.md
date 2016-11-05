## 异步请求 Ajax

这可能是最后一章了，其余的暂时不看。

### 一些有用的局部变量

###### `ajaxLocation`

默认读取 `location.href` 作为 Ajax 请求的默认 url。但是在 IE 8 以下，当修改了 `document.domain`，再获取 `location.href` 时会出现异常，因此需要捕获异常，并在 `catch` 块中利用 `document.location` 来获取当前的 url。

###### `ajaxLocParts`

`/^([\w.+-]+:)(?:\/\/([^\/?#:]*)(?::(\d+)|)|)/` 这个正则表达式会从 `ajaxLocation` 取出协议、域名、端口，分别存在类数组对象的 0、1、2 下标中。

###### `prefilters`

前置过滤器集，结构为数据类型与前置过滤器数组的映射。

###### `transports`

请求发送器工厂函数集，其结构为数据类型和请求发送器工厂函数的映射。

### 构造请求选项

###### `jQuery.ajaxSetup(target[,settings])`

将设置的选项与默认的选项进行合并。

```
jQuery.ajaxSetup = function( target, settings ) {
    // 如果有选项
    return settings ?
        // 创建一个设置对象，先将 jQuery.ajaxSettings 的属性放进去，
        // 然后将选项也放进去
        ajaxExtend( ajaxExtend( target, jQuery.ajaxSettings ), settings ) :
        // 并将设置对象的属性放进 jQuery.ajaxSettings 对象里
        ajaxExtend( jQuery.ajaxSettings, target );
};
```

###### `jQuery.ajaxSettings`

默认的选项。

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
	headers: {}, // 一个额外的"{键:值}"对映射到请求一起发送。此设置会在beforeSend 函数调用之前被设置 ;因此，请求头中的设置值，会被 beforeSend     函数内的设置覆盖。
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

将除了 `flatOptions` 中的选项以外的深拷贝到 `target` 对象上。

###### 安装 `script` 的 `dataType`

```
jQuery.ajaxSetup({
	accepts: {
		script: "text/javascript, application/javascript, application/ecmascript, application/x-ecmascript"
	},
	contents: {
		script: /(?:java|ecma)script/
	},
	converters: {
		"text script": function( text ) {
			jQuery.globalEval( text );
			return text;
		}
	}
})
```

###### jsonp 的默认设置

```
jQuery.ajaxSetup({
	jsonp: "callback",
	jsonpCallback: function() {
		var callback = oldCallbacks.pop() || ( jQuery.expando + "_" + ( ajax_nonce++ ) );
		this[ callback ] = true;
		return callback;
	}
});
```

### 序列化请求数据

###### `jQuery.param(data,traditional)`

用于将表单元素数组或键值对转化为字符串参数。

1. 如果 `traditional` 为 `false` 则会使用 jQuery 1.3.2 以下的方式来序列化参数。区别在于是否深度序列化。
2. 如果是数组或 jQuery 对象，那么当做表单元素集合，那么遍历数组，将 `name` 和 `vaule` 插入数组 `s` 中。
3. 如果是对象，那么调用 `buildParams` 来构造参数，会根据 `traditional` 来进行深或浅的构造。

###### `jQuery.fn.serializeArray`

负责将表单元素编码为一个对象数组，每个对象包含了 `name` 和 `value`，可以供 `jQuery.param` 进行序列化。根据 W3C 规范，下面的被包含元素有如下要求：

1. 必须含有 `name` 属性。
2. 必须未被禁用。
3. 选取选中的复选框和单选按钮。
4. 选取 `select` 和 `textarea` 元素。
5. 不选取 `type` 为 `file`、`image`、`button` 的 `input` 元素。
6. 不选取 `button` 元素。

###### `jQuery.fn.serialize`

将 `jQuery.fn.serializeArray` 得到的对象数组利用 `jQuery.param` 序列化为字符串。

```
serialize: function() {
	return jQuery.param( this.serializeArray() );
}
```

### 前置过滤器、请求发送器初始化与执行

前置过滤器是一个函数，在每次请求被发送之前被调用，用于处理自定义选项或修改已有选项，或将当前请求重定向到另一个数据类型。

请求发送器是一个对象，含有方法 `send(requestHeaders,done)` 和 `abort`，用于发送和取消请求。在 `jQuery.ajax` 中会根据数据类型获取相应的请求发送器，然后发送或取消请求。

##### 初始化

1. 在 Ajax 模块顶部，初始化 `prefilters`、`transports` 为空对象。
2. 初始化 `jQuery.ajaxPrefilter`、`jQuery.ajaxTransport` 为 `addToPrefiltersOrTransports`。
3. 使用第二步初始化的添加方法，添加内部的前置过滤器和请求发送器。内置了 `json`、`jsonp`、`script` 三个前置过滤器，以及 `script`、普通请求两种请求发送器。

###### `addToPrefiltersOrTransports`

`jQuery.ajaxPrefilter`、`jQuery.ajaxTransport` 添加前置过滤器和请求发送器实际上调用的是这个方法。

1. 传入需要添加到的目标对象，然后返回一个匿名函数来操作这个目标对象。
2. 对于返回的匿名函数如果没有传入 `dataTypeExpression`，那么表示 `*` 匹配其它所有请求。
3. 将 `dataTypeExpression` 按空格分割为数组。
4. 如果待插入的是函数则处理，遍历数组。
5. 将其插入目标对象的的指定类型的键中，如果数据以 `+` 开头则插在头部。

##### 执行

###### `inspectPrefiltersOrTransports`

1. 根据 `options.dataTypes[0]` 调用内部函数 `inspect`，如果没有返回值，则调用 `inspect('*')`
2. 在 `inspect` 内部，会遍历 `dataType` 对应的前置过滤器或请求发送器对象，调用前置过滤器或工厂函数。
3. 如果当前是在应用前置过滤器，且返回的是 `字符串`，那么递归调用。返回的是 `undefined`。
4. 如果当前是在应用请求发送器，返回的是请求发送器。

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
 - `url`：如果传入了 `url` 则使用，否则使用配置中的 `s.url`，再否则就是用 `ajaxLocation`，并去掉路径中的哈希，修正协议为当前使用的协议即与 `ajaxLocParts[0]` 相同。
 - `type`：依次从 `option.method`、`option.type`、`s.method`、`s.type` 中取。
 - `dataTypes`：根据空格切割为数组，没有则为 `*`。
 - `crossDomain`：利用正则求出上面修正后的 `url` 的协议、域名、端口，并与 `ajaxLocParts` 比较，得出是否跨域。
 - `data`：如果 `data` 不是字符串，并且 `s.processData` 设置为 `true`，那么 jQuery 会调用 `jQuery.param( s.data, s.traditional );` 帮我们修正对象为字符串。
6. 应用前置过滤器。见上文。
7. 如果在前置过滤器中终止，那么直接返回 `jqXHR`。
8. 如果没有禁用全局事件且没有其他 Ajax 请求在执行，那么触发 `ajaxStart` 事件。
9. 继续修正参数。
 - `type`：改为大写。
 - `hasContent`：是否有请求内容，GET、HEAD 没有。
10. 缓存 `url` 到 `cacheURL`。
11. 如果 `hasContent` 为 `false`，那么特殊处理 GET、HEAD 请求。
 1. 如果设置了 `s.data`，那么将其拼到 url 后面。
 2. 如果禁用了缓存 `s.cache=false`，那么通过添加一个时间戳参数的方式来使缓存无效。
12. 设置请求头。 
 - 如果 `s.ifModified===true` 那么设置缓存头。缓存头被存在 `jQuery.lastModified`、`jQuery.etag` 中，以 `cacheURL` 为键存储。
 - 如果 `s.data && s.hasContent && s.contentType !== false || options.contentType`，那么需要设置请求头 `Content-Type` 为 `s.contentType`，表示发送到服务器的内容类型。
 - 设置 `Accept`，根据 `dataTypes[0]` 到 `s.accepts` 中获取，表示该请求想要接收的响应类型。
 - 最后把 `s.headers` 中的内容设置到头部。这里面都是调用 `jqXHR.setRequestHeader` 进行设置。
13. 调用 `s.sendBefore`，这是发送请求前最后能干预的请求的地方了。
14. 添加成功、失败、完成的回调函数。
15. `inspectPrefiltersOrTransports` 获取请求发送器。
16. 如果没有得到请求发送器，那么调用 `done( -1, "No Transport" );` 自动终止。
17. 否则，修改 `jqXHR.readyState=1`，触发全局事件 `ajaxSend`，设置定时器，调用 `transport.send( requestHeaders, done );` 发送请求。
18. 返回 `jqHXR` 对象。

###### 回调函数。
 
1. 如果 `state` 已经为 2，直接返回。 
2. 设置 `state` 为 2。 
3. 清空定时器、缓存的响应头字符串、请求发送器。
4. `readyState` 改为 4，根据状态得到 `isSuccess`。
5. 读取响应数据 `ajaxHandleResponses`。
6. 数据类型转换 `ajaxConvert`