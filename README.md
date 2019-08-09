
##数据流向图

#### 写入

![图片描述](https://tapd.tencent.com/tfl/captures/2019-06/tapd_10138461_base64_1561712783_17.png)
* job消费需判断entity_id和item_id是否有效，如果无效则不写入
* 接口写入时支持批量功能避免调用方多次调用
#### 读取

![图片描述](https://tapd.tencent.com/tfl/captures/2019-06/tapd_10138461_base64_1561712915_15.png)

## 表结构

###实体配置表
```
CREATE TABLE `log_entity_config` (
  `entity_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名称',
  `query_mapping_filed` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT 'id对应值;例如书，则为CBID',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态：1.有效 2：无效',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`entity_id`),
  KEY `name` (`name`),
  KEY `query_mapping_filed` (`query_mapping_filed`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='日志实体配置表(对应业务类型 比如书)'
```

### 实体项配置表
```
CREATE TABLE `log_item_config` (
  `item_id` bigint(20) unsigned NOT NULL COMMENT '主键',
  `entity_id` bigint(20) unsigned NOT NULL DEFAULT '1' COMMENT '实体id',
  `item_field_name` varchar(30) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '实体项对应名,该字段用于接口传入使用',
  `name` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名称',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态：1.有效 2：无效',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`item_id`),
  KEY `entity_id` (`entity_id`),
  UNIQUE KEY  `entity_id_fieid` (`entity_id`,`item_field_name`),
  KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='日志修改属性配置表（具体到字段）'
```
### 平台表
 ```
CREATE TABLE `log_platform_config` (
  `platform_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '名称',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态：1.有效 2：无效',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`platform_id`),
  KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='日志平台配置表'
```
### 实体项枚举映射表
 ```
CREATE TABLE `log_enum_config` (
  `enum_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `item_id` tinyint(4) NOT NULL DEFAULT '1' COMMENT 'item_id,item_config表id',
  `item_value` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT 'item值',
  `item_display_name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT 'item展示值',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态：1.有效 2：无效',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`enum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='修改项枚举表'
```

### 流水日志表
```
CREATE TABLE `item_log_{按年月日分表}` (
`IDX` bigint(20) NOT NULL,
`entity_id` int(11) unsigned NOT NULL DEFAULT '1' COMMENT '所属实体，entity表id字段',
`item_id` int(3) unsigned NOT NULL COMMENT '修改项，item配置表id字段',
`id` varchar(50) COLLATE utf8_unicode_ci NOT NULL DEFAULT '' COMMENT '该类实体对应的id',
`old_value` mediumtext COLLATE utf8mb4_unicode_ci NOT NULL,
`display_old_value` mediumtext COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '展示旧值文本',
`new_value` mediumtext COLLATE utf8mb4_unicode_ci NOT NULL,
`display_new_value` mediumtext COLLATE utf8mb4_unicsode_ci NOT NULL COMMENT '展示新值文本',,
`user_id` bigint(11) NOT NULL DEFAULT '0' COMMENT '操作人id',
`user_name` varchar(30) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作人名字',
`user_email` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作人邮箱',
`reason` mediumtext COLLATE utf8mb4_unicode_ci NOT NULL default '' COMMENT '操作原因',
`platform` int(11) unsigned NOT NULL DEFAULT '1' COMMENT '平台 platform表id字段',
`create_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '创建时间',
`source` int(11) unsigned NOT NULL DEFAULT '1' COMMENT '来源 1.页面，2.消息表，3.队列，4.接口 ',
`entrance` text COLLATE utf8mb4_unicode_ci NOT NULL default '' COMMENT '入口',
`user_ip` varchar(50) COLLATE utf8_unicode_ci NOT NULL DEFAULT '' COMMENT '操作IP',
PRIMARY KEY (`IDX`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='日志流水表';
```

###接口数据格式
```
{
	"user_id": 11,
	"user_name": "张三",
    "user_email": "zhangsan@yuewen.com",
	 "platform": 1,
	"user_ip": "10.222.222.32",
	"requset_url": "http://service.book.qq.com/",
      "create_time": "2019-01-01 23:33:22",
      "reason": "作者要求下架",
	"items": [{
		"entity_id": 1
		"id": "13784938500744506",
		"item_id": 1,
		"old_value": "择天记",
		"display_old_value": "择天记",
		"new_value": "择天记(全册)",
		"display_new_value": "择天记(全册)"
	}, {
		"entity_id": 1,
		"id": "13784938500744506",
		"item_id": 2,
		"old_value": "19",
		"new_value": "-1"
	}]
}
```

## 接入流程
### 

![图片描述](https://tapd.tencent.com/tfl/captures/2019-07/tapd_10138461_base64_1562578623_95.png)
* 页面(source:1)： 页面产生的数据变化entrance为json，需包含页面入口地址entrance：{'tikect':'http://service.com/bianji?'}
* 通知表(source:2):通知表entrance为json，需包含消息表主键id entrance：{'tikect':'1212122332'}，包括批处理
* 消息队列(source:3):消费消息队列，entrance：{'tikect':'S0021212122'}
* 接口(source:4)： 接口产生的数据变化entrance为json，需包含接口地址entrance：{'tikect':'http://api.service.com/bianji?'}

消息门面服务
NRZX.ItemLogFacadeServer.****ItemLogFacadeServant****
