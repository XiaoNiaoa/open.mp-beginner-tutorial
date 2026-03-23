# samp/openmp gtasa联机服务器 MySQL 数据库教程

编写：小鸟 [Github](https://github.com/XiaoNiaoa)
时间：2026/03/23

### 数据保存：读、查、改、删

---

> 这篇教程只**教你用代码操作数据库里的数据**。
> 不涉及登录注册系统，只讲最核心的数据库操作原理和用法。

---

## 第一章：数据库？

**数据库在哪？**

你可以把数据库当成一个 **Excel 文件**，它就在你电脑硬盘上。

这个 Excel 文件里有很多张表格，每张表格存某一类型的数据，比如玩家数据 载具数据 房屋数据：

```
数据库（openmp）
├── players 表        ← 存玩家数据
│   ├── id | name | gold | level | skin
│   ├──  1 | Tom  | 5000 |    10 |  287
│   └──  2 | Jane | 2000 |     5 |   86
│
├── houses 表         ← 存房产数据
│   └── ...
│
└── vehicle 表        ← 存载具数据
    └── ...
```

**为什么不用 .ini 文件或者 .txt 文件存？**

可以，但MySQL数据库的优势：

- **高并发处理能力**：不会阻塞主线程，支持非阻塞/异步查询
- **速度快**：性能强劲，毫秒级返回结果，不需要历遍查找数据
- **安全**：数据安全可靠，也可以设置定时备份数据
- **灵活**：可以使用SQL语句执行各种复杂的数据库操作
- **维护管理**：可视化工具可直接管理、修改、备份数据

---

## 第二章：为什么要连接数据库？

数据库是一个**独立运行的软件**（MySQL），它和你的 open.mp 服务端是两个程序。

就像你用 QQ 发消息，你要先登录QQ**连接**腾讯的服务器，才能使用它发消息。

你的 Pawn 脚本要操作数据库，也需要先**建立连接**，告诉数据库：

- 我是谁（用户名/密码）
- 我要操作哪个数据库

建立连接之后，你才能往里面读数据、写数据。

---

## 第三章：安装准备

### 3.1 在本地安装 MySQL

下载 MariaDB Server （80多mb）

下载地址：[https://mariadb.org/download/](https://mariadb.org/download/)

#### 安装过程有两点需要注意

- 在user settings步骤的时候需要你创建一个密码，这个密码你在后面连接的时候需要用到

- 确认TCP port是3306

MariaDB 安装包自带 HeidiSQL，安装完成之后你的桌面会有一个快捷方式：HeidiSQL，这是一个可视化数据库管理工具，不会操作的新手，网上也有非常多的视频教程更加直观，我就不多说了

### 3.2 下载 MySQL R41-4 插件

下载地址：[https://github.com/pBlueG/SA-MP-MySQL/releases/tag/R41-4](https://github.com/pBlueG/SA-MP-MySQL/releases/tag/R41-4)

解压后，把文件按下表复制到服务端对应位置：

| 文件 | 复制到 |
|------|--------|
| `plugins/mysql.dll` | 服务端 `plugins/` 目录 |
| `libmariadb.dll` | 服务端**根目录**（和 omp-server.exe 同级） |
| `log-core.dll` | 服务端**根目录** |
| `pawno/include/a_mysql.inc` | 服务端 `qawno/include/` 目录 |

### 3.3 在 config.json 里加载插件

```json
"pawn": {
    "legacy_plugins": [
        "mysql"
    ]
}
```

启动服务器，控制台看到这行说明插件加载成功：

```
Loading plugin: mysql
 >> plugin.mysql: R41-4 successfully loaded.
```

---

## 第四章：创建数据库并连接数据库

插件装好了，接下来在脚本里建立连接，首先你要先使用 HeidiSQL 创建一个名为 openmp 的数据库。

创建好之后，在pawn写以下代码

```pawn
#include <open.mp>
#include <a_mysql>    // 引入 MySQL 插件头文件

// 全局变量：保存连接句柄
// 句柄相当于钥匙，后续所有操作都要带上它
new MySQL:g_SQLHandle;

public OnGameModeInit()
{
    // mysql_connect("数据库地址", "用户名", "密码", "数据库名")
    g_SQLHandle = mysql_connect("127.0.0.1", "root", "password", "openmp");
    //                              ↑ 127.0.0.1 = 本机，本地固定写这个

    // 检查连接是否成功
    if(g_SQLHandle == MYSQL_INVALID_HANDLE || mysql_errno(g_SQLHandle) != 0)
    {
        print("[MySQL] 连接失败！请检查 MySQL 是否运行。");

        // 连不上数据库就关服，避免带着错误运行
        // 这也是很多新手拿到开源图打开就闪退的原因
        // 因为大部分服务器都是使用MySQL作为数据库
        // 你没有正确搭建数据库环境，下面这行代码会自动关闭服务器
        SendRconCommand("exit");   
        return 1;
    }

    print("[MySQL] 数据库连接成功！");
    return 1;
}
```

---

## 第五章：创建数据表

连接成功后，需要在数据库里建一张表来存玩家数据。

**什么是 SQL？**

SQL 是操作数据库专用的语言，每一条 SQL 语句就是给数据库下一道命令，其实你会发现SQL语句就像是说人话一样，教程的最后会展示一些实用便利的SQL语句示例。

```pawn
public OnGameModeInit()
{
    mysql_query(g_SQLHandle,
        "CREATE TABLE IF NOT EXISTS `players` (\
        `id`    INT          NOT NULL AUTO_INCREMENT,\
        `name`  VARCHAR(24)  NOT NULL UNIQUE,\
        `gold`  INT          DEFAULT 0,\
        `level` INT          DEFAULT 1,\
        `skin`  INT          DEFAULT 86,\
        PRIMARY KEY (`id`))", 
        false
    );

    // 说明：
    // CREATE TABLE = 创建表
    // IF NOT EXISTS = 如果表不存在才创建，已存在就跳过
    // id    - 整数，不能为空，自动递增（每插入一行自动+1，相当于每行的编号）
    // name  - 最多24字符的字符串，不能为空，且不能重复（UNIQUE）
    // gold  - 整数，不填时默认值是 0
    // level - 整数，不填时默认值是 1
    // skin  - 整数，不填时默认值是 86
    // PRIMARY KEY(id) - 设 id 为主键，数据库用它来精确定位某一行

    print("[MySQL] 数据表就绪！");
    return 1;
}
```

执行完这段代码后，你的数据库里就有了一张这样结构的空表：

```
players 表
┌────┬──────┬──────┬───────┬──────┐
│ id │ name │ gold │ level │ skin │
├────┼──────┼──────┼───────┼──────┤
│    │      │      │       │      │
└────┴──────┴──────┴───────┴──────┘
```

### 常用列类型速查

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| `INT` | 整数 | 金币、等级、皮肤 ID |
| `VARCHAR(n)` | 最多 n 个字符的字符串 | 玩家名字、称号 |
| `FLOAT` | 小数 | 坐标、血量 |
| `TEXT` | 长文本（无长度限制） | 个人简介、日志 |
| `TINYINT(1)` | 0 或 1，常用作布尔值 | 是否是管理员、是否封禁 |

### 常用列约束速查

| 约束 | 说明 |
|------|------|
| `NOT NULL` | 不允许为空，插入时必须填这一列 |
| `DEFAULT 值` | 不填时使用的默认值 |
| `UNIQUE` | 该列的值在整张表里不能重复 |
| `AUTO_INCREMENT` | 自动递增，配合 `INT` 主键使用 |
| `PRIMARY KEY` | 主键，每行的唯一标识，一张表只能有一个 |

### 我需要背这些语句吗？

不需要，没有人会背这些，包括写了很多年代码的开发者，你只需要大概知道每种语句是干什么用的，要用的时候回来翻一下，或者直接百度搜 MySQL 创建表格语法，几秒钟就找到了，又或者直接可视化工具输出具体的SQL语句

---

## 第六章：同步 vs 异步（非常重要）

在讲增删改查之前，必须先理解这个概念，否则你会做出卡服务器的代码。

### 同步查询（mysql_query）—— 阻塞查询

```pawn
mysql_query(g_SQLHandle, "SELECT * FROM players");
// 同步查询会让服务器等待数据库返回结果，期间其他代码暂停
// OnGameModeInit 是服务器刚启动时执行的，此时没有任何玩家在线
// 有玩家在线时永远不要使用阻塞查询，除非你真的知道你在干什么
```

就像你在餐厅点餐，服务员站在那里死等厨房做完，期间不接待任何其他桌。

> 但新手不必害怕这个阻塞查询，阻塞过程可能才千分之一秒，也可能因特殊软件异常或你使用的是远程数据库导致的网络波动会持续更久，但安全意识很重要，只要记住在必要时仅在服务器初始化和启动时同步，其他地方异步即可

### 异步查询（mysql_tquery）—— 非阻塞查询

```pawn
mysql_tquery(g_SQLHandle, "SELECT * FROM players", "我的回调函数", "i", playerid);
// 把查询任务交给后台线程，然后立刻执行其他代码
// 等查询完成后，会自动调用 "我的回调函数"
```

就像你在餐厅点完餐，服务员给你一个号码牌，你的菜好了他会来找你，服务员继续招待其他桌。

---

## 第七章：INSERT — 插入（新增一行数据）

**场景**：玩家第一次进入服务器，在数据库里创建一条新的记录。

### SQL 语句解释

```sql
INSERT INTO `players` (`name`, `gold`, `skin`) VALUES ('Tom', 100, 86)
```

- `INSERT INTO` = 向某张表插入数据
- `players` = 表名
- `(name, gold, skin)` = 要填写哪几列信息
- `VALUES (...)` = 信息对应的值

`id` 不需要填，因为它是 `AUTO_INCREMENT`，数据库会自动分配。

### 在 Pawn 里写

```pawn
InsertPlayer(playerid)
{
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    // 使用 mysql_format 防止玩家名字里有单引号导致 SQL 出错（SQL 注入防护）
    // %e 是转义
    new query[128];
    mysql_format(g_SQLHandle, query, sizeof(query),
        "INSERT INTO `players` (`name`, `gold`, `skin`) VALUES ('%e', 50, 86)",
        name
    );

    // 插入完成后调用 OnPlayerInserted，传入 playerid 告诉回调是哪个玩家
    mysql_tquery(g_SQLHandle, query, "OnPlayerInserted", "i", playerid);

    printf("[MySQL] 玩家 %s 数据正在创建...", name);
    return 1;
}

// mysql_tquery 完成后自动调用这里
// mysql 会把sql语句的返回结果传入此回调中
forward OnPlayerInserted(playerid);
public OnPlayerInserted(playerid)
{
    if(!IsPlayerConnected(playerid)) return 0;

    // cache_insert_id() 返回刚刚插入的那行的 id
    new insertID = cache_insert_id();

    if(insertID > 0)
        printf("[MySQL] 玩家 %d 已创建，数据库 ID：%d", playerid, insertID);

    return 1;
}
```

---

## 第八章：SELECT — 查询（读取数据）

**场景**：玩家进入服务器，从数据库里读取他的金币、等级、皮肤。

### SQL 语句解释

```sql
SELECT `gold`, `level`, `skin` FROM `players` WHERE `name` = 'Tom'
```

- `SELECT` = 我要查询
- `gold`, `level`, `skin` = 查这几列（用 `*` 代表查所有列）
- `FROM players` = 从 players 表里查
- `WHERE name = 'Tom'` = 条件：只要名字是 Tom 的那行

### 在 Pawn 里写

```pawn
// 存玩家数据的全局数组
new g_Gold[MAX_PLAYERS];
new g_Level[MAX_PLAYERS];
new g_Skin[MAX_PLAYERS];

LoadPlayerData(playerid)
{
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    new query[128];
    mysql_format(g_SQLHandle, query, sizeof(query),
        "SELECT `gold`, `level`, `skin` FROM `players` WHERE `name` = '%e' LIMIT 1", 
        name
    );
    // LIMIT 1 = 表示限制只查1行，名字是唯一的所以这样写更高效
    mysql_tquery(g_SQLHandle, query, "OnPlayerDataLoaded", "is", playerid, name);
    return 1;
}

forward OnPlayerDataLoaded(playerid, const name[]);
public OnPlayerDataLoaded(playerid, const name[])
{
    if(!IsPlayerConnected(playerid)) return 0;

    // 查询结果有几行
    if(cache_num_rows() == 0)
    {
        // 没找到这个玩家，说明他是新玩家，去插入一条记录
        InsertPlayer(playerid);
        return 1;
    }

    // 从结果的第 0 行（第一行）读取数据，填入全局数组
    // cache_get_value_name_int(行号, "列名", 存到哪个变量)
    cache_get_value_name_int(0, "gold",  g_Gold[playerid]);
    cache_get_value_name_int(0, "level", g_Level[playerid]);
    cache_get_value_name_int(0, "skin",  g_Skin[playerid]);

    printf("[MySQL] 玩家 %s 数据加载完毕：金币=%d 等级=%d 皮肤=%d", name, g_Gold[playerid], g_Level[playerid], g_Skin[playerid]);

    // 数据读好了，让玩家出生
    SpawnPlayer(playerid);
    return 1;
}
```

### 读取不同类型数据的函数

| 函数 | 用途 |
|------|------|
| `cache_get_value_name_int(行, "列", 变量)` | 读取整数 |
| `cache_get_value_name_float(行, "列", 变量)` | 读取浮点数 |
| `cache_get_value_name(行, "列", 变量, 长度)` | 读取字符串 |

---

## 第九章：UPDATE — 修改（更新数据）

**场景**：玩家下线时，把他当前的金币、等级、皮肤写回数据库。

### SQL 语句解释

```sql
UPDATE `players` SET `gold` = 500, `level` = 3 WHERE `name` = 'Tom'
```

- `UPDATE players` = 更新 players 表
- `SET gold = 500, level = 3` = 把这些列设置成新的值
- `WHERE name = 'Tom'` = 只改名字是 Tom 的那行

> **WHERE 非常重要**：如果不写 WHERE，整张表所有行都会被修改！

### 在 Pawn 里写

```pawn
SavePlayerData(playerid)
{
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    new query[256];
    mysql_format(g_SQLHandle, query, sizeof(query),
        "UPDATE `players` SET `gold` = %d, `level` = %d, `skin` = %d WHERE `name` = '%e'",
        g_Gold[playerid],
        g_Level[playerid],
        g_Skin[playerid],
        name
    );

    // 保存不需要读取回调结果，传空字符串 "" 表示不需要回调
    mysql_tquery(g_SQLHandle, query, "");

    printf("[MySQL] 玩家 %s 数据已保存", name);
}

// 玩家下线时调用
public OnPlayerDisconnect(playerid, reason)
{
    SavePlayerData(playerid);
    return 1;
}
```

---

## 第十章：DELETE — 删除（删除一行数据）

**场景**：把这个玩家的数据从数据库里彻底删除。

### SQL 语句解释

```sql
DELETE FROM `players` WHERE `name` = 'Tom'
```

- `DELETE FROM players` = 从 players 表里删除
- `WHERE name = 'Tom'` = 只删名字是 Tom 的那行

> **同样注意 WHERE**：没有 WHERE 条件，整张表的数据全删光！

### 在 Pawn 里写

```pawn
DeletePlayerData(playerid)
{
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    new query[128];
    mysql_format(g_SQLHandle, query, sizeof(query),
        "DELETE FROM `players` WHERE `name` = '%e'",
        name
    );
    mysql_tquery(g_SQLHandle, query, "OnPlayerDataDelete", "s", name);

    
    return 1;
}

forward OnPlayerDataDelete(const name[]);
public OnPlayerDataDelete(const name[])
{
    // 如果影响的行数大于0
    if(cache_affected_rows() > 0)
        printf("[MySQL] 玩家 %s 的数据已删除", name);

    return 1;
}
```

---

## 第十一章：查询多行数据

前面的 SELECT 只查了一个玩家（一行），有时候你需要查多行，比如「排行榜」。

**场景**：查询金币最多的前 5 名玩家。

### SQL 语句解释

```sql
SELECT `name`, `gold` FROM `players` ORDER BY `gold` DESC LIMIT 5
```

- `ORDER BY gold DESC` = 按 gold 列降序排列（DESC = 从大到小，ASC = 从小到大）
- `LIMIT 5` = 只取前 5 行

### 在 Pawn 里写

```pawn
ShowTopPlayers(playerid)
{
    mysql_tquery(g_SQLHandle,
        "SELECT `name`, `gold` FROM `players` ORDER BY `gold` DESC LIMIT 5",
        "OnTopPlayerLoaded",
        "i", playerid
    );
    return 1;
}

forward OnTopPlayerLoaded(playerid);
public OnTopPlayerLoaded(playerid)
{
    if(!IsPlayerConnected(playerid)) return 0;

    new rows = cache_num_rows();   // 获取返回了几行
    if(rows == 0)
    {
        SendClientMessage(playerid, -1, "[排行榜] 暂无数据");
        return 1;
    }

    SendClientMessage(playerid, 0xFFDD44FF, "=== 金币排行榜 ===");

    new name[MAX_PLAYER_NAME];
    new gold;
    
    // 遍历每一行结果
    for(new i = 0; i < rows; i++)
    {
        cache_get_value_name(i, "name", name, sizeof(name));
        cache_get_value_name_int(i, "gold", gold);

        SendClientMessage(playerid, 0xFFFFFFFF, "#%d %s — %d 金币", i + 1, name, gold);
    }
    return 1;
}
```

## 第十二章：异步的安全防护措施

这一章讲的问题**在实际中极罕见发生**，但一旦发生就会造成数据写错玩家，值得了解并防范。

### 问题是什么？

异步查询的特点是：**你发出查询，然后去做别的事，等数据库回来再处理结果。**

这中间有一段时间差，通常只有1毫秒（千分之一秒）不到。正常情况下没有任何问题。但考虑这个极端场景：

```
1. 玩家 海绵宝宝 连入服务器（ID = 5）
2. 发出 SELECT 非阻塞查询，去数据库读 海绵宝宝 的数据
3. 查询还没回来（1毫秒的空档）
4. 玩家 海绵宝宝 突然断线了，此时ID 5 空出来了
5. 另一个玩家 派大星 立刻连入，空出来 ID 5 就分配给了派大星
6. 数据库查询回来了，把 海绵宝宝 的数据写进了 ID 5 的数组
7. 结果：派大星 拿到了 海绵宝宝 的数据
```

这个概率极其罕见——需要同一个玩家槽位在毫秒级内被复用，并且又刚好卡在发出查询 -> 查询结果返回来 的这个过程内

**但对于认真的服务器来说，防一下是好习惯。**

### 解决思路：给每个玩家槽位标一个「版本号」

每次玩家连入或断开，版本号 +1。

查询发出时记录当前版本号，查询回来时检查版本号是否还一致——不一致说明这个槽位已经换人了，直接丢弃结果。
```pawn
// 全局：每个槽位的版本号
new g_PlayerRace[MAX_PLAYERS];

public OnPlayerConnect(playerid)
{
    g_PlayerRace[playerid]++;   // 玩家连入，版本号 +1
    LoadPlayerData(playerid);
    return 1;
}

public OnPlayerDisconnect(playerid, reason)
{
    SavePlayerData(playerid);
    g_PlayerRace[playerid]++;   // 玩家断开，版本号 +1
    return 1;
}
```

发出查询时，把当前版本号一起传进回调参数：
```pawn
LoadPlayerData(playerid)
{
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    new query[128];
    mysql_format(g_SQLHandle, query, sizeof(query),
        "SELECT `gold`,`level`,`skin` FROM `players` WHERE `name`='%e' LIMIT 1",
        name
    );

    // 把 race 一起传入回调，格式 "ii" = 两个整数
    mysql_tquery(g_SQLHandle, query, "OnPlayerDataLoaded", "ii", playerid, g_PlayerRace[playerid]);
    return 1;
}
```

回调里第一件事就是核对版本号：
```pawn
forward OnPlayerDataLoaded(playerid, race);
public OnPlayerDataLoaded(playerid, race)
{
    // 核对版本号
    // 不一致说明这个 ID 已经换了别的玩家，直接丢弃
    if(race != g_PlayerRace[playerid]) return 0;

    // 版本号一致，正常处理
    if(cache_num_rows() == 0)
    {
        InsertPlayer(playerid);
        return 1;
    }

    cache_get_value_name_int(0, "gold",  g_Gold[playerid]);
    cache_get_value_name_int(0, "level", g_Level[playerid]);
    cache_get_value_name_int(0, "skin",  g_Skin[playerid]);

    SpawnPlayer(playerid);
    return 1;
}
```

---

### 这几行代码做了什么
```
玩家 海绵宝宝 连入（ID=5）  → g_PlayerRace[5] = 1，查询携带 race=1 发出
玩家 海绵宝宝 断线          → g_PlayerRace[5] = 2
玩家 派大星 连入（ID=5）    → g_PlayerRace[5] = 3
查询回来，race 1 ≠ g_PlayerRace[5] 3  → 丢弃，派大星不受影响
```

**养成习惯，所有玩家异步回调都加上这个检查**

---

## 附录：MySQL R41-4 完整函数列表

本教程只覆盖了最核心的几个函数。R41-4 还提供了更多功能，全部函数的说明和用法可以在官方 Wiki 查阅：

[https://github.com/pBlueG/SA-MP-MySQL/wiki](https://github.com/pBlueG/SA-MP-MySQL/wiki)

遇到具体需求时，去 Wiki 查一下，通常几分钟就能找到你需要的。

## 附录：MySQL 实用便利的查询语句示例

直接在数据库里做加减（不需要先读再写）
```sql
UPDATE `players` SET `gold` = `gold` + 100 WHERE `name` = 'Tom'
```

玩家下线时保存数据，但不确定他的记录存不存在，与其先 SELECT 判断再决定 INSERT 还是 UPDATE，不如一句搞定
```sql
INSERT INTO `players` (`name`, `gold`, `level`)
VALUES ('Tom', 100, 1)
ON DUPLICATE KEY UPDATE `gold` = VALUES(`gold`), `level` = VALUES(`level`)
```

LIMIT + OFFSET（分页）排行榜翻页时很有用，`OFFSET` 是跳过多少行，`LIMIT` 是取多少行：
```sql
-- 第一页（第 1-10 名）
SELECT `name`, `gold` FROM `players` ORDER BY `gold` DESC LIMIT 10 OFFSET 0

-- 第二页（第 11-20 名）
SELECT `name`, `gold` FROM `players` ORDER BY `gold` DESC LIMIT 10 OFFSET 10
```

统计服务器总注册人数、某个帮派的成员数等：
```sql
SELECT COUNT(*) AS `total` FROM `players`
SELECT COUNT(*) AS `total` FROM `players` WHERE `faction` = 3
```

SUM / MAX / MIN / AVG（计算相关语句）
```sql
-- 所有玩家总金币
SELECT SUM(`gold`) AS `total_gold` FROM `players`

-- 最高等级
SELECT MAX(`level`) AS `max_level` FROM `players`

-- 平均金币
SELECT AVG(`gold`) AS `avg_gold` FROM `players`
```

模糊搜索，管理员搜索玩家名字时很实用
```sql
SELECT `name`, `level` FROM `players` WHERE `name` LIKE '%Tom%'
-- % 是通配符，匹配任意字符
-- 'Tom%'  = 以 Tom 开头
-- '%Tom'  = 以 Tom 结尾
-- '%Tom%' = 包含 Tom
```

**这里就不一一列举了，总之功能很强大很便利，这是文本保存带来不了的巨大优势**


---

#GTA# #圣安地列斯# #侠盗猎车手# #圣安地列斯联机# #samp# #gta联机# #gtasa联机# #openmp# #omp# #open.mp# #gtasa#

社区交流群: 673335567

论坛: https://open-mp.cn/