---
layout: post
title: 使用python和redis实现实时聊天室
category: tech
tag: 技术
---

实时的实现一般有ajax long poll / iframe stream / websocket 这三种。websocket有浏览器的限制;iframe会使页面一直处于loading状态(如果没有这个限制，iframe stream相比long poll还是比较有优势的); ajax long poll因为它的跨浏览器特性，加上实现比较简单，所以不少的实时应用都选择了long poll, 这里也选择ajax long poll来实现

放个截图先：

<img src="/image/comet_chat.png" />

主要工具：python (flask / gevent / apscheduler) + redis

目前这个聊天室的功能还比较简单，可以实时更新全局在线用户，当前聊天室在线用户和聊天内容，都是通过一个ajax长连接实现的。在这个连接里定义了需要实时的内容，相应的内容一旦在服务端更新，可以马上推送到各个客户端。

实时最大的挑战在于如何能在服务端尽量简单，灵活地实现。因为越是简单，就越不容易滋生bug，即使出现问题，查找起来也很方便;越是灵活，就越容易应付多变的需求。以聊天室为例，当新用户加入后需要更新所有客户端的当前成员列表;有成员退出后也要更新该列表。每个聊天室有了新内容后都要实时推到所有客户端;给某个用户发私信，对方也可以实时收到。这些如果处理地不好，很容易与当前的逻辑发生耦合，为将来的调整和bug的查找带来困难。

依靠强大的redis，设计了这个简易的实时聊天室，基本架构如下:

<img src="/image/comet_arch.png" />

匆忙完成了初步的代码，还有不少需要优化的地方，有些功能也还没加上（如私信的实时通知）。接下来的一段时间内会逐步完善，如果有更好的idea，欢迎沟通 :)

代码地址：<a href="https://github.com/lzyy/chat">https://github.com/lzyy/chat</a>

如果使用过程中遇到什么问题，或有好的建议可以在<a href="https://github.com/lzyy/chat/issues">这里</a>反馈

### version 0.2

* 调整了实时的策略，抛弃pubsub模式，使用zset+timestamp。同时修正了在聊天内容频繁发送时会丢部分信息的bug
* 添加了管理员角色（目前只能用于删除聊天室）
* 修正重复名的bug（无法完全避免，只能在登录时验证）

### version 0.1.1

* 调整了添加聊天室的体验
* 更快的聊天内容显示速度
* 可以删除自己创建的聊天室
* bugfixes

### version 0.1

#### intro

某天晚上打球回来，到家已将近11点，洗完澡，代码敲到2点左右倒下，天还没亮就被猫叫醒，醒来后就一直想如何改进这个聊天室，觉得能在一个页面实时看到每个聊天室的内容，还是很酷的，于是就按着这个想法实现了。

PS: 如果不是全新安装，需要执行 bin/python scripts/clear_key.py 来清除redis cache

#### changelog

* 可以自定义昵称
* 调整了聊天页的内容和样式
* 添加了home页（显示所有的聊天室，并实时更新每个聊天室的内容）
* 使用coffee-script重写了js部分
* bugfixes

#### screenshot

<img src="/image/comet_home_0.1.png" />


<img src="/image/comet_room_0.1.gif" />
