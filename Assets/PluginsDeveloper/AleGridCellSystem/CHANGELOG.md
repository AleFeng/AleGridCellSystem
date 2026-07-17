# Changelog
本项目所有值得注意的变更都会记录在此文件中。
格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [1.1.0] - 2026-07-17
### 修复（Fixed）
- 数值解析统一改用 `InvariantCulture`：修复在小数分隔符为逗号的系统区域（如德 / 法 / 俄语环境）下 Aseprite 数据导入解析出错的问题（`GridSystemEditorLibrary`）。
- `GridCellSystemManager` 坐标校验补充 `layer < 0` 保护，避免负层索引抛 `IndexOutOfRange`。
- 组装件生成改用 `PrefabUtility.InstantiatePrefab`，替代 Unity 6 已弃用的 `ConnectGameObjectToPrefab`（消除 CS0618 警告，并生成正确的嵌套预制体实例）。

### 变更（Changed）
- 更正公共方法拼写 `CheckGridItemSizeHasGirdItem` → `CheckGridItemSizeHasGridItem`；旧名保留为 `[Obsolete]` 转发别名（暂不影响现有调用）。

### 移除（Removed）
- ⚠️ **破坏性**：移除已弃用的旧场景渲染排序方案 —— `GridItemSortInfo` 类、`GridCellSystemManager` 的 `AddGridItemSortInfo` / `RemoveGridItemSortInfo` / `ClearAllViewSortGridItem`、`GridItemComponent.SortInfo`、`GridCoordFloat.Magnitude`。若外部代码引用了这些成员，请改用自有的渲染排序逻辑。

## [1.0.0] - 2026-07-17
首个 UPM 版本。

### 新增（Added）
- 三维分层网格、多格占用与朝向、状态系统、定位系统、区域系统、真实体积 / 碰撞、坐标转换与 2.5D 显示、Aseprite 编辑器资产管线、工作流扩展。
- 运行时与编辑器程序集定义（`.asmdef`）与 `package.json`，支持通过 Package Manager 以 git URL 安装。

### 修复（Fixed）
- 坐标除法运算符 Y 分量误用 X 分量（`GridCoordFloat` / `GridCoord` 的 `operator /`）。
- 定位接收者检测遍历取错列表，导致越界 / 逻辑错误（`ExecuteLocationReceiverCheck`）。
- `GridItemComponent.EnableLocationEmitter` 误判接收者标志。
- 区域组移除区域后整体边界重算失效（`index` 从不自增）。
- `GridCoordFloat.Magnitude` 求值错误（嵌套 sqrt）。
- 斜面碰撞 Mesh 路径硬编码导致加载失败，改为按资源名检索。

### 变更（Changed）
- 命名空间由 `FsGridCellSystem` 统一重命名为 `AleGridCellSystem`（含菜单路径 `Tools/AleGridCellSystem/...`）。
- 移除对外部框架（EntrustSystem / ConfigSystem / GuildGridModel）的依赖；编辑器材质创建改为可配置着色器并回退到内置着色器，使插件可独立编译运行。
- 为 `GridCoord` 补充值语义的 `Equals` / `GetHashCode`。
- `GridItemToolsWindowConfig.cs` 由 GBK 重新编码为 UTF-8。

### 移除（Removed）
- 临时的 `GenerateMesh` 编辑器菜单与相关代码。
