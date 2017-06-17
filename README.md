# Gomoku
this is a Gomoku game's client and server, which bulid by canvas and swoole

这是一个使用HTML5 Canvas构建客户端，PHP Swoole构建服务端，使用服务端与客户端使用Websocket协议通信的五子棋小游戏
客户端依赖Amaze UI用于为游戏主界面提供具有响应式布局的栅格布局，DOM操作全部使用原生JS完成

服务端部分使用了Swoole扩展实现Websocket服务端的构建，存储部分使用的是PDO扩展与MySQL进行数据库交互。
数据库有4个表：
- users表引擎为MEMORY，用于存储当前已连接服务端的用户uid与状态（该uid用于网络对战时标识用户）
- users_log表引擎为InnoDB，用于持久化存储客户端与服务端的连接记录，方便进行各种统计信息。（使用触发器与users表同步）
- rooms表引擎为MEMORY，用于记录当前正在游戏中的房间，该表中维护两个user实体之间的关系，并且记录最后行棋者，步数（着数）等信息用于防止某个用户重复行棋。
- records表引擎为MEMORY，用于记录棋谱状态，防止用户在同一个坐标重复行棋，以及进行判赢算法（五子棋中判断棋盘上出现活五则该玩家获胜）统计赢家。如果需要持久化存储棋谱数据，用户可自行建表，然后使用触发器保持该表与棋谱表的同步。
注意：服务端Swoole扩展需要额外编译安装


**服务端在Ubuntu 14 + PHP 5.6 + Swoole 1.8.6 + MySQL 5.5下测试通过**
**客户端在IE Edge，Chrome，Firefox等支持HTML5的主流浏览器下均可正常使用。**

# 使用方法
- 服务器正确安装PHP 5.5以及以上版本，然后使用`pecl install swoole`或者手动编译的方式安装Swoole 1.8.x以及更高版本（低版本的Swoole扩展某些回调函数参数与新版本不同，将会导致程序运行错误）
- 服务器正确安装MySQL 5.x及以上版本，修改`server.php`第13-15行有关数据库连接信息的常量配置，然后正确创建数据库，字符集为UTF-8，并且导入gobang.sql。
- 服务器防火墙开放相应IP及端口（默认监听地址与端口为0.0.0.0:1997，表示在所有IP上监听1997端口，可在`server.php`的17-18行修改），如果为家庭网络，请正确配置端口转发，如果没有外网IP请向运营商索取。
- 客户端修改`./js/welcome.js`中第22行的Websocket URI为对应服务端的IP和端口
- 打开客户端浏览器，点击开始游戏，如果稍等片刻后左上角的Uid部分能正确显示服务端分发给客户端的uid，并且左侧的玩家列表能够正确显示服务端上所有未在游戏中的其他客户端uid，则正常。否则等待片刻后浏览器将会弹出“Websocket连接错误”的提示框，请检查上述各项设置是否正常，并且观察服务端上是否有报错信息，如果有，欢迎您将报错信息以Email的形式发送到867597730@qq.com，如果您对PHP以及本项目开发感兴趣，欢迎加群374107842。

# 未完成功能
- 读秒功能还未开发，但是数据库中已经预留好相关字段，如果要做到较为安全的反作弊，那么务必需要保证服务端和客户端每秒通信一次，高并发下需要处理好性能问题。
- 流量攻击方面还未做好防御，目前可以考虑在硬件层面进行过滤。
- 判赢算法使用的是传统的DFS搜索，还有优化空间，比如根据落子点向四个方向进行活四活五判定，如果需要更高效率可以考虑使用PHP７或者改用扩展实现。

# 关于本项目
本项目为作者学习Canvas和Swoole后的一个练手项目，共花三天时间左右写完。其中服务端客户端代码都有许多不规范之处，例如客户端未进行模块划分，未使用前端框架进行数据与视图的绑定，Websocket通信和Canvas操作等代码未使用OOP思想进行封装，耦合过重。在规范的项目中，这些方法函数应该进行封装，部分交互操作应该抽象出接口，以备后续扩展（比如说可以扩展棋盘纵横线数量成围棋）。服务端由于Swoole本身的回调式写法，以及编写匆忙，因此只有单文件，许多地方优化有限，还请谅解。

## 项目名称：在线对战五子棋（http:// gomoku.changwei.me）
### 技术栈：PHP，MySQL，Apache，Swoole 1.7.3，HTML5 Canvas，Websocket。运行环境：腾讯云1核1G内存，CentOS 6.5
### 开发规模：1人，3天
### 开发时间：2017年1月
## 项目简介：
该项目可以实现两个可访问外网的电脑在支持HTML5新特性浏览器的情况下进行双人五子棋对战。
## 项目优点：
无需安装任何客户端，纯HTML5实现，效率高，用户体验良好，手机平板等触控设备也可以获得良好的操作体验。
## 项目缺陷：
1.	低判赢算法使用的是传统的DFS搜索效率，可改进为根据落子点向四个方向进行活四活五判定，且PHP本身的低性能更加为该缺陷雪上加霜，导致在单个对局中棋子过多的情况下，用户落子操作会出现明显的卡顿延迟。
2.	Websocket服务端未使用心跳包会导致两个问题，第一是标准五子棋对局的读秒功能无法实现，第二是可能会出现用户思考落子时间过长时，客户机到服务端链路中的路由，交换设备根据自己的TTL以及相关策略主动断开Websocket链接导致对局终止，影响用户体验。
3.	无断线重连机制，如果用户在信号不佳的移动网络环境中使用，可能会导致无法正常对局。
## 项目总结：
还需在业余的课程学习中自己安排一下算法方面的学习，对后续开发以及个人提升都会很有帮助。涉及到移动端的长连接服务端开发，需要更多的模拟现实中的网络环境，发现一些潜在问题，针对网络环境不佳的情况进行各种容错处理以及断线重连机制。
