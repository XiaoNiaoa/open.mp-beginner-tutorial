# 零基础openmp/SAMP gtasa联机服务器开发教程 - 第二章

编写：小鸟 [Github](https://github.com/XiaoNiaoa)
时间：2026/03/23

> 本教程面向完全零基础的新手，每一节只讲一个知识点，循序渐进。  

> 本章节先以快速了解 Pawn 语言和 openmp 游戏实际应用为开端，让新手更快通过实际游戏表现获得反馈，以避免一开始就接触过多语法，建议阅读完本章节再开始第三章巩固进阶编程语言细节

---

## 目录

| # | 章节 |
|---|------|
| 1 | [open.mp 是什么](#1-openmp-是什么) |
| 2 | [开启服务器](#2-开启服务器) |
| 3 | [第一个脚本](#3-第一个脚本) |
| 4 | [注释](#4-注释) |
| 5 | [变量](#5-变量) |
| 6 | [define](#6-define) |
| 7 | [字符串与 format](#7-字符串与-format) |
| 8 | [运算符](#8-运算符) |
| 9 | [if / else 判断](#9-if--else-判断) |
| 10 | [switch 多分支](#10-switch-多分支) |
| 11 | [for 循环](#11-for-循环) |
| 12 | [while 循环](#12-while-循环) |
| 13 | [函数](#13-函数) |
| 14 | [数组](#14-数组) |
| 15 | [回调函数（Callback）](#15-回调函数callback) |
| 16 | [SendClientMessage — 发消息](#16-sendclientmessage--发消息) |
| 17 | [GameText — 屏幕大字](#17-gametext--屏幕大字) |
| 18 | [TextDraw — 屏幕 UI](#18-textdraw--屏幕-ui) |
| 19 | [Dialog — 对话框](#19-dialog--对话框) |
| 20 | [Pickup — 拾取物](#20-pickup--拾取物) |
| 21 | [Checkpoint — 检查点](#21-checkpoint--检查点) |
| 22 | [Vehicle — 载具](#22-vehicle--载具) |
| 23 | [Actor — NPC](#23-actor--npc) |
| 24 | [Timer — 计时器](#24-timer--计时器) |
| 25 | [Keys — 按键检测](#25-keys--按键检测) |

---

## 1. open.mp 是什么

**open.mp**（open multiplayer）是一个全新的 侠盗猎车手：圣安地列斯 的多人游戏服务端，完全向下兼容 **SA-MP** 。

你可以用 **Pawn 脚本语言** 来编写游戏服务器的逻辑，比如：

- 玩家加入时显示欢迎消息
- 在地图上放置 NPC、载具、拾取物
- 制作商店、任务、角色扮演系统

**Pawn** 是一门类似 C 语言的脚本语言，语法简单，非常适合入门。

> **open.mp vs SA-MP**：两者语法完全一样。open.mp 是活跃维护的现代版本。

---

## 2. 开启服务器

### 第一步：下载 open.mp 服务端

前往官网 [https://www.open.mp](https://www.open.mp) 下载最新版本，解压到任意文件夹。

解压后你会看到这样的目录结构：

```
components: open.mp 核心组件
filterscripts: 服务器脚本文件（辅助脚本）
gamemodes: 服务器游戏模式文件（主图）
models: 服务器自定义模型（纹理 .txd .dff）
plugins: 服务器插件文件（传统插件）
qawno: Pawn 编辑器程序及包含文件
scriptfiles: INI 配置文件及其他资源
bans.json: 封禁列表文件
config.json: 服务器配置文件
omp-server.exe: open.mp 服务器主程序
```

### 第二步：配置 config.json

打开 `config.json`，找到 `main_scripts` 这一行，改成你的主图名称 mygamemode

根据需求设置端口 服务器名称 需要添加的插件 辅助脚本 修改高强度的rcon密码

```json
{
    "name": "我的服务器",
    "network": {
        "port": 7777
    },
    "max_bots": 0,
    "max_players": 100,
    "pawn": {
        "main_scripts": [
            "mygamemode 1"
        ]
    },
    "rcon": {
        "password": "changeme1"
    },
}
```

关于config.json更多详情可查阅 https://open.mp/docs/server/config.json 但几乎大部分设置保持默认即可

### 第三步：打开 qawno 编辑器

双击 `qawno/qawno.exe`，这就是你写代码和编译脚本的地方。

> **提示**：推荐使用 VSCode，代码补全更强大。入门阶段用 qawno 就够了。

---

## 3. 第一个脚本

### 创建脚本文件

在 qawno 中，点击菜单 **File → New Blank**，然后输入以下代码：

```pawn
#include <open.mp>

main(){}

public OnGameModeInit()
{
    SetGameModeText("我的第一个open.mp服务器");
    AddPlayerClass(1, 2495.3547, -1688.2319, 13.6774, 351.1646, WEAPON_M4, 500, WEAPON_KNIFE, 1, WEAPON_COLORT45, 100);
	AddStaticVehicle(522, 2493.7583, -1683.6482, 12.9099, 270.8069, -1, -1);
    return 1;
}
```

### 保存并编译

点击 **File → Save**，保存为 `mygamemode.pwn`，保存到 `gamemodes/` 目录下。

然后按 **F5** 编译。

编译成功后，在同目录会生成 `mygamemode.amx` 文件。

### 启动服务器

双击 `omp-server.exe`，就成功启动服务器了。

### 代码解释

| 代码 | 含义 |
|------|------|
| `#include <open.mp>` | 引入 open.mp 的所有函数，必须写在第一行 |
| `public OnGameModeInit()` | 服务器启动时自动执行的函数 |
| `SetGameModeText("...")` | 设置服务器模式名称（显示在服务器列表） |
| `AddPlayerClass(...)` | 在职业选择中添加一个职业 用于让玩家可以以自己选择的皮肤出生|
| `AddStaticVehicle(...)` | 在游戏模式中添加静态车辆（模型会预加载给玩家） |
| `return 1;` | 函数结束，返回 1 表示成功 |

> **规则**：每条语句末尾必须有 **分号 `;`**，花括号 `{}` 表示代码块的开始和结束。

---

## 4. 注释

注释是写给人看的说明文字，**编译器会完全忽略它**，不影响程序运行。

写注释的好处是以后回看代码时会轻松很多。

### 单行注释

用 `//` 开头，从这里到行尾都是注释：

```pawn
// 这是一行注释，不会被执行
SetGameModeText("我的第一个open.mp服务器");  // 也可以写在代码后面
```

### 多行注释

用 `/* ... */` 包裹，可以跨越多行：

```pawn
/*
    这里是多行注释。
    可以写很多行。
    常用于说明。
*/
public OnGameModeInit()
{
    return 1;
}
```

> **建议**：新手可以给每个函数写一行注释说明它的用途，给复杂逻辑写注释说明思路。

---

## 5. 变量

变量是用来**存储数据**的容器，就像一个贴了标签的盒子。

### 声明变量

用 `new` 关键字声明变量：

```pawn
new myScore;         // 声明一个整数变量，初始值默认是 0
new myScore = 10;    // 声明并赋初始值
```

### 变量类型

**整数（默认类型）**

```pawn
new myAge = 25;
new myGold = 100;
new myScore = 0;
```

**浮点数（小数）**

浮点数需要在变量名前加 `Float:` 标签：

```pawn
new Float:myHealth = 100.0;
new Float:mySpeed  = 3.14;
new Float:myX      = 1234.5;   // 坐标通常是浮点数
```

### 变量命名规范

变量名只能包含字母、数字、下划线，且不能以数字开头，推荐：g_ 前缀表示全局变量

```pawn
new g_Score = 0;
new g_Speed = 0.0;
new g_Name[32];
```

> 这种前缀命名法不是强制要求，但能让代码更易读，虽然不是主流实践，但比较适用于pawn开发。

### 全局变量 vs 局部变量

```pawn
// 全局变量：写在所有函数外面，整个脚本都能访问，g_ 前缀表示全局（global）
new g_PlayerScore[MAX_PLAYERS];

public OnPlayerConnect(playerid)
{
    // 局部变量：写在函数内部，只在这个函数里有效，驼峰命名法‌（camelCase）：首词小写，后续大写
    new defaultScore = 50;
    g_PlayerScore[playerid] = defaultScore;
    return 1;
}
```

---

## 6. define

它本质上是一个文本替换符号，而不是一个真正的变量或常量，从使用效果看，它表现得像一个常量（运行过程中不能改变其数值），在编译时直接替换，不占用内存

常量常用于替代魔法数字，使代码更具可读性和可维护性，在这篇教程中我们暂时称它为常量。

### 定义常量

```pawn
#define SERVER_NAME     "我的第一个open.mp服务器"
#define MAX_GOLD        999
#define START_GOLD      50
#define PI              3.14159
```

### 如何使用

```pawn
public OnGameModeInit()
{
    SetGameModeText(SERVER_NAME);   // 等同于 SetGameModeText("我的第一个open.mp服务器");
    return 1;
}

public OnPlayerConnect(playerid)
{
    g_Gold[playerid] = START_GOLD;  // 等同于 g_Gold[playerid] = 50;
    return 1;
}
```

> **规范**：全部大写，单词间用下划线分隔，例如 `MAX_PLAYERS`、`SKIN_NORMAL`。

---

## 7. 字符串与 format

字符串就是一段文字。在 Pawn 中，字符串用**字符数组**存储。

### 声明字符串

```pawn
new g_Name[32];                 // 最多存 31 个字符（留一位给结束符）
new g_Msg[128];                 // 消息通常用 128 或 256
new g_Name[MAX_PLAYER_NAME];   // 用内置常量，等同于 g_Name[24]
```

### 获取玩家名字

```pawn
new g_Name[MAX_PLAYER_NAME];
GetPlayerName(playerid, g_Name, sizeof(g_Name)); // sizeof(g_Name) 自动计算数组大小，不需要手动填数字
```

### format — 格式化字符串

`format` 可以把变量的值拼进字符串里，类似其他语言的字符串模板：

```pawn
new string[128];
new name[MAX_PLAYER_NAME];
new gold = 100;

GetPlayerName(playerid, name, sizeof(name));

format(string, sizeof(string), "你好，%s！你有 %d 枚金币。", name, gold);
// 结果："你好，CJ！你有 100 枚金币。"
```

### 格式占位符

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `%s` | 字符串 | `"Hello, %s"` |
| `%d` | 整数 | `"金币：%d"` |
| `%f` | 浮点数 | `"血量：%.1f"` |
| `%i` | 整数（同 `%d`） | `"等级：%i"` |

> `%.1f` 表示保留 1 位小数，`%.2f` 保留 2 位，以此类推。

---

## 8. 运算符

### 算术运算符

```pawn
new a = 10;
new b = 3;

new add  = a + b;   // 13  加
new sub  = a - b;   // 7   减
new mul  = a * b;   // 30  乘
new div  = a / b;   // 3   除（整数除法，舍去小数）
new mod  = a % b;   // 1   取余（10 除以 3 余 1）
```

### 快捷赋值

```pawn
new gold = 100;
gold += 50;   // 等同于 gold = 100 + 50  →  150
gold -= 20;   // 等同于 gold = 100 - 20  →  80
gold *= 2;    // 等同于 gold = 100 * 2   →  200
gold /= 4;    // 等同于 gold = 100 / 4   →  25
```

### 自增 / 自减

```pawn
new i = 0;
i++;    // i 变成 1，等同于 i = i + 1
i--;    // i 变成 0，等同于 i = i - 1
```

### 比较运算符（返回真/假）

| 运算符 | 含义 |
|--------|------|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` | 大于 |
| `<` | 小于 |
| `>=` | 大于等于 |
| `<=` | 小于等于 |

```pawn
new gold = 100;

if(gold == 100)  { /* 如果gold等于100 则成立 */ }
if(gold > 50)    { /* 如果gold大于50 则成立 */ }
if(gold != 0)    { /* 如果gold不等于0 则成立 */ }
```

> **常见错误**：判断用 `==`（两个等号），赋值用 `=`（一个等号）。写成 `if(gold = 100)` 是赋值，不是判断！

### 逻辑运算符

| 运算符 | 含义 | 示例 |
|--------|------|------|
| `&&` | 且（两个都成立） | `gold > 0 && isAlive` |
| `\|\|` | 或（任意一个成立） | `isAdmin \|\| isVIP` |
| `!` | 非（取反） | `!IsPlayerConnected(i)` |

---

## 9. if / else 判断

`if / else` 根据条件决定执行哪段代码。

### 基础用法

```pawn
new gold = 80;

// 如果金币大于100
if(gold >= 100)
{
    SendClientMessage(playerid, 0xFFFF00FF, "金币充足，可以购买！");
}
// 否则
else
{
    SendClientMessage(playerid, 0xFF4444FF, "金币不足！");
}
```

### if / else if / else

```pawn
new myLevel = 5;

if(myLevel >= 10)
{
    SendClientMessage(playerid, -1, "你是高级玩家！");
}
else if(myLevel >= 5)
{
    SendClientMessage(playerid, -1, "你是中级玩家。");
}
else
{
    SendClientMessage(playerid, -1, "你是新手。");
}
```

---

## 10. switch 多分支

当你需要根据一个变量的**多个具体值**做不同处理时，`switch` 比一堆 `else if` 更清晰。

### 基础用法

```pawn
new g_Class = 2;

switch(g_Class)
{
    case 0:
    {
        SendClientMessage(playerid, -1, "你选择了：农民");
        SetPlayerSkin(playerid, 86);
    }
    case 1:
    {
        SendClientMessage(playerid, -1, "你选择了：骑士");
        SetPlayerSkin(playerid, 287);
    }
    case 2:
    {
        SendClientMessage(playerid, -1, "你选择了：商人");
        SetPlayerSkin(playerid, 19);
    }
    default:
    {
        // 以上都不匹配时
        SendClientMessage(playerid, -1, "未知职业");
    }
}
```

### 实际应用：处理对话框响应

```pawn
public OnDialogResponse(playerid, dialogid, response, listitem, inputtext[])
{
    if(dialogid == DIALOG_SHOP)
    {
        if(!response) return 1;   // 点了取消，不处理，等同于if(response == false)

        switch(listitem)
        {
            case 0: SetPlayerSkin(playerid, 86);   // 选择了对话框的第一个选项
            case 1: SetPlayerSkin(playerid, 287);  // 选择了对话框的第二个选项
            case 2: SetPlayerSkin(playerid, 19);   // 选择了对话框的第三个选项
        }
    }
    return 1;
}
```

> `switch` 中每个 `case` 后面的值必须是**整数或常量**，不能是字符串或浮点数。

---

## 11. for 循环

当你需要**重复执行**某段代码时，用循环。

`for` 循环适合已知重复次数的场景。

### 基础结构

```pawn
//   初始值; 条件; 每次循环后执行
for(new i = 0; i < 5; i++)
{
    // 这里的代码会执行 5 次（i = 0, 1, 2, 3, 4）
    print("循环中...(已循环%d次)", i + 1);
}
```

### 遍历所有在线玩家


```pawn
// 给所有在线玩家发送消息
for(new i = 0; i < MAX_PLAYERS; i++)
{
    if(IsPlayerNPC(i) == false && inGame[i] == true)
    {
        SendClientMessage(i, 0xFFFF00FF, "[公告] 比赛开始！");
        GivePlayerWeapon(i, WEAPON_AK47, 1500);
        SetPlayerPos(i, 0.0, 0.0, 2.5);
        // 给所有玩家加金币
        g_Gold[i] += 10;
    }
}
```

> 但通常历遍玩家我们会选择foreach库，更高效。  
> **注意**：`i < MAX_PLAYERS` 不能写成 `i <= MAX_PLAYERS`，否则会越界。  
> `MAX_PLAYERS` 是最大玩家数，有效的 playerid 是 `0` 到 `MAX_PLAYERS - 1`。

---

## 12. while 循环

`while` 循环适合**不知道要循环多少次、只知道循环条件**的场景。

### 基础结构

```pawn
new i = 0;
while(i < 5)
{
    print("执行中...");
    i++;     // 一定要记得让条件向终止靠近，否则会导致无限死循环！
}
```

### while vs for

| 场景 | 推荐 |
|------|------|
| 固定次数的重复（如遍历所有玩家） | `for` |
| 条件不满足就一直循环（如等待某个状态） | `while` |

> **警告**：循环体内一定要有让条件变化的语句，否则会进入**死循环**，导致服务器卡死。

---

## 13. 函数

函数是把一段代码**打包起来命名**，方便重复调用，避免重复写同样的代码。

### 定义函数

```pawn
// 函数名(参数列表)
// {
//     代码内容
//     return 返回值;
// }

GiveGold(playerid, amount)
{
    g_Gold[playerid] += amount;
    SendClientMessage(playerid, 0xFFDD44FF, "你获得了金币！");
    return 1;
}
```

### 调用函数

```pawn
public OnPlayerConnect(playerid)
{
    GiveGold(playerid, 50);    // 玩家连入时给 50 金币
    return 1;
}
```

### 带特定返回值的函数

```pawn
// 检查玩家金币是否足够，返回 true（足够）或 false（不足）
bool:HasEnoughGold(playerid, cost)
{
    if(g_Gold[playerid] >= cost)
    {
        return true;   // 足够
    }
    return false;       // 不足
}

// 使用：
if(HasEnoughGold(playerid, 100) == true)
{
    SendClientMessage(playerid, -1, "可以购买！");
}
```

### 不需要带任何返回值的函数

```pawn
void:ResetGold(playerid)
{
    g_Gold[playerid] = 0;
    SendClientMessage(playerid, 0xFFDD44FF, "你的金币被清空了");
}
```

### stock 函数

在 Pawn 中，如果你定义了函数但某些情况下没用到，编译器会发出警告。  
加上 `stock` 关键字可以消除这个警告：

```pawn
stock GetPlayerGold(playerid)
{
    return g_Gold[playerid];
}
```

> **原则**：把重复用到 3 次以上的代码封装成函数。函数名需要清晰描述它做什么，例如 `GiveGold`、`ShowShop`、`KickPlayer`。

---

## 14. 数组

数组是**一组同类型数据的集合**，用一个名字访问多个值。

### 声明数组

```pawn
new g_Scores[10];           // 存 10 个整数
new Float:g_Positions[3];   // 存 3 个浮点数（X, Y, Z）
new g_Name[MAX_PLAYER_NAME]; // 字符串本质也是字符数组
```

### 访问数组元素

数组下标从 **0** 开始：

```pawn
new g_Scores[5];

g_Scores[0] = 100;   // 第 1 个元素
g_Scores[1] = 85;    // 第 2 个元素
g_Scores[4] = 60;    // 第 5 个（最后一个）
// g_Scores[5]  ← 越界！不能访问，会报错
```

### 最常见：每个玩家存一份数据

```pawn
// 全局声明：每个玩家 ID 对应一个值
new g_Gold[MAX_PLAYERS];
new g_Level[MAX_PLAYERS];
new g_Skin[MAX_PLAYERS];

// 使用 playerid 作为下标访问对应玩家的数据
public OnPlayerConnect(playerid)
{
    g_Gold[playerid]  = 50;
    g_Level[playerid] = 1;
    g_Skin[playerid]  = 86;
    return 1;
}
```

### 二维数组

```pawn
// 存 5 个检查点的坐标（每个坐标有 X、Y、Z 三个值）
new Float:g_CpPos[5][3] = {
    {100.0, 200.0, 10.0},   // 检查点 0
    {150.0, 220.0, 10.0},   // 检查点 1
    {200.0, 240.0, 10.0},   // 检查点 2
    {250.0, 260.0, 10.0},   // 检查点 3
    {300.0, 280.0, 10.0}    // 检查点 4
};

// 访问检查点 2 的 Y 坐标：
new Float:y = g_CpPos[2][1];   // 240.0
```

---

## 15. 回调函数（Callback）

回调函数是 open.mp **服务端自动调用**的特殊函数。

你不需要手动调用它们，只需要在脚本里**定义**它们，当对应事件发生时，服务端会自动执行。

### 规律

所有回调函数都以 `public On` 开头：

```pawn
public OnGameModeInit()      // 服务器启动时
public OnGameModeExit()      // 服务器关闭时
public OnPlayerConnect()     // 玩家进入服务器时
public OnPlayerDisconnect()  // 玩家离开服务器时
public OnPlayerSpawn()       // 玩家出生时
public OnPlayerDeath()       // 玩家死亡时
public OnPlayerText()        // 玩家发送聊天消息时
```

### 示例：完整的玩家连入/断开流程

```pawn
public OnPlayerConnect(playerid)
{
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    SendClientMessageToAll(0x88FF88FF, "[服务器] %s 加入了游戏", name);
    return 1;
}

public OnPlayerDisconnect(playerid, reason)
{
    // reason: 0=超时  1=主动退出  2=被踢
    new name[MAX_PLAYER_NAME];
    GetPlayerName(playerid, name, sizeof(name));

    SendClientMessageToAll(0xAAAAAAFF, "[服务器] %s 离开了游戏", name);
    return 1;
}
```

> **返回值**：大多数回调函数返回 1 表示正常处理。返回 0 在某些回调里有特殊含义（如阻止某些行为）

---

## 16. SendClientMessage — 发消息

`SendClientMessage` 是向玩家的**聊天框**发送消息，是最基础的通知手段。

> **注意**：`SendClientMessage` 支持中文，不支持 emoji 表情符号。

### 语法

```pawn
SendClientMessage(playerid, color, "消息内容");
```

### 颜色格式（RGBA）

颜色用 8 位十六进制数表示：`0xRRGGBBAA`

| 参数 | 含义 |
|------|------|
| RR | 红色分量（00~FF） |
| GG | 绿色分量（00~FF） |
| BB | 蓝色分量（00~FF） |
| AA | 透明度（FF = 完全不透明） |

```pawn
SendClientMessage(playerid, 0xFFFFFFFF, "白色消息");
SendClientMessage(playerid, 0xFF4444FF, "红色消息");
SendClientMessage(playerid, 0x44FF44FF, "绿色消息");
SendClientMessage(playerid, 0xFFDD44FF, "金黄色消息");
SendClientMessage(playerid, -1,         "也是白色（-1 是快捷写法）");
```

### 颜色常量（推荐定义）

```pawn
// 在脚本顶部定义常用颜色
#define COLOR_WHITE   0xFFFFFFFF
#define COLOR_RED     0xFF4444FF
#define COLOR_GREEN   0x44FF44FF
#define COLOR_YELLOW  0xFFDD44FF
#define COLOR_GRAY    0xAAAAAAFF

// 使用时更直观
SendClientMessage(playerid, COLOR_GREEN, "[商店] 购买成功！");
SendClientMessage(playerid, COLOR_RED,   "[商店] 金币不足！");
```

### 发给所有玩家

```pawn
SendClientMessageToAll(COLOR_YELLOW, "[公告] 服务器将在 5 分钟后重启。");
```

### 在消息里嵌入变量

```pawn
SendClientMessage(playerid, COLOR_GREEN, "[商店] 购买成功！剩余金币：%d 枚", g_Gold[playerid]);
```

---

## 17. GameText — 屏幕大字

`GameText` 在玩家屏幕显示文字，会自动消失。  
适合用于：重要事件提示、区域名称、倒计时。

> **注意**：GameText **不支持中文**，只能写英文和符号。

### 语法

```pawn
GameTextForPlayer(playerid, "文字内容", 持续毫秒, 样式);
GameTextForAll("文字内容", 持续毫秒, 样式);   // 对所有玩家显示
```

### 颜色标签

在文字中插入颜色标签改变颜色：

```
~w~  白色      ~r~  红色      ~g~  绿色
~b~  蓝色      ~y~  黄色      ~p~  紫色
~n~  换行      ~h~  加亮
```

### 样式速查

https://open.mp/docs/scripting/resources/gametextstyles

### 示例

```pawn
// 进入区域时提示
GameTextForPlayer(playerid, "~y~Town Market", 3000, 3);

// 组合颜色与换行
GameTextForPlayer(playerid, "~g~Quest Complete!~n~~w~Reward: 100 Gold", 4000, 3);

// 全服公告
GameTextForAll("~r~Pirates~w~ are attacking!", 5000, 3);
```

---

## 18. TextDraw — 屏幕 UI

TextDraw 是**固定显示在屏幕上**的文字，不像 GameText 那样会消失。  
常用于：服务器 Logo、血量 HUD、分数显示。

### 详细资料

https://open.mp/docs/scripting/resources/TextDraws

### 创建一个右上角 Logo

**第一步：声明变量**

```pawn
new Text:g_MyText;   // Text: 是 TextDraw 的类型标签
```

**第二步：在 OnGameModeInit 里创建**

```pawn
public OnGameModeInit()
{
    // x, y坐标是基于640x480画布（与屏幕分辨率无关）
    g_MyText = TextDrawCreate(240.0, 580.0, "Welcome to my OPEN.MP server");
    return 1;
}
```

**第三步：在 OnPlayerSpawn 里显示给玩家**

```pawn
public OnPlayerSpawn(playerid)
{
    TextDrawShowForPlayer(playerid, g_MyText);
    return 1;
}
```

**玩家断开时隐藏**

```pawn
public OnPlayerDisconnect(playerid, reason)
{
    TextDrawHideForPlayer(playerid, g_MyText);
    return 1;
}
```

### 字体样式说明

https://open.mp/docs/scripting/functions/TextDrawFont

---

## 19. Dialog — 对话框

Dialog 是弹出在玩家屏幕上的**交互窗口**，是最核心的 UI 交互方式。

### 对话框类型

https://open.mp/docs/scripting/resources/dialogstyles

### 第一步：定义对话框 ID

```pawn
// 每个对话框需要一个唯一 ID，用常量管理
#define DIALOG_NULL 0
#define DIALOG_LOGIN 1
#define DIALOG_WELCOME 2
#define DIALOG_WEAPONS 3
```

```pawn
// 你也可以用枚举，他会自动分配ID，第一个是0 后面自动递增
enum {
    DIALOG_NULL,        // 0
    DIALOG_LOGIN,       // 1
    DIALOG_WELCOME,     // 2
    DIALOG_WEAPONS      // 3
}
```

### 第二步：显示对话框

```pawn
// ShowPlayerDialog(玩家ID, 对话框ID, 样式, 标题, 内容, 按钮1, 按钮2)
// 对话框样式示例: DIALOG_STYLE_MSGBOX:
ShowPlayerDialog(playerid, DIALOG_WELCOME, DIALOG_STYLE_MSGBOX, "提示", "你已进入服务器", "关闭", "");

// 对话框样式示例: DIALOG_STYLE_INPUT:
ShowPlayerDialog(playerid, DIALOG_LOGIN, DIALOG_STYLE_INPUT, "登录", "请在下方输入密码登陆服务器:", "登录", "取消");

// 对话框样式示例: DIALOG_STYLE_LIST:
ShowPlayerDialog(playerid, DIALOG_WEAPONS, DIALOG_STYLE_LIST, "武器", "AK47\nM4\n狙击枪", "选择", "关闭");

// 对话框样式示例: DIALOG_STYLE_PASSWORD:
ShowPlayerDialog(playerid, DIALOG_LOGIN, DIALOG_STYLE_PASSWORD, "登录", "请在下方输入密码登陆服务器:", "登录", "取消");

// 对话框样式示例: DIALOG_STYLE_TABLIST:
ShowPlayerDialog(playerid, DIALOG_WEAPONS, DIALOG_STYLE_TABLIST, "武器商店", "沙鹰\t$5000\t100\n短管散弹枪\t$5000\t100\n手枪\t$1000\t50", "购买", "取消");

// 对话框样式示例: DIALOG_STYLE_TABLIST_HEADERS:
ShowPlayerDialog(playerid, DIALOG_WEAPONS, DIALOG_STYLE_TABLIST_HEADERS, "武器商店", "武器\t价格\t弹药数量\n沙鹰\t$5000\t100\n短管散弹枪\t$5000\t100\n手枪\t$1000\t50", "购买", "取消");
```

> `\n` 是换行符，在 `DIALOG_STYLE_LIST` 里每个 `\n` 分隔一个选项。

### 第三步：处理玩家的响应

```pawn
public OnDialogResponse(playerid, dialogid, response, listitem, inputtext[])
{
    // response  = 1 点了左边按钮
    //           = 0 点了右边按钮或按了 ESC
    // listitem  = 玩家选的是第几行（从 0 开始）
    // inputtext = 玩家输入的文字

    if (dialogid == DIALOG_RULES)
    {
        if (response) // 如果玩家点击了左边的按钮
        {
            SendClientMessage(playerid, COLOR_GREEN, "你点击了左边的按钮!");
        }
        else // 否则(玩家点击了右边的按钮或按下ESC键)
        {
            SendClientMessage(playerid, COLOR_GREEN, "你点击了右边的按钮!");
        }
        return 1; // 返回停止后续代码的执行
    }
    return 0;
}
```

---

## 20. Pickup — 拾取物

Pickup 是放在地图上的**可触发拾取物**，玩家走触碰到之后会自动触发，不需要按键。  
常用于：金币、道具、任务触发点、商店入口。

### 创建 Pickup

```pawn
new g_pickupGold;   // 保存 pickup 的 ID，响应时用来判断

public OnGameModeInit()
{
    // CreatePickup(模型ID, 类型, X, Y, Z, 虚拟世界)
    // 类型 2 = 拾取后消失，15 秒后重生
    g_pickupGold = CreatePickup(1274, 2, 2009.8658, 1220.0293, 10.8206, -1);
    //                          ↑模型 ↑类型   ↑坐标                     ↑-1=所有世界可见

    return 1;
}
```

### 常用模型 ID

| 模型 ID | 外观 |
|---------|------|
| 1210 | 公文包 |
| 1212 | 金钱 |
| 1239 | 信息 |
| 1240 | 爱心 |
| 1273 | 绿色房子 |

更多可查阅: https://open.mp/docs/scripting/resources/pickupids

### 响应拾取事件

```pawn
public OnPlayerPickUpPickup(playerid, pickupid)
{
    if(pickupid == g_pickupGold)
    {
        g_Gold[playerid] += 20;
        SendClientMessage(playerid, COLOR_YELLOW, "捡到了金币！+20");
        GameTextForPlayer(playerid, "~y~+20 Gold", 1500, 5);
    }
    return 1;
}
```

### Pickup 类型说明

| 类型值 | 行为 |
|--------|------|
| 0 | 无特殊属性，不可被拾取 |
| 1 | 永久存在 仅允许脚本控制 |
| 14 | 仅限车辆拾取，触发检查点音效后消失 |

更多类型可查阅: https://open.mp/docs/scripting/resources/pickuptypes

---

## 21. Checkpoint — 检查点

Checkpoint 是地图上的**红色发光圆柱区域**，玩家走进去自动触发。  
常用于：任务目标点、区域进入触发、引导新玩家，每个玩家最多只能显示一个检查点。

当玩家进入检查点时，会调用 `OnPlayerEnterCheckpoint` 回调

### 设置检查点

```pawn
// 每个玩家同时只能有一个活动的 Checkpoint
// SetPlayerCheckpoint(玩家ID, X, Y, Z, 检查点尺寸半径)
SetPlayerCheckpoint(playerid, 1982.6150, -220.6680, -0.2432, 3.0);
```

### 响应进入事件

```pawn
public OnPlayerEnterCheckpoint(playerid)
{
    SendClientMessage(playerid, COLOR_GREEN, "到达目标地点！");

    // 触发后通常需要清除，避免反复触发
    DisablePlayerCheckpoint(playerid);

    // 在这里做你想做的事，比如给与玩家金钱
    GivePlayerMoney(playerid, 1000);

    return 1;
}
```

### RaceCheckpoint — 比赛检查点

当玩家进入时，会调用 `OnPlayerEnterRaceCheckPoint` 回调

```pawn
// SetPlayerRaceCheckpoint(玩家ID, 类型, 当前X,Y,Z, 下一目标X,Y,Z, 半径)
SetPlayerRaceCheckpoint(playerid, CP_TYPE_GROUND_NORMAL, 644.3091, 1767.0223, 4.9970, 650.6734, 1812.0367, 4.9970, 3.0);

public OnPlayerEnterRaceCheckpoint(playerid)
{
    DisablePlayerRaceCheckpoint(playerid);
    SendClientMessage(playerid, COLOR_GREEN, "到达！");
    return 1;
}
```

---

## 22. Vehicle — 载具

### 创建载具

```pawn
// CreateVehicle(模型, X, Y, Z, 角度, 颜色1, 颜色2, 重生延迟秒)
new vehicleid = CreateVehicle(411, 100.0, 200.0, 10.0, 90.0, -1, -1, 60);
// -1 = 随机颜色，60 = 无人使用 60 秒后重生
```

### 让玩家进入载具

```pawn
// PutPlayerInVehicle(玩家ID, 载具ID, 座位)
// 座位 0 = 驾驶座，1 = 副驾驶，2/3 = 后排
PutPlayerInVehicle(playerid, vehicleid, 0);
```

### 载具模型 ID

请查阅: https://open.mp/docs/scripting/resources/vehicleid

### 进出载具回调

```pawn
public OnPlayerEnterVehicle(playerid, vehicleid, ispassenger)
{
    // ispassenger: 0 = 驾驶座，1 = 乘客
    if(!ispassenger)
    {
        SendClientMessage(playerid, COLOR_WHITE, "你正在进入载具的驾驶位。");
    }
    return 1;
}

public OnPlayerExitVehicle(playerid, vehicleid)
{
    SendClientMessage(playerid, COLOR_WHITE, "你正在离开载具");
    return 1;
}
```

---

## 23. Actor — NPC

Actor 是**站在世界里的静态 NPC 角色**，可以设置外观和动作动画，但不会移动，但功能有限

### 创建 Actor

```pawn
new g_ActorSmith;   // 保存 actor ID

public OnGameModeInit()
{
    // CreateActor(皮肤ID, X, Y, Z, 朝向角度)
    g_ActorSmith = CreateActor(179, 316.1, -134.0, 999.6, 90.0);

    return 1;
}
```

### 给 Actor 播放动画

```pawn
// ApplyActorAnimation(actorID, 动画库, 动画名, 速度, 循环, lockX, lockY, 冻结, 时长)
ApplyActorAnimation(g_ActorSmith, "DEALER", "shop_pay", 4.1, false, false, false, false, 0);
```

### 玩家靠近时按键交互

```pawn
// 工具函数：计算玩家到 Actor 的距离
stock Float:DistToActor(playerid, actorid)
{
    new Float:px, Float:py, Float:pz;
    new Float:ax, Float:ay, Float:az;
    GetPlayerPos(playerid, px, py, pz);
    GetActorPos(actorid, ax, ay, az);
    return floatsqroot((px-ax)*(px-ax)+(py-ay)*(py-ay)+(pz-az)*(pz-az));
}

// 在按键回调里检测距离并触发对话
public OnPlayerKeyStateChange(playerid, KEY:newkeys, KEY:oldkeys)
{
    if(PRESSED(KEY_YES))
    {
        if(DistToActor(playerid, g_ActorSmith) < 3.0)
        {
            ShowPlayerDialog(playerid, DIALOG_SMITH, DIALOG_STYLE_MSGBOX,
                "[ 武器商人 ]",
                "欢迎！我能为你提供精良的武器。",
                "好的", ""
            );
        }
    }
    return 1;
}
```

### 服务器关闭时清理

```pawn
public OnGameModeExit()
{
    DestroyActor(g_ActorSmith);
    return 1;
}
```

---

## 24. Timer — 计时器

Timer 用于**延迟执行**或**定期执行**某个函数，是实现游戏事件、冷却时间、倒计时的基础工具。

### 创建计时器

```pawn
// 一次性：2000 毫秒（2 秒）后执行，false = 不重复
SetTimer("MyFunction", 2000, false);

// 重复：每 5000 毫秒（5 秒）执行一次，true = 重复
new g_Timer = SetTimer("MyRepeatFunc", 5000, true);

// 停止计时器（需要提前保存它的 ID）
KillTimer(g_Timer);
```

### 带参数的计时器

```pawn
// SetTimerEx("函数名", 间隔ms, 是否重复, "参数格式", 参数值...)
SetTimerEx("SendDelayedMsg", 3000, false, "i", playerid);

// 对应函数必须先用 forward 声明
forward SendDelayedMsg(playerid);
public SendDelayedMsg(playerid)
{
    if(!IsPlayerConnected(playerid)) return 0;
    GameTextForPlayer(playerid, "~w~Welcome to~n~~y~Island Town!", 3000, 3);
    return 1;
}
```

### 参数格式字符

| 格式符 | 含义 |
|--------|------|
| `i` | 整数（int） |
| `f` | 浮点数（float） |
| `s` | 字符串 |

### 实际应用：每 60 秒播报随机事件

```pawn
new g_EventTimer;

public OnGameModeInit()
{
    g_EventTimer = SetTimer("IslandEvent", 60000, true);
    return 1;
}

public OnGameModeExit()
{
    KillTimer(g_EventTimer);   // 服务器关闭时记得停止
    return 1;
}

forward IslandEvent();
public IslandEvent()
{
    switch(random(3))
    {
        case 0: SendClientMessageToAll(COLOR_ORANGE, "[小镇] 市集开放！来广场逛逛。");
        case 1: SendClientMessageToAll(COLOR_RED,    "[小镇] 海湾发现海盗船！");
        case 2: SendClientMessageToAll(COLOR_GREEN,  "[小镇] 今日好天气，渔获翻倍！");
    }
    return 1;
}
```

> **注意**：使用 `SetTimerEx` 的函数**必须**在使用前加 `forward` 声明，否则编译会报错或运行异常。

---

## 25. Keys — 按键检测

`OnPlayerKeyStateChange` 在玩家**按下或松开按键**时触发，用于实现交互操作。

官方文档: https://open.mp/docs/scripting/callbacks/OnPlayerKeyStateChange

### 基础宏定义

```pawn
// 在脚本顶部定义这两个宏，后面判断按键时更方便
#define PRESSED(%0)  (((newkeys) & (%0)) && !((oldkeys) & (%0)))
#define RELEASED(%0) (!((newkeys) & (%0)) && ((oldkeys) & (%0)))
```

### 基础用法

```pawn
public OnPlayerKeyStateChange(playerid, KEY:newkeys, KEY:oldkeys)
{
    // PRESSED = 刚按下（本帧按着，上帧没按）
    if(PRESSED(KEY_FIRE))
    {
        SendClientMessage(playerid, COLOR_WHITE, "你按了鼠标左键");
    }

    // RELEASED = 刚松开
    if(RELEASED(KEY_FIRE))
    {
        SendClientMessage(playerid, COLOR_WHITE, "你松开了鼠标左键");
    }

    return 1;
}
```

### 按键常量

请查阅: https://open.mp/docs/scripting/resources/keys

### 完整交互示例：按 Y键 与 NPC 对话

```pawn
#define PRESSED(%0) (((newkeys) & (%0)) && !((oldkeys) & (%0)))

public OnPlayerKeyStateChange(playerid, newkeys, oldkeys)
{
    if(PRESSED(KEY_YES))
    {
        // 检查玩家是否靠近铁匠 NPC
        if(DistToActor(playerid, g_ActorSmith) < 3.0)
        {
            ShowPlayerDialog(playerid, DIALOG_SMITH, DIALOG_STYLE_MSGBOX,
                "[ 武器商人 ]",
                "你好，旅人！需要武器吗？",
                "需要", "不了"
            );
        }
    }
    return 1;
}
```

---

#GTA# #圣安地列斯# #侠盗猎车手# #圣安地列斯联机# #samp# #gta联机# #gtasa联机# #openmp# #omp# #open.mp# #gtasa#

社区交流群: 673335567

论坛: https://open-mp.cn/