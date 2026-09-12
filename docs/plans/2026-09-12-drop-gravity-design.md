# 掉落物重力与物理设计稿

## 目标

- 让世界中所有掉落物（`drops[]`）具有真实物理行为：受重力下落、与实心方块碰撞停留在表面、在水中上浮直到漂浮在水面。
- 保留现有的「轻微上下浮动」视觉动画（纯渲染层，不影响物理位置）。

## 现状

- 当前 `drops[]` 的条目仅包含 `type, x, y, w, h, phase`，没有速度/物理状态。
- `updateDrops(dt)` 只负责：距离判定拾取 + `phase++`。掉落物生成后永远悬停在生成位置。
- `renderDrops()` 用 `floatY = sin(time*4 + phase) * 3` 做视觉浮动（不影响 `drop.y`）。
- 项目已有一套通用碰撞：`applyPhysicsAndCollisions(entity, dt)` + `resolveHorizontalCollisions` + `resolveVerticalCollisions`，基于 tile 级 `isSolid()`。

## 方案（已确认）

### 1) 数据结构扩展

在 `spawnDrop(type, x, y)` 为每个掉落物新增：

```js
{
  vx: 0,
  vy: 0,
  onGround: false
}
```

- 生成初速度：`vx=0, vy=0`（纯垂直下落，不做水平弹射爆出）。
- `onGround` 标记用于后续可优化（例如落到地面后不再做水判定/减重力计算），先保留字段。

### 2) updateDrops 物理段（每帧）

在原「拾取判定 + phase++」之前，插入物理段，按顺序执行：

1. **水 / 熔岩判定**：
   - 取 drop 中心的 tile：`tx = floor((x+w/2)/TILE_SIZE)`, `ty = floor((y+h/2)/TILE_SIZE)`
   - `tile = getTile(tx, ty)`
   - 若 `tile === WATER`：`vy = max(vy - 1800 * dt, -220)`（向上浮力，最大上浮速度 ~ 7 格/秒），**跳过下面正常重力**
   - 其他情况（熔岩 / 空气）：走正常重力。

2. **重力**：
   ```
   vy = min(vy + PHYSICS.gravity * dt, PHYSICS.maxFallSpeed)
   ```
   直接复用玩家 / 动物的重力常量，保持手感一致。

3. **水平位移 + 碰撞**：
   ```
   drop.x += drop.vx * dt
   resolveHorizontalCollisions(drop)
   ```
   （生成时 vx=0，之后每次都会命中「vx===0 → 跳过 collision」路径，开销低，为将来爆出弹跳预留）

4. **垂直位移 + 碰撞**：
   ```
   drop.y += drop.vy * dt
   drop.onGround = false
   const hitY = resolveVerticalCollisions(drop)
   if (hitY && drop.vy > 0) drop.onGround = true
   ```

5. **虚空清理**：
   - `drop.y > worldPixelHeight + TILE_SIZE * 10` → splice 移除（避免掉落到虚空后的泄漏）。

6. **原有逻辑**：
   - 距离拾取（半径 28，保持）
   - `phase += dt`（保持渲染浮动相位累积）

### 3) 渲染层

`renderDrops()` 的 `floatY = sin(...) * 3` 完全保留。
- 只加到绘制坐标 `py`，不改 `drop.y` 实际物理位置，因此：
  - 物理上落在地面是真正贴着地面
  - 视觉上仍有轻微浮动效果

## 验收标准

1. 击杀动物/挖掉方块后，掉落物从生成点开始受重力 **向下加速**，速率和玩家下落一致。
2. 掉落物 **不会穿透草 / 泥土 / 石头 / 木板 / 工作台 / 树叶 等实心方块**，会停在表面。
3. 掉落物掉进水里：**上浮直到贴在水面（WATER 顶层 tile）后漂浮**。
4. 掉落物掉出世界底部后自动消失，不残留。
5. 拾取（靠近 28 像素）和浮动视觉动画 **不受影响**。
