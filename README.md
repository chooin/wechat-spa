# 单页面应用（SPA）在微信端的问题与解决方案

- [安装和使用微信js-sdk](#安装和使用微信js-sdk)
- [标题无法更新](#标题无法更新)
- [Oauth2授权登录](#Oauth2授权登录)
- [微信分享](#微信分享)
- [支付安全目录](#支付安全目录)

## 安装和使用微信js-sdk
1. npm安装微信js-sdk
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
    }, 9)
  }
  body.appendChild(iframe)
}
</pre>

## Oauth2授权登录
1. hash统一用"#"，如：http://localhost:8080/#/home/index
2. 微信菜单或者分享页面进入网站时做统一的授权页面，如访问 http://localhost:8080/#/home/index 页面，则先访问 http://localhost:8080/static/auth.html?redirect=http%3a%2f%2flocalhost%3a8080%2f%23%2fhome%2findex ，token等信息在auth.html页面处理完成后再跳转到要访问的页面

参考：[统一授权登录](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

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
参考：[Oauth2授权登录](#Oauth2授权登录)的微信授权登录方案

注：每次切换路由Android与iOS微信所获取到的安全路径不同