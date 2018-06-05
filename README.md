# 微信端单页面应用（SPA）常见问题汇总及解决方案

#### 🌈 这事非常重要：

- 路由启用 hash 模式，hash 务必是 `#`，如：https://example.com/#/home/index

  > 采用 history 模式将无法复制出当前的 URL

- 参数请用 `?` 的形式获取，如：https://example.com/#/home/index?search=content

  > 不采用 `?` 的形式获取参数会产生很多支付安全目录

-  新建一个页面用于微信授权登录，如：在根目录 static 文件夹下新建 [auth.html](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

  > 解决很多支付安全目录的问题

- Nginx 配置

  ```
  add_header "Cache-Control" "no-cache, private";
  ```

  > 解决 window.location.href 跳转页面被浏览器缓存的问题

- 涉及调用 jsapi 的页面都得重新配置 wx.config

#### 完整流程图如下：

<img src="https://github.com/Chooin/wechat-spa/blob/master/pictures/flow.png" width="780" height="auto" />

#### 目录：

- [安装和使用微信 js-sdk](#安装和使用微信js-sdk)
- [标题更新](#标题更新)
- [微信授权登录](#微信授权登录)
- [微信分享](#微信分享)
- [微信支付](#微信支付)
- [白屏](#白屏)
- [禁忌](#禁忌) ❗❗❗

#### 安装和使用微信js-sdk

##### 方法 1 (推荐)

在入口 index.html 文件引入微信的 js-sdk 文件，webpack 配置参考：[中文](https://webpack.docschina.org/configuration/externals/) / [English](https://webpack.js.org/configuration/externals/)

index.html
  
``` html
<script src="//res.wx.qq.com/open/js/jweixin-1.2.2.js"></script>
```
  
webpack.config.js
  
``` js
externals: {
  wx: 'wx'
}
```
  
如何使用：
  
``` js
import wx from 'wx'
  
wx.ready(() => {
  console.log('hello Wechat!')
})
```

##### 方法 2

如何安装：

``` sh
npm install weixin-js-sdk --save
# 或
yarn add weixin-js-sdk
```

如何使用：

``` js
import wx from 'weixin-js-sdk'

wx.ready(() => {
  console.log('Hello Wechat!')
})
```

#### 标题更新

在切换页面路由之后需在 body 里面添加 iframe，随后移除掉 iframe 即可，代码如下

``` js
// iPhone，iPod，iPad 下无法更新标题
if (/ip(hone|od|ad)/i.test(window.navigator.userAgent)) {
  let iframe = document.createElement('iframe')
  iframe.style.display = 'none'
  iframe.src = '/favicon.ico'
  iframe.onload = () => {
    setTimeout(() => {
      iframe.remove()
    }, 10)
  }
  document.body.appendChild(iframe)
}
```

#### 微信授权登录
通过微信菜单或微信分享访问 SPA 应用需先访问授权登录页面(如先访问：https://example.com/static/auth.html )，在授权登录页面设置 token 等信息后再跳回到 index.html 文件所在的根目录下(如：https://example.com/wx/ )，然后利用 SPA 应用路由的钩子跳转到实际要访问的地址。

授权登录页面参考：[auth.html](https://github.com/Chooin/wechat-spa/blob/master/examples/auth)

#### 微信分享

分享配置都正确，进入链接后页面一直显示不对

> **解决方案：** 在分享的地址后面添加一个随机字符串，如：`/product/index?share_at=${Date.now()}`

微信分享参考代码如下：

``` js
import wx from 'wx'
import axios from 'axios'

const share = ({
  title,
  desc,
  fullPath,
  imgUrl
}) => {
  let link = fullPath.indexOf('?') > -1
    ? `${baseURL.APP}/#${fullPath}&share_at=${Date.now()}`
    : `${baseURL.APP}/#${fullPath}?share_at=${Date.now()}`
  wx.showAllNonBaseMenuItem()
  wx.onMenuShareTimeline({
    title,
    link,
    imgUrl
  })
  wx.onMenuShareAppMessage({
    title,
    desc,
    link,
    imgUrl
  })
}

const $_wechat = () => {
  return new Promise((resolve, reject) => {
    // 获取服务端微信配置信息
    axios.get('https://api.example.com/v1/wechat/config', {
      params: {
        url: window.location.href.split('#')[0]
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
      wx.ready(() => { // 配置wx.config成功
        resolve({
          wx,
          share
        })
      })
    }).catch(() => {
      reject('配置wx.config失败')
    })
  }
}

// 调用分享
$_wechat().then(res => {
  res.share({ // 配置分享
    title: 'wechat-spa',
    desc: 'Wechat SPA',
    fullPath: '/home/index',
    imgUrl: 'https://www.baidu.com/img/bd_logo1.png'
  })
}).catch(_ => console.warn(_))
```

#### 微信支付

造成支付失败的原因：iOS 识别支付安全目录路径规则是进入 SPA 应用的第一个页面所对应的 url，举个例子：

第一次进入的 URL | iOS 获取到的安全目录
--------- | --------
https://example.com/#/home/index | https://example.com/#/home/index
https://example.com/#/me/index | https://example.com/#/me/index
https://example.com/#/product/index | https://example.com/#/product/index

这样我们要配置很多的安全目录路径，但微信平台仅允许设置3个安全目录路径，直接进入 SPA 应用的页面是行不通的

> **解决思路：** 我们进入 SPA 应用的第一个页面都是 https://example.com/ 然后通过 SPA 应用路由钩子重定向到自己想要访问的页面。

#### 白屏

微信支付后立即跳转到其他页面有一定几率出现白屏（长按屏幕可以复制出文字或图片地址），解决方案：

``` js
// 延迟跳转即可解决
setTimeout(() => {
  window.location.replace('/payment/success') // 跳转逻辑
}, 500)
```

注：微信内置浏览器的 bug，图片无法批量上传也可以通过 `setTimeout` 方法解决

#### 问题反馈

如内容有误请反馈给我，谢谢

有什么问题可以加我好友，大家一起交流

QQ：465353876


