
项目  | 描述
------------- | -------------
维护人员  | 奚杉杉
创建时间  | 2015-06-19

# TOC

- [TL;DR;](#user-content-tldr)
- [问题域](#user-content-问题域)
- [当前非重点](#user-content-当前非重点)
  - [前端基础架子](#user-content-1-前端基础架子)
  - [前端开发辅助工具搭建](#user-content-2-前端开发辅助工具搭建)
  - [data fetching](#user-content-3-data-fetching)
  - [API 设计](#user-content-4-api设计)
  - [functional & reactive programming](#user-content-5-functionalreactive-programming)

## TL;DR;

以下为我在前端实践中遇到的问题的记录，解决方案探讨的记录等一切记录。

以下前端特质HR项目中实施的前端，关键词为：点评人力资源平台，PC端，ace admin，react，flux等。

参考示例：[IDP 前端](http://code.dianpingoa.com/shanshan.xi/ehr-ace-seed)。IDP 非常简单，代码大多为架子代码，业务代码很少。

## 问题域

以下为非合并罗列：

1. 前端基础架子，即分层，separation of concerns
2. 前端开发辅助工具搭建，包括包管理，task runner，和其他辅助工具
3. data fetching
4. API 设计
5. functional / reactive programming
6. 组件开发
7. 全局样式和组件样式
8. jQuery 等 DOM 操作

## 当前非重点

满足以下条件：已经完成，对于目前可用，继续研究无太大突破，项目不紧急需要的特性。

### 1. 前端基础架子

目前方案已经经过面试管理和绩效前台/配置，IDP的项目实践，可以支撑未来几个项目开发。

特征：

1. mental model 简单清晰，顺应自然思维，没有 hacky 逻辑；
2. 学习成本低，入手快；
3. separation of concerns，且分层很清晰

一级目录

```
├── README.md
├── assets                       // 当前存放 ace admin assets
├── build                        // webpack dev server 的 publicPath
├── dist                         // browser-sync（模拟 production） 和 production 的 dist 目录
├── gulpfile.js
├── package.json
├── src                          // 项目文件
├── webpack                      // webpack 公共配置
├── webpack-dev.config.js        // webpack dev 配置，包括 HMR
├── webpack-production.config.js // webpack production 配置
└── webpack-sync.config.js       // webpack browser-sync 配置
```

src 目录

```
├── client                 // 项目文件
│   ├── alt.js             // alt 实例初始化
│   ├── main.js            // entry point
│   ├── pages              // 页面文件，包括 actions, stores
│   ├── router.js          // route 公共配置
│   ├── routes.js          // route 的页面，path 配置
│   └── utils              // 常用工具
├── dumb                   // dumb 组件，包括 ace等其他组件
│   ├── ace
│   ├── ace.js
│   ├── select
│   ├── select.js
│   ├── table
│   └── table.js
├── lib                    // 第三方库
│   ├── getrandomstring.js
│   └── superagent
└── server                 // 本地开发 mock server 配置
    ├── apis.js
    ├── config.js
    ├── express.js
    ├── index.html
    ├── index.js
    ├── json
    ├── main.js
    └── sync

```


pages 目录（*）

```
├── App                       // 主页面
│   ├── App.js                // dump component，使用 flux actions
│   ├── App.scss              // 样式文件
│   └── AppContainer.js       // smart component
├── MyDetail                  // 业务页面1
│   ├── MyDetailContainer.js
│   └── MyDetailPage.js
├── MyList                    // 业务页面2
│   ├── MyListContainer.js
│   └── MyListPage.js
├── SubDetail                 // 业务页面3
│   ├── SubDetailContainer.js
│   └── SubDetailPage.js
├── SubList                   // 业务页面4
│   ├── SubListContainer.js
│   └── SubListPage.js
├── actions                   // actions，主要包括直接 dispatch 页面，和 async actions（ajax 请求）
│   ├── CommonActions.js
│   └── IdpActions.js
└── stores                    // stores, 数据操作
    ├── CommonStore.js
    └── IdpStore.js

```

（*）此为核心，有几个特点

1. 必须包装 Container（smart component），包括 data fetch（注1），stores 监听，获取 stores 值并使用 props 传递给 children。
2. stores（注2） 和 actions 根据架构分层（注3），业务页面根据业务分层（smart，dumb，stylesheets）。

**注1**：这里在 Container 的 componentDidMount 获取，也可以在 componentWillMount，也可以更复杂的，给出公共 static 接口，由 react-router 获取。这里选择最简单清晰的方式，没有看出这几个 hook 的区别，特别是 react-router。

**注2**：简单应用 store 可以为一个（IdpStore）。除了代码量可能会变多之外，多个 stores 没有看出什么好处。另外，这一块也是未成熟的地方。暂时不深究，flux 也没有给出很好的解释。之后可以从其他设计得到灵感。

**注3**：actions 包括 serverActionCreators 和 viewActionCreators）。这块和 pages 耦合紧密，可以放到 pages 里。

### 2. 前端开发辅助工具搭建

已经成熟，在做服务端渲染前，可以冻结。

```
├── gulpfile.js
├── webpack                      // webpack 公共配置
├── webpack-dev.config.js        // webpack dev 配置，包括 HMR
├── webpack-production.config.js // webpack production 配置
└── webpack-sync.config.js       // webpack browser-sync 配置
```

webpack 都是 low-level 的 API，具体使用看 gulpfile 就行。

### 3. data fetching

上面 1 提到的是何时 fetch data，包括如何处理 async actions。

这里的是指 ajax lib 选择，目前使用最简单的 superagent 获取，简单加了两个插件，即 AOP 插件和 no-cache 插件。

期望，发送请求时，给出 hook，设置为 loading，请求成功，请求失败，这样可以设置页面 loading 效果。其他优化需要深入研究

### 4. API设计

暂时未成熟，使用广义的 RESTful 接口。强依赖后端。前端 data-agnostic。

需要继续研究，改善前后端通讯。

### 5. functional & reactive programming

暂时不研究。

以上为当前非重点领域。

## 当前重点