先了解一下用类（Class）的形式去创建一个 WordPress REST 接口，接口可以处理注册新用户的请求。在小程序那里准备一个注册用户用的页面，用户输入，想要注册的用户名，邮件地址，还有密码申请登录，遇到错误会在小程序的页面上显示出来。一切正常，后端接口把用户存储在数据库里，小程序得到服务端的成功提示以后，自动发起登录请求，成功以后会给用户签发 JWT，打开用户的个人档案页面显示相关的信息。新发布的《[微信小程序：应用后台\_\_用户注册](https://ninghao.net/course/5548)》介绍了整个过程。

## 接口

后端服务接口我还是用了 WordPress，因为它本身就有一套完整的用户管理系统，不需要单独去开发。用户可以属于某个用户角色，不同的用户角色有不同的权限，我们可以管理用户的角色与角色对应的权限。

WordPress 核心的 REST 接口里面有一个 Users 接口（wp/v2/users），可以管理用户，不过这要验证用户的权限。只能是有创建新用户这个权限的用户才能添加新的用户，也就是用户要使用 Users 接口要先通过身份验证。所以这个接口并不适合注册新的用户。

不过我们可以基于这个核心自带的 Users 接口去创建一个新的接口，可以使用类的形式去创建，就是把一个接口需要的功能，包含在一个类里面，里面可以有处理请求用的方法，检查权限用的方法等等。REST 接口类一般可以去继承 WP\_REST\_Controller，我需要的用户注册功能，跟核心的 Users 接口很相近，所以也可以直接让类去继承 WP\_REST\_Users\_Controller 。

直接可以去复制 WP\_REST\_Users\_Controller 类里面，用来创建新用户的方法（create\_item），接口的权限回调要自己来定义，核心的权限回调要检查用户的权限。我们自己的注册用户用的接口的权限回调可以检查用户是不是在请求里包含 roles 属性，因为如果你允许用户自己修改 roles 属性，他可以把自己的用户角色设置成 administrator，这样他就会是网站的管理员。

```
  public function create_item_permissions_check( $request ) {
    if ( ! empty( $request['roles'] ) ) {
      return new WP_Error( 'rest_cannot_create_user', __( 'Sorry, you are not allowed to create new users.' ), array( 'status' => rest_authorization_required_code() ) );
    }

    return true;
  }
```

## 小程序

在小程序这里，创建一个新的注册用户用的页面，页面上有几个文本字段，用户名，邮件地址，密码。按下提交按钮，可以把用户在文本字段里输入的内容发送给我们准备好的注册用户用的 REST 接口。接口可以处理发送过来的数据，检查一下，没有问题就在系统上创建一个新的用户，用户信息会存储在数据库里。

服务接口如果成功创建了资源就会返回 201 状态码，小程序这里可以判断一下，如果返回 201 ，我们就可以使用用户在注册页面上输入的用户名与密码，发起登录请求。登录成功以后，服务端返回签发的 JWT，小程序拿到 JWT，把用户带到个人档案页面，显示用户相关的信息。

![](https://work.ninghao.net/wp-content/uploads/2017/12/Snip20171206_2.png)

![](https://work.ninghao.net/wp-content/uploads/2017/12/Snip20171206_6.png)

![](https://work.ninghao.net/wp-content/uploads/2017/12/Snip20171206_7.png)

![](https://work.ninghao.net/wp-content/uploads/2017/12/Snip20171206_8.png)

