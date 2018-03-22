![](https://work.ninghao.net/wp-content/uploads/2017/12/Snip20171207_1.png)

用户可以在小程序那里选择用微信登录， 在用微信登录之前要把用户的微信帐号与网站帐号绑定在一起，就是在数据库里记录一下用户微信给我们提供的 openid 。绑定的时候可以保存用户的一些其它的信息，比如用户的微信头像，所在城市，省份等等。在小程序上调用 wx.getUserInfo 接口可以获取到微信用户的相关信息。得到以后，把这些信息发送到后端服务接口来处理。

处理绑定微信帐号，可以先在后端服务创建一个 REST 接口。 一个 WordPress 自定义 REST 接口类里的一个方法：

```
  public function bind( $request ) {
    $js_code   = $request['code'];
    $user_id   = $request['userId'];
    $user_info = $request['userInfo']['userInfo'];

    $session = $this->get_weixin_session( $js_code );
    if ( is_wp_error($session) ) {
      return $session;
    }

    $user = $this->get_user_by_openid( $session['openid'] );

    if ( $user ) {
      return new WP_Error(
        'weixin_rest_already_bind',
        __( '您的微信帐号与某个用户已经绑定在一起了。' ),
        [
          'status' => 400
        ]
      );
    }

    $this->update_user_weixin_session( $user_id, $session );
    $this->update_user_weixin_user_info( $user_id, $user_info );
    return 'ok';
  }
```

方法里面的 _get\_weixin\_session_是个自定义的方法，可以根据微信的登录码去请求用户的_session\_key_与_openid_。方法里的 _get\_user\_by\_openid_也是个自定义方法，可以根据用户的_openid_获取到网站上的用户。_update\_user\_weixin\_session_可以保存微信用户的_openid_与_session\_key_。_update\_user\_weixin\_user\_info_这个方法可以保存微信用户的相关信息。

## 微信用户登录状态

用微信用户的登录凭证，可以到微信服务端那里换取微信用户的登录状态，状态里面一般有个_openid_还有个_session\_key_。_openid_指的是微信用户在我们的应用（小程序）里的唯一标识，它的值不会变。_session\_key_是微信用户的会话信息，也就是微信用户的登录状态，它的值在小程序那里每次调用_wx.login_获取登录凭证的时候，可能会刷新微信用户的这个_session\_key_，所以它的值是会变化的。

获取微信用户的登录状态要准备 AppID 与 AppSecret（应用密钥），这些信息在小程序的管理后台可以得到。这些信息比较敏感，所以我把它放在项目的 .env 里面了。我们的[WordPress 项目](https://ninghao.net/course/5245)是用 Bedrock 项目模板创建的，所以可以直接编辑项目根目录下的_.env_文件，添加新的环境变量，然后在项目里可以用_env\('变量名'\)_获取到环境变量文件里的值。

在方法里用了 WordPress 提供的_wp\_remote\_get_这个函数去请求配置好的获取微信登录状态的服务接口地址。得到的响应里面，body 属性里的东西就是我们需要的_openid_与_session\_key_，不过数据的格式是 JSON，所以得用_json\_decode_处理一下才能在 PHP 里用。

```
  public function get_weixin_session( $js_code ) {
    $API_BASE = 'https://api.weixin.qq.com/sns/jscode2session';
    $APP_ID   = env('WX_APP_ID');
    $SECRET   = env('WX_APP_SECRET');
    $url      = "$API_BASE?appid=$APP_ID&secret=$SECRET&js_code=$js_code&grant_type=authorization_code";

    $response = wp_remote_get( $url );
    $session  = json_decode( $response['body'], true );

    if ( isset( $session['errcode'] ) ) {
      return new WP_Error(
        $session['errcode'],
        $session['errmsg'],
        [
          'status' => 400
        ]
      );
    }

    return $session;
  }
```

## 根据 openid 获取用户

下面这个方法可以根据微信用户的 openid 获取到对应的网站用户。里面用了 WordPress 的_get\_users_查询符合指定条件的用户，条件就是用户的_wx\_opeind_这个 meta 的值等于指定的 openid 。 找到用户就返回用户列表里面的第一个用户。

```
  public function get_user_by_openid( $openid ) {
    $users = get_users([
      'meta_key' => 'wx_openid',
      'meta_value' => $openid,
      'meta_compare' => '='
    ]);

    if ( empty( $users ) ) {
      return NULL;
    }

    return $users[0];
  }

```

## 保存用户的 openid

小程序那里发送绑定微信帐号的请求，就是把微信用户的 openid ，还有微信用户的相关信息保存起来。这里用了WordPress 的 _update\_user\_meta_，保存了指定用户的 meta 信息。

```
  public function update_user_weixin_session( $user_id, $session ) {
    update_user_meta( $user_id, 'wx_openid', $session['openid'] );
    update_user_meta( $user_id, 'wx_session_key', $session['session_key'] );
  }
```

```
  public function update_user_weixin_user_info( $user_id, $user_info ) {
    update_user_meta( $user_id, 'wx_avatar_url', $user_info['avatarUrl'] );
    update_user_meta( $user_id, 'wx_city', $user_info['city'] );
    update_user_meta( $user_id, 'wx_country', $user_info['country'] );
    update_user_meta( $user_id, 'wx_gender', $user_info['gender'] );
    update_user_meta( $user_id, 'wx_language', $user_info['language'] );
    update_user_meta( $user_id, 'wx_nickname', $user_info['nickName'] );
    update_user_meta( $user_id, 'wx_province', $user_info['province'] );
  }
```



