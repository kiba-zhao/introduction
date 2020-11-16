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
* [http框架约定](#http框架约定)
  * http请求日志
  * http异常处理
  * http补充内容传递
  * swagger接口文档
  * 接口请求验证
* 数据操作约定
  * sequelize
  * redis
  * rocketmq

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
        const res = await ctx.service.simple.find({...ctx.query,...ctx.params});
        ctx.body = res;
    }
    
  /**
   * 新建一个资源
   */
    async post() {
        const { ctx } = this;
        const res = await ctx.service.simple.createOne({...ctx.request.body,...ctx.params});
        if (!res) { return; }
        ctx.body = res;
        ctx.status = 201;
    }
    
  /**
   * 获取一个资源内容
   */
    async get() {
        const { ctx } = this;
        const res = await ctx.service.simple.findOne({ ...ctx.query, ...ctx.params });
        if (!res) { return; }
        ctx.body = res;
    }

   /**
    * 更新某个资源内容
    */
    async put() {
        const { ctx } = this;
        const {id,...opts} = ctx.params;
        const res = await ctx.service.simple.replaceOne({...ctx.request.body,...opts,id},{...ctx.query,...opts});
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
        const res = await ctx.service.simple.updateOne({...ctx.request.body,...opts,id},{...ctx.query,...opts});
        if (!res) { return; }
        ctx.body = res;
        ctx.status = 200;
    }
    
   /**
    * 删除某个资源
    */
    async delete() {
        const { ctx } = this;
        const res = await ctx.service.simple.deleteOne({ ...ctx.query, ...ctx.params });
        if (!res) { return; }
        ctx.status = 204;
    }
    
   /**
    * 删除所有资源
    */
    async clean() {
        const { ctx } = this;
        const res = await ctx.service.simple.deleteAll({ ...ctx.query, ...ctx.params });
        if (!res) { return; }
        ctx.status = 204;
    }
    
   /**
    * 获取某个资源的元信息(返回数据格式，内容大小)
    */
    async head() {
        const { ctx } = this;
        const res = await ctx.service.simple.findOne({ ...ctx.query, ...ctx.params });
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
   */
   async find(condition) {}

  /**
   * 获取一个匹配的资源
   * @param {Object} condition 匹配条件
   */
   async findOne(condition) {}

  /**
   * 新建一个资源
   * @param {Object} entity 资源内容
   */
   async createOne(entity) {}
   
  /**
   * 更新一个匹配资源的内容
   * @param {Object} entity 更新资源内容
   * @param {Object} condition 匹配条件
   */
   async replaceOne(entity,condition){}
   
  /**
   * 更新一个匹配资源的部分内容
   * @param {Object} entity 更新资源内容
   * @param {Object} condition 匹配条件
   */
   async updateOne(entity,condition){}
   
  /**
   * 销毁一个匹配的资源
   * @param {Object} condition 匹配条件
   */
   async deleteOne(condition) {}
   
  /**
   * 销毁匹配的所有资源
   * @param {Object} condition 匹配条件
   */
   async deleteAll(condition) {}
   
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

## http框架约定 ##
