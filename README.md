# 微信端单页面应用（SPA）常见问题汇总及解决方案

#### 这事非常重要：

1. 路由的hash务必是“#”，如：http://example.com/wx/#/home/index 
2. 涉及**微信支付**的应用部署目录务必是二级或三级，建议通过修改Nginx、Apache配置重写url实现，或者修改Webpack的配置实现。
3. 新建一个页面用于微信授权登录，如：在根目录static文件夹下新建[auth.html](https://github.com/Chooin/wechat-spa/blob/master/examples/auth) （所有需要进入SPA应用的url地址都要通过该页面进行跳转，如：微信分享，菜单）
4. 涉及调用jsapi的页面都得重新配置wx.config
5. 流程图：
<img src="https://github.com/Chooin/wechat-spa/blob/master/pictures/flow.png" width="780" height="auto" />

#### 目录：

- [安装和使用微信js-sdk](#安装和使用微信js-sdk)
- [配置wx.config](#配置wx.config)
- [标题无法更新](#标题无法更新)
- [微信授权登录](#微信授权登录)
- [微信分享](#微信分享)
- [微信支付](#微信支付)

## 安装和使用微信js-sdk

1. npm安装

```
npm install weixin-js-sdk --save
```

2. ES6使用

```
import wx from 'weixin-js-sdk'
```

注：也可以通过修改Webpack配置解决，具体请自行查资料。

## 配置wx.config

非支付页面使用window.location.href.split('#')[0]，支付页面使用window.location.href，将得到的url传递给服务端进行签名操作

参考：[微信分享](#微信分享)

## 标题无法更新

在切换页面路由之后需在body里面添加iframe，随后移除掉iframe即可，代码如下

```
// iPhone，iPod，iPad下无法更新标题
if (/ip(hone|od|ad)/i.test(window.navigator.userAgent)) {
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
通过微信菜单或微信分享访问SPA应用需先访问授权登录页面(如先访问：http://example.com/static/auth.html )，在授权登录页面设置token等信息后再跳回到index.html文件所在的根目录下(如：http://example.com/wx/ )，然后利用SPA路由的钩子跳转到实际要访问的地址。

授权登录页面参考：[auth.html](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

## 微信分享

分享的url务必是 http://example.com/static/auth.html?redirect_uri={SPA应用部署的地址}&fullPath={访问的路由} ，

SPA应用部署的地址，如：http://example.com/wx/ ；访问的路由，如：/product 或 /product?page=1。两个值都需要通过encodeURIComponent转成url编码

参考代码如下：

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
          appId: res.data.appId,
          timestamp: res.data.timestamp,
          nonceStr: res.data.nonceStr,
          signature: res.data.signature,
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
  const share = ({title, desc, fullPath, imgUrl}) => {
    let url = window.location.href
    let redirect_uri = encodeURIComponent(url.split('#')[0])
    let fullpath = encodeURIComponent(fullPath)
    let link = `http://example.com/auth.html?redirect_uri=${redirect_uri}&fullPath=${fullPath}`
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

  return {
    config,
    share
  }
}

// 调用分享
_wechat().config().then(res => {
  // 分享信息
  let title = 'wechat-spa'
  let desc = 'Wechat SPA'
  let fullPath = '/home/index'
  let imgUrl = 'https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_120x44dp.png'
  // 配置分享
  _wechat().share({
    title,
    desc,
    fullPath,
    imgUrl
  })
}, err => {
  console.warn(err)
})
```

## 微信支付

造成支付失败的原因：iOS识别支付安全目录路径规则是进入SPA应用的第一个页面所对应的url。

**如：** 我们进入SPA应用的第一个页面是 http://example.com/wx/#/home/index 或是 http://example.com/wx/#/me/index ，则iOS获取到的安全目录路径分别是 http://example.com/wx/#/home/index 、 http://example.com/wx/#/me/index 。这样我们要配置很多的安全目录路径，但微信平台仅允许设置3个安全目录路径，直接进入SPA应用的页面是行不通的。

**解决思路：** 我们进入SPA应用的第一个页面都是 http://example.com/wx/ 然后通过SPA路由的钩子重定向到自己想要访问的页面。

## 联系方式

有质疑、有误的，反馈给我，谢谢

QQ：465353876
