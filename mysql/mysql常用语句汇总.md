### 常用语句

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `user_id` bigint(20) NOT NULL COMMENT '用户id',//Long
  `gender` CHAR(1) NOT NULL COMMENT '性别',
  `user_name` varchar(255) DEFAULT '' COMMENT '用户名称', //String
  `direction` decimal(36,18) NOT NULL comment '方向rate',
  `user_phone` varchar(50) DEFAULT '' COMMENT '用户手机',
  `address` varchar(255) DEFAULT '' COMMENT '住址',
  `weight` int(3) NOT NULL DEFAULT '1' COMMENT '权重，大者优先', //Integer
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',//Date
  `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO `user` VALUES (10000, 0, 'STOMA_GREATER_10', 0, 1000);
```

修改表语句

```mysql
ALTER TABLE sp_reply_rule ADD COLUMN associated_message VARCHAR(1000) DEFAULT NULL COMMENT '关联消息';
ALTER TABLE `sp_account_bind` ADD unique(`outer_sa_id`);
```

