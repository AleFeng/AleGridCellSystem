<p align="center">
  <img alt="GitHub Release" src="https://img.shields.io/github/v/release/AleFeng/AleGridCellSystem?color=blue">
  <img alt="GitHub Downloads (all assets, all releases)" src="https://img.shields.io/github/downloads/AleFeng/AleGridCellSystem/total?color=green">
  <img alt="GitHub Repo License" src="https://img.shields.io/badge/license-GPL--2.0-blueviolet">
  <img alt="GitHub Repo Issues" src="https://img.shields.io/github/issues/AleFeng/AleGridCellSystem?color=yellow">
</p>

<p align="center">
  🌍
  <a href="./README.md">中文</a> |
  <a href="./README_EN.md">English</a> |
  日本語
</p>

<p align="center">
  📥
  <a href="#-インストール">インストール</a> |
  <a href="#-クイックスタート">クイックスタート</a>
</p>

# AleGridCellSystem - 3D グリッドセルシステム
AleGridCellSystem は `Unity` 向けの **3D グリッドセルシステム**で、**建築・配置・定位・エリア分割**といったグリッドベースのゲームプレイに使用できます。  
「グリッドのロジック」と「表示の見た目」を完全に分離しているのが特徴です。グリッド上のアイテムは**純粋なデータ**であり（`GameObject` や Unity のコライダーを必須としません）、1 つのアイテムが**複数セルにまたがり**、**向き**を持ち、**任意のステート**を保持し、占有サイズとは独立した**実体積（コリジョン）**を持てます。  
その上に、**定位システム**（周囲のアイテムと相対方向の検出）、**エリアシステム**（部屋分割とフォーカス断面表示）、そして `Aseprite` を用いた**エディタ用アセットパイプライン**（グリッドアイテムのプレハブ・メッシュ・コライダー・組み立て体をワンクリックで生成）を備えています。

> 名前空間は `FsGridCellSystem` で、以下の API はすべてこの名前空間に含まれます。

## 📜 目次
- [✨ 概要](#-概要)
  - [特徴](#特徴)
- [💻 動作環境](#-動作環境)
- [📦 インストール](#-インストール)
  - [手動インストール](#手動インストール)
  - [依存関係](#依存関係)
- [🧩 コアコンセプト](#-コアコンセプト)
- [🌱 クイックスタート](#-クイックスタート)
- [🧱 グリッドとセルの操作](#-グリッドとセルの操作)
- [🚦 ステートシステム](#-ステートシステム)
- [🎯 定位システム](#-定位システム)
- [🗂️ エリアシステム](#-エリアシステム)
- [🧊 実体積とコライダー](#-実体積とコライダー)
- [🖼️ 座標変換と表示](#-座標変換と表示)
- [🛠️ エディタツール](#-エディタツール)
  - [Aseprite アセットパイプライン](#aseprite-アセットパイプライン)
  - [プレハブの一括生成](#プレハブの一括生成)
  - [更新と設定](#更新と設定)
- [🔌 ワークフロー拡張](#-ワークフロー拡張)
- [📁 ディレクトリ構成](#-ディレクトリ構成)
- [📋 TODO リスト](#-todo-リスト)
- [📄 ライセンス](#-ライセンス)

## ✨ 概要
本システムを使えば、3D グリッド空間での配置 / 建築系のゲームプレイをすばやく実装できます。  
グリッドは**複数のレイヤー（Layer）**からなり、各レイヤーは `X（横）/ Y（奥行き）/ Z（高さ）` の座標を持つ 3D セル配列です。各セルには 1 つの**グリッドアイテム（GridItem）**を配置でき、アイテムは複数セルにまたがり、**主セル（Main）**と**子セル（Sub）**を区別します。どの子セルを参照しても主セルへ辿れます。  
アイテムはグリッド上では**純粋なデータ**なので、大量のアイテムを効率的に管理できます。表示レイヤー（`ViewRoot`）とコリジョンレイヤー（`ColliderRoot`）は、アイテムに付随する任意の要素であり、グリッドロジックとは独立しています。  
さらに**座標変換**（グリッド ↔ ワールド ↔ 2.5D 表示座標）と**描画深度ソート**を内蔵し、2.5D / 疑似アイソメトリック（isometric）風のピクセルゲームを実装しやすくしています。

### 特徴
| 特徴 | 説明 |
| --- | --- |
| 3D レイヤードグリッド | 複数レイヤーの 3D セル配列。`X` 横 / `Y` 奥行き / `Z` 高さ。セルのワールドサイズと各軸のセル数をカスタマイズ可能。 |
| 複数セル占有と向き | アイテムは複数セル（主セル + 子セル）にまたがり、4 方向（`Down/Left/Right/Up`）の向きに対応。左右向き時は占有サイズを自動で入れ替え。 |
| 配置と衝突チェック | アイテムを押し込む前に対象範囲の占有状況をチェックし、重なり配置を防止。 |
| ステートシステム | 各セルは任意の `int` ステート集合を保持でき、範囲に対して `All`（すべて）/ `Anyone`（いずれか）で判定。 |
| 定位システム | エミッター / レシーバーモデルで、範囲内のアイテムとその**相対方向**（上下左右前後）をリアルタイム検出。あり / なし / のみ / いずれか のマッチモードに対応。 |
| エリアシステム | グリッドを**エリアグループ**と**エリア**（部屋）に分割し、エリア内のアイテムを集計。**フォーカスエリア**（フォーカスより上のエリアを隠して階層断面表示）に対応。 |
| 占有 / コリジョンの分離 | 占有サイズと実際のコリジョン体積を**分離**。1 つのアイテムが複数の実体積を持ち、実行時に切り替え可能。 |
| 座標変換 | グリッド ↔ ワールド ↔ 2.5D 表示座標を相互変換。描画深度ソートを内蔵。 |
| Aseprite アセットパイプライン | Aseprite で描画・データ出力し、プレハブ・メッシュ・コライダー・組み立て体をワンクリック生成。 |
| 拡張可能なワークフロー | プロセッサースクリプトで生成 / 更新フローにフックし、アイテム種別ごとにプロジェクト独自ロジックを注入。 |

## 💻 動作環境
- `Unity 2021.3 LTS` 以降（ソースは C# 8 の `switch` 式などを使用しています）。
- **ランタイムコア**（`GridCellSystemManager`、`GridItemComponent` および各データ型）は `UnityEngine` のみに依存し、単体でプロジェクトロジックに組み込めます。
- **エディタ用アセットパイプライン**（プレハブ / メッシュ / コライダーの自動生成）は `Aseprite` から出力したグリッドデータテキストに依存します。詳細は [Aseprite アセットパイプライン](#aseprite-アセットパイプライン) を参照。

## 📦 インストール
### 手動インストール
本システムはソースコードで提供されます。プラグインフォルダをそのままプロジェクトへコピーしてください。

1. 本リポジトリをダウンロードまたはクローンします。
2. `Assets/PluginsDeveloper/AleGridCellSystem` フォルダ全体を、あなたのプロジェクトの `Assets` ディレクトリへコピーします。
3. Unity のコンパイル完了後、`FsGridCellSystem` 名前空間下の各種 API を利用できます。

> UPM（Package Manager）で git URL からインストールしたい場合は、プラグインのルートに `package.json`（および任意で `.asmdef` アセンブリ定義）を追加し、該当パスを参照してください。

### 依存関係
本プラグインは作者のフルプロジェクトから切り出したものです。**ランタイムコアは単体で使用できます**が、**一部のエディタ機能**は同梱されていない外部フレームワークを参照しています。

- `GridSystemEditorLibrary`（エディタライブラリ）は `EntrustSystem`、`ConfigSystem.Instance.CreateMaterial(...)`、および `GetOrAddComponent` などの拡張を使用します。
- `GridItemComponent` 内の**非推奨**（deprecated）な旧描画ソートメソッドが `GuildGridModel` を参照します。

これらのフレームワークがプロジェクトに無い場合は、統合時に上記のエディタ関連の参照を削除 / 置換するか、ランタイムコアのみを利用してください。

## 🧩 コアコンセプト
- **グリッドとレイヤー**：グリッド全体は `GridCellSystemManager` が管理し、内部は「レイヤー群」です。各レイヤーは `X × Y × Z` の 3D セル配列で、`Init(...)` でセルのワールドサイズ・各軸のセル数・レイヤー数を指定します。
- **座標系**：`GridCoord` では `X` が横、`Y` が奥行き、`Z` が**高さ**です。これは Unity とは異なる点に注意してください——**Unity では高さが Y 軸**であり、変換時にシステムが自動で Y / Z を入れ替えます。整数座標には `GridCoord`、小数精度が必要な場合は `GridCoordFloat` を使用し、両者は暗黙的に相互変換でき、`Vector3` とも相互運用できます。
- **グリッドアイテム（GridItem）**：`GridItemData` で表され、値 `Value`、種別 `EGridItemType`（`Main` / `Sub`）、主セル座標 `MainGridCoord`、占有サイズ `GridItemSize`、向き `Direction` を持ちます。向きが `Left` / `Right` のとき、占有サイズの X / Y は自動で入れ替わります（`GetGridItemSizeAtDirection`）。
- **コンポジション型の `GridItemComponent`**：ランタイムでのアイテム機能の担い手で、データ・ステート・実体積・表示・定位などの能力を内包します。プロジェクト独自の「GridItem 基底クラス」（`MonoBehaviour`）の**メンバーフィールドとして合成（コンポジション）**して使う設計です。クラスに `GridItemComponent` フィールドがあれば、グリッドアイテムの全機能を得られます。
- **実体積 `RealVolume`**：「アイテムがどのセルを占有するか」と「実際のコリジョン形状」は別物です。実体積はバウンディングボックスのサイズ・ローカル座標・対応するコライダーノードを表します。1 つのアイテムが複数の実体積を持ち、実行時に切り替えられます。

## 🌱 クイックスタート
以下は `GridCellSystemManager` を直接使ってグリッドデータを管理する例です。

```csharp
using FsGridCellSystem;
using UnityEngine;

// 1. グリッドを生成・初期化：セルのワールドサイズ 1×1×1、グリッド 20×20×10、1 レイヤー
var grid = new GridCellSystemManager();
grid.Init(
    cellUnitSizeX: 1f, cellUnitSizeY: 1f, cellUnitSizeZ: 1f,
    gridCellCountX: 20, gridCellCountY: 20, gridCellCountZ: 10,
    layerCount: 1);

// 2. 2×1×1、下向き、値 1001 のアイテムを (3,4,0) に配置
var data = new GridItemData
{
    Value        = 1001,
    GridItemType = EGridItemType.Main,
    MainGridCoord= new GridCoord(3, 4, 0),
    GridItemSize = new GridCoord(2, 1, 1),
    Direction    = EDirection.Down,
};

if (grid.PushMainGridItemData(0, data))
    Debug.Log("配置成功");
else
    Debug.Log("対象範囲が占有されているため配置失敗");

// 3. 取得：どの子セルからでも主セルの値へ辿れる
int value = grid.GetMainGridItemValue(0, new GridCoord(4, 4, 0)); // → 1001

// 4. 座標変換：グリッド ↔ ワールド
GridCoordFloat world = grid.GetWorldPosition(new GridCoord(3, 4, 0));
GridCoord coord      = grid.GetGridCoord(world);

// 5. アイテムを削除（すべての子セルも併せてクリア）
grid.RemoveMainGridItemData(0, new GridCoord(3, 4, 0));
```

## 🧱 グリッドとセルの操作
`GridCellSystemManager` はグリッドの読み書きと問い合わせ API を提供します（`layer` はレイヤーのインデックス）。

| メソッド | 説明 |
| --- | --- |
| `Init(cellUnitSizeX/Y/Z, gridCellCountX/Y/Z, layerCount)` | グリッドを初期化：セルのワールドサイズ、各軸のセル数、レイヤー数。 |
| `GetGridItem(layer, gridCoord, isNullCreateNew=false)` | 指定座標のセル（`GridItemComponent`）を取得。 |
| `GetMainGridItem(layer, gridCoord)` | 主セルを取得。子セルにヒットした場合はその主セルへ辿ります。 |
| `GetMainGridItemValue(layer, gridCoord)` | 主セルの値を取得。 |
| `PushMainGridItemData(layer, gridItemData)` | アイテムを押し込む。**事前に占有範囲の競合をチェック**し、競合時は `false` を返します。 |
| `SetMainGridItemValue(layer, gridItemData)` | 主セルの値を設定し、占有サイズに応じて子セルを敷き詰めます。 |
| `PopMainGridItemValue(layer, gridCoord)` | 主セルを取り出して削除（データのコピーを返す）。 |
| `RemoveMainGridItemData(layer, gridCoord)` | 主セルとそのすべての子セルを削除。 |
| `CheckGridItemSizeHasGirdItem(layer, gridCoord, gridItemSize)` | 指定サイズの範囲内にすでにアイテムがあるか検査。 |

> アイテムは `GridItemSize` 範囲内のすべてのセルを占有し、そのうち 1 つが**主セル**（完全なデータを保持）、残りが**子セル**（主セルを指す）です。これにより「複数セルの家具 / 建築」などを正しく配置・問い合わせ・一括削除できます。

## 🚦 ステートシステム
各セルはカスタムな `int` ステート集合（例：「予約済み」「通行不可」など）を保持でき、範囲に対して判定できます。

```csharp
const int STATE_RESERVED = 1;

// 2×1×1 の範囲全体にステートを付与
grid.SetGridItemState(0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1), STATE_RESERVED, true);

// 範囲内の「すべて」がステートを持つか判定
bool allReserved = grid.CheckGridItemSizeState(
    0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1),
    STATE_RESERVED, ECheckStateType.All);

// または「いずれか」のセルがステートを持つか判定
bool anyReserved = grid.CheckGridItemSizeState(
    0, new GridCoord(3, 4, 0), new GridCoord(2, 1, 1),
    STATE_RESERVED, ECheckStateType.Anyone);
```

## 🎯 定位システム
定位システムは、**あるアイテムの周囲範囲にどのアイテムがあり、それらが自分に対してどの方向にあるか**（上 / 下 / 左 / 右 / 前 / 後）を検出します。隣接判定やスナップ判定などに便利です。**エミッター（Emitter）/ レシーバー（Receiver）** モデルに基づきます。

- **エミッター**：レシーバーに感知されるアイテム。`EnableLocationEmitter = true`。
- **レシーバー**：範囲を能動的に走査するアイテム。`EnableLocationReceiver = true` と検出範囲の設定が必要。

```csharp
// 前提：アイテムの GridItemComponent をグリッドマネージャーへ渡す
gridItem.SetGridCellSystemManager(grid);

// レシーバー：自身の中心を基準に「上方向」3 セルの範囲を検出
gridItem.SetLocationCenter(ELocationCenterType.MiddleCenter);
gridItem.SetLocationReceiverRange(ELocationReceiverRangeType.Up, 3f);
gridItem.EnableLocationReceiver = true;

// 別のアイテムをエミッターとして設定
otherItem.EnableLocationEmitter = true;

// 問い合わせ：範囲内に「上方向」のエミッターが存在するか
bool hasAbove = gridItem.CheckLocationEmitterInfoDirection(
    ELocationCheckMode.Have, ELocationDirection.Up);

// 範囲内エミッター情報の変化を監視
gridItem.OnLocationEmitterInfoChange = item => { /* 見た目の更新など */ };
```

- 検出範囲タイプ `ELocationReceiverRangeType`：`Around`（全方向）、`LeftRight` / `FrontBack` / `UpDown`（軸方向）、および単方向の `Up/Down/Left/Right/Front/Back`。
- マッチモード `ELocationCheckMode`：`Have`（指定方向を含む）、`None`（含まない）、`Only`（完全一致のみ）、`Or`（いずれか 1 つを含む）。

## 🗂️ エリアシステム
エリアシステムは、グリッドを複数の**エリア（`AreaInfo`）**に分割し、それらを**エリアグループ（`AreaGroupInfo`）**へまとめます。部屋 / 階層 / 機能ゾーンなどの分割に使えます。

```csharp
int groupId = 1;

// グループ 1 にキッチンエリアを登録：起点 (0,0,0)、サイズ 5×5×3
grid.AddAreaInfo(groupId, new AreaInfo("Room_Kitchen", new GridCoord(0, 0, 0), new GridCoord(5, 5, 3)));

// ある座標がどのエリアに入るかを問い合わせ
AreaInfo area = grid.CheckInAreaInfo(new GridCoord(2, 2, 0));

// フォーカスエリア：このエリアにフォーカスし、それより「上」のエリアを隠す（例：上階の床を断面表示）
grid.SetFocusAreaInfo(area);

// フォーカス解除。すべてのエリアが再び可視に
grid.SetFocusAreaInfo(null);
```

- 各エリアは**境界内のグリッドアイテムを集計**し（`AddIntraGridItem` / `RemoveIntraGridItem`）、可視状態を一括で切り替えられます（`SetVisibleState`）。
- エリアは値 `Value`（設定テーブルの ID など）を保持でき、変化時に `OnValueChange` コールバックで通知します。
- エリアグループは配下の全エリアの**全体境界**を自動維持し、包含判定を高速に行えます。

## 🧊 実体積とコライダー
「アイテムがどのセルを占有するか」（`GridItemSize`）と「実際のコリジョン形状」は分離されています。後者は**実体積 `RealVolume`** が表します。

- 各 `RealVolume` は、**正規化されたバウンディングボックスのサイズ**、**ローカル座標**（占有範囲の左下隅を原点とする）、および結び付いた**コライダーノード GameObject** を含みます。
- 1 つのアイテムは**複数**の実体積を持ち、`SetRealVolumeCur(keyName)` で有効なコリジョンを実行時に切り替えられます（同一オブジェクトの異なる形態など）。
- エディタパイプラインは、Aseprite から出力された体積データに基づいてコライダーを自動生成し（`BoxCollider`、斜面は `MeshCollider`）、複雑な立面には**矩形の切り分け・組み立て**を行ってコライダー数を削減します。

## 🖼️ 座標変換と表示
グリッド座標・ワールド座標・**2.5D 表示座標**の相互変換を内蔵し、2.5D / 疑似アイソメトリックのピクセル表現を容易にします。

| メソッド | 説明 |
| --- | --- |
| `GetWorldPosition(gridCoord)` | グリッド座標 → ワールド座標。 |
| `GetGridCoord(worldPosition)` | ワールド座標 → グリッド座標（境界値のアンチジッター用オフセット付き）。 |
| `GetGridCoordFloat(worldPosition)` | ワールド座標 → 小数グリッド座標。 |
| `GetGridCoordToViewPos(gridCoord)` | 3D グリッド座標 → 2.5D 表示座標（高さを Y に加算し、Z で描画前後を区別）。 |
| `GetWorldPosToViewPos(gridPos)` / `GetViewPosToWorldPos(gridPos)` | 3D ワールド座標 ↔ 2.5D 表示座標の相互変換。 |

> 2.5D 投影では、オブジェクトの**高さが表示 Y 軸に加算**され、奥行き座標から**描画深度**が算出されるため、「より手前 / より高い」オブジェクトが後方のオブジェクトを正しく遮蔽します。`GridCellSystemManager` にはこれに基づく描画ソートキュー（`AddGridItemSortInfo` など、旧方式として保持）も残されています。

## 🛠️ エディタツール
メニュー `Tools/FsGridCellSystem/GridItemToolsWindow` から**グリッドアイテムツールウィンドウ**を開けます。グリッドアイテムのプレハブ、および複数のグリッドアイテムで構成される**組み立て体（PreformedUnit）**プレハブを**一括生成 / 更新**するためのツールです。

### Aseprite アセットパイプライン
本ツールのデータソースは `Aseprite` です。Aseprite でピクセルのグリッド素材を描き、付属の Lua 出力スクリプトでグリッド設定を `.txt` データテキストへ出力します。ツールはそのテキスト（ヘッダーにセルのピクセルサイズ等、続いて各アイテムのサイズ・位置・実体積・スクリプトタグ・画像リストなど）を解析し、Unity 上で以下を自動生成します。

- **アイテムプレハブ**：プロジェクトの GridItem スクリプトを付与し、`GridItemComponent` データをインポート。
- **表示ノード `ViewRoot`**：画像リストから子ノードと `MeshRenderer` を生成し、対応するメッシュ（`Cube`（立方体）と `Slope`（斜面）の 2 種に対応。斜面はテクスチャの端ピクセルを読み取って UV を推定）を生成。
- **コリジョンノード `ColliderRoot`**：実体積データに基づいてコライダーを生成。

### プレハブの一括生成
ウィンドウで必要な設定を済ませ、**「グリッドアイテムグループプレハブを生成」**をクリックすると一括生成できます。主な設定項目：

- **ワークフロープロセッサースクリプト**：任意。プロジェクトのロジックを生成フローへフックします（[ワークフロー拡張](#-ワークフロー拡張) を参照）。
- **グリッドアイテム画像フォルダ**：`ViewRoot` の画像は、このフォルダから名前で検索されます。
- **グリッドアイテムプレハブ検索フォルダ**：既存プレハブの再利用に使い、重複生成を防ぎます。
- **グリッドアイテムデータテキスト**：Aseprite から出力した `.txt`。
- **グリッドアイテムプレハブ出力フォルダ**：生成結果の出力ルートディレクトリ。
- **グリッドアイテム基底スクリプト / スクリプト設定**：付与する GridItem スクリプトを指定。データ内の `scriptTag` に応じて異なるサブクラススクリプトを付与できます。

### 更新と設定
- **単一グリッドアイテムプレハブの更新**：1 つの既存プレハブについて、スクリプトクラス / `GridItemComponent` / `ViewRoot` / `ColliderRoot` を選択的に更新します（コライダーの更新は再生成となるため、手動調整を上書きする可能性があります。慎重に）。
- **ツール設定のインポート / エクスポート**：ウィンドウ設定を保存 / 読み込みでき、チームで同じ生成パラメータを共有できます。
- **追加機能**：組み立て体全体の更新、組み立て体内すべての `ViewRoot` のエディタプレビュー位置の一括設定など。

## 🔌 ワークフロー拡張
自動生成 / 更新フローにプロジェクト独自ロジックを注入するには、以下の 2 クラスを継承します。

- **`GridItemToolsWindowProcessor`**：ワークフローの総合エントリーポイント。継承してツールウィンドウに設定すると、`OnCreateGridItemPrefab` / `OnUpdateGridItemPrefab` / `OnCreatePreformedUnitPrefab` などのコールバックを受け取れます。
- **`GridItemToolsWindowProcessorGridItemNode`**：**特定種別**の GridItem 用の処理ノード。`GetTargetGridItemType()` をオーバーライドして対象型を指定し、必要なコールバックをオーバーライドします。`Processor` が全ノードを自動収集して型ごとに振り分けるため、構造が明確で保守しやすくなります。

## 📁 ディレクトリ構成
```text
Assets/PluginsDeveloper/AleGridCellSystem/
├─ Sources/
│  ├─ GridCellSystemManager.cs                       # グリッドマネージャー + データ型（GridCoord / GridItemData / AreaInfo …）
│  ├─ GridItemComponent.cs                           # グリッドアイテムコンポーネント（合成：データ / ステート / 定位 / 実体積 / 表示）
│  ├─ Tools/
│  │  └─ GridSystemLibrary.cs                        # ランタイムユーティリティライブラリ（予約）
│  └─ Editor/
│     ├─ GridItemToolsWindow.cs                      # グリッドアイテムツールウィンドウ
│     ├─ GridItemToolsWindowConfig.cs                # ツールウィンドウ設定（ScriptableObject）
│     ├─ GridItemToolsWindowProcessor.cs             # ワークフロープロセッサー基底クラス
│     ├─ GridItemToolsWindowProcessorGridItemNode.cs # 種別ごとの処理ノード基底クラス
│     └─ GridSystemEditorLibrary.cs                  # エディタコアライブラリ（Aseprite 解析 / メッシュ / コライダー生成）
└─ Resource/
   └─ Mesh_Slope.asset                               # 斜面コリジョン用メッシュ
```

## 📋 TODO リスト
- **グリッドシステム**
  - レイヤーごとのセルサイズと初期化パラメータのカスタマイズ。
  - より充実したマルチレイヤー連携とレイヤー横断クエリ API。
- **描画 / ツール**
  - 描画ソート方式の整理・統一（現在は旧ソートキューを保持）。
  - 更新後のプレハブ編集シーンの自動リロード。
  - サンプルシーン（Samples）とスクリーンショット / GIF ドキュメントの追加。

## 📄 ライセンス
本プロジェクトは [GNU GPL v2.0](./LICENSE) ライセンスの下で公開されています。
