### 安装和使用微信jsdk
1. npm安装微信jsdk
<pre>
<code>
npm install weixin-js-sdk --save
</code>
</pre>

2. ES6使用
<pre>
<code>
import wx from 'weixin-js-sdk'
</code>
</pre>
### 一、标题无法更新
在切换页面路由之后需在body里面添加iframe，随后移除掉iframe即可，代码如下
#### ES6
<pre>
<code>
let iframe = document.createElement('iframe')
let body = document.querySelector('body')
iframe.style.display = 'none'
iframe.src = '/favicon.ico'
body.appendChild(iframe)
setTimeout(() => { iframe.remove(); }, 300)
</code>
</pre>
#### ES5
<pre>
<code>
var iframe = document.createElement('iframe')
var body = document.querySelector('body')
iframe.style.display = 'none'
iframe.src = '/favicon.ico'
body.appendChild(iframe);
setTimeout(function () { iframe.remove(); }, 300)
</code>
</pre>

### 二、Oauth2授权登录
1. hash统一用"#"，如：http://localhost:8080/#/home/index
2. 微信菜单或者分享页面进入网站时做统一的授权页面，如访问 http://localhost:8080/#/home/index 页面，则先访问 http://localhost:8080/static/auth.html?redirect=http%3a%2f%2flocalhost%3a8080%2f%23%2fhome%2findex ，token等信息在auth.html页面处理完成后再跳转到要访问的页面

### 三、微信分享
每次切换路由的时候都重新配置微信jsdk，代码如下
#### ES6
<pre>
<code>
wx.config(() => {
  wx.onMenuShareTimeline({
    ..
  })
  wx.onMenuShareAppMessage({
    ..
  })
})
</code>
</pre>
#### ES5
<pre>
<code>
wx.config(function () {
  wx.onMenuShareTimeline({
    ..
  })
  wx.onMenuShareAppMessage({
    ..
  })
})
</code>
</pre>

### 四、支付安全目录
参考：[Oauth2授权登录](#Oauth2授权登录)的微信登录方案