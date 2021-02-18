建表并导入数据：


```mysql
CREATE TABLE `province` (
  `province_id` int(11) NOT NULL AUTO_INCREMENT,
  `province_name` varchar(255) DEFAULT NULL,
  `gdp` double(11,0) DEFAULT NULL,
  PRIMARY KEY (`province_id`)
) ENGINE=InnoDB AUTO_INCREMENT=36 DEFAULT CHARSET=utf8;

INSERT INTO `province` VALUES ('1', '北京', '24899');
INSERT INTO `province` VALUES ('2', '天津', '17885');
INSERT INTO `province` VALUES ('3', '河北省', '31827');
INSERT INTO `province` VALUES ('4', '山西省', '13766');
INSERT INTO `province` VALUES ('5', '内蒙古自治区', '18632');
INSERT INTO `province` VALUES ('6', '辽宁省', '28669');
INSERT INTO `province` VALUES ('7', '吉林省', '14063');
INSERT INTO `province` VALUES ('8', '黑龙江省', '15083');
INSERT INTO `province` VALUES ('9', '上海', '27466');
INSERT INTO `province` VALUES ('10', '江苏省', '76086');
INSERT INTO `province` VALUES ('11', '浙江省', '46485');
INSERT INTO `province` VALUES ('12', '安徽省', '24117');
INSERT INTO `province` VALUES ('13', '福建省', '28519');
INSERT INTO `province` VALUES ('14', '江西省', '18364');
INSERT INTO `province` VALUES ('15', '山东省', '67008');
INSERT INTO `province` VALUES ('16', '河南省', '40160');
INSERT INTO `province` VALUES ('17', '湖北省', '32297');
INSERT INTO `province` VALUES ('18', '湖南省', '31224');
INSERT INTO `province` VALUES ('19', '广东省', '79512');
INSERT INTO `province` VALUES ('20', '广西壮族自治区', '18245');
INSERT INTO `province` VALUES ('21', '海南省', '4044');
INSERT INTO `province` VALUES ('22', '重庆', '17558');
INSERT INTO `province` VALUES ('23', '四川省', '32680');
INSERT INTO `province` VALUES ('24', '贵州省', '11734');
INSERT INTO `province` VALUES ('25', '云南省', '14869');
INSERT INTO `province` VALUES ('26', '西藏自治区', '1148');
INSERT INTO `province` VALUES ('27', '陕西省', '19165');
INSERT INTO `province` VALUES ('28', '甘肃省', '7152');
INSERT INTO `province` VALUES ('29', '青海省', '2572');
INSERT INTO `province` VALUES ('30', '宁夏回族自治区', '2911');
INSERT INTO `province` VALUES ('31', '新疆维吾尔自治区', '9550');
INSERT INTO `province` VALUES ('32', '台湾省', '0');
INSERT INTO `province` VALUES ('33', '香港特别行政区', '0');
INSERT INTO `province` VALUES ('34', '澳门特别行政区', '0');
```


![img](MySQL 查询数据并按顺序添加序号.assets/20171010141538018)



MYSQL查询语句：


```mysql
SELECT
	province_id,
	province_name,
	gdp,
	(@i :=@i + 1) AS No
FROM
	province,
	(SELECT @i := 0) AS it
ORDER BY
	gdp DESC
```


![img](MySQL 查询数据并按顺序添加序号.assets/20171010141954141)

