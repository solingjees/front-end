# [微信小程序]小程序框架对比

![截屏2020-10-24 01.00.36](/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-24 01.00.36.png)

+ `Taro`：使用`React`开发方式，周边生态较为强大，有`Taro UI`（多端适配、组件丰富）、`AT-UI`等`UI`框架；

+ `Wepy`：微信官方的小程序框架，由于历史遗留原因没有集成到原生小程序中（无语.....），提供了组件化、`promise`、`npm`、`es6`、`sass`等`css`预编译语言 等原生小程序中原先不支持（现在有些支持了）的扩展的特性，就是原生小程序的细节优化，还支持多种 插件，比如打包优化；

  它具有以下特性：

  + 使用`Vue Observer`实现数据绑定；
  + 支持`Vue watch / computed /mixin`等特性；
  + 基于原生组件实现组件化开发；
  + 支持`TypeScript`（原生支持）；

+ `uni-app`：适合小型的、需要快速原型和交付、多端开发的项目，完美支持`app`端，只支持`HBuilder`编辑器才能写`uni-app`；

+ `mpvue`：它具有以下优点：

  + 使用`vue`的语法编写，`vue`学习曲线相较于`react`更加平缓，中小企业使用较多；
  + 开发效率较高，开发迭代了`2`个大版本，比较稳定；

+ 原生：在项目较小、性能要求较高时使用；

是否支持`h5`不是非常重要，因为在小程序的基础上，代码不需要大改就可以支持`h5`了，高度可复用；

同时支持`h5`的框架，还是要为`h5`单独编写不同的逻辑，因此本质上是一致的；

更多对比参考`https://ask.dcloud.net.cn/article/35867`；

# [`mpvue`]第一个`mapvue`项目

## 原理

+ `mpvue`保留继承了`vue.runtime`的核心语法和`vue.js`的基础能力；
+ `mpvue-template-compiler` 提供了将` vue` 的模板语法转换到小程序的` wxml` 语法的能力；
+ 修改了 `vue` 的建构配置，使之构建出符合小程序项目结构的代码格式： `json/wxml/wxss/js` 文件；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-24 23.05.19.png" alt="截屏2020-10-24 23.05.19" style="zoom:50%;" />



## 生命周期

`mpvue`支持所有的主流`vue.js`生命周期，包括：`beforeCreate`、created`、beforeMount`、`mounted`、`beforeUpdate`、`updated`、`activated`、`deactivated`、`beforeDestroy`、`destroyed`；

但是要注意，小程序`onReady`后才触发`mounted`生命周期；

同时还支持微信小程序`app`和`page`的原生的生命周期，包括：`beforeCreate`、`created`、`beforeMount`、`mounted`、`beforeUpdate`、`updated`、`activated`、`deactivated`、`beforeDestroy`、`destroyed`、`onLaunch`、`onShow`、`onHide`；

但是，尽量不要使用小程序的生命周期；

## `mpvue`中的`vue`

+ `mpvue`本身已经集成了`px2px-loader`来将我们的`px`转化为`rpx`，因此不需要担心样式属性单位换算的问题；

+ 要注意，小程序本身不支持路由，在小程序中，每一个页面都是`webview`，都是单独的页面，是一个多页面的应用，而传统的`vue`创建的都是单页面应用，`vue-router`不同路由的跳转实际上还是在单个页面中进行跳转，因此在多页面的应用中是无法跳转到其他页面的；

  页面跳转是小程序的行为，我们调用：

  ```
   wx.navigateTo({
        url: '../logs/logs'
   })
  ```

  进行跳转；

+ `mpvue`中无法使用`v-html`指令、`BOM/DOM`等所有属性、方法；

+ `template`中不支持复杂的`javascript`表达式，比如：

  ```
  <!-- 这种就不支持，建议写 computed -->
  <p>{{ message.split('').reverse().join('') }}</p>
  ```

  目前可以使用的有 `+ - * % ?: ! == === > < [] .`，剩下的还待完善；

+ `template`不支持过滤器；

+ 不支持在` template` 内使用`methods` 中的函数；

+ 因为微信小程序渲染时，`class`还是要绑定地动态地渲染的，因此为节约性能，我们将 `Class` 与` Style` 的表达式通过 `compiler` 硬编码到 `wxml `中，不做任何改变，支持语法如下：

  `class `支持的语法:

  ```
  <p :class="{ active: isActive }">111</p>
  <p class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }">222</p>
  <p class="static" :class="[activeClass, errorClass]">333</p>
  <p class="static" v-bind:class="[isActive ? activeClass : '', errorClass]">444</p>
  <p class="static" v-bind:class="[{ active: isActive }, errorClass]">555</p>
  ```

  style 支持的语法:

  ```text
  <p v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }">666</p>
  <p v-bind:style="[{ color: activeColor, fontSize: fontSize + 'px' }]">777</p>
  ```

  不支持 [官方文档：Class 与 Style 绑定](https://cn.vuejs.org/v2/guide/class-and-style.html) 中的 `classObject` 和 `styleObject` 语法。

+ 嵌套列表渲染，必须指定不同的索引；

  ```
  <!-- 在这种嵌套循环的时候， index 和 itemIndex 这种索引是必须指定，且别名不能相同，正确的写法如下 -->
  <template>
      <ul v-for="(card, index) in list">
          <li v-for="(item, itemIndex) in card">
              {{item.value}}
          </li>
      </ul>
  </template>
  ```

+ 几乎全支持事件处理器；`mpvue`引入了 `Vue.js` 的虚拟 `DOM`，在前文模版中绑定的事件会被挂在到` vnode` 上，同时我们的` compiler` 在` wxml` 上绑定了小程序的事件，并做了相应的映射，所以你在真实点击的时候通过` runtime `中`handleProxy`通过事件类型分发到 `vnode` 的事件上，同 `Vue` 在` WEB` 的机制一样，所以可以做到无损支持。同时还顺便支持了自定义事件和 `$emit` 机制。

  在 `input` 和 `textarea` 中 `change` 事件会被转为 `blur` 事件。

更多注意点参考`http://mpvue.com/mpvue/`，这个部分比较重要；

## 启动第一个`mpvue`程序

`mpvue`只需要使用`vue-cli`即可以安装，因为本质上是`vue`的魔改，还是属于`vue`系的；

首先，我们安装`@vue/cli`工具；

```
npm install --global @vue/cli
```

接着，我们要使用官方的模版初始化一个`mpvue`项目，在此之前，我们安装`vue-cli-init`模块，否则无法使用`vue init`命令；

```
npm install --global @vue/cli-init
```

接下来，我们开始初始化`mpvue`项目：

```
vue init mpvue/mpvue-quickstart solingjees-wxapp
```

这时候，会进入一个交互式输入状态，按需填写即可：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-25 01.27.02.png" alt="截屏2020-10-25 01.27.02" style="zoom:50%;" />

其中，要注意填写`appid`，但即使填错了，后续也可以在微信开发者工具中更改；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-25 01.52.04.png" alt="截屏2020-10-25 01.52.04" style="zoom:50%;" />

要注意，我们的项目初始化后是没有任何依赖的，我们`cd`到项目目录下，下载一下依赖：

```
npm install
```

最后，我们将项目导入微信开发者工具，方便我们调试；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-25 01.30.29.png" alt="截屏2020-10-25 01.30.29" style="zoom:50%;" />

找到项目后会自动填充项目的`AppID`，不需要手动再填写了；

最后，我们使用以下命令启动`mpvue`开发服务器；

```
npm run dev
```

一般来讲，我们使用`vs code`来开发微信小程序，微信开发者工具只作为调试器，这是因为微信开发者工具对`mpvue`这类`vue`框架不太友好，没有语法高亮等一系列编辑器辅助，不太方便，而`vs code`是有的，因此对开发会方便一些、容易一些；

同时，初始化项目已经安装了`eslint`和`prettier`，你可以再下载一个`prettier-eslint npm`插件，它能保证在每一次`eslint --fix`之前进行一次`prettier`格式化；

> `eslint`和`prettier`最大的区别是`prettier`可以格式化 `.css`, `.less`, `.scss`, or `.json`文件，而`eslint`不能；

# [`mpvue`]目录结构及重要注意点

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-29 23.01.03.png" alt="截屏2020-10-29 23.01.03" style="zoom:30%;" />

+ `dist`目录下包含的是`mpvue`打包生成的目录，是经过`webpack`打包后可以部署到微信小程序的包，其内部的`wxml`文件悉数经过了打包压缩，`wxss`经过了预编译和格式转换（`px`转`rpx`）；

+ `webpack`等一系列的打包配置都在`build`文件夹下；

  <img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-10-29 23.06.29.png" alt="截屏2020-10-29 23.06.29" style="zoom:40%;" />

+ `+WXSS`的格式转换的配置在`utils.js`文件中，我们找到该文件的这一行：

  ```
   var px2rpxLoader = {
      loader: 'px2rpx-loader',
      options: {
        baseDpr: 1,
        rpxUnit: 0.5 //1px = 2rpx
      }
    }
  ```

  该部分就配置了格式转换的配置，你可以更改`rpxUnit`，如果`UI`设计稿提供的是`750px`，那么就可以设置`rpxUnit`为`1`，这样，我们写的`px`都会被`1:1`替代为`rpx`；

+ 在`webpack`的配置中，我们着重注意`webpack.bacis.conf.js`：

  ```
   resolve: {
      extensions: ['.js', '.vue', '.json'],
      alias: {
        'vue': 'mpvue',
        '@': resolve('src')
      },
      ...
    },
  ```

  上面是引用的基地址，是我们着重要注意的一点，能很大程度上方便我们的开发；

+ 由于不同的平台（`百度`、`微信`、`支付宝`等）有不同的`api`，因此需要在使用一些平台特有的`api`前进行检查，`mpvue`提供了检查平台的能力，在`app.js`中简要说明了不同平台的判断方法：

  ```
    /*
       * 平台 api 差异的处理方式:  api 方法统一挂载到 mpvue 名称空间, 平台判断通过 mpvuePlatform 特征字符串
       * 微信：mpvue === wx, mpvuePlatform === 'wx'
       * 头条：mpvue === tt, mpvuePlatform === 'tt'
       * 百度：mpvue === swan, mpvuePlatform === 'swan'
       * 支付宝(蚂蚁)：mpvue === my, mpvuePlatform === 'my'
       */
  ```

  下方就是一个应用：

  ```
   if (mpvuePlatform === 'my') {
        logs = mpvue.getStorageSync({key: 'logs'}).data || []
        logs.unshift(Date.now())
        mpvue.setStorageSync({
          key: 'logs',
          data: logs
        })
      } 
  ```

+ 如果你需要在生产模式下使用`sourceMap`，那么可以在`config/index.js`文件中打开：

  ```
   build: {
      ...
      productionSourceMap: false,
      ...
    },
  ```

+ 在`build`目录下，有`dev-client.js`和`dev-server.js`，说明`mpvue`打包后会构成一个支持服务端渲染的服务器；

+ `build`目录下的`check-versions.js`用于检查我们的项目当前所使用的`node`和`npm`版本是否符合我们规定的要求，我们可以在`package.json`中设置我们要求项目所满足的版本，具体的原理其实非常简单，就是获取`packageConfig.engines.npm`和`pa ckageConfig.engines.node`和当前系统使用的`npm`和`node`进行比较，如果不符合就停止`npm run dev`就行了，然后使用`chalk`组件显示信息到控制台即可，具体过程参考代码，相信几分钟就可以明白原理了；

  ```
   "engines": {
      "node": ">= 4.0.0",
      "npm": ">= 3.0.0"
    },
  ```

  ```js
  var chalk = require('chalk')
  var semver = require('semver')
  var packageConfig = require('../package.json')
  var shell = require('shelljs')
  function exec (cmd) {
    return require('child_process').execSync(cmd).toString().trim()
  }
  
  var versionRequirements = [
    {
      name: 'node',
      currentVersion: semver.clean(process.version), //获取当前版本,process.version就是node版本
      versionRequirement: packageConfig.engines.nodev //获取package.json中的预设版本
    }
  ]
  
  //shell.which查找npm的命令文件的执行路径，如果存在，说明安装了npm包管理器，可以使用全局命令npm --version
  if (shell.which('npm')) {
    versionRequirements.push({
      name: 'npm',
      currentVersion: exec('npm --version'), //npm属于系统工具，使用命令来获取
      versionRequirement: packageConfig.engines.npm
    })
  }
  
  module.exports = function () {
    var warnings = []//存放警告
    for (var i = 0; i < versionRequirements.length; i++) {
      var mod = versionRequirements[i]
      //一一比较
      if (!semver.satisfies(mod.currentVersion, mod.versionRequirement)) {
       //比较失败就报错
        warnings.push(mod.name + ': ' +
          chalk.red(mod.currentVersion) + ' should be ' +
          chalk.green(mod.versionRequirement)
        )
      }
    }
    
    //如果有警告信息，就打印出来
    if (warnings.length) {
      console.log('')
      console.log(chalk.yellow('To use this template, you must update following to modules:'))
      console.log()
      for (var i = 0; i < warnings.length; i++) {
        var warning = warnings[i]
        console.log('  ' + warning)
      }
      console.log()
      //强制退出进程，停止启动服务器
      process.exit(1)
    }
  }
  ```

# [`vant`、`mpvue`]在`mpvue`中使用`vant`作为`UI`框架

首先，我们需要在`mpvue`项目根目录中`npm run dev`，来生成`dist/wx`目录，这个目录是最后打包生成的微信原生小程序包；

由于我们的`vant`是给微信原生小程序使用的，不是给`mpvue`使用的，因此我们需要在`dist/wx`目录下安装`vant`，而不是在`mpvue`项目根目录下安装；

我们进入`dist/wx`目录，接着`npm init -y`初始化`npm`，接着安装：

```
npm i @vant/weapp -S --production
```

这样还不够，别忘了我们的`npm`包并不能直接被微信小程序使用，我们还需要`构建npm`一次；

我们前往`微信开发者工具app`，接着点击右上角详情--本地设置，勾选`ES6转ES5`、`使用npm模块`、`不校验合法域名`（主要是第二个，其他两个没勾的要勾上），然后`工具----构建npm`，等待一会儿`vant`就成功构建到小程序中了；

> 注意点：
>
> + 如果你的`prettier`出现版本过低的情况，最好安装`v2`的版本，`latest`的版本可能出现一些兼容性的问题；
>
> + 当我们实际编写代码时比较多地使用`sass`，但原生`quickStart`模版是不提供`sass-loader`的，我们需要自行安装：
>
>   ```
>   npm install sass-loader@7 node-sass
>   ```
>
>   注意，`sass-loader`最好要`7`左右的版本，`8`的版本可能会出现一些问题；
>
>   在其他的一些框架中，我们需要配置`webpack`才能使用`sass`，但是`mpvue`已经配置好了`webpack`对这些预编译器的`loaders`，具体可以参考`build/utils.js`，因此我们不再需要配置了；
>
> + 如果出现`js`末尾带逗号的问题，那么就在`.prettierrc`文件中添加对他的修改：
>
>   ```
>     "trailingComma": "none"
>   ```

`vant`使用也非常简单，我们首先在`app.json`中引入要使用的组件，比如我们要引入`van-button`：

```
"usingComponents": {
  "van-button": "@vant/weapp/button/index"
}
```

然后我们就可以直接在`vue`文件中使用该组件了：

```
<van-button type="default">默认按钮</van-button>
```

# [微信小程序]在原生微信小程序中使用完全功能`svg`的方法

在一般的`web`前端应用程序中，我们使用`svg`标签去使用`svg`图片；

但是令人遗憾的是，微信小程序压根不支持`svg`标签，导致我们只能使用`img`标签引用`svg`图片（不是`svg`不支持了，只是`svg`标签不支持，但是`img`标签是支持`svg`的）：

```html
<img src="/static/icons/home.svg" alt="" style="{width:60rpx,height:60rpx}"/>
```

看上去曲线救国了，实则不然，封装的`svg`图片的`img`标签并没有设置`svg`图片颜色的能力，因此，这样并没有达到我们的要求，我们要能够自定义我们的图标的颜色；

要怎么达到我们的目的？

一张`svg`图片的内容其实就是一段`html`代码：

```html
<svg viewBox="64 64 896 896" focusable="false" xmlns="http://www.w3.org/2000/svg">
   <path d="M946.5 505L534.6 93.4a31.93 31.93 0 00-45.2 0L77.5 505c-12 12-18.8 28.3-18.8 45.3 0 35.3 28.7 64 64 64h43.4V908c0 17.7 14.3 32 32 32H448V716h112v224h265.9c17.7 0 32-14.3 32-32V614.3h43.4c17 0 33.3-6.7 45.3-18.8 24.9-25 24.9-65.5-.1-90.5z" />
</svg>
```

这段代码将被传递给`img`标签的`src`属性进行渲染；

那么其实很明显，我们要在`svg`图片被传递给`img`标签渲染之前，修改`svg`图片代码里的`path`元素的`fill`属性来修改`svg`图片的颜色，这样就能保证我们的图片的颜色是我们想要的；

要怎么做呢？

首先，我们要读取`svg`图片文件的内容，即其内部的`html`字符串，由于这已经不是`node`环境了，因此我们必须使用`wx`的`api`来读取`svg`的`html`字符串：

```js
  wx.getFileSystemManager().readFile({
        filePath: "/static/icons/home.svg", //要读取的svg图片文件
        encoding: 'binary', //二进制编码
        success: (res) => { //读取成功后，可以从res.data中获取html字符串
        }
  )}
```

接下来，我们就是要去添加`fill`属性到`<path `子字符串后面，使用正则表达式进行匹配即可，如果有了`fill`属性，就修改它，这样我们就组成了一段指定了具体颜色的`svg`的`html`片段了；

```js
 wx.getFileSystemManager().readFile({
        filePath: this.src,
        encoding: 'binary',
        success: (res) => {
          //匹配看是否有fill属性
          const reg = /fill="#[a-zA-Z0-9]{0,6}"/g //注意是全部匹配
          let str = ''
          // 2.修改fill属性 ，svg -> fill 判断是否有fill属性
          if (reg.test(res.data)) {
            // fill属性存在
            str = res.data.replace(reg, `fill="${this.color}"`) //修改颜色
          } else {
            str = res.data.replace(/<path\s/g, `<path fill="${this.color}" `) //添加颜色
          }
         ...
        }
      })
```

最后，我们要将其转译为`base64`，为什么？因为我们已经将其从文件中读出来了，我们不会将其再次存储为文件，那样太复杂了，`img`支持`base64`编码的`svg`，因此，我们可以直接将其转译为`base64`编码，这样方便一些，我们使用`js-base64`这个库来实现，请自行安装，接下来使用它进行转译，但是生成的`base64`字符串是不带有任何描述符的，我们需要额外自行添加描述符：

```js
   wx.getFileSystemManager().readFile({
        filePath: this.src,
        encoding: 'binary',
        success: (res) => {
          const reg = /fill="#[a-zA-Z0-9]{0,6}"/g
          let str = ''
          // 2.修改fill属性 ，svg -> fill 判断是否有fill属性
          if (reg.test(res.data)) {
            // fill属性存在
            str = res.data.replace(reg, `fill="${this.color}"`)
          } else {
            str = res.data.replace(/<path\s/g, `<path fill="${this.color}" `)
          }
          // 3.base64 svg -> base64
          const base64 = Base64.encode(str) //转化为base64
          // 4.imgUrl = base64
          this.imgUrl = 'data:image/svg+xml;base64,' + base64 //添加描述符生成
        }
      })
```

这样，我们生成的`imgUrl`就是一段可以直接作为渲染对象的`svg`了，且完美支持颜色自定义，将其赋值给`img`标签的`src`属性即可，完整代码如下：

```vue
<template>
  <div>
    <img :src="imgUrl" :style="{width: size + 'rpx' , height: size + 'rpx'}" alt="" />
  </div>
</template>

<script>
import {Base64} from 'js-base64'
export default {
  name: 'icon',
  props: {
    size: {
      type: [Number, String],
      default: 36
    },
    src: [String],
    color: [String]
  },
  data () {
    // base64 -> image
    return {
      imgUrl: ''
    }
  },
  created () {
    if (this.color) {
      // 1.读取svg图片
      wx.getFileSystemManager().readFile({
        filePath: this.src,
        encoding: 'binary',
        success: (res) => {
          const reg = /fill="#[a-zA-Z0-9]{0,6}"/g
          let str = ''
          // 2.修改fill属性 ，svg -> fill 判断是否有fill属性
          if (reg.test(res.data)) {
            // fill属性存在
            str = res.data.replace(reg, `fill="${this.color}"`)
          } else {
            str = res.data.replace(/<path\s/g, `<path fill="${this.color}" `)
          }
          // 3.base64 svg -> base64
          const base64 = Base64.encode(str)
          // 4.imgUrl = base64
          this.imgUrl = 'data:image/svg+xml;base64,' + base64
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped></style>
```

我们这样使用这个组件：

```
  <Icon src="/static/icons/home.svg" size="60" color="#02d199"></Icon> 
```

# [微信小程序]`tabBar`

## 构造原生`tabBar`

设置底部导航栏是微信小程序的原生功能，`mpvue`并没有进行封装；

微信小程序将`tabBar`的设置的方法设计的足够简单，我们直接在`app.json`中添加这样的配置即可；

```json
{
  ...
  //表明是tabBar的配置
  "tabBar": {
        "color": "#999", //前景色
        "backgroundColor": "#fafafa", //背景色
        "selectedColor": "#02d199", //选中的时候的前景色
        "borderStyle": "white", //上边界的颜色
        //渲染的导航栏项目，至少2个，最多5个
        "list": [
            {
                "text": "首页", //显示的文字
                "pagePath": "pages/index/main", //路由对应的页面
                "iconPath": "static/images/tab_home_no.png", //没有选中该项时，所显示的图标所在的位置
                "selectedIconPath": "static/images/tab_home_yes.png" //没有选中该项时，所显示的图标所在的位置
            },
            {
                "text": "消息",
                "pagePath": "pages/index/main",
                "iconPath": "static/images/tab_news_no.png",
                "selectedIconPath": "static/images/tab_news_yes.png"
            },
            {
                "text": "热门",
                "pagePath": "pages/index/main",
                "iconPath": "static/images/tab_popular_no.png",
                "selectedIconPath": "static/images/tab_popular_yes.png"
            },
            {
                "text": "我的",
                "pagePath": "pages/index/main",
                "iconPath": "static/images/tab_my_no.png",
                "selectedIconPath": "static/images/tab_my_yes.png"
            }
        ],
        "position": "bottom" //导航条所在屏幕的位置
    },
    ...
 }
```

要注意，这里配置的所有的图片都不是`svg`格式，如果你想要使用`svg`格式，那么必须要自定义`tabBar`；

微信小程序提供了自定义`tabBar`的能力，在上面的应用中我们也看到了，微信小程序提供的原生的能力比较薄弱。因此，如果我们希望使用更加强大的导航栏，那么必须自定义；

## 构造自定义 `tabBar`

原生的`tabBar`能力比较弱小，当我们需要自定义一些比较酷炫的功能时，原生的`tabBar`是无法完成的，因此我们需要自定义我们的`tabBar`；

定义方法很简单，首先我们配置`app.json；`

```
 "tabBar": {
        ...
        "custom":true
    },
```

这样，原生的`tabBar`就被隐藏掉了；

然后，我们新建一个新的`tabBar`的`vue`组件，作为我们自定义的`tabBar`；

为了保证我们的`tabBar`的层级最高，因此我们使用`cover-view`和`cover-image`包裹我们的组件；

我们接下来只要按照经典的`vue`组件的构建步骤来创建`tabBar`即可；

但是有一些要注意的地方：

+ 我们使用了在之前的知识点中创建的`Icon`组件，因为这个名称是微信小程序的保留字，因此必须映射为一个新的名字，如下就映射成了`v-icon`；
+ 不同的`svg`图标在未经过任何尺寸修饰情况下尺寸、设计观感可能是不一样的，我们需要对个例进行微调，来使的我们的图标都是统一、整齐的；
+ 由于当用户选中不同的`tabBar`菜单选项时会修改`v-icon`的`color`，因此，我们必须在`Icon`组件中添加对`color props`的监听来保证`Icon`能够重新生成`svg`图片；

```vue
<template>
  <div>
    <cover-view class="tab-bar">
      <!-- 上边界 -->
      <!-- <cover-view class="tab-bar-border"></cover-view> -->
      <!-- 实体 -->
      <cover-view
        v-for="(item, index) in list"
        :key="index"
        class="tab-bar-item"
        :data-index="index"
        :data-path="item.pagePath"
        @click="switchTab(item,index)"
      >
        <v-icon
          :src="item.iconPath"
          :size="item.size"
          :color="selected === index ? selectedColor:color"
        ></v-icon>
        <cover-view :style="{ 'color': selected === index ? selectedColor : color }">{{item.text}}</cover-view>
      </cover-view>
    </cover-view>
  </div>
</template>

<script>
import Icon from '@/components/icon/index'
export default {
  name: 'tabbar',
  components: {
    'v-icon': Icon
  },
  data () {
    return {
      selected: 0,
      color: '#7a7e83',
      selectedColor: '#02d199',
      list: [
        {
          text: '首页',
          pagePath: '/pages/index/main',
          iconPath: '/static/icons/home.svg',
          size: 60
        },
        {
          text: '消息',
          pagePath: '/pages/info/main',
          iconPath: '/static/icons/msg.svg',
          size: 50
        },
        {
          text: '热门',
          pagePath: '/pages/index/main',
          iconPath: '/static/icons/faved.svg',
          size: 50
        },
        {
          text: '我的',
          pagePath: '/pages/index/main',
          iconPath: '/static/icons/person.svg',
          size: 60
        }
      ]
    }
  },
  methods: {
    switchTab (item, index) {
      // const url = data.pagePath
      // wx.switchTab({ url: item.pagePath })
      this.selected = index
    }
  }
}
</script>

<style lang="scss" scoped>
.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 48px;
  background: white;
  display: flex;
  padding-bottom: 5px;
  box-shadow: 0px -4px 6px 0px rgba(36, 36, 37, 0.08);
}

// // 上边界
// .tab-bar-border {
//   background-color: rgba(0, 0, 0, 0.33);
//   position: absolute;
//   left: 0;
//   width: 100%;
//   height: 1px;
//   transform: scaleY(0.5);
// }

.tab-bar-item {
  flex: 1;
  text-align: center;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  flex-direction: column;
}

.tab-bar-item cover-image {
  width: 27px;
  height: 27px;
}

.tab-bar-item cover-view {
  font-size: 10px;
}
</style>
```

整体创建的步骤和普通的`vue`组件别无二致，只要使用语句：

```
wx.switchTab({ url: item.pagePath })
```

来切换路由页面即可；

然后我们应用这个组件到`pages/index/index.vue`内，就可以使用`tabBar`了；

## 使用`vant`的`tabBar`

或许，有了更加优秀的第三方的方案；

`vant`提供了一个`navBar`组件，也就是`tabBar`；

首先，我们在`app.json`中引入`van-nav-bar`：

```
"usingComponents": {
  "van-nav-bar": "@vant/weapp/nav-bar/index"
}
```

接下来，我们在组件中使用`van-nav-bar`：

```vue
 <template>
  <div>
    <van-tabbar
      :active="active"
      @change="onChange"
    >
      <van-tabbar-item icon="home-o">标签</van-tabbar-item>
      <van-tabbar-item icon="search">标签</van-tabbar-item>
      <van-tabbar-item icon="friends-o">标签</van-tabbar-item>
      <van-tabbar-item icon="setting-o">标签</van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      active: 0
    }
  },
  methods: {
    onChange (event) {
    // event.detail 的值为当前选中项的索引
      this.active = event.mp.detail
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
```

当用户点击非当前页面的的标签、`tab`页面想要发生更换时，`van-bar`会触发`onChange`事件，会将当前要前往的`tab`页面的`index`传递给`event.mp.detail`，`mp`是小程序底层的组件所传递给`event`的对象，因此我们可以获取到用户想去的页面，并将其赋值给`active`，并将`active`传递给`van-tabbar`的`active`用于点亮具体的标签；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 13.59.48.png" alt="截屏2020-11-08 13.59.48" style="zoom:50%;" />



# [`CSS`]`flex`布局

`flex`布局是非常重要的布局方式，它革新了以往的传统布局形式，用更加直观的方式来设置我们的布局；

`flex`布局对接受`flex`布局的元素的内部元素的布局生效；

同时，我们的坐标是从左上角开始计算的，作为`start`位置；

其中首先最重要的是以下几个属性：

+ `flex-direction`：`flex`布局的主轴，可以设置为`column`、`row`；

+ `justify-content`：内部元素在`flex`布局主轴上的位置，主要可以设置为`flex-start`、`flex-end`、`center`、`space-around`（等间隙距离排布，其中最左侧、最右侧的空格是中间间隙的一半）、`space-evenly`（等间隙距离排布，最左侧、最右侧的空格和中间间隔间隙相同）；

  下图是`space-around`：

  <img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 00.35.22.png" alt="截屏2020-11-08 00.35.22" style="zoom:50%;" />

  下图是`space-evenly`：

  <img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 00.36.04.png" alt="截屏2020-11-08 00.36.04" style="zoom:50%;" />

  > + 当你的`flex`布局处于`row`当方向时，如果不想用`justify-content`来做横向的对齐，并且你的`flex`布局内部的元素都是文本类元素，那么可以使用`text-align`来代替，它可以起到相同的效果，同时它不依赖`flex`布局；
  > + 如果外层`div`使用了`flex`布局，那么内部全部的元素都会按照`flex`的`align-items`进行副轴上的定位，比如父元素设置`align-items`为`flex-start`，如果有一个内部元素有特殊要求，比如某一个内部元素需要在父元素中的副轴定位设置为`flex-start`，但是又不想影响到其他同级的子元素在父元素中的定位，那么可以在该内部元素上使用`align-self`来做个性化定位处理，该属性会覆盖父元素的`align-items`，使用`align-self`所指定的在父元素中的副轴定位；

+ `align-items`：内部元素在`flex`布局副轴上的位置，可以设置为`flex-start`、`flex-end`、`center`，`space-around`、`space-evenly`；

内部元素在不同的参数下的位置表现如下图；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 00.06.38.png" alt="截屏2020-11-08 00.06.38" style="zoom:50%;" />

比如，下方就是一个`justify-content:flex-start; align-items:center`的例子；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 00.20.26.png" alt="截屏2020-11-08 00.20.26" style="zoom:50%;" />

如果在此基础上再添加一个`flex-direction:column`，那么我们的`justify-content`就会和`align-items`互换，因为主轴变成了纵向轴；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 00.26.26.png" alt="截屏2020-11-08 00.26.26" style="zoom:50%;" />

# [`vant`]使用第三方`iconfont`图标库

`vant`所提供的图标难免会出现难以满足我们需求的情况，因此我们还是希望能尽量使用`iconfont`图标库；

那么怎么使用呢？

如果你有一个`iconfont`图标库，那么你就可以直接在`iconfont`项目的控制台中点击`新icon来袭`来生成一个能够引用自己的`iconfont`图标项目下的所有图标的`css`文件；

但是在生成之前，我们需要将`fontclass`前缀修改为这样的一种格式`<Font family名称>-`；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 14.38.09.png" alt="截屏2020-11-08 14.38.09" style="zoom:50%;" />

假设我们的`font family`设置为`iconfont`，那么`close`图标的类名就是`.iconfont-close`，等一下在组件中使用时你就知道为什么要这样做了；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 14.18.08.png" alt="截屏2020-11-08 14.18.08" style="zoom:50%;" />

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 14.18.48.png" alt="截屏2020-11-08 14.18.48" style="zoom:50%;" />

接着我们可以直接引用该`css`文件，在`index.html`中直接内联该`css`文件；

但是如果该`css`文件下载地址遥远或者一些其他原因导致`css`文件下载缓慢，可能会影响整个页面的显示，`swiper.js`就是这样一个库，因此最好下载到本地；

我们可以在在浏览器新标签页面直接打开这个`css`文件的文件地址，然后复制该`css`内的全部内容；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 14.21.32.png" alt="截屏2020-11-08 14.21.32" style="zoom:67%;" />

最后粘贴到`app.vue`中的`style`标签内部就可以本地地全局使用`css`图标了；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 14.22.46.png" alt="截屏2020-11-08 14.22.46" style="zoom:33%;" />

那么我们怎么使用呢？`vant`提供了`van-icon`组件，该组件用于显示图标，其中最重要的有两个属性，一个是`name`属性，一个是`class-prefix`属性，`name`属性表示我们所需要使用的图标的无前缀的名称，而`class-prefix`表示我们需要使用的前缀，同时还表示我们要使用的`font-family`，默认是`van-icon-`；

```
<van-icon
      name="home"
      size="40"
      color="#000fff"
      class-prefix="iconfont"
 ></van-icon>
```

如果我们不设置`class-prefix`，编译后相当于这样，其中，`van-icon`类表示`font-family`，`van-icon-home`表示具体的图标：

```
<div class"van-icon van-icon-home"
 ></div>
```

我们自己导入的`iconfont.css`内容如下：

```
@font-face {
  font-family: 'iconfont';
  src: url('//at.alicdn.com/t/font_1791095_cuhp3xhlxmp.eot?t=1588159188890'); /* IE9 */
  src: url('//at.alicdn.com/t/font_1791095_cuhp3xhlxmp.eot?t=1588159188890#iefix')
      format('embedded-opentype'),
    /* IE6-IE8 */
      url('xxx')
      format('woff2'),
    url('//at.alicdn.com/t/font_1791095_cuhp3xhlxmp.woff?t=1588159188890')
      format('woff'),
    url('//at.alicdn.com/t/font_1791095_cuhp3xhlxmp.ttf?t=1588159188890')
      format('truetype'),
    /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
      url('//at.alicdn.com/t/font_1791095_cuhp3xhlxmp.svg?t=1588159188890#iconfont')
      format('svg'); /* iOS 4.1- */
}

.iconfont {
  font-family: 'iconfont' !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.iconfont-home:before {
  content: '\e610';
}
```

那么如果要使用我们自己导入的图标，我们只要修改`class-prefix`就行，比如设置为`ficonfont`，那么编译后的结果就相当于：

```
<div class"iconfont iconfont-home"
 ></div>
```

这也是为什么`class-prefix`必须和`font-family`一致的原因；‘如果设置`class-prefix`为`fonta`，`font-family`设置为`iconfont`，那么我们的`iconfont.css`就会是这样：

```
@font-face {
  font-family: 'iconfont';
  ...
}

.iconfont {
  font-family: 'iconfont' !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.fonta-home:before {
  content: '\e610';
}
```

但是`van-icon`在引用中如果设置`class-prefix`为`fonta`，编译后就是这样的：

```
<div class"fonta fonta-home"
 ></div>
```

很明显，我们并没有`fonta`这个`font-family`，因此就会出现图标不存在的问题，所以两者必须是一致的，否则引用会出错；

# [`vant`]`taBbar`的最终效果

根据上面三块内容的讲解，我们完成了一个是用`vant`、`iconfont`、`flex布局`制作的`tabBar`，代码如下：

```vue
<template>
  <div>

    <van-tabbar
      active-color="#02d199"
      inactive-color="#ccc"
      :active="active"
      @change="onChange"
    >
      <van-tabbar-item class="item">
        <van-icon
          name="home"
          size="28"
          class-prefix="iconfont"
        ></van-icon>
        <div>首页</div>
      </van-tabbar-item>
      <van-tabbar-item class="item small">
        <van-icon
          name="msg"
          size="24"
          class-prefix="iconfont"
        ></van-icon>
        <div>消息</div>
      </van-tabbar-item>
      <van-tabbar-item class="item small">
        <van-icon
          name="faved"
          size="24"
          class-prefix="iconfont"
        ></van-icon>
        <div>热门</div>
      </van-tabbar-item>
      <van-tabbar-item class="item">
        <van-icon
          name="person"
          size="28"
          class-prefix="iconfont"
        ></van-icon>
        <div>我的</div>
      </van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      active: 0
    }
  },
  methods: {
    onChange (event) {
    // event.detail 的值为当前选中项的索引
      this.active = event.mp.detail
    }
  }
}
</script>

<style lang="scss" scoped>
.item {
  text-align: center;
  align-self: flex-end;
  padding-bottom: 5px;
  div {
    padding-top: 5px;
  }
}
</style>
```

# [`vs code`]`eslint`和`prettier`的冲突问题

在`vue`项目中，我们一般使用`vetur`作为默认的编辑器格式化工具：

```
  "[vue]": {
    "editor.defaultFormatter": "octref.vetur"
  },
```

在使用了`eslint`作为检查格式、纠错格式的情况下，我不建议使用`prettier`作为`vetur`的格式化具体`js`、`html`的工具，`html`部分使用`js-beautiful-html`，`js`部分直接设为`none`，这是因为`prettier`和`eslint`总会发生冲突，比如它们两者对函数名后面是否有括号往往是使用不同的策略的，`vetur`在`eslint`格式化之后执行，会覆盖`eslint`的格式化结果，导致后来`eslint`检查错误是失败的，进一步导致编译不通过，因此倒不如直接设置`vetur`格式化`js`为`none`，这样不会覆盖`eslint`的格式化结果，最省事；

# [`vant`]`Tab`标签页

`vant`还提供了`Tab`标签页，使用方法和之前一样，我们首先引入要使用的组件到`app.json`中：

```
"van-tab": "@vant/weapp/tab/index",
"van-tabs": "@vant/weapp/tabs/index"
```

接着，我们在`vue`文件中这样写：

```vue
<template>
  <div>
    <van-tabs
      class="custom-tab"
      v-model="activeTab"
      @change="onTabChange"
    >
      <van-tab title="首页">首页</van-tab>
      <van-tab title="提问">提问</van-tab>
      <van-tab title="建议">建议</van-tab>
      <van-tab title="分享">分享</van-tab>
      <van-tab title="讨论">讨论</van-tab>
    </van-tabs>
    <van-tabbar
      active-color="#02d199"
      inactive-color="#ccc"
      :active="active"
      @change="onChange"
    >
      <van-tabbar-item class="item">
        <van-icon
          name="home"
          size="28"
          class-prefix="iconfont"
        ></van-icon>
        <div>首页</div>
      </van-tabbar-item>
      <van-tabbar-item class="item small">
        <van-icon
          name="msg"
          size="24"
          class-prefix="iconfont"
        ></van-icon>
        <div>消息</div>
      </van-tabbar-item>
      <van-tabbar-item class="item small">
        <van-icon
          name="faved"
          size="24"
          class-prefix="iconfont"
        ></van-icon>
        <div>热门</div>
      </van-tabbar-item>
      <van-tabbar-item class="item">
        <van-icon
          name="person"
          size="28"
          class-prefix="iconfont"
        ></van-icon>
        <div>我的</div>
      </van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      active: 0,
      activeTab: 0
    }
  },
  methods: {
    onChange (event) {
    // event.detail 的值为当前选中项的索引
      this.active = event.mp.detail
    },
    onTabChange (event) {
      console.log('onTabChange -> event', event)
      this.activeTab = event.mp.detail.index
    }
  }
}
</script>

<style lang="scss" scoped>
.item {
  text-align: center;
  align-self: flex-end;
  padding-bottom: 5px;
  div {
    padding-top: 5px;
  }
}

.custom-tab {
//  --tabs-default-color: #02d199;
//  --tabs-bottom-bar-color: #02d199;
//  --tabs-active-text-color: #02d199;
//  --tabs-text-color: #666;
}
</style>

```

> 但是我发现`v1.5.2`的`vant`貌似存在一些问题，如果你不使用`v-model`而是使用`active`接收`activeTab`来显示当前所在的标签页，那么在首屏加载时会无法显示，因此还是使用`v-model`更好；

显示结果如下：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 16.51.17.png" alt="截屏2020-11-08 16.51.17" style="zoom:50%;" />

但是颜色不正确，怎么办？其实有好多方法来解决这个问题：

+ 直接找到标签页组件所对应编译解释后的`html`元素，然后为它单独设置`css`，这是最快速的方法，但是会提升代码的理解难度；
+ 使用官方给`van-tabs`提供的`color`属性设置颜色，但是太受限了，能更改的太少了；
+ 更改系统主题，`vant`可以使得我们更改单个元素的主题，这是非常棒的解决方案，不仅不用了解编译后的情况，同时还不需要受到严格的组件属性限制；

`vant`的主题是用`less`编写的，你可以查看`https://github.com/youzan/vant-weapp/blob/dev/packages/common/style/var.less`来查看所有的主题配置；

那么怎么修改呢？也有两种方法：

+ 下载整个`vant`源代码，修改`less`文件后重新编译打包；
+ `vant`为我们提供了另外一种更加方便的方法，我们可以直接为`van-tabs`修改`CSS`属性即可；

我们使用第二种方法，这是最简便的方法；

首先，我们要查看主题配置文件，找到`tabs`所对应的主题配置的选项，有如下的选项：

```
// Tabs
@tabs-default-color: @red;
@tabs-line-height: 44px;
@tabs-card-height: 30px;
@tabs-nav-background-color: @white;
@tabs-bottom-bar-height: 3px;
@tabs-bottom-bar-color: @tabs-default-color;
```

然后我们为要修改指定`tabs`的样式，我们这样写：

```
//van-tabs组件名
.custom-tab {
  --tabs-default-color: #02d199;
  --tabs-bottom-bar-color: #02d199;
  --tabs-active-text-color: #02d199;
  --tabs-text-color: #666;
}
```

我们发现，经过比较，我们只要添加要修改的主题属性并将`@`修改为`--`即可，这样我们就完成了该组件的主题的配置；

原理其实非常好理解，就是有一个额外的监听修饰该组件的类上是否带上这些属性的类，如果带上了这个属性，就继承该属性的值并修改真正的控制对应样式的属性即可，这种行为只要通过`css`即可实现，非常简单；

实现的代码和之前类似，只要替换`.custom-tab`类即可；

效果如下：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 17.25.17.png" alt="截屏2020-11-08 17.25.17" style="zoom:50%;" />

# [`vant`、微信小程序]顶部搜索框

一般在小程序中，我们的搜索框都是在和顶部右上角`关闭小程序`按钮行对齐；这一方面是节省本来就不多的屏幕空间，另一方面是因为搜索这个功能在小程序端其实并不是使用频次最高的功能，因此可以放在和`关闭小程序`按钮齐平的位置，因为该位置虽然显眼但是不常用，何乐而不为呢？

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 22.37.32.png" alt="截屏2020-11-08 22.37.32" style="zoom:50%;" />

为了和`小程序关闭`按钮齐平，我们首先必须先禁用掉原生小程序的顶部栏（保留`右上角关闭`按钮），操作方式很简单；

首先我们在`pages/index/`下新建`main.json`，该文件保存所有对该`index.vue`页面的配置；

我们可以添加如下配置禁用原声小程序的顶部栏：

```
{
  //"navigationBarTitleText":"首页", //设置标题
  "navigationStyle":"custom" //设置导航栏为自定义
}
```

这样，我们的导航栏就被隐藏掉了：

接下来，我们使用`vant`提供的`search`组件，将其添加到我们页面的顶部；

```
//app.json
  "van-search": "@vant/weapp/search/index"
  //index.vue
  <div
      class="search"
      :style="{'padding-top':barHeight+'px'}"
    >
      <van-search
        :value="value"
        shape="round"
        placeholder="请输入搜索关键词"
      />
    </div>
```

下图就是添加了`search`组件的效果：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 22.46.43.png" alt="截屏2020-11-08 22.46.43" style="zoom:50%;" />

显然，因为我们的导航栏被我们隐藏掉了，我们的微信小程序的顶部甚至延伸到了顶部的安全区----状态栏部分的顶部，近几年飞快发展的异型屏，导致不同的机型有不同的状态栏高度，比如`iphone7plus`的状态栏高度为`20px`，而`iphone xs`的状态栏高度达到了`44px`，因此，如果我们希望我们的微信小程序在不同的手机上体验一致，那么搜索框距离顶部的距离就必须是动态计算的；

那么如何动态计算呢？我们首先需要获得状态栏的高度，这样我们才能动态计算；

好在微信设计了一个获得系统信息的`api`，可以获得状态栏的高度；

```
getNavBarHeight () {
      let statusBarHeight = 0
      //获取系统信息，包括statusBar的高度
      wx.getSystemInfo({
        success: (result) => {
          console.log('result = ', result)
          statusBarHeight = result.statusBarHeight //获取状态栏的高度
          ...
        }
      })
    }
```

我们获取之后理当可以直接使用了，但是还有一个问题，`iOS`和`android`的`关闭小程序`按钮距离状态栏的高度又是不一致的，但是我们又需要我们的搜索框和`关闭小程序`的按钮保持齐平；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-08 23.45.01.png" alt="截屏2020-11-08 23.45.01" style="zoom:50%;" />

一般来讲，`iOS`的`关闭小程序`按钮距离状态栏为`6px`的距离，`android`距离顶部状态栏为`8px`；

因此，我们需要将状态栏高度和不同平台`关闭小程序`按钮距离状态栏的距离纳入搜索框的`padding-top`的计算，这样我们的搜索框才能和`关闭按钮`齐平，具体修改如下：

```
 getNavBarHeight () {
      let statusBarHeight = 0
      wx.getSystemInfo({
        success: (result) => {
          console.log('result = ', result)
          statusBarHeight = result.statusBarHeight
          let isIOS = result.system.indexOf('iOS') > -1 //result.system可以获取当前使用的系统平台，比如iOS 12.14.01
          if (isIOS) {
            this.barHeight= statusBarHeight + 5 //加6-1是因为搜索框比关闭按钮尺寸大1*2，为了保持齐平，离状态栏的距离要比关闭按钮离状态栏的距离-1
          } else {
            this.barHeight = statusBarHeight + 7
          }
        }
      })
```

关于为什么取`5`和`7`这两个值，以`iOS`为例，是因为我们的搜索框为`34px`，关闭按钮为`32px`，我们希望搜索框和关闭按钮齐平，如果`关闭按钮`距离状态栏是`6px`，那么搜索框距离状态栏的距离就是`6-(34-32)/2=5`，`android`类似；

其他部分容易理解，不再赘述，详细代码如下：

```
<template>
  <div>
    <div class="search"
      :style="{'padding-top':barHeight+'px'}">
      <van-search
        :value="value"
        shape="round"
        placeholder="请输入搜索关键词"
      />
    </div>
    ...
  </div>
</template>

<script>
...
export default {
  name: 'app',
  // props: ['item'],
  data () {
    return {
      value: '',
      active: 0,
      activeTab: 0,
      barHeight: 0,
      ...
    }
  },
  onLoad () {
    this.getNavBarHeight()
  },
  ...
  methods: {
    ...
    getNavBarHeight () {
      let statusBarHeight = 0
      wx.getSystemInfo({
        success: (result) => {
          console.log('result = ', result)
          statusBarHeight = result.statusBarHeight
          let isIOS = result.system.indexOf('iOS') > -1
          if (isIOS) {
            this.barHeight = statusBarHeight + 5
          } else {
            this.barHeight = statusBarHeight + 7
          }
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
...

.search {
  padding-top: 25px;
  width: 70%;
  --search-padding: 0px; //必须设置主题中的search-padding为0px，否则`van-search`会带上padding-top，因为我们修改的是外层的`div`的padding-top，不是van-search组件的
  padding-left: 8px;
  padding-bottom: 8px;
  @media (max-width: 320px) {
    width: 60%;
  }
}
...
</style>
```

# [`mpvue`、微信小程序]`API`接口`Promise`化、`async await `支持

又是一个老生常谈的话题，我们一般希望微信的接口比如`wx.getSystemInfo()`是`Promise`的，这样我们就不需要使用下面的写法了：

```
wx.getSystemInfo({
   success:(res)=>{
      console.log('res:'.res)
   }
})
```

如果我们能这样写呢？`data`就是`success`钩子返回的`res`；

```
const data = await wx.getSystemInfo()
```

虽然，`mpvue`支持`async await`的语法，但是微信小程序默认并不支持，但是`mpvue`是使用`webpack`进行打包的，因此得益于`webpack`，它会使用`babel-loader`将`async await`转化为异步的方法（类似`promise`）；

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-09 21.47.24.png" alt="截屏2020-11-09 21.47.24" style="zoom:100%;" />

如果你想要使用在原生微信小程序中使用`async await`语法，那么需要打开`增强编译`功能，该功能增强了`es6转es5`的能力，该功能在`详情`里，下图是`增强编译`增强的功能列表：

<img src="/Users/sulingjie/Library/Application Support/typora-user-images/截屏2020-11-09 22.36.40.png" alt="截屏2020-11-09 22.36.40" style="zoom:50%;" />

并不是，`wx.getSystemInfo`根本不是`Promise`的，因此`wx.gteSystemInfo().then()`根本就是不存在的；

那么我们可以封装一下`wx.getSystemInfo()`方法让它支持`Promise`的语法：

```
getSys(){
    return new Promise((resolve.reject)=>{
        wx.getSystemInfo({
            success:res=>{
                resolve(res)
            }.
            fail:err=>{
               reject(err)
            }
        })
    })
}
...
getSys().then(res=>{
   console.log('res:',res)
}).catch(err=>{
   console.log('err:',err)
})
```

好麻烦啊，每一个微信的`api`都要经过一次封装；

于是，微信官方提供了一个`npm`库来将`wx`所有的`api`转化为`promise`的版本：

```
npm install --save miniprogram-api-promise
```

别忘了`npm构建`，否则这个库我们还是用不了；

接下来，为了方便，我们创建一个单独的文件``utils/wxp.js`，内容：

```
import { promisifyAll } from 'miniprogram-api-promise'

const wxp = {}

promisifyAll(wx,wxp)

export default wxp
```

导出来的`wxp`就是被`promise`化后的`wx`对象，现在，我们可以直接这样使用`promise`化后的`wx`接口了:

```
import wxp from '@/utils/wxp'

...

wxp.getSystemInfo().then(console.log) //promise语法
wxp.showModal().then(wxp.openSetting())

// compatible usage
wxp.getSystemInfo({success(res) {console.log(res)}}) //经过promise化的`wxp`接口保留了`wx`的非promise语法
```

如果你不希望所有接口都`promise`化，你也可以只对单个`wx api promise`化，但是这种方法不太推荐；

```
import { promisify } from 'miniprogram-api-promise';

// promisify single api
promisify(wx.getSystemInfo)().then(console.log)
```

# [``]

























