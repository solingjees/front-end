# lay UI的安装使用

## 安装

lay UI是适配vue.js的一个模块化UI框架，使用

~~~js
npm i layui-src(not recommend)
~~~

安装或使用CDN，网址在`https://www.layuicdn.com/?from=layui`：

~~~html
<link rel="stylesheet" type="text/css" href="https://www.layuicdn.com/layui/css/layui.css">//引入css样式文件
<script src="https://www.layuicdn.com/layui/layui.js"></script>
//引入js文件
~~~

或直接从官方下载文件，注意要把文件保存到`src`目录下，否则编译后无法访问；

## 使用

lay UI使用比较简单，样式的添加直接在`tag`的`class`内添加lay UI的`class`即可；

~~~html
 <form class="layui-form  layui-form-pane "></from>
~~~



# code . 的使用

若没有配置`VS code`的全局命令，在`VS code`的搜索`command/ctrl + shift + p`中搜索`code`，选择`install code command in path`，即可安装code的`cli`命令，然后我们就可以使用`code .`来在命令行快速打开当前的文件夹到`VS code`中；

~~~js
code .
~~~

