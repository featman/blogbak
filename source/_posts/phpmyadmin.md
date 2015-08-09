title: phpmyadmin导入excel表格->mysql 
date: 2015-8-8 20:23:23
categories: 数据库技术 
tags: [excel,随笔,没有别的了]
description: 介绍phpmyadmin导入excel的方法
---

### 写在开头
帮助文档以phpMyadmin数据库建表开始，然后再到后边的用csv导入到数据库表中。写这些的目的是 因为插入的东西是中文，所以需要注意编码问题。
<!--more-->
### 用到的工具
excel，phpmyadmin，emeditor
### 过程
- phpMyadmin建表
登陆lorne.pw/database中，在新建数据库中写上数据库名字 divider，整理选择utf8_general_ci，然后点击创建。

![image description](http://img-storage.qiniudn.com/15-8-8/98049354.jpg)

- 选择进入数据库divider，然后选择新建一张表，写上名字dividerInfo，并写上字段5个，然后点击执行。

![image description](http://img-storage.qiniudn.com/15-8-8/10194909.jpg)

- 在这张新建数据表的窗口下，点击添加各字段详细信息，如下，值得注意的是中文用varchar就好，长度写上大概的就可以：

![image description](http://img-storage.qiniudn.com/15-8-8/61706716.jpg)

- 在同一个窗口下，注意选择整理：然后选择utf8_general_ci，然后点击保存表就建好了。

![image description](http://img-storage.qiniudn.com/15-8-8/5406617.jpg)

- 新建excel表格，编辑学生信息，如下所示：
（注意列的顺序与表要严格一致，第一行不需要些字段名称）

![image description](http://img-storage.qiniudn.com/15-8-8/20837690.jpg)

- 编辑完点击文件-另存为-计算机，然后弹出对话框，点击对话框的工具-web选项-编码。


![image description](http://img-storage.qiniudn.com/15-8-8/87494927.jpg)

- 选择将此文档另存为uniconde(UTF-8),然后点击确定，此时进入第10步。


![image description](http://img-storage.qiniudn.com/15-8-8/5380601.jpg)

- 在下图中选择保存为csv格式。

![image description](http://img-storage.qiniudn.com/15-8-8/54058994.jpg)


- 此时桌面出现csv格式后，这时就可以进入12步，把csv上传到phpMyadmin中。
在phpMyadmin主界面，选择数据库divider，然后选择表dividerInfo，最后选择导入。

![image description](http://img-storage.qiniudn.com/15-8-8/91318425.jpg)

- 选择从计算机中上传student.csv文件。其他不用管，配置如图所示。

 ![image description](http://img-storage.qiniudn.com/15-8-8/44047196.jpg)

- 此时还需要一步，就是在桌面生成的student.csv文件，右键用windows自带的记事本打

 ![image description](http://img-storage.qiniudn.com/15-8-8/60755242.jpg)
 
*** *附**
 ![image description](http://img-storage.qiniudn.com/15-8-8/28900338.jpg)
 
** 其实有一种更为简单的方法，就是直接在桌面上右键新建一个txt，然后用自带的记事本编辑，
比如编辑：
11,胡威,99,三中,高一5班
12,胡方法,10,二中,高一10班


这个方法需要注意的事项是每个字段与每个字段之间的逗号必须要用英文的逗号。
编辑结束后，点击文件另存为，选择编码为utf-8，保存为csv格式。这种方法不适合批量输入，因为不方便输入逗号
**



