完成小程序开发前的[准备工作](https://ninghao.net/blog/5036)，就可以去学习开发微信小程序了，主要就是学习小程序这套框架，它跟一般的前端框架非常像，比如 React，Vue 等等。开发用的语言是 JavaScript，在本地用微信开发者工具开发与调试， 在模拟器上可以预览小程序的运行。想在手机微信里运行小程序，可以把代码推送给微信，然后手机扫码，有权限的用户就可以预览小程序了，整个开发过程很友好。

## 注册小程序

小程序需要一个主要的逻辑文件，一个配置文件，还有一个样式文件。在主逻辑文件里你得用 App\(\) 函数去注册一个小程序，注册的时候里面可以添加一些初始化的数据，小程序的生命周期方法，还有自定义的方法。在小程序的配置文件里可以使用小程序框架规定的一些配置选项，去配置我们的小程序。小程序可以有个主要的样式文件，在这里去定义在程序页面上需要共用的一些样式。

## app.js

app.js 是小程序的主要逻辑文件，在这里需要用 App\(\) 注册一个小程序。这个文件需要在小程序的根目录的下面。

```
App({})
```

注册的时候给它一个对象参数，在这个对象里添加小程序里面需要的初始化数据，生命周期，还有自定义的方法。

### 初始化数据

下面添加了一个自定义的数据叫 globalData，它里面添加了一个 greeting ，对应的值是 hello ~ 。在小程序的页面上，你可以使用这个初始化数据。

```
App({
  globalData: {
    greeting: 'hello ~'
  }
})
```

### 生命周期

生命周期就是小程序在不同的阶段要做的一些事情，你可以告诉小程序在什么时候要做什么事。onLaunch，onShow，onHide，还有 onError。

比如你想在小程序初始化以后做些事情，可以在注册小程序的方法里添加一个 onLaunch 方法。onHide 会在小程序进入到后台以后被执行，进入前台以后又会执行 onShow。

```
App({
  onLaunch() {
    console.log('初始化')
  }
})
```

### getApp

在小程序的页面上，你可以使用 getApp\(\) 得到小程序实例，得到以后你可以访问小程序的初始化数据，自定义方法等等。

```
const app = getApp()
console.log(app.globalData.greeting)
```

## app.json

app.json 是小程序的配置文件，用小程序提供的配置选项你配置小程序，比如小程序里的页面，窗口的样式，标签栏等等。这个文件需要在小程序的根目录的下面。

**示例：**

```
{
  "pages": [
    "pages/wxml/wxml",
    "pages/index/index",
    "pages/event/event"
  ],
  "window": {
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "ninghao.net",
    "navigationBarTextStyle": "black",
    "enablePullDownRefresh": true,
    "backgroundTextStyle": "dark",
    "backgroundColor": "#ededed"
  },
  "tabBar": {
    "color": "rgba(0, 0, 0, 0.6)",
    "selectedColor": "#000000",
    "borderStyle": "white",
    "backgroundColor": "#ededed",
    "list": [
      {
        "text": "首页",
        "pagePath": "pages/index/index",
        "iconPath": "assets/icons/home.png",
        "selectedIconPath": "assets/icons/home-active.png"
      },
      {
        "text": "视图",
        "pagePath": "pages/wxml/wxml",
        "iconPath": "assets/icons/web.png",
        "selectedIconPath": "assets/icons/web-active.png"
      },
      {
        "text": "活动",
        "pagePath": "pages/event/event",
        "iconPath": "assets/icons/event.png",
        "selectedIconPath": "assets/icons/event-active.png"
      }
    ]
  },
  "debug": true
}

```

pages 里面配置了小程序里包含的页面，window 配置了小程序的窗口相关的样式，tarBar 给小程序配置了一个标签栏。debug 打开了小程序的调试功能，这样会在小程序的开发者工具的调试控制台，会输出一些有用的信息。

## app.wxss

wxss（weixin style sheets） 是小程序里用的样式，跟咱们平时用的 CSS 差不多，只不过微信定制了一下，添加了一些自己的东西。基本上你可以把它想成是我们平时用的 CSS 。app.wxss 是小程序的主样式文件，这个文件要放在小程序的根目录的下面。这个样式文件里的东西，在小程序的其它地方都可以用到。小程序里的每个页面也都可以拥有属于自己的样式文件。

**示例：**

```
.container {
  box-sizing: border-box;
  height: 80vh;
  padding: 8px;
  background: #f8f8f8;
  color: rgba(0, 0, 0, 0.85);
}
```



