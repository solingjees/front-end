# [`Nuxt.js`]`Vue.js vs Nuxt.js`

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-02 22.24.18.png" alt="截屏2020-09-02 22.24.18" style="zoom:50%;" />

# [`SSR`]什么是`SSR`

`Server-Side Rendering(SSR)`服务端渲染是用户第一次请求/刷新页面时，由服务端响应`HTML`字符串，可以省去省去浏览器端首次渲染的工作，加快首屏显示速度；

它有以下特点：

+ 更好的`SEO`；
+ 更快的内容到达时间(`time-to-content`)

在`Vue`的框架下，有以下两种方式来实现`SSR`；

+ `vue & vue-server-renderer 2.3.0+`；
+ `Nuxt.js`；

`SSR`的工作原理如下：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-02 22.36.20.png" alt="截屏2020-09-02 22.36.20" style="zoom:50%;" />

+ 服务器将浏览器所需要的`HTML`先构建好，然后发直接发送到浏览器，因为浏览器不需要再去做各种各样的解释、构建等工作，而只需要将`html`渲染到页面上，因此首屏加载速度会有明显提升；
+ 然后我们要下载`js`文件，之前我们只是发送了`html`字符串，我们的前端页面还是不可以进行交互的（`Viewable`，只能看不能交互），现在我们需要额外下载`js`，来使其可以交互；
+ 当`js`下载完毕后，我们的页面和`js`还是零散的，并没有结合在一起，还需要进一步组装成一个框架实例（`react`、`vue`），接下来我们的页面就可以进行交互了（`Interactable`）；

与`SSR`相对的是`CSR(Client-Server Renderiing)`，和`SSR`有显著不同的地方，那就是当浏览器请求页面时，服务器会将所有的文件都下载回来，然后再进行框架组装和渲染，一切就绪后页面才是可以看的、可以交互的，也就是说`Viewable`和`Interactive`都是在最后一起实现的，即使我们的`html`页面准备渲染了，`js`没有准备好，我们的页面还是看不到的，和`SSR`相比，首屏加载速度有明显的劣势；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-02 22.46.44.png" alt="截屏2020-09-02 22.46.44" style="zoom:50%;" />

下图是一个实例，我们可以发现`SSR`的首屏加载早于`CSR`，但是要注意，`SSR`需要更多的时间来完成页面所有内容的加载与渲染；

![截屏2020-09-02 22.52.25](/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-02 22.52.25.png)

# [`Vue SSR`]编写通用代码的约束

+ 由于我们现在需要在服务器端来创建`vue`实例，因此必须在服务器上使用该用户的数据来形成`vue`实例，并且每一个请求必须单独创建一个`vue`实例，否则如果都将同一个`vue`实例回传，可能出现数据的交叉污染（`cross-request state pollution`），就是数据弄混了，把一个用户的数据反而传递给另一个用户了，因此我们要保证每一个请求都是全新的、独立的应用程序实例，从而防止造成交叉污染；

  如下就不会产生交叉污染：

  ```js
  // app.js
  const Vue = require('vue')
  
  module.exports = function createApp (context) {
    return new Vue({
      data: {
        url: context.url
      },
      template: `<div>访问的 URL 是： {{ url }}</div>`
    })
  }
  ```

  ```js
  // server.js
  const createApp = require('./app')
  
  server.get('*', (req, res) => {
    const context = { url: req.url }
    const app = createApp(context) //重新创建了一个vue实例
  
    renderer.renderToString(app, (err, html) => {
      // 处理错误……
      res.end(html)
    })
  })
  ```

  同时，我们的页面在渲染之前已经准备好了状态数据，并且状态不再是响应式的，因此能节约一些将数据转化为响应式数据的开销，因为我们只是需要`HTML`页面而已，不需要这些东西；

+ 组件生命周期函数只有`beforeCreate`和`created`这两个钩子函数可以在服务端渲染时被调用，并且要尽量不要在`beforeCreate`和`created`钩子中使用`setInterval`这样的副作用函数，因为在`SSR`期间，即使我们的组件被销毁了，它也是无法调用销毁钩子函数来销毁函数副作用的，它将会永远保留下来，因此尽量将副作用函数放在`mounted`这类钩子函数中使用；

+ 通用代码不可以使用`window`、`document`、`node`等这一类只有一端可以使用的`api`，也就是要保证平台无关性，因为在服务端根本没有`window`这类的浏览器对象，也就导致根本就无法渲染成功；

+ `SSR`期间尽量不要使用自定义指令（`v-show`），因为这些自定义指令可能会直接操作`dom`，而在服务器渲染期间，根本没有`document`的`api`，导致自定义指令必然出错；

  如果需要的话，有两种方式来使用自定义指令：

  + 可以将组件抽象化，并运行在`Vitual-DOM level`上，如使用`render function`来创建组件，这样就不会在服务端渲染期间被执行渲染了；
  + 但是部分自定义组件等不容易被替换成组件，那么你可以使用`directives`选项提供的服务器版本来代替原来的自定义指令；

# [`Vue SSR`]第一个实例-----基础`SSR`工程（重点）

`webpack`是我们进行服务端渲染的核心工具，因为我们的`Vue`就是使用`webpack`渲染生成的，客户端的`vue-cli`默认集成了`webpack`的配置，所以我们看不到，但是在服务器端，我们不能安装`vue-cli`，因此需要自己配置了，这是一个繁琐的过程，但是一配好一劳永逸，并且加深理解；

下图是完整的使用`webpack`进行服务端渲染的构建过程；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-11 00.41.18.png" alt="截屏2020-09-11 00.41.18" style="zoom:50%;" />

我们创建一个新的工程来理解服务端渲染。首先来创建一个新的工程，该工程目录如下：

```
src
├── components
│   ├── Hello.vue
├── App.vue
├── app.js # 通用 entry(universal entry)
├── entry-client.js # 仅运行于浏览器
└── entry-server.js # 仅运行于服务器
```

其中`app.vue`内容如下：

```vue
<template>
  <div id="app">
    <Hello></Hello>
  </div>
</template>

<script>
import Hello from './components/Hello.vue'
export default {
  components: {
    Hello,
  },
}
</script>

<style></style>
```

`Hello.vue`内容如下：

```vue
<template>
  <div>hello world</div>
</template>

<script>
export default {
  name: 'hello',
}
</script>

<style></style>
```

这些都是非常简单的内容，就是`app.vue`内引用`Hello.vue`，然后屏幕上就会显示`hello world`字符串，仅此而已；

然后是`app.js`，它的内容有些特殊，内容如下：

```js
import Vue from 'vue'
import App from './App.vue'

// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp() {
    const app = new Vue({
        // 根实例简单的渲染应用程序组件。
        render: h => h(App)
    })
    return {
        app
    }
}
```

我们发现`app.js`内部导出了一个生成我们刚刚编辑的`vue`实例的函数，至于为什么这样做，是为了后续我们的进一步编写的方便，到之后你就会发现，我们需要打包两次，这两次对`vue`实例的使用都不同，因此需要导出工厂函数，建议先这样写着；

这样我们就完成了一个`vue`实例的封装，之后我们就能调用`app.js`的默认导出函数创建`vue`实例；

服务端渲染顾名思义，就是将我们的`vue`项目渲染成一个`html`字符串，这个`html`字符串以`id`为`app`的`div`元素为最外层的`html`元素，接着将这个`html`字符串进一步封装一下成为一个可以在前端直接挂载渲染的`html`字符串（比如增加`head`等元素的封装），最后将这个`html`字符串直接送到前端；

接下来，我们要创建一个`server-entry.js`文件，这是一个对`vue`项目的进一步地封装：

```js
import {
    createApp
} from './app'

export default context => {
    const {
        app
    } = createApp()
    return app
}
```

根据`vue ssr`官网的说明，我们必须要获取创建的`vue`实例的`app`属性，这个属性包含了我们要使用的`vue`实例，正如你在上面代码所看到的，我们同样将该过程封装为一个函数，返回这个`app`属性，当我们进行服务端渲染时，就会需要使用`server-entry.js`文件导出的`app`属性，我们只要这样定义即可，服务端渲染函数会自动调用文件默认导出函数来获得生成的`vue`实例来创建`html`字符串；

接着，根据`vue ssr`官网的要求，我们要将我们的以`server.entry.js`为入口的`vue`项目自定义打包成一个适用于服务端渲染的包，因为我们的项目里存在大量在`node`环境下无法执行的`api`，比如`document`对象，因此直接进行服务端渲染是不现实的，必须要打包成适用于`node`环境的、适用于服务端渲染的项目，只有这样我们的`vue`项目才能配合`vue-server-renderer`（`vue ssr`官方样例的服务端渲染插件工具）将我们的项目服务端渲染成`html`字符串；

为了打包，我们需要配置`webpack`，自然地，我们在根目录下创建`config`文件夹，创建一个`webpack.base.config.js`文件，该文件提供基础的`webpack`配置，内容如下：

```js
const path = require('path')
const { VueLoaderPlugin } = require('vue-loader')
const webpackClientConfig = require('./webpack.client.config')
const resolve = (dir) => path.join(path.resolve(__dirname, '../'), dir) //构成绝对路径
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const isProd = process.env.NODE_ENV === 'production'
// const {
//     CleanWebpackPlugin
// } = require('clean-webpack-plugin')
module.exports = {
  mode: isProd ? 'production' : 'development',
  output: {
    path: resolve('dist'),
    publicPath: '/dist/',
    filename: '[name].[chunkhash].js',
  },
  resolve: {
    alias: {
      public: resolve('public'),
    },
  },
  module: {
    noParse: /es6-promise\.js$/, //不对该文件进行操作
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader', //使用vue-loader对vue文件进行解释
        options: {
          compileOptions: {
            preserveWhitespace: false, //不保留空格
          },
        },
      },
      {
        test: /\.js$/,
        loader: 'babel-loader', //转译es6语法格式到commonJs
        exclude: /node_modules/,
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'url-loader', //处理路径
        options: {
          limit: 10000, //限制文件大小
          name: '[name].[ext]?[hash]',
        },
      },
      {
        test: /\.s(a|c)ss?$/, 
        use: ['vue-style-loader', 'css-loader', 'sass-loader'], //转译css
      },
    ],
  },
  performance: {
    hints: false, //不显示打包过程
  },
  plugins: [
    new VueLoaderPlugin(), //使用Vue的loader插件
    new FriendlyErrorsPlugin(), //显示友好的提示（react、vue脚手架都已经默认配备了）
    // new CleanWebpackPlugin(),
  ],
}

```

接着，我们再编写一个`webpack.server.config.js`文件用于服务端渲染的配置，此处配置比较复杂：

```js
const { merge } = require('webpack-merge')
const nodeExternals = require('webpack-node-externals')
const baseConfig = require('./webpack.base.config.js')
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')

module.exports = merge(baseConfig, {
  // 将 entry 指向应用程序的 server entry 文件
  entry: './src/entry-server.js',

  // 这允许 webpack 以 Node 适用方式(Node-appropriate fashion)处理动态导入(dynamic import)，
  // 并且还会在编译 Vue 组件时，
  // 告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
  target: 'node',  //必须将vue工程编译成node格式，因为服务端渲染会在服务端运行，必须在node环境中可用

  // 对 bundle renderer 提供 source map 支持
  devtool: 'source-map',

  // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
  output: {
    libraryTarget: 'commonjs2', //将使用的插件导出为node格式，否则在服务端渲染时无法使用
  },

  // https://webpack.js.org/configuration/externals/#function
  // https://github.com/liady/webpack-node-externals
  // 外置化应用程序依赖模块。可以使服务器构建速度更快，
  // 并生成较小的 bundle 文件。
  externals: nodeExternals({
    // 不要外置化 webpack 需要处理的依赖模块。
    // 你可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件，
    // 你还应该将修改 `global`（例如 polyfill）的依赖模块列入白名单
    // whitelist: /\.css$/
    allowlist: /\.css$/,
  }),

  // 这是将服务器的整个输出
  // 构建为单个 JSON 文件的插件。
  // 默认文件名为 `vue-ssr-server-bundle.json`
  plugins: [new VueSSRServerPlugin()], //使用服务端渲染的插件
})
```

接着，我们修改一下我们的`package.json`脚本来将我们的`vue`项目进行打包：

```js
 "scripts": {
    "build": "rimraf dist && npm run build:server",
    "build:server": "webpack --config config/webpack.server.config.js"
  },
```

于是，我们就可以使用`nom run build`打包我们的项目为一个服务端渲染的项目了，会产出以下文件（只有`vue-ssr-server-bunble.json`文件）：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-13 22.58.38.png" alt="截屏2020-09-13 22.58.38" style="zoom:50%;" />

该文件包含以下内容：

```
{
  "entry": "main.5b3d8af844a0ead6bd58.js",
  "files": {
     "....."
  }
}
```

其中，`files`包含的就是用于服务端渲染的代码的字符串；

接着，我们要配置我们的服务器，当用户请求我们的页面时，我们就使用生成的这个`json`文件进行服务端渲染生成`html`字符串，这个字符串就包含了首屏加载的所有内容，接着封装后回传给客户端`html`字符串，客户端随即会进行挂载渲染，最后显示出我们的首屏页面；

我们使用`express`作为服务器，监听`8080`端口；

```js
const server = require('express')()

// 在服务器处理函数中……
server.get('*', (req, res) => {
   ...
})

server.listen(8080)
```

接着，我们要使用`vue-server-renderer`进行服务端渲染：

```js
const {
    createBundleRenderer
} = require('vue-server-renderer')

const bundle = require('./dist/vue-ssr-server-bundle.json')

const renderer = createBundleRenderer(bundle, //引入服务端渲染文件
{
    runInNewContext: false, // 推荐
})
```

`renderer`就是一个服务端渲染器，当渲染完成后，我们就可以得到`html`内容（非字符串格式），`html`内容以`id=app`的`div`元素为根元素，显然，这种格式回传前端是无法被浏览器识别的，因此我们要提供一层模版，`vue-server-renderer`允许我们为生成的`html`结构添加模版；

我们在`src`下编写`index.template.html`文件作为模版，`vue ssr`增强了功能，允许我们使用插值的方式更加灵活地生成自定义的模版：

```html
<!DOCTYPE html>
<html>

<head>
    <!-- 使用双花括号(double-mustache)进行 HTML 转义插值(HTML-escaped interpolation) -->
    <title>{{ title }}</title>

    <!-- 使用三花括号(triple-mustache)进行 HTML 不转义插值(non-HTML-escaped interpolation) -->
    {{{ meta }}}
</head>

<body>
    <!--vue-ssr-outlet-->
</body>

</html>
```

其中`vue-ssr-outlet`注释不可以省去，这是告诉服务端渲染器，将`html`结构挂载在此处；

正如代码中所展示的，`title、meta`都是插值，其中`{{}}`代表可以进行`html`转义内部的插值，`{{{}}}`表示不会转义内部的插值，主要作用是插入用户自定义的`html`字符串，可以防止被转义掉；

具体怎么用，往下看；

我们将模版添加到渲染的配置中：

```js
const path = require('path')
const resolve = file => path.resolve(__dirname, file)
const templatePath = resolve("./src/index.template.html")
const template = fs.readFileSync(templatePath, 'utf-8') //使用utf-8格式读取文件，否则会出现乱码

const renderer = createBundleRenderer(bundle, {
    runInNewContext: false, // 推荐
    template, // （可选）页面模板
})
```

这样我们的渲染器就算制作完成了，我们只要进一步在路由中这样写，来进行服务端渲染即可；

```js
// 在服务器处理函数中……
server.get('*', (req, res) => {
    //设置要传入到模版中的差值
    const context = {
        title: 'hello ssr with webpack',
        meta: `
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width,initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
        `
    }
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(
       context, //传入插值
       (err, html) => {
        // 处理异常……
        res.end(html) //回调函数返回生成的html字符串，这个字符串就是一个完整的html字符串，直接回传给前端即可
    })
})
```

这样我们就完成了服务端渲染的任务；

但是我们还遗漏了一件事情，我们只是回传了`html`字符串，客户端没有任何的交互能力，因此下一步，我们要做客户端渲染，我们要在回传的`html`字符串中添加`script`标签，使得当客户端挂载完`html`后下载一个包含`vue`实例的`js`文件，该文件会自动将`vue`实例挂载到`id=app`的`div`节点上，托管整个页面给`vue`；

那么为什么称为客户端渲染呢？因为我们是通过下载`js`，然后在客户端进行执行、进行渲染的；

下一步，我们要使用`webpack`打包这样一个`vue`实例，然后使用`html`内嵌`script`标将签该实例的`js`文件传回前端：

+ 执行时自动创建`vue`实例；
+ 执行时自动将创建的`vue`实例挂载到`div=app`的`div`节点上；

于是，我们要创建一个新的`webpack`编译入口`client-entry.js`，这个文件会初始化一个`vue`实例，并将其`dom`结构挂载到`id=app`的`div`节点上，因此，我们可以发现，客户端渲染喝服务端渲染都需要`createApp`用于执行不同的行为，因此`app.js`必须配置为一个工厂函数：

```js
import {
    createApp
} from './app'

// 客户端特定引导逻辑……

const {
    app
} = createApp()

// 这里假定 App.vue 模板中根元素具有 `id="app"`
app.$mount('#app')
```

接着，配置`webapck.client.config.js`:

```js
const webpack = require('webpack')
const {
    merge
} = require('webpack-merge')
const baseConfig = require('./webpack.base.config.js')
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')

module.exports = merge(baseConfig, {
    entry: './src/entry-client.js',
    optimization: {
        // 重要信息：这将 webpack 运行时分离到一个引导 chunk 中，
        // 以便可以在之后正确注入异步 chunk。
        // 这也为你的 应用程序/vendor 代码提供了更好的缓存。
        // new webpack.optimize.CommonsChunkPlugin({
        //     name: "manifest",
        //     minChunks: Infinity
        // }),
        // 此插件在输出目录中
        // 生成 `vue-ssr-client-manifest.json`。
        splitChunks: {
            name: "manifest", //打包出来的名字
            minChunks: Infinity
        }
    },
    plugins: [
        new VueSSRClientPlugin() //使用vue ssr的客户端渲染插件
    ]
})
```

该配置中最核心的是`VueSSRClientPlugin`，该插件会以`entry-client.js`为入口打包整个`vue`项目，并输出为一个`js`文件，还附带一个说明文件`xxx.json`；

我们配置一下脚本：

```
 "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "rimraf dist && npm run build:client && npm run build:server",
    "build:client": "webpack --config config/webpack.client.config.js",
    "build:server": "webpack --config config/webpack.server.config.js"
  },
```

现在，我们只要使用命令`npm run build`就可以同时进行服务端渲染的代码和客户端渲染的代码的打包工作了；

打包结果是这样的：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-13 22.58.38.png" alt="截屏2020-09-13 22.58.38" style="zoom:50%;" />

最上面的`js`文件就是我们的`vue`实例，`manifest.json`文件是说明文件，该文件内部指明了要传递给前端的`js`文件地址；

与服务端渲染一样，我们只要在服务端渲染器配置即可，它就会自动将我们的`vue`实例作为`script`元素插入到传回到前端的`html`字符串中，最终在页面成功挂载时下载脚本并应用`vue`实例；

```js
...
const bundle = require('./dist/vue-ssr-server-bundle.json')
const clientManifest = require('./dist/vue-ssr-client-manifest.json') //引用minifest说明文件
const templatePath = resolve("./src/index.template.html")
const template = fs.readFileSync(templatePath, 'utf-8')

const renderer = createBundleRenderer(bundle, {
    runInNewContext: false, 
    template, 
    clientManifest // 给前端使用的vue文件，要传递给前端的js
})

// 在服务器处理函数中……
server.get('*', (req, res) => {
    const context = {
        title: 'hello ssr with webpack',
        meta: `
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width,initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
        `
    }
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(context, (err, html) => {
        // 处理异常……
        res.end(html)
    })
})

server.listen(8080)
```

返回给前端的`html`字符串就带上了`script`标签，用于下载`js`，下载完成执行时就会将我们的`vue`实例托管`id=app`的`div`元素下的所有内容；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-14 00.11.23.png" alt="截屏2020-09-14 00.11.23" style="zoom:50%;" />

至此，一个完整的`vue`实例就完成了，并且在前端成功显示出来了`hello world`的字符串；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-09-14 00.13.47.png" alt="截屏2020-09-14 00.13.47" style="zoom:50%;" />

但是，这也只是基础的`SSR`；

# [`Vue SSR`]`vue ssr`配置开发热加载（重点）

往往是这种情况，如果我们在生产环境下，前端用户访问页面，我们就将我们的打包好的文件、热加载生成的`html`字符串返回前端；

但是如果我们是在开发环境下，那么就不一样了，我们需要创建一个监视文件修改来重新打包的服务和热更新前端的服务，每一次我们更新了文件，这个服务器就会更新我们的文件，然后刷新前端页面；

但是，这两种环境下的配置有相似之处：

+ 都需要配置服务端渲染器，所以我们就将渲染器的对象作为文件内全局变量；
+ 都需要使用同一个创建渲染器的函数，所以我们就将创建渲染器的过程封装成一个函数供我们方便调用；
+ 都要最后使用服务端渲染器渲染`html`字符串，所以封装成一个方法供生产环境和开发环境调用来生成字符串，并且将该字符串直接传递给前端；

因此，我们首先封装这些我们需要公共使用的对象和方法：

```js
const {
    createBundleRenderer
} = require('vue-server-renderer')
const path = require('path')

const isProd = process.env.NODE_ENV === 'production'
const resolve = (file) => path.resolve(__dirname, file) //获取完整路径

//两个环境都需要设置renderer，直接抽离出来
let renderer, readyPromise

const templatePath = resolve('./src/index.template.html') //获取模版路径
...

//创建渲染器的函数的封装，由于开发模式和生产模式的bundle和options不同，因此需要分开
const createRenderer = (bundle, options) => {
    //返回创建的实例
    return createBundleRenderer(
        bundle,
        Object.assign(options, {
            basedir: resolve('./dist'), //设置基础目录
            runInNewContext: false, // 推荐
        })
    )
}

//渲染器渲染，不管开发模式还是生产模式都需要调用这个函数
const render = (req, res) => {
    const context = {
        title: 'hello ssr with webpack',
        meta: `
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width,initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
        `,
    }
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(context, (err, html) => {
        // 处理异常……
        res.end(html)
    })
}
```

那么生产环境的配置就很简单了；

如果当前在生产环境下，我们就配置渲染器的要使用的包为`/dist`目录下的文件，因为我们使用`webpack`打包的后的文件都在`/dist`目录下；

```js
...

if (isProd) {
    //如果当前是生产环境
    const bundle = require('./dist/vue-ssr-server-bundle.json')
    const clientManifest = require('./dist/vue-ssr-client-manifest.json')
    const template = fs.readFileSync(templatePath, 'utf-8') //我们这里必须是读取二进制流，因此是静态的

    //设置服务端渲染器
    renderer = createRenderer(bundle, {
        template, // （可选）页面模板
        clientManifest, // （可选）客户端构建 manifest
    })
} else {
   ...
}

...
```

最后我们在服务器的路由上应用它即可：

```js
// 在服务器处理函数中……
server.get(
    '*',
    isProd ?
    render :
   ... //开发环境下的行为
)

server.listen(8080)
```

这样，我们就完成了生产环境下的配置，如果当前的服务器处于生产环境下，那么我们就直接调用`render`函数返回`html`字符串；

接下来，我们思考开发环境怎么配置；

当我们处于开发环境下时，当我们的项目启动后，我们要创建一个`dev server`来监视我们的代码文件，当我们的代码文件更新了，我们就重新打包我们的文件，当打包完成后，我们再去更新前端的代码；

首先，我们要创建这样一个`dev server`，我们以函数的形式来封装这个`dev server`，同时将一个表示当前`dev server`的`promise`对象返回去，当内部调用了该`promise`的`resolve`方法，外部就会可以调用`then()`方法去接收这个事件并执行相应的行为；那么我们为什么要这样设计呢？是因为，当我们的代码发生了更新并重新打包后，我们必须通知调用者我们的包已经更新并打包完成了，也就是说这是一个异步的行为，那么`promise`是最合适的实现方式，当内部打包完成后就去调用`resolve`方法去通知调用者我们的包已经更新了，使的调用者知道包已经更新了，可以发送给前端一份新的代码；

在`dev server`中，我们要在以下三种情况进行重新打包：

+ 当我们的服务端渲染的源代码发生更新时；
+ 当我们的客户端渲染的源代码发生更新时；
+ 当我们的`html`模版文件的源代码发生更新时；

这三种的任意一种情况发生都会调用表示当前`dev server`的`promise`的`resolve`方法去通知调用者-----我们的包更新了；

我们先来书写一些公共的、这三种情况都需要调用的成分：

```js
const fs = require('fs')
const chokidar = require('chokidar')
const webpack = require('webpack')
const path = require('path')
const clientConfig = require('./webpack.client.config')
const serverConfig = require('./webpack.server.config')
const hotMiddleware = require('webpack-hot-middleware')
const devMiddleware = require('webpack-dev-middleware')
const NFS = require('memory-fs')
//热重载的服务器

//读取指定文件系统（fs）下的文件（file），将文件以utf-8格式来读取并返回
const readFile = (fs, file) => {
    try {
        return fs.readFileSync(path.join(clientConfig.output.path, file), 'utf-8')
    } catch (e) {}
}
//创建服务器的函数，传递服务器实例、模版文件路径、打包完成的回调函数
const setupServer = (app, templatePath, cb) => {
    let bundle //保存服务端渲染的包的json格式文件
    let clientManifest //保存客户端渲染的申明文件
    let template //html文件的模版
    let ready //更新完毕后调用，会触发外部函数回调

    template = fs.readFileSync(templatePath, 'utf-8') //读取文件二进制流

    const readyPromise = new Promise((r) => (ready = r))

    //文件更新后的回调函数，会触发cb()，cb函数会告诉调用者我们更新完成，并将新的render的配置都发出去，这是因为我们的包更新了，必须使用新的包信息来生成新的render，否则会出现不一致的情况
    const update = () => {
        //如果bundle/clientManifest更新了
        if (bundle && clientManifest) {
            cb(bundle, {
                template,
                clientManifest,
            })
            ready() //触发resolve函数，告知调用者我们已经更新成功了
        }
    }
 }
```

注意，在这里，我们很巧妙地将`promise`的`resolve`方法作为了一个函数内变量，使的函数内都可以调用它来通知调用者-----我们包更新成功了；

`update()`函数做两件事情：

+ 调用回调函数，在本项目中是为了更新`render`，如果不更新`render`，是无法更新我们的数据返回的；
+ 触发`promise`的`then`，目的是使的原用户实例返回新的`html`字符串；

接着，我们编写服务端渲染情况部分的代码；当服务端渲染依赖的代码发生更新，我们就需要重新打包一份新的代码；

由于我们处于开发模式下，因此每一次将文件写到磁盘上速度较慢且效率非常低下，所以我们会将数据写到内存中，这也要求我们使用一个不同的文件系统，因此我们需要使用`memory-fs`库来使用内存文件系统，而不能使用原来的磁盘文件系统；

接下来是服务端渲染的代码，相应的导入语句见上一代码：

```js
  //服务器渲染
    const mfs = new NFS() //
    let serverCompiler = webpack(serverConfig) //使用serverConfig生成一个编译器

    serverCompiler.outputFileSystem = mfs //将服务端渲染的文件系统设置为内存

    //当所依赖的文件发生了变化，{}表示webpack就会自动打包，打包完成后会调用回调函数
    serverCompiler.watch({}, (err, stats) => {
        if (err) throw err //如果打包出错，直接停止
        stats = stats.toJson() //将返回的打包描述信息转化为json格式
        stats.errors.forEach((err) => console.error(err))
        stats.warnings.forEach((err) => console.warn(err))
        if (stats.errors.length) return //如果打包结果有错误，就停止
        bundle = JSON.parse(readFile(mfs, 'vue-ssr-server-bundle.json')) //重新读取json文件内的内容，更新bunble
        update()
    })
```

接下来是客户端渲染的部分，这部分会稍微复杂；

为了实现页面热加载，因此我们必须将`webpack-hot-middleware/client`也打包到我们要传递给前端的客户端渲染代码中，这里面封装了一个`websocket`热更新的客户端；

同时要在打包的配置中添加一个新的插件：`new webpack.HotModuleReplacementPlugin()`，这个插件会在打包时将`webpack-hot-middleware/client`包的代码整合到客户端渲染代码中，使得客户端渲染代码可以正常进行使用；

```js
//配置client端的代码的修改
//给webpack配置添加新的plugin，用于和客户端构成websocket连接
clientConfig.plugins.push(new webpack.HotModuleReplacementPlugin())

//将热加载添加到入口，使的将webpack-hot-middleware/client的代码打包到我们的前端文件中
clientConfig.entry.app = [
  'webpack-hot-middleware/client',
  clientConfig.entry.app,
]

clientConfig.output.filename = '[name].js'
```

然后，我们要使用客户端渲染的`webpack`配置，创建一个客户端打包器`compiler`；

进一步地，我们要依照客户端代码打包器创建一个`dev server`服务，并将其挂载到我们的`koa`服务器上，当`webpack`重新打包完成后就会通知`koa`，然后`koa`进一步地会通知到热加载的服务；

接着将我们的热更新的`websokect`服务也应用到我们的`koa`服务器上，当`webpack`重新打包后，热加载服务会接收到来自于`koa`的消息，然后就可以发送消息到前端的`websocket`使得前端重新发送请求拉取页面；

```js
  //配置webpack打包器
const clientCompiler = webpack(clientConfig)

//使用打包器上配置webpack打包服务器
const devServer = devMiddleware(clientCompiler, {
  noInfo: true, //不需要console提示
  publicPath: clientConfig.output.publicPath, //编译到的根目录，注意，webpack热加载会打包到内存，所以是看不到的
  logLevel: 'silent'
})

app.use(devServer) //应用webpack服务器

app.use(hotMiddleware(clientCompiler))
//启动热加载服务，使用clientCompiler的配置对文件进行监听，一旦发生变化，就会通知
//webpack服务器进行打包
```

最后，我们要监听客户端渲染依赖的代码的更新的事件，当客户端渲染依赖的代码重新打包完成后，我们重新读取`vue-ssr-client-manifest.json`，并调用`update()`函数通知-----调用`setupServer`函数的调用者的服务端渲染器`renderer`所依赖的配置文件已经更新；

```js
   clientCompiler.hooks.done.tap('clientsBuild', (stats) => {
        stats = stats.toJson()
        stats.errors.forEach((err) => console.error(err))
        stats.warnings.forEach((err) => console.warn(err))
        if (stats.errors.length) return
        clientManifest = JSON.parse(
            readFile(
                devServer.fileSystem, //由于文件处于内存中，因此必须如此获得文件系统
                'vue-ssr-client-manifest.json'
            )
        )
        update()
    })

    //对html模版文件的监听很简单，就是使用chokidar库去监听html模版文件即可
    //监视html文件的更新，nodejs的文件监听无法对文件名的修改进行监听，而且很差劲
    chokidar.watch(templatePath).on('change', () => {
        //重新读取文件内容
        template = fs.readFileSync(templatePath, 'utf8')
        console.log('template is updated')
        //触发更新的回调函数
        update()
    })
```

当我们处于开发环境中，我们在`server.js`中去调用`setupServer()`函数去创建服务，在这里，当我们的服务端渲染、客户端渲染的文件发生了变化并重新打包，就会调用下方的回调函数，去更新`renderer`；

同时由于热加载服务已经通过`webSocket`通知了前端-----代码已经更新，因此前端会发送一次新的请求来获取新的代码，而在此之前我们的`renderer`已经更新完成了，因此会获取新的代码；

```js
if (isProd) {
   ...
} else {
    //开发模式下，我们创建一个热重载的服务，每次我们更新了代码都会自动重新打包
    readyPromise = require('./config/setup-dev-server')(
        server,
        templatePath,
        (bundle, options) => {
            renderer = createRenderer(bundle, options)
        }
    )
}
```

处于开发环境下时，我们要稍微处理一下我们的服务器的行为，每一次客户端请求时我们都要求直到新的重新打包的任务完成后才会将新的字符串返回（实际好像没有什么用）：

```js
server.get(
    '*',
    isProd ?
    render :
    (req, res) => {
        readyPromise.then(() => {
            render(req, res)
        })
    }
)
```

这样，我们就完成了热加载的分环境配置，这是很难以理解的一部分，要多多阅读和修缮；

完整代码如下：

```js
//server.js
const Vue = require('vue')
const {
    createBundleRenderer
} = require('vue-server-renderer')
const server = require('express')()
const fs = require('fs')
const path = require('path')
const resolve = (file) => path.resolve(__dirname, file)

const isProd = process.env.NODE_ENV === 'production'

//创建渲染器的函数的封装，由于开发模式和生产模式的bundle和options不同，因此需要分开
const createRenderer = (bundle, options) => {
    //返回创建的实例
    return createBundleRenderer(
        bundle,
        Object.assign(options, {
            basedir: resolve('./dist'), //设置根目录
            runInNewContext: false, // 推荐
        })
    )
}

//两个环境都需要设置renderer，直接抽离出来
let renderer, readyPromise
const templatePath = resolve('./src/index.template.html')

if (isProd) {
    //如果当前是生产环境，直接将我们开发环境下的文件作为结果
    const bundle = require('./dist/vue-ssr-server-bundle.json')
    const clientManifest = require('./dist/vue-ssr-client-manifest.json')
    const template = fs.readFileSync(templatePath, 'utf-8') //我们这里必须是读取二进制流，因此是静态的

    //设置服务端渲染器
    renderer = createRenderer(bundle, {
        template, // （可选）页面模板
        clientManifest, // （可选）客户端构建 manifest
    })
} else {
    //开发模式下，我们创建一个热重载的服务，每次我们更新了代码都会自动重新打包
    readyPromise = require('./config/setup-dev-server')(
        server,
        templatePath,
        (bundle, options) => {
            renderer = createRenderer(bundle, options)
        }
    )
}

//渲染器渲染，不管开发模式还是生产模式都需要调用这个函数
const render = (req, res) => {
    const context = {
        title: 'hello ssr with webpack',
        meta: `
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width,initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
        `,
    }
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(context, (err, html) => {
        // 处理异常……
        res.end(html)
    })
}

// 在服务器处理函数中……
server.get(
    '*',
    isProd ?
    render :
    (req, res) => {
        readyPromise.then(() => {
            render(req, res)
        })
    }
)

server.listen(8080)
```

```js
//setup-dev-Server
const fs = require('fs')
const chokidar = require('chokidar')
const webpack = require('webpack')
const path = require('path')

const clientConfig = require('./webpack.client.config')
const serverConfig = require('./webpack.server.config')

const hotMiddleware = require('webpack-hot-middleware')
const devMiddleware = require('webpack-dev-middleware')
const MFS = require('memory-fs')
//热重载的服务器

const readFile = (fs, file) => {
    try {
        return fs.readFileSync(path.join(clientConfig.output.path, file), 'utf8')
    } catch (e) {}
}

const setupServer = (app, templatePath, cb) => {
    let bundle
    let clientManifest
    let template //模版
    let ready //更新完毕后调用，会触发外部函数回调

    template = fs.readFileSync(templatePath, 'utf8') //读取文件二进制流

    const readyPromise = new Promise((r) => (ready = r))

    //文件更新后的回调函数，会触发cb()
    const update = () => {
        if (bundle && clientManifest) {
            ready() //触发resolve函数
            cb(bundle, {
                template,
                clientManifest,
            })
        }
    }

    //服务器渲染
    const mfs = new MFS()
    let serverCompiler = webpack(serverConfig)

    serverCompiler.outputFileSystem = mfs //将服务端渲染的文件系统设置为内存

    serverCompiler.watch({}, (err, stats) => {
        if (err) throw err
        stats = stats.toJson()
        stats.errors.forEach((err) => console.error(err))
        stats.warnings.forEach((err) => console.warn(err))
        if (stats.errors.length) return
        bundle = JSON.parse(readFile(mfs, 'vue-ssr-server-bundle.json'))
        update()
    })

    //配置client端的代码的修改
    //给webpack配置添加新的plugin，用于和客户端构成websocket连接
    clientConfig.plugins.push(new webpack.HotModuleReplacementPlugin())

    //将热加载添加到入口，使的将webpack-hot-middleware/client的代码打包到我们的前端文件中
    clientConfig.entry.app = [
        'webpack-hot-middleware/client',
        clientConfig.entry.app,
    ]

    clientConfig.output.filename = '[name].js'

    //配置webpack打包器
    const clientCompiler = webpack(clientConfig)

    //使用打包器上配置webpack打包服务器
    const devServer = devMiddleware(clientCompiler, {
        noInfo: true, //不需要console提示
        publicPath: clientConfig.output.publicPath, //编译到的根目录，注意，webpack热加载会打包到内存，所以是看不到的
        logLevel: 'silent'
    })

    app.use(devServer) //应用webpack服务器

    app.use(hotMiddleware(clientCompiler))
    //启动热加载服务，使用clientCompiler的配置对文件进行监听，一旦发生变化，就会通知
    //webpack服务器进行打包

    clientCompiler.hooks.done.tap('clientsBuild', (stats) => {
        stats = stats.toJson()
        stats.errors.forEach((err) => console.error(err))
        stats.warnings.forEach((err) => console.warn(err))
        if (stats.errors.length) return
        clientManifest = JSON.parse(
            readFile(
                devServer.fileSystem, //由于文件处于内存中，因此必须如此获得文件系统
                'vue-ssr-client-manifest.json'
            )
        )
        update()
    })


    //监视html文件的更新，nodejs的文件监听无法对文件名的修改进行监听，而且很差劲
    chokidar.watch(templatePath).on('change', () => {
        //重新读取文件内容
        template = fs.readFileSync(templatePath, 'utf8')
        console.log('template is updated')
        //触发更新的回调函数
        update()
    })

    return readyPromise
}

module.exports = setupServer
```

# [`Vue SSR`]`vue ssr`配置路由（重点）

在我们的项目必定要使用路由进行显示对应的页面，这在`vue ssr`中要怎么使得服务端渲染器按照我们的路由返回对应页面呢？

其实想法很简单，就是服务端接收用户要前往的路由，然后服务端渲染时切换一下`vue`实例的路由，接着再把`vue`实例渲染成`html`字符串返回前端就行了；

首先，我们需要将路由信息从`http`请求中拿到，然后传入我们的服务端渲染器中，我们这样传递：

```js

//渲染器渲染，不管开发模式还是生产模式都需要调用这个函数
const render = (req, res) => {
    const context = {
        title: 'hello ssr with webpack',
        meta: `
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width,initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
        `,
        url: req.url //我们从请求中拿到路由
    }
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(context, (err, html) => {
       //这里做了一个友好性的返回，当路由配置失败就会产生err，具体err来自于entry-server.js
        if (err) {
            if (err.code === 404) {
                res.status(404).end('Page not found')
            } else {
                res.status(500).end('Internal Server Error')
            }
        } else {
            res.end(html)
        }
    })
}


...
// 在服务器处理函数中……
server.get(
    '*',
    isProd ?
    render :
    (req, res) => {
        readyPromise.then(() => {
            render(req, res)
        })
    }
)

server.listen(8080)
```

我们直接修改了`context`的内容，将`url`作为其属性传递给了`renderer`，`renderer`会将`context`传递给渲染器的`bundle`所指定的服务端渲染的入口，即`entry-server.js`的导出函数会接受到它；

接着，我们就要去使用传递过来的`context`内的数据，修改`entry-server.js`入口的代码；

```js
import {
    createApp
} from './app.js'

export default context => {
    // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
    // 以便服务器能够等待所有的内容在渲染前，
    // 就已经准备就绪。
    return new Promise((resolve, reject) => {
        const {
            app,
            router //从createApp中将我们的vue-router导出来
        } = createApp() //创建vue实例

        // 设置服务器端 router 的位置，根据传递进来的路由导航到一个新的路由上，这个过程是异步的，不一定会立刻完成
        router.push(context.url)

        // 等到 router 将可能的异步组件和钩子函数解析完，就是完成路由的页面配置
        router.onReady(() => {
            const matchedComponents = router.getMatchedComponents()
            // 匹配不到的路由，执行 reject 函数，并返回 404
            if (!matchedComponents.length) {
                return reject({
                    code: 404 //触发err回调
                })
            }

            // Promise 应该 resolve 应用程序实例，以便它可以渲染
            resolve(app)
        }, reject //如果router配置失败了，那么就直接触发err
        )
    })
}
```

如上图所示，我们返回一个`promise`，这是因为我们内部有异步的操作，必须用`promise`进行封装；

我们会获取`context.url`参数，然后根据要前往的路由手动切换`vue`的路由，因为我们的路由是动态路由，因此切换路由是需要时间的，所以我们必须使用`router.onReady`去监听路由切换完成事件，然后我们去测试这个切换后的路由是否存在（其实还可以进行更加严格的测试），如果没有路由匹配到，说明我们当前要前往的路由根本不存在，所以直接`reject`出去，如果成功了，就将这新的`app`实例传递出去；

同时，如果`router.onReady`失败了，那么也`reject`；

这个`reject`会被`renderer`接收到，然后直接传递给`renderer`的回调函数进行处理；

如果`resolve`了，那么会将`resolve`传递出来的`app`进行`html`字符串构建，完成后会将`html`字符串作为第二参数传递给`renderer`的回调函数；如下所示：

```js
//err就是reject传递出来的错误
//html就是生成的html字符串
renderer.renderToString(context, (err, html) => {
       //这里做了一个友好性的返回，当路由配置失败就会产生err，具体err来自于entry-server.js
        if (err) {
            if (err.code === 404) {
                res.status(404).end('Page not found')
            } else {
                res.status(500).end('Internal Server Error')
            }
        } else {
            res.end(html)
        }
    })
```

由于我们要设置要前往的路由，因此我们需要将`vue-router`引入我们的`vue`项目，由于我们要在`entry-server.js`中手动地切换路由，因此我们还要将`router`导出来：

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp() {
    const app = new Vue({
        router,
        // 根实例简单的渲染应用程序组件。
        render: h => h(App)
    })
    return {
        app,
        router
    }
}
```

当然，引入后还不够，我们这里必须为我们的`vue`项目配置路由，方法和普通的建立是一致的，其他的`vue`的修改就略过了，基本就是一些`router-view`的修改，和一般前端`vue`的结构别无二致;

```js
//建立一个router.js文件
// router.js
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

export default new Router({
    mode: 'history',
    routes: [
        // ...
        {
            path: '/hello',
            component: () => import( /* webpackChunkName: 'hello' */ './components/Hello.vue'),
        }
    ]
})
```

但是要注意我们这里使用了动态引入的语法`()=>import()`来引入要路由的`vue`文件，由于我们的`webpack`是自己配置的，`babel`也是自己配置的，如果我们的`babel`只使用了`@babel/preset-env`这一个预设，那么它会将`()=>import()`这种动态引入语法解析错误，导致`webpack`在进行动态引入的时候发生崩溃，因此我们需要引入一个新插件`@babel/plugin-syntax-dynamic-import`使得`babel`可以正确识别这种动态引入的语法，以至于`webpack`可以正确进行处理；

对于这些动态引入的组件，`renderer`在生成`html`代码时会将当前路由所对应的`js`交互文件作为`link`标签引进来，首屏加载的动态组件的`js`文件的`link`标签的`as`属性为`preload`，其他后续动态加载的组件但是首屏没有被渲染的`js`文件被设置为`prefetch`，这样在前端挂载时时就可以下载这些文件到前端本地添加交互行为；

使用方法很简单，就在`.babelrc.js`文件里添加一下就行了：

```js
{
    "presets": [
        "@babel/preset-env",
        {
            "modules": false
        }
    ],
    "plugins": [
        "@babel/plugin-syntax-dynamic-import"
    ]
}
```

我们成功地配置了服务端渲染的路由，但是对于客户端渲染来说，我们也要进行修改，当我们的客户端渲染代码（就是一个创建`vue`实例的代码）到达前端执行时，我们需要等待一个包含`vue`实例的`js`文件和对应路有的动态加载的组件的`js`文件下载完毕，但是`vue`实例的挂载必须在动态加载的组件`js`文件到达并被`vue-router`成功解析之后才可以进行，因此我们这样修改`entry-client.js`入口文件；

```js
import {
    createApp
} from './app'

// 客户端特定引导逻辑……

const {
    app,
    router
} = createApp()

//在router完成了对动态组件的解析后执行回调函数
router.onReady(() => {
    // 这里假定 App.vue 模板中根元素具有 `id="app"`
    app.$mount('#app')
})
```

这样我们就完整地配置好了一个可以进行路由的服务端渲染的项目；

# [`Vue SSR`]`vue ssr`的数据预取（重点）

我们希望在我们的页面首屏加载时用户的数据已经填充好了，这样能进一步提升`SEO`；

第一步，我们需要为每一个服务端渲染的实例添加`store`、`router`，不使用公共的`store`、`router`是为了防止产生数据污染；

因此这要求我们每一次服务端渲染时，都创建新的`store`和`router`，所以我们稍微修改一下`store`和`router`文件的暴露方法，将创建`store`和`router`的方法暴露出来；

`src/store/index.js`的内容如下，我们将创建`store`的方法暴露出来，其中相关数据用于演示：

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

//这两个方法可以获取到数据，比如http请求获取数据
import {
    getData,
    getData2
} from '@/api/data'

export function createStore() {
    return new Vuex.Store({
        state: {
            lists: [],
        },
        actions: {
            getDataAction({
                commit
            }) {
                return getData().then((res) => {
                  //将获取的数据填入state.lists store中
                    commit('setData', res)
                })
            },
            getDataAction2({
                commit
            }) {
                return getData2().then((res) => {
                    commit('setData', res)
                })
            },
        },
        mutations: {
            setData(state, data) {
                state.lists = data
            },
        },
    })
}
```

其中，我们使用的`getData、getData2`的方法是这样获取数据的，只是为了演示：

```js
const getData = () => new Promise((
    resolve
) => {
    setTimeout(() => {
        resolve([{
                id: 1,
                item: 'hello'
            },
            {
                id: 2,
                item: 'hello2'
            }
        ])
    }, 1000)
})

const getData2 = () => new Promise((
    resolve
) => {
    setTimeout(() => {
        resolve([{
                id: 3,
                item: 'hello3'
            },
            {
                id: 4,
                item: 'hello4'
            }
        ])
    }, 1000)
})


export {
    getData,
    getData2
}
```

`src/router.js`我们修改成这样，同样暴露出创建`router`的方法：

```js
// router.js
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

export function createRouter() {
  return new Router({
    mode: 'history',
    routes: [
      // ...
      {
        path: '/hello',
        component: () =>
          import( /* webpackChunkName: 'hello' */ './components/Hello.vue'),
      },
      {
        path: '/hello1',
        component: () =>
          import( /* webpackChunkName: 'hello1' */ './components/Hello1.vue'),
      },
    ],
  })
}
```

接着，在我们创建`vue`实例时，我们把它们绑定到我们的`vue`实例上，同时，我们要使用`vuex-router-sync`插件将新建的`router`和`vuex`同步、相关，最后需要将`store`导出去，因为在客户端、服务端渲染时都需要使用它；

```js
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from './router'
import { createStore } from './store'
import { sync } from 'vuex-router-sync'
// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp() {
  // 为了防止状态数据污染，因此需要给每一个实例都重新创建router和store
  const router = createRouter()
  const store = createStore()

  // 同步路由状态(route state)到 store
  sync(store, router)

  const app = new Vue({
    router,
    store,
    // 根实例简单的渲染应用程序组件。
    render: (h) => h(App),
  })
  return {
    app,
    router,
    store,
  }
}
```

第二步，我们要在服务端渲染时，读取到用户要前往的路由，因为我们不需要其他没有被渲染的路由的数据，然后我们取出这个路由所需要使用的所有路由组件；

我们怎么取出所有路由组件所需要预取的数据呢？

其实只要这样做，我们将一个预取数据的方法绑定在每一个路由组件的对象上，每一个路由组件的预取数据的方法都会将这个路由组件需要的数据取出来（比如通过发送`http`请求获取后端服务器的数据），然后保存到`store`中，这样我们就可以完成对这个路由上所有数据的预取了，但是你要注意，在这个阶段，我们并没有渲染组件，因此`this`是无效的，任何的对元素的操作都是无效的；

首先，我们在一个路由路径上所有的组件都添加一个方法`asyncData`，这个方法会调用`store`的`action`来获取数据并将获取的数据保存到`store.state`中，这样我们就完成了这个路由组件的数据的预取，当然，在真实的应用中，在这一步，我们可以通过对`url`后的参数的解析来确定地返回某一个固定的数据，比如前端请求列表页面，并且带上一个`id`的参数，那么在服务端渲染时就可以读取到这个`id`参数，并在数据预取阶段将确定的列表通过`http`请求得到并存储到`store`中；

如下图所示，我们就在`Hello.vue`文件的导出对象上添加了`asyncData`方法，该方法接收`store route`，因为在这个阶段，我们的组件根本没有被渲染，取不到`this.$store`属性来获取`store`；该方法将`store.dispatch`返回出去，因为这是一个异步方法；

```vue
<template>
  <div>
    hello world{{ item }}
    <p></p>
    <router-link to="/hello1">hello1 link</router-link>
  </div>
</template>

<script>
export default {
  name: 'hello',
  asyncData({ store,route }) {
    return store.dispatch('getDataAction')
  },
  computed: {
    item() {
      return this.$store.state.lists
    },
  },
}
</script>

<style></style>
```

我们修改`entry-server.js`：

+ 首先创建`vue`实例，同时要在原来的基础上将`store`导出来，然后和原来一样，使用代码为我们的`vue`实例设置路由；

+ 当路由设置成功后，我们获取路由上所有的组件，如果该路由上没有组件，说明路由失败，就`reject`；

  如果路由上有组件，那么就遍历所有的组件对象，并且执行该组件对象的`asyncData`方法来拉取数据，这个方法返回一个`promise`，因此我们使用`map`函数来获取路由上所有组件`asyncData`方法返回的`promise`，而我们需要路由上所有的组件都完成数据预取之后进一步操作，因此进一步使用`promise.all`来包裹`asyncData`返回的`promise`，这个方法会依次执行所有的`promise`，直到所有的`promise`完成后执行`then`，同时要注意，我们要传递当前的`router`和`store`给`asyncData`方法；

+ 当所有的路由组件的`asyncData`执行完毕，我们就将该实例的`store.state`传递给`context.state`，原因是我们要使得客户端渲染的`store`数据和服务端渲染的数据一致，因此需要将该`store.state`返回给前端，服务端渲染对`context.state`进行了特殊的处理，会将其作为`window.__INITIAL_STATE__`的数据内容注入到生成的`html`中，然后会在`html`字符串到达前端时将`window.__INITIAL_STATE__`设置好，这样就保证了`store`数据的一致性；同时将`app`的`vue`实例返回给前端；

而我们还需要修改`Hello.vue`文件，使得其对`store`内的数据进行监听，当`store`内的数据改变时更新`Hello.vue`组件依赖的数据，见上图代码；

```js
import {
    createApp
} from './app.js'

export default (context) => {
    // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
    // 以便服务器能够等待所有的内容在渲染前，
    // 就已经准备就绪。
    return new Promise((resolve, reject) => {
        const {
            app,
            router,
            store
        } = createApp() //创建vue实例

        // 设置服务器端 router 的位置，导航到一个新的路由上
        router.push(context.url).catch((err) => err)

        // 等到 router 将可能的异步组件和钩子函数解析完
        router.onReady(() => {
            //获取路由上匹配的所有组件
            const matchedComponents = router.getMatchedComponents()
            if (!matchedComponents.length) {
                return reject({
                    code: 404
                })
            }

            // 对所有匹配的路由组件调用 `asyncData()`
            Promise.all(
                    matchedComponents.map((Component) => {
                        if (Component.asyncData) {
                            return Component.asyncData({
                                store,
                                route: router.currentRoute,
                            })
                        }
                    })
                )
                .then(() => {
                    // 在所有预取钩子(preFetch hook) resolve 后，
                    // 我们的 store 现在已经填充入渲染应用程序所需的状态。
                    // 当我们将状态附加到上下文，
                    // 并且 `template` 选项用于 renderer 时，
                    // 状态将自动序列化为 `window.__INITIAL_STATE__`，并注入 HTML。
                    context.state = store.state
                    resolve(app)
                })
                .catch(reject)
        }, reject)
    })
}
```

返回给前端的`html`代码如下：

```html
<!DOCTYPE html>
<html>

<head>
    <!-- 使用双花括号(double-mustache)进行 HTML 转义插值(HTML-escaped interpolation) -->
    <title>hello ssr with webpack</title>
    <!-- 使用三花括号(triple-mustache)进行 HTML 不转义插值(non-HTML-escaped interpolation) -->
    
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width,initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
        jbhbkkkkkksss
<link rel="preload" href="/dist/app.js" as="script"><link rel="preload" href="/dist/hello1.js" as="script"><link rel="prefetch" href="/dist/hello.js"></head>

<body>
    <div id="app" data-server-rendered="true"><div>hello world1[
  {
    &quot;id&quot;: 3,
    &quot;item&quot;: &quot;hello3&quot;
  },
  {
    &quot;id&quot;: 4,
    &quot;item&quot;: &quot;hello4&quot;
  }
]</div></div><script>window.__INITIAL_STATE__={"lists":[{"id":3,"item":"hello3"},{"id":4,"item":"hello4"}],"route":{"path":"\u002Fhello1","hash":"","query":{},"params":{},"fullPath":"\u002Fhello1","meta":{},"from":{"name":null,"path":"\u002F","hash":"","query":{},"params":{},"fullPath":"\u002F","meta":{}}}}</script><script src="/dist/app.js" defer></script><script src="/dist/hello1.js" defer></script>
</body>

</html>
```

对于客户端渲染来讲，也是一样的套路；

+ 首先，我们创建`vue`实例，并将`app、router、store`导出来，同时我们将`window.__INITIAL_STATE__`替换`store`，这样保证数据的同步；
+ 然后，要注意，我们首次请求页面时，数据已经是最新的了，那么就没必要再拉取一次数据，我们只需要在`router`再次解析路由时更新数据即可，这些操作已然和服务端渲染无关了；我们只要在`router.onReady`中写即可保证首屏加载时不会再次向服务器请求取数据，这个`api`在`router`生命周期中只会执行一次；
+ 然后我们需要在更新路由时更新`store`中的数据，那么我们只要在`router.onReady`中添加一个监听函数`router.beforeResolve`即可，这个`api`会在每一次`router`发生解析时执行，在此之前，动态路由的`js`文件已经下载好了等待解析；在这个`api`的回调函数中，我们需要比较旧路由和新路由的区别，将需要新渲染的路由摘录出来，并依次执行它们的`asyncData`方法更新`store`，就这么简单；

```js
import {
    createApp
} from './app'
import Vue from 'vue'
// 客户端特定引导逻辑……

const {
    app,
    router,
    store
} = createApp()

//如果window.__INITIAL_STATE__存在，说明store存在，就将服务端渲染的store内的数据替代本地的store的state
if (window.__INITIAL_STATE__) {
    store.replaceState(window.__INITIAL_STATE__)
}

router.onReady(() => {
    // 添加路由钩子函数，用于处理 asyncData.
    // 在初始路由 resolve 后执行，
    // 以便我们不会二次预取(double-fetch)已有的数据。
    // 使用 `router.beforeResolve()`，以便确保所有异步组件都 resolve。
    // 全部的异步组件都在等待解析
    router.beforeResolve((to, from, next) => {
        const matched = router.getMatchedComponents(to)
        const prevMatched = router.getMatchedComponents(from)

        // 我们只关心非预渲染的组件
        // 所以我们对比它们，找出两个匹配列表的差异组件
        let diffed = false
        const activated = matched.filter((c, i) => {
            return diffed || (diffed = (prevMatched[i] !== c))
        })

        if (!activated.length) {
            return next()
        }

        // 这里如果有加载指示器 (loading indicator)，就触发

        Promise.all(activated.map(c => {
            if (c.asyncData) {
                return c.asyncData({
                    store,
                    route: to
                })
            }
        })).then(() => {

            // 停止加载指示器(loading indicator)

            next()
        }).catch(next)
    })
    app.$mount('#app')
})
```

但是，这样做其实体验并不好，在每一次路由解析前获取所有数据后才切换页面，如果数据的获取很漫长，那么页面就会长时间不跳转，因此，会给用户一种割裂、速度缓慢的感觉；

因此，为了解决这个问题，我们也可以使用下面的方案，这个方案会在页面跳转时快速跳转并显示，但是数据的加载是异步的，所以会显示页面的框架，数据会缓慢填充进来，但是体验变好了一些，并且代码更加精简：

```js
Vue.mixin({
  beforeMount () {
    const { asyncData } = this.$options
    if (asyncData) {
      // 将获取数据操作分配给 promise
      // 以便在组件中，我们可以在数据准备就绪后
      // 通过运行 `this.dataPromise.then(...)` 来执行其他任务
      this.dataPromise = asyncData({
        store: this.$store,
        route: this.$route
      })
    }
  }
})
```

这样，我们就实现了一整套数据预取的逻辑；









































