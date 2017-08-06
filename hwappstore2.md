本篇作为 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)的第一篇。【MVC与MVP】

 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)连载记录项目[仿华为应用市场](https://github.com/ScWen7/HWAPPStore)的开发过程。

本系列的代码同步更新到[https://github.com/ScWen7/HWAPPStore](https://github.com/ScWen7/HWAPPStore) 。

---------------------------------------------------------------------------------------------------

# 前言

​        关于项目的结构，目前使用最多的是MVC与MVP。大家对这个两个名词也是耳熟能详。可以说MVP模式是MVC模式在Android上的一种变体，为了更好地细分视图(View)与模型(Model)的功能，让View专注于处理数据的可视化以及与用户的交互，同时让Model只关系数据的处理，让Model和View完全解耦。本篇文章内容为作者对MVC于MVP 的一些个人看法。

# MVC

MVC三大要素：Model 实体逻辑处理层   View  视图层     控制层的Controller

![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/hwp_mvc.png)

+ 其中View层UI界面，用于展示数据和用户输入;在Androird项目中对应的是 layout.xml文件和自定义View

+ Model层就是实体类和数据库等数据处理;在Android 中对应的是JavaBean、DAO和各种处理数据Helper等。

+ Controller控制器控制Model和View;在Android 中对应的就是Activity或Fragment。

View层接受用户的输入，然后通过Controller修改对应的Model实例或者View层也可以直接更新Model实例的数据；同时，当Model实例的数据发生变化的时候，需要修改UI界面，可以通过Controller更新。比如你的界面有一个按钮，按下这个按钮去登录，这个按钮是view层的，是使用xml来写的，而那些和网络连接相关的代码写在其他类里，比如你可以写一个专门的LoginHelper类，这个就是model层。而连接View与Model是通过button.setOnClickListener()这个函数，这个函数就写在了activity中，对应于controller层。

大家想过这样会有什么问题吗？显然是有的，按照MVC的分层，问题就在于xml作为view层，控制能力实在太弱了，实际上关于该布局文件中的数据绑定的操作，事件处理的代码都在Activity中，造成了Activity既像View又像Controller。

而在Activity接收用户的输入，此外还要承担一些生命周期的工作。Activity是在Android开发中充当非常重要的角色，特别是TA的生命周期的功能，所以开发的时候我们经常把一些业务逻辑直接写在Activity里面，这非常直观方便，代价就是Activity会越来越臃肿，如果是一个逻辑很复杂的页面，activity是不是动辄上千行呢？这样不仅写起来麻烦，维护起来更是噩梦。而且如果是一些可以通用的业务逻辑（比如用户登录），写在具体的Activity里就意味着这个逻辑不能复用了。

MVC还有一个重要的缺陷，view层和model层是相互可知的，这意味着两层之间存在耦合，耦合对于一个大型程序来说是非常致命的，如果业务调整的话，要维护起来就难了。


# MVP
MVP三大要素：Model 实体逻辑处理层   View  视图层     Presenter 管理者Model与View之间的连接纽带
![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/hwp_mvp.png)

- View层UI界面，用于展示数据和用户输入;在Androird项目中对应的是 layout.xml文件、Activity

- Model层就是实体类和数据库等数据处理;在Android 中对应的是JavaBean、DAO和各种处理数据Helper等。

- Presenter层作是Model 与View之间的连接桥梁，决定了业务的逻辑

View层接受用户的输入，通知Presenter事件产生，Presenter操纵Model去请求处理数据，处理完成之后，由Presenter去通知View进行视图的更新。

再拿登录的例子来说，这个按钮是view层的，网路请求的LoginModel在Model 层中，通过Presenter中的函数通知需要执行登录操作，Presenter调用Model中的方法进行登录，登录完成回调到Presenter中，Presenter在通知View 去显示登录完成的状态。View 与Model 是互相不可知的，只有Presenter 对双方是可见的。

View 层只要专注于UI 的显示，Model只关注于数据的请求和处理，其他的逻辑全部交于Presenter 来进行处理。

# 优缺点
### MVP的优点:
   (1)降低耦合度 View与Model完全分离
   (2)模块职责划分明显  视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去
   (3)逻辑清晰 代码的可阅读性增强
   (4)代码复用 对于一些Presenter、Model进行复用
   (5)代码灵活性
其中我觉得很爽的有三点：
+ Activity 代码变得更加简洁，很多时候阅读代码，都是从Activity开始的，对着一个1000+行代码的	Activity，各种调用，看了都觉得难受。
  使用MVP之后，Activity就能瘦身许多了，基本上只有初始化，视图监听的操作。其他的就是对Presenter的调用，还有对View接口的实现。这种情形下阅读代码就容易多了，一看就知道视图执行了那些操作。
+ 逻辑很清晰，View,Model，Presenter三层各司其职，只要看Presenter的接口，就能明白这个模块都有哪些业务，很快就能定位到具体代码。
+ 方便进行单元测试一般单元测试都是用来测试某些新加的业务逻辑有没有问题，对于MVP来说，我们可以在没有编写完成Model 或者View的情况下进行测试，进行一些数据或者视图的模拟测试

### MVP模式缺点：
+ 由于对视图的渲染放在了Presenter中，所以视图和Presenter的交互会过于频繁。
+ 代码量大，在原来的基础上增加了Presenter,View 的接口，同时还需要定义很多回调的接口增加了很多的接口和方法，在Android中一个DEX最多容纳65536个方法，虽然现在高版本对多Dex支持较好，但是还是要最大化的较少方法数。
+ 还有一点，如果Presenter过多地渲染了视图，往往会使得它与特定的视图的联系过于紧密。一旦视图需要变更，那么Presenter也需要变更了

### 对于两种模式的选择
对于MVC与MVP没有绝对的好坏，结合上面的分析:
+ 对于逻辑相对简单的界面，并且业务复用频率低，采取MVC 的形式，直接在Activity 中编写业务逻辑代码
+ 对于逻辑复杂，业务复用频率很高的部分采用MVP 的形式


# 下篇预告

本文作者只是从理论上介绍了MVC 与MVP，下一篇  结合实际业务用代码提现 MVP 在项目中的使用 

[MVP使案例](https://github.com/ScWen7/Blogs/blob/master/hwappstore2.md)