# SAMP 迁移到 openmp 完整指南 + pawn 编译报错全解析

---

> 本篇适合：
> - 已有 SA-MP 服务器，想迁移到 open.mp 的开发者
> - 在编译时遇到错误不知道怎么处理的新手

---

## 第一部分：迁移步骤

### 第一步：下载 open.mp 服务端

前往 [https://github.com/openmultiplayer/open.mp/releases](https://github.com/openmultiplayer/open.mp/releases) 下载最新版本。

- `open.mp-win-x86.zip` — Windows 服务器
- `open.mp-linux-x86.tar.gz` — Linux 服务器

### 第二步：解压

目录结构
```
omp-server/
├── components/      ← open.mp 原生组件（某些插件放这里而不是 plugins）
├── filterscripts/   ← 过滤脚本（辅助脚本）
├── gamemodes/       ← 游戏模式（主脚本 .amx 放这里）
├── models/          ← 自定义模型文件（.txd .dff）
├── plugins/         ← 旧版插件（.dll 或 .so）
├── qawno/           ← Pawn 编辑器 + include 头文件
│   └── include/     ← 把 .inc 头文件放这里
├── scriptfiles/     ← INI 文件等资料
├── bans.json        ← 封禁列表
├── config.json      ← 服务器配置（替代 SA-MP 的 server.cfg）
└── omp-server.exe   ← 服务器主程序
```

### 第三步：放置脚本插件文件

- 把你的游戏模式 `.pwn` 源文件放入 `gamemodes/` 文件夹
- 把所需的 `.inc` 头文件（如 `sscanf2.inc`、`streamer.inc`）放入 `qawno/include/`
- 把插件文件（如 `streamer.dll`）放入 `plugins/`

> **YSI 用户**：需要升级到 [YSI-5.x](https://github.com/pawn-lang/YSI-Includes/releases)，YSI-4 与 open.mp 不兼容。

> **FCNPC 用户**：FCNPC 插件已被官方 open.mp NPC 组件取代，转换代码即可使用。

> **YSF 用户**：open.mp 已内置了大多数 YSF 的原生函数，不再需要 YSF。

**注意**：有些插件需要放进 `components/` 而不是 `plugins/`，具体查阅以下清单：

#### 常见插件

| 插件 | 状态 | 备注 |
|------|------|------|
| **Streamer** | ✅ 兼容 | 放 `plugins/`，正常加载 |
| **MySQL R41-4** | ✅ 兼容 | 放 `plugins/`，正常加载 |
| **CrashDetect** | ✅ 兼容 | 开发调试，放 `plugins/` |
| **Whirlpool** | ✅ 兼容 | 放 `plugins/` 但不建议使用 |
| **PawnPlus** | ✅ 兼容 | 放 `plugins/`，open.mp 对其有专项优化 |
| **pawn-memory** | ✅ 兼容 | 放 `plugins/` |
| **JIT** | ✅ 兼容 | 代码稳定后可用来提升性能，放 `plugins/` |
| **Profiler** | ✅ 兼容 | 性能分析工具，放 `plugins/` |
| **FileManager** | ✅ 兼容 | 放 `plugins/` |
| **sscanf** | 📦 放 components/ | 使用 [sscanf 2.13.8](https://github.com/Y-Less/sscanf/releases/tag/v2.13.8) |
| **Pawn.CMD** | 📦 放 components/ | 使用 [3.4.0-omp](https://github.com/katursis/Pawn.CMD/releases/tag/3.4.0-omp) |
| **Pawn.RakNet** | 📦 放 components/ | 使用 [1.6.0-omp](https://github.com/katursis/Pawn.RakNet/releases/tag/1.6.0-omp) |
| **sampvoice** | 📦 放 components/ | 使用 [v3.1.5-omp](https://github.com/AmyrAhmady/sampvoice/releases/tag/v3.1.5-omp) |
| **discord-connector** | 📦 放 components/ | 使用 [v0.3.6-pre](https://github.com/maddinat0r/samp-discord-connector/releases/tag/v0.3.6-pre) |
| **rustext** | ⚠️ 需升级 | 使用 [v2.0.11-nomemhack](https://github.com/ziggi/rustext/releases/tag/v2.0.11) |
| **keylistener** | ⚠️ 需升级 | 使用 [1.1.2-pr](https://github.com/edgyaf/keylistener/releases/tag/1.1.2-pr) |
| **BCrypt** | ✅ 兼容 | [samp-bcrypt](https://github.com/Sreyas-Sreelal/samp-bcrypt)，密码哈希推荐方案 |
| **SHA256_PassHash** | 🔁 有替代 | open.mp 中已标记为废弃，改用 BCrypt |
| **nativechecker** | 🔁 有替代 | open.mp 已内置 native 检查机制，不再需要 |
| **YSF** | 🔁 有替代 | open.mp 已内置大多数 YSF 原生函数，不再需要 |
| **SKY** | 🔁 有替代 | 改用 **Pawn.RakNet** |
| **中文名插件** | 🔁 有替代 | open.mp 有内置函数可实现兼容中文昵称 |
| **FCNPC** | ❌ 不兼容 | open.mp 官方内置 NPC 组件，性能更好，不再需要它 |

---

- plugins/    ← 旧版（legacy）插件放这里，在 config.json 的 legacy_plugins 里加载
- components/ ← open.mp 原生组件放这里，服务器启动时自动加载，无需配置

以上资料来源个人经验和 open.mp/docs/server/Installation

---

### 第四步：修改 #include

打开你的 `.pwn` 文件，把第一行：

```pawn
#include <a_samp>
```

改为：

```pawn
#include <open.mp>
```

然后按 **F5** 编译。

### 第五步：配置 config.json

用记事本打开 `config.json`，按需修改：

**加载主图：**

```json
"main_scripts": [
    "你的脚本文件名 1"
]
```

**加载插件：**

```json
"legacy_plugins": [
    "mysql",
    "sscanf",
    "streamer"
]
```

**加载过滤脚本：**

```json
"side_scripts": [
    "filterscripts/你的过滤脚本名"
]
```

**设置 RCON 密码：**

```json
"rcon": {
    "allow_teleport": false,
    "enable": false,
    "password": "你的强密码"
}
```

> 如何把旧的 `server.cfg` 转换为 `config.json`，参考官方文档：https://www.open.mp/docs/server/config.json

### 第六步：启动服务器

**Windows：** 双击 `omp-server.exe`

**Linux：**
```bash
chmod +x omp-server
./omp-server
```

---

## 第二部分：SA-MP 到 open.mp 的变化

有以下几个变化

---

### 变更一：拼写统一为英式英语

open.mp 将所有函数名统一为英式拼写，`Color` 改为 `Colour`，例如：

```pawn
// SA-MP 写法（会报 warning）
TextDrawBoxColor(textid, 0xFF0000FF);
TextDrawColor(textid, 0xFFFFFFFF);

// open.mp 正确写法
TextDrawBoxColour(textid, 0xFF0000FF);
TextDrawColour(textid, 0xFFFFFFFF);
```

**批量替换方法**：在你的编辑器里按 **Ctrl+H**，把所有 `.....Color` 替换为 `.....Colour`。

如果不想逐个修改，可以在 include 顶部加上：

```pawn
#define MIXED_SPELLINGS
#include <open.mp>
```

---

### 变更二：GetPlayerPoolSize 等函数已移除

`GetPlayerPoolSize()`、`GetVehiclePoolSize()`、`GetActorPoolSize()` 这三个函数在 open.mp 中已被移除，因为它们本身就存在 bug（返回的是最高 ID 而不是数量，而且没有玩家时会返回错误数据）。

```pawn
// 错误：这些函数不存在了
for(new i = 0; i <= GetPlayerPoolSize(); i++) { }

// 正确：直接用 MAX_PLAYERS
for(new i = 0; i < MAX_PLAYERS; i++)
{
    if(IsPlayerConnected(i)) { }
}

// 或者用 foreach（需要 YSI）
foreach(new i : Player) { }
```

同理：
- `GetPlayerPoolSize()` → 换成 `MAX_PLAYERS`
- `GetVehiclePoolSize()` → 换成 `MAX_VEHICLES`
- `GetActorPoolSize()` → 换成 `MAX_ACTORS`

---

### 变更三：死亡不再自动扣 100 元

SA-MP 中玩家死亡后会自动扣除 100 元（住院费）。open.mp 移除了这个机制，让脚本自己管理金钱。

如果你的脚本之前为了「修复」这个扣钱机制，在 `OnPlayerDeath` 或 `OnPlayerSpawn` 里手动加了 `GivePlayerMoney(playerid, 100)`，现在**应该删掉这行**，否则玩家死亡后反而会多 100 元。

如果你的脚本确实依赖这个「死亡扣钱」功能，在 `OnPlayerDeath` 里手动加上：

```pawn
public OnPlayerDeath(playerid, killerid, WEAPON:reason)
{
    GivePlayerMoney(playerid, -100); // 手动模拟死亡扣款
    return 1;
}
```

---

### 变更四：HideMenuForPlayer 行为修正

SA-MP 中 `HideMenuForPlayer` 会忽略你传入的菜单 ID，直接关掉玩家当前打开的任意菜单。

open.mp 修正了这个行为，现在它只会关闭你指定的那个菜单。

```pawn
// SA-MP 的老写法（在 open.mp 里可能不按预期工作）
HideMenuForPlayer(gShopMenu, playerid);

// open.mp 正确写法：先获取玩家当前菜单再关闭
HideMenuForPlayer(GetPlayerMenu(playerid), playerid);
```

---

### 变更五：SetPlayerAttachedObject 不再跨模式保留

SA-MP 中玩家身上的附加物件（attached objects）在游戏模式重启后仍然保留。open.mp 修正了这个行为，重启后附加物件会消失。

如果你需要在模式重启后保留玩家的附加物件，需要在 `OnPlayerConnect` 里重新为他设置。

---

### 变更六：GameText 样式统一（不再渐变）

SA-MP 中有些 GameText 样式会闪烁、有些会忽略时间参数。open.mp 统一用 TextDraw 重新实现了这些样式，外观相同但不再渐变。如果你的玩家反馈 GameText 显示效果有变化，这是正常现象。

---

### 自动升级工具

open.mp 提供了一个自动升级工具，可以批量修复旧代码中的 tag 问题、const 问题和拼写问题：

- 工具已包含在 `qawno/upgrader/` 文件夹中
- 在线版本：https://github.com/openmultiplayer/upgrade

**建议先用工具跑一遍，再手动处理剩余的报错。**

---

## 第三部分：编译错误与警告完全手册

编译器报的信息分两种：

- **error（错误）**：必须修复，否则无法生成 `.amx` 文件
- **warning（警告）**：可以运行，但代码存在潜在问题，建议修复

---

## 常见 Error（错误）

---

### error 001: expected token

**含义**：期望某个符号但没找到，通常是缺少分号、括号不匹配。

```pawn
// 错误：缺少分号
new gold = 100
SetPlayerHealth(playerid, 100.0)

// 正确
new gold = 100;
SetPlayerHealth(playerid, 100.0);
```

```pawn
// 错误：括号不匹配
if(gold > 0
{
    // ...
}

// 正确
if(gold > 0)
{
    // ...
}
```

---

### error 017: undefined symbol "xxx"

**含义**：用了一个没有定义过的变量、函数或常量。

**最常见的三种原因：**

**1. 名字拼写错误**

```pawn
// 错误
SenClientMessage(playerid, -1, "hello"); // 少了 d
new scroe = 0;                           // 字母顺序错了

// 正确
SendClientMessage(playerid, -1, "hello");
new score = 0;
```

**2. SetTimerEx 的目标函数忘记 forward**

```pawn
// 错误：没有 forward，运行时找不到函数
SetTimerEx("MyFunc", 1000, false, "i", playerid);

public MyFunc(playerid) { return 1; }


// 正确：使用前先 forward
forward MyFunc(playerid);

SetTimerEx("MyFunc", 1000, false, "i", playerid);

public MyFunc(playerid) { return 1; }
```

**3. 变量在使用之前没有声明**

```pawn
// 错误：先用再声明
gold += 100;
new gold = 0;

// 正确：先声明再用
new gold = 0;
gold += 100;
```

---

### error 021: symbol already defined "xxx"

**含义**：同一个名字被定义了两次。

```pawn
// 错误：同名变量重复声明
new gold = 0;
new gold = 100; // 重复了

// 错误：同名函数重复定义
MyFunction() { return 1; }
MyFunction() { return 0; } // 重复了
```

也可能是 `.inc` 文件被 `#include` 了两次，导致里面的函数被重复引入。

解决方法：在每个 `.inc` 文件顶部加防重复引入守卫：

```pawn
#if defined _MY_INC
    #endinput
#endif
#define _MY_INC
```

---

### error 025: function heading differs from prototype

**含义**：`forward` 声明的参数和实际 `public` 函数的参数不一致，或者参数类型不匹配。

```pawn
// 错误：forward 和 public 参数不一致
forward MyFunc(playerid);
public MyFunc(playerid, value) { return 1; } // 多了一个参数
```

open.mp 特别常见的情况是回调参数的 tag 不匹配：

```pawn
// 错误（SA-MP 写法）
public OnPlayerDeath(playerid, killerid, reason)

// 正确（open.mp 要求 WEAPON: tag）
public OnPlayerDeath(playerid, killerid, WEAPON:reason)
```

```pawn
// 错误
public OnPlayerEditAttachedObject(playerid, response, index, modelid, boneid, ...)

// 正确（open.mp 要求 EDIT_RESPONSE: tag）
public OnPlayerEditAttachedObject(playerid, EDIT_RESPONSE:response, index, modelid, boneid, ...)
```

---

### error 010: invalid function or declaration

**含义**：函数定义或声明的语法写错了。

```pawn
// 错误：函数名后面缺括号
MyFunction
{
    return 1;
}

// 正确
MyFunction()
{
    return 1;
}
```

---

### error 029: invalid expression, assumed zero

**含义**：表达式写法不合法，通常是运算符用错或者判断写法有问题。

```pawn
// 错误：判断用了单等号（赋值）而不是双等号（比较）
if(gold = 100) { }  // 这是赋值，不是比较

// 正确
if(gold == 100) { }
```

---

### error 035: argument type mismatch (argument N)

**含义**：函数调用时，第 N 个参数的类型与函数期望的类型不符。

```pawn
// 错误：SetPlayerHealth 需要 Float，传了整数
SetPlayerHealth(playerid, 100);

// 正确：加小数点
SetPlayerHealth(playerid, 100.0);
```

```pawn
// 错误：传了字符串给需要整数的参数
SetPlayerSkin(playerid, "86");

// 正确
SetPlayerSkin(playerid, 86);
```

---

### error 047: array sizes do not match

**含义**：把一个数组赋值给另一个数组，但两者大小不一样。

```pawn
new a[10];
new b[20];
a = b; // 错误：大小不同，不能直接赋值

// 正确：用 for 循环逐元素复制，或者用 memcpy
for(new i = 0; i < 10; i++) a[i] = b[i];
```

---

### error 004: function not implemented

**含义**：只有 `forward` 声明，没有对应的 `public` 函数体。

```pawn
// 错误：只 forward 了，没有写函数体
forward MyFunc(playerid);
// 下面没有 public MyFunc ...

// 正确：forward 之后要有对应的实现
forward MyFunc(playerid);
public MyFunc(playerid)
{
    return 1;
}
```

---

## 常见 Warning（警告）

---

### warning 213: tag mismatch

**含义**：变量或参数的 tag（类型标签）不匹配。这是迁移到 open.mp 最常见的警告。

open.mp 为很多函数参数增加了 tag，让编译器能检查你传的值是否合理：

```pawn
// 警告：传了普通整数而不是 bool
TogglePlayerControllable(playerid, 1);

// 正确：用 true/false
TogglePlayerControllable(playerid, true);
```

```pawn
// 警告：传了数字而不是枚举值
TextDrawFont(textid, 1);
GivePlayerWeapon(playerid, 4, 1);

// 正确：用枚举常量
TextDrawFont(textid, TEXT_DRAW_FONT_1);
GivePlayerWeapon(playerid, WEAPON_KNIFE, 1);
```

**如果暂时不想处理这些警告**，可以在顶部加：

```pawn
#define NO_TAGS
#include <open.mp>

// 或者只关闭 213 号警告
#pragma warning disable 213
```

---

### warning 234: function is deprecated

**含义**：你用的函数已经过时，open.mp 建议换用新的写法。

**TextDrawColor → TextDrawColour**

```pawn
TextDrawColor(textid, 0xFFFFFFFF);   // 警告
TextDrawColour(textid, 0xFFFFFFFF);  // 正确
```

**GetPlayerPoolSize / GetVehiclePoolSize / GetActorPoolSize**

```pawn
GetPlayerPoolSize()  // 警告：已移除
MAX_PLAYERS          // 正确
```

**SHA256_PassHash（不安全的密码哈希）**

```pawn
SHA256_PassHash(...); // 警告：SHA-256 不安全
// 改用 BCrypt：https://github.com/Sreyas-Sreelal/samp-bcrypt
```

---

### warning 214 / 239: const 相关警告

**含义**：把字符串或数组传给没有 `const` 修饰的参数，或者反过来。

```pawn
// 警告：参数应该是 const
public MyFunction(string[])
// 正确：加上 const
public MyFunction(const string[])
```

---

### warning 203: symbol is never used

**含义**：你声明了一个变量或函数，但从来没有使用过。

```pawn
// 警告：声明了但没用
new iUnusedVar = 0;

MyUnusedFunction()
{
    return 1;
}
```

解决方法：删掉没用的变量/函数，或者用 `stock` 修饰函数告诉编译器「不用也不要警告」：

```pawn
stock MyMaybeUnusedFunction()
{
    return 1;
}
```

---

### warning 204: symbol is assigned a value that is never used

**含义**：给变量赋值了，但这个值从来没被读取。

```pawn
new iResult = SomeFunction(); // 警告：iResult 赋了值但后面没用到
```

通常意味着这行代码没有实际效果，检查是否逻辑有误。

---

### warning 211: possibly unintended assignment

**含义**：在 `if` 条件里用了单等号 `=`，可能是误把赋值写成了比较。

```pawn
// 警告：这是赋值，不是比较，结果永远为 true
if(gold = 100) { }

// 正确：比较用双等号
if(gold == 100) { }
```

---

### warning 219: local variable shadows a variable at a higher level

**含义**：在内层作用域声明了一个和外层同名的变量，内层的「遮住」了外层的，容易引发逻辑混乱。

```pawn
new gold = 100; // 外层

public OnPlayerConnect(playerid)
{
    new gold = 50; // 警告：这个 gold 遮住了外层的 gold
    // 在这个函数里，gold 指的是 50 那个
}
```

解决方法：给内层变量换个名字，或者直接用外层变量。

---

### warning 225: unreachable code

**含义**：有一段代码永远不会被执行到，通常是 `return` 后面还有代码。

```pawn
MyFunction()
{
    return 1;
    SendClientMessage(playerid, -1, "这行永远不会执行"); // 警告
}
```

---

### warning 209: function should return a value

**含义**：函数应该有返回值，但某个分支没有 `return`。

```pawn
// 警告：else 分支没有 return
MyFunction(iValue)
{
    if(iValue > 0)
    {
        return 1;
    }
    // 如果 iValue <= 0，函数执行完没有 return
}

// 正确：所有分支都要有 return
MyFunction(iValue)
{
    if(iValue > 0)
    {
        return 1;
    }
    return 0;
}
```

---

### warning 202: number of arguments does not match definition

**含义**：调用函数时传入的参数数量和函数定义不一致。

```pawn
// 函数定义需要 2 个参数
MyFunction(playerid, value)
{
    return 1;
}

// 警告：只传了 1 个参数
MyFunction(playerid);

// 正确：传够参数
MyFunction(playerid, 100);
```

---

## 运行时报错（服务器控制台）

这些不是编译错误，而是服务器运行时打印在控制台的提示。

---

### Couldn't announce legacy network to open.mp list

```
[Info] Couldn't announce legacy network to open.mp list.
[Info] [Server Error] Status: 406
```

**含义**：服务器无法被 open.mp 的服务器列表访问到。

**原因**：
- 你在本地运行，没有公网 IP
- 防火墙阻止了对应端口

**影响**：**不影响服务器正常运行**，玩家仍然可以通过 IP 直连，只是不会显示在公开服务器列表里。

---

### Insufficient specifiers given to format

```
[Warning] Insufficient specifiers given to `format`: "?" < 1
```

**含义**：`format` 函数里，格式字符串中的占位符数量少于你传入的参数数量。

```pawn
new str[32];
new name[32] = "Tom";

format(str, sizeof(str), "Hello", name); // 警告：格式字符串里没有 %s，但传了 name

// 正确
format(str, sizeof(str), "Hello %s", name);
```

---

## 有用的资源

| 资源 | 地址 |
|------|------|
| open.mp 官方文档 | https://open.mp/docs |
| open.mp 新增函数列表 | https://open.mp/docs/server/omp-functions |
| config.json 配置说明 | https://www.open.mp/docs/server/config.json |
| 自动升级工具 | https://github.com/openmultiplayer/upgrade |
| 官方 Discord | https://discord.gg/samp |

---

## 延伸阅读
| 地址 | 描述 |
|------|------|
| https://github.com/pawn-lang/samp-stdlib/tree/consistency-overhaul | 更新后的 SA:MP 包含文件，增加了常量正确性及更多标签。 |
| https://github.com/samp-incognito/samp-streamer-plugin/pull/435 | 一个流加载器插件的拉取请求，提供了关于此标签系统的更多信息。 |
| [https://github.com/pawn-lang/compiler/wiki/What's-new#const-correctness](https://github.com/pawn-lang/compiler/wiki/What's-new#const-correctness) | 编译器的变更，错误地将常量正确性列为“破坏性”变更。 |
| https://github.com/pawn-lang/compiler/wiki/Const-Correctness | 关于常量正确性及代码更新的更多信息。 |
| https://github.com/pawn-lang/sa-mp-fixes/ | 许多修复的起源，包括若干已集成至 open.mp 但未在此列出的细微修复。 |
| https://github.com/pawn-lang/compiler/raw/master/doc/pawn-lang.pdf | 获取有关强标签与弱标签的更多信息。 |
| https://github.com/openmultiplayer/upgrade | 一个可自动完成大量标签与常量正确性升级的工具。 |

如果您在运行服务器时仍有问题，请加入官方的 open.mp Discord 服务器：https://discord.gg/samp

在 [#openmp-support](https://discord.com/channels/231799104731217931/966398440051445790) 频道提问。

---

#GTA# #圣安地列斯# #侠盗猎车手# #圣安地列斯联机# #samp# #gta联机# #gtasa联机# #openmp# #omp# #open.mp# #gtasa#

社区交流群: 673335567

论坛: https://open-mp.cn/