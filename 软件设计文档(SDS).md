# 软件设计文档 (SD)

    注：闲余翻身微信小程序同时也是该小组成员在 2019 年春季学期中山大学系统分析与设计课程的大作业。按照本课程作业要求在 SD 文档开头点明。

## 1. Client

### 1.1 技术选型及理由

**微信小程序框架**

我们选择了微信小程序开发框架作为用户点餐系统的前端框架。

- 小程序框架提供了自己的视图层描述语言 WXML 和 WXSS，以及基于 JavaScript 的逻辑层框架，容易上手；
- 在视图层与逻辑层间提供了数据传输和事件系统，支持响应式、双向数据绑定等特性，可以让数据与视图非常简单地保持同步：当做数据修改的时候，只需要在逻辑层修改数据，视图层就会做相应的更新，因而能够很好地支持丰富的用户交互体验支持，可以让开发者方便的聚焦于数据与逻辑上；
- 框架还管理了整个小程序的页面路由，可以做到页面间的无缝切换，并给以页面完整的生命周期，开发者需要做的只是将页面的数据，方法，生命周期函数注册进框架中，其他的一切复杂的操作都交由框架处理；
- 此外，框架还提供了微信风格的组件和丰富的api，实现了尽量以简单、高效的方式让开发者可以在微信中开发具有原生 APP 体验的服务。
- 4g网络和移动支付的普及推动了微信小程序作为点餐系统的应用发展，使用微信小程序作为点餐系统对用户更加快捷和便利。

基于以上理由，选择微信小程序作为前端框架对用户和开发者都十分友好。

#### 1.2 架构设计

**wepy框架**

wepy框架是腾讯官方推出的小程序开发框架，开发风格接近于 Vue.js，支持组件 Props 传值，自定义事件、组件分布式复用Mixin、计算属性函数computed、模板内容分发slot等等。支持组件化开发，完美解决组件隔离，组件嵌套，组件通信等问题，组件通信等问题，支持使用第三方 npm 资源，自动处理 npm 资源之间的依赖关系，完美兼容所有无平台依赖的 npm 资源包，通过 polyfill 让小程序完美支持 Promise，解决回调烦恼，可使用Generator Function / Class / Async Function 等特性，大大提升开发效率。框架对小程序本身进行了优化，如请求列对处理，优雅的事件处理，生命周期的补充，性能的优化等等，支持样式编译器：Less/Sass/Styus，模板编译器：wx-ml/Pug，代码编译器：Babel/Typescript。

**wepy项目目录结构**

```
├── dist                   小程序运行代码目录（该目录由WePY的build指令自动编译生成，请不要直接修改该目录下的文件）
├── node_modules           
├── src                    代码编写的目录（该目录为使用WePY后的开发目录）
|   ├── components         WePY组件目录（组件不属于完整页面，仅供完整页面或其他组件引用）
|   |   ├── com_a.wpy      可复用的WePY组件a
|   |   └── com_b.wpy      可复用的WePY组件b
|   ├── pages              WePY页面目录（属于完整页面）
|   |   ├── index.wpy      index页面（经build后，会在dist目录下的pages目录生成index.js、index.json、index.wxml和index.wxss文件）
|   |   └── other.wpy      other页面（经build后，会在dist目录下的pages目录生成other.js、other.json、other.wxml和other.wxss文件）
|   └── app.wpy            小程序配置项（全局数据、样式、声明钩子等；经build后，会在dist目录下生成app.js、app.json和app.wxss文件）
└── package.json           项目的package配置
```

一个page或者component包含:

| 脚本部分 | `<script></script>`标签中的内容 | 对应原生.js文件和.json文件 |
| -------- | ------------------------------- | -------------------------- |
| 结构部分 | `<template></template>`模板部分 | 对应于原生的`.wxml`文件    |
| 样式部分 | `<style></style>`样式部分       | 对应于原生的`.wxss`文件    |

**wepy的编译过程如下**

![](https://cloud.githubusercontent.com/assets/2182004/22774706/422375b0-eee3-11e6-9046-04d9cd3aa429.png)

#### 1.3 模块划分

根据业务逻辑和UI设计图，将用户点餐系统前端分为多个模块，按照wepy开发标准，每个页面对应pages目录下一个.wpy文件

主要页面模块包括：

* **授权** ：用户对小程序进行授权，小程序获取用户的部分微信个人信息，包括昵称头像，地区等。
* **登录模块**：用户输入对应的邮箱以及密码登录小程序
* **注册模块**：用户输入中大邮箱或QQ邮箱，进行注册并验证，在邮箱中获取验证码之后，完善其余注册信息。
* **任务列表模块**：用户获取当前可接受任务的列表
* **发布任务模块**：用户发布任务，填写相关标题、描述、人数、报酬、以及图片描述等信息，当前可发布问卷、取快递、其他三种类型。
* **个人信息模块**：用户个人信息管理模块，实现用户个人信息的修改与查看。
* **我的任务模块**：用户接受的所有任务的管理
* **我的发布模块**：用户发布的所有任务的管理
* **任务详情模块**：任务的详细信息显示

此外，还定义了三个组件，构成组件模块。三个组件包括

* 任务卡片组件，主要封装了任务简单信息的样式结构和交互事件
* topbar组件，顶部导航栏组件
* navbar组件，底部导航栏组件

为了是样式个美观，还是用了一些样式库和组件库，如`weui`，`iview`

加上小程序的全局文件，构成app模块； 还有资源文件，包括图片，构成资源模块； 最后还有小程序的工具配置模块，和公共代码模块，api模块用于请求数据，各个模块关系如下：

![](images/client-jia-gou.png)

最终目录结构:

![](images/client-mu-lu.png)

其中src目录具体结构如下

    ├── src
        ├── app.wpy                 //入口文件
        ├── api                     //api接口
        |   ├── index.js            // api的index文件
        |   ├── task.js             // 任务相关的api连接函数
        |   ├── user.js             // 用户相关的api连接函数
        ├── components              //自定义组件
        |   ├── questionitem.wpy    // 问卷中问题组件
        |   ├── topbar.wpy          // 顶部导航栏组件
        |   ├── taskcard.wpy        // 任务卡片组件
        ├── images                  //图片文件夹
        ├── pages                   //页面
        │   ├── answerquestion.wpy  //问卷回答
        │   ├── authorize.wpy       //授权页面
        │   ├── completetask.wpy    //完成任务
        │   ├── createtask.wpy      //创建任务
        |   ├── detail.wpy          //任务详情页面
        |   ├── edit-user-info.wpy  //用户信息修改界面
        |   ├── favorite.wpy        //收藏界面  
        |   ├── home.wpy            //主界面
        |   ├── login.wpy           //登录页面
        |   ├── message.wpy         //消息页面
        |   ├── mypublish.wpy       //我发布的页面
        |   ├── mytask.wpy          //我的任务
        |   ├── point.wpy           //积分信息显示
        |   ├── profile.wpy         //我页面
        |   ├── question.wpy        //问卷发布页面
        |   ├── recharge.wpy        //充值页面
        |   ├── register.wpy        //注册页面
        |   ├── search.wpy          //搜索页面
        |   ├── setpass.wpy         //修改密码
        |   ├── userinfo.wpy        //修改个人信息
        ├── resource                //资源文件
        │   ├── iview               //iview组件
        ├── style                   //样式
        │   ├── weui.wxss           //weui样式文件
        │   ├── icon.less           //icon样式文件
        ├── utils                   //工具包
        │   ├── contant.js          //缓存常量定义文件    
    ├── package.json                //配置文件
    ├── wepy.config.js              //配置文件   

#### 1.1.4 软件设计技术

**MVVM设计模式**

MVVM的全称为Model-View-ViewModel，M表示Model，V表示视图View，VM表示数据与模型，当前端View变化时，由于View与VM进行了绑定，VM又与M进行交互，从而使M得到了改变；当M发生变化时，小程序检测到变化并通知VM，由于VM和V进行了绑定，因此V得到改变。

wepy内的设计思想就包含了MVVM，因此小程序内页面和组件模块都自动采用了MVVM设计模式。

