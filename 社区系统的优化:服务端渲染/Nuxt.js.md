# [`Nuxt.js`]什么是`Nuxt.js`

+ `Nuxt.js`是一个基于`Vue.js`的通用应用框架；

+ 预设块利用`Vue.js`开发服务端渲染所需要的各种配置；

+ 提供静态站点，主要用于那些不用经常去变化的页面，比如博客（典型的`hexo`），一旦写好基本不变，还有网站的`about`页面这种类型，都可以用静态站点固定下来，使得我们不再需要动态的进行服务器渲染了，因为数据基本不变，因此对`SEO`比较友好；

+ 提供异步数据加载，自行封装了`axios`等流行的请求库，增强了它们的功能；比如下图就是`Nuxt.js`的`axios`模块的功能，它添加了失败重试、请求代理等原先`axios`没有的功能；

  <img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 00.33.41.png" alt="截屏2020-10-05 00.33.41" style="zoom:30%;" />

  

+ 中间件支持；

+ 布局支持，开发者可以自行选择在`Nuxt.js`中使用的`UI`框架，比如`element.js`；

# [`Nuxt.js`]安装`Nuxt.js`

`Nuxt.js`的安装非常简单，我们使用以下命令首先下载`Nuxt.js`的安装工具；

```
npx create-nuxt-app nuxt-demo
```

安装完后，接着终端就会进入交互式创建`Nuxt.js`项目的过程，下方是一个典型的配置；

![截屏2020-10-05 00.38.26](/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 00.38.26.png)

当我们安装完成后，我们就可以进入项目，典型的项目结构如下，要注意是没有`src`目录的，并且编写代码时必须严格遵守它的目录结构：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 00.45.40.png" alt="截屏2020-10-05 00.45.40" style="zoom:30%;" />

然后，我们可以使用命令`npm run dev`启动开发模式服务等，在启动过程中，控制台上会打印客户端渲染、服务端渲染代码的打包的进度，以及内存使用的情况等，可以说是非常贴心了；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 00.50.25.png" alt="截屏2020-10-05 00.50.25" style="zoom:50%;" />

一般来讲，我们都会在项目里使用`eslint`作为代码检查、纠错，如果`eslint`启动时报错了，八成是因为缺少`eslint-plugin-html`这个库；

接着，我们定制改错的行为，一般我们会使用`vetur`对`vue`文件进行改错，这个插件可以定制对不同代码块所使用的`vs code`的格式化插件，比如你可以对`vue`文件内的`script`标签内的内容使用`prettier`这个`vs code`插件来改错，对`template`标签内的`html`内容使用`beautify-html`这个`vs code`插件进行规范与改错，这些都可以在`首选项->vetur`中配置，如下图，比如将`template`代码块的`vs code`格式化插件修改为`prettier`：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 00.59.33.png" alt="截屏2020-10-05 00.59.33" style="zoom:30%;" />

当然，你需要安装`prettier`这个`vs code`插件才会起效果，这时候就会使用项目内的`.prettierrc`作为默认的`prettier`配置文件；

```
//.prettierrc
{
  "semi": false,
  "singleQuote": true
}
```

进一步地，如果你使用`eslint`，那么可以配合`prettier-eslint`这个`npm`插件来实现每一次`eslint --fix`修正样式之前都先`prettier`一次，使用`prettier`来增强格式修正的能力；

# [`Nuxt.js`]`Nuxt.js`结构构成

`Nuxt.js`对我们的文件结构有严格的限定，但是就是因为这些限定，使得我们编写代码可以更加聚焦于业务本身，而不需要过多关注于其他的配置，比如路由、静态文件等等；

文件结构如下：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 22.27.59.png" alt="截屏2020-10-05 22.27.59" style="zoom:20%;" />

它们的功能分别如下：

+ `layouts`：视图，代表你看到的页面的最外层的框架，在这一层你可以放置`header`、`footer`等每一个页面或者大部分页面都需要使用的内容，默认情况下在`layouts`目录下会有一个`default.vue`，`default.vue`内的`Nuxt`元素就是页面的插入位置，默认情况下所有的`pages`都在`default.vue`的框架下；

  ```vue
  //default.vue
  <template>
    <div>
      <nav-bar></nav-bar> //添加nav-bar元素
      <Nuxt /> //按照路由渲染的页面
    </div>
  </template>
  <script>
  import Navbar from '~/components/Navbar'
  export default {
    components: {
      'nav-bar': Navbar,
    },
  }
  </script>
  ```

+ `pages`：页面，就是我们的显示的主体部分，它是整个浏览器标签页面的主体，默认是在`default.vue`这个`layout`下；该文件夹下的每一个`Vue`文件都是一个路由；

  在`pages`文件夹下新建一个称为`create.vue`，访问它的路由就是`/create`，它的`layout`默认是`~/layouts/default.vue`，我们不需要写任何配置，只需要在`pages`文件夹下新建这个`vue`文件即可，就可以进行路由了，是不是很方便？

  ```vue
  <template>
    <div>create</div>
  </template>
  
  <script>
  export default {}
  </script>
  
  <style lang="scss" scoped></style>
  ```

+ `components`：组件，是自由的功能元素块，可以安插在`layouts`和`pages`中的任何一个角落，如下图就是一个组件，在`layout`中我们通过传统的引用来使用它；

  ```vue
  <template>
    <div>
      <nuxt-link class="link" to="/create">create link</nuxt-link> //router-link改为nuxt-link表示链接
      <nuxt-link class="link" to="/about">about link</nuxt-link>
      <nuxt-link class="link" to="/list">list link</nuxt-link>
    </div>
  </template>
  
  <script>
  export default {}
  </script>
  
  <style lang="scss" scoped>
  .link {
    margin-right: 10px;
  }
  </style>
  ```

+ `assets`：素材，代表一系列预编译的内容，比如图片，`Nuxt.js`在打包时对此进行了一些优化，如果图片尺寸小于`1k`，那么会直接转译为`base64`代码，如果大于`1k`则会赋予图片`hash`值并存储图片文件：

  <img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-05 22.53.06.png" alt="截屏2020-10-05 22.53.06" style="zoom:30%;" />

+ `plugins`：插件，会在每一次服务端渲染、创建`context`上下文对象后、使用`context`对象前立即调用该文件夹下所有的`plugin`，为当前请求的`nuxt.js`的`context`实例插入新的属性或方法，提供新的行为；

+ `middleware`：中间件，可以用来改变底层框架的行为，类似`koa`的中间件机制对功能进行增强；

+ `static`：资源，代表打包时直接使用而不进行编译处理的文件；

# [`Nuxt.js`]预取数据

和`Vue SSR`一样，`Nuxt.js`也可以进行预取数据，原理是一致的；

在`Vus SSR`中，我们是这样来进行预取数据的，在要服务器渲染的`Vue`组件下添加`asyncData`钩子，在服务端渲染该组件时，会执行`asyncData`钩子来预取数据并将数据存入`store`中，之后在渲染组件时自行将`store`中的对应数据取出来进行渲染；

`Nuxt.js`也是类似这样，但是它增强了功能；

## `asyncData`预取数据

首先，同样地，我们要在`Vue`组件中添加`asyncData`钩子，该函数用于预先取出数据，该钩子会在我们的组件渲染前调用，因此是取不到任何关于组件的对象的；

> 但是要注意，只有页面`pages`组件才可以用`asyncData`；

下方就是一个简单的`Nuxt.js`预取数据的例子；

```vue
<template>
  <div class="container">
    userAgent is :{{ userAgent }}
  </div>
</template>

<script>
export default {
  asyncData(context) {
    return {
      userAgent: context.req
        ? context.req.headers['user-agent'] : 'No user agent',
    }
  },
}
</script>

```

你应该看到了，`asyncData`钩子函数接受`context`上下文参数作为输入，该对象包含存储代表当前请求对应的`Nuxt.js`实例，包含了诸如`request、response、env`等等极为丰富的信息，比如，我们可以从中获取到当前的请求信息（`context.req`），更多的可以获取到的信息见`https://zh.nuxtjs.org/api/context`，这里对可以获取的属性进行了详细的说明；

`asyncData`返回一个包含预取数据的对象或者一个`promise`，`promise`的`then`函数内必须返回一个包含预取数据的对象，这个对象会在组件渲染期间和`data`方法返回的数据进行组合最后一并返回给当前组件进行渲染；

也就是说，上图实例中的`template` 内的`userAgent`响应式数据是可以正确获取的，来自于我们当前组件预取阶段的数据；

从这里我们看出来，我们不需要像`Vue SSR`一样需要额外搭配`store`来实现数据预取，`Nuxt.js`都帮我们完成了，非常方便；

## 添加`nuxtjs/axios`

在`asyncData`钩子函数中，比较常见的操作是进行`axios`请求获取数据；

`Nuxt.js`提供了一个定制的`axios`库用于服务端渲染，我们这样安装它：

```
npm install @nuxtjs/axios
```

单单是安装到`node-modules`内是不够的，我们还要将它放到我们的`context`上下文对象中，我们在`nuxt.config.js`添加配置：

```js
  // Modules (https://go.nuxtjs.dev/config-modules)
  modules: [
    // https://go.nuxtjs.dev/axios
    '@nuxtjs/axios',
  ],
```

这样，`@nuxtjs/axios`就注入到`context`对象上了，我们可以从`context`对象上获取`$axios`对象；

```js
export default {
  asyncData({ $axios }) {
    // const result = await $axios.get('http://localhost:8000/posts')
    const result = await $axios.$get('http://localhost:8000/posts') //$get可以直接获取返回数据的data
    return {
      // posts: result.data,
      posts: result,
    }
  },
}
```

绝大多数时候，我们都需要对`axios`进行拦截，比如`axios`请求失败了，我们就改变要渲染的路由，那么在`Nuxt.js`中怎么实现呢？

`Nuxt.js`的`plugin`闪亮登场，我们可以在`plugin`下新建`axios.js`，我们定义该文件封装所有的`axios`的拦截器；

`Nuxt.js`会在`context`使用前执行所有的`plugin`来修改、增强`context`的属性、行为；

按照官方`https://axios.nuxtjs.org/helpers`的写法，我们这样做，为`$axios`添加一个`onError`的回调：

```js
export default function ({ $axios, redirect }) {
  $axios.onError((error) => {
    if (error.response.status === 500) {
      redirect('/sorry')
    }
  })
}
```

`axios.js`直接导出一个函数，该函数接收`context`上下文对象，其内部就是修改了`$axios onError`后的行为，当`$axios`请求失败，就会调用`onError`函数重定向到新路由；

`Nuxt.js`会读取`plugins`文件夹下所有的`plugin`的导出函数一一执行，完成挂载；

更多的用法参考`nuxtjs/axios`的官网；

## 使用`json-server`搭建简易`mock`接口服务器

现在我们的后端服务器是不存在的状态，因此我们要`mock`一个返回指定数据的接口；

如果我们实在是不愿意使用`doclever`、`mock.js`这类大型的`mock`工具，那么可以使用`json-server`这个`nodejs`库，这个库可以使得我们方便地启动一个本地的静态接口服务器；

首先，我们安装它：

```
npm install -g json-server
```

然后，我们可以创建一个`db.json`文件，该文件内的结构是这样的：

```json
{
  "posts": [{
    "id": 1,
    "title": "json-server",
    "author": "typicode"
  }],
  "comments": [{
    "id": 1,
    "body": "some comment",
    "postId": 1
  }],
  "profile": {
    "name": "typicode"
  }
}
```

如果`json`文件只有单层，那么每一个属性都对应一个接口，比如`posts`属性就是`localhost:xxxx/posts`接口，这个接口就会将`posts`属性的值返回回来；

`json-server`本质上是`express`封装的，我们这样来启动它：

```
json-server --watch db.json --port 8000
```

这样，我们就成功启动它了，是不是很简单，`json-server`适合临时、偷懒的情况，一般大型的系统开发还是使用`doclever`的比较好；

# [`Nuxt.js`]`Universal Mode`

在一般的`web`应用中，没有使用任何框架时，我们发送`url`和路由是直接拉取该`url`要前往的页面的`html`和内联的`js`的，然后可以直接渲染出来；当我们要跳转路由，会继续下载新的`html`和`js`文件进行渲染，也就是说，每到一个路由就需要下载新的`html`和`js`，因为新的`html`的到来，旧的`html`的所有节点都会被覆盖掉、重新设置，所以性能会偏差，同时要下载的文件有些多、耗费时间；

并且一般我们不会采用原生`web`来编写代码，速度慢、难以管理、而且比较`low`，所以一般都会使用框架；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-10 22.26.28.png" alt="截屏2020-10-10 22.26.28" style="zoom: 25%;" />

`Vue`、`React`框架所构成的项目都是`SPA`单页面应用，采用了不同的、复杂一些的下载流程；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-10 22.30.15.png" alt="截屏2020-10-10 22.30.15" style="zoom: 25%;" />

`SPA`应用加载速度很慢，正如上图所言，它要经过以下几个阶段：

+ 下载`index.html`，这个文件是所有单页面应用的起始页面，不管是任何页面都是首先获取`index.html`（慢）；
+ 下载`Vue`应用相关的`Javascript`文件，包含`Vue`实例（慢）；
+ 初始化`Vue`应用；
+ 初始化`Vue`路由，并且导航到对应的路由组件，这时候还没有渲染；
+ 请求`API`接口，拉取必要渲染所需的数据，比如`beforeMount`钩子函数（慢）；
+ 渲染页面；

相比前者，它不是直接下载该路由所对应的`html`文件和`js`文件就一了百了了，因为`index.html`本身无法表达页面上形形色色的组件（相比前者的一个缺点），后续还需要使用`Vue`实例来接管页面、导航路由、渲染页面，这整个过程耗时长、首屏速度比第一种慢；但是它好的一点是我们后续切换异步路由每次只需要下载一个`js`文件即可，不需要额外下载`html`文件，而且得益于虚拟节点，我们不需要更新整个页面的节点，因此后续路由速度快，节约性能、带宽、服务器资源；

而`Nuxt.js`是`Universal Mode`，即为前面两种应用模式的混合形态，具有这两者所共有的优点；

若是第一次请求该网址的路由，它会直接获取当前要前往的路由所对应的页面，而不是获取`index.html`入口页面，因此这个页面是可以直接显示给用户的，比第二种加载速度更快了，这点是第一种的特征；

同样地，后续需要下载`Vue`实例等`js`文件，下载完毕后接管页面；

`Nuxt.js`设计所有的路由默认都是异步路由，因此打包时默认开启`splitChunk`，如下图所示；当要前往一个新路由时，就会下载对应的`js`文件；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-11 00.32.40.png" alt="截屏2020-10-11 00.32.40" style="zoom:33%;" />

这部分是后者的特征，因此是一种混合的应用形态：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-10 22.40.17.png" alt="截屏2020-10-10 22.40.17" style="zoom: 25%;" />

+ 首先是依然是下载`about.html`；
+ 但是这时候我们直接显示这个`html`文件，这个文件已经包含了我们所需要的看到的数据库数据，正是服务端渲染提供了这样的能力；
+ 接着我们再请求`Javascript`文件，该文件包含`Vue`实例；
+ 然后初始化`Vue`应用和路由，启动`Vue`应用接管页面的行为；

我们发现，显示页面更加早了，用户的体验升级了；

而在我们要从当前页面前往一个新路由时，和`SPA`单页面应用一样都只要下载`js`文件即可；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-10 23.46.39.png" alt="截屏2020-10-10 23.46.39" style="zoom: 25%;" />

如果第二次进入相同的路由，我们也不再去请求该路由对应的`js`文件，而是仅仅进行`API`请求（`asyncData、beforeMounted`）填充数据，接着显示页面即可；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-10 23.53.31.png" alt="截屏2020-10-10 23.53.31" style="zoom: 25%;" />

# [`Nuxt.js`]链接预取

强大的`Nuxt.js`为我们的客户端侧添加了很多重要的优化，其中最重要的就是链接预取的特性；

它使得我们当在用于链接到其他路由的按钮进入用户的浏览器视口后就进行对这些链接所表示的路由的`js`文件的预取，这样做能优化我们进入新路由的加载速度，因为文件在页面加载之前就已经被下载到了本地，因此只剩下解析`js`文件准备页面节点、渲染页面的过程，速度会很快；

如下图，就是这三个链接按钮进入视口后加载的`js`文件；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-11 00.42.48.png" alt="截屏2020-10-11 00.42.48" style="zoom:30%;" />

# [`Linux`]解决网络端口占用问题

当我们出现网络端口占用时，我们可以使用以下命令查看指定端口的占用情况：

```
lsof -i:8080
```

然后我们可以使用以下命令停止指定网络端口的服务项目：

```oz
kill -9 <进程PID>
```

# [`Nuxt.js`]鉴权（重点）

`Nuxt.js`完美符合人想怎么懒就怎么懒的特性，设计出了`nuxtjs/auth`这一个我非常喜欢的模块，它封装了我们的用户的登录、获取用户信息、用户退出等非常常用的平台功能，大大简化代码量；

在这里，我们简单地去使用它，更多的用法参考官方文档`https://auth.nuxtjs.org/`，想必不需要几分钟你就能完全看懂；

首先，我们这样来下载它，要注意，`auth`模块依赖`axios`模块，因此两个都要安装：

```
yarn add @nuxtjs/auth @nuxtjs/axios
```

接着，在`nuxt.config.js`中，我们将`auth`模块集成到我们的项目中；

```
 // Modules (https://go.nuxtjs.dev/config-modules)
  modules: [
    // https://go.nuxtjs.dev/axios
    '@nuxtjs/axios',
    '@nuxtjs/auth',
  ],
```

这样，`$auth`属性就挂载到`Vue`实例上了；

对于前端来讲，登录无非就是发送一个请求到后端，数据包含账号、密码等信息，然后获得一个`token`保存到本地`localstorage`作为用户凭证，就是这样一个非常简单的过程；

类似的，获取用户信息就是将`token`作为请求的`authorization`发送到后端，然后后端读取`ctx.header.authorization`并从数据库中读取用户信息并将用户信息返回前端，同时也保存为`localstorage`；

`auth`模块将登录、获取用户数据就定义为这样的固定的过程，而大部分的应用也都是这么设计的；

正是因为这样，`nuxt.js`将登录、获取用户数据直接封装为函数，方便我们直接进行调用；

比如`this.$auth.loginWith`方法是登录、获取用户数据的结合体；

首先，我们配置`auth`的登录、获取用户数据的`axios`请求`api`、请求方法类型、要保存到`localstorage`的`res.data`的字段；

```js
auth: {
    //策略，每一个策略都是一组用户鉴权的配置
    strategies: {
      //策略，默认策略
      local: {
        //策略所使用的行为
        endpoints: {
          // 登录过程所需要的配置
          login: {
            //所请求的api
            url: '/login/login',
            method: 'post', //方法名
            propertyName: 'token', // 请求完成后res.data所存储到localstorage的字段，设置为false表示接收所有
          },
          // 退出
          logout: {
            url: '/api/auth/logout',
            method: 'post',
          },
          // 用户信息，当用户登录成功后会发送该请求获取用户的数据
          user: {
            url: '/public/userInfo',
            method: 'get',
            propertyName: 'data',
          },
        },
        // tokenRequired: true,
        // tokenType: 'bearer',
        // globalToken: true,
        // autoFetchUser: true
      },
    },
  },
```

我们在`index.vue`中如是调用`this.$auth.loginWith`：

```vue
<template>
  <div class="container">
    <p>邮箱：<input v-model="username" type="text" /></p>
    <p>密码：<input v-model="password" type="text" /></p>
    <p>图片：<input v-model="code" type="text" /></p>
    <p>sid：<input v-model="sid" type="text" /></p>
    <button type="button" @click="login()">登录</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
      code: '',
      sid: '',
    }
  },
  mounted() {
    window.vue = this
  },
  methods: {
    login() {
      this.$auth.loginWith('local', {
        data: {
          username: this.username,
          password: this.password,
          code: this.code,
          sid: this.sid,
        },
      })
    },
  },
}
</script>
```

`this.$auth.loginWith`的第一个参数是策略类型，第二个参数是传递到请求中的配置；

`this.$auth.loginWith`函数会携带第二个参数中的`data`字段下的内容作为`req.body`，发送`local`策略中定义的`login`所对应的请求`url`到后端，然后将`res.data`中返回的`token`字段存储到`localstorage`，并会设置`this.$auth.$state.loggedIn`设置为`true`，之后在任何的页面中，我们都可以使用这个参数来查看当前是否是处于登录状态；

接着，会发送`lcoal`策略中定义的`user`所对应的请求`url`到后端来获取用户的数据，这时候会带上`login`时所获得的`token`到头部的`authorization`中，并将`res.data.data`保存到本地`localstorage`中；由于这个数据过于常用，因此还会保存在`this.$auth.$state.user`中，因此，如果我们需要使用用户的数据，直接在`user`中获取就行了，是不是非常方便；

这也是`nuxt.js`设计`auth`的初衷，将一切都变得愈发简单和易用；

这样，`loginWith`函数就完成了用户的登录过程；

这个过程非常简单，只要几下的配置就行了，非常方便，适合懒到极限的开发者；

更多的，你也可以查看官网的第三方登录的配置来实现更加高级的业务；

然后，我们配置一下`axios`的`baseURL`为后端服务器；

```
 // Axios module configuration (https://go.nuxtjs.dev/config-axios)
  axios: {
    baseURL: 'http://localhost:3000',
  },
```

同时，设置一下前端服务器地址；

```
 server: {
    // 服务器端口
    port: 8080,
    // 主机
    host: '0.0.0.0',
  },
```

# [`Nuxt.js`]从`Vue.js`迁移到`Nuxt.js`

+ 修改所有的`router-link`为`nuxt-link`，如果有必要的话，修改`to`属性从命名路由`:to="{name:'login'}"`改为`to="/login"`直接路由；

+ 将上层框架的`header`、`footer`等元素迁移到`layouts`中，并将需要路由决定的部分用`<Nuxt></Nuxt>`代替；

  ```
  <template>
    <div>
      <imooc-header></imooc-header>
      <imooc-panel></imooc-panel>
      <Nuxt />
      <imooc-footer></imooc-footer>
    </div>
  </template>
  ```

+ `Nuxt.js`并不支持`Mixin`的用法，将`Mixin`的所有配置移入具体组件；

+ 将所有的`axios`请求放在路由组件的`asyncData`中完成，所有的`components`都只能接收数据而不能使用`asyncData`预渲染，如果你需要在子组件中请求自己的数据，那么自行安装公版的`axios`在`vue`的生命周期函数中请求数据，或者将其作为嵌套路由中的组件；

+ 修改嵌套路由；由于现在所有的路由行为都是注入的，因此我们要改变我们进行嵌套路由的形式，具体格式参见如下，详情参考官网；

  <img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-15 00.36.48.png" alt="截屏2020-10-15 00.36.48" style="zoom:50%;" />

  同时修改上层路由的`router-view`为`nuxt-child`；

+ 如果必要的话，将预取的数据存入`$store`供`Vue`实例使用；

+ 如果必要的话，修改所有的鉴权行为，都用`$auth`进行替代；

总体上，迁移要注意的小细节很多，尽量少出现迁移的情况；

# [`Chrome`]利用`Chrome`控制台的`Performance`选项卡进行性能分析

我们可以利用`Chrome`控制台的`Performance`选项卡对我们的应用的性能进行分析；

在我们进入页面之前，我们点击`record`，开始性能分析：

![截屏2020-10-15 00.59.06](/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-15 00.59.06.png)

然后我们键入`url`前往页面；

页面加载完成之后，我们点击`stop`停止性能分析；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-15 01.00.14.png" alt="截屏2020-10-15 01.00.14" style="zoom:50%;" />

`Chrome`就会生成一个性能报告供我们查看：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-15 01.01.16.png" alt="截屏2020-10-15 01.01.16" style="zoom:50%;" />

我们可以拖拽上方过程图来查看每一个时刻的渲染情况：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-15 01.05.56.png" alt="截屏2020-10-15 01.05.56" style="zoom:50%;" />

还可以查看`network`中的文件加载和渲染过程：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-15 01.06.50.png" alt="截屏2020-10-15 01.06.50" style="zoom:50%;" />

其他用法参见`Chrome`官网；

