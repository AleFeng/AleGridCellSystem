<p align="center">
  <img alt="GitHub Release" src="https://img.shields.io/github/v/release/AleFeng/AleGridCellSystem?color=blue">
  <img alt="GitHub Downloads (all assets, all releases)" src="https://img.shields.io/github/downloads/AleFeng/AleGridCellSystem/total?color=green">
  <img alt="GitHub Repo License" src="https://img.shields.io/badge/license-GPL--2.0-blueviolet">
  <img alt="GitHub Repo Issues" src="https://img.shields.io/github/issues/AleFeng/AleGridCellSystem?color=yellow">
</p>

<p align="center">
  🌍
  中文 |
  <a href="./README_EN.md">English</a> |
  <a href="./README_JA.md">日本語</a>
</p>

<p align="center">
  📥
  <a href="#-安装">安装</a> |
  <a href="#-快速开始">快速开始</a>
</p>

# AleGridCellSystem - 三维网格单元系统
AleGridCellSystem 是一套面向 `Unity` 的**三维网格单元系统**，用于以网格为基础的**建造、放置、定位、区域划分**等玩法。  
它把「网格逻辑」和「显示表现」彻底解耦：网格中的物品是**纯数据**（不强制依赖 `GameObject` 与 Unity 碰撞器），一件物品可以**跨多个单元格**、拥有**朝向**、携带**任意状态**，并可挂载与占地尺寸相互独立的**真实碰撞体积**。  
在此之上还提供了**定位系统**（检测周围物体及其相对方向）、**区域系统**（房间划分与焦点剖面显示），以及一套基于 `Aseprite` 的**编辑器资产管线**，可一键批量生成网格物品预制体、网格 Mesh、碰撞器与组装件。

> 命名空间为 `AleGridCellSystem`，代码内 API 均在此命名空间下。

## 📜 目录
- [✨ 简介](#-简介)
  - [项目特性](#项目特性)
- [💻 环境要求](#-环境要求)
- [📦 安装](#-安装)
  - [使用 UPM（推荐）](#使用-upm推荐)
  - [手动安装](#手动安装)
  - [依赖说明](#依赖说明)
- [🧩 核心概念](#-核心概念)
- [🌱 快速开始](#-快速开始)
- [🧱 网格与单元格操作](#-网格与单元格操作)
- [🚦 状态系统](#-状态系统)
- [🎯 定位系统](#-定位系统)
- [🗂️ 区域系统](#-区域系统)
- [🧊 真实体积与碰撞](#-真实体积与碰撞)
- [🖼️ 坐标转换与显示](#-坐标转换与显示)
- [🛠️ 编辑器工具](#-编辑器工具)
  - [Aseprite 资产管线](#aseprite-资产管线)
  - [生成预制件及批量网格物品](#生成预制件及批量网格物品)
  - [更新与配置](#更新与配置)
- [🔌 工作流扩展](#-工作流扩展)
- [📁 目录结构](#-目录结构)
- [📋 待办事项列表](#-待办事项列表)
- [📄 许可证](#-许可证)

## ✨ 简介
使用本系统，你可以在三维网格空间中快速实现建造 / 摆放类玩法。  
网格由**若干层（Layer）**的三维单元格数组组成，坐标为 `X 横向 / Y 纵向 / Z 高度`。每个单元格可以放置一个**网格物品（GridItem）**，物品可以跨越多个单元格并区分**主单元格（Main）**与**子单元格（Sub）**——查询任意子单元格都能回溯到主单元格。  
物品在网格中是**纯数据**，因此可以高效地管理大量物品；显示层（`ViewRoot`）与碰撞层（`ColliderRoot`）作为可选内容挂接在物品之上，与网格逻辑相互独立。  
系统同时内置了**坐标转换**（网格 ↔ 世界 ↔ 2.5D 显示坐标）与**渲染深度排序**，方便实现 2.5D / 伪等距（isometric）风格的像素游戏。

### 项目特性
| 特性 | 描述 |
| --- | --- |
| 三维分层网格 | 多层三维单元格数组，`X` 横向 / `Y` 纵向 / `Z` 高度，可自定义单元格世界尺寸与各轴网格数量。 |
| 多格占用与朝向 | 物品可跨多个单元格（主单元格 + 子单元格），支持 `Down/Left/Right/Up` 四向朝向，横竖朝向时自动翻转占地尺寸。 |
| 占位与冲突检测 | 压入物品前自动检查目标范围是否被占用，避免重叠放置。 |
| 状态系统 | 每个单元格可携带任意 `int` 状态集合，支持按范围进行 `All`（全部）/ `Anyone`（任一）检测。 |
| 定位系统 | 发起者 / 接收者模式，实时检测范围内的物体及其**相对方向**（上下左右前后），支持 有 / 无 / 仅有 / 或 多种匹配模式。 |
| 区域系统 | 将网格划分为**区域组**与**区域**（房间），统计区域内物品，支持**焦点区域**（隐藏高于焦点层的区域，实现剖面看楼层）。 |
| 真实体积与碰撞分离 | 占地尺寸与实际碰撞体积**解耦**，一件物品可挂多组真实体积并在运行时切换。 |
| 坐标转换 | 网格坐标 ↔ 世界坐标 ↔ 2.5D 显示坐标互转，内置渲染深度排序。 |
| Aseprite 资产管线 | 在 Aseprite 中绘制并导出数据，一键批量生成预制体、Mesh、碰撞器与组装件。 |
| 工作流可扩展 | 通过处理器脚本挂钩创建 / 更新流程，为不同类型的物品注入项目自定义逻辑。 |

## 💻 环境要求
- `Unity 2021.3 LTS` 或更新版本（源码使用了 C# 8 的 `switch` 表达式等特性）。
- **运行时核心**（`GridCellSystemManager`、`GridItemComponent` 及各数据类型）仅依赖 `UnityEngine`，可独立集成到你的项目逻辑中。
- **编辑器资产管线**（自动生成预制体 / Mesh / 碰撞器）依赖由 `Aseprite` 导出的网格数据文本，详见 [Aseprite 资产管线](#aseprite-资产管线)。

## 📦 安装
### 使用 UPM（推荐）
通过 Unity Package Manager 以 git URL 安装：

1. 打开 `Window → Package Manager`。
2. 点击左上角 `+` → `Add package from git URL...`。
3. 粘贴以下地址并点击 `Add`：

```
https://github.com/AleFeng/AleGridCellSystem.git?path=/Assets/PluginsDeveloper/AleGridCellSystem
```

或直接在工程的 `Packages/manifest.json` 的 `dependencies` 中添加：

```json
"com.alefeng.alegridcellsystem": "https://github.com/AleFeng/AleGridCellSystem.git?path=/Assets/PluginsDeveloper/AleGridCellSystem"
```

> 需要锁定版本时，可在 URL 末尾追加标签 / 分支名，例如 `...AleGridCellSystem#1.1.0`。

### 手动安装
1. 下载或克隆本仓库。
2. 将 `Assets/PluginsDeveloper/AleGridCellSystem` 整个文件夹拷贝到你自己工程的 `Assets` 目录下。
3. 等待 Unity 编译完成，即可在 `AleGridCellSystem` 命名空间下使用各类 API。

### 依赖说明
插件**自包含、可独立编译运行**，运行时与编辑器均只依赖 Unity 自身，无第三方框架依赖。

- 编辑器的自动生成管线以 `Aseprite` 导出的网格数据文本为输入（见 [Aseprite 资产管线](#aseprite-资产管线)）。
- 生成显示网格所用的材质着色器可通过 `GridSystemConfig.viewMaterialShaderName` 配置（默认 `Able/Lit-Alpha`）；当工程内找不到该着色器时，会自动回退到内置着色器 `Sprites/Default`。

## 🧩 核心概念
- **网格与分层**：整个网格由 `GridCellSystemManager` 管理，内部是「层组」——每层是一个 `X × Y × Z` 的三维单元格数组。通过 `Init(...)` 指定单元格世界尺寸、各轴数量与层数。
- **坐标系**：网格坐标 `GridCoord` 中 `X` 为横向、`Y` 为纵向、`Z` 为**高度**。请注意这与 Unity 不同——**Unity 中高度是 Y 轴**，坐标转换时系统会自动交换 Y / Z。整数坐标用 `GridCoord`，需要小数精度时用 `GridCoordFloat`，两者可隐式互转，也支持与 `Vector3` 互转。
- **网格物品（GridItem）**：由 `GridItemData` 描述，包含数值 `Value`、类型 `EGridItemType`（`Main` 主 / `Sub` 子）、主单元格坐标 `MainGridCoord`、占地尺寸 `GridItemSize` 与朝向 `Direction`。当朝向为 `Left` / `Right` 时，占地尺寸的 X / Y 会自动翻转（`GetGridItemSizeAtDirection`）。
- **组合式组件 `GridItemComponent`**：这是运行时物品的功能载体，内含数据、状态、真实体积、显示与定位等能力。它被设计为**组合**在你项目自定义的「GridItem 基类」（一个 `MonoBehaviour`）中作为成员字段使用——只要你的类里有一个 `GridItemComponent` 字段，就拥有了网格物品的全部能力。
- **真实体积 `RealVolume`**：物品「占用哪些单元格」与它「实际的碰撞形状」是两回事。真实体积描述了包围盒尺寸、本地坐标与对应的碰撞器节点；一件物品可以有多组真实体积并在运行时切换。

## 🌱 快速开始
下面演示如何直接使用 `GridCellSystemManager` 管理网格数据：

```csharp
using AleGridCellSystem;
using UnityEngine;

// 1. 创建并初始化网格：单元格世界尺寸 1×1×1，网格 20×20×10，1 层
var grid = new GridCellSystemManager();
grid.Init(
    cellUnitSizeX: 1f, cellUnitSizeY: 1f, cellUnitSizeZ: 1f,
    gridCellCountX: 20, gridCellCountY: 20, gridCellCountZ: 10,
    layerCount: 1);

// 2. 放置一件 2×1×1、朝下、数值为 1001 的网格物品到 (3,4,0)
var data = new GridItemData
{
    Value        = 1001,
    GridItemType = EGridItemType.Main,
    MainGridCoord= new GridCoord(3, 4, 0),
    GridItemSize = new GridCoord(2, 1, 1),
    Direction    = EDirection.Down,
};

if (grid.PushMainGridItemData(0, data))
    Debug.Log("放置成功");
else
    Debug.Log("目标范围已被占用，放置失败");

// 3. 查询：任意子单元格都能回溯到主单元格数值
int value = grid.GetMainGridItemValue(0, new GridCoord(4, 4, 0)); // → 1001

// 4. 坐标转换：网格坐标 ↔ 世界坐标
GridCoordFloat world = grid.GetWorldPosition(new GridCoord(3, 4, 0));
GridCoord coord      = grid.GetGridCoord(world);

// 5. 移除物品（会连同其所有子单元格一起清除）
grid.RemoveMainGridItemData(0, new GridCoord(3, 4, 0));
```

## 🧱 网格与单元格操作
`GridCellSystemManager` 提供了网格的读写与查询接口（`layer` 为层索引）：

| 方法 | 说明 |
| --- | --- |
| `Init(cellUnitSizeX/Y/Z, gridCellCountX/Y/Z, layerCount)` | 初始化网格：单元格世界尺寸、各轴单元格数量、层数。 |
| `GetGridItem(layer, gridCoord, isNullCreateNew=false)` | 获取指定坐标的单元格（`GridItemComponent`）。 |
| `GetMainGridItem(layer, gridCoord)` | 获取主单元格；若命中的是子单元格会自动回溯到其主单元格。 |
| `GetMainGridItemValue(layer, gridCoord)` | 获取主单元格的数值。 |
| `PushMainGridItemData(layer, gridItemData)` | 压入物品，**放置前会检测占地范围是否冲突**，冲突则返回 `false`。 |
| `SetMainGridItemValue(layer, gridItemData)` | 设置主单元格数值，并按占地尺寸铺设子单元格。 |
| `PopMainGridItemValue(layer, gridCoord)` | 弹出并移除主单元格（返回其数据副本）。 |
| `RemoveMainGridItemData(layer, gridCoord)` | 移除主单元格及其全部子单元格。 |
| `CheckGridItemSizeHasGridItem(layer, gridCoord, gridItemSize)` | 检测某尺寸范围内是否已有物品。 |

> 一件物品占据 `GridItemSize` 范围内的所有单元格，其中一个为**主单元格**（记录完整数据），其余为**子单元格**（指向主单元格）。这让「多格家具 / 建筑」等能被正确放置、查询与整块移除。

## 🚦 状态系统
每个单元格可以携带一组自定义 `int` 状态（例如「已预定」「不可通行」等），并按范围进行检测：

```csharp
const int STATE_RESERVED = 1;

// 为一块 2×1×1 的范围整体添加状态
grid.SetGridItemState(0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1), STATE_RESERVED, true);

// 检测该范围是否「全部」都带有此状态
bool allReserved = grid.CheckGridItemSizeState(
    0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1),
    STATE_RESERVED, ECheckStateType.All);

// 也可检测「任意一个」单元格是否带有此状态
bool anyReserved = grid.CheckGridItemSizeState(
    0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1),
    STATE_RESERVED, ECheckStateType.Anyone);
```

## 🎯 定位系统
定位系统用于**检测某个物品周围范围内有哪些物品，以及它们相对自己的方向**（上 / 下 / 左 / 右 / 前 / 后），常用于「相邻检测」「拼接判定」等玩法。它基于**发起者（Emitter）/ 接收者（Receiver）** 模型：

- **发起者**：会被接收者感知的物品，`EnableLocationEmitter = true`。
- **接收者**：主动检测范围的物品，`EnableLocationReceiver = true`，并配置检测范围。

```csharp
// 前提：把物品的 GridItemComponent 交给网格管理器
gridItem.SetGridCellSystemManager(grid);

// 接收者：以自身中心为基准，检测「上方」3 格范围
gridItem.SetLocationCenter(ELocationCenterType.MiddleCenter);
gridItem.SetLocationReceiverRange(ELocationReceiverRangeType.Up, 3f);
gridItem.EnableLocationReceiver = true;

// 另一件物品作为发起者
otherItem.EnableLocationEmitter = true;

// 查询：范围内是否存在位于「上方」的发起者
bool hasAbove = gridItem.CheckLocationEmitterInfoDirection(
    ELocationCheckMode.Have, ELocationDirection.Up);

// 监听范围内发起者信息变化
gridItem.OnLocationEmitterInfoChange = item => { /* 刷新表现等 */ };
```

- 检测范围类型 `ELocationReceiverRangeType`：`Around`（全方向）、`LeftRight` / `FrontBack` / `UpDown`（轴向），以及单向的 `Up/Down/Left/Right/Front/Back`。
- 匹配模式 `ELocationCheckMode`：`Have`（含指定方向）、`None`（不含）、`Only`（仅有、完全匹配）、`Or`（含其一即可）。

## 🗂️ 区域系统
区域系统把网格划分为若干**区域（`AreaInfo`）**，并归入**区域组（`AreaGroupInfo`）**，可用于房间 / 楼层 / 功能区等划分：

```csharp
int groupId = 1;

// 在区域组 1 中登记一块厨房区域：起点 (0,0,0)，尺寸 5×5×3
grid.AddAreaInfo(groupId, new AreaInfo("Room_Kitchen", new GridCoord(0, 0, 0), new GridCoord(5, 5, 3)));

// 查询某坐标落在哪个区域内
AreaInfo area = grid.CheckInAreaInfo(new GridCoord(2, 2, 0));

// 焦点区域：聚焦该区域，隐藏「高于」它的区域（例如剖开上层楼板看室内）
grid.SetFocusAreaInfo(area);

// 取消聚焦，全部区域恢复可见
grid.SetFocusAreaInfo(null);
```

- 每个区域会**统计落在其边界内的网格物品**（`AddIntraGridItem` / `RemoveIntraGridItem`），并可整体切换可见性（`SetVisibleState`）。
- 区域可携带数值 `Value`（如配置表 ID），数值变化时通过 `OnValueChange` 回调通知。
- 区域组会自动维护其下所有区域的**整体边界**，便于快速做包含判断。

## 🧊 真实体积与碰撞
物品「占据哪些单元格」（`GridItemSize`）与它「实际的碰撞形状」是分离的。后者由**真实体积 `RealVolume`** 描述：

- 每个 `RealVolume` 包含**包围盒单位化尺寸**、**本地坐标**（以占地范围左下角为原点）以及绑定的**碰撞器节点 GameObject**。
- 一件物品可以拥有**多组**真实体积，通过 `SetRealVolumeCur(keyName)` 在运行时切换当前生效的碰撞体（例如同一物体的不同形态）。
- 编辑器管线会依据 Aseprite 导出的体积数据自动生成碰撞器（`BoxCollider`，斜面则用 `MeshCollider`），并对复杂立面进行**方块裁切拼装**以减少碰撞器数量。

## 🖼️ 坐标转换与显示
系统内置了网格坐标、世界坐标与 **2.5D 显示坐标**之间的互转，方便实现 2.5D / 伪等距的像素表现：

| 方法 | 说明 |
| --- | --- |
| `GetWorldPosition(gridCoord)` | 网格坐标 → 世界坐标。 |
| `GetGridCoord(worldPosition)` | 世界坐标 → 网格坐标（含临界值防抖偏移）。 |
| `GetGridCoordFloat(worldPosition)` | 世界坐标 → 小数网格坐标。 |
| `GetGridCoordToViewPos(gridCoord)` | 三维网格坐标 → 2.5D 显示坐标（高度叠加到 Y，并用 Z 值区分渲染前后）。 |
| `GetWorldPosToViewPos(gridPos)` / `GetViewPosToWorldPos(gridPos)` | 三维世界坐标 ↔ 2.5D 显示坐标互转。 |

> 在 2.5D 投影中，物体的**高度会叠加到显示的 Y 轴**，同时以纵向坐标换算出**渲染深度**，从而让「更靠前 / 更高」的物体正确遮挡后方物体。`GridCellSystemManager` 中还保留了一套基于此的渲染排序队列（`AddGridItemSortInfo` 等，标注为旧方案）。

## 🛠️ 编辑器工具
菜单 `Tools/AleGridCellSystem/GridItemToolsWindow` 打开**网格物品工具窗口**，用于**批量创建 / 更新**网格物品预制体，以及由多个网格物品组成的**组装件（PreformedUnit）**预制体。

### Aseprite 资产管线
本工具的数据来源是 `Aseprite`：在 Aseprite 中绘制像素网格素材，并通过配套的 Lua 导出脚本把网格配置导出为 `.txt` 数据文本。工具解析该文本（头部含单元格像素尺寸等，逐条为每件物品的尺寸、位置、真实体积、脚本标记、图片列表等），再在 Unity 中据此自动生成：

- **物品预制体**：挂上项目的 GridItem 脚本并导入 `GridItemComponent` 数据；
- **显示节点 `ViewRoot`**：按图片列表生成子节点与 `MeshRenderer`，并生成对应的 Mesh（支持 `Cube` 方块与 `Slope` 斜面两种类型，斜面会读取贴图边缘像素来推算 UV）；
- **碰撞节点 `ColliderRoot`**：依据真实体积数据生成碰撞器。

### 生成预制件及批量网格物品
在窗口中完成必要配置后，点击 **「生成网格物品组预制件」** 即可批量生成。主要配置项：

- **工作流处理器脚本**：可选，用于把项目逻辑接入生成流程（见 [工作流扩展](#-工作流扩展)）。
- **网格物品图片文件夹**：`ViewRoot` 配图时从此文件夹按名称检索图片。
- **网格物品预制体检索文件夹**：用于复用已有预制体，避免重复创建。
- **网格物品数据文本**：Aseprite 导出的 `.txt`。
- **网格物品预制体导出文件夹**：生成结果的输出根目录。
- **网格物品脚本基类 / 脚本配置**：指定挂载的 GridItem 脚本；可按数据中的 `scriptTag` 为不同物品挂载不同子类脚本。

### 更新与配置
- **更新单个网格物品预制体**：针对单个既有预制体，选择性更新脚本类 / `GridItemComponent` / `ViewRoot` / `ColliderRoot`（更新碰撞器会重建，请谨慎，可能覆盖手工调整）。
- **导入 / 导出工具配置**：可保存 / 读取窗口配置，便于团队共享同一套生成参数。
- **额外功能**：整体组装件更新、批量设置组装件内所有 `ViewRoot` 的编辑器预览位置等。

## 🔌 工作流扩展
若希望在自动创建 / 更新流程中注入项目自定义逻辑，可继承以下两个类：

- **`GridItemToolsWindowProcessor`**：工作流的总入口。继承它并配置到工具窗口，即可接收 `OnCreateGridItemPrefab` / `OnUpdateGridItemPrefab` / `OnCreatePreformedUnitPrefab` 等回调。
- **`GridItemToolsWindowProcessorGridItemNode`**：针对**某一类** GridItem 的处理节点。重写 `GetTargetGridItemType()` 指定目标类型，再重写所需回调；`Processor` 会自动收集所有节点并按类型分发，结构更清晰、易于维护。

## 📁 目录结构
```text
Assets/PluginsDeveloper/AleGridCellSystem/
├─ Sources/
│  ├─ GridCellSystemManager.cs                       # 网格管理器 + 数据类型（GridCoord / GridItemData / AreaInfo …）
│  ├─ GridItemComponent.cs                           # 网格物品组件（组合式：数据 / 状态 / 定位 / 真实体积 / 显示）
│  ├─ Tools/
│  │  └─ GridSystemLibrary.cs                        # 运行时工具库（预留）
│  └─ Editor/
│     ├─ GridItemToolsWindow.cs                      # 网格物品工具窗口
│     ├─ GridItemToolsWindowConfig.cs                # 工具窗口配置（ScriptableObject）
│     ├─ GridItemToolsWindowProcessor.cs             # 工作流处理器基类
│     ├─ GridItemToolsWindowProcessorGridItemNode.cs # 单类型处理节点基类
│     └─ GridSystemEditorLibrary.cs                  # 编辑器核心库（Aseprite 解析 / Mesh / Collider 生成）
└─ Resource/
   └─ Mesh_Slope.asset                               # 斜面碰撞用 Mesh
```

## 📋 待办事项列表
- **网格系统**
  - 自定义每层（Layer）的单元格尺寸与初始化参数。
  - 更完善的多层交互与跨层查询接口。
- **渲染 / 工具**
  - 精简并统一渲染排序方案（当前保留了旧的排序队列）。
  - 预制体编辑场景在更新后的自动重载。
  - 补充演示场景（Samples）与截图 / GIF 说明。

## 📄 许可证
本项目基于 [GNU GPL v2.0](./LICENSE) 许可证发布。
