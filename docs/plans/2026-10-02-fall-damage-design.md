# 所有生物的摔落伤害设计稿

## 目标

- 让世界中**所有陆生生物**（猪、牛、羊、鸡、僵尸、骷髅、蜘蛛）从高处落地时受到与玩家一致的**摔落伤害**（掉落格数超过 2 后开始扣血）。
- 水生生物（鱼、鱿鱼）保持不受伤。
- 玩家当前的摔落伤害逻辑**抽成通用函数**复用，避免以后参数漂移。

## 现状

- 玩家已实现摔落伤害：在 `createPlayer()` 里有 `player.airbornePeakY`，落地时调用 `updatePlayerFallDamage(player, wasOnGround)`，规则：
  - 2 格以内安全
  - 超过 2 格后，**damage = 实际掉落格数 - 2**（向下取整）
- 动物有物理碰撞（`applyPhysicsAndCollisions` + `onGround`），但**没有任何摔落结算**：无论从多高掉下 `hp` 都不变。
- 动物的数据结构里也**没有** `airbornePeakY` 这类字段，无法直接照抄玩家算法。

## 方案（已确认）

### 1) 抽通用函数：`applyFallDamage(entity, wasOnGround, options)`

```js
function applyFallDamage(entity, wasOnGround, { thresholdBlocks = 2 } = {}) {
  if (!entity) return;
  if (typeof entity.airbornePeakY !== "number") {
    entity.airbornePeakY = entity.y;
  }
  const isLanded = wasOnGround === false && entity.onGround === true && entity.vy >= 0;
  if (isLanded) {
    const fallPx = Math.max(0, entity.y - entity.airbornePeakY);
    const fallBlocks = fallPx / TILE_SIZE;
    if (fallBlocks > thresholdBlocks) {
      const damage = Math.floor(fallBlocks - thresholdBlocks);
      if (damage > 0) applyDamage(entity, damage);
    }
    entity.airbornePeakY = entity.y;
    return;
  }
  // 空中：记录当前最高点（y 最小 = 最高位置）
  if (entity.y < entity.airbornePeakY) {
    entity.airbornePeakY = entity.y;
  }
}
```

- **玩家也改为调这个函数**（参数保持 `thresholdBlocks = 2`），保证未来改动一致。
- 使用 `entity.onGround` 的「off→on 且 `vy >= 0`」作为落地判定（和玩家现有逻辑相同）。

### 2) 玩家字段复用

- `player.airbornePeakY` 已存在，无需改动。
- `updatePlayerFallDamage` 函数可以**保留作为 wrapper**（避免其他调用点找不到），内部直接调用 `applyFallDamage(player, wasOnGround)`；也可以直接删掉、在 `updatePlayer` 里替换调用。实现时选「保留 wrapper」，改动风险最低。

### 3) 动物接入

**A. createAnimal 初始化字段**

在 `createAnimal(kind, x)` 新增：

```js
animal.airbornePeakY = y;   // y 是动物生成位置 y
```

（使用生成时的实际 `y`，而不是固定常量，避免平地小跳误触发大掉落）

**B. updateAnimals 统一结算**

为**陆生动物**在各自 AI 逻辑之后调用：

```
applyFallDamage(animal, wasOnGround)
```

陆生分支包括：

- `kind === "pig" / "sheep" / "cow" / "chicken"`
- `kind === "zombie"`  （走 `updateZombie` 分支，结算放在其 return 之后即可）
- `kind === "skeleton"`（走 `updateSkeleton` 分支）
- `kind === "spider"`  （走 `updateSpider` 分支）

**水生生物跳过**：

- `kind === "fish"` —— 不调用（保持不摔）
- `kind === "squid"` —— 不调用

实现细节：在 `updateAnimals(dt)` 开头先缓存 `wasOnGround = animal.onGround`（在任何 AI/物理之前），等物理跑完之后再结算，和玩家流程一致。

### 4) 与动物死亡掉落联动

- `applyFallDamage` 内部使用 `applyDamage(entity, damage)`：
  - 如果 `hp <= 0`，会走 `killAnimal(animal)` → `spawnAnimalDrops(animal)`（现有路径）
  - 因此从悬崖摔死的动物会**正常掉落物品 + 经验球**（保持和「被玩家击杀」的非经验部分一致）
- 摔死的动物**不算玩家击杀**（不会给经验球，保持现有 `killAnimal` 逻辑——经验球目前只有玩家主动击杀才给）

## 验收标准

1. 玩家在平地掉落 ≤ 2 格 **不扣血**（保持原有）。
2. 玩家从 5 格高落到地面：扣 **3** 血（5-2=3）。
3. 猪/牛/羊/鸡 从 6 格高落到地面：扣 **4** 血（6-2=4），hp<4 时直接死亡并掉落对应物品。
4. 僵尸 / 骷髅 / 蜘蛛 从 8 格高落到地面：扣 **6** 血，骷髅 hp 不足会摔死并掉骨头/弓/箭（和被击杀相同）。
5. 鱼 / 鱿鱼 在水中上下移动**不掉血**，被冲上岸摔在地上按陆生规则结算（如果发生的话）。
6. 蜘蛛爬墙上去再跳下来：按「最高点 → 落地」的差计算，不会因为爬墙过程中的 `onGround` 抖动被错误重置（因为只在落地 off→on 且 `vy>=0` 时才结算并重置）。
