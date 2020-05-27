##  mysql开启binlog，确保binlog格式为row，并且创建复制的账号
```
[mysqld]
log-bin=mysql-bin # 开启 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1
```

```sql
CREATE USER canal IDENTIFIED BY '123';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

## mysql表结构转换为gp的格式
使用Navicat Premium，点击工具，结构同步，左边选择mysql，右边选择Greenplum，同时去掉选项中的创建记录，就能在gp中创建表了。

## 导出mysql的数据进行初始化

查看mysql可以导出的目录
```sql
show variables like 'secure_file_priv';
```

执行导出
```sql
mysqldump -uroot -P3306 -p --add-locks=0 --skip-lock-tables --single-transaction --skip-tz-utc --quick --tab=/var/lib/mysql-files/ --fields-terminated-by='|' --master-data=2 dbname tblname
```

导出完成时会显示binlog位置信息，记录下来
```
bin.001172', MASTER_LOG_POS=33983444
```

如果表里含有多行字符，需要额外处理把换行去掉，不然导入到greenplum会报错，先导到开发服把换行符删除
```sql
UPDATE tbl SET content = REPLACE(REPLACE(content, CHAR(10), ''), CHAR(13), '');
```

## 启动gp文件服务

把导出的数据拷贝到其中一台GP服务器10.230.130.2

```shell
scp -r root@10.160.1.1:/var/lib/mysql-files /tmp
mv /tmp/mysql-files/* /home/gpadmin/external_files
```

在10.230.130.2这台机器上启动gpfdist文件服务

```shell
nohup gpfdist -d /home/gpadmin/external_files -p 8081 -l gpfdist.log &
```

## 导入数据到GP

使用develop账号登录到gp master(10.230.130.1)导入初始化数据，可以直接用navicat也可以用psql命令行。

```sql
--------------------------------------------------------------------------------------------------- 创建修改更新时间的触发器

CREATE OR REPLACE FUNCTION dw_update_at() RETURNS TRIGGER AS
$$
BEGIN
    NEW.dw_updated_at = current_timestamp;
    RETURN NEW;   
END;
$$
language plpgsql;

--------------------------------------------------------------------------------------------------- ods_order

CREATE TABLE ods_order (
  id int PRIMARY KEY,
  order_no bigint,
  uid int,
  amount numeric(10),
  status smallint,
  create_time timestamp,
  created_at timestamp,
  updated_at timestamp
) DISTRIBUTED BY (id);

COMMENT ON COLUMN "public"."ods_order"."id" IS '订单id';
COMMENT ON COLUMN "public"."ods_order"."order_no" IS '订单编号';
COMMENT ON COLUMN "public"."ods_order"."uid" IS '用户id';
COMMENT ON COLUMN "public"."ods_order"."amount" IS '总金额';
COMMENT ON COLUMN "public"."ods_order"."status" IS '状态';
COMMENT ON COLUMN "public"."ods_order"."create_time" IS '生成时间';
COMMENT ON TABLE "public"."ods_order" IS '订单表';

-- 创建外部表
CREATE READABLE EXTERNAL TABLE ext_ods_order
(LIKE ods_order)
LOCATION ('gpfdist://10.230.130.2/order.txt')
FORMAT 'TEXT' (DELIMITER '|')
LOG ERRORS SEGMENT REJECT LIMIT 5;

-- 从外部表导入数据到本地表
INSERT INTO ods_order (select * from ext_ods_order);

-- 添加创建时间和更新时间字段
ALTER TABLE ods_order ADD COLUMN dw_created_at timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP;
ALTER TABLE ods_order ADD COLUMN dw_updated_at timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP;

-- 创建时间和更新时间利用触发器做到自动更新
CREATE TRIGGER update_ods_order_updated_at BEFORE UPDATE ON ods_order FOR EACH ROW EXECUTE PROCEDURE dw_update_at();

--授予develop权限
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA TO develop;
```

## 安装canal
 - 安装[canal-deploy](https://github.com/alibaba/canal/wiki/QuickStart)
 - 安装[canal-adapter](https://github.com/alibaba/canal/wiki/ClientAdapter)

按照官网教程配置好数据库和表映射就可以增量更新了。

## 常用命令
```sql
-- 查看导入外部表的错误信息
SELECT * from gp_read_error_log('ext_ods_order');

-- 清空导入外部表的错误信息
SELECT gp_truncate_error_log('ext_ods_order');
```

查看kafka消息
```shell
/workspace/app/kafka/bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092  --from-beginning --topic egatee
```

查看kafka消费状态
```shell
/workspace/app/kafka/bin/kafka-consumer-groups.sh --bootstrap-server 127.0.01:9092 --describe --group ods
GROUP     TOPIC     PARTITION         CURRENT-OFFSET           LOG-END-OFFSET        LAG
组        主题      分区              当前已消费的条数         总条数               未消费的条数
```

