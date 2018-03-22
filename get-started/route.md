在小程序里的页面，你要在小程序的主配置文件 app.json 里面说明一下，这些页面可以放在 pages 属性的下面。在里面描述一下页面的地址就行了，比如 pages/index/index，这就是 index 页面的地址，这个页面应该在 pages 目录下面的 index 这个目录里，最后的 index 是页面的名字，这里不需要指定扩展名，小程序会自动给我们加上。

```
{
  "pages": [
    "pages/wxml/wxml",
    "pages/index/index",
    "pages/event/event",
    "pages/demo/demo"
  ],
}
```

## 页面导航

小程序内部页面与页面之间的链接，需要用 navigator 组件，组件里面包装的东西就是链接文字，组件上的属性可以配置链接属性，比如用 url 属性设置一下链接的地址。像这样：

```
<navigator url="/pages/demo/demo">demo</navigator>
```

在页面上会显示一个 demo 链接，点击这个链接打开的就是 demo 页面。链接上可以包含查询参数，这些参数在页面上可以得到，你可以把这些参数告诉应用的后端去处理。

```
<navigator url="/pages/demo/demo?id=1001">demo</navigator>
```

这样我们在 demo 页面上就可以得到 id 这个参数的值，就是 1001。我们可以在页面的 onLoad 生命周期里得到参数的值。

```
 onLoad(options) {
   console.log(options)
 }
```

onLoad 里的 options 里的东西就是一个对象，里面就是查询参数，还有对应的值。

## 链接的类型

链接的有几种类型，默认叫 navigate，就是点击链接，会打开新的页面，页面上会有一个返回按钮，用户可以点击返回打开之前的页面。

![](https://work.ninghao.net/wp-content/uploads/2017/08/Snip20170817_1.png)

小程序里的 wx.navigateTo 这个接口可以使用代码切换显示的页面。在开发者工具的调试界面上，可以在 Console 里试一下接口。

```
wx.navigateTo({
  url: '/pages/demo/demo?id=1001'
})
```

### 重定向

redirect，这种类型的链接会先关掉当前打开的页面，然后重定向到新的页面，在新页面上用户不能返回到他之前访问的页面。设置链接类型可以在 navigator 组件上添加一个 open-type 属性。

```
<navigator url="/pages/demo/demo?id=1001" open-type="redirect">demo</navigator>
```

![](https://work.ninghao.net/wp-content/uploads/2017/08/Snip20170817_2.png)

导航栏上没有返回按钮。

使用小程序的 wx.redirectTo 接口可以模拟打开 redirect 这种链接的行为。

```
wx.redirectTo({
  url: '/pages/demo/demo?id=1001'
})
```

### 切换标签

链接也可以切换标签，这种类型的链接是 switchTab 。

```
<navigator url="/pages/index/index" open-type="switchTab">demo</navigator>
```

点击 demo 这个链接，会把页面切换到 /pages/index/index 这个地址的标签。小程序里的 wx.switchTab 接口可以模拟打开 switchTab 类型的链接。

```
wx.switchTab({
  url: '/pages/index/index'
})
```

## 学习

[订阅宁皓网](https://ninghao.net/signup)，可以在线学习开发小程序。还可以学习相关的配套课程，比如 JavaScript 语言，ES2015 的写法，为小程序创建后端接口等等。后端服务接口可以存储小程序需要的数据，这个接口可以用 WordPress，Drupal，Laravel，Rails，Nodejs 去创建，这些东西你都可以在宁皓网找到相关课程。

