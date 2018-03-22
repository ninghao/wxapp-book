下面用个例子演示在[微信小程序](https://ninghao.net/package/weixin-app)里使用 Promise，async，await 这些东西。小程序里有个_wx.authorize_接口可以向用户要些权限，访问用户对应的资源，比如得到用户的位置，获取用户的相关信息等等。_wx.getSetting_这个接口可以得到用户的授权配置信息，比如用户是否已经授权我们使用他的用户信息。

## 回调

假设在我的小程序想创建个函数可以检查用户的授权信息的状态，比如我想检查用户信息（_scope.userInfo_）的授权状态，使用回调的形式来定义这个函数，像这样：

```
const authUserInfo = (callback) => {
  wx.getSetting({
    success: (response) => {
      if (response.authSetting['scope.userInfo']) {
        return callback(true)
      }

      wx.authorize({
        scope: 'scope.userInfo',
        success: () => {
          return callback(true)
        },
        fail: () => {
          return callback(false)
        }
      })
    }
  })
}
```

这样用：

```
authUserInfo((result) => {
  console.log(result)
})
```

在_wx.getSetting_的成功回调里，它的_response.authSetting_里面包含的就是授权信息。

## Promise

现在我们用 Promise 的形式定义之前的函数，它可以像这样：

```
const authorize = (setting) => () => {
  return new Promise((resolve, reject) => {
    wx.getSetting({
      success: (response) => {
        if (response.authSetting[setting]) {
          resolve(true)
        }

        wx.authorize({
          scope: setting,
          success: () => {
            resolve(true)
          },
          fail: () => {
            reject(false)
          }
        })
      }
    })
  })
}
```

用函数创建函数：

```
const authUserInfo = authorize('scope.userInfo')
```

Promise 的用法：

```
authUserInfo()
  .then(() => {
    console.log('yes')
  })
  .catch(() => {
    console.log('no')
  })
```

## async，await

先导入[regenerator-runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js)。

```
import regeneratorRuntime from '../../libs/regenerator-runtime'
```

用法：

```
async
 onTapSubmitButton () {
  try {
    
await
 authUserInfo()
    console.log('yes')
  } catch (error) {
    console.log('no')
  }
}
```

> 文章参考了“[一斤代码](http://www.jianshu.com/u/d0dea96b2432)” 的文章，与 “[Aki](https://github.com/Akiq2016)” （宁皓 QQ 群里她叫喵~）提供的小程序模板。

## 参考资料

1. [微信小程序中使用Promise进行异步流程处理](http://www.jianshu.com/p/e92c7495da76)
2. [Akiq2016/mini-program-template](https://github.com/Akiq2016/mini-programm-template)
3. [Facebook Regenerator Runtime](https://github.com/facebook/regenerator/tree/master/packages/regenerator-runtime)



