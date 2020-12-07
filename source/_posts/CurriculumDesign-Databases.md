---
title: 数据库课程设计——火车票售票系统
date: 2020-04-26 14:52:34
categories: 
- 课程设计
tags:
- 数据库
- vue
- springboot
---
数据库课程设计的题目，设计了一个火车票售票系统，实现了列车信息查询，车票查询及购买，订单查询，个人信息管理等功能，数据是从12306爬取的真实数据。  
<!-- more -->
## 项目链接
前端项目链接：
https://github.com/ccclll777/db_design_web

后端项目链接：
https://github.com/ccclll777/db_design_service

**如果觉得有帮助，请点个star吧**
## 题目简述
**火车票售票系统**
 - 车次管理（车次，起止地点，到达时间，开车时间）
 - 坐席管理（车厢号，座位号）
 - 售票（直达，换乘）改签，退票
 - 余票查询
 - 订单查询
 - 用户管理
## 开发环境与技术
 - **开发工具**
工具：WebStorm ,IntelliJ IDEA
Mysql8.0.12   Tomcat 8.0   Maven  Git  Redis
 - **前端技术栈**
HTML+CSS+Javascript
前端框架:Vue.js+ElementUi
 - **后端技术栈**
Spring Boot为框架
MyBatis操作数据库
Redis进行用户信息缓存
Maven作为项目构建工具
## 需求分析


**1.用户登录注册**
用户注册登陆后可以使用系统的所有功能，如添加乘客，购买车票，查询订单等等

**2.系统需要提供基础的列车信息查询：**
根据车次查询列车是否正常运行，以及查看列车的基本信息（如列车类型，始发站，终点站，开车时间，到达时间，运行时间，车厢数等等）
根据车次，查询列车经停站信息（包含这趟列车每一个经停车站的车站名，到达时间，开车时间，运行时间等信息）

**3.系统需要提供根据车次查询列车详细信息的功能：**
根据始发站和终点站，查询可以满足自己行程要求并且正常运行的列车（可以根据开车时或者运行时间进行排序），并且可以进一步查看中间经过的的车站信息，以及开车时间，到达时间等。
系统需要提供接续换乘一次的查询，根据输入的出发站和终点站，可以查询换乘一次满足条件的列车，并且可以根据开车时间或总运行时间进行排序。

**4.车票购买**
在查询到符合自己出行条件的列车后，可以查询列车的剩余座位以及购买车票。
首先添加乘客（添加需要购买车票的乘客）——>进行座位选择（为每一位乘客选择座位)——>订单支付——>购票成功
接续换乘车票购买流程类似，只不过在选座时，需要选择两趟列车的座位。
**5.系统需要提供用户的个人信息修改功能以及修改密码功能。**

**6.用户可以给除自己以外别的乘客购买车票，所以提供添加乘客的功能，每个用户下都可以添加多个乘客，从而为别的乘客购买车票。**

**7.系统提供订单的查询功能，可以查询到与自己有关的所有订单，比如所有订单，未支付订单，未出行订单。**

**8.未支付订单针对下单但是没有支付的订单，可以在规定的时间内进行支付操作，如果在规定时间内没有完成操作，则订单会作废，变成未完成支付的订单。**

**9.未出行订单针对已经支付但是没有出行的订单，可以在未出行订单中查看自己的出行计划。未出行订单可以进行改签操作，改签相同出发站和终点站的其他列车。**

**10.未出行订单还可以进行退票操作，从而取消订单。**

## 数据库概念设计
本系统中一共6个实体集，分别是，用户实体，乘客实体，列车信息实体，列车经停信息实体，订单实体，列车座位信息实体

**（1）用户实体：**
保存注册系统的用户的信息，主码为用户电话号码，用来作为每一个用户的唯一标记，同时电话号码也作为登录系统的用户名来使用。其中存在用户自定义的完整性约束：用户类型（0为学生，1为成人，2为管理员），性别（0为女性，1为男性）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426225834623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（2）乘客实体：**
每个用户下可以添加多个乘客信息，然后为多个乘客购票。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426225849883.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**(3)列车信息实体：**
列车信息存储了列车整体信息，每一趟列车都有一条自己的总的列车信息，表示这趟列车是正常运行，或者是停开的。列车编号作为列车的主码（由于列车编号相同的列车，由于开车方向的不同，在途中可能改变车次，但是列车编号是固定不变的）![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426225900397.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（4）列车经停信息实体：**
每个列车都有经停信息表，存储了列车停靠的不同车站，以车站编号排序。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426225909759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（5）列车座位信息表，为了简化系统设计的难度，统一固定车次为D,G开头的列车设置特等座，一等座和二等座三个座位类型，其他类型的车设置软卧，硬卧，硬座三个座位类型。并且如果座位类型相同，则车厢的座位数以及布局是相同的。比如：特等座一节车厢18座，一排2座，一等座一节车厢52座，一排4座，二等座一节车厢85座，一排5座。软卧一节车厢36个位置，一列2个位置，分为上下铺。硬卧一节车厢66个位置，一列3个位置，分为上中下铺。硬座一节车厢120个位置，一排6个座位。
列车座位信息的主码为列车编号，车厢编号。外码为列车编号，参照列车信息实体。
当车厢号固定时，说明座位类型已经固定，具体的位置是根据座位号对于相应一排的座位数求模运算得到的，比如特等座5号为5/2 +1 =3  3排
5%2 = 1 A座 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426225924719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（6）订单信息实体：**
存储了系统中所有的订单信息，订单信息实体的主码为订单编号，外码为用户电话号码（用户实体主码），乘客身份证号码（乘客实体主码），列车编号（列车信息主码）.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426225936769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**整体E-R图为：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230014717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230019119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**说明：**
（1）每个用户可以添加多个乘客
（2）每个用户可以拥有多个订单
（3）每个订单属于一个乘客
（4）每个订单拥有一趟列车的信息
（5）每个订单拥有一个列车的座位信息
（6）每个订单拥有两个列车经停的车站信息
（7）每个列车有多个经停的车站
（8）每个列车拥有多个车厢的座位信息

## E-R图向关系模式的转化
（1）实体转化为关系模式
用户（电话号码，密码，身份证号，邮箱，真实姓名，用户类型，性别，地址）
乘客（乘客身份证号，乘客真实姓名，乘客电话号码，乘客类型，地址）
列车信息（列车编号，车次，列车类型，列车车厢数，列车始发站，列车终点站，列车开车时间，列车到达时间，列车到达日期，列车运行时间，列车状态）
列车座位信息（编号，车厢号，座位类型，座位数）
列车经停信息（车次，车展编号，车站名，到达时间，总运行时间，开车时间）
订单信息（订单编号，出发站编号，到达站编号，车厢号，座位编号，订单创建时间，订单状态，开车时间）
（2）联系转化为关系模式
用户拥有乘客（用户电话号码，乘客身份证号码）
用户拥有订单（用户电话号码，订单编号）
乘客拥有订单（乘客身份证号码，订单编号）
订单拥有列车（订单编号，列车编号）
订单拥有座位（订单编号，车厢号，座位号）
列车信息拥有列车经停站信息（列车编号，车次，车站编号）
列车拥有座位（列车编号，车厢号）
（3）实体与联系进行合并，E-R图中的一对一与一对多的联系，通过参照完整性与关系模式进行合并，并进行一定修改，合并。
用户（电话号码，密码，身份证号，邮箱，真实姓名，用户类型，性别，地址）
乘客（用户电话号码，乘客身份证号，乘客真实姓名，乘客电话号码，乘客类型，地址）
列车信息（列车编号，车次，列车类型，列车车厢数，列车始发站，列车终点站，列车开车时间，列车到达时间，列车到达日期，列车运行时间，列车状态）
列车座位信息（列车编号，车厢号，座位类型，座位数）
列车经停信息（列车编号，车次，车站编号，车站名，到达时间，总运行时间，开车时间）
订单信息（订单编号，用户电话号码，乘客身份证号码，列车编号，出发站编号，到达站编号，车厢号，座位编号，订单创建时间，订单状态，开车时间）
## 数据库逻辑设计
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230314346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230405802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230416251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/202004262304254.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230434131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230657148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230440827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230708617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230450657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
## 数据库物理结构设计
**(1)用户表：**
在用户表中，由于查询以及更新的条件都是用户电话号码，所以将用户电话设置为主码，会相应的建立索引，提高查询的效率。
**(2)乘客表：**
主码为用户电话号码以及乘客身份证号码，由于有许多查询以这两个为条件，所以设置成主码增加查询效率。虽然在更新时比较耗时，但是更新的频率比查询的频率少很多。
**(3)列车信息表：**
以列车编号为主码，会自动建立索引，提高查询效率。
**(4)列车经停信息表：**
这个表的查询量较大，但插入，删除，更新的操作较小，所以在这个表上建立一定的索引时非常合算的。
由于需要根据车次以及列车编号查询列车经停信息，所以可以在车次与列车编号上建立普通索引即可，提高查询效率。
在根据出发站以及到达站进行对符合条件的列车进行检索时，需要用到车站名称，所以应该在车站名称上建立普通索引
**（5）列车座位表：**
列车座位表的主码是列车编号以及车厢号，会自动建立索引，查询时也会以这两个为条件，所以不用在增加其他索引。
**（6）订单表：**
由于订单表的插入频率以及查询频率都很高，而建立索引会降低插入的效率，而不建立索引会降低查询的效率，所以需要在两者之间进行取舍。
由于订单表的主码为订单编号，会自动建立唯一索引。
用户电话号码，和乘客身份证号码作为外码，常常成为查询的条件，所以应该在外码上建立普通索引。
在查询剩余车票信息时，需要先在订单列表中查询某辆车在某个时间的那一段路程已经被订购过。查询时的条件需要有出发车站编号，到达车站编号，列车编号，订单状态，开车日期	。
所以可以考虑在出发车站编号，到达车站编号，列车编号，订单状态，开车日期上建立普通索引。提高查询效率。毕竟查询的次数要远远多余订票的次数，所以查询的次数会多余插入的次数，用空间来换取更快的效率，还是可以接受的。

## 项目整体结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230742935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
说明：
**（1）用户首先进行账号的注册，然后进行登录操作**
      登录之后进入主页，然后有不同的板块进行选择
**（2）列车信息查询板块：**
列车信息：可以查询所有正常运行的列车信息
列车时刻表：可以根据车次查询列车的经停信息，
列车查询：可以根据出发站和到达站查询符合条件的列车
接续换乘：可以根据出发站和到达站查询换乘一次能到达的列车
**（3）车票查询及购买板块：**
    余票查询：可以根据出发站和到达站查询符合条件的列车并且查看余票以及订购车票（跳转到车票购买界面）
车票购买：添加乘客后，可以为乘客选座，然后支付，购票成功。
接续换乘：可以根据出发站和到达站查询换乘一次能到达的列车，然后进行购票，跳转到接续换乘车票购买界面。
接续换乘车票购票：添加乘客后，为乘客进行选座操作，然后支付，购票成功
**（4）订单信息板块：**
     全部订单：查询用户名下的全部订单状态
     未支付订单：查询用户名下全部未支付的订单，然后可以进行订单的支付操作
     未出行订单：可以查询到已经支付，但是还未出行的订单，可以进行退票以及改签操作。
     订单改签：进行重新的选座已经支付操作。
**（5）个人信息板块：**
      查看个人信息：查看用户的个人信息。
修改个人信息：修改用户的个人信息
修改密码：修改登录密码
查看乘客信息：查看此用户名下的所有乘客。
添加乘客：为此用户名下添加乘客，之后可以为该乘客购票

## 界面截图
**（1）列车信息查询界面**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042623092077.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（2）列车时刻表查询
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426230948887.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（3）列车查询**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231041586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231044852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（4）接续换乘查询**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231101869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231107309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

（5）余票查询以及购买
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231136480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231142386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231146475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231151186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（6）订单列表**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231208778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
## 关键sql语句设计
**（1）根据车次查询列车所有经停站信息**

```sql
select   b.station_no as 车站编号 ,b.station_name as  车站名, 
           b.train_number as 车次 , b.start_time  as 开车时间, 
           b.arrive_time as 到达时间 , b.running_time  as 历时 
                 from train_parking_station（列车经停站信息表） as a ,train_parking_station  as b 
                         where a.train_number = '车次'   and a.train_no = b.train_no 
                                   order by b.station_no
```

**（2）根据起始站，目的站查询符合条件的列车**

```sql
select C.train_no as 列车编号 ,C.train_number as 车次 ,
         C.station_name as 起始站 ,D.station_name as 目的站 ,             
         C.station_no as 起始站编号 , D.station_no as  目的站编号  ,
         C.start_time as 开车时间 , D.arrive_time as 到达时间,             
         C.running_time as 到起始站已运行时间 ,D.running_time as 到目的站已运行时间             
         from train_parking_station(列车经停站信息表) as C ,train_parking_station as D 
         where C.train_no = D.train_no 
         and C.station_name ='济南西'
         and D.station_name = '上海虹桥'     
         and C.station_no <  D.station_no        
         and C.train_no in
         (select train_no 
                   from   train_info        
                 where  train_running_type = '正在运行')
```

**（3）根据起始站，目的站查询符合条件的列车**

```sql
select C.train_no as 列车编号 ,C.train_number as 车次 ,
         C.station_name as 起始站 ,D.station_name as 目的站 ,             
         C.station_no as 起始站编号 , D.station_no as  目的站编号  ,
         C.start_time as 开车时间 , D.arrive_time as 到达时间,             
         C.running_time as 到起始站已运行时间 ,D.running_time as 到目的站已运行时间             
         from train_parking_station(列车经停站信息表) as C ,train_parking_station as D 
         where C.train_no = D.train_no 
         and C.station_name ='济南西'
         and D.station_name = '上海虹桥'     
         and C.station_no <  D.station_no        
         and C.train_no in
         (select train_no 
                   from   train_info        
                 where  train_running_type = '正在运行')
```
**（4）根据起始站，目的站查询换乘的列车**

```sql
select A.train_no as 列车-1编号 ,A.train_number as 列车-1车次, D.train_no as 列车-2编号 ,
         D.train_number as 列车-2车次, A.station_no as 起始站编号,
         A.station_name as 起始站名称, B.station_no as 列车-1换乘站编号 , 
         B.station_name as 列车-1换乘站名称 ,C.station_no as 列车-2换乘站编号,
         D.station_no as 目的站编号,  D.station_name as 目的站名称,
         A.start_time as 列车-1发车时间 , B.arrive_time as 列车-1到达时间, 
         C.start_time as 列车-2发车时间 ,D.arrive_time as 列车-2到达时间 
         from  train_parking_station as A , train_parking_station as B , 
                  train_parking_station as C ,train_parking_station as D 
                  where A.station_name = '哈尔滨' and D.station_name = '广州' 
                           and B.station_name = C.station_name 
                           and   A.train_no = B.train_no  and C.train_no = D.train_no and B.train_no <> C.train_no   
                           and B.arrive_time < C.arrive_time  
                           and  A.station_no <B.station_no and C.station_no<D.station_no
                           and A.train_no  in (select train_no from train_info where train_running_type = '正在运行')
                           and C.train_no in  (select train_no from train_info where train_running_type = '正在运行')
```
**（5）根据起始站编号，目的站编号，列车编号查询列车途中经停车站**

```sql
select A.train_no as 列车编号, A.train_number as 车次 ,
         A.station_name as 起始站 ,B.station_name as 目的站, 
         A.station_no as 经停站-1编号 , B.station_no as 经停站-2编号 ,
         A.start_time as 开车时间 , B.arrive_time as 到达时间,
         A.running_time as 到经停站-1已运行时间 ,B.running_time as 到经停站-2已运行时间 
         from train_parking_station as A ,train_parking_station as B 
         where A.station_no between '起始站编号' and '目的站编号'
         and  B.station_no between '起始站编号' and '目的站编号'
         and A.train_no = '列车编号'                
         and A.train_no = B.train_no 
         and B.station_no = A.station_no +1 order by A.station_no ,B.station_no 
```
（6）根据起始站编号，目的站编号，列车编号,日期查询在这一段行程中已经被占用的座位
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426231637520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```sql
select            train_no, carriage_no , seat_no             from order_list     
     where  start_station_no < '起始站编号' and end_station_no > '目的站编号' 
               and train_no = '列车编号' and left(train_start_date,10) = '日期' 
               and order_status <> '已退票' and order_status <> '已改签' and order_status <> '未完成支付'

union select      train_no,carriage_no , seat_no             from order_list 
    where  start_station_no < '目的站编号'  and end_station_no > '目的站编号'                                                                                     and  train_no = '列车编号' and left(train_start_date,10) = '日期' 
              and order_status <> '已退票'  and order_status <> '已改签' and order_status <> '未完成支付'
    
union select      train_no,carriage_no , seat_no              from order_list                                            
      where start_station_no > '起始站编号' and  end_station_no <'目的站编号'   
                and order_status <> '已退票' and order_status <> '已改签' and order_status <> '未完成支付' 
                and train_no = '列车编号' and left(train_start_date,10) = '日期'
 
union select     train_no,carriage_no , seat_no             from order_list                                                  
            where start_station_no = '起始站编号' and  end_station_no = '目的站编号'   
                     and train_no = '列车编号' and left(train_start_date,10) = '日期'
                    and order_status <> '已退票' and order_status <> '已改签'  and order_status <> '未完成支付'
```

## 参考文献
【1】Database Systems Concepts (第六版) ，Abraham， Silberschatz等著，机械工业出版社，2013
【2】数据库系统概论（第五版） ，王珊、萨师煊 ，高等教育出版社，2014
【3】Spring Boot开发实战 陈光剑编著
【4】Spring Boot+Vue全栈开发实战  王松著
【5】vue.js2  尤雨溪著