# `MongoDB`流行度

<img src="MongoDB.assets/image-20200426193755600.png" alt="image-20200426193755600" style="zoom:50%;" />

# `MongoDB`的优势

+ `JSON`文档格式；

  + 最自然的、高效的数据描述形式；
  + 支持数组和嵌套对象；
  + 支持`Schema`的扩展和动态化，如静态方法，生命周期钩子；

+ 强大的查询语言；

  + 强大的描述性的查询语言；
  + 支持聚合查询和模型查询：地理位置查询、图表查询、文本查询；

  + 语句就是`JSON`格式，容易组合和阅读；

+ 包含关系型数据库的优点；

  + 完整的`ACID`事务特性；
  + 支持联合查询；
  + 使用引用和嵌套来表示关系；

+ 支持云服务、备份服务、统计服务；

+ 商业的、智能的连接件（`BI Connector`），将`MongoDB`连接到各种外围数据服务，如`MySQL`数据库通信、对不同数据库数据整合进行统计分析的服务；

# `MongoDB Docker`安装

我们使用`docker-compose`进行安装，`docker-compose.yml`文件如下：

```yaml
version: '3.1'

services:

  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root //超级管理员账户
      MONGO_INITDB_ROOT_PASSWORD: example //超级管理员密码
    ports:
      - 27017:27017
    volumes:
      - /home/mongotest:/data/db
```

然后使用步骤来完成`mongodb`的安装：

```
cd /home
mkdir mongotest
chmod -R 777 mongotest
cd mongotest
vi docker-compose.yml //输入完后保存退出
docker-compose up -d
```

最后使用以下命令添加防火墙规则：

```
firewall-cmd --add-port=27017/tcp --permanent
firewall-cmd --reload
```

# `MongoDB`基本使用

`MongoDB`内建了一些角色供我们选择，常用的有：

+ `readWrite`;
+ `dbAdmin`：数据库管理员；
+ `dbOwner`：`readWrite+dbAdmin+userAdmin`；

然后我们需要进入`MongoDB`的`cli`环境来操作我们的数据库，在`docker`部署中的`MongoDB`，我们需要使用以下命令进入`docker`的`cli`环境：

```
docker exec -it mongotest_mongo_1 mongo
```

## `MongoDB`数据库进入、切换角色、显示数据库列表

当我们进入了`cli`环境后，然后我们可以使用以下命令进入一个数据库：

```
use admin //这会新建/进入一个数据库
```

然后使用以下代码认证用户：

```
db.auth('root','example')
```

该命令认证了以`example`为密码的`root`用户，如果返回`1`说明认证成功；

> 在一次`cli`环境操作中，只能使用一次`db.auth()`函数进行认证一次，否则接下来的一些诸如插入数据等会报错；

由于我们的`root`用户是`dbOwner`角色，因此可以使用以下命令来查看数据库列表：

```
show dbs
```

<img src="MongoDB.assets/image-20200426222829399.png" alt="image-20200426222829399" style="zoom: 33%;" />

> 如果您没有登陆，那么是看不数据库列表等信息的；
>
> 如果您没有进入数据库，那么是无法登陆的；

## `MongoDB`数据库的增删改查

我们新建并进入一个数据库：

```
use testdb
```

我们可以使用以下命令来新建`user`：

```
db.createUser({user:'test',pwd:'123456',roles:[{role:'dbOwner',db:'testdb'}]})
```

这一步新建了一个`test`用户，这个用户密码是`123456`，在`testdb`数据库中的角色为`dbOwner`；

同样的，我们可以使用以下命令来修改`user`的密码：

```
db.changeUserPassword("test", "654321")
```

然后使用以下命令切换到该`user`：

```
db.auth('test','123456')
```

接着我们可以使用以下命令来为某一个`collection`插入一条数据：

```
db.users.insertOne({name:'imooc',age:30,email:'1600346867@qq.com'})
```

如果插入成功会返回以下结果：

<img src="MongoDB.assets/image-20200426225739360.png" alt="image-20200426225739360" style="zoom:50%;" />

这一步给`users collection`插入了一条`name`为`imooc`、`age`为`30`、`email`为`1600346867@qq.com`的数据；大家到这一步应该发现我们不需要和关系型数据库一样先新建表、表结构再进行插入，而是简简单单的书写数据进行插入就行了，这就是`NoSQL`的一个非常重要的特点；

> 在进行这步操作前，要保证当前`cli`环境下只进行认证了一次；

然后我们可以使用以下命令来显示一下所有的`collection`，来验证是否有`users`这张表的创建；

```
show collections
```

我们会得到一个`collections`列表：

<img src="MongoDB.assets/image-20200426230049900.png" alt="image-20200426230049900" style="zoom: 33%;" />

进一步地，我们可以插入一条新的数据，但是我们可以不一定使用和第一条数据相同地结构：

```
 db.users.insertOne({name:'imooc1',age:'30'}) //我们不设置email字段
```

然后我们可以使用以下命令来查询数据：

```
db.users.find({}) //查询所有匹配{}的数据，即打印所有该collection下的数据
```

<img src="MongoDB.assets/image-20200426230828631.png" alt="image-20200426230828631" style="zoom:50%;" />

我们同样打印出来了两条数据，而且还结构不统一，因此我们验证了`MongoDB`拥有高度的结构自由，可以自由扩展字段；

然后我们可以去更新某一条数据：

```
db.users.updateOne({name:'imooc1'},{$set:{email:'test1@imooc.com'}})
```

<img src="MongoDB.assets/image-20200426231531378.png" alt="image-20200426231531378" style="zoom:50%;" />

由于我们的更新操作不一定就是添加字段，因此需要使用`$set`来指明是修改或添加操作，在`NoSQL`的数据库中，往往`set`等价于`add or modify`；

我们更新了符合`{name:'imooc1}`的数据，给这些符合的数据添加了`email:'test@imooc.com'`这个新的`key-value`对；

最后是`delete`操作；

```
db.users.deleteOne({name:'imooc1'})
```

![image-20200426231850134](MongoDB.assets/image-20200426231850134.png)

该操作删除了所有`name`为`imooc1`的数据；

更多`CRUD`操作参见https://docs.mongodb.com/manual/crud/；

# `Robo 3T GUI`工具的使用

`Robo 3T`是极为好用的`MongoDB`的`GUI`工具，直接在官网下载对应的版本即可；

首次打开安装后，会显示以下界面，我们选择`create`来创建一个新连接；

<img src="MongoDB.assets/image-20200426234434591.png" alt="image-20200426234434591" style="zoom:50%;" />

然后我们设置连接的`name`和`address`：

<img src="MongoDB.assets/image-20200426234712638.png" alt="image-20200426234712638" style="zoom:50%;" />

接着配置连接的数据库、用户名、密码，配置完毕后我们可以点击左下角的`Test`测试我们的连接；

<img src="MongoDB.assets/image-20200426234750297.png" alt="image-20200426234750297" style="zoom:50%;" />

当我们`Test`完毕后就点击`Save`保存，这样我们成功创建了一条可用的连接，然后选择该连接进行`connect`：

<img src="MongoDB.assets/image-20200426234946043.png" alt="image-20200426234946043" style="zoom:50%;" />

接着我们的主面板就会显示我们的数据库下的所有信息了:

<img src="MongoDB.assets/image-20200426235045229.png" alt="image-20200426235045229" style="zoom:50%;" />

增删改查操作在`GUI`下格外简单；

在`Collections`下的任意`collection`上右击选择`Insert Document`，就会打开一个编辑器界面，我们以`JSON`格式输入要保存的数据然后`save`即可；

<img src="MongoDB.assets/image-20200426235649154.png" alt="image-20200426235649154" style="zoom:33%;" />

查询也很简单，我们在页面顶端编辑区输入`db.xxx.find({...})`查询语句然后`Ctrl+Enter`即可查询；

<img src="MongoDB.assets/image-20200426235753906.png" alt="image-20200426235753906" style="zoom:50%;" />

自然地，其他的操作语句也可以在顶端编辑区执行，比如`deleteOne`；

<img src="MongoDB.assets/image-20200427000214129.png" alt="image-20200427000214129" style="zoom:50%;" />

删除数据只需要在数据条上右击选择`Delete Document`即可；

`modify`操作亦是如此，只不过会打开数据编辑页面进行更改；

<img src="MongoDB.assets/image-20200426235847025.png" alt="image-20200426235847025" style="zoom:50%;" />

# `MongoDB`的备份与恢复

`MongoDB`的备份与恢复非常简单；

我们只需要输入以下命令就可以备份了：

```
docker exec -it <container_name> mongodump  \ 
-h localhost  \ //指定需要备份的MongoDB地址
-u test \ //指定用户名
-p 123456 \ //指定用户密码
-d testdb \ //指定dababase，默认为全部数据库都要备份
-o /tmp/test //指定备份输出目录
```

> 要注意用户在数据库内的权限，否则可能出现认证问题而无法备份；

但是这样备份会备份到容器内部的`/tmp/test`目录，我们使用以下命令将其导出到宿主机的`/tmp/test`目录：

```
docker cp <container_name>:/tmp/test /tmp/test
```

然后我们可以到宿主机的`/tmp/test`下查看我们的备份：

<img src="MongoDB.assets/image-20200427005926382.png" alt="image-20200427005926382" style="zoom:50%;" />

而恢复一般都是超级管理员用户使用，我们使用以下命令来将我们的备份恢复：

```
docker exec -it <container_name> mongorestore 
-h localhost \ //指定愮恢复的MongoDB地址
-u root \ //指定执行恢复操作的数据库用户名
-p example \ //指定执行恢复操作的数据库用户密码
--dir /tmp/test //指定需要恢复的数据库容器内的备份
```

如果我们希望使用宿主机上的备份进行恢复，我们可以将备份存放到容器内挂载到宿主机上的目录，或者手动使用`docker cp`命令将宿主机上的备份`copy`到容器内部进行使用；

# `Oracle/MySQL、MongoDB、Mongoose`的概念类比

<img src="MongoDB.assets/image-20200427120819149.png" alt="image-20200427120819149" style="zoom:50%;" />

# `Mongoose`的安装和基本连接

`Mongoose`是`Node.js`下的一个操作`MongoDB`的库；

我们使用以下命令进行安装：

```
npm install -S mongoose
```

基础使用如下：

```js
const mongoose = require('mongoose') //引用mongoose库

mongoose.connect('mongodb://test:123456@www.solingjees.site:27017/testdb', {
    useNewUrlParser: true, //使用新的Url解析器
    useUnifiedTopology:true //使用新的dicovery and monitoring engine
}) 
//连接数据库
//连接的url结构如下：mongodb://<username>:<password>@<serverurl>:<serverport>/<database>

const User = mongoose.model('users', { name: String, age: Number, email: String })
//新建/拉取数据库中的collection/Model，Schema是结构，model是模型
const imooc = new User({
    name: 'imooc-test',
    age: 30,
    email:'imooc@imooc.com'
})
//新建一个collection instance

imooc.save().then(()=>console.log('save ok')) //存入数据库
```

如果我们的控制台出现以下的`warning`：

<img src="MongoDB.assets/image-20200427123925639.png" alt="image-20200427123925639" style="zoom:50%;" />

对应第一条`warning`，我们需要在我们的`mongoose.connect`方法中添加下面的`opion`即可：

```
useUnfiedTopology:true
```

对应第二条`warning`，我们需要安装`saslprep`库：

```
npm install -S saslprep
```

这样就不会有这样两条`warning`了；

# `Mongoose`和`Koa`的集成和深入使用

首先我们需要编写配置`Mongoose`连接的文件，我们在`src/config`目录下新建`index.js`文件，该文件主要是`Mongoose`的配置参数，如`URL`：

```js
//核心配置文件
const DB_URL = 'mongodb://test:123456@www.solingjees.site:27017/testdb'

export default {
  DB_URL,
}
```

然后编写`src/config/DBHelper.js`文件，该文件提供`MongoDB`的连接，我们首先使用`mongoose.connect`去连接我们的数据库，然后可以使用`mongoose.connection`对象获取到我们的连接信息，进一步可以调用`mongoose.connection.on()`进行事件监听，如`connected`、`error`、`disconnected`；

```js
import mongoose from 'mongoose'
import config from './index'

//创建连接
mongoose.connect(config.DB_URL, {
    useNewUrlParser: true,//使用新的url解析器
    useUnifiedTopology:true //使用新的discovery and monitor engine 
})

//监听连接成功
mongoose.connection.on('connected', () => {
     console.log(`Mongoose connection open to ${config.DB_URL}`)
})

//监听连接失败
mongoose.connection.on('error', (err) => {
    console.log(`Mongoose connection error:${err}`)
})

//监听连接断开
mongoose.connection.on('disconnected', () => {
    console.log('Mongoose connection disconnected')
})

export default mongoose
```

我们把一些连接之类的基础配置放在`src/config`文件夹下有助于我们管理我们的连接，在我们的业务逻辑中，我们只需要调用`src/config/DBHelper.js`暴露出来的`mongoose`对象操作数据库而不需要重新考虑数据库的连接问题；

我们可以在`/src/model`文件夹下新建一个`test.js`文件来测试连接/远程创建一个`collection`，首先我们获取`mongoose.Schema`类，`Schema`类用于生成一个`Schema`，这个类的数据仍然是`Object`，只是添加了一些外围方法，然后我们使用`mongoose.model(<collection_name>,<schema>)`来远程新建/连接一个`model`；

> `Schema`代表数据表

```js
import mongoose from '../config/DBHelper'
var Scheme = mongoose.Schema
var TestSchema = new Scheme({
  name: { type: String },
  age: { type: Number },
  email: { type: String },
})
export default mongoose.model('users', TestSchema)
```

接着我们可以编写`/src/model/demo.js`文件来进行`Mongoose`的增删改查，我们可以使用`./test/js`暴露出来的`model`来新建一个`instance`，就是使用`new xxx()`的方法实例化即可；

增加一条数据只要使用`instance.save()`即可增加；

查询数据不需要`instance`，直接使用`Class.find()`即可；

更新数据需要使用`Class.updateOne(<attach options>,<modify>)`来修改我们的数据；

删除数据使用`Class.deleteOne(<attach options>)`，来删除数据；

> 要注意的是，这些对数据库的操作都是异步的，也就是如果要接收到返回的`data`，我们需要使用`Promise.then()`或者`Async/Await`来取得数据；

```js
import User from './test'

const user = {
  name: 'solingjees',
  age: 22,
  email: '1600346867@qq.com',
}

const insertMethod = async () => {
  const data = new User(user)
  const result = await data.save()
  console.log(result)
}

const findMethod = async () => {
  const result = await User.find()
  console.log(result)
}

const updateMethod = async () => {
  const result = await User.updateOne(
    { name: 'solingjees' },
    { email: 'ss@s.com' }
  )
  console.log(result)
}

const deleteMethod = async () => {
  const result = await User.deleteOne({ name: 'solingjees' })
  console.log(result)
}
```

> 在`Koa`中，如果使用了`es6`语法，那么测试时请使用`babel-node`命令来测试；





