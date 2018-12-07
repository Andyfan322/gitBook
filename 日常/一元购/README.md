# 三、一元购

* ###介绍		
 商品来自pdd，采用系统初步抓取，再由运营人员筛选出符合规则的商品作为一元购商品。原则上商品补贴为5元上下，用户购买后实际到手价为1元。计算公式：==1元=券后价-返现-补贴==
 
* ###商品规则
  * 券后价 (8,15)
  * ?<=补贴<=?，即 ?<=券后价-返现-1<=?
* ###购物规则
  * 用户购买后需要拉新购买，即 1+n(新人)模式拼团
  * 互斥：新用户参加一元购 VS 新人首次下单返三元红包
* ###状态/触发说明
   * 状态：
      * 团状态：0:开团中；1:过期；2:失效（开团中，团长订单发生退款行为）；3:商品在pdd下架
      * 是否人满：0:否；1:是
   * 触发
	   1. 订单拉取
		   * a.团长触发
		        * 开团中
		        * 失效
		    * b.团员触发
		        * 人满
		2. 小程序触发
		   * 过期
		   * pdd下架
* ###功能
   * 运营后台
     * 商品库添加/更新
   * 小程序
     * 一元购商品列表
     * 我的一元购列表
     * 商品详情
     * 我的团详情
     * MQ
     * 朋友圈
   * 订单
     * 开团/加入团 

* ###表结构
     
```sql
CREATE TABLE `one_yuan_products` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `ctime` timestamp NOT NULL DEFAULT current_timestamp() COMMENT '创建时间',
  `mtime` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp() COMMENT '最后修改时间',
  `need_num` int(2) NOT NULL DEFAULT 1 COMMENT '需要参加团新人数，排除团长',
  `is_show` int(2) DEFAULT 1 COMMENT '是否显示  0.表示否， 1.表示是，默认为1',
  `pdd_taken_down` int(2) DEFAULT 0 COMMENT 'pdd是否下架  0.表示否， 1.表示是，默认为0',
  `start_time` datetime NOT NULL COMMENT '推荐时间（始）',
  `end_time` datetime DEFAULT NULL COMMENT '推荐时间（结束）',
  `product_id` bigint(20) NOT NULL COMMENT '商品id',
  `product_img` varchar(500) DEFAULT '' COMMENT '商品主图',
  `product_name` varchar(255) NOT NULL COMMENT '商品名称',
  `cust_img` varchar(500) DEFAULT NULL COMMENT '自定义图',
  `cust_name` varchar(255) NOT NULL COMMENT '商品名称',
  `dis_price` decimal(20,6) DEFAULT NULL COMMENT '券后价',
  `reappearance` decimal(20,6) DEFAULT NULL COMMENT '返现金额',
  `subsidy_amount` decimal(20,6) DEFAULT NULL COMMENT '补贴金额',
  `sort` int(3) DEFAULT NULL COMMENT '升序',
  `sales_num` int(10) DEFAULT NULL COMMENT '销量',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='一元购物，商品库表';

CREATE TABLE `one_yuan_shopping_group` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `ctime` datetime NOT NULL COMMENT '创建时间',
  `mtime` datetime NOT NULL COMMENT '最后修改时间',
  `user_id` bigint(20) NOT NULL COMMENT '团长Id',
  `avatar` varchar(255) DEFAULT '' COMMENT '头像',
  `nick_name` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '昵称',
  `end_time` datetime DEFAULT NULL COMMENT '满员时间点',
  `expire_time` datetime NOT NULL COMMENT '过期时间点',
  `invalid_time` datetime DEFAULT NULL COMMENT '失效时间点',
  `pdd_taken_down_time` datetime DEFAULT NULL COMMENT 'pdd商品下架状态，触发时间点',
  `platform_order_id` bigint(20) NOT NULL COMMENT '平台订单表id',
  `order_sn` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '推广订单编号',
  `order_amount` int(11) NOT NULL DEFAULT 0 COMMENT '实际支付金额，单位为分',
  `reappearance` decimal(20,6) DEFAULT NULL COMMENT '返现金额',
  `subsidy_amount` decimal(20,6) DEFAULT NULL COMMENT '补贴金额',
  `one_yuan_products_id` bigint(20) NOT NULL COMMENT '一元商品表Id',
  `goods_id` bigint(20) NOT NULL COMMENT '商品id',
  `goods_name` varchar(255) NOT NULL DEFAULT '' COMMENT '商品名',
  `goods_thumbnail_url` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '商品缩略图',
  `is_enough` int(2) NOT NULL DEFAULT 0 COMMENT '满员，1:是；0否',
  `need_num` int(2) NOT NULL DEFAULT 2 COMMENT '需要入团人数',
  `status` int(2) NOT NULL DEFAULT 0 COMMENT '状态 0:开团中，1:过期，2:失效（开团中，团长订单发生退款行为），3:商品在pdd下架',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='一元购物，开团表';

CREATE TABLE `one_yuan_shopping_group_details` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `ctime` datetime NOT NULL COMMENT '创建时间',
  `mtime` datetime NOT NULL COMMENT '最后修改时间',
  `user_id` bigint(20) NOT NULL COMMENT '团员Id',
  `avatar` varchar(255) DEFAULT '' COMMENT '头像',
  `nick_name` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '昵称',
  `platform_order_id` bigint(20) NOT NULL COMMENT '平台订单表id',
  `order_sn` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '推广订单编号',
  `order_amount` int(11) NOT NULL DEFAULT 0 COMMENT '实际支付金额，单位为分',
  `reappearance` decimal(20,6) DEFAULT NULL COMMENT '返现金额',
  `subsidy_amount` decimal(20,6) DEFAULT NULL COMMENT '补贴金额',
  `one_yuan_shopping_group_id` bigint(20) NOT NULL COMMENT '开团表Id',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='一元购物，开团明细表';
```
* 关键点
  * 团长返现：订单正常+团满；若是新人（注意和首单返现互斥）
  * 团员返现：订单正常+新人（注意和首单返现互斥）
  * 平台参数表，关键参数（id，类型，邀请人Id，开团表Id，返现金额）
  * 订单表，关键参数，开团表Id
  * 点击列表商品，团长：团详情/开团；团员：开团
  * 点击分享链接：团长：团详情；团员：开团/加入团
  
* ###流程图	
  
  ![](http://47.95.12.0:3389/ftp/整体流程.png)

  ![](http://47.95.12.0:3389/ftp/进入商品列表.png)

  ![](http://47.95.12.0:3389/ftp/1.png)
  
  ![](http://47.95.12.0:3389/ftp/2.png)
  
  ![](http://47.95.12.0:3389/ftp/进入商品详情.png)
  
  ![](http://47.95.12.0:3389/ftp/订单.png)
