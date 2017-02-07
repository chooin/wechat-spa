# 微信端单页面应用（SPA）常见问题汇总及解决方案

- [安装和使用微信js-sdk](#安装和使用微信js-sdk)
- [标题无法更新](#标题无法更新)
- [授权登录](#授权登录)
- [微信分享](#微信分享)
- [支付安全目录](#支付安全目录)

## 安装和使用微信js-sdk
1. npm安装
<pre>
npm install weixin-js-sdk --save
</pre>
2. ES6使用
<pre>
import wx from 'weixin-js-sdk'
</pre>

## 标题无法更新
在切换页面路由之后需在body里面添加iframe，随后移除掉iframe即可，代码如下
#### ES6
<pre>
// iPhone，iPod，iPad下无法更新标题
if (/ip(hone|od|ad)/i.test(navigator.userAgent)) {
  let iframe = document.createElement('iframe')
  let body = document.querySelector('body')
  iframe.style.display = 'none'
  iframe.src = '/favicon.ico'
  iframe.onload = () => {
    setTimeout(() => {
      iframe.remove()
    }, 10)
  }
  body.appendChild(iframe)
}
</pre>

## 授权登录
1. hash统一用"#"，如http://localhost:8080/#/home/index
2. 新建一个页面单独用于微信授权登录，如：在网站根目录下新建auth.html
3. 用户首次访问网站需先访问授权登录页面，在授权登录页面设置好相关信息后再跳回实际要访问的页面，如：用户访问http://localhost:8080/#/home/index页面，则先访问http://localhost:8080/auth.html?redirect_uri=http%3a%2f%2flocalhost%3a8080%2f%23%2fhome%2findex，在auth.html页面做微信的授权登录及token等信息的设置，然后跳回

案例参考：[授权登录页面](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

## 微信分享
每次切换路由的时候都重新配置微信js-sdk，代码如下
#### ES6
<pre>
// 微信config设置
const wechatConifg = () => {
  return new Promise((resolve, reject) => {
    // 获取服务端微信配置信息
    axios.get(`${urls.api}/wechats/config`).then(res => {
      wx.config({
        debug: false,
        appId: res.appId,
        timestamp: res.timestamp,
        nonceStr: res.nonceStr,
        signature: res.signature,
        jsApiList: [
          'onMenuShareTimeline',
          'onMenuShareAppMessage'
        ]
      })
      resolve('wechat config success')
    }, () => {
      reject('wechat config fail')
    })
  })
}

// 分享设置
wechatConifg(() => {
  wx.ready(() => {
    wx.onMenuShareTimeline({
      ..
    })
    wx.onMenuShareAppMessage({
      ..
    })
  })
}, (err) => {
  console.warn(err)
})
</pre>

## 支付安全目录
参考：[授权登录](#授权登录)的微信授权登录方案

注：每次切换路由Android与iOS微信所获取到的安全路径不同