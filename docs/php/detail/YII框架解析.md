## YII框架主要模块

* 控制器（Controllers）
* 业务模型（Models）
* 视图（Views）
* 独立模块（Modules）
* 过虑器（Filters）
* 部件（Widgets）
* 前端资源（Assets）
* 一些应用扩展（Extensions）
* 入口脚本（index.php）
* 主应用（Application）
* 组件（Components）

![](https://github.com/bingbo/blog/blob/master/images/yii/yii%E6%A8%A1%E5%9D%97%E5%9B%BE.jpg)



## YII框架处理请求涉及的主要类

基本上所有的功能模块都是一个组件，都继承于`Component`基类，有些最基本的属性和方法。

![](https://github.com/bingbo/blog/blob/master/images/yii/yii%E4%B8%BB%E8%A6%81%E7%B1%BB%E5%9B%BE.jpg)

## YII框架请求处理流程

请求由入口文件`index.php`进入，然后会加载相应的`autoload`类自动加载机制和全局`YII`对象，同时会根据`config\web.php`的配置进行初始化`Application`，其中会对最基本的组件进行注入及初始化工作，之后由`UrlManager`进行请求解析，路由到相应的`Controller`及`Action`进行处理请求，如下图所示：

![](https://github.com/bingbo/blog/blob/master/images/yii/yii%E4%B8%BB%E8%A6%81%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

## YII业务模型涉及的主要类

这里列出了业务模型进行业务逻辑处理涉及到的主要一些类，基本上也是做为组件存在，也都继承于`Component`。利用`PDO`与各种数据库进行交互。

![](https://github.com/bingbo/blog/blob/master/images/yii/%E4%B8%9A%E5%8A%A1%E6%A8%A1%E5%9E%8B%E7%B1%BB%E5%9B%BE.jpg)

## YII业务模型逻辑处理流程

这里省去请求路由，直接进行模型业务逻辑的处理。即由相应的`Action`直接到相应的`Model`，基本上一个`Model`对应数据库的一个表，其里面所有的操作基本上是对数据库及数据的逻辑操作，其逻辑处理流程如下图所示：

![](https://github.com/bingbo/blog/blob/master/images/yii/%E4%B8%9A%E5%8A%A1%E6%A8%A1%E5%9E%8B%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)