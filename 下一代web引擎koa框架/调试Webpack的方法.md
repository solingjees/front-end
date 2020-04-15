# 使用console.log()直接打印webpack配置到控制台

最简单的方式调试，直接在web.config.js文件中使用，如：

~~~js
const webpackconfig = {
   ...
}

+ console.log(webpackconfig)
~~~

# 使用nodeJs内置的监视器和chrome调试台进行调试

## 第一步 配置webpack.config.js

在webpack.config.js中的webpackconfig对象（见console.log()样例提供的样板）前设置debugger标识符以在跑程序的时候在这个位置停止；

~~~
debugger
const webpackconfig = {
   ...
}
~~~



## 第一步 输入命令启动监视服务

使用如下命令启动node的监视服务，这个服务需要传递一个命令或者文件作为监视对象，我们要监视webpack（--inline热加载模式）的执行，因此我们要将webpack的打包命令作为我们的监视对象，这样当这个命令在执行时我们可以监视整个执行的过程：

~~~js
npx node --inspect-brk ./node_modules/.bin/webpack --inline --progress
~~~

在这段代码中，我们监视待执行代码（./node_module/.bin/webpack --inline --progress）的执行，[`webpack.HotModuleReplacementPlugin`](https://webpack.js.org/plugins/hot-module-replacement-plugin/)插件用于启动HMR，且如果webpack或webpack-dev-server带有--inline或--hot会直接启动HMR而不用将插件配置到webpack.config.js中；

注意，windows和linux不同，执行上该命令会出错，会在webpack处出现js解析错误，因此需要使用下面这个命令：

~~~windows
npx node --inspect-brk ./node_modules/webpack/bin/webpack.js --inline --progress
~~~

这两个命令中的webpack参数部分，前者是shell命令段，后者是js文件，可能是windows以为这是js文件但是不能转化为shell识别所以出错；

## 第二步 打开chrome连接监视服务

在浏览器中输入

~~~
chrome://inspect/#device
~~~

即可进入chrome的调试服务入口，会自动搜索到已开启的监视服务；

首先会打开我们的监视对象的执行入口，点击运行即可跑起我们要监视执行的文件或任务；

如果使用了console.log()在webpack.config.js中进行打印，那么就会在控制台上打印出来；

## 第三步 开始调试

可以在代码中添加`debugger`作为断点，chrome调试台点击运行后，就会在debugger的位置上停止；可以使用step over进行单步调试等或者添加断点，推荐在下图这个位置设置断点：

~~~
debugger
const webpackconfig = {
   ...
}
-> 断点位置
~~~

好处是这个webpackconfig的内部已经被执行了一遍，所有的配置会被chrome调试台直接显示出来，或者可以自行在console输出webpackconfig检视

## 第四步 调试糖(选配)

将命令添加到package.json的script标签下，方便我们用npm命令开启监视器；

~~~js
{
    "scripts": {
    "webpack:debug": "node --inspect-brk ./node_modules/webpack/bin/webpack.js --inline --progress"
  },
}
~~~

~~~js
npm run webpack:debug
~~~

# 使用VScode调试功能进行调试（不仅仅调试webpack）

## 第一步 添加NodeJs环境下的launch.js

新建或修改launch.js文件，这个文件会在.vscode文件下，是我们的调试配置文件（如果我们是调试koa这种建立在nodeJs上的对象，我们就要设置NodeJs环境）；

## 第二步 添加配置

添加配置其实就是将要调试的命令拆分成属性进行配置，并且增加一些vsCode的调试功能；

以添加nodemon配置为例，我们添加一个”nodemon安装程序“，这种建立新配置的方式其实就是配置运行程序的命令和VScode的调试项，我们需要看情况修改program（启动目标，即文件入口）和（runtimeExecutable）运行时执行的命令，可以通过修改runtiemArgs来传递命令项到`runtimeExecutable`中，如下：

~~~js
 "configurations": [{
        "type": "node",
        "request": "launch",
        "name": "nodemon",
        "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/nodemon",
        "program": "${workspaceFolder}/src/index.js",
        "restart": true,
        "console": "integratedTerminal",
        "internalConsoleOptions": "neverOpen",
        "skipFiles": [
            "<node_internals>/**"
        ],
        "runtimeArgs": ["--exec", "./node_modules/.bin/babel-node"] //nodemon的执行参数
    }]
~~~

等价于

~~~js
npx nodemon --exec babel-node /src/index.js
~~~

## 第三步 启动调试

在调试面板选中我们的配置对象（如nodemon），点击`开始调试`，此时便处于`调试状态`；

现在我们可以在vsCode中添加断点，然后访问页面到达断点，运用vsCode一系列调试功能来进行调试；

可以在`监视`面板查看变量进行调试；

TIPS：如果我们在路由下的业务逻辑中添加一个断点，当我们按下`继续`时，如果没有访问这个路由，那么是不会到达断点的，会一直正常执行；