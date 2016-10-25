## 构造 jQuery 对象

jQuery 对象是一个类数组对象，有连续的整型属性、`length` 属性和大量的 jQuery 方法。它由构造函数 `jQuery()` 创建，简写`$()`。

### jQuery 构造函数

###### `jQuery(selector[, context])`

传入选择器及可选的上下文，jQuery 会根据上下文或者 `document` 查找匹配的 DOM 元素，找到则返回一个包含了匹配元素的 jQuery 对象。没有匹配元素那么会返回一个空的 jQuery 对象，`length` 为 0。因此判断 jQuery 对象是否为空，并不是与 `null` 进行比较，而是看 `length` 是否为 0。

###### `jQuery(html[, ownerDocument])`

和上面的一样，同样是传入一个字符串参数，jQuery 会判断这个字符串参数是选择器还是 HTML 代码，然后进行不同的处理。如果第一个参数时单个标签，那么会使用 `document.createElement` 来创建一个 DOM 元素，否则会利用 `innerHTML` 的机制来创建 DOM 元素。第二个参数表示新创建 DOM 元素的文档对象，默认为当前文档。

###### `jQuery(html, props)`

这个方法实际上是和上面方法类似的，第一个参数时单个标签时，第二个参数可以使一组属性值，然后由 `attr` 方法把属性和事件设置到新创建的 DOM 元素上。

###### `jQuery(element)`、`jQuery(elementArray)`

传入 DOM 元素或则元素数组，封装到 jQuery 对象中并返回。常用于事件监听中把 `this` 分装成 jQuery 对象。

###### `jQuery(object)`

传入普通的 js 对象，分装到 jQuery 对象中返回。可以方便地在普通 js 对象上实现自定义事件的绑定和触发。

###### `jQuery(callback)`

`callback` 绑定为 `ready` 事件的一个监听函数，当 DOM 解析完成时，会触发 `ready` 事件，然后调用这个监听函数。

> 原本 `ready` 指得是 `DOMContentLoaded` 事件，但是对于 IE 6、7、8，jQuery 使用 `readyStateChange` 来进行兼容，IE 11 弃用了 `readyStateChange` ，jQuery 使用 `doScroll` 来进行兼容。因此此处的 `ready` 本质上是这三种兼容方式的统称。

###### `jQuery(jQuery object)`

创建一个副本返回，两者引用的是同一个 DOM 元素。

###### `jQuery()`

返回一个空的 jQuery 对象，`length` 为 0。

### jQuery 无 `new` 构建实例的实现

```
var jQuery = function( selector, context ) {
		// 实际上是在内部用 new 调用了另一个构造函数
		return new jQuery.fn.init( selector, context, rootjQuery );
	}
jQuery.fn = jQuery.prototype = {
	// The current version of jQuery being used
	jquery: core_version,

	constructor: jQuery,
	init: function( selector, context, rootjQuery ) {
		// ...
	},
	...
}
// 通过覆盖原型的方式，把 jQuery.prototype 覆盖到 jQuery.fn.init.prototype 上
jQuery.fn.init.prototype = jQuery.fn;
// ...
```

###### 为什么要无 `new` 构造 ？

个人认为比较简洁吧。underscore 也选择的是无 `new` 构造对象。

###### 为什么要有 `jQuery.fn` ？

比起 `prototype` 省了 7 个字符。

###### 为什么使用 `jQuery.fn.init` 创建的对象能够访问到 jQuery.prototype 上的属性和方法？

`jQuery.fn.init.prototype = jQuery.fn;` 将真正的构造函数 `jQuery.fn.init` 的原型设为了 `jQuery.fn` 也就是 `jQuery.prototype`。

### `jQuery.fn.init(selector, context, rootjQuery)`

看了上一小节，我们知道了 jQuery 实际是调用 `jQuery.fn.init` 构造的对象，接下来分析 `jQuery.fn.init` 来看看它是如何实现第一小节讲到的 7 种构造函数的。代码有点长，不贴了，把处理的分支写一下：

1. `selector` 可以转换为 `false`：`$("")`、`$(null)`、`$(undefined)`、`$(false)`，直接返回 `this`，即空的 jQuery 对象。
2. `selector` 是字符串。
 1. 
3. `selector` 是 DOM 元素，设置第一个元素及 `context` 属性为该 DOM 元素，`length` 为 1。 
4. `selector` 是函数：`return rootjQuery.ready(selector);`。
5. `selector` 是 jQuery 对象，把 `selector`、`context` 设置给 `this`，在后面的 `return jQuery.makeArray(selector, this)` 中会把包含的元素复制到当前的 jQuery 对象中。
6. 其它任意类型值：`return jQuery.makeArray(selector, this)`。

