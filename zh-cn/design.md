# fino设计文档

首先fino由以下几个模块组成

![image-20210922151512656](https://tva1.sinaimg.cn/large/008i3skNly1gupfft4tylj60ro0amab402.jpg)

其中数据模块和调度模块：

![image-20210922151548394](https://tva1.sinaimg.cn/large/008i3skNly1gupffpxjjyj60rm0bm0tv02.jpg)



那么这些模块是怎么协调运作的呢？

首先就是注册模块， 所有的子模块调用registerApp方法， 会得到这样的一个appList

![](https://tva1.sinaimg.cn/large/008i3skNly1gpzbity8thj30fk0c675m.jpg)

之后调度模块就是根据activeWhen字段，对所有的应用进行调度。 即**挂载应用，加载资源， 开启沙箱， 设置运行环境等工作 **。

跟`single-spa`不同的是。 因为`single-spa`需要在同一个页面上挂载多个应用，

所以可能会发生同一时间多个应用需要同时调度的情况。 

所以它们采用了一个事件循环的机制来管理和调度各个应用的生命周期。 

<span style="color:#4ba8c9">但是因为fino框架的应用是以页面为单位。</span> 所以不存在多个应用同时调度和加载的问题。 

我们只需要针对于应用进行一个预加载就可以了。 

所以流程上会比简单的single-spa更简单。

我们来简单分析一下它的流程。



先看一下一个app的生命周期：

![](https://tva1.sinaimg.cn/large/008i3skNly1gpzn9i3uyqj313405wdh4.jpg)

再来看一下总体的流程

![](https://tva1.sinaimg.cn/large/008i3skNly1gpzn8hxmjdj31re0r8793.jpg)







每个模块的分析见各自模块。

接下来看一下整体架构

