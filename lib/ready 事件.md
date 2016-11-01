## ready 事件

在第二章的时候我们略过了 `jQuery.ready` 这个方法，在看完事件系统以后，来补一个番外。

### 什么是 ready 事件 ？

ready 事件是一种更普遍的说法，在 IE 9+ 以及其它现代浏览器中就是 `DOMContentLoaded` 事件，这个事件会在 DOM 结构加载完毕时触发。通常在这个时候我们的 Js 就可以执行了。

而在 IE 9 以下，一般会用 `onreadystatechange` 事件来代替，或者 `document.docuementElement.doScroll('left')` 来兼容，我们把 IE 9 以下 `onreadystatechange` 事件和 `DOMContentLoaded` 事件统一叫做 `ready` 事件，本质上没有改变，还是代表了 DOM 结构加载完毕。

另外，load 事件在 ready 后触发，load 事件要求资源都加载完毕。

### jQuery 兼容实现