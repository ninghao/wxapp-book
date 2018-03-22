[微信的小程序](https://ninghao.net/package/weixin-app)相当于是一套前端（Frontend）应用的框架，让它变成一个真正能用的 App，我们还得给它提供一个后端服务，或者叫应用后台（Backend）。几乎所有的后端应用框架都支持为小程序提供后端服务。[WordPress](https://ninghao.net/package/wordpress)，[Drupal](https://ninghao.net/package/drupal)，[Rails](https://ninghao.net/package/rails)，[Node.js](https://ninghao.net/package/nodejs)...   了解一下 RESTful 风格的后端服务接口。

后端服务通过 RESTful 风格的接口为小程序提供数据，或者处理从小程序那里发送过来的数据。比如可以给小程序提供一个内容列表让它在页面上显示，可以接收小程序发送（POST，DELETE，PATCH）过来的数据，比如把内容保存到后端的数据库里。

## WordPress

WordPress 是一套非常流行的开源内容管理系统。你可以用它管理内容，比如文章，图片，文件，用户等等。这些内容管理功能，都有一个对应的 REST 接口，所以非常适合为其它类型的应用提供后端服务，比如小程序，前端应用，移动端应用。

宁皓网最近发布了《[微信小程序：应用后台（WordPress）](https://ninghao.net/course/5457)》课程，可以帮您使用 WordPress 作为小程序的应用后台，为小程序提供后端服务。

## 准备

先根据《[WordPress 开发：开发环境](https://ninghao.net/course/5245)》这个课程在本地运行一个 WordPress 网站。《[WordPress 开发：生产环境](https://ninghao.net/course/5270)》演示了怎么把本地开发的项目部署到真正的服务器上。想用 WordPress 网站作为小程序的应用后台，得保证这个网站已经发布到互联网上了，就是任何人都可以通过互联网访问到你的网站。

在小程序的开发阶段，你可能希望使用在本地开发环境上的 WordPress ，这样调试起来更方便。我们可以使用 SSH 通道的方法，在本地与服务器之间打个通道，这样小程序对后端服务接口的访问会被转发到你的本地开发环境上。《[在互联网访问本地开发环境](https://ninghao.net/course/5163)》里面介绍了方法。

**在服务器上的用来转发请求的 Nginx 配置：**

```
upstream wpdev {
  server 127.0.0.1:7689;
}

server {
  listen       443 ssl;
  server_name  wp-dev.ninghao.net;
  ssl          on;
  index        index.html;

  ssl_certificate           /etc/nginx/ssl/wp-dev.ninghao.net/214318278430706.pem;
  ssl_certificate_key       /etc/nginx/ssl/wp-dev.ninghao.net/214318278430706.key;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers               AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
  ssl_prefer_server_ciphers on;

  location / {
   proxy_set_header  X-Real-IP  $remote_addr;
   proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header  Host $http_host;
   proxy_redirect    off;
   expires           off;
   sendfile          off;
   proxy_pass        http://wpdev;
  }
}
```

打通道用的命令：

```
ssh -vnNT -R 7689:192.168.50.5:443 用户@服务器IP
```

_7689_是服务器与本地主机连接用的端口号，_192.168.50.5_是本地开发环境上的虚拟机的 IP 地址，_443_是虚拟机开放的端口，因为我在本地开发环境上运行的 WordPress 启用了 HTTPS，所以端口号是 443。

## HTTPS

直接用 HTTP 传输的数据很有可能会被中间人给拦截下来，因为数据没加密，所以他们会知道具体的数据。更安全的方法是使用 HTTPS 加密传输数据，HTTPS 现在是网站的标配，网站必须要支持使用它。小程序也要求后端服务接口必须通过 HTTPS 传输数据。《[HTTPS 与 HTTP2](https://ninghao.net/course/4539)》这个课程介绍了为网站启用 HTTPS 的方法。

## 应用

一切准备就绪，在一个小程序页面上的_onLoad_生命周期方法里，去请求 WordPress 的文章列表接口，然后把请求得到的数据放到小程序的页面上使用。

### 逻辑

    const API_BASE = 'https://wp-dev.ninghao.net/wp-json'
    const API_ROUTE = 'wp/v2/posts'

    Page({
      data: {
        entities: [],
      },
      onLoad () {
        wx.request({
          url: `${ API_BASE }/${ API_ROUTE }`,
          success: (response) =
    >
     {
            console.log(response)
            const entities = response.data
            this.setData({
              entities
            })
          }
        })
      }
    })

在页面上添加了一个叫_entities_的数据，默认它是个空白的数组。在_onLoad_方法里，使用_wx.request_这个小程序提供的请求功能去请求 WordPress 网站的文章列表接口。假设一切正常，文章列表内容会在_response.data_这个属性里，我们把它交给了页面上的_entities_这个数据。这样在页面的视图文件上，就可以利用这些数据去显示一些内容了。

### 视图

```
<
view class="media-box" 
  
wx:for="{{ entities }}" wx:key="{{ item.id }}"
>
<
view class="media-box__bd"
>
<
view class="media-box__title"
>
{{ item.title.rendered }}
<
/view
>
<
/view
>
<
/view
>
```

在视图里面，使用_wx:for_循环处理了一下页面上的_entities_数据，它里面的东西就是从 WordPress 那里得到的一个文章列表内容。在视图里，绑定输出了一个_item.title.rendered_，_item_是每次循环的时候当前项目的默认的名字，这里表示的就是一个文章内容。_title_是文章的标题，_rendered_是具体的标题的内容。

现在，在小程序的页面上你应该会看到一个来自 WordPress 网站里的文章列表内容。

> [订阅宁皓网](https://ninghao.net/signup)，在线学习所有小程序相关课程。



