![](https://work.ninghao.net/wp-content/uploads/2017/12/Snip20171214_5.png)

最近为微信小程序设计与开发一个评论功能，后面会用视频的形式为大家展示出来，先说下大概的思路。评论一般会嵌入到其它内容里面显示，比如一个文章内容主体的下面可以显示一个评论列表，可以添加新的评论。

## 接口

因为评论内容要嵌入到其它内容里面，也就是用户在某个内容上面提交的评论要属于这个内容，就是评论内容与被评论的内容之间要关联在一起。比如可以在接口里面用 post 属性表示评论所属的内容。评论之间还可以进行回复，一个人留了一条评论，另一个人可以回复这条评论，就是评论与回复之间也应该有个关联，比如在接口里用 parent 表示这条评论回复的是哪条评论。

## 应用

在小程序里面设计与开发评论功能，用我们之间介绍过的知识完全可以解决。先在内容页面上显示一个评论列表，我们之间介绍过实施[内容列表的功能](https://ninghao.net/course/5457)，列表可以[无限滚动](https://ninghao.net/video/5474)加载新的列表项目。

内容页面加载以后（onLoad），去请求（[wx.request](https://ninghao.net/video/5233)）一个评论列表，只返回属于当前内容的评论。把得到的数据放在页面的数据里，然后交给页面视图去显示评论列表内容。

评论列表数据里面有些东西要做点特别处理，比如评论的时间，我想用 x 以前这种形式显示，moment.js 可以做到。要为小程序项目导入一个 moment.js。下面是我在内容页面上添加的一个处理请求得到的评论列表数据的方法：

```
  transformComments (comments) {
    return comments.map((item) => {
      const content = {
        ...item.content,
        wxml: towxml.toJson(item.content.rendered, 'html')
      }

      const fromNow = moment.utc(item.date).local().fromNow()

      let comment = {
        ...item,
        content,
        fromNow
      }

      if (item.parent !== 0) {
        const inReplyTo = item._embedded['in-reply-to'][0]
        const inReplyContentWxml = towxml.toJson(inReplyTo.content.rendered, 'html')
        const inReplyToFromNow = moment.utc(item.date).local().fromNow()

        const reply = {
          ...inReplyTo,
          wxml: inReplyContentWxml,
          fromNow: inReplyToFromNow
        }
        comment = {
          ...comment,
          reply
        }
      }

      return comment
    })
  },
```

评论列表项目里面的评论内容里面可能会包含 HTML 标签，[使用 Towxml 处理了 HTML 标签](https://ninghao.net/video/5497)，把它们转换成小程序的组件，然后再把处理好的数据放到评论列表数据里面。

```
const content = {
  ...item.content,
  wxml: towxml.toJson(item.content.rendered, 'html')
}
```

评论的时间的显示可以使用 moment.js 处理一下，完成以后再把处理好的时间格式放到评论列表数据里面，这样在页面视图上就可以显示处理之后的时间格式了。

```
const fromNow = moment.utc(item.date).local().fromNow()

let comment = {
  ...item,
  content,
  fromNow
}
```

页面加载以后，可以一下，每隔多久就执行一下这个评论数据处理，这样用户在浏览内容与评论的时候，评论的时间的显示会发生改变。比如一条评论一开始显示的发布日期是 几秒钟之前，过了一会儿，执行了评论数据处理之后，这条评论的发布日期可能会变成 1 分钟之前。

```
setInterval(() => {
  const comments = this.transformComments(this.data.comments)

  this.setData({
    comments
  })
}, 60000)
```

> [订阅宁皓网](https://ninghao.net/signup)，安静地在线学习[微信小程序的设计与开发](https://ninghao.net/package/weixin-app)。



