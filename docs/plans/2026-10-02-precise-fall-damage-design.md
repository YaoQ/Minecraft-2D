# 摔落伤害精细公式设计稿

## 目标

- 把摔落伤害从「按整格（`Math.floor`）结算」改为「按实际像素高度精算」，小数部分也计入伤害（例如掉 4.3 格就扣 2.3 hp）。
- 玩家和陆生动物共享同一套算法（仍走 `applyFallDamage`），避免手感差异。

## 现状

- 目前的摔落结算：
  ```
  fallBlocks = Math.floor(fallPx / TILE_SIZE);
  damage = Math.max(0, fallBlocks - thresholdBlocks);
  ```
- 后果：**4.0 格和 4.9 格扣血一样**，都是 2 血，阶梯感很强，手感不连续。
- `applyDamage(entity, amount)` 内部直接 `hp -= amount`，天然支持浮点，所以改完不需要额外改伤害管线。

## 方案（已确认 A）

### 1) 公式更新

在 `applyFallDamage(entity, wasOnGround, options)` 中：

```
fallBlocks = fallPx / TILE_SIZE     // 去掉 Math.floor，保留小数
damage = Math.max(0, fallBlocks - thresholdBlocks)
```

- 不再强制整格，2.0–2.9 格全部都会扣 0 点到 0.9 点（2.0 刚好 0，不扣）
- 和 MC Java 的「按实际 fall distance（浮点）」手感一致。

### 2) 联动更新

- **玩家**：因为玩家也用 `applyFallDamage`，自动享受到浮点精细伤害。
- **动物**：同上（陆生）。
- **死亡逻辑**：`hp <= 0` 判定保持不变，浮点 hp 会在 `0.x` 时还活着，再受一次伤害再死，更自然。

## 验收标准

1. 从 2.0 格高度（约 32 px）落下：**不扣血** ✅ （`max(0, 2 - 2) = 0`）
2. 从 3.0 格落下：扣 **1.0** 血
3. 从 3.5 格落下：扣 **1.5** 血（原旧版会扣 1.0，阶梯差异消除）
4. 从 5.7 格落下：扣 **3.7** 血
5. 玩家和动物的 hp 在 UI（目前动物不画血条，只有 hurtTimer 闪红）/ 玩家血条（`hp/maxHp`）里都能体现连续递减（因为 `applyDamage` 直接存浮点 hp，血条天然就是连续的）。
