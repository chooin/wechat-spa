# 微信端单页面应用（SPA）常见问题汇总及解决方案

#### 这事非常重要：

1. 路由的hash务必是“#”，如：http://example.com/#/home/index ，**微信支付**必须二级或三级目录，如：http://example.com/wx/#/home/index
2. 新建一个页面用于微信授权登录（包括微信分享），如：在网站根目录下新建auth.html
3. 涉及调用jsapi的页面都得重新配置wx.config

#### 目录：

- [安装和使用微信js-sdk](#安装和使用微信js-sdk)
- [配置wx.config](#配置wx.config)
- [标题无法更新](#标题无法更新)
- [微信授权登录](#微信授权登录)
- [微信分享](#微信分享)
- [微信支付](#微信支付)

## 安装和使用微信js-sdk

1.npm安装

```
npm install weixin-js-sdk --save
```

2.ES6使用

```
import wx from 'weixin-js-sdk'
```

## 配置wx.config

配置wx.config一般页面使用window.location.href.split('#')[0]，支付页面使用window.location.href

参考：[微信分享](#微信分享)

## 标题无法更新
在切换页面路由之后需在body里面添加iframe，随后移除掉iframe即可，代码如下
```
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
```

## 微信授权登录
用户首次访问网站需先访问授权登录页面，在授权登录页面设置好相关信息后再跳回实际要访问的页面，如：用户访问 http://example.com/#/home/index 页面，则先访问 http://example.com/auth.html?redirect_uri=http%3a%2f%2flocalhost%3a8080%2f%23%2fhome%2findex ，在auth.html页面完成授权登录、token等信息的写入，然后跳回到实际访问的页面

案例参考：[微信授权登录](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

## 微信分享
分享的uri务必是 http://example.com/auht.html?redirect_uri=实际要访问的地址 ，配置好token等信息然后再跳回到实际要访问的地址，代码如下：
```
import wx from 'weixin-js-sdk'
import axios from 'axios'

cosnt _wechat = () => {
  // wx.config配置
  const config = () => {
    return new Promise((resolve, reject) => {
      // 获取服务端微信配置信息
      axios.get('http://api.example.com/wechat/config', {
        params: {
          url: window.location.href.indexOf('/cart/payment') > 0 ? window.location.href : window.location.href.split('#')[0]
        }
      }).then(res => {
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

  // 分享配置
  const share = ({title, desc, link, imgUrl}) => {
    let link = `http://example.com/auth.html?redirect_uri=${encodeURIComponent(link)}`
    wx.ready(() => {
      wx.onMenuShareTimeline({
        title,
        link,
        imgUrl,
        success () {},
        cancel () {}
      })
      wx.onMenuShareAppMessage({
        title,
        desc,
        link,
        imgUrl,
        success () {},
        cancel () {}
      })
    })
  }

  return {config, share}
}

// 调用分享
_wechat().config().then(res => {
  _wechat().share({
    title,
    desc,
    link,
    imgUrl
  })
}, err => {
  console.warn(err)
})
```

## 微信支付
1.进入支付页面将hash从“#”设置成“?#”，如：原来支付页面：http://example.com/wx/#/cart/payment ,修改后的页面：http://example.com/wx/?#/cart/payment 

方法一（推荐）

```
if (window.location.href.indexOf('?#') < 0) {
  window.history.pushState({}, '', '?#/cart/payment')
}
.. // 业务代码
```

方法二

```
if (window.location.href.indexOf('?#') < 0) {
  window.location.href = window.location.href.replace('#', '?#')
} else {
  .. // 业务代码
}
```

2.完成支付操作后重新将“?#”重新设置成“#”，代码如下

方法一（推荐）

```
if (window.location.href.indexOf('?#') > 0) {
  window.history.pushState({}, '', window.location.href.replace('?#', '#'))
}
.. // 业务代码
```

方法二

```
if (window.location.href.indexOf('?#') > 0) {
  window.location.href = window.location.href.replace('?#', '#')
} else {
  .. // 业务代码
}
```

注：每次切换路由Android与iOS所获取到的支付安全目录不同，刷新支付页面可以解决