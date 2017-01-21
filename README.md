## 单页面应用（SPA）在微信端的问题与解决方案

- [安装和使用微信jsdk](#安装和使用微信jsdk)
- [标题无法更新](#标题无法更新)
- [Oauth2授权登录](#Oauth2授权登录)
- [微信分享](#微信分享)
- [支付安全目录](#支付安全目录)

### 一、安装和使用微信jsdk
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
### 二、标题无法更新
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

### 三、Oauth2授权登录
1. hash统一用"#"，如：http://localhost:8080/#/home/index
2. 微信菜单或者分享页面进入网站时做统一的授权页面，如访问 http://localhost:8080/#/home/index 页面，则先访问 http://localhost:8080/static/auth.html?redirect=http%3a%2f%2flocalhost%3a8080%2f%23%2fhome%2findex ，token等信息在auth.html页面处理完成后再跳转到要访问的页面

### 四、微信分享
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

### 五、支付安全目录
参考：[Oauth2授权登录](#Oauth2授权登录)的微信授权登录方案
注：每次切换路由Android与iOS微信所获取到的安全路径不同