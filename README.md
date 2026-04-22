### Docker Hive 电商数据分析项目
基于 Docker 的 Hive 数据仓库项目，对 Olist 巴西电商卖家数据集进行完整的数据清洗与分析。
![Hive](https://img.shields.io/badge/Hive-2.3.2-yellow)
![Docker](https://img.shields.io/badge/Docker-20.10-blue)
![Hadoop](https://img.shields.io/badge/Hadoop-2.7.4-green)

### 分析结果（核心成果）
卖家按州分布（Top 5）
| 州 | 卖家数量 | 占比 |
|----|---------|------|
| SP (圣保罗) | 1,849 | 59.7% |
| PR (巴拉那) | 349 | 11.3% |
| MG (米纳斯吉拉斯) | 244 | 7.9% |
| SC (圣卡塔琳娜) | 190 | 6.1% |
| RJ (里约热内卢) | 170 | 5.5% |

卖家按城市分布（Top 10）
| 城市 | 卖家数量 |
|------|---------|
| sao paulo | 694 |
| curitiba | 127 |
| rio de janeiro | 96 |
| belo horizonte | 68 |
| ribeirao preto | 52 |
| guarulhos | 50 |
| ibitinga | 49 |
| santo andre | 45 |
| campinas | 41 |
| maringa | 40 |


卖家按邮编分布（Top 10）
| 邮编前缀 | 卖家数量 |
|---------|---------|
| "14940" | 49 |
| "13660" | 10 |
| "16200" | 9 |
| "13920" | 9 |
| "87050" | 8 |
| "01026" | 8 |
| "14020" | 8 |
| "13481" | 7 |
| "37540" | 7 |
| "87015" | 6 |

### 完整操作步骤：
### 1. 启动环境并进入容器
进入 Hive 容器
docker exec -it docker-hive-master-hive-server-1 bash

### 2. 准备 HDFS 目录
# 删除旧数据
hdfs dfs -rm -r /data/retail
# 创建新目录
hdfs dfs -mkdir -p /data/retail

### 3. 从宿主机复制数据到容器
# 在宿主机 PowerShell 中执行
docker cp .\data\olist_sellers_dataset.csv docker-hive-master-hive-server-1:/opt/hive/

### 4. 上传数据到 HDFS
# 在容器内执行
hdfs dfs -put /opt/hive/olist_sellers_dataset.csv /data/retail/
# 验证文件已上传
hdfs dfs -ls /data/retail
# 输出：
Found 1 items
-rw-r--r--   3 root supergroup     174703 2026-04-20 14:13 /data/retail/olist_sellers_dataset.csv

### 5. 启动 Beeline 连接 Hive
beeline -u jdbc:hive2://localhost:10000

### 6. 创建数据库和表
-- 使用数据库
USE retail_db;

-- 删除旧表
DROP TABLE IF EXISTS sellers;

-- 创建卖家表（分隔符为逗号）
CREATE TABLE sellers (
    seller_id STRING,
    seller_zip_code_prefix STRING,
    seller_city STRING,
    seller_state STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

### 7. 加载数据到表

LOAD DATA INPATH '/data/retail/olist_sellers_dataset.csv'
INTO TABLE sellers;

### 8. 验证数据加载
SELECT * FROM sellers LIMIT 10;
# 输出：(省略)
| seller_id | seller_zip_code_prefix | seller_city | seller_state |
|-----------|------------------------|-------------|--------------|
| "3442f8959a84dea7ee197c632cb2df15" | "13023" | campinas | SP |
| d1b65fc7debc3361ea86b5f14c68d2e2 | "13844" | mogi guacu | SP |
| ce3ad9de960102d0677a81f5d0bb7b2d | "20031" | rio de janeiro | RJ |
| c0f3eea2e14555b6faeea3dd58c1b1c3 | "04195" | sao paulo | SP |
| "51a04a8a6bdcb23deccc82b0b80742cf" | "12914" | braganca paulista | SP |
| c240c4061717ac1806ae6ee72be3533b | "20920" | rio de janeiro | RJ |
| e49c26c3edfa46d227d5121a6b6e4d37 | "55325" | brejao | PE |
| "1b938a7ec6ac5061a66a3766e0e75f90" | "16304" | penapolis | SP |
| "768a86e36ad6aae3d03ee3c6433d61df" | "01529" | sao paulo | SP |

### 9. 数据清洗

CREATE VIEW sellers_clean3 AS
SELECT *
FROM sellers
WHERE seller_id NOT IN ('seller_id', '"seller_id"')
  AND seller_state IS NOT NULL
  AND seller_state != '';

### 10. 验证清洗结果

-- 查看清洗后数据
SELECT * FROM sellers_clean3 LIMIT 10;
（省略部分）
| seller_id | seller_zip_code_prefix | seller_city | seller_state |
|-----------|------------------------|-------------|--------------|
| "3442f8959a84dea7ee197c632cb2df15" | "13023" | campinas | SP |
| d1b65fc7debc3361ea86b5f14c68d2e2 | "13844" | mogi guacu | SP |
| ce3ad9de960102d0677a81f5d0bb7b2d | "20031" | rio de janeiro | RJ |
| c0f3eea2e14555b6faeea3dd58c1b1c3 | "04195" | sao paulo | SP |
| "51a04a8a6bdcb23deccc82b0b80742cf" | "12914" | braganca paulista | SP |
| c240c4061717ac1806ae6ee72be3533b | "20920" | rio de janeiro | RJ |


-- 去重查看
SELECT DISTINCT * FROM sellers_clean2;
（省略部分）
| seller_id | seller_zip_code_prefix | seller_city | seller_state |
|-----------|------------------------|-------------|--------------|
| "0015a82c2db000af6aaaf3ae2ecb0532" | "09080" | santo andre | SP |
| "001cca7ae9ae17fb1caed9dfb1094831" | "29156" | cariacica | ES |
| "001e6ad469a905060d959994f1b41e4f" | "24754" | sao goncalo | RJ |
| "002100f778ceb8431b7a1020ff7ab48f" | "14405" | franca | SP |
| "003554e2dce176b5555353e4f3555ac8" | "74565" | goiania | GO |
| "004c9cd9d87a3c30c522c48c4fc07416" | "14940" | ibitinga | SP |

### 11. 数据分析查询

#### 按州统计卖家数量
SELECT seller_state, COUNT(*) AS cnt 
FROM sellers_clean3 
GROUP BY seller_state 
ORDER BY cnt DESC;

#### 按城市统计（Top 10）
SELECT seller_city, COUNT(*) AS cnt 
FROM sellers_clean3 
GROUP BY seller_city 
ORDER BY cnt DESC 
LIMIT 10;

#### 按邮编前缀统计（Top 10）
SELECT seller_zip_code_prefix, COUNT(*) AS cnt 
FROM sellers_clean3 
GROUP BY seller_zip_code_prefix 
ORDER BY cnt DESC 
LIMIT 10;
```
项目结构
docker-hive-master/
├── data/                    # 数据文件目录
│   └── olist_sellers_dataset.csv
├── conf/                    # Hive 配置文件
├── docker-compose.yml       # Docker 编排文件
├── Dockerfile               # Docker 镜像构建文件
└── README.md               # 项目说明文档

注意事项
1. 大文件处理：原始数据文件（如 2019-Oct.csv 约 3.13GB）未上传到 GitHub，如需使用请自行下载
2. 分隔符问题：CSV 文件使用逗号 `,` 作为分隔符
3. 表头处理：原始数据包含表头行，已在视图中过滤
4. 执行引擎警告：`Hive-on-MR is deprecated` 是正常提示，不影响查询结果

数据源
[Olist Brazilian Ecommerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

技术栈
| 组件 | 版本 |
|------|------|
| Docker | 28.4.0 |
| Hadoop | 2.7.4 |
| Hive | 2.3.2 |
| PostgreSQL | Metastore |

作者
wencai244-commits
