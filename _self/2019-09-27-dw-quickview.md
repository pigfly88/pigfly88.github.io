---
layout: post
category: "mix"
title:  "数据仓库概览"
---

OLTP：操作性事务
OLAP：分析型
ETL
E：Extract（抽取），从多个数据源获取数据
T：Transform（转换），按照一定的规则转换成我们需要的格式
L：Load（加载），将转换好的数据保存到数据仓库
数据经过清洗以后再进入Hadoop可以减少垃圾数据

可以把ETL想象成饭店的后厨制作菜品的一个过程，厨师根据顾客下的单，选取不同的原材料（抽取），然后进行烹饪（转换），到最后送到顾客饭桌上（加载）。
DW/BI系统提供菜单（报表）告诉顾客什么数据可用，

维度模型建立
1. 选择业务过程
2. 确定粒度（每一行表示什么）
3. 确定维度（每一行包含哪些内容）
4. 确定事实

零食店案例：
1. 业务过程：POS交易单
2. 粒度：POS交易的单个商品
3. 维度：日期、门店、支付方式
4. 

数据抽取
我们做饭之前，先要从菜场各个摊位去买我们需要的原材料，如青菜，番茄，鸡蛋，和鱼，然后把菜上的老叶子去掉，鱼鳞和内脏去掉，洗干净。建立OLAP应用之前，我们要想办法把各个独立系统的数据抽取出来，经过一定的转换和过滤，存放到一个集中的地方，成为数据仓库。这个抽取，转换，加载的过程叫ETL（Extract， Transform，Load）.相应的开发工具Oracle有DataStage，微软有SQL Server Integration Services，Pentaho有Kettle。这些ETL工具一般都支持图形化流程建模，文本文件映射导入，XML,XSLT,可执行SQL，javascript等。


数据建模
材料准备好后，我们要规划他们可以做出什么样的菜。首先我们选择主要材料：如鱼，同样是鱼，可以有多种烧法，红烧，清蒸，油炸，水煮。不同的烧法还要搭配相应的辅助材料，如红烧一定要酱油和葱姜。想好了菜单，实际上就已经把这些原材料按不同的组合建立了一定的关系。对于OLAP应用，也要根据客户需求，我们对数据仓库中这些物理存在的表要进行逻辑建模，以某些重要的事实数据（如销售数据）为核心，建立与其他物理表（维度表）之间的业务关系。如销售数据跟部门表，客户表之间的关系。事实和维度之间的组合，就建立了将来做多维查询的基础。建模过程形成的结果在各中平台上的叫法不一样，如BO的叫Universe，Oracle中叫Cube，SqlServer2005的叫统一维度模型UDM，开源Pentaho中也叫Cube。相应的开发工具BO有Business Objects Crystal Decisions，Oracle有 Analytic WorkspaceManager ，SqlServer2005有BusinessIntelligence Development Studio，Pentaho有Schema Workbench。相对其他商业产品，Schema Workbench比较简单，也没有和软件开发平台如Eclipse集成在一起。

 

多维查询
准备好了原材料和相应的菜单，接下来就是根据要求烧菜了。这当中需要有一种表达需求的语言，例如同样是这个红烧鱼，有的人希望甜一些，有些人不喜欢放葱。如果有一个标准的语言描述这种执行要求，就能保证烧的菜符合你的口味了。同样，有了表达逻辑关系的模型Cube，数据仓库中也导入了业务数据，我们还要告诉执行引擎如何取得我们真正所要的数据。这个查询语言就是MDX(Multidimensional Expression)，它是微软在1997年首次提出，并为多家厂商采用。如果要学习它的相关语法，微软MSDN上有详细的文档：http://technet.microsoft.com/zh-cn/library/bb500184.aspx

 

数据展现
烧好了菜，还要决定如何上菜，是用碗，用盘子还是砂锅，这些都要根据所做的菜式和客户要求。MDX查询返回的是多维数据，普通的二维表很难表现超过2个维度的数据，如果要进行数据的钻取等操作更是难上加难。各厂家的技术平台都有想应的实现技术。比较底层的界面表现技术Oracle 有Business Intelligence Beans，开源的有JPivot，这些需要开发相应的展示页面和维护界面，但可以和已有的系统紧密结合。另外为了方便用户使用和维护，也有做成可运行程序的系统平台。如Oracle有Oracle Business IntelligenceFoundation，开源的有SpagoBI，Pentaho BI Platform等。这些系统都有完整的DashBoard，多维查询，报表等功能，使用维护都比较方便，缺点就是比较庞大笨重。


以上是建立OLAP应用的几个重要环节和相关技术，最后总结一下：用户需求——数据建模——数据仓库

 

用户需求决定了如何设计模型和数据仓库，数据模型又是描述数据仓库的逻辑关系，而数据模型和数据仓库的某些技术限制也可能影响用户需求的实现。这三者之间是相互依存和影响着的。而MDX查询，又是这三者之间的粘合剂，它表达了用户的需求，经过OLAP引擎的解析，根据数据模型的描述，从数据仓库找到所需要的数据。