# 使用npm-check-updates更新依赖

## 第一步 安装npm-check-updates

这个npm包（只能）用于更新我们的package.json依赖项的版本显示，若不是大版本更新，API没有经过大型的抛弃、增加、重构和修改，建议使用；

~~~js
npm install -g npm-check-updates
~~~

使用方法:

~~~
ncu //查看所有的依赖是否有版本更新
ncu --help //查看帮助
ncu -u //更新我们的package.json依赖的版本
~~~

## 第二步 更新依赖的版本

首先我们先删除node_modules（可以不删，但是鉴于稳定性，最好删掉）

~~~
mac/linux：rm -rf node_modules
windows：直接右键删除

~~~

然后使用

~~~
npm install
~~~

重新安装所有依赖

或

~~~
npm install rimraf -D  //安装rimraf插件：用于删除文件
rimraf node_modules 
~~~



# 使用koa-compose集成中间件

## 第一步 安装koa-compose

这个命令只有集成koa的中间件的功能，使用：

~~~js
npm install koa-compose --save
~~~

安装，引入这个中间件：

~~~js
const compose = require('koa-compose') //commonJs
import compose from 'koa-compose'//es6
~~~

## 第二步 使用中间件

~~~js
import compose from 'koa-compose'
~~~

使用compose([a,b,c,...])来集成所有的中间件，以简化代码

~~~js
//本文件使用babel-node启动
import Koa from 'koa'
const app = new Koa()
// const path = require('path')
import path from 'path'
import helmet from 'koa-helmet'
import statics from 'koa-static'
import koaBody from 'koa-body'
import jsonutil from 'koa-json' //美化json
import cors from '@koa/cors'
import compose from 'koa-compose'

const middleware = compose([
  koaBody(),//预处理body数据
  statics(path.join(__dirname, '../public')),//映射静态文件，如图片
  cors(),//处理跨域问题
  jsonutil({ pretty: false, param: 'pretty' }),//美化json输出
  helmet()//处理headers安全问题
])

app.use(middleware)//使用中间件
~~~

# 使用多文件式webpack来进行更高粒度的配置

## 第一步 建立多个webpack 的配置文件

首先建立3（一般为3个）个webpack的配置文件，分别为

~~~js
webpack.config.base.js //基本配置
webpack.config.dev.js //开发环境下的配置
webpack.config.prod.js //生产环境下的配置
~~~

## 第二步 在webpack.config.base.js中配置全局常量（选配）

在打包编译过程中，我们有些时候需要按照当前的开发模式来执行webpack，如，server会按照`process.env.NODE_ENV`进行配置端口，但是webpack编译打包时（处于编译时环境）不能直接使用node.js的`process.env`，因此我们要在webpackconfig中使用`webpack.DefinePlugin({})`来将process.env.NODE_ENV手动传入webpack编译时环境；

~~~js
const webpackconfig = {
   ...
   plugins: [
        new webpack.DefinePlugin({
            'process.env': {
                NODE_ENV: (process.env.NODE_ENV === 'production' || process.env.NODE_ENV === 'prod') ? '"production"' : '"developement"'
            }//根据当前运行时环境中的NODE_ENV进行编译时环境配置
        })
    ],    
}

~~~

tips：`webpack.DefinePlugin`会直接进行文本替换，如，`process.env`会直接被替换为‘production’或‘development'，所以必须有两个引号，如`webpack`打包时出现process.env.NODE_ENV时，会这样替换

~~~js
process.env.NODE_ENV 先匹配 process.env
再匹配NODE_ENV -> "production"，
若没有引号     ->  production   // 报错undefined
~~~

## 第三步 使用webpack-merge在webpack.config.dev/prod.js中合并web.config.base.js的配置

`webpack-merge`能合并我们的webpack配置，使用

~~~
npm install webpack-merge -D
~~~

安装到开发依赖中，使用时将多个webpackconfig（Object）传入即可；

~~~js
//webpack.config.dev.js
const webpackMerge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.config.base')
const webpackconfig = webpackMerge(baseWebpackConfig, {
  //开发模式下的webpack配置
})

module.exports = webpackconfig
~~~

## 第四步 配置webpack.config.dev/prod.js的配置

将mode的配置和更多独属于固定阶段打包的配置从`webpack.config.base.js`中分离到`webpack.config.dev/prod.js`中；

~~~js
//webpack.config.dev.js
const webpackMerge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.config.base')
const webpackconfig = webpackMerge(baseWebpackConfig, {
  devtool: 'eval-source-map',//配置调试方式
  mode: 'development',
  stats: {//配置统计信息
    children: false //不显示依赖的调试信息
  }
})
module.exports = webpackconfig
~~~

在配置webpack.config.prod.js时，我们要安装`TerserWebpackPlugin`来压缩我们的用户代码，用

~~~js
npm install terser-webpack-plugin --save-dev
~~~

来安装；

~~~js
//webpack.config.prod.js
const webpackMerge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.config.base')
const TerserWebpackPlugin = require('terser-webpack-plugin')
const webpackConfig = webpackMerge(baseWebpackConfig, {
  mode: 'production',
  stats: { children: false, warnings: false }, //打包时不要依赖信息，不要警告信息
  optimization: { // 配置优化参数
    minimizer: [ //配置简化器
      new TerserWebpackPlugin({
        terserOptions: {
          //压缩时不要警告
          warnings: false,
          compress: {
            //上线后启用所有的console函数
            drop_console: false,
            dead_code: true, // 去除访问不到的代码
            drop_debugger: true //去除debugger语句
          },
          output: {
            comments: false, //删除注释
            beautify: false //输出是否美化，即语句全部是在单行上的
          },
          mangle: true //压碎，即混淆
        },
        parallel: true, // 是否使用多线程来提高打包速度，默认为os.cpus().length - 1
        sourceMap: true
      })
    ]
  }
})
module.exports = webpackConfig
~~~

# 提取代码

我们希望将代码中被公共使用的代码部分分离出来，这样能进一步压缩代码，因此我们使用`optimization.splitChunks`进行配置

~~~js
const webpackconfig = {
    optimization:{
       splitChunks: {
           //配置缓存组群
      cacheGroups: {
        commons: {//表示一个缓存组，符合以下规则的都会被打包进这个chunks内
          name: 'commons',//split chunk的名字
          chunks: 'initial',//表示只从入口模块进行拆分，即这个chunk只能来源于入口文件
          minChunks: 3,//拆分进来的chunk最少被引用3次
          enforce: true//忽略（或不继承）splitChunks.minSize, splitChunks.minChunks, splitChunks.maxAsyncRequests and splitChunks.maxInitialRequests，就算模块已经被其他cacheGroup匹配到，也会合并进来 
        }
      }
    }        
  }
}
~~~



# 路径提取

当我们的webpack的配置文件过多，每一个文件都配置路径非常繁琐，因此可以将重要路径集中在一个文件方便配置文件调用：

~~~js
const path = require('path')

//exports的快捷方式写法 <=> modules.exports
exports.resolve = function resolve(dir) {
    return path.join(__dirname, '..', dir)
}
//将外部重要的src目录和dist目录封装一下方便调用
exports.APP_PATH = exports.resolve('src')
exports.DIST_PATH = exports.resolve('dist')
~~~

# package.json的scripts命令增强

添加`rimraf`包来删除dist目录，本质上类似linux的rimraf ，安装：

~~~js
npm run rimraf -save-dev
~~~

使用：

~~~js
rimraf <文件夹名>
~~~

~~~js
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",//default
    "start": "nodemon src/index.js",//用nodemon启动一个服务器运行代码（commmonjs）
    "start:es6": "nodemon --exec babel-node src/index.js",//用nodemon启动一个服务器运行代码（含有es6）
    "webpack:debug": "node --inspect-brk ./node_modules/webpack/bin/webpack.js --inline --progress",
        //调试webpack的配置，使用node的监视服务检视webpack的执行
    "build": "cross-env NODE_ENV=prod webpack --config config/webpack.config.prod.js",
        //设定环境变量，用webpack打包production
    "dev": "cross-env NODE_ENV=dev nodemon --exec babel-node --inspect=9230 ./src/index.js",
        //设定环境变量，用nodemon启动一个服务器运行代码并开启nodemon的监视服务（含有es6）
    "clean": "rimraf dist"
        //清除dist目录
  },
~~~







