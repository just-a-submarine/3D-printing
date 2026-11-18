# CLAUDE.md — ROV Hull 3D 列印專案規範

## 專案結構

- `ROV_Hull.FCStd`：唯一 FreeCAD 工作檔（含 `ROV_Hull`、`ROV_EndCap_Front`、`ROV_EndCap_Rear`、`ROV_Fasteners` 物件）。**原地修改，禁止另存新檔**。
- `ROV_Hull.stl` / `ROV_EndCaps.stl`（前+後蓋）/ `ROV_Fasteners.stl`（螺絲×16+螺帽×16）：匯出時直接覆蓋同名檔。
- `docs/潛水艇外殼3D列印需求文件.md`：規格之源（目前 v5.1），§13 為完整災難重建腳本。
- `docs/建模日誌.md`：每次建模後必須新增條目（背景/決策/量測表）。
- `screenshots/`：截圖統一存放，命名 `YYYYMMDD_視角_描述.png`。
- `tasks/todo.md`：交接文件（≤100 行），任務完成後更新。

## 建模規則（FreeCAD MCP）

- 一律 Part 模組 CSG 布林法（`execute_code`），不用 PartDesign。
- 工作流程：讀需求文件 → 更新文件 → Pre-flight（list_documents、baseline BoundBox）→ 建模 →
  驗證（Solids=1 / isValid / BoundBox / 數值探針）→ 截圖 ≥4 視角 → 匯出 STL → 更新日誌。
- 每個特徵步驟後檢查 `isValid` 與 `Solids==1`，異常立即停（參考 §13 腳本的 `chk()`）。
- **等半徑曲面相切必偏移 0.1mm**（見 tasks/lessons.md、tasks/todo.md 改模前必讀 #1）。
- **艙內特徵（平臺/棚架/凹槽/電池艙）必須在所有 `cut bore()` 之後建，之後嚴禁再 cut bore()**（教訓 #5）。
- fillet 只能對「未布林的乾淨原始體」做（如馬達臂 box），禁止對布林後複雜體。
- **螺旋幾何（螺紋）布林會靜默失敗**（教訓 #8）：螺紋與芯柱必須 **fuzzy fuse(0.05)** 並斷言
  體積增量；`common(環形刀具, 螺旋面)` 會假陰性回 0，驗證改盒狀刀具＋cut 法交集。
- 局部修改優先；全參數變更（如改徑）才用 §13 重建腳本。

## v5.1 關鍵尺寸（不可隨意動）

OD=120 / 壁厚 1.5 / ID=117 / L=250；法蘭 OD=144×6、8×D4.8@BCD132（起始 22.5°）；
觀察窗 Z=210 墊座 R20 頂面 Y=62、孔 D=30.2；煙囪 Z=90 OD=32、錐孔內壁 D14→頂端 D26；
馬達孔一律 R15（垂直深 50、孔頂 Y=−43；水平深 45、孔頂 Z=189）；臂 fillet r5；
平臺頂 Y=−40（Z=8~242；內部兩段空心腔 Y=−55~−48 ∩ R57、Z=12~100 / 150~230、+Z 端 45° 收斜，
中段 100~150 實心）；電池艙 X±29 / Z=12~89 / 底 Y=−45（電池 75×56×66）；
配重凹槽 X±22.1 / Z=160~227.6 / 底 Y=−45（配重塊 65.6×42.2）；
墊圈凹槽 R53.7~58.6、槽底 Z=7/243（蓋環伸入 1mm、墊圈空間 2mm）；
列印螺絲芯 D3.0、牙頂 D4.4、pitch 2、牙深 0.7（真螺紋）。完整參數見需求文件 §13。

## 注意

- 舊 skill `freecad-rov-hull-csg` 的常數（OD=90、L=200、Workspace_cloud 路徑）已過時，
  方法論可用、參數以本 repo 需求文件為準。
- 列印參數用 `docs/Bambu_Studio_防水切片參數.md`。
- 列印螺絲僅能手緊；斷裂/滑牙改金屬 M4×20+螺帽（孔位相容）。
