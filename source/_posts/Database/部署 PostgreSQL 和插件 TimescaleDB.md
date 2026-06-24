---
title: 部署 PostgreSQL 和插件 TimescaleDB
date:  2026-06-21 09:04:40

math: false
mermaid: false
category_bar: false

categories:
  - Database

tags:
  - Docker
  - PostgreSQL
  - TimescaleDB

excerpt: "使用 Docker 部署 timescale 的官方镜像"
permalink: /posts/20260621-090440.html
---
## 1. 使用 Docker 部署

> [timescale/timescaledb - Docker Image](https://hub.docker.com/r/timescale/timescaledb?tag=latest-pg18)

Timescale 官方提供了一个已经打包好 PostgreSQL 和 TimescaleDB 甚至调优过内核参数的镜像。使用以下命令拉取镜像：

```Bash
docker pull timescale/timescaledb:2.28.0-pg18
```

{% note warning %}
请根据需求选择合适的版本，但是注意不要使用带 `-oss` 后缀版本。其为 Apache 纯开源版，屏蔽了一些自动化特性。
{% endnote %}

创建容器命令如下：

```Bash
docker run -d --name timescaledb \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=Password \
  -e TZ=Asia/Shanghai \
  -v timescaledb_data:/var/lib/postgresql/data \
  timescale/timescaledb:2.28.0-pg18
```

{% note info %}
修改 `Password` 为你的密码。
{% endnote %}

## 2. 创建数据库

在 PostgreSQL 中，插件的生效范围是数据库级别，而不是服务器级别。这种隔离机制保证了系统的纯净和稳定。

假设 PostgreSQL 服务器里有三个业务数据库：`user_db`、`order_db` 和 `metrics_db`。即使在服务器上安装了 TimescaleDB 的插件，这三个数据库依然只是普通的 PG 数据库。当登录到 `metrics_db` 并加载 TimescaleDB 插件后，只有 `metrics_db` 会变成支持时序特性的数据库，其他两个数据库丝毫不受影响。

1. 登录数据库命令行：

```Bash
docker exec -it timescaledb psql -U postgres
```

2. 创建新数据库：

```SQL
CREATE DATABASE metrics_db;
```

3. 切换数据库：

```SQL
\c metrics_db
```

4. 加载 TimescaleDB 插件：

```SQL
CREATE EXTENSION IF NOT EXISTS timescaledb;
```

## 3. 配置数据表

创建了业务数据库之后，下一步就是创建一张新表，并将其配置为 TimescaleDB 的超表。

### 3.1 创建新表：

```SQL
CREATE TABLE device_metrics (
    created_at    TIMESTAMPTZ       NOT NULL,  -- 带时区的时间戳
    -- 其它字段
    payload       BYTEA             NOT NULL
);
```

{% note warning %}
TimescaleDB 强依赖时间戳，所以表里必须有一个时间字段。此外，在 TimescaleDB 中，如果表要设置主键或唯一约束，主键中必须包含时间列，否则无法成功转换为超表。
{% endnote %}

### 3.2 配置超表

对 `device_metrics` 表，按 `created_at` 字段对数据进行分区。

```SQL
SELECT create_hypertable('device_metrics', by_range('created_at', INTERVAL '1 day'));
```

- `INTERVAL '1 day'`：设置生成物理文件块的间隔为 1 天；

### 3.3 配置数据自动保留策略

TimescaleDB 的后台定时任务会自动扫描，当某一天的数据块完全超过设定时间后，直接 Drop 掉释放空间。

```SQL
SELECT add_retention_policy('device_metrics', INTERVAL '3 days');
```

- `INTERVAL '3 days'`：数据保留的时间，超过此时间即删除；
