<p align="center">
  <img alt="GitHub Release" src="https://img.shields.io/github/v/release/AleFeng/AleGridCellSystem?color=blue">
  <img alt="GitHub Downloads (all assets, all releases)" src="https://img.shields.io/github/downloads/AleFeng/AleGridCellSystem/total?color=green">
  <img alt="GitHub Repo License" src="https://img.shields.io/badge/license-MIT-blueviolet">
  <img alt="GitHub Repo Issues" src="https://img.shields.io/github/issues/AleFeng/AleGridCellSystem?color=yellow">
</p>

<p align="center">
  🌍
  <a href="./README.md">中文</a> |
  English |
  <a href="./README_JA.md">日本語</a>
</p>

<p align="center">
  📥
  <a href="#-installation">Install</a> |
  <a href="#-quick-start">Quick Start</a>
</p>

# AleGridCellSystem - 3D Grid Cell System
AleGridCellSystem is a **3D grid cell system** for `Unity`, designed for grid-based gameplay such as **building, placement, positioning, and area partitioning**.  
It fully decouples "grid logic" from "visual representation": items in the grid are **pure data** (they do not require a `GameObject` or a Unity collider). A single item can **span multiple cells**, have an **orientation**, carry **arbitrary states**, and own a **real collision volume** that is independent of its footprint.  
On top of this it provides a **location system** (detect nearby items and their relative direction), an **area system** (room partitioning with a focus/cutaway view), and an `Aseprite`-based **editor asset pipeline** that generates grid-item prefabs, meshes, colliders, and assembled units with one click.

> The namespace is `AleGridCellSystem`; all APIs below live under it.

## 📜 Table of Contents
- [✨ Introduction](#-introduction)
  - [Features](#features)
- [💻 Requirements](#-requirements)
- [📦 Installation](#-installation)
  - [Via UPM (recommended)](#via-upm-recommended)
  - [Manual install](#manual-install)
  - [Dependencies](#dependencies)
- [🧩 Core Concepts](#-core-concepts)
- [🌱 Quick Start](#-quick-start)
- [🧱 Grid & Cell Operations](#-grid--cell-operations)
- [🚦 State System](#-state-system)
- [🎯 Location System](#-location-system)
- [🗂️ Area System](#-area-system)
- [🧊 Real Volume & Colliders](#-real-volume--colliders)
- [🖼️ Coordinate Conversion & Display](#-coordinate-conversion--display)
- [🛠️ Editor Tooling](#-editor-tooling)
  - [Aseprite asset pipeline](#aseprite-asset-pipeline)
  - [Batch-generating prefabs](#batch-generating-prefabs)
  - [Updates & config](#updates--config)
- [🔌 Workflow Extension](#-workflow-extension)
- [📁 Project Structure](#-project-structure)
- [📋 To-Do List](#-to-do-list)
- [📄 License](#-license)

## ✨ Introduction
With this system you can quickly build placement/construction gameplay in a 3D grid space.  
The grid is made of **several layers**, each being a 3D array of cells with coordinates `X (horizontal) / Y (depth) / Z (height)`. Each cell can hold one **grid item (GridItem)**; an item may span multiple cells and distinguishes a **Main cell** from **Sub cells** — querying any sub cell resolves back to its main cell.  
Items are **pure data** in the grid, so large numbers of them can be managed efficiently. The display layer (`ViewRoot`) and collision layer (`ColliderRoot`) are optional pieces attached to an item, independent of the grid logic.  
The system also has built-in **coordinate conversion** (grid ↔ world ↔ 2.5D display) and **render depth sorting**, making it easy to build 2.5D / pseudo-isometric pixel-art games.

### Features
| Feature | Description |
| --- | --- |
| 3D layered grid | Multi-layer 3D cell arrays: `X` horizontal / `Y` depth / `Z` height, with customizable cell world size and per-axis cell counts. |
| Multi-cell footprint & orientation | Items can span multiple cells (main + sub cells) and support four orientations (`Down/Left/Right/Up`); footprint auto-swaps when facing left/right. |
| Placement & conflict check | Before pushing an item, the target range is checked for occupancy to prevent overlapping placement. |
| State system | Each cell can carry an arbitrary set of `int` states, checked over a range with `All` or `Anyone` semantics. |
| Location system | Emitter/receiver model that detects nearby items and their **relative direction** (up/down/left/right/front/back), with Have / None / Only / Or matching modes. |
| Area system | Partition the grid into **area groups** and **areas** (rooms), track items inside an area, and support a **focus area** (hide areas above the focus level for a cutaway floor view). |
| Footprint / collision separation | The occupied footprint and the actual collision volume are **decoupled**; an item can own multiple real volumes and switch between them at runtime. |
| Coordinate conversion | Convert between grid, world, and 2.5D display coordinates, with built-in render depth sorting. |
| Aseprite asset pipeline | Draw in Aseprite, export data, and generate prefabs, meshes, colliders, and assembled units in one click. |
| Extensible workflow | Hook into the create/update flow via a processor script to inject project-specific logic per item type. |

## 💻 Requirements
- `Unity 2021.3 LTS` or newer (the source uses C# 8 features such as `switch` expressions).
- The **runtime core** (`GridCellSystemManager`, `GridItemComponent`, and the data types) depends only on `UnityEngine` and can be integrated into your project logic on its own.
- The **editor asset pipeline** (auto-generating prefabs / meshes / colliders) relies on grid data text exported from `Aseprite` — see [Aseprite asset pipeline](#aseprite-asset-pipeline).

## 📦 Installation
### Via UPM (recommended)
Install through the Unity Package Manager using a git URL:

1. Open `Window → Package Manager`.
2. Click the `+` in the top-left → `Add package from git URL...`.
3. Paste the address below and click `Add`:

```
https://github.com/AleFeng/AleGridCellSystem.git?path=/Assets/PluginsDeveloper/AleGridCellSystem
```

Or add it directly to the `dependencies` of your project's `Packages/manifest.json`:

```json
"com.alefeng.alegridcellsystem": "https://github.com/AleFeng/AleGridCellSystem.git?path=/Assets/PluginsDeveloper/AleGridCellSystem"
```

> To pin a version, append a tag/branch name to the URL, e.g. `...AleGridCellSystem#1.1.0`.

### Manual install
1. Download or clone this repository.
2. Copy the entire `Assets/PluginsDeveloper/AleGridCellSystem` folder into the `Assets` directory of your own project.
3. Wait for Unity to compile, then use the APIs under the `AleGridCellSystem` namespace.

### Dependencies
The plugin is **self-contained and compiles standalone**; both the runtime and editor assemblies depend only on Unity itself — no third-party frameworks.

- The editor's auto-generation pipeline takes grid data text exported from `Aseprite` as input (see [Aseprite asset pipeline](#aseprite-asset-pipeline)).
- The shader used for generated display meshes is configurable via `GridSystemConfig.viewMaterialShaderName` (default `Able/Lit-Alpha`); if that shader is not found in the project, it automatically falls back to the built-in `Sprites/Default`.

## 🧩 Core Concepts
- **Grid & layers**: The whole grid is managed by `GridCellSystemManager`, internally a set of layers — each layer is an `X × Y × Z` 3D cell array. `Init(...)` specifies the cell world size, per-axis counts, and layer count.
- **Coordinate system**: In `GridCoord`, `X` is horizontal, `Y` is depth, and `Z` is **height**. Note this differs from Unity, where **height is the Y axis** — the system swaps Y/Z during conversion. Use `GridCoord` for integer coordinates and `GridCoordFloat` when you need decimals; the two convert implicitly and also interop with `Vector3`.
- **Grid item**: Described by `GridItemData` — the value `Value`, type `EGridItemType` (`Main` / `Sub`), main-cell coordinate `MainGridCoord`, footprint `GridItemSize`, and orientation `Direction`. When the orientation is `Left` / `Right`, the footprint's X / Y auto-swaps (`GetGridItemSizeAtDirection`).
- **Composition-based `GridItemComponent`**: This is the runtime carrier of an item's capabilities — data, state, real volume, display, and location. It is designed to be **composed** as a member field inside your project's own "GridItem base class" (a `MonoBehaviour`): as long as your class has a `GridItemComponent` field, it gains all grid-item capabilities.
- **Real volume `RealVolume`**: "Which cells an item occupies" and "its actual collision shape" are two different things. A real volume describes a bounding-box size, a local position, and the associated collider node; an item can have multiple real volumes and switch between them at runtime.

## 🌱 Quick Start
The example below uses `GridCellSystemManager` directly to manage grid data:

```csharp
using AleGridCellSystem;
using UnityEngine;

// 1. Create and initialize a grid: cell world size 1×1×1, grid 20×20×10, 1 layer
var grid = new GridCellSystemManager();
grid.Init(
    cellUnitSizeX: 1f, cellUnitSizeY: 1f, cellUnitSizeZ: 1f,
    gridCellCountX: 20, gridCellCountY: 20, gridCellCountZ: 10,
    layerCount: 1);

// 2. Place a 2×1×1, downward-facing item with value 1001 at (3,4,0)
var data = new GridItemData
{
    Value        = 1001,
    GridItemType = EGridItemType.Main,
    MainGridCoord= new GridCoord(3, 4, 0),
    GridItemSize = new GridCoord(2, 1, 1),
    Direction    = EDirection.Down,
};

if (grid.PushMainGridItemData(0, data))
    Debug.Log("Placed");
else
    Debug.Log("Target range is occupied; placement failed");

// 3. Query: any sub cell resolves back to the main cell's value
int value = grid.GetMainGridItemValue(0, new GridCoord(4, 4, 0)); // → 1001

// 4. Coordinate conversion: grid ↔ world
GridCoordFloat world = grid.GetWorldPosition(new GridCoord(3, 4, 0));
GridCoord coord      = grid.GetGridCoord(world);

// 5. Remove the item (clears it together with all its sub cells)
grid.RemoveMainGridItemData(0, new GridCoord(3, 4, 0));
```

## 🧱 Grid & Cell Operations
`GridCellSystemManager` provides read/write and query APIs for the grid (`layer` is the layer index):

| Method | Description |
| --- | --- |
| `Init(cellUnitSizeX/Y/Z, gridCellCountX/Y/Z, layerCount)` | Initialize the grid: cell world size, per-axis cell counts, layer count. |
| `GetGridItem(layer, gridCoord, isNullCreateNew=false)` | Get the cell (`GridItemComponent`) at a coordinate. |
| `GetMainGridItem(layer, gridCoord)` | Get the main cell; if a sub cell is hit, it resolves back to its main cell. |
| `GetMainGridItemValue(layer, gridCoord)` | Get the main cell's value. |
| `PushMainGridItemData(layer, gridItemData)` | Push an item; **checks the footprint range for conflicts** beforehand and returns `false` on conflict. |
| `SetMainGridItemValue(layer, gridItemData)` | Set the main cell's value and lay out sub cells per the footprint. |
| `PopMainGridItemValue(layer, gridCoord)` | Pop and remove the main cell (returns a copy of its data). |
| `RemoveMainGridItemData(layer, gridCoord)` | Remove the main cell and all of its sub cells. |
| `CheckGridItemSizeHasGridItem(layer, gridCoord, gridItemSize)` | Check whether a range of a given size already contains an item. |

> An item occupies every cell within `GridItemSize`; one of them is the **main cell** (holding the full data) and the rest are **sub cells** (pointing to the main cell). This lets multi-cell furniture/buildings be placed, queried, and removed as a whole.

## 🚦 State System
Each cell can carry a set of custom `int` states (e.g. "reserved", "blocked"), checked over a range:

```csharp
const int STATE_RESERVED = 1;

// Add a state to a whole 2×1×1 range
grid.SetGridItemState(0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1), STATE_RESERVED, true);

// Check whether "all" cells in the range have the state
bool allReserved = grid.CheckGridItemSizeState(
    0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1),
    STATE_RESERVED, ECheckStateType.All);

// Or check whether "any" cell has the state
bool anyReserved = grid.CheckGridItemSizeState(
    0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1),
    STATE_RESERVED, ECheckStateType.Anyone);
```

## 🎯 Location System
The location system **detects which items are within a range around a given item, and their direction relative to it** (up / down / left / right / front / back), useful for adjacency detection, snapping, and similar gameplay. It is based on an **Emitter / Receiver** model:

- **Emitter**: an item that receivers can sense — `EnableLocationEmitter = true`.
- **Receiver**: an item that actively scans a range — `EnableLocationReceiver = true` plus a configured detection range.

```csharp
// Prerequisite: give the item's GridItemComponent to the grid manager
gridItem.SetGridCellSystemManager(grid);

// Receiver: relative to its own center, detect a 3-cell range "above"
gridItem.SetLocationCenter(ELocationCenterType.MiddleCenter);
gridItem.SetLocationReceiverRange(ELocationReceiverRangeType.Up, 3f);
gridItem.EnableLocationReceiver = true;

// Another item acts as an emitter
otherItem.EnableLocationEmitter = true;

// Query: is there an emitter located "above" within range?
bool hasAbove = gridItem.CheckLocationEmitterInfoDirection(
    ELocationCheckMode.Have, ELocationDirection.Up);

// Listen for changes to the in-range emitter info
gridItem.OnLocationEmitterInfoChange = item => { /* refresh visuals, etc. */ };
```

- Detection range types `ELocationReceiverRangeType`: `Around` (all directions), `LeftRight` / `FrontBack` / `UpDown` (axial), and single-direction `Up/Down/Left/Right/Front/Back`.
- Matching modes `ELocationCheckMode`: `Have` (contains the given direction), `None` (does not contain), `Only` (exact match), `Or` (contains any one).

## 🗂️ Area System
The area system partitions the grid into **areas (`AreaInfo`)** grouped into **area groups (`AreaGroupInfo`)**, useful for rooms / floors / functional zones:

```csharp
int groupId = 1;

// Register a kitchen area in group 1: origin (0,0,0), size 5×5×3
grid.AddAreaInfo(groupId, new AreaInfo("Room_Kitchen", new GridCoord(0, 0, 0), new GridCoord(5, 5, 3)));

// Find which area a coordinate falls into
AreaInfo area = grid.CheckInAreaInfo(new GridCoord(2, 2, 0));

// Focus area: focus this area and hide areas "above" it (e.g. cut away upper floors)
grid.SetFocusAreaInfo(area);

// Clear the focus; all areas become visible again
grid.SetFocusAreaInfo(null);
```

- Each area **tracks the grid items within its bounds** (`AddIntraGridItem` / `RemoveIntraGridItem`) and can toggle their visibility as a whole (`SetVisibleState`).
- An area can carry a `Value` (e.g. a config-table ID) and notify via the `OnValueChange` callback when it changes.
- An area group automatically maintains the **overall bounds** of all its areas for fast containment checks.

## 🧊 Real Volume & Colliders
"Which cells an item occupies" (`GridItemSize`) is separate from "its actual collision shape". The latter is described by a **real volume `RealVolume`**:

- Each `RealVolume` contains a **normalized bounding-box size**, a **local position** (origin at the lower-left corner of the footprint), and a bound **collider node GameObject**.
- An item can own **multiple** real volumes and switch the active one at runtime via `SetRealVolumeCur(keyName)` (e.g. different forms of the same object).
- The editor pipeline auto-generates colliders from the volume data exported by Aseprite (`BoxCollider`, or `MeshCollider` for slopes), and performs **square cut-and-assemble** on complex facades to reduce the collider count.

## 🖼️ Coordinate Conversion & Display
The system has built-in conversion between grid, world, and **2.5D display** coordinates, making 2.5D / pseudo-isometric pixel art easy:

| Method | Description |
| --- | --- |
| `GetWorldPosition(gridCoord)` | Grid coordinate → world coordinate. |
| `GetGridCoord(worldPosition)` | World coordinate → grid coordinate (with a threshold anti-jitter offset). |
| `GetGridCoordFloat(worldPosition)` | World coordinate → fractional grid coordinate. |
| `GetGridCoordToViewPos(gridCoord)` | 3D grid coordinate → 2.5D display coordinate (height added onto Y, Z used to order render depth). |
| `GetWorldPosToViewPos(gridPos)` / `GetViewPosToWorldPos(gridPos)` | Convert between 3D world and 2.5D display coordinates. |

> In the 2.5D projection, an object's **height is added onto the display Y axis**, while its depth coordinate is converted into a **render depth**, so that objects that are "more in front / higher up" correctly occlude those behind. `GridCellSystemManager` also keeps a render-sorting queue based on this (`AddGridItemSortInfo`, etc., marked as the legacy approach).

## 🛠️ Editor Tooling
The menu `Tools/AleGridCellSystem/GridItemToolsWindow` opens the **Grid Item Tools window**, used to **batch create / update** grid-item prefabs as well as **assembled-unit (PreformedUnit)** prefabs composed of multiple grid items.

### Aseprite asset pipeline
The tool sources its data from `Aseprite`: draw pixel grid assets in Aseprite and, via a companion Lua export script, export the grid configuration to a `.txt` data file. The tool parses that text (a header with the cell pixel size, then per-item size, position, real volumes, script tag, image list, etc.) and generates the following in Unity:

- **Item prefab**: attaches the project's GridItem script and imports `GridItemComponent` data;
- **Display node `ViewRoot`**: generates child nodes and `MeshRenderer`s from the image list, plus the corresponding meshes (supports `Cube` and `Slope` types; slopes read the texture's edge pixels to infer UVs);
- **Collision node `ColliderRoot`**: generates colliders from the real-volume data.

### Batch-generating prefabs
After completing the required configuration in the window, click **"Generate grid-item group prefab"** to batch-generate. Key settings:

- **Workflow processor script**: optional, to hook project logic into the generation flow (see [Workflow Extension](#-workflow-extension)).
- **Grid-item images folder**: images for `ViewRoot` are looked up by name from this folder.
- **Grid-item prefab search folder**: used to reuse existing prefabs and avoid duplicates.
- **Grid-item data text**: the `.txt` exported from Aseprite.
- **Grid-item prefab export folder**: the output root directory for generated results.
- **Grid-item base script / script settings**: specify the GridItem script to attach; different subclass scripts can be attached per the `scriptTag` in the data.

### Updates & config
- **Update a single grid-item prefab**: selectively update the script class / `GridItemComponent` / `ViewRoot` / `ColliderRoot` of one existing prefab (updating the collider rebuilds it — use with care, as it may overwrite manual tweaks).
- **Import / export tool config**: save/load the window configuration so a team can share the same generation parameters.
- **Extras**: whole-assembly updates, batch-setting the editor-preview positions of all `ViewRoot`s within an assembly, and more.

## 🔌 Workflow Extension
To inject project-specific logic into the auto create/update flow, subclass the following two classes:

- **`GridItemToolsWindowProcessor`**: the overall entry point of the workflow. Subclass it and assign it in the tool window to receive callbacks like `OnCreateGridItemPrefab` / `OnUpdateGridItemPrefab` / `OnCreatePreformedUnitPrefab`.
- **`GridItemToolsWindowProcessorGridItemNode`**: a processing node for **one kind** of GridItem. Override `GetTargetGridItemType()` to specify the target type, then override the callbacks you need; the `Processor` automatically collects all nodes and dispatches by type, keeping things clear and maintainable.

## 📁 Project Structure
```text
Assets/PluginsDeveloper/AleGridCellSystem/
├─ Sources/
│  ├─ GridCellSystemManager.cs                       # Grid manager + data types (GridCoord / GridItemData / AreaInfo …)
│  ├─ GridItemComponent.cs                           # Grid-item component (composition: data / state / location / real volume / display)
│  ├─ Tools/
│  │  └─ GridSystemLibrary.cs                        # Runtime utility library (reserved)
│  └─ Editor/
│     ├─ GridItemToolsWindow.cs                      # Grid Item Tools window
│     ├─ GridItemToolsWindowConfig.cs                # Tool window config (ScriptableObject)
│     ├─ GridItemToolsWindowProcessor.cs             # Workflow processor base class
│     ├─ GridItemToolsWindowProcessorGridItemNode.cs # Per-type processing node base class
│     └─ GridSystemEditorLibrary.cs                  # Editor core library (Aseprite parsing / Mesh / Collider generation)
└─ Resource/
   └─ Mesh_Slope.asset                               # Mesh for slope collision
```

## 📋 To-Do List
- **Grid system**
  - Per-layer cell sizes and initialization parameters.
  - Richer multi-layer interaction and cross-layer query APIs.
- **Rendering / tools**
  - Streamline and unify the render-sorting scheme (a legacy sort queue is currently retained).
  - Auto-reload of the prefab editing scene after updates.
  - Add sample scenes and screenshots / GIF documentation.

## 📄 License
This project is released under the [MIT](./LICENSE) license.
