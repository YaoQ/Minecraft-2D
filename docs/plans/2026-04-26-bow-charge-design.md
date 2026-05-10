# 弓箭蓄力伤害设计稿

## 目标

- 当玩家快捷栏选中弓（`bow`）时：
  - 右键按下开始蓄力
  - 右键松开发射弓箭
  - 按住时间越久，弓箭伤害越高（不封顶）

## 现状

- 右键当前是“立即发射”：
  - `mousedown` → `handleMouseAction` → `tryShootBow` → `shootArrow`
- 玩家箭伤害当前为固定值（需改为随蓄力时间增长）

## 交互规则

- 右键按下（且当前选中 `bow`）：
  - 进入蓄力状态，不立刻发射
  - 期间更新瞄准目标点（鼠标移动）
  - 不触发“吃食物 / 放置方块”
- 右键松开：
  - 若处于蓄力状态则发射，并退出蓄力状态
  - 生存模式在发射时检查并消耗 `arrow`（没有箭则不发射）

## 伤害计算（不封顶）

- `holdSec = (nowMs - bowChargeStartMs) / 1000`
- `damage = 4 + floor(holdSec / 0.3)`
  - 0 秒：4 点
  - 每约 0.3 秒：+1 点
  - 不封顶（按住越久越高）

## 数据与实现点

- `player` 新增字段：
  - `isBowCharging`
  - `bowChargeStartMs`
  - `bowChargeTargetX / bowChargeTargetY`
- 新增事件：
  - `mouseup`：松开时发射（用 `window` 监听，避免鼠标移出画布导致不触发）
  - `mousemove`：蓄力时更新目标点

## 验收标准

- 选中弓时右键按住不会立刻射箭
- 松开右键会发射弓箭
- 按住越久，命中伤害越高（可明显体验）
- 不选弓时，右键行为保持原有逻辑（吃食物 / 放置方块）

