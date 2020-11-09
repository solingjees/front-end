# koa es化

使koa可以使用es的语法进行编写的最重要的原因便是`babel`，我们在`.babelrc`中如下配置：

~~~js
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "node": "current"//编译成node当前版本（process.versions.node）支持的版本
        }
      }
    ]
  ]
}

~~~

于是我们可以在`koa`中可以舍弃`commonjs`的写法，和前端一样使用`es`的写法：

~~~js
require => import ... from ...
添加class
modules.exports => export default
etc...
~~~

在我们调试之前，必须使用`babel-node`来转译代码;

在我们`webpack`打包时，必须使用`babel-loader`来转译代码；

# koa controller class化

在babel配置的基础上，我们可以舍弃之前的`commonjs`的分离的`function`写法，使用`class`进行集成，使得代码更加优雅：

~~~js
//所有的controller集成在DemoController中
class DemoController {
  constructor() {}
  
  async demo(ctx) {
    ctx.body = {
      msg: 'body message'
    }
  }
}

export default new DemoController()
~~~

# koa中间件压缩

当我们的代码处于生产环境，我们的process.env.NODE_ENV被我们设置为`‘production’`，这时候如果我们有性能的需求，就需要做中间件的压缩来加快我们的访问速度，这在高并发的状态下尤为重要，使用`koa-compress`来压缩中间件：

~~~js
npm install koa-comporess -S
~~~

使用：

~~~js
import compress from 'koa-compress'
const isDevNode = process.env.NODE_ENV === 'production' ? false : true
//将这个组件放在第一个中间件
//运行时如果是生产模式就开启压缩
id(!isDevNode){
    app.use(compress())
}
~~~

