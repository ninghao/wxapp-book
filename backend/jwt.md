在微信小程序里，用户的相关信息是在后端服务那里保存的。我们可以给用户在小程序上准备一个登录页面，用户输入他的用户名还有密码，可以申请登录。后端服务收到登录请求，验证用户输入的登录信息是否有效，如果有效就给用户签发一个 JSON Web Token，小程序收到签发的 Token 会把它存储在自己的 Storage 里，下回用户再请求需要权限的资源的时候，可以带着签发的这个 Token，服务端那里会验证 Token 的有效性，然后决定是否要执行用户请求的动作。

上面说的就是在小程序上实施基于 JSON Web Token 的身份验证的主要流程。在新发布的《[微信小程序：应用后台\_\_身份验证](https://ninghao.net/course/5509)》这个课程里介绍了如何开发实现这种身份验证的功能。后端服务我用了 WordPress，你也可以选择其它框架作为小程序的后端服务，比如 Drupal，Laravel，Rails，Node.js 等等。流程基本都是一样的。

## 后端服务

小程序要有个后端服务为它提供功能，想要为用户做身份验证，后端服务应该有一套用户管理系统。就是系统可以管理大家申请的帐号，系统要有自己的权限控制。一般用户管理系统都支持为用户分配不同的角色（Role），用户可以属于其中的某些角色，作为管理员， 我们可以为不同的用户角色分配不同的权限，就是能在网站应用上执行的一些动作，比如创建内容是个权限，编辑内容也是个权限。

在 WordPress 上，用户的权限统一被称为_Capabilities_，权限的具体的名字一般可以表示它具体能做的事情，比如_publish\_posts_，表示发布内容，_edit\_users_，表示编辑用户 ...  WordPress 里面默认还设置了几种用户角色，_administrator_（管理员），_editor_（内容编辑），_author_（内容作者），_contributor_（贡献者），_subscriber_（订阅者），这些角色都有默认的一些权限。

要注意的是，WordPress 并没有提供管理用户角色与对应权限的界面。如果你想添加新的角色，或者修改角色的对应的权限，可以在自定义主题或插件里用代码去实现。或者也可以安装一个叫 Pods 的插件，然后启用它的 Roles and Capabilities 功能扩展。这样你就会看到一个管理用户角色与权限的界面。

## JWT 接口

后端服务要有一个签发与验证 JWT 的接口，在 [WordPress：基于 JWT 的身份验证](https://ninghao.net/blog/5453) 里面介绍了方法。接口应该接受用户发送过来的登录信息，验证用户名与密码是否匹配，如果验证通过，就去给用户签发一个 Token。签发 JWT 类型的 Token 需要一些特定的规则，在 《[JWT：JSON Web Token](https://ninghao.net/course/5018)》 介绍了方法。一般我们会使用一些库去做这件事，如果是 PHP 应用，可以使用 Firebase 的[php-jwt](https://github.com/firebase/php-jwt)。在课程里我们用了一个 WordPress  JWT 插件，这个插件在签发还有验证 Token 的时候就使用了 php-jwt 提供的功能。

## 小程序

在小程序上实现用户登录的功能，可以先去准备一个登录页面，页面上可以包含用户名，密码文本框，还有一个登录按钮。按一下登录按钮可以把用户在用户名与密码文件框里输入的内容发送给后端服务接口，这个接口就是签发 JWT 用的。

![](https://work.ninghao.net/wp-content/uploads/2017/11/Snip20171127_1.png)

小程序得到了 Token 以后要把它存储在 Storage 里面，Storage 就是一个名值对的存储，它只能存储字符串，服务器签发返回来的 Token 是个对象，所以我们可以把这个 Token 对象用 JSON.stringify 处理一下，然后再存储在 Storage 里。

```
const _token = JSON.stringify(token)
wx.setStorageSync('jwt', _token)
```

用的时候可以再用 JSON.parse 处理一下存储在 Storage 里的 Token 数据。

```
const _jwt = wx.getStorageSync('jwt')
const jwt = JSON.parse(_jwt)
```

服务端签发过来的 Token ，除了 Token 本身，还有一些用户相关的信息，比如头像，名字，邮件等等。我们可以直接在小程序的页面上利用这些数据。

![](https://work.ninghao.net/wp-content/uploads/2017/11/Snip20171127_2.png)

## 验证身份

用户在小程序里面成功登录以后，得到了服务端签发的 Token，下次用户再去执行一些需要验证身份的动作的时候（比如发布新内容），要带上一个 Authorization 这个 Header，对应的值是 Bearer 空格后面再加上具体的 Token 的值。

    wx.request({
      url: `${ API_BASE }/${ API_ROUTE }`,
      method: 'POST',
      header: {
        'Authorization': `Bearer ${ this.data.jwt.token }`
      },
      data: {
        ...this.data.entity
      },
      success: (response) => {

      }
    })

## 相关资源

1. [WordPress：基于 JWT 的身份验证](https://ninghao.net/blog/5453)
2. [Firebase / php-jwt](https://github.com/firebase/php-jwt)



