注册了小程序以后，就需要为小程序准备一些页面了。每个页面都有三个基本的东西，一个主逻辑文件（.js），一个视图文件（.wxml），还有一个样式文件（.wxss）。页面需要的这几个文件可以放在一个单独的目录的下面。

## 主逻辑

先创建页面的主逻辑文件，比如放在 pages/index/ 这个目录的下面，页面的主逻辑文件的名字可以是 index.js ，在这个文件里，用 Page\(\) 函数注册一个页面。

```
Page({})
```

注册的时候可以给它一个对象参数，在它里面可以添加页面的初始化数据，生命周期，事件处理，还有自定义的方法。

### 生命周期

页面也有自己的生命周期，就是页面在不同的阶段要做的一些事情。onLoad，onShow，onReady，onHide，onUnload。需要在什么阶段做事情，就在注册页面的 Page\(\) 里面添加一个对应的生命周期方法，这些方法会自动被执行。

```
Page({
  onLoad() {
    console.log('页面加载')
  }
})
```

### 事件处理

你可以在页面的视图上绑定自定义的事件，然后指定对应的事件处理，在页面的主逻辑里面，你可以定义事件处理方法。还有几个固定的事件处理，不需要特别在视图上绑定，比如下拉刷新页面（onPullDownRefresh），页面滚动到底部（onReachBottom），还有滚动页面（onPageScroll），都会触发对应的事件处理。

```
Page({
  onPullDownRefresh() {
    console.log('下拉刷新页面')
  },
  onReachBottom() {
    console.log('到底儿了~')
  },
  onPageScroll(calculations) {
    console.log(calculations)
  },
})
```

注意下拉刷新功能你需要在小程序的主配置里面打开。

```
{
 "window": {
   "enablePullDownRefresh": true,
  }
}
```

### 初始化数据

在注册页面用的 Page\(\) 函数里，添加一个 data 属性，这里的东西就是页面上的初始化的数据，在页面的视图文件里，可以直接使用这些初始化的数据。

```
const app = getApp()

Page({
  data: {
    greeting: app.globalData.greeting
  }
})
```

上面我们把小程序全局的数据 globalData 里的 greeting 交给了页面的一个初始化数据，就是 greeting。在小程序的视图文件里，我们可以绑定输出这个数据。

## 视图

小程序页面的视图文件是 wxml（weixin markup language），跟我们平时用的 HTML 非常像。小程序里定制了一些组件，这些组件就相当于是 HTML 里的元素，或者叫标签。

**示例：**

pages/index/index.wxml

```
<view>
  <text>{{ greeting }}</text>
</view>
```

上面用的 view，text 这些东西是小程序里的组件，在 text 组件里绑定了一个页面上的初始化数据，就是 data 里的东西，这里绑定了一个 greeting 。

## 样式

每个页面都可以有自己的样式文件，文件的扩展名是 wxss 。它就相当于是页面上用的一个样式表，你可以把页面上独有的一些样式放在页面的样式文件里，然后在页面上去使用这些样式。在小程序的视图文件上应用样式跟在 HTML 页面上应用样式一样，你可以在组件上添加特定的 class，然后在样式文件里去定义这个 class 的样式，也可以在组件上添加 style 属性，去在组件上添加内嵌的样式。

> [订阅宁皓网](https://ninghao.net/signup)，学习[开发小程序](https://ninghao.net/course/5054)。



