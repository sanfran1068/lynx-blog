---
layout: post
title: "EggJS源码阅读-启动流程与代码实现"
date: 2020-05-23 12:30:00
comments: true
categories: 
- SourceCodeReading
tags:
- JavaScript
- 源码学习
- EggJS
- NodeJS
---

egg是阿里开源的一个框架，为企业级框架和应用而生，相较于express和koa，有更加严格的目录结构和规范。不同的团队可以基于egg，根据自己的需求封装出适合团队业务的更上层框架。本文对 egg.js 源码进行一个简要的入门阅读和解析。

<!-- more -->

### egg如何启动

![fca8379caeffb9c194aa5369db1c563b.png](https://img.alicdn.com/imgextra/i4/O1CN01gbgC311e0ZRz21oLP_!!6000000003809-0-tps-1268-1002.jpg)

根据`package.json`中的`script`命令，可以看到执行的直接是`egg-bin dev`的命令。找到`egg-bin`文件夹中的`dev.js`,会看到里面会去执行外层的`start-cluster`文件:

```javasript
require(options.framework).startCluster(options);
```

此处的`options.framework`其实指的就是`egg`框架，然后调用了`egg-cluster`包中的`startCluster`方法，egg正式迈出了启动的第一步。

[源码流程]

`egg`框架采用了`master-agent-worder`的集群模式，如果所示，官方文档中也对于这三种进程的启动和通信给出了较为详细的说明：

- Master 启动后先 fork Agent 进程
- Agent 初始化成功后，通过 IPC 通道通知 Master
- Master 再 fork 多个 App Worker
- App Worker 初始化成功，通知 Master
- 所有的进程初始化成功后，Master 通知 Agent 和 Worker 应用启动成功

我们尝试从源码中大致将这个流程实现出来：

```javascript
// egg-cluster/lib/master.js
ready.mixin(this);              // 将 ready 方法挂在 Master 上
this.detectPorts().then(() => {  // 检测端口
    this.forkAgentWorker();      // 启动 agent 进程，发送 agent-start 消息给 master 进程
});

// master 进程一旦接收到 agent-start 消息后，开始创建 worker 进程
this.once('agent-start', this.forkAppWorkers.bind(this));
```

[源码流程]

### Agent与AppWorker的实现

#### AgentWorker与AppWorker进程的启动

![de59fd88a438492231f38fb2032ffd5b.png](https://img.alicdn.com/imgextra/i4/O1CN01Fyk5k31EQVsRROl7M_!!6000000000346-0-tps-1337-1029.jpg)


在启动`AgentWorker`和`AppWorker`时，会分别加载`agent_worker.js`和`app_worker.js`两个文件并创建进程，其中`agent_worker.js`中会创建`Agent`类的实例，而`app_worker.js`中会创建`Application`类的实例。

两种进程在启动时都会调用`this.loader.load()`方法来加载自己相应的一些插件和自定义的扩展。

![18d21732106d0293791bf03d6481784b.png](https://img.alicdn.com/imgextra/i1/O1CN01aX0q421ZQbtFSpqUz_!!6000000003189-0-tps-1335-716.jpg)

基于`egg_loader`实现了`AppWorkerLoader`和 `AgentWorkerLoader`，上层框架基于这两个类来扩展，`Loader`的扩展只能在框架进行。

```javascript
import { AppWorkerLoader } from 'egg'
import common from './common'

class MyAppWorkerLoader extends AppWorkerLoader {
    constructor(opt) {
    super(opt);
        // 自定义初始化
    }

    loadConfig() {
        super.loadConfig();
        // 对 config 进行处理
    }

    load() {
        super.load();
        // 自定义加载其他目录
        // 或对已加载的文件进行处理
    }
}
Object.assign(MyAppWorkerLoader.prototype, {
    // 可以根据自己的需求，重写一些获取配置的方法
    getServerEnv(){};
    _preloadAppConfig(){}
    getTypeFiles(){}
})
export default MyAppWorkerLoader
```

#### Agent如何实现

![f2ac3732452f487f21e11c0de480f3df.png](https://img.alicdn.com/imgextra/i1/O1CN01zkrj4o1OUliCxYhjc_!!6000000001709-0-tps-1093-760.jpg)

`Agent`对象在`egg-cluster`创建环节中被创建出来，继承自`egg.Agent`对象，该对象继承`EggApplication`,且`loader`为`./lib/loader/agent_worker_loader.js`文件，继承自`egg-core.eggLoader`对象，整体继承链如上图。

```javascript
// egg/lib/agent.js
class Agent extends EggApplication {
  constructor(options = {}) {
    options.type = 'agent';
    // 1.完成父类构建
    super(options);
    // 2.驱动loader执行load方法
    this.loader.load();
    // 3.dump相关配置文件进入./run目录下
    this.dumpConfig();
    
    setInterval(() => {}, 24 * 60 * 60 * 1000);
    // 4.监听异常事件
    this._uncaughtExceptionHandler = this._uncaughtExceptionHandler.bind(this);
    process.on('uncaughtException', this._uncaughtExceptionHandler);
  }
}
```

`Agent`类的实现中，主要是实例化了`EggApplication`，调用了`this.loader.load()`方法来加载各种文件和配置。

```javascript
// egg/lib/egg.js
class EggApplication extends EggCore {
    constructor(options = {}) {
        options.mode = options.mode || 'cluster';
        // 1.原型EggCore构建
        super(options);
        // 2.调用loadConfig，在agent的实现中，指向的并不是egg-core.loader，而是agent_worker_loader
        this.loader.loadConfig();
        // 3.ready事件
        this.ready(() => process.nextTick(() => {
          const dumpStartTime = Date.now();
          this.dumpConfig();
          this.dumpTiming();
        }));
        // 监听unhandleRejection事件
        // 4.cluster初始化
        this[CLUSTER_CLIENTS] = [];
        this.cluster = (clientClass, options) => {};
    }
}
```

在`EggApplication`类的实现中，我们可以看到是继承自`EggCore`类，在父类构建好之后，会调用`this.loader.loadConfig()`方法，该方法的`loader`实例实际指向了`AgentWorkerLoader（agent_worker_loader.js）`：

```javascript
// egg/lib/loader/agent_worker_loader.js
class AgentWorkerLoader extends EggLoader {
  loadConfig() {
    this.loadPlugin();  // loadPlugin from egg-core/lib/loader/mixin/plgin.js
    super.loadConfig(); // loadConfig from egg-core/lib/loader/mixin/config.js
  }
  
  load() {
    this.loadAgentExtend();
    this.loadContextExtend();

    this.loadCustomAgent();
  }
}
```

`loadPlugin`方法会加载三种插件：
- `eggPlugins`，从eggjs框架配置的插件，也就是`egg/config/plugins.js`文件中egg框架自带的插件；
- `appPlugins`，每个应用自己配置的插件，也就是`myproject/config/plugins.js`，用户可以自定义配置一些特殊的插件；
- `customPlugins`，应用启动命令中参数EGG_PLUGINS值所代表的插件；

最后会将这三种插件都挂在app实例上：`this.plugins = enablePlugins`;

`loadConfig`方法会加载三种配置：
- `appConfig`，每个应用自己独有的配置，其中会按顺序加载两个配置，一个是默认配置`config.default`，另一个是当前环境的配置`config.${this.serverEnv}`，也就是`myproject/config`下的一些配置文件加载
- 加载自定义添加的`plugin`插件的配置
- 加载框架`egg`的配置，即`egg/config`
- 重新加载应用`app`的配置，即`myproject/config`下的

最后将合并的配置挂载在`app`实例上`this.config = target`;

#### Application如何实现

上面提到，`AppWorker`在实例化的过程中，会调用`this.loader.load()`。进入具体这个`Application`所对应的`loader.load`方法，可以发现`Application`的实现比`Agent`的实现多调用了很多加载的方法：

- `this.loadApplicationExtend();`，该方法的调用会给应用加载扩展方法，加载路径为`app\extend\application.js`, 会将对应的对象挂载在app 应用上。
- `this.loadRequestExtend();`，加载`app\extend\request.js`
- `this.loadResponseExtend();`，加载`app\extend\response.js`
- `this.loadContextExtend();`，加载`app\extend\context.js`
- `this.loadHelperExtend();`，加载`app\extend\helper.js`
- `this.loadService()`
- `this.loadMiddleware()`
- `this.loadController()`
- `this.loadRouter()`

### 总结

1. egg启动服务集群，采用了`master-agent-worker`模式，`AgentWorker`和`AppWorker`都由Application(egg/lib/applicaton.js) -> EggApplication(egg/lib/egg.js) -> EggCore(egg-core/lib/egg.js) -> KoaApplication(koa)原型链进行继承

2. `Agent`和`Application`在实例化的过程中，都会调用相应的Loader去加载自己所需的插件和配置，且加载顺序严格按照**插件plugin-框架framework-应用application**这样一个顺序

![0464781205f05b89c1b8807280114dd8.png](https://img.alicdn.com/imgextra/i1/O1CN015m0LOC1ZwfcF0EBGW_!!6000000003259-2-tps-888-2235.png)
