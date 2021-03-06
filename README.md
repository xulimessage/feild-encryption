#运行本项目之前，需要配置系统环境变量DB_PBE_PWD的值

#背景：
近几年互联网用户数据常有发生，如12306网站用户数据泄漏事件，导致用户的信息变得极为不安全，为保障用户信息安全，决定对用户核心数据进行加密

#数据泄露方式：
外部，黑客入侵数据库
内部，工作人员数据导出（开发、OPM、运维）

#数据安全等级评估：
建议只对可以联系到用户的数据才对其加密，如电话、邮箱等；以下是用户数据安全等级评估：
等级	字段	处理方式	 	 
一级	姓名、手机号、微信号、QQ、邮箱	
1.入库数据为加密数据，只能通过软件方可显示明文
2.在OPM管理平台以全密或半密展示
3.明文excel数据需要邮件审批后，交由运维解密后返还明文excel数据
 	 
二级	身份证号、银行卡号	半密展示	 	 
三级	资产金额、交易金额	明文或全**展示	 	 
三种展示方式： 
•原文，即不改变数据库取出的内容将其展示 
•半密，即模糊掉中间部分，保留展示首尾，如手机号13801258088，展示为138*****8088 
•全*，即隐藏展示，以占位符*替换，一个*表示一个字符或字节
 	 

 	 
#技术	
【支撑条件】
1.统一用户信息获取来源，解密方式统一；使用redis存储用户数据，从redis读取用户数据时，使用插件解密用户数
2.一级数据加解密使用需要培训，以开发规范要求技术开发严格遵守
【加解密技术选型】
考虑到数据加解密只在公司内部使用，且综合使用性能，选择对称加解密算法为PBE 
PBE——Password-based encryption（基于密码加密）。其特点在于口令由用户自己掌管，不借助任何物理媒体；采用随机数（这里我们叫做盐）杂凑多重加密等方法保证数据的安全性。是一种简便的加密方式。 
1.其中口令密码由运维管理，可以将其配置在环境变量中，秘钥key为DB_PBE_PWD
2.盐取当前明文md5后中的8位字符转出的byte数组
3.加密的excel需审批后发送附件，由运维执行java工具对excel中数据进行解密
 	 
 	 
#注意：
1.上线时，因为用户中心改造，所以网站所有服务需要停止服务
2.数据库数据加密初始化，需借助task调用程序加密数据库当前明文
3.上线后需要在技术部开会分享用户数据安全，务必贯彻执行（数据加解密使用、OPM可以展示用户哪些数据）
 	 
 
#其他：
加密长度调查	
原内容长度1~7，加密后长度为20
原内容长度8~15，加密后长度为32
原内容长度16~23，加密后长度为40
原内容长度24~31，加密后长度为52
原内容长度32~39，加密后长度为64
原内容长度40~47，加密后长度为72
原内容长度48~55，加密后长度为84
……
与秘钥key的长度无关

对功能的增删改查影响	
1.查询不支持模糊查询
2.加密的字段明文长度不得超过60
 
加解密效率	
解密：
1.5W数据量excel解密耗时约28秒
2.10W数据量excel解密耗时约60秒
压力测试：
1秒并发100情况下，新增修改加密入库一次不超过0.5秒
 
#测试
1.resources目录下有表结构脚本，请先执行
2.访问http://localhost:8080/可以看到swagger提供的测试页面
3.test目录下的单元测试分为对数据库造数据、excel的解密测试