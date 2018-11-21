# 吃货大作战
 
* ### 周边业务
   * 活动页
      * 规则，banner，弹幕（初始化机器人10个），今日/明日商品列表
   * 我的答题页
      * 列表（进行中->Self，已失效->MQ/ALL,已抢完->MQ/All,未领取->Other,已领取->Self）
      * 答题详情页
      * 领奖页

   * 朋友圈图，购买（要放redis做弹幕），拉取订单后置答题表状态为已领取
   * 判断小组件
	     * 商品状态
	       * 发起：hp（正常显示有库存，没删除）pdd：没下架
	       * 帮答：hp（没下架，结束时间在24内）pdd：没下架 
	         * 异常：更新所有团为已经抢完
	     * 选题（单个人不能重复，即 n次-n道）
   * MQ
      * 每次发起成功后：24h延迟答题限期消息，可以触发（进行中/未领取-->已失效/已抢完）
      * 12题全部答完后：24h延迟领奖限期消息，可以触发（未领取-->已失效/已抢完)   

             
* ###核心业务
   * 发起人答题 
     * 发起---判断(是否发起过&&商品状态)---选题
     * 提交---判断(商品状态)---DB+Redis（个人对商品发起状态）+MQ（答题限期）
   * 参与答题 
     * 参与---判断(团状态&&商品状态&&是否帮助答题过)---选题
     * 提交--判断(商品状态，团状态不管了，即使答题期间答满人也无所谓)--DB+Redis（个人今日帮答次数，互帮情况）---判断(最后一个)---开奖(set 待领取+MQ（领奖限期）)

* ###答题表
 ```sql
 CREATE TABLE `answer` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `ctime` datetime NOT NULL COMMENT '创建时间',
  `mtime` datetime NOT NULL COMMENT '最后修改时间',
  `is_owner` int(2) NOT NULL DEFAULT 0 COMMENT '是否发起人，0:不是，1:是',
  `user_id` bigint(20) NOT NULL COMMENT '答题者ID',
  `avatar` varchar(255) DEFAULT NULL COMMENT '答题者头像',
  `nick_name` varchar(255) DEFAULT NULL COMMENT '答题者昵称',
   `parent_id` bigint(20) DEFAULT NULL COMMENT '发起答题记录Id，帮答者非空',
  `res_id` bigint(20) NOT NULL COMMENT '资源位表ID',
  `product_id` bigint(20) NOT NULL COMMENT '商品id',
  `product_name` varchar(255) NOT NULL COMMENT '商品名称',
  `img` varchar(255) DEFAULT '' COMMENT '商品主图',
  `price` decimal(16,2) NOT NULL COMMENT '商品价格',
  `correct` int(2) NOT NULL DEFAULT 0 COMMENT '是否答对，0:否；1:是',
  `status` int(2) NOT NULL DEFAULT 0 COMMENT '状态 0:进行中，1:失效，2:已抢完  3:待领取，4:已领取',
  PRIMARY KEY (`id`),
  KEY `idx_user_res_id` (`user_id`,`res_id`),
  KEY `idx_res_id` (`res_id`),
  KEY `idx_user_id` (`user_id`)
  KEY `idx_is_owner` (`is_owner `)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT=' 答题表';
```   

* ###设计思路
   * 考虑本次答题最后一次答题的竞态情况与资金无关，故不做过多的严格逻辑限制，即最后1+位答题结束不提示答题不成功！答题数目为12题，DB关键查询字段做索引，故Redis不承载太多答题信息，目前只将：个人对商品发起状态，个人帮答次数，互帮情况做缓存。候选答题近500道，非结构化，还可能不稳定，DB添加若json我需要解析，正常字段，后期若变更需要改变表结构，后台添加功能字段也复杂，此时mongo最合适不过了，excel和json导入均可。	  

* ### 流程图
	     
![](/Users/huangfan/Desktop/work/发起.png)  

      
![](/Users/huangfan/Desktop/work/参与.png) 
		       	       
		       
		       
		       
		       
		       
		       
		       	  