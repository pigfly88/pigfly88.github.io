## 个人信息

- 朱程辉 | 男 | 1988 | 深圳市宝安区沙井街道
- 北京信息科技大学(2007/9~2011/7) | 本科 | 电子商务 | 英语四级
- **Tel: 137-1414-8963**&nbsp;&nbsp;&nbsp;&nbsp;Email: pigfly1988@gmail.com

## 个人能力

十年以上后端开发经验，做过电商、游戏、大数据分析。对高并发、高可用架构有深入理解，有微服务实战经验，熟悉docker、k8s。

## 工作经验

#### 光汇石油 2020/6~2023/1
负责加油 app 的后端开发。使用到的后端语言有：php、java、golang。主要工作内容：

- 负责中台管理系统的开发与维护，基于 Yii + Bootstrap。
- 负责聚合层接口开发，基于 swoole。
- 负责服务端接口开发，基于 SpringCloud 微服务 + Gin。
	
#### 传音控股 2019/7~2020/6
负责数据仓库项目，整合公司各个业务系统的数据，清洗并加载到数据仓库，结合BI工具实现各个维度的数据统计分析和可视化。基于 php + ClickHouse，单表最大十亿。主要工作内容：

- 多进程数据同步。
- 数据迁移和同步，将 MySQL、Oracle、HANA 等各种异构数据库的数据迁移到 ClickHouse，并支持增量同步。
- 集成 BI 工具 Tableau。
	
#### 球友科技 2016/7~2019/2
负责开发的产品是一个专业的球赛资讯、社交和竞猜平台。主要工作内容：

- 接口性能调优。通过调整缓存策略、优化慢查询和耗时任务异步执行，qps 提升100倍。
- 接口防刷机制。在接口中加入了 Token 验证，并基于多维度(IP,UA,UID,UUID等)做频率限制，在关键操作上提高验证码复杂度，逐渐把攻击损失减少和控制下来。
- 在开发阶段，通过数据工厂和 MockServer，缩短了联调和测试的时间。

#### 博雅互动 2014/7~2016/2
负责开发的产品是公司最热门的一款棋牌类游戏-德州扑克，DAU 580万。主要工作内容：

- 利用 Redis 实现签到、排行榜、异步任务队列（牌局统计、发奖励等等）。
- 随着业务快速发展，数据库变得越来越大，qps 10k+，单台MySQL已经不足以支撑下去，通过分库分表提升查询速度，后期也参与了海量玩家数据从 MySQL 迁移到 MongoDB。
- 开发运营活动模块，几百个活动经常需要来回切换或者变更配置，之前是人工改代码的方式，效率太低效，改为在管理后台统一进行管理。
- 编写 socket 客户端，与 C++ 服务通讯。
- 监控告警系统：监控守护进程，php 错误告警、业务异常告警。

#### OPPO 2013/7~2014/5
公司主要经营手机及周边产品。负责官方商城的后台开发，主要工作内容：

- 数据库优化。对大表做拆分，调整 MySQL 配置使得 qps 从2k提升到9k。
- 优化 Redis 缓存策略，提高缓存命中率。
- 开发配置化活动模块，让运营在后台可以通过简单几步配置就能实现直降、礼包等活动。
- 开发管理后台通用模块，管理后台大部分模块的功能大同小异，采用继承和自定义配置，实现快速开发 crud 后台页面。

#### XDA 2011/8~2013/5
负责手机资讯网站的后端开发。主要工作内容：

- 对于首页的热点数据，利用 Memcached 和 Redis 做前置缓存，降低 MySQL 负载。
- 随着访问量越来越大，页面加载速度越来越慢的情况下，独立开发了 html 静态化模块，通过压缩与合并静态资源，搭配 CDN，使得页面加载时间从几秒缩短到几百毫秒。
- Discuz 论坛维护。
	