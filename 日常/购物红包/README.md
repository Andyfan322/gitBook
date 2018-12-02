# 二、购物红包
>新用户定义：活动上线日起，没有帮拆过红包的用户。

* 红包规则
  * 获得规则
	  * 用户下单后获取红包
	  * 订单确认拉取过来后发放
	  * **==红包有效期4h发生退款退货，购物红包立马触发失效操作==**
  * 红包本身规则
     * 红包+返现不能超过实付金额，最大为券后价50%；
	  * 单个红包最小为0.1元，最大为50元( **==< 0.1元不发放红包，>50元按照50元封顶==**)
  	  * 红包有效期时间为4h内，即从订单拉取过来后获取红包，开始倒计时4h

  * 用户领取规则
  		* 设置黑名单，拉人红包只给0.01
	  	* 红包需要分享拉人过来帮拆，才能正真的获得
	  	* 每人每天可以帮拆1次
	  	* 每人每天可以帮助同一个人帮拆1次
	  	* 帮拆与被帮人获得金额同等
  
  * 帮拆金额规则
     * 红包杠杆系数
     * 老用户：0.1~0.15
     * 新用户：0.3~0.8

 * 红包的状态：待分享、帮拆中、 拆光、 过期、失效    
  
* 表

```sql
CREATE TABLE `shoping_red_pac` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `ctime` datetime NOT NULL COMMENT '创建时间',
  `mtime` datetime NOT NULL COMMENT '最后修改时间',
  `user_id` bigint(20) NOT NULL COMMENT '用户Id',
  `start_time` datetime NOT NULL COMMENT '第一次分享/帮拆开始时间',
  `end_time` datetime NOT NULL COMMENT '拆光时间',
  `expire_time` datetime NOT NULL COMMENT '过期时间',
  `platform_order_id` bigint(20) NOT NULL COMMENT '平台订单表id',
  `goods_id` bigint(20) NOT NULL COMMENT '商品id',
  `goods_name` varchar(20) NOT NULL COMMENT '商品名',
  `goods_thumbnail_url` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '商品缩略图',
  `order_amount` int(11) NOT NULL DEFAULT 0 COMMENT '实际支付金额，单位为分',
  `redPac_amount` int(11) NOT NULL DEFAULT 0 COMMENT '红包金额，单位为分',
  `status` int(2) NOT NULL DEFAULT 0 COMMENT '状态 0:待分享，1:帮拆中，2:拆光，3:过期，4:失效',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户购物红包表';

CREATE TABLE `shoping_red_pac_details` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `ctime` datetime NOT NULL COMMENT '创建时间',
  `mtime` datetime NOT NULL COMMENT '最后修改时间',
  `shoping_red_pac_id` bigint(20) NOT NULL COMMENT '购物红包表id',
  `help_user_id` bigint(20) NOT NULL COMMENT '帮拆用户Id',
  `owner_amount` int(11) NOT NULL DEFAULT 0 COMMENT '购物主获得红包金额，单位为分',
  `helper_amount` int(11) NOT NULL DEFAULT 0 COMMENT '帮拆者获得红包金额，单位为分',
  PRIMARY KEY (`id`),
  KEY `idx_shoping_red_pac_id` (`shoping_red_pac_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户购物红包，帮拆明细表';
```

* 条件
	  1. 红包金额区间 0.1 -50
	  2. 红包状态（抢光 过期）
	  3. 曾经帮拆过此红包
	  4. 个人每天可帮拆总次数
	  5. 个人每天帮同一个人帮拆总次数
	  6. 是否为最后一个红包

	  
* redis key

	|key | 默认值/说明|过期时间|
	|---|---|---|
	|黑名单用户集合 || |
	|帮拆用户集合 || |
	|每日可以帮拆次数 |1|	
	|每日帮同一个人帮拆次数 |1|	|
	|红包杠杆系数|2|	|
	|老用户拆红包范围，单位分|10，15|	|
	|新用户拆红包范围，单位分 |30，80|	|
	|用户单日已经帮拆次数|0|今日剩余时间|	|
	|帮拆详情 |为了限制一个红包同一个人只能帮拆一次|红包过期时间为4h，可设置略大于4h||	
* 功能点
  * 页面提醒
  * 购物红包列表
  * MQ&模版消息&朋友圈分享图
  * 发红包
  * 分享红包
  * 帮拆红包(并发情况下，采用Redisson分布式锁)

* 流程图

![](http://47.95.12.0:3389/ftp/20181128135156.png)

![](http://47.95.12.0:3389/ftp/20181127161628.png)

![](http://47.95.12.0:3389/ftp/20181128140450.png)

	
	
	
	
	
	
	
	
	
	
	
	
	
	

