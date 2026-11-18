# ROV Hull 3D 列印 todo
最後更新：2026-11-19

## 現況（交接）
- ROV 潛水艇 PETG 外殼：FreeCAD（MCP `execute_code`，Part CSG 布林法）建模，Bambu Lab A1 + PETG HF 列印。
- 目前版本 **v5.1**（2026-07-19）。接手先讀本檔 + `CLAUDE.md` + `docs/潛水艇外殼3D列印需求文件.md`（規格之源，§13 參數區與災難重建腳本）。
- 尺寸、特徵、列印參數已寫在 README 與需求文件，這裡不重複。
- 工作檔：根目錄 `ROV_Hull.FCStd`（唯一，原地修改禁止另存；物件 `ROV_Hull`、`ROV_EndCap_Front`、`ROV_EndCap_Rear`、`ROV_Fasteners`）。
- 匯出：`ROV_Hull.stl`、`ROV_EndCaps.stl`（前+後蓋）、`ROV_Fasteners.stl`（螺絲×16+螺帽×16），覆蓋同名檔。
- 舊路徑 `D:\Workspace_cloud\School_Project\Electronics-2\3D列印\V4\` 已失效（搬遷前位置）。
- 版本歷程細節在 `docs/建模日誌.md`。

### v5.1 驗證結果（2026-07-19）
- HULL：BoundBox 202×172×250、Solids=1、isValid、Volume **482,823 mm³**（≈613 g，挖空省 88 g）；12 個探針全過（空腔、頂板 3 mm、腔底、配重塊放入、電池艙／馬達孔／引線孔／墊圈凹槽未受損）。
- 螺絲 371.8 mm³（螺紋 +36.5）、螺帽 233.7 mm³（內牙扇形 76% ≈ 理論）；過 D4.8 孔干涉 0、旋合干涉 0（cut 法 + fuse 體積和法交叉驗證）。
- 截圖：`screenshots/20260719_*_v51_rebuild.png`（4 視角）+ `20260719_closeup_v51_fastener_threads.png`。

### v5.1 設計決策
- 螺紋遺失根因：`core.fuse(thread)` 回 Solids=2 被 common 丟掉 → 改 **fuzzy fuse(0.05)**。規格：芯 D3.0、牙深 0.7、牙頂 D4.4（可過 D4.8 孔）、pitch 2；螺帽刀具 +0.25 徑向間隙。
- 平臺兩段封閉空心腔（Y=−55~−48 ∩ R57 ∩ X±32；Z=12~100、150~230，+Z 端 45° 收斜可列印）；中段 Z=100~150 實心保護垂直馬達殼。頂板一般 8 mm、電池艙／配重凹槽下 3 mm。
- 配重凹槽 X±22.1、Z=160~227.6、深 5（配重塊 65.6×42.2 + 2 mm 間隙）。

### 改模前必讀（完整版見 `tasks/lessons.md`）
1. 半徑相等曲面相切 = OCC 退化：切割半徑偏移 0.1 mm（如觀察窗孔 15.1 vs 球殼 15.0）；fuse 也一樣（平臺／棚架外半徑取 58.6 埋入壁內）。
2. 布林中途壞掉不會立刻報錯：每步 `chk()`（isValid + Solids==1），壞了立刻 raise。
3. 內壁是弧面：徑向孔最小孔徑會小於名目值，規格是「最小」時補直圓柱刀（錐孔 R7×4）。
4. 數值探針位置要對：錐孔頂面才達最大徑（D26 探針放 Y=69.9）。
5. 艙內特徵（平臺／棚架／馬達殼根部）必須在所有 `cut bore()` 之後 fuse，之後嚴禁再 cut bore()。
6. fillet 只能對未布林的乾淨原始體（臂 box 先 fillet 再 fuse）。
7. 螺旋幾何布林會靜默失敗：用 fuzzy fuse(0.05) 並斷言體積增量；`common(環形刀具, 螺旋面)` 會假陰性回 0 → 改盒狀刀具 + cut 法交集，關鍵結論雙方法交叉。
8. `freecad-csg-modeling`／`freecad-rov-hull-csg` skills 內舊參數（OD=90 等）已過時。

## 待辦
- [ ] 首印實測：螺栓孔 D4.8（M4 需 ≥4.0）、馬達孔 D30（緊配，過緊砂磨）、列印螺絲旋合手感（牙深 0.7 接近 FDM 極限，過緊反覆旋合磨合）。
- [ ] 下水配重：淨浮力約 +790 g，配重塊放前段凹槽，與後段電池艙配平。
- [ ] 墊圈自裁：OD≈117／ID≈108、厚 2.5~3 mm（內胎疊 2~3 層），壓縮量 = 厚 − 2 mm。

## 已知問題／刻意不處理
- 列印螺絲只能手緊；斷裂改金屬 M4×20（孔位相容）。
- 切片用 `docs/Bambu_Studio_防水切片參數.md`：殼體直立（後端朝下），臂／法蘭需 tree support；端蓋平面朝下免支撐；螺絲直立建議 0.12~0.16 層高。
- `docs/建模日誌.md` 舊條目提到 CONTEXT.md 屬歷史紀錄，不改。
