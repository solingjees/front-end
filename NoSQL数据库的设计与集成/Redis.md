

# `Redis`的特点和应用场景

`Redis`是完全开源免费的，遵守`BSD`协议，是一个高性能的`key-value`的数据库；

`Redis`使用`Redis-Server`运行，然后用户使用`Redis-Client`去连接`Redis-Server`，因此我们只需要进入`Redis-Client`使用密码去连接`Redis-Server`即可；

`Redis`和其他的数据库相比，有以下的特点：

+ 高性能，可持久化；
+ `key-value`结构，支持多种数据类型，如`set`、`list`、`zset`、`hash`等；
+ 支持事务、数据的原子性；

`Redis`有以下应用场景：

+ 缓存（读写性能优异），提升用户对数据的访问速度，降低数据库的压力，有灵活的键淘汰策略；
+ 计数`&`消息系统，支持高并发、发布/订阅阻塞队列功能；
+ 支持`Redis`集群分布式会话`session`和分布式锁（秒杀）；

# `Redis`和`MongoDB`的对比

存储方式不一样：`key-value vs Document`；

使用方式和可靠性不一样：`MongoDB`带有`MongoDB SQL`和全套`ACID`支持；

应用场景不一样：高性能缓存 `vs` 海量数据分析；

# `Redis`的`Docker`安装

在`docker`中使用以下命令进行安装：

```yaml
docker run -itd \ 
--restart=always  \
-p 15001:6379  \
--name redis_test \ 
-v /home/redisTest:/data redis  \ 
redis-server --requirepass 123456 //启动redis-server,redis-client连接需要密码123456
```

或者等价的，使用`docker-compose.yml`进行安装：

```yaml
version: '3'
services:
  redis-test:
    image: 'redis'
    restart: always
    container_name: 'redis_test'
    ports:
      - 15001:6379
    volumes:
      - /home/redisTest:/data
    command: ["redis-server","--requirepass","123456"]
```

> `Redis`的`Windows`版本比较老旧，不建议在`Windows`上使用`redis`；

相关的配置在`GitHub`的`antirez/redis`的`redis.conf`中可以查看，其中比较重要的有：

+ `Redis`的运行端口：

  <img src="Redis.assets/image-20200429124730030.png" alt="image-20200429124730030" style="zoom:50%;" />

+ `Redis`的数据一般都会保存在内存中，默认使用`redis.conf`内的持久化规则进行数据持久化；

  如下图所示，`Redis`配置了一些持久化规则：

  + 在`900s`内修改至少`1`个键值`=>`持久化到宿主机；
  + 在`300s`内修改至少`10`个键值`=>`持久化到宿主机；
  + 在`60s`内修改至少`10000`个键值`=>`持久化到宿主机；

  如果一个规则被匹配了，就会进行持久化，然后会重新开始计算时间和键值；

  <img src="Redis.assets/image-20200429125037196.png" alt="image-20200429125037196" style="zoom:50%;" />

> 注意端口的放行；

# `Redis`的命令

`Redis`命令的参考推荐该网站http://doc.redisfans.com/

## `redis connection`

首先，我们需要进入我们的`redis`的`redis-cli`的`shell`环境：

```
docker exec -it <redis_container> redis-cli
```

或者我们使用`/bin/bash`来进入`redis container`的交互式终端，然后再执行`redis-cli`命令来连接到`redis-server`：

```
docker exec -it <redis_container> /bin/bash
> redis-cli //在交互式终端下
```

如果我们设置了`password`来来保护我们的`redis`，我们需要输入以下命令进行鉴权：

```
auth <password>
```

或者修改进入`shell`环境的命令直接在命令中进行鉴权登陆：

```
docker exec -it <redis_container> redis-cli -h 127.0.0.1 -a <password>
```

进一步地，如果我们需要修改`password`，我们可以使用如下命令进行修改：

```
config set requirepass <password>
```

若我们需要打印信息，可以使用如下命令：

```
echo "hello world"
```

如果我们需要检测`redis-server`是否运行，可以使用如下命令：

```
ping //如果正常运作，会返回pong
```

如果我们需要退出`redis-cli`，可以使用如下命令：

```
quit //退出镜像使用exit
```

## `redis database`

在`Redis`中，以`database`区分不同的数据集，`Redis`的配置文件中配置了当前`database`的数量；

<img src="Redis.assets/image-20200429150015274.png" alt="image-20200429150015274" style="zoom:50%;" />

我们可以使用以下命令来切换不同的`database`：

```
select <database_index> //database没有名字，只有索引
```

不同`database`的数据相互隔离，互不影响；

## `redis get & set`

我们使用以下命令来设置和获取`key-value`：

```
set <key> <value> //ex:set name imooc
get <key> //ex:get name
```

对于`Integer`类型的数据，我们使用以下两个命令操作`value`的自增自减`1`：

```
incr <key> //ex:incr weight  即weight + 1
decr <key> //ex:decr weight  即weight - 1
```

## `redis keys & exists`

如果我们需要查询符合特定结构的`keys`，如找出所有`331003`带头的`key`，我们可以使用如下命令：

```
keys * //所有的keys
keys 331003* //331003带头的keys
```

进一步地，如果我们需要确定某些`keys`是否存在，可以使用如下命令：

```
exists <key1> <key2> ... //ex:exists test1 test2
```

会返回所提供的`keys`列表中`keys`存在的数量；

<img src="Redis.assets/image-20200429151321978.png" alt="image-20200429151321978" style="zoom:50%;" />

## `redis del`

如果我们需要删除某些`keys`，可以使用如下命令：

```
del <key1> <key2> ... //ex:del test1
```

会返回成功删除的数量：

<img src="Redis.assets/image-20200429151607733.png" alt="image-20200429151607733" style="zoom:50%;" />

## `redis hash`

我们的数据存储可能需要多级存储，比如临时存储一个用户的多份订单信息，往往需要将这些订单信息都存在该用户`key`下，用`JSON`语言表示即为：

```
"user":{
"purchase1":...,
"purchase2":...
}
```

而之前的操作无法满足我们的要求，因此我们可以使用`hash`系列的命令来实现多级的数据存储；

这些命令如下：

```
hset <key> <field> <value> //ex:hset solingjees purchase1 iphonexsmax，即将iphonexsmax存储在solingjees这个key下的purchase1 field中，以下类同
hget <key> <field> //ex:hget solingjees purchase1
hgetall <key> ex:hget solingjees 获取该key的所有filed-value对
hmset <key> <field1> <value1> <field2> <value2> ... //ex： hmset solingjees name sulingjie  age 31
hmget <key> <field1> <field2> ... //ex: hmget solingjees name age
```

## `redis list`

我们的数据也可以存储在一个`list`中，我们可以使用以下命令头来操作`list`：

```
LPUSH //头部增加元素
LPOP //头部移除元素
RPUSH //尾部增加元素
RPOP //尾部移除元素
LPUSH //判断list key是否存在，如果不存在，不执行LPUSH
RPUSHX //判断list key是否存在，如果不存在，不执行RPUSH
```

## `redis set(collection)`

大部分的内容都比较简单，只需要注意一下：

```
SADD <set-key> <value>//往set-key中插入一个元素，如果该value已存在，不执行ADD
```

## `redis subscribe & publish`

有时候我们需要让一个`redis-cli`能够获得另一个`redis-cli`发送给他的消息，因此`channel`应运而生，`redis-cli`订阅一个或多个`channel`，然后其他的`redis-cli`往一个`channel`内发布信息，这样`redis-cli`订阅者就能接收到其他`redis-cli`的消息了：

```
//redis-cli subscribe-end:
subscribe <channel1>  <channel2> ... //订阅多个频道的消息 ex:subscribe imooc imooc1
//这条命令执行后，subscribe-end会一直在前台交互式终端处于订阅状态并接收消息
```

<img src="Redis.assets/image-20200429160630363.png" alt="image-20200429160630363" style="zoom:50%;" />

然后在另外一个`redis-cli`中去`publish`一个消息到`channel`中：

```
//redis-cli publish-end
publish <channel> <messgae-string> //发布一个消息到一个频道 ex:publish imooc "hello world"
```

<img src="Redis.assets/image-20200429160747467.png" alt="image-20200429160747467" style="zoom:50%;" />

这样`subscribe-end`就会接收到消息并打印出来；

<img src="Redis.assets/image-20200429160523994.png" alt="image-20200429160523994" style="zoom:50%;" />

> 更多关于发布者和订阅者的命令查看文档；

## `redis server`

我们可以使用`config set`命令来配置一些`redis`配置，如密码等；

使用`client list`来查看所有与`redis-server`连接的`redis-cli`：

<img src="Redis.assets/image-20200429161540250.png" alt="image-20200429161540250" style="zoom:50%;" />

在上图中有三个`redis-cli`，每一行打印的数据中`idle`代表该`redis-cli`空闲的时间，因此我们想到对一些长久没有动作的`redis-cli`，我们可以手动关闭；

我们可以使用以下命令来关闭一个`redis-cli`：

```
client kill <addr> //ex:client kill 41.216.186.89:1084
```

当我们在生产环境中，可能出现查询非常慢的操作，我们可以使用`slowlog`来记录它们，我们使用以下命令来记录`100us`的查询：

```
config set slowlog-log-slower-than 100
```

我们可以使用`slowlog get`命令来获取慢查询日志，使用`slowlog len`命令获取日志数量，使用`slowlog reset`来清空日志；

还有一个非常重要的命令`flushdb`和`flushall`，前者用于删除当前`database`内的数据，后者用于删除全部`database`的数据，后者的使用要非常小心；

但是同样地，`redis`提供了备份的功能，我们使用以下命令进行备份：

```
save //就一个字
```

这样我们就成功将当前的数据库备份了；我们可以使用`config get dir`命令查看所备份的目录；

<img src="Redis.assets/image-20200429212126852.png" alt="image-20200429212126852" style="zoom:50%;" />

比如上图，所有备份数据都保存为了`/data`目录下的`dump.rdb`文件，如果不在`/data`目录下，我们需要手动将`dump.rdb`文件移入，接着当我们`docker restart`时，`redis`就会从`/data/dump.rdb`加载恢复我们的数据；

但是在实际的生产环境中，我们一般不使用`save`命令进行备份，因为这是同步的，会阻塞来自于客户端的请求，因此我们更多地使用`bgsave`来异步地备份我们的数据，这会新建一个子进程来进行备份；备份完成后，我们可以使用`lastsave`来查看备份是否成功；

# `Redis`的`GUI`工具

我们可以使用以下几种`GUI`工具来操作`Redis`，但是我们推荐第一个：

+ `Another Redis Desktop Manager`；
+ `Medis`（收费，可自行构建）；
+ `Redis Desktop Manager`（收费）；

以`Another Redis Desktop Manager`为例，当我们打开界面后，我们点击`New Connection`创建一个新的连接，在弹出的创建页面中，我们输入`IP`地址、端口、鉴权密码、连接名可以创建一个新的连接；

<img src="Redis.assets/image-20200429213057565.png" alt="image-20200429213057565" style="zoom: 33%;" />

然后我们就可以进入操作的界面了，可以查看`Redis`的状态、内存使用情况、服务器情况、数据库情况和`Redis`全局信息，信息非常齐全，而且非常方便，基本覆盖我们需要了解的所有信息；

<img src="Redis.assets/image-20200429213835724.png" alt="image-20200429213835724" style="zoom:50%;" />

我们可以点击左上角`新增key`添加新的`key`，点选左侧列表的`key`查看内容；

对于`key-value`，我们可以直观的进行删除、修改的操作；

![image-20200429214602857](Redis.assets/image-20200429214602857.png)

对于`hash object`，我们可以也是可以点选每一个`field`右侧的`编辑`、`删除`来操作`field`：

<img src="Redis.assets/image-20200429214136444.png" alt="image-20200429214136444" style="zoom:50%;" />

`GUI`工具很简单，就不多赘述了；

# `Redis`和`Node.js`集成

首先我们安装必要的插件；

```
npm install -S redis bluebird
```

我们在`src/config/RedisConfig.js`中书写如下代码：

```js
import redis from 'redis'
import { promisifyAll } from 'bluebird' //使用bluebird的增强的promise功能将同步API转化为异步promise的形式
import config from './index'
const options = {
  host: config.REDIS.host, //来自于./index连接配置文件
  port: config.REDIS.port,
  password: config.REDIS.password,
  detect_buffers: true, //返回值以buffer形式返回，否则会以string返回，目的是要取出返回值内的数据，所以不能以string返回
  retry_strategy: function (options) {
    if (options.error && options.error.code === 'ECONNREFUSED') {
      // End reconnecting on a specific error and flush all commands with
      // a individual error
      return new Error('The server refused the connection')
    }
    if (options.total_retry_time > 1000 * 60 * 60) {
      // End reconnecting after a specific timeout and flush all commands
      // with a individual error
      return new Error('Retry time exhausted')
    }
    if (options.attempt > 10) {
      // End reconnecting with built in error
      return undefined
    }
    // reconnect after
    return Math.min(options.attempt * 100, 3000)
  },
}

// const client = redis.createClient(options)
const client = promisifyAll(redis.createClient(options)) //使用promisifyAll对一个对象下的所有方法promise化出一个对应的Async方法

//error事件监听
client.on('error', (err) => {
  console.log(`redis client error:${err}`)
})

const setValue = (key, value) => {
  if (typeof value === 'undefined' || value === null || value === '') return
  if (typeof value === 'string') {
    client.set(key, value)
  } else if (typeof value === 'object') {
    Reflect.ownKeys(value).forEach((item) => {
      client.hset(key, item, value[item], redis.print)
    })
  }
}

// const { promisify } = require('util')
// const getAsync = promisify(client.get).bind(client)
// 将get方法异步
const getValue = (key) => {
  return client.getAsync(key) //添加Async表示使用bluebird的promise
}

const getHValue = (key) => {
  // v8 promisify method use util ,must node > 8
  //   return promisify(client.hgetall).bind(client)(key)

  // bluebird async
  return client.hgetallAsync(key)
}

const delValue = (key) => {
  client.del(key, (err, res) => {
    if (res === 1) {
      console.log('delete successfully')
    } else {
      console.log(`delete redis key error:${err}`)
    }
  })
}

export { client, getValue, setValue, getHValue, delValue }
```

然后我们编写`test.js`来测试`get & set`；

```js
import { client, getValue, setValue, getHValue, delValue } from './RedisConfig'

setValue('imooc', 'imooc message from redis client')

getValue('imooc').then((res) => console.log(`getValue:${res}`))

delValue('imooc')

setValue('imoocobj', {
  name: 'solingjees',
  age: 32,
  email: '1600346867@qq.com',
})

getHValue('imoocobj').then((res) =>
  console.log(`getHValue:${JSON.stringify(res, null, 2)}`)
)

```

我们将`Redis`的连接配置暴露在`./index.js`文件下方便我们通过修改`Redis`的连接参数来连接不同的`Redis`服务器；

```js
//核心配置文件
const DB_URL = 'mongodb://test:123456@www.solingjees.site:27017/testdb'
const REDIS = {
  host: 'www.solingjees.site',
  port: 15001,
  password: '123456',
}
export default {
  DB_URL,
  REDIS,
}
```

> 这里编写的三个文件都在同一个目录下：
>
> ------ `config`
>
> ----------- `index.js` //连接配置文件
>
> ----------- `RedisConfig.js` //核心`API`：`get`，`set`，`hget`，`hset`
>
> ----------- `test.js` //测试文件，后续删除

