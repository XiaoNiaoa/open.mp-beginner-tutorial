# openmp/samp gtasa联机服务器插件开发 完全指南

编写：小鸟 [Github](https://github.com/XiaoNiaoa)
时间：2026/03/23

> 从零开始，用 C++ 为 openmp 服务器开发自定义组件。
>
> 本教程面向有 Pawn 脚本基础、想转 C++ 开发的 SAMP/openmp 服务器开发者。

本教程将教大家制作一个完整的查克拉系统简单插件实现

为什么标题包含samp但是确实openmp类的教程，因为很多人搜索sa-mp，但sa-mp已经彻底消逝了，有许多遗留问题且不再维护更新，openmp是最优解，你应该使用openmp，而且它向下兼容samp（我指的是pawn层），开发过程基本一模一样

---

## 目录

1. [环境搭建](#1-环境搭建)
2. [创建项目](#2-创建项目)
3. [最小可运行组件](#3-最小可运行组件)
4. [组件的生命周期](#4-组件的生命周期)
5. [创建自己的native函数](#5-创建自己的native函数)
6. [事件处理](#6-事件处理)
7. [玩家数据扩展](#7-玩家数据扩展)
8. [如何扫描amx的public](#8-如何扫描amx的public)
9. [完整示例](#9-完整示例)

---

## 1. 环境搭建

这里将演示轻量级环境搭建，你只需要安装三样东西和 [VSCode](https://code.visualstudio.com/) 。

### 1.1 安装工具

**Visual Studio Code**（代码编辑器）

https://code.visualstudio.com/

**Visual Studio Build Tools**（不是完整的 Visual Studio）

这是微软的 C++ 编译器。你可以理解成pawn里的`pawncc.exe`，安装时只勾选「使用 C++ 的桌面开发」工作负载，其他全不选。

下载：https://visualstudio.microsoft.com/visual-cpp-build-tools/

**CMake**

构建系统，管理编译流程。

下载：https://cmake.org/download/

推荐下载 ZIP 版，解压到固定目录（如 `D:\tools\cmake\`），然后把 `D:\tools\cmake\bin` 加入系统 PATH 环境变量。干净利落，不想用了直接删文件夹。

如果选 Installer 版，安装时勾选 "Add CMake to the system PATH"。

**Git**

用于完整获取 open.mp SDK，以及后续各种开发所需的资源。

下载：https://git-scm.com

### 1.2 VSCode 扩展

打开 VSCode，安装以下扩展：

- **C/C++**（Microsoft）— 代码提示、语法高亮、调试
- **CMake Tools**（Microsoft）— CMake 项目集成，一键编译

### 1.3 验证安装

任意终端(比如 cmd)输入以下指令逐个验证是否安装配置成功：

```bash
cmake --version    # 应该显示版本号
git --version      # 应该显示版本号
```

---

## 2. 创建项目

### 2.1 初始化目录

选一个你喜欢的位置，创建项目：

```
新建文件夹 my-component
在根目录新建 src 的文件夹
在根目录新建 lib 的文件夹
```

### 2.2 添加 open.mp SDK 和相关依赖

在`lib`文件夹内点击鼠标右键，选择 `Open Git Bash here` 并执行以下命令

```bash
git clone --recursive https://github.com/openmultiplayer/open.mp-sdk sdk
git clone --recursive https://github.com/openmultiplayer/pawn-natives
git clone --recursive https://github.com/openmultiplayer/compiler pawn
```

等待获取完成，这会在`lib`文件夹内创建 `sdk/` `pawn/` `pawn-natives/`目录。

### 2.3 创建 CMakeLists.txt

在项目根目录创建 `CMakeLists.txt` 并复制以下内容粘贴进去：

```cmake
cmake_minimum_required(VERSION 3.19)
project(my-component)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(lib/sdk)

add_library(${PROJECT_NAME} SHARED
    src/main.cpp
)

# 解决库的兼容性问题
add_definitions(
	-DHAVE_STDINT_H=1
	-DGLM_FORCE_SSE2=1
	-DGLM_FORCE_QUAT_DATA_WXYZ=1
	-DNOMINMAX=1
	-Dnssv_CONFIG_SELECT_STRING_VIEW=nssv_STRING_VIEW_NONSTD
	-Dspan_CONFIG_SELECT_SPAN=span_SPAN_NONSTD
	-DPAWN_CELL_SIZE=32 
)

target_include_directories(${PROJECT_NAME} PRIVATE
    lib/
    lib/pawn/source
    lib/pawn/source/linux
    lib/pawn-natives
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    OMP-SDK
)
```

### 2.4 项目结构

最终你的目录应该长这样：

```
my-component/
├── CMakeLists.txt      ← 构建配置
├── lib/
    ├── sdk/            ← open.mp SDK
    ├── pawn-natives/   ← 用于快速定义 PAWN 原生函数
    └── pawn/           ← 关联库
└── src/
    └── main.cpp        ← 你的代码
```

### 2.5 配置 VSCode

用 VSCode 打开 `my-component` 文件夹。CMake Tools 扩展会自动检测到 `CMakeLists.txt`。

**选择编译器（Kit）：**

1. 按 `Ctrl+Shift+P`，输入 `CMake: Select a Kit`
2. 选择 `Visual Studio 生成工具 2026 Release x86`，如果没有这个选项，选择`[扫描工具包]`即可搜索。

**选择构建类型：**

1. 按 `Ctrl+Shift+P`，输入 `CMake: Select Variant`
2. 开发时选 `Debug`，发布时选 `Release`

> **注意：** 如果你使用的是 CMake 4.x，可能会遇到旧版依赖的兼容性警告。在项目根目录创建 `.vscode/settings.json`：
>
> ```json
> {
>     "cmake.configureArgs": [
>         "-DCMAKE_POLICY_VERSION_MINIMUM=3.10"
>     ]
> }
> ```

---

## 3. 最小可运行组件

创建 `src/main.cpp`，写入以下代码：

```cpp
#include <sdk.hpp>

class MyComponent final : public IComponent
{
private:
    ICore* core_ = nullptr;

public:
    // 每个组件必须有一个全局唯一 ID
    // 去 https://open.mp/uid 生成一个替换下面的ID
    PROVIDE_UID(0x123456789ABCDEF0);

    // 组件名称，显示在服务端加载日志里
    StringView componentName() const override
    {
        return "MyComponent";
    }
    // 版本号
    SemanticVersion componentVersion() const override
    {
        return SemanticVersion(1, 0, 0, 0);
    }
    // 组件加载时调用
    void onLoad(ICore* c) override
    {
        core_ = c;
        core_->printLn("Hello World!");
    }

    // 其他组件初始化完成后调用
    void onInit(IComponentList* components) override {}

    // 所有组件和脚本都就绪后调用
    void onReady() override {}

    // 其他组件卸载时调用
    void onFree(IComponent* component) override {}

    // 组件自身被释放
    void free() override { delete this; }

    // GMX（换图）时调用
    void reset() override {}
};

// 组件入口点 —— 服务端通过这个宏发现并加载你的组件
COMPONENT_ENTRY_POINT()
{
    return new MyComponent();
}
```

**编译：**

按 `F7`（或 `Ctrl+Shift+P` → `CMake: Build`）。编译成功后，在 `build/` 目录下会生成 `my-component.dll`。

**测试：**

把 `.dll` 复制到 open.mp 服务端的 `components/` 目录，启动服务端，你应该能在控制台看到：

```
Loading component MyComponent.dll
        Successfully loaded component MyComponent (1.0.0.0) with UID 123456789abcdef0

Hello World!
```

恭喜，你的第一个组件跑起来了。

---

## 4. 组件的生命周期

组件的函数按固定顺序调用。理解这个顺序非常重要：

```
服务端启动
  │
  ├── onLoad(ICore*)           ← 拿到核心接口
  │
  ├── onInit(IComponentList*)  ← 查询其他组件（如 IPawnComponent）
  │
  ├── onAmxLoad(IPawnScript&)  ← Pawn 脚本加载，扫描 public 函数
  │
  ├── onReady()                ← 一切就绪，可以开始工作
  │
  │   ... 服务器运行中 ...
  │
  ├── onAmxUnload(IPawnScript&) ← Pawn 脚本卸载，清理相关数据
  │
  ├── onFree(IComponent*)      ← 组件卸载时
  │
  └── free()                   ← 释放自身
```

**关键点：**

- `onLoad` 里只保存 `ICore*` 指针，不要访问其他组件（它们可能还没加载）
- `onInit` 里通过 `components->queryComponent<IPawnComponent>()` 获取 Pawn 组件
- `onAmxLoad` 是注册 native 函数和扫描 public 的地方

---

## 5. 创建自己的native函数

Native 函数让 Pawn 脚本能够调用你的 C++ 代码。

### 5.1 基本概念

在 Pawn 里写：

```pawn
native MyAdd(a, b);

public OnGameModeInit()
{
    new a = 10, b = 20;
    new result = MyAdd(a, b);
    printf("%d + %d = %d", a, b, result);
    return 1;
}
```

在 C++ 里你需要实现函数并注册到 AMX。

### 5.2 使用 SCRIPT_API 宏（推荐）

open.mp 提供了 `SCRIPT_API` 宏，自动处理参数的类型转换：

```cpp
#include <sdk.hpp>
#include <Server/Components/Pawn/pawn.hpp>
#include <Server/Components/Pawn/Impl/pawn_natives.hpp>
#include <Server/Components/Pawn/Impl/pawn_impl.hpp>

SCRIPT_API(MyAdd, int(int a, int b))
{
    return a + b;
}
```

`SCRIPT_API` 的参数含义：
- 第一个参数：函数名，必须和 Pawn 里的 `native` 名称完全一致
- 第二个参数：返回类型(参数列表)

### 5.3 注册时机

在组件的 `onAmxLoad` 里调用 `pawn_natives::AmxLoad`，它会自动把所有 `SCRIPT_API` 定义的函数注册到 AMX：

```cpp
void onAmxLoad(IPawnScript& script) override
{
    pawn_natives::AmxLoad(script.GetAMX());
}
```

### 5.4 完整的组件 + Native 示例

```cpp
#include <sdk.hpp>
#include <Server/Components/Pawn/pawn.hpp>
#include <Server/Components/Pawn/Impl/pawn_natives.hpp>
#include <Server/Components/Pawn/Impl/pawn_impl.hpp>

class MyComponent final : public IComponent, public PawnEventHandler
{
private:
    ICore* core_ = nullptr;
    IPawnComponent* pawn_ = nullptr;

public:
    // 每个组件必须有一个全局唯一 ID
    // 去 https://open.mp/uid 生成一个替换下面的ID
    PROVIDE_UID(0x123456789ABCDEF0);

    StringView componentName() const override { return "MyComponent"; }
    SemanticVersion componentVersion() const override { return SemanticVersion(1, 0, 0, 0); }

    void onLoad(ICore* c) override
    {
        core_ = c;

        // 初始化 AMX 查找表
        setAmxLookups(core_);  
    }

    void onInit(IComponentList* components) override
    {
        pawn_ = components->queryComponent<IPawnComponent>();
        if (pawn_)
        {
            setAmxFunctions(pawn_->getAmxFunctions());
            setAmxLookups(components);
            pawn_->getEventDispatcher().addEventHandler(this);
        }
    }

    void onAmxLoad(IPawnScript& script) override
    {
        pawn_natives::AmxLoad(script.GetAMX());
    }

    void onAmxUnload(IPawnScript& script) override {}
    void onReady() override {}
    void onFree(IComponent* component) override {}
    void free() override { delete this; }
    void reset() override {}

    ~MyComponent()
    {
        if (pawn_) pawn_->getEventDispatcher().removeEventHandler(this);
    }
};

COMPONENT_ENTRY_POINT()
{
    return new MyComponent();
}

SCRIPT_API(MyAdd, int(int a, int b))
{
    return a + b;
}
```

重新编译插件并开启服务器，你就可以在控制台看到 `10 + 20 = 30`

---

## 6. 事件处理

open.mp 通过 EventHandler 接口分发事件。你的组件实现对应接口，注册到分发器，就能收到事件。

### 6.1 常见的 EventHandler

| 接口 | 事件 |
|--|--|
| `PlayerConnectEventHandler` | 玩家连接/断开 |
| `PlayerTextEventHandler` | 玩家聊天/命令 |
| `PawnEventHandler` | AMX 脚本加载/卸载 |

### 6.2 示例：监听玩家连接

```cpp
class MyComponent final
    : public IComponent
    , public PawnEventHandler
    , public PlayerConnectEventHandler  // 实现连接事件接口
{
    // ... 省略其他成员 ...

    void onInit(IComponentList* components) override
    {
        // 注册 Pawn 事件
        pawn_ = components->queryComponent<IPawnComponent>();
        if (pawn_)
        {
            setAmxFunctions(pawn_->getAmxFunctions());
            setAmxLookups(components);
            pawn_->getEventDispatcher().addEventHandler(this);
        }

        // 注册玩家连接事件
        core_->getPlayers().getPlayerConnectDispatcher().addEventHandler(this);
    }
    // 析构函数里注销
    ~MyComponent()
    {
        if (pawn_)  pawn_->getEventDispatcher().removeEventHandler(this);
        if (core_) core_->getPlayers().getPlayerConnectDispatcher().removeEventHandler(this);
    }
    // 玩家连接时调用
    void onPlayerConnect(IPlayer& player) override
    {
        core_->printLn("玩家 %d 进入了服务器!", player.getID());
    }
};
```

---

## 7. 玩家数据扩展

在 Pawn 里，你可能习惯用全局数组存储玩家数据：

```pawn
new pScore[MAX_PLAYERS];
new pLevel[MAX_PLAYERS];
new bool:pLoggedIn[MAX_PLAYERS];
```

在 open.mp 的 C++ 组件里，对应的机制是 **IExtension**。它把数据"挂"在玩家对象上。

### 7.1 定义数据结构

```cpp
struct PlayerEnergy final : IExtension
{
    // 每个 Extension 需要一个全局唯一 ID
    // 去 https://open.mp/uid 生成
    PROVIDE_EXT_UID(0xABCDEF1234567890);

    // 你的自定义数据
    int energy = 100;
    int maxEnergy = 100;

    void freeExtension() override { delete this; }
    void reset() override
    {
        energy = 100;
        maxEnergy = 100;
    }
};
```

### 7.2 在玩家连接时挂载

```cpp
void onPlayerConnect(IPlayer& player) override
{
    // 创建 PlayerEnergy 实例并挂到玩家身上
    // 第二个参数 true 表示玩家断开时自动调用 freeExtension
    player.addExtension(new PlayerEnergy(), true);
}
```

### 7.3 操作数据

```cpp
auto* playerData = queryExtension<PlayerEnergy>(player);
if (playerData)
{
    playerData->energy += 100;
}
```

### 7.4 为什么不用 `std::unordered_map<int, PlayerEnergy>`？

你可以用，但 IExtension 有几个优势：

- **生命周期自动管理**：玩家断开时 `freeExtension` 自动调用，不会忘记清理
- **GMX 自动重置**：换图时 `reset` 自动调用
- **数据跟着对象走**：不需要传 playerid 再去查 map，直接从 `IPlayer&` 取

---

## 8. 如何扫描amx的public

这是反方向的交互：你的 C++ 组件主动调用 Pawn 脚本里定义的 `public` 函数。

### 8.1 理解 AMX 虚拟机

Pawn 脚本编译后是 `.amx` 文件，由 AMX 虚拟机执行。每个 `public` 函数在 AMX 里有一个编号（index）。要调用它，你需要：

1. **找到编号** — 用 `amx_FindPublic` 或在 `onAmxLoad` 时扫描
2. **压入参数** — 用 `amx_Push`（整数）或 `amx_PushString`（字符串）
3. **执行** — 用 `amx_Exec`
4. **清理** — 用 `amx_Release` 释放字符串内存

### 8.2 扫描 Public 函数

假设 Pawn 里有：

```pawn
forward OnPlayerEnergyChange(playerid, oldEnergy, newEnergy);
public OnPlayerEnergyChange(playerid, oldEnergy, newEnergy)
{
    printf("玩家 %d 的查克拉发生了变化: %d -> %d", playerid, oldEnergy, newEnergy);
}
```

在 `onAmxLoad` 时扫描它：

```cpp
// 保存 AMX 指针和 public index
struct ScriptInfo
{
    AMX* amx = nullptr;
    int onEnergyChange = -1;  // -1 表示该脚本里没有这个函数
};

std::vector<ScriptInfo> scripts_;

void onAmxLoad(IPawnScript& script) override
{
    pawn_natives::AmxLoad(script.GetAMX());

    AMX* amx = script.GetAMX();
    ScriptInfo info;
    info.amx = amx;

    // 方法一：按名称精确查找
    amx_FindPublic(amx, "OnPlayerEnergyChange", &info.onEnergyChange);
    // 如果找到，info.onEnergyChange >= 0
    // 如果没找到，info.onEnergyChange 保持 -1

    scripts_.push_back(info);
}
```

### 8.3 调用 Public 函数

找到 index 后，就可以调用了。**AMX 是栈式虚拟机，参数要倒序压入。**

**调用无参数函数：**

```pawn
// Pawn 侧
forward OnSomethingHappen();
public OnSomethingHappen()
{
    print("damn!");
}
```

```cpp
// C++ 侧
void callOnSomethingHappen(AMX* amx, int index)
{
    cell returnValue;
    amx_Exec(amx, &returnValue, index);
    // returnValue 是 Pawn 函数的返回值
}
```

**调用带整数参数的函数：**

```pawn
// Pawn 侧
forward OnPlayerEnergyChange(playerid, oldEnergy, newEnergy);
public OnPlayerEnergyChange(playerid, oldEnergy, newEnergy)
{
    printf("玩家 %d 的查克拉发生了变化: %d -> %d", playerid, oldEnergy, newEnergy);
    return 1;
}
```

```cpp
// C++ 侧
void callOnPlayerEnergyChange(AMX* amx, int index, int playerid, int oldEnergy, int newEnergy)
{
    // Pawn: OnPlayerEnergyChange(playerid, oldEnergy, newEnergy)
    // 倒序压栈
    amx_Push(amx, newEnergy);
    amx_Push(amx, oldEnergy);
    amx_Push(amx, playerid);

    cell returnValue;
    amx_Exec(amx, &returnValue, index);
}
```

**调用带字符串参数的函数：**

```pawn
// Pawn 侧
forward OnPlayerMessage(playerid, const message[]);
public OnPlayerMessage(playerid, const message[])
{
    printf("Player %d: %s", playerid, message);
    return 1;
}
```

```cpp
// C++ 侧
void callMessage(AMX* amx, int index, int playerid, const char* message)
{
    cell addr_message;
    amx_PushString(amx, &addr_message, nullptr, message, 0, 0);
    amx_Push(amx, playerid);

    cell retval;
    amx_Exec(amx, &retval, index);

    // 必须释放 addr_message 不然会导致内存泄漏
    amx_Release(amx, addr_message);
}
```

### 8.4 AMX API 速查表

可查阅：https://open.mp/docs/tutorials/PluginDevelopmentGuide#amx-functions

| 函数 | 用途 |
|--|--|
| `amx_NumPublics(amx, &count)` | 获取 public 函数数量 |
| `amx_GetPublic(amx, index, name)` | 按 index 获取函数名 |
| `amx_FindPublic(amx, name, &index)` | 按名称查找 index |
| `amx_Push(amx, value)` | 压入整数参数 |
| `amx_PushString(amx, &addr, nullptr, str, 0, 0)` | 压入字符串参数 |
| `amx_Exec(amx, &retval, index)` | 执行 public 函数 |
| `amx_Release(amx, addr)` | 释放 PushString 分配的内存 |

---

## 9. 完整示例

下面是一个完整的组件，包含上述所有概念。功能很简单：追踪玩家的"查克拉"，提供 native 函数给 Pawn 操作，并在查克拉变化时回调 Pawn。

### 9.1 C++ 组件（src/main.cpp）

```cpp
#include <sdk.hpp>
#include <Server/Components/Pawn/pawn.hpp>
#include <Server/Components/Pawn/Impl/pawn_natives.hpp>
#include <Server/Components/Pawn/Impl/pawn_impl.hpp>

#include <vector>
#include <string>

// 玩家数据扩展(查克拉)
struct PlayerEnergy final : IExtension
{
    // 每个组件必须有一个全局唯一 ID
    // 去 https://open.mp/uid 生成一个替换下面的ID
    PROVIDE_EXT_UID(0x1A2B3C4D5E6F7890);

    int energy = 100;
    int maxEnergy = 100;

    void freeExtension() override { delete this; }
    void reset() override
    {
        energy = 100;
        maxEnergy = 100;
    }
};

// 组件主类
class EnergyComponent final
    : public IComponent
    , public PawnEventHandler
    , public PlayerConnectEventHandler
{
private:
    ICore* core_ = nullptr;
    IPawnComponent* pawn_ = nullptr;

    // 保存 AMX 脚本中 OnPlayerEnergyChange 的 index
    struct ScriptInfo
    {
        AMX* amx = nullptr;
        int onEnergyChange = -1;
    };
    std::vector<ScriptInfo> scripts_;

public:
    PROVIDE_UID(0x9F8E7D6C5B4A3210);

    StringView componentName() const override { return "EnergySystem"; }
    SemanticVersion componentVersion() const override { return SemanticVersion(1, 0, 0, 0); }

    void onLoad(ICore* c) override
    {
        core_ = c;
        setAmxLookups(core_);
        core_->printLn("[查克拉插件] 查克拉系统已加载.");
    }

    void onInit(IComponentList* components) override
    {
        pawn_ = components->queryComponent<IPawnComponent>();
        if (pawn_)
        {
            setAmxFunctions(pawn_->getAmxFunctions());
            setAmxLookups(components);
            pawn_->getEventDispatcher().addEventHandler(this);
        }
        core_->getPlayers().getPlayerConnectDispatcher().addEventHandler(this);
    }

    // Pawn 事件 
    void onAmxLoad(IPawnScript& script) override
    {
        pawn_natives::AmxLoad(script.GetAMX());

        // 扫描 Pawn 脚本中的回调函数
        AMX* amx = script.GetAMX();
        ScriptInfo info;
        info.amx = amx;
        amx_FindPublic(amx, "OnPlayerEnergyChange", &info.onEnergyChange);
        scripts_.push_back(info);
    }

    void onAmxUnload(IPawnScript& script) override
    {
        AMX* amx = script.GetAMX();
        scripts_.erase(
            std::remove_if(scripts_.begin(), scripts_.end(), [amx](const ScriptInfo& s) { return s.amx == amx; }), scripts_.end());
    }

    // 玩家事件 
    void onPlayerConnect(IPlayer& player) override
    {
        player.addExtension(new PlayerEnergy(), true);
    }

    // C++ 调用 Pawn 回调 
    void callOnPlayerEnergyChange(int playerid, int oldEnergy, int newEnergy)
    {
        for (auto& s : scripts_)
        {
            if (s.onEnergyChange < 0)
                continue;

            // Pawn: OnPlayerEnergyChange(playerid, oldEnergy, newEnergy)
            // 倒序压栈
            amx_Push(s.amx, newEnergy);
            amx_Push(s.amx, oldEnergy);
            amx_Push(s.amx, playerid);

            cell retval;
            amx_Exec(s.amx, &retval, s.onEnergyChange);
        }
    }

    // 生命周期 
    void onReady() override {}
    void onFree(IComponent* component) override
    {
        if (component == pawn_)
        {
            pawn_ = nullptr;
            setAmxFunctions();
            setAmxLookups();
        }
    }
    void free() override { delete this; }
    void reset() override {}

    ~EnergyComponent()
    {
        if (pawn_) pawn_->getEventDispatcher().removeEventHandler(this);
        if (core_) core_->getPlayers().getPlayerConnectDispatcher().removeEventHandler(this);
    }
};

// 全局指针，给 SCRIPT_API 用
static EnergyComponent* gComponent = nullptr;

COMPONENT_ENTRY_POINT()
{
    gComponent = new EnergyComponent();
    return gComponent;
}

// Native 函数
// native GetPlayerEnergy(playerid);
SCRIPT_API(GetPlayerEnergy, int(IPlayer& player))
{
    auto* data = queryExtension<PlayerEnergy>(player);
    return data ? data->energy : 0;
}

// native SetPlayerEnergy(playerid, energy);
SCRIPT_API(SetPlayerEnergy, bool(IPlayer& player, int energy))
{
    auto* data = queryExtension<PlayerEnergy>(player);
    if (!data)
        return false;

    int oldEnergy = data->energy;
    data->energy = (energy > data->maxEnergy) ? data->maxEnergy : energy;

    // 触发 Pawn 回调
    if (data->energy != oldEnergy && gComponent)
        gComponent->callOnPlayerEnergyChange(player.getID(), oldEnergy, data->energy);

    return true;
}

// native GetPlayerMaxEnergy(playerid);
SCRIPT_API(GetPlayerMaxEnergy, int(IPlayer& player))
{
    auto* data = queryExtension<PlayerEnergy>(player);
    return data ? data->maxEnergy : 0;
}

// native SetPlayerMaxEnergy(playerid, maxEnergy);
SCRIPT_API(SetPlayerMaxEnergy, bool(IPlayer& player, int maxEnergy))
{
    auto* data = queryExtension<PlayerEnergy>(player);
    if (!data)
        return false;

    data->maxEnergy = maxEnergy;
    if (data->energy > data->maxEnergy)
        data->energy = data->maxEnergy;

    return true;
}
```

### 9.2 Pawn Include 文件（energy.inc）

```pawn
// 函数声明
native GetPlayerEnergy(playerid);
native SetPlayerEnergy(playerid, energy);
native GetPlayerMaxEnergy(playerid);
native SetPlayerMaxEnergy(playerid, maxEnergy);

// 回调声明
forward OnPlayerEnergyChange(playerid, oldEnergy, newEnergy);
```

### 9.3 Pawn 测试脚本

```pawn
#include <open.mp>
#include "energy.inc"

main() {}

public OnPlayerConnect(playerid)
{
    // 连接时默认 energy = 100
    new energy = GetPlayerEnergy(playerid);
    printf("[查克拉系统] Player %d 进入了服务器, 查克拉 = %d", playerid, energy);

    // 设置为 50，会触发 OnPlayerEnergyChange
    SetPlayerEnergy(playerid, 50);

    return 1;
}

// C++ 组件在查克拉变化时回调这个函数
public OnPlayerEnergyChange(playerid, oldEnergy, newEnergy)
{
    printf("[回调] 玩家 %d 的查克拉发生了变化: %d -> %d", playerid, oldEnergy, newEnergy);
    return 1;
}
```

### 9.4 数据流向图

```
玩家连接
  │
  ├─ open.mp 触发 onPlayerConnect
  │   └─ C++ 创建 PlayerEnergy, 挂到玩家对象上
  │
  ├─ Pawn OnPlayerConnect 被调用
  │   └─ 调用 native SetPlayerEnergy(playerid, 50)
  │        │
  │        ├─ SCRIPT_API 自动解析 playerid → IPlayer&
  │        ├─ queryExtension<PlayerEnergy>(player) 取出数据
  │        ├─ 修改 energy: 100 → 50
  │        └─ callOnPlayerEnergyChange(playerid, 100, 50)
  │             │
  │             ├─ amx_Push(amx, 50)         ← newEnergy
  │             ├─ amx_Push(amx, 100)        ← oldEnergy
  │             ├─ amx_Push(amx, playerid)
  │             └─ amx_Exec → 调用 Pawn 的 OnPlayerEnergyChange

```

---

## 附录：资源

- **c++学习网站**: https://www.w3school.com.cn/cpp/cpp_reference.asp
- **open.mp SDK 仓库**：https://github.com/openmultiplayer/open.mp-sdk
- **open.mp 组件模板示例**：https://github.com/openmultiplayer/pawn-template
- **open.mp 完整组件模板示例**：https://github.com/openmultiplayer/full-template
- **使用组件实现的gamemode示例**：https://github.com/openmultiplayer/rivershell-cpp
- **UID 生成器**：https://open.mp/uid
- **open.mp 官方文档**：https://www.open.mp/docs
- **amx相关函数说明文档**：https://open.mp/docs/tutorials/PluginDevelopmentGuide#amx-functions


---

#GTA# #圣安地列斯# #侠盗猎车手# #圣安地列斯联机# #samp# #gta联机# #gtasa联机# #openmp# #omp# #open.mp# #gtasa#

社区交流群: 673335567

论坛: https://open-mp.cn/