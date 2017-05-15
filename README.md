# 微信端单页面应用（SPA）常见问题汇总及解决方案

#### 🌈 这事非常重要：

1. 路由启用 hash 模式，hash 务必是“#”，如：http://example.com/wx/#/home/index
2. 涉及**微信支付**的应用部署目录务必是二级或三级，建议通过修改 Nginx、Apache 配置重写 url 实现，或者修改 webpack 的配置实现
3. 新建一个页面用于微信授权登录，如：在根目录 static 文件夹下新建 [auth.html](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)（所有需要进入 SPA 应用的 url 地址都要通过该页面进行跳转，如：微信分享，菜单）
4. 涉及调用 jsapi 的页面都得重新配置 wx.config
5. 不要缓存 SPA 应用入口页面，给入口页面的 head 部分添加以下代码
``` bash
# 防止页面缓存
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="0">
<meta http-equiv="pragma" content="no-cache">
```
6. 从分享链接或微信公众号菜单进入 http://example.com/wx/#/home/index 页面，流程图如下：
<img src="https://github.com/Chooin/wechat-spa/blob/master/pictures/flow.png" width="780" height="auto" />

#### 目录：

- [安装和使用微信 js-sdk](#安装和使用微信js-sdk)
- [配置 wx.config](#配置wx.config)
- [标题更新](#标题更新)
- [微信授权登录](#微信授权登录)
- [微信分享](#微信分享)
- [微信支付](#微信支付)
- [禁忌](#禁忌) ❗❗❗

## 安装和使用微信js-sdk

1. npm 安装

```
npm install weixin-js-sdk --save
```

2. ES6 使用

```
import wx from 'weixin-js-sdk'
```

注：也可以通过修改 webpack 配置解决，具体请自行查资料。

## 配置wx.config

非支付页面使用 window.location.href.split('#')[0]，支付页面使用 window.location.href，将得到的 url 传递给服务端进行签名操作

参考：[微信分享](#微信分享)

## 标题更新

在切换页面路由之后需在 body 里面添加 iframe，随后移除掉 iframe 即可，代码如下

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
通过微信菜单或微信分享访问 SPA 应用需先访问授权登录页面(如先访问：http://example.com/static/auth.html )，在授权登录页面设置 token 等信息后再跳回到 index.html 文件所在的根目录下(如：http://example.com/wx/ )，然后利用 SPA 应用路由的钩子跳转到实际要访问的地址。

授权登录页面参考：[auth.html](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

## 微信分享

分享的 url 务必是 http://example.com/static/auth.html?redirect_uri={SPA应用部署的地址}&full_path={访问的路由} ，redirect_uri 对应 SPA 应用部署的地址，如：http://example.com/wx/ ；full_path 对应访问的路由，如：/product 或 /product?page=1，这两个参数都需要通过 encodeURIComponent 转码

参考代码如下：

```
import wx from 'weixin-js-sdk'
import axios from 'axios'

cosnt _wechat = () => {
  // wx.config配置
  const config = () => {
    return new Promise((resolve, reject) => {
      // 获取服务端微信配置信息
      axios.get('http://api.example.com/v1/wechat/config', {
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
        wx.ready(() => { resolve(wx) }) // 配置wx.config成功
      }, () => {
        reject('配置wx.config失败')
      })
    })
  }

  // 分享配置
  const share = ({ title, desc, fullPath, imgUrl }) => {
    let url = window.location.href
    let redirectUri = encodeURIComponent(url.split('#')[0])
    let fullPath = encodeURIComponent(fullPath)
    let link = `http://example.com/static/auth.html?redirect_uri=${redirectUri}&full_path=${fullPath}`
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
  }

  return {
    config,
    share
  }
}

// 调用分享
_wechat().config().then(res => {
  // 配置分享
  _wechat().share({
    title: 'wechat-spa',
    desc: 'Wechat SPA',
    fullPath: '/home/index',
    imgUrl: 'https://www.baidu.com/img/bd_logo1.png'
  })
}, err => {
  console.warn(err)
})
```

## 微信支付

造成支付失败的原因：iOS 识别支付安全目录路径规则是进入 SPA 应用的第一个页面所对应的 url。

**如：** 我们进入 SPA 应用的第一个页面是 http://example.com/wx/#/home/index 或是 http://example.com/wx/#/me/index ，则 iOS 获取到的安全目录路径分别是 http://example.com/wx/#/home/index 、 http://example.com/wx/#/me/index 。这样我们要配置很多的安全目录路径，但微信平台仅允许设置3个安全目录路径，直接进入 SPA 应用的页面是行不通的。

**解决思路：** 我们进入 SPA 应用的第一个页面都是 http://example.com/wx/ 然后通过 SPA 应用路由钩子重定向到自己想要访问的页面。

## 禁忌
1. 不要使用类似以下格式的 url
``` bash
# 点击返回到上个页面，页面无法渲染
http://example.com/?time=1494315429992#/home/index

```

2. 路由不要使用 history 模式

## 问题反馈

内容有误请反馈给我，谢谢

有什么问题可以加我好友，大家一起交流

QQ：465353876
