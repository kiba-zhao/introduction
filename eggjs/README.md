# eggjs使用说明 #
[eggjs](https://eggjs.org/zh-cn/)专注于提供 Web 开发的核心功能和一套灵活可扩展的插件机制．它可以通过选择或定制各种插件，来达到帮助开发团队和开发人员降低开发和维护成本的目的。

[eggjs](https://eggjs.org/zh-cn/)奉行『约定优于配置』，按照一套统一的约定进行应用开发．我们基于后端架构的实际情况，利用framework和plugin定义了更为详细的约定．

## 约定项 ##
* [eggjs基本约定](#eggjs基本约定)
  * [请求路由](#请求路由)
  * [请求处理](#请求处理)
  * [业务逻辑](#业务逻辑)
  * [功能函数](#功能函数)
  * [离线任务](#离线任务)
* [http开发约定](#http开发约定)
  * [http请求日志](#http请求日志)
  * [http异常处理](#http异常处理)
  * [http补充内容](#http补充内容)
  * [参数注入](#参数注入)  
  * [swagger接口文档](#swagger接口文档)
  * [接口请求验证](#接口请求验证)
* [数据操作约定](#数据操作约定)
  * [sequelize](#sequelize)
  * [redis](#redis)
  * [rocketmq](#rocketmq)
* [调试](#调试)
* [发布](#发布)
* [快速开发](#快速开发)

## eggjs基本约定 ##
首先，我们需要遵照[eggjs](https://eggjs.org/zh-cn/)包含的[基本约定](https://eggjs.org/zh-cn/basics/structure.html)．
比如：目录结构，配置，插件开发，单元测试等．


### 请求路由 ###
在遵照[eggjs](https://eggjs.org/zh-cn/)关于路由的[使用规则](https://eggjs.org/zh-cn/basics/router.html)之后，我们定义了更进一步约定．
我们按照[restful接口](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)的方式进行设计开发，因此路由需要遵照相应的路由规则．

支持restful接口访问的router，如下:
``` javascript
module.exports = app => {
    const { router, controller } = app;
    
    // 列出所有资源
    if(controller.simple.index)
        router.get('/simples', controller.simple.index);
    // 新建一个资源
    if(controller.simple.post)
        router.post('/simples', controller.simple.post);
    // 获取一个资源内容
    if(controller.simple.get)
        router.get('/simples/:id', controller.simple.get);
    // 更新一个资源内容
    if(controller.simple.put)
        router.put('/simples/:id', controller.simple.put);
    // 更新一个资源的部分内容
    if(controller.simple.patch)
        router.patch('/simples/:id', controller.simple.patch);
    // 删除某个资源
    if(controller.simple.delete)
        router.delete('/simples/:id', controller.simple.delete);
    // 删除所有资源
    if(controller.simple.clean)
        router.delete('/simples', controller.simple.clean);
    // 获取某个资源的元信息(返回数据格式，内容大小)
    if(controller.simple.head)
        router.head('/simples/:id', controller.simple.head);
    // 获取某个资源的相关信息
    if(controller.simple.options)
        router.options('/simples/:id', controller.simple.options);
    // 获取资源的相关信息
    if(controller.simple.allow)
        router.options('/simples', controller.simple.allow);
};
```
> 这里采用的method命名与eggjs示例不一致，这样主要是为了方便找到函数相对应的http methods．

### 请求处理 ###
在[eggjs](https://eggjs.org/zh-cn/)里，使用[controller](https://eggjs.org/zh-cn/basics/controller.html)进行http请求处理．主要用于将http请求，转换处理为业务逻辑所需要的数据．并将数据提交给相应的业务进行处理．

``` markdown
1.获取用户通过 HTTP 传递过来的请求参数。
2.校验、组装参数。
3.调用 Service 进行业务处理，必要时处理转换 Service 的返回结果，让它适应用户的需求。
4.通过 HTTP 将结果响应给用户。

```

支持restful接口访问的controller，如下:
> PS:　一个controller定义一个资源

``` javascript
const { Controller } = require('egg');
const OPTIONS = Symbol('CONTROLLER_OPTIONS');
const ALLOW_OPTIONS = Symbol('CONTROLLER_ALLOW_OPTIONS');

class SimpleController extends Controller {

    constructor(...args){
        super(...args);
        
        const options = ['OPTIONS'];
        const allow_options = options.slice(0);
        
        if(this.index)
            allow_options.push('GET');
        if(this.clean)
            allow_options.push('DELETE');
        if(this.post)
            allow_options.push('POST');
        
        if(this.get)
            options.push('GET');
        if(this.head)
            options.push('HEAD');
        if(this.put)
            options.push('PUT');
        if(this.patch)
            options.push('PATCH');
        if(this.delete)
            options.push('DELETE');
        
        this[OPTIONS] = options.join(', ');
        this[ALLOW_OPTIONS] = allow_options.join(', ');
    }

   /**
    * 列出所有资源
    */
    async index(){
        const { ctx } = this;
        const res = await ctx.service.simple.find(ctx.query,ctx.params);
        ctx.body = res;
    }
    
  /**
   * 新建一个资源
   */
    async post() {
        const { ctx } = this;
        const res = await ctx.service.simple.createOne(ctx.request.body,ctx.params);
        if (!res) { return; }
        ctx.body = res;
        ctx.status = 201;
    }
    
  /**
   * 获取一个资源内容
   */
    async get() {
        const { ctx } = this;
        const {id,...opts} = ctx.params;
        const res = await ctx.service.simple.findOne({...ctx.query,id},opts);
        if (!res) { return; }
        ctx.body = res;
    }

   /**
    * 更新某个资源内容
    */
    async put() {
        const { ctx } = this;
        const {id,...opts} = ctx.params;
        const res = await ctx.service.simple.replaceOne(ctx.request.body,{...ctx.query,id},opts);
        if (!res) { return; }
        ctx.body = res;
        ctx.status = 200;
    }
    
   /**
    * 更新某个资源的部分内容
    */
    async patch() {
        const { ctx } = this;
        const {id,...opts} = ctx.params;
        const res = await ctx.service.simple.updateOne(ctx.request.body,{...ctx.query,id},opts);
        if (!res) { return; }
        ctx.body = res;
        ctx.status = 200;
    }
    
   /**
    * 删除某个资源
    */
    async delete() {
        const { ctx } = this;
        const {id,...opts} = ctx.params;
        const res = await ctx.service.simple.deleteOne({...ctx.query,id},opts);
        if (!res) { return; }
        ctx.status = 204;
    }
    
   /**
    * 删除所有资源
    */
    async clean() {
        const { ctx } = this;
        const res = await ctx.service.simple.deleteAll(ctx.query,ctx.params);
        if (!res) { return; }
        ctx.status = 204;
    }
    
   /**
    * 获取某个资源的元信息(返回数据格式，内容大小)
    */
    async head() {
        const { ctx } = this;
        const {id,...opts} = ctx.params;
        const res = await ctx.service.simple.findOne({...ctx.query,id},opts);
        if (!res) { return; }
        const buffer = Buffer.form(JSON.stringify(res));
        ctx.set({
            'Content-Type': 'application/json; charset=utf-8',
            'Content-length': buffer.byteLength,
        });
        ctx.status = 200;
    }
    
   /**
    * 获取某个资源的相关信息
    */
    async options(){
        const { ctx } = this;
        ctx.set({
            'Allow': this[OPTIONS],
        });
        ctx.status = 200;
    }
    
   /**
    * 获取资源的相关信息
    */
    async allow(){
        const { ctx } = this;
        ctx.set({
            'Allow': this[ALLOW_OPTIONS],
        });
        ctx.status = 200;
    }
    
}

```

### 业务逻辑 ###
eggjs约定将业务逻辑封装在[service](https://eggjs.org/zh-cn/basics/service.html)中．

支持restful接口访问的资源业务逻辑接口，约定如下:

``` javascript
const { Service } = require('egg');
class SimpleService extends Service {

  /**
   * 列出匹配条件的所有资源
   * @param {Object} condition 匹配条件
   * @param {Object} opts 可选项
   */
   async find(condition,opts) {}

  /**
   * 获取一个匹配的资源
   * @param {Object} condition 匹配条件
   * @param {Object} opts 可选项   
   */
   async findOne(condition,opts) {}

  /**
   * 新建一个资源
   * @param {Object} entity 资源内容
   * @param {Object} opts 可选项      
   */
   async createOne(entity,opts) {}
   
  /**
   * 更新一个匹配资源的内容
   * @param {Object} entity 更新资源内容
   * @param {Object} condition 匹配条件
   * @param {Object} opts 可选项
   */
   async replaceOne(entity,condition,opts){}
   
  /**
   * 更新一个匹配资源的部分内容
   * @param {Object} entity 更新资源内容
   * @param {Object} condition 匹配条件
   * @param {Object} opts 可选项
   */
   async updateOne(entity,condition,opts){}
   
  /**
   * 销毁一个匹配的资源
   * @param {Object} condition 匹配条件
   * @param {Object} opts 可选项
   */
   async deleteOne(condition,opts) {}
   
  /**
   * 销毁匹配的所有资源
   * @param {Object} condition 匹配条件
   * @param {Object} opts 可选项
   */
   async deleteAll(condition,opts) {}
   
}
```

### 功能函数 ###
通常的业务逻辑中，可能会包含一些在多种业务中使用的功能．比如：提取转换数据格式,id生成等．这部分功能代码，通常需要在helper.js中实现．

mongo数据转换&提取示例:
``` javascript
// app/extend/helper/mongose.js
/**
 * 将mongo model数据提取为普通json对象
 */
function extractMongoEntity(model) {
  const { _id, ...entity } = model._doc;
  return entity;
}

/**
 * 将普通json对象转换为mongo model操作的数据
 */
function transformMongo(app, data) {
  if (!data) { return data; }
  const ObjectId = app.mongoose.Types.ObjectId;
  const { id, ...opts } = data;
  return id ? { ...opts, _id: new ObjectId(id) } : opts;
}


// app/service/simple.js
const { Service } = require('egg');
class SimpleService extends Service {
    
    async findOne(condition) {
        const { ctx } = this;
        const _condition = ctx.helper.transformMongo(app, condition);
        const res = await app.model.Simple.findOne(_condition);
        return ctx.helper.extractMongo(res);
    }
    
}

```

### 离线任务 ###
除了正常的http接口调用外，我们还需要一些业务功能，需要使用[任务程序](https://eggjs.org/zh-cn/basics/schedule.html)来实现．

#### 定时任务 ####

``` javascript
const Subscription = require('egg').Subscription;

class Simple extends Subscription {

  // 通过 schedule 属性来设置定时任务的执行间隔等配置
  static get schedule() {
    return {
      interval: '1m', // 1 分钟间隔
      type: 'worker', // 随机指定一个worker执行
    };
  }
  
  // subscribe 是真正定时任务执行时被运行的函数
  async subscribe() {
    const res = await this.ctx.curl('http://www.api.com/cache', {
      dataType: 'json',
    });
    this.ctx.app.cache = res.data;
  }
}
```

适合定时任务的一些常见业务场景：
  * 定时复查更新待支付订单状态
  * 定时清理过期缓存数据

#### 手动任务 ####
由其他功能代码触发执行的离线任务程序．通常是通过消息处理插件触发．

``` javascript
const { Subscription } = require('egg');

class Simple extends Subscription {
  // 禁止被定时触发
  static get schedule() {
    return { disable: true };
  }
  
  // 触发时执行的功能函数
  async subscribe(message) {
    const { ctx } = this;
  }
}
```
常见业务场景：
  * 支付成功后的后续通知消息发送
  * 文章发布后的订阅通知消息发送

## http开发约定 ##
我们还需要实现一些业务逻辑以外的功能，来满足方便服务使用的需求．

### http请求日志 ###
记录http请求日志，有助于确认接口被请求调用的情况．可以方便的帮助开发人员找到或排除问题．

我们需要记录的http日志有两种
  * 访问当前服务接口的请求
  * 当前服务访问其他服务接口的请求
  
> 我们可以通过使用[egg-http-logger](https://github.com/kiba-zhao/egg-http-logger)插件来实现请求日志的记录需求．

### http异常处理 ###
在服务或业务逻辑出现错误时，需要通过抛出错误状态以及一些错误信息，来宣告请求出错的原因．

> 通过使用[egg-http-error](https://github.com/kiba-zhao/egg-http-error)插件来实现

### http补充内容 ###
在服务接口的功能里，会出现一些都需要使用的数据．比如：用户标识,应用标识等．

常用的补充内容：
  * 应用标识(App-ID)：标识不同应用的数据．
  * 认证标识(Auth-ID)：标识当前用户.
  * 客户端名称(Client-Name)：请求访问发起的客户端名称（网关名称）
  * 服务名称(Service-Name)：请求访问发起的服务端名称（服务名）
  
> 为了避免反复书写处理这类数据的功能代码，推荐使用[egg-http-relay](https://github.com/kiba-zhao/egg-http-relay)插件．

> 该插件可以将指定的http header头，设置到ctx.params中．以及在使用eggjs的HttpClient请求其他接口时，将指定http header头传递过去．

### 参数注入 ###
通常我们需要在业务逻辑中，使用http header头里的一些信息．为了方便开发，我们将headers数据注入到ctx.params中使用

> 推荐使用[egg-params-inject](https://github.com/kiba-zhao/egg-params-inject)插件

### swagger接口文档 ###
其他服务或客户端访问当前服务接口，需要提供接口文档以便开发请求访问功能的应用程序．

通常我们需要至少提供以下入口的接口文档：
  * 服务接口文档(openapi.yml)：通常由其他服务或网管请求访问
  * web接口文档(webapi.yml)：通过web网关请求访问的接口

> 推荐使用[egg-swagger](https://github.com/kiba-zhao/egg-swagger)插件．

> 我们需要遵照openapi 3.x规范的接口文档放在app/docs目录下.

### 接口请求验证 ###
请求访问接口的数据，在执行业务逻辑功能前，需要按照指定的数据格式进行检查．不符合格式的数据，需要直接抛出错误提示．
我们希望尽量减少验证代码的开发工作，因此推荐使用swagger接口文档里的接口描述进行验证．

> 插件尚未完成

> 插件会读取app/docs/openapi.yml来进行接口请求验证．

## 数据操作约定 ##
推荐在服务运行之前，先在存储里定义好数据的结构．

### sequelize ###
我们使用[sequelize](https://sequelize.org/)来操作关系性数据库．并且在initdb目录下创建初始化数据库脚本．

> 推荐使用[egg-sequelize](https://github.com/eggjs/egg-sequelize)插件.

### redis ###
我们使用redis来缓存数据

> 推荐使用[egg-redis](https://github.com/eggjs/egg-redis)插件．

### rocketmq ###
我们使用rocketmq来作为服务的消息队列．我们可以通过发布消息来通知其他[离线任务](#离线任务)，也可以通过订阅消息来触发

>插件尚未完成

## 调试 ##
本地开发需要有一个调试环境，这个环境包含相关的数据库，缓存，消息队列以及三方接口．我们将服务运行的环境设置在docker-compose.yml里．利用docker构建本地开发环境

docker-compose.yml示例文件内容：

``` yaml
version: '3'
services:
  mysql:
    image: "mysql:5"
    volumes:
     - ./mysql:/docker-entrypoint-initdb.d
    environment:
      MYSQL_ROOT_PASSWORD: example
    ports:
     - "127.0.0.1:3306:3306"
  redis:
    image: "redis:5-alpine"
    ports:
     - "127.0.0.1:6379:6379"
  json-server:
    image: clue/json-server
    volumes:
     - ./mockdata:/data
    command: --routes /data/routes.json
    ports:
     - "127.0.0.1:3000:80"
  namesrv:
    image: rocketmqinc/rocketmq:4.3.0
    command: sh mqnamesrv
    ports:
      - "127.0.0.1:9876:9876"
  mqbroker:
    image: rocketmqinc/rocketmq:4.3.0
    command: sh mqbroker -n namesrv:9876
    ports:
      - "127.0.0.1:10911:10911"
      - "127.0.0.1:10909:10909"
    depends_on: namesrv
```

yml内容说明：
  * 数据存储： mysql
  * 缓存: redis
  * 伪造http接口: json-server
  * rockermq: namesrv + mqbroker


> eggjs断点调试请参考[官方教程](https://eggjs.org/zh-cn/core/development.html#%E4%BD%BF%E7%94%A8-egg-bin-%E8%B0%83%E8%AF%95)

## 发布 ##
为了方便布局，我们将服务整个打包成docker镜像．

Dockerfile示例文件内容:

``` dockerfile
FROM node:lts-alpine

ENV NODE_ENV=production

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

COPY app ./app
COPY config ./config
COPY package.json ./package.json
COPY node_modules ./node_modules

EXPOSE 80
CMD npm run start
```

## 快速开发 ##
