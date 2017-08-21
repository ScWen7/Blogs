本篇作为 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)的第四篇。【LoadMoreRecyclerView扩展之添加Header和Footer】

 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)连载记录项目[仿华为应用市场](https://github.com/ScWen7/HWAPPStore)的开发过程。

本系列的代码同步更新到[https://github.com/ScWen7/HWAPPStore](https://github.com/ScWen7/HWAPPStore) 。

---------------------------------------------------------------------------------------------------

# 前言

​     	在上一篇[封装加载更多的RecyclerView](http://www.jianshu.com/p/ceb6f4f64c99)中对RecylerView加载更多的封装进行了分析和实现，本篇是在上一篇的基础上对LoadMoreRecyclerView 进行进一步的扩展**支持添加Header和Footer View**

### 实现分析

在上一篇[封装加载更多的RecyclerView](http://www.jianshu.com/p/ceb6f4f64c99)中分析道RecyclerView 实现上拉刷新是采用封装Adapter的方式来实现的。而添加HeaderView 和FooterView 与其道理相同保留上拉加载将上拉加载更多的视图作为一个单独的Type 视图放在RecyclerView列表的底部，HeaderView作为一系列的类型放在列表的首端，FooterView作为一系列类型放在列表尾端，在上拉加载Type之前。各种类型示意图如下：

![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/loadrecy3.png)

**关于Header和Footer 本篇里面每添加一个Header或者Footer相当于添加以一个新的类型，如果要添加很多相同类型的Header或者Footer ，则需要进行修改，否则该类型的View的重用次数很少，影响RecyclerView 的性能**

#### Header
#### Footer
#### 封装完成代码


