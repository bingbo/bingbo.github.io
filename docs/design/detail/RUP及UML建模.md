
## RUP(统一过程)

软件工程的过程。它提供了在开发组织中分派任务和责任的纪律化方法。它的目标是在可预见的日程和预算前提下，确保满足最终用户需求的高质量产品。


### RUP基石

* 方法 - 建议的开发软件系统的策略与方法

   方法学定义了能够有助于开发一个系统的若干模型，一个高层次的抽象概括；还定义了一组图形化规范表示法来描写建议的模型，形成文档；还定义了开发过程中的不同活动；结构化方法学，面向对象方法学

* 工具
* 过程


### RUP核心原则

* Use case driven（用例驱动）
* Architecture centric （以体系架构为中心）
* Incremental（增量）
* Iterative（迭代）


### RUP四个阶段

![](https://github.com/bingbo/blog/blob/master/images/uml/%CD%BC%C6%AC1.png)

* 构思阶段 ：包括用户沟通和计划活动两个方面，强调定义和细化用例，并将其作为主要模型。
* 细化阶段 ：包括用户沟通和建模活动，重点是创建分析和设计模型，强调类的定义和体系结构的表示。
* 构建阶段 ：将设计转化为实现，并进行集成和测试。
* 移交阶段 ：将本次迭代的可用产品移交给用户。


### 迭代

![](https://github.com/bingbo/blog/blob/master/images/uml/%CD%BC%C6%AC2.png)

1. Use case modeling			用例建模
2. Domain modeling			领域建模
3. Interaction modeling(Object Interaction Modeling & Design)对象交互 建模与设计
4. Derive design class diagram	推导设计类图
5. Implementation and deployment	实施和部署


## UML（统一建模语言）

(UML)又称统一建模语言或标准建模语言，是始于1997年一个OMG标准，它是一个支持模型化和软件系统开发的图形化语言，为软件开发的所有阶段提供模型化和可视化支持，包括由需求分析到规格，到构造和配置。


### Use Case Modeling（用例建模）

![](https://github.com/bingbo/blog/blob/master/images/uml/use_case.jpg) 
### Domain Modeling（领域建模）

![](https://github.com/bingbo/blog/blob/master/images/uml/class.jpg)

1. 收集应用领域信息
2. 头脑风暴
3. 分类汇总
4. 可视化
5. 找出对象、属性及关联关系 
 
### Object Interaction Modeling & Design（对象交互建模与设计）
 
![](https://github.com/bingbo/blog/blob/master/images/uml/sequence.jpg)

**分析和设计的区别**

分析 | 设计
---- | ---- 
面向应用问题 |	面向软件解决方案
应用领域建模 |      软件系统建模
介绍了世界是什么 |	指出软件解决方案
以项目为导向的决策 |	系统决策
应该允许多个设计方案 |	通常减少实现的选择

### Design Class Diagram（设计类图） 

![](https://github.com/bingbo/blog/blob/master/images/uml/d_class.jpg)

### Implementation（实施） 

**Component View**

![](https://github.com/bingbo/blog/blob/master/images/uml/component.jpg)

**Activity Diagram**

![](https://github.com/bingbo/blog/blob/master/images/uml/%BB%E6%CD%BC8.jpg)

**State Diagram**

![](https://github.com/bingbo/blog/blob/master/images/uml/state.jpg)
