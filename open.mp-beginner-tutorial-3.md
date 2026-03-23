# 零基础openmp/SAMP gtasa联机服务器开发教程 - 第三章(终篇)

编写：小鸟 [Github](https://github.com/XiaoNiaoa)
时间：2026/03/23

> 本篇是新手教程第二章的**基础巩固**部分，覆盖新手教程第二章中未涉及或一笔带过的语言细节。  
> 建议在读完新手教程第二章、能跑起来第一个脚本之后，再来阅读本篇。


老实说，Pawn 和 openmp/samp 能做的事，远比这新手教程三章展示的多得多。更复杂的数据结构、更多的回调和函数、插件生态、性能优化……这些东西没有讲，也不可能全部讲完。剩下的路，要靠你自己走

方法其实很简单：**想做一个功能，就去查它需要什么函数；看到别人的代码，就去读懂它在做什么。** 日积月累，你脑子里的「工具库」会越来越大，能做的事情也会越来越多。遇到看不懂的，查文档、搜索、问社区，这是每一个开发者每天都在做的事，你也不例外。

官方文档：[https://open.mp/docs](https://open.mp/docs)  

后续仍然会更新更多教程，但不是pawn新手教程系列

---

## 目录

| # | 章节 |
|---|------|
| 1 | [关键字：变量声明修饰符](#1-关键字变量声明修饰符) |
| 2 | [const — 不可修改的变量](#2-const--不可修改的变量) |
| 3 | [static — 静态变量](#3-static--静态变量) |
| 4 | [forward — 函数前向声明](#4-forward--函数前向声明) |
| 5 | [native — 原生函数声明](#5-native--原生函数声明) |
| 6 | [enum — 枚举](#6-enum--枚举) |
| 7 | [Tag（标签）— 类型系统](#7-tag标签--类型系统) |
| 8 | [编译指令 #define](#8-编译指令-define) |
| 9 | [编译指令 #include](#9-编译指令-include) |
| 10 | [编译指令 #if / #else / #endif](#10-编译指令-if--else--endif) |
| 11 | [编译指令 #pragma](#11-编译指令-pragma) |
| 12 | [运算符补充：sizeof](#12-运算符补充sizeof) |
| 13 | [运算符补充：三目运算符 ?:](#13-运算符补充三目运算符-) |
| 14 | [运算符补充：位运算](#14-运算符补充位运算) |
| 15 | [控制结构补充：do-while](#15-控制结构补充do-while) |
| 16 | [控制结构补充：break 与 continue](#16-控制结构补充break-与-continue) |
| 17 | [switch 进阶：范围与列表匹配](#17-switch-进阶范围与列表匹配) |
| 18 | [字符串函数](#18-字符串函数) |
| 19 | [玩家指令（Commands）](#19-玩家指令commands) |
| 20 | [sscanf — 解析指令参数](#20-sscanf--解析指令参数) |
| 21 | [printf 与 print — 控制台输出](#21-printf-与-print--控制台输出) |
| 22 | [代码组织：#include 拆分文件](#22-代码组织include-拆分文件) |

---

## 1. 关键字：变量声明修饰符

在 Pawn 里，声明变量和函数时可以在前面加上**修饰关键字**，控制它们的可见性和行为。

主教程只用了最基础的 `new`，这里把所有修饰符都介绍清楚：

| 关键字 | 用于 | 作用 |
|--------|------|------|
| `new` | 变量 | 声明一个普通变量（最常用） |
| `const` | 变量 | 声明后不能被修改 |
| `static` | 变量/函数 | 限制作用域 |
| `stock` | 函数/变量 | 未使用时不报警告 |
| `public` | 函数 | 可被服务端直接调用（回调） |
| `forward` | 函数 | 前向声明，让编译器预先知道函数存在 |
| `native` | 函数 | 声明由 C/C++ 实现的内置函数 |

---

## 2. const — 不可修改的变量

`const` 声明的变量在初始化后**不能再被修改**，编译器会在你试图修改时报错。

```pawn
const MAX_LEVEL = 50;
// MAX_LEVEL = 60;   ← 编译报错！不能修改 const 变量
```

### const 与 #define 的区别

```pawn
#define MAX_A  50      // 编译时文本替换，不占内存，不带类型
const   MAX_B = 50;    // 占内存，有类型，编译器会检查类型
```

在 Pawn 里，绝大多数情况下用 `#define` 更普遍，`const` 更常见于：

```pawn
// 函数参数中的 const 数组：告诉编译器这个数组在函数内部不会被修改
stock MyFunction(const string[])
{
    print(string);   // 只读，不修改
}
```

---

## 3. static — 静态变量

`static` 有两种用法：

### 用法一：局部 static 变量（函数内）

普通局部变量每次调用函数时都会重置为 0，而 `static` 局部变量**只初始化一次，之后保留上次的值**：

```pawn
// 普通局部变量：每次调用都从 0 开始
CountNormal()
{
    new i = 0;
    i++;
    printf("普通：%d", i);   // 永远输出 1
}

// static 局部变量：值会在调用之间保留
CountStatic()
{
    static iCount = 0;   // 只在第一次调用时初始化
    iCount++;
    printf("静态：%d", iCount);   // 第1次=1, 第2次=2, 第3次=3...
}
```

### 用法二：全局 static 变量（文件私有）

在函数外部声明的 `static` 全局变量，只能在**本文件**内访问，其他文件无法访问，同时还可以避免相同名称的变量冲突：

```pawn
// 只有本 .pwn 文件才能访问这个变量
static g_Value;
```

> **新手建议**：入门阶段不用刻意区分，`static` 局部变量是最常用到的，记住「值会保留」这个特点就够了。

---

## 4. forward — 函数前向声明

Pawn 编译器从上到下读取代码。如果函数 A 调用了函数 B，但 B 写在 A 的后面，编译器就会报错说找不到 B。

`forward` 告诉编译器：「这个函数存在，后面会定义它」。

### 什么时候必须用 forward？

**1. SetTimerEx 的目标函数**（最常见）

```pawn
// 必须在使用前 forward 声明
forward MyTimerFunc(playerid);

// ... 某处调用 ...
SetTimerEx("MyTimerFunc", 3000, false, "i", playerid);

// 函数的实际执行逻辑
public MyTimerFunc(playerid)
{
    SendClientMessage(playerid, -1, "3秒到了！");
    return 1;
}
```

**2. 相互调用的函数**

```pawn
forward FuncB();   // 先声明 B

FuncA()
{
    FuncB();   // 调用 B，编译器已经知道 B 存在了
}

FuncB()
{
    // ...
}
```

> **规则**：只要是 `public` 函数且通过字符串名称调用的（比如 Timer），**都必须 forward**。

---

## 5. native — 原生函数声明

`native` 声明的函数是由服务端或插件用 C/C++ 实现的函数，不是在 Pawn 里定义的。

`#include <open.mp>` 已经帮你声明好了所有内置的 native 函数，所以你平时直接调用 `SendClientMessage`、`CreateVehicle` 这些，底层都是 native。

**你通常不需要自己写 native 声明**，除非你在使用额外的插件，或者想给自定义函数加上 Pawno 的自动提示：

```pawn
// 声明一个插件提供的函数（例如 sscanf 插件）
native sscanf(const str[], const format[], ...);

// 利用 native 重命名内置函数（高级用法）
native old_print = print;
// 现在 print 不可用，改为用 old_print 调用
```

> **新手结论**：不需要手写 `native`，知道它是「C/C++ 实现的函数」就够了。

---

## 6. enum — 枚举

枚举是定义**一组有名字的整数常量**的方式，让代码更有可读性。

### 基础用法：替代一组 #define

```pawn
// 不用枚举的写法（容易混乱）
#define CLASS_PEASANT   0
#define CLASS_KNIGHT    1
#define CLASS_MERCHANT  2
#define CLASS_FISHER    3

// 用枚举（更整洁）
enum E_CLASS
{
    CLASS_PEASANT,    // 自动赋值 0
    CLASS_KNIGHT,     // 自动赋值 1
    CLASS_MERCHANT,   // 自动赋值 2
    CLASS_FISHER      // 自动赋值 3
}

new g_PlayerClass[MAX_PLAYERS];

public OnPlayerSpawn(playerid)
{
    g_PlayerClass[playerid] = CLASS_KNIGHT;
    return 1;
}
```

### 高级用法：用枚举定义结构化数组（最强用法）

这是 SA-MP/open.mp 脚本中最常见的枚举用途，可以把多种数据整合在一个二维数组里：

```pawn
// 定义玩家数据的"字段"
enum E_PLAYER_DATA
{
    PLAYER_GOLD,        // 金币
    PLAYER_LEVEL,       // 等级
    PLAYER_SKIN,        // 皮肤 ID
    PLAYER_KILLS,       // 击杀数
    PLAYER_DEATHS       // 死亡数
}

// 声明二维数组，第二维用枚举大小
new g_PlayerData[MAX_PLAYERS][E_PLAYER_DATA];

// 使用：用字段名代替数字下标，可读性极强
public OnPlayerConnect(playerid)
{
    g_PlayerData[playerid][PLAYER_GOLD]   = 50;
    g_PlayerData[playerid][PLAYER_LEVEL]  = 1;
    g_PlayerData[playerid][PLAYER_SKIN]   = 86;
    g_PlayerData[playerid][PLAYER_KILLS]  = 0;
    g_PlayerData[playerid][PLAYER_DEATHS] = 0;
    return 1;
}

// 读取
public OnPlayerDeath(playerid, killerid, reason)
{
    g_PlayerData[playerid][PLAYER_DEATHS]++;
    if(killerid != INVALID_PLAYER_ID)
    {
        g_PlayerData[killerid][PLAYER_KILLS]++;
        g_PlayerData[killerid][PLAYER_GOLD] += 10;   // 击杀奖励 10 金
    }
    return 1;
}
```

> 这种写法比单独维护 `g_Gold[]`、`g_Level[]`、`g_Skin[]` 多个数组更整洁，是进阶脚本的标准写法。

---

## 7. Tag（标签）— 类型系统

Pawn 是「弱类型」语言，所有变量本质上都是 32 位整数。  
**Tag（标签）** 是 Pawn 对类型的模拟，让编译器在你混用不同「类型」时发出警告。

### 常见的内置 Tag

| Tag | 含义 | 示例 |
|-----|------|------|
| （无 tag） | 普通整数 | `new iScore = 0;` |
| `Float:` | 浮点数 | `new Float:fX = 0.0;` |
| `bool:` | 布尔值 | `new bool:bAlive = true;` |
| `Text:` | TextDraw ID | `new Text:tdLogo;` |
| `PlayerText:` | 玩家 TextDraw | `new PlayerText:tdHUD;` |

### Tag Mismatch 警告

```pawn
new Float:health;
new score;

score = health;   // 警告：tag mismatch！把 Float 赋给整数
```

修复方法：用类型转换强制去除 tag：

```pawn
score = _:health;   // _: 去除 tag，变成普通整数再赋值（不推荐随意用）
score = floatround(health);   // 正确做法：用转换函数
```

### bool 类型

```pawn
new bool:g_IsAdmin[MAX_PLAYERS];

public OnPlayerConnect(playerid)
{
    g_IsAdmin[playerid] = false;   // 用 true/false，不用 0/1
    return 1;
}

// 判断
if(g_IsAdmin[playerid])
{
    SendClientMessage(playerid, -1, "你是管理员！");
}
```

---

## 8. 编译指令 #define

`#define` 是**预处理指令**，在代码编译前进行纯文本替换。

### 基础用法：定义常量

```pawn
#define MAX_GOLD    999
#define SKIN_KNIGHT 287
```

### 带参数的宏（函数式宏）

```pawn
// 定义一个带参数的宏，%0 是第一个参数，%1 是第二个，以此类推
#define IsValidPlayer(%0) ((%0) >= 0 && (%0) < MAX_PLAYERS && IsPlayerConnected(%0))

// 使用：
if(IsValidPlayer(playerid))
{
    SendClientMessage(playerid, -1, "有效的玩家");
}
// 实际展开为：
// if(((playerid) >= 0 && (playerid) < MAX_PLAYERS && IsPlayerConnected(playerid)))
```

### 主教程用过的按键宏

```pawn
// 这两个就是带参数的宏
#define PRESSED(%0)  (((newkeys) & (%0)) && !((oldkeys) & (%0)))
#define RELEASED(%0) (!((newkeys) & (%0)) && ((oldkeys) & (%0)))
```

### 注意事项

```pawn
// 宏是纯文本替换，不是函数，没有类型检查，要小心
#define DOUBLE(%0)  %0 * 2

new result = DOUBLE(3 + 4);
// 展开为：3 + 4 * 2 = 11，不是 14！
// 正确写法：
#define DOUBLE(%0)  ((%0) * 2)   // 用括号包裹，避免运算符优先级问题
```

---

## 9. 编译指令 #include

`#include` 把另一个文件的内容**插入到当前文件**，就像把那个文件的代码复制过来一样。

### 两种写法

```pawn
#include <open.mp>       // 尖括号：在系统 include 目录（qawno/include/）查找
#include "myfile"       // 引号：在当前目录查找
```

### 常见的 include 文件

```pawn
#include <open.mp>       // open.mp 全量头文件（必须）

// 常用插件头文件
#include <sscanf2>       // 字符串解析（指令参数处理）
#include <Pawn.CMD>      // 指令系统插件
#include <streamer>      // 动态对象流送插件
```

### 自定义 .inc 文件

当脚本变大时，可以把部分代码拆分到 `.inc` 文件里：

```pawn
// shop.inc 文件里写商店相关的函数
// 在主脚本里引入
#include "shop"
```

### 注意事项

如果文件格式是`.inc`可以直接写文件名，如果是其他格式比如 `.pwn`, `.dat`, `txt` 则需要写文件格式

比如: `#include "myfile.pwn"`

---

## 10. 编译指令 #if / #else / #endif

`#if` 是**编译时的条件判断**，决定某段代码要不要被编译进去。  
注意：这和运行时的 `if` 不同，`#if` 在编译阶段就生效。

### 常见用法：区分 GameMode 和 FilterScript

```pawn
// 根据是否定义了 FILTERSCRIPT，决定编译哪个初始化函数
#if defined FILTERSCRIPT
    public OnFilterScriptInit()
    {
        print("FilterScript 加载！");
        return 1;
    }
#else
    public OnGameModeInit()
    {
        print("GameMode 加载！");
        return 1;
    }
#endif
```

### 用 #if 开关调试模式

```pawn
// 在脚本顶部定义，需要调试时就定义它，发布时删掉
#define DEBUG_MODE

// 在代码中用 #if 检查
#if defined DEBUG_MODE
    print("[DEBUG] 服务器已以调试模式启动");
#endif

// 也可以在函数里用（但 #if 在函数内部也是编译时判断）
// 示例中如果没有定义 DEBUG_MODE，则 #if - #endif 内的代码等于不存在，不会有任何开销
public OnPlayerConnect(playerid)
{
    #if defined DEBUG_MODE
        new name[MAX_PLAYER_NAME];
        GetPlayerName(playerid, name, sizeof(name));
        printf("[DEBUG] 玩家连入: %s (ID: %d)", name, playerid);
    #endif
    return 1;
}
```

### defined 运算符

```pawn
// defined 检查某个符号是否被 #define 过
#if defined MY_CONSTANT
    // MY_CONSTANT 存在时编译这段
#else
    // 不存在时编译这段
#endif
```

---

## 11. 编译指令 #pragma

`#pragma` 给编译器传递特殊指令，控制编译行为。

### 最常用的几个

```pawn
// 关闭 tab 缩进警告
#pragma tabsize 0

// 关闭某个编号的警告
#pragma warning disable 200   // 关闭某个编号的警告

// 标记某个变量为「可能不会被用到，不要报警告」
new value;
#pragma unused value
```

---

## 12. 运算符补充：sizeof

`sizeof` 返回数组的**元素数量**（不是字节数），是 Pawn 特有的运算符。

```pawn
new arr[10];
printf("%d", sizeof(arr));   // 输出 10

new name[MAX_PLAYER_NAME];
printf("%d", sizeof(name));  // 输出 MAX_PLAYER_NAME 的值（24）
```

### 为什么要用 sizeof？

**避免硬编码数组大小**，让代码更安全：

```pawn
new message[128];

// 不好的写法：手动填 128，万一以后改了数组大小容易忘记改这里
format(message, 128, "你好，%s", name);

// 好的写法：用 sizeof 自动获取
format(message, sizeof(message), "你好，%s", name);
// 以后就算把 128 改成 256，这里也不需要动
```

### 在函数参数里传 sizeof

```pawn
GetPlayerName(playerid, name, sizeof(name));
//                                ↑ 告诉函数数组有多大，防止越界
```

---

## 13. 运算符补充：三目运算符 ?:

三目运算符是 `if/else` 的简写，适合简单的赋值判断：

### 基本语法

```pawn
// 格式：条件 ? 成立时的值 : 不成立时的值
new result = (mygold > 100) ? 1 : 0;
// 等同于：
// if(mygold > 100) result = 1;
// else            result = 0;
```

### 实际应用

```pawn
// 根据玩家是否在线，获取名字
new string[6];
format(string, sizeof(string), "%s", IsPlayerConnected(targetid) ? "在线" : "离线");

// 根据职业决定皮肤
SetPlayerSkin(playerid, ((g_PlayerClass[playerid] == CLASS_KNIGHT) ? 287 : 86));

// 嵌套三目
new levelName[16];
format(levelName, sizeof(levelName), "%s",
    level >= 10 ? "高级" :
    level >= 5  ? "中级" : "新手"
);
```

---

## 14. 运算符补充：位运算

位运算直接操作数字的二进制位，在 SA-MP/open.mp 中主要用于**按键检测**和**标志位**。

### 基础位运算符

| 运算符 | 名称 | 说明 |
|--------|------|------|
| `&` | 按位与 | 两位都是 1 才得 1 |
| `\|` | 按位或 | 任意一位是 1 就得 1 |
| `^` | 按位异或 | 两位不同才得 1 |
| `~` | 按位非 | 取反 |
| `<<` | 左移 | 相当于乘以 2 的 n 次方 |
| `>>` | 右移 | 相当于除以 2 的 n 次方 |

### 在按键检测中的应用

按键状态是一个整数，每一个「二进制位」代表一个按键是否被按下：

```pawn
public OnPlayerKeyStateChange(playerid, KEY:newkeys, KEY:oldkeys)
{
    // & 运算：检查某个按键位是否为 1（按下状态）
    if(newkeys & KEY_YES)
    {
        // KEY_YES 位是 1，说明 Y 键被按着
    }

    // PRESSED 宏就是用位运算实现的：
    // PRESSED(KEY_YES) 展开为：
    // ((newkeys & KEY_YES) && !(oldkeys & KEY_YES))
    // 意思是：现在按着 AND 之前没按 = 刚刚按下

    return 1;
}
```

### 用位运算存储多个布尔标志

```pawn
// 用一个整数存储多个开关状态（节省内存）
#define FLAG_ADMIN    (1 << 0)   // 二进制 0001
#define FLAG_VIP      (1 << 1)   // 二进制 0010
#define FLAG_MUTED    (1 << 2)   // 二进制 0100
#define FLAG_JAILED   (1 << 3)   // 二进制 1000

new g_Flags[MAX_PLAYERS];

// 设置标志（用 | 打开某位）
g_Flags[playerid] |= FLAG_ADMIN;

// 清除标志（用 & 和 ~ 关闭某位）
g_Flags[playerid] &= ~FLAG_MUTED;

// 检查标志（用 & 检测某位）
if(g_Flags[playerid] & FLAG_ADMIN)
{
    SendClientMessage(playerid, -1, "你是管理员");
}
```

---

## 15. 控制结构补充：do-while

`do-while` 是 `while` 的变体，区别是**先执行一次，再判断条件**。  
即使条件一开始就是假的，循环体至少也会执行一次。

```pawn
// while：先判断，可能一次都不执行
while(false)
{
    print("永远不会执行");
}

// do-while：先执行，再判断，至少执行一次
do
{
    print("至少执行一次！");
}
while(false);   // 注意：while 后面要加分号
```

### 实际应用

```pawn
// 寻找一个空的数据槽位（至少要进去看一次）
new slot = 0;
do
{
    if(g_SlotFree[slot])
    {
        break;   // 找到了，跳出
    }
    slot++;
}
while(slot < MAX_SLOTS);
```

> **新手提示**：`do-while` 在脚本里不太常用，了解即可。

---

## 16. 控制结构补充：break 与 continue

这两个关键字用于控制循环的流程。

### break — 立即跳出循环

```pawn
// 找到第一个死亡的玩家后停止循环
for(new i = 0; i < MAX_PLAYERS; i++)
{
    if(!IsPlayerConnected(i)) continue;   // 跳过未连接的（见下）

    new Float:health;
    GetPlayerHealth(i, health);

    if(health <= 0.0)
    {
        printf("玩家 %d 已死亡", i);
        break;   // 找到了就不用继续循环了
    }
}
```

### continue — 跳过本次，进入下一轮

```pawn
// 给所有在线玩家发消息，跳过管理员
for(new i = 0; i < MAX_PLAYERS; i++)
{
    if(!IsPlayerConnected(i)) continue;   // 没连接，跳过

    if(g_Flags[i] & FLAG_ADMIN) continue; // 是管理员，跳过

    SendClientMessage(i, -1, "非管理员消息");
}
```

### 在 switch 中不需要 break

> Pawn 的 `switch` 每个 `case` 是**独立的，不会穿透**到下一个 case，这与 C 语言不同，不需要写 `break`。

```pawn
// Pawn：不会穿透，不需要 break
switch(value)
{
    case 1: print("value 是 1");   // 执行后直接跳出 switch
    case 2: print("value 是 2");
    case 3: print("value 是 3");
}
```

---

## 17. switch 进阶：范围与列表匹配

Pawn 的 `switch` 比 C 语言更强大，支持**范围**和**列表**匹配。

### 列表匹配（多个值合并成一个 case）

```pawn
new level = 3;

switch(level)
{
    case 1, 2, 3:       // 匹配 1 或 2 或 3
    {
        SendClientMessage(playerid, -1, "初级段位");
    }
    case 4, 5, 6:       // 匹配 4 或 5 或 6
    {
        SendClientMessage(playerid, -1, "中级段位");
    }
    case 7, 8, 9, 10:
    {
        SendClientMessage(playerid, -1, "高级段位");
    }
}
```

### 范围匹配（连续区间）

```pawn
switch(level)
{
    case 1 .. 3:    // 匹配 1 到 3（含两端）
    {
        SendClientMessage(playerid, -1, "初级段位");
    }
    case 4 .. 6:    // 匹配 4 到 6
    {
        SendClientMessage(playerid, -1, "中级段位");
    }
    case 7 .. 10:
    {
        SendClientMessage(playerid, -1, "高级段位");
    }
    default:
    {
        SendClientMessage(playerid, -1, "段位超出范围");
    }
}
```

---

## 18. 字符串函数

Pawn 内置了很多处理字符串的函数，这里列出脚本开发中最常用的几个。

### strlen — 获取字符串长度

```pawn
new name[MAX_PLAYER_NAME];
GetPlayerName(playerid, name, sizeof(name));

new len = strlen(name);
printf("名字长度：%d", len);

// 常用：检查输入是否为空
if(strlen(inputtext) == 0)
{
    SendClientMessage(playerid, -1, "输入不能为空！");
    return 1;
}
```

### strcmp — 比较两个字符串

```pawn
// strcmp(str1, str2, ignorecase, length)
// 返回 0 = 相同，非 0 = 不同

new input[32] = "hello";

if(strcmp(input, "hello", true) == 0)   // true = 忽略大小写
{
    print("字符串匹配！");
}

// 常用简写（检查是否不同）
if(strcmp(input, "admin"))
{
    // 不相同时执行（非 0 = true）
}
```

### strcat — 拼接字符串

```pawn
new result[24] = "Hello, ";
strcat(result, "World!");
// result 现在是 "Hello, World!"
```

### strfind — 查找子字符串

```pawn
new string[] = "Welcome to the server!";
new pos = strfind(string, "server", true);   // 返回位置下标，找不到返回 -1
if(pos != -1)
{
    printf("找到了 server，位置：%d", pos);   // 输出 15
}
```

### 字符串与数字互转

```pawn
// 字符串 → 整数
new string[] = "123";
new number = strval(string);   // number = 123

// 整数 → 字符串（用 format）
new output[16];
format(output, sizeof(output), "%d", number);
```

---

## 19. 玩家指令（Commands）

玩家在聊天框输入 `/指令名` 时触发 `OnPlayerCommandText` 回调。  

### 方法一：直接用 strcmp（基础做法）

```pawn
new g_Gold[MAX_PLAYERS] = {100, ...};

public OnPlayerCommandText(playerid, cmdtext[])
{
    // cmdtext 是玩家输入的完整内容，例如 "/heal"

    if(strcmp(cmdtext, "/heal", true) == 0)
    {
        SetPlayerHealth(playerid, 100.0);
        SendClientMessage(playerid, -1, "血量已恢复！");
        return 1;   // ← 一定要 return 1，表示指令已处理
    }
    if(strcmp(cmdtext, "/gold", true) == 0)
    {
        SendClientMessage(playerid, -1, "你现在有 %d 枚金币。", g_Gold[playerid]);
        return 1;
    }

    // 没有匹配的指令：return 0 让服务器显示"未知指令"提示
    return 0;
}
```

### 方法二：用 Pawn.CMD 插件（推荐做法）

Pawn.CMD 是一个流行的指令处理插件，语法更简洁，性能更好：

自行下载安装: https://github.com/katursis/Pawn.CMD

```pawn
#include <Pawn.CMD>   // 需要安装 Pawn.CMD 插件

// Pawn.CMD 的写法：CMD:指令名(playerid, params[])
CMD:heal(playerid, params[])
{
    SetPlayerHealth(playerid, 100.0);
    SendClientMessage(playerid, -1, "血量已恢复！");
    return 1;
}

CMD:gold(playerid, params[])
{
    SendClientMessage(playerid, -1, "你现在有 %d 枚金币。", g_Gold[playerid]);
    return 1;
}
```

### 指令权限检查

```pawn
CMD:kick(playerid, params[])
{
    // 只有管理员才能用
    if(!(g_Flags[playerid] & FLAG_ADMIN))
    {
        SendClientMessage(playerid, -1, "你没有权限使用此指令！");
        return 1;
    }

    // ... 踢人逻辑 ...
    return 1;
}
```

---

## 20. sscanf — 解析指令参数

当指令带参数时（例如 `/kick 5` 或 `/give 3 100`），需要从字符串里把参数提取出来。  
`sscanf` 插件是处理这类问题的标准工具。

自行下载安装: https://github.com/Y-Less/sscanf

### 基础用法

```pawn
// sscanf(输入字符串, 格式, 变量1, 变量2, ...)
// 返回 0 = 解析成功，非 0 = 解析失败（参数不够或格式不对）

CMD:give(playerid, params[])
{
    new targetID, amount;

    // 格式 "ii" = 两个整数
    if(sscanf(params, "ii", targetID, amount))
    {
        SendClientMessage(playerid, -1, "用法：/give [玩家ID] [金币数]");
        return 1;
    }

    if(!IsPlayerConnected(targetID))
    {
        SendClientMessage(playerid, -1, "该玩家不在线！");
        return 1;
    }

    g_Gold[targetID] += amount;

    SendClientMessage(playerid, -1, "向玩家 %d 赠送了 %d 枚金币。", targetID, amount);
    SendClientMessage(targetID, -1, "管理员赠送了 %d 枚金币给你。", amount);
    return 1;
}

CMD:skin(playerid, params[])
{
    new skinID;

    if(sscanf(params, "i", skinID))
    {
        SendClientMessage(playerid, -1, "用法：/skin [皮肤ID]");
        return 1;
    }

    SetPlayerSkin(playerid, skinID);
    g_Skin[playerid] = skinID;

    SendClientMessage(playerid, -1, "皮肤已更换为 %d。", skinID);
    return 1;
}
```

### sscanf 格式说明符

| 格式符 | 含义 |
|--------|------|
| `i` | 整数 |
| `f` | 浮点数 |
| `s` | 字符串（到空格结束） |
| `s[32]` | 最大 32 字符的字符串 |
| `u` | 玩家名字或 ID（自动解析） |
| `p<分隔符>` | 设置分隔符，如 `p,` 用逗号分隔 |

### sscanf 相关文档说明

如何使用: https://github.com/Y-Less/sscanf?tab=readme-ov-file#use
格式说明符: https://github.com/Y-Less/sscanf?tab=readme-ov-file#specifiers

---

## 21. printf 与 print — 控制台输出

这两个函数把内容输出到**服务器控制台窗口**，玩家看不到，只能在服务器后台终端看到。  
常用于调试和记录日志。

### print — 输出纯文字

```pawn
print("服务器启动了！");
print("[DEBUG] 这是一条调试信息");
```

### printf — 带格式化输出

```pawn
new playerCounts = 0;
for(new i = 0; i < MAX_PLAYERS; i++)
    if(IsPlayerConnected(i)) playerCounts++;

printf("[服务器] 当前在线玩家：%d", playerCounts);
printf("[DEBUG] 玩家 %d 的金币：%d", playerid, g_Gold[playerid]);
```

### 格式符与 format 相同

```pawn
printf("整数：%d，浮点：%.2f，字符串：%s", 42, 3.14, "hello");
// 输出：整数：42，浮点：3.14，字符串：hello
```

---

## 22. 代码组织：#include 拆分文件

当你的脚本越来越大，所有代码都写在一个 `.pwn` 文件里会很难维护。 

按**功能模块**拆分成多个 `.inc` 文件，再在主文件里引入。

### 比如

```
gamemodes/
├── main.pwn            ← 主文件（只负责引入和框架）
└── include/
    ├── shop.inc        ← 商店系统
    ├── npc.inc         ← NPC 系统
    ├── vehicle.inc     ← 载具系统
    └── commands.inc    ← 所有指令
```

### 主文件写法

```pawn
// mian.pwn
#include <open.mp>

// 引入各功能模块
#include "include/shop.inc"
#include "include/npc.inc"
#include "include/vehicle.inc"
#include "include/commands.inc"

// 主文件只写框架回调
public OnGameModeInit()
{
    InitShop();       // 调用 shop.inc 里的初始化函数
    InitNPCs();       // 调用 npc.inc 里的初始化函数
    InitVehicles();   // 调用 vehicle.inc 里的初始化函数

    return 1;
}
```

### shop.inc 示例

```pawn
// 注意：这里不需要再 #include <open.mp>，主文件已经引入了

// 商店相关的变量、函数、指令都写在这里
InitShop()
{
    // 创建商店相关的 Pickup 等
    return 1;
}
```

### 进阶：ALS Hook 系统（让 .inc 自己挂载回调）

Pawn 不允许同一个回调定义两次，也就是说`OnGameModeInit` `OnPlayerDeath` 等等这些回调只能有一个

当模块越来越多，每次写新模块都要回去主文件里的各个回调中加入新模块的功能，容易忘记、切不容易维护管理。

让每个 `.inc` 文件自己管理自己需要使用的回调，执行完自己的逻辑后再把控制权传给下一个模块，形成一条链。

#### 原理

1. 我先把 `OnGameModeInit` **重命名**成 `shop_OnGameModeInit`（用 `#define` 宏替换）
2. 然后我自己写一个新的 `OnGameModeInit`，在里面先执行我的逻辑，再调用 `shop_OnGameModeInit`
3. 下一个 `.inc`（比如 `npc.inc`）接着同样操作，把我写的 `OnGameModeInit` 再重命名为 `npc_OnGameModeInit`，再写新的 `OnGameModeInit`...
4. 最终形成一条调用链：`OnGameModeInit` → `npc_OnGameModeInit` → `shop_OnGameModeInit` → 主文件的 `return 1`

```pawn
// shop.inc 文件内部

new g_ShopPickup;

InitShop()
{
    g_ShopPickup = CreatePickup(1242, 2, 100.0, 200.0, 10.0, -1);
    return 1;
}

public OnGameModeInit()
{
    InitShop();

    #if defined shop_OnGameModeInit
        return shop_OnGameModeInit();
    #else
        return 1;
    #endif
}
#if defined _ALS_OnGameModeInit
    #undef OnGameModeInit
#else
    #define _ALS_OnGameModeInit
#endif
#define OnGameModeInit shop_OnGameModeInit
#if defined shop_OnGameModeInit
    forward shop_OnGameModeInit();
#endif
```

#### 有了 ALS 之后，主文件什么都不用写

只需要#include进去，干净利落

```pawn
#include <open.mp>
#include "include/shop.inc"    
#include "include/npc.inc"     
#include "include/vehicle.inc" 

public OnGameModeInit()
{
    return 1;
}
```
---

#GTA# #圣安地列斯# #侠盗猎车手# #圣安地列斯联机# #samp# #gta联机# #gtasa联机# #openmp# #omp# #open.mp# #gtasa#

社区交流群: 673335567

论坛: https://open-mp.cn/