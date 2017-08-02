本篇作为 [从0开始开发一款应用市场APP]()系列的先导篇。从本篇起,开始连载记录整个APP的开发过程。

本系列的代码同步更新到[https://github.com/ScWen7/HWAPPStore](https://github.com/ScWen7/HWAPPStore) ,可以根据Git 的记录查看相应的代码。

---------------------------------------------------------------------------------------------------

目前已经完成的：

 [先导篇]()

**(未完待续,持续更新中......)**

## App效果
![screen1](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen1.png-blog) ![screen2](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen2.png-blog)![screen3](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen3.png-blog)


![screen4](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen4.png-blog)![screen5](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen5.png-blog)![screen6](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen6.png-blog)


![screen7](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen7.png-blog) ![screen8](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/screen8.png-blog)


### 项目使用到的技术点：
1. MVP+Dagger2    
    1. MVC与MVP   
    2. MVP使用案例     
    3. Dagger2使用    
    4. Application+BaseActivity+BaseFragment 的封装
2. RxJava2 + Retrofit2 
    1. 网络请求   
    2. 数据持久化     
    3. 异常处理    
    4. 多文件下载断点续传
    5. 文件上传
3. RecyclerView 的封装
    1. 打造通用的Adapter   
    2. 多Item布局实现     
    3. 添加header和footer   
    4. 添加section分区操作
4. 自定义View
    1. 顶部轮播图的实现   
    2. 悬浮搜索框   
    3. 弹性RecyclerView和ScrollView实现
    4. 下载进度条
    5. 自动伸缩TextView
    6. SubTabNavigator
    7. 自定义应用标签（分配行原理，测量控件，分配位置）
5. GreenDao的使用
6. SVG与IconFont
    1. 使用SVG矢量图
    2. IconFont的使用 
7. LaodingPage 的使用封装
    1. 界面显示逻辑
    2. 界面显示封装
8. 6.0运行时权限处理
    1. 界面申请案例
    2. 权限申请的封装
9. 7.0FileProvider 的适配