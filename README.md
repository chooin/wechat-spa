## 标题无法更新
在切换页面路由之后执行以下代码
<pre>
<code>
let iframe = document.createElement('iframe')
let body = document.querySelector('body')
iframe.style.display = 'none'
iframe.src = '/favicon.ico'
body.appendChild(iframe)
setTimeout(() => { iframe.remove() }, 300)
</code>
</pre>