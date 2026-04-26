# 蜘蛛（Spider）设计稿

## 目标

- 新增怪物：蜘蛛（`spider`）
- 属性：
  - 血量：16
  - 近战攻击伤害：3
- 行为：
  - 仅夜晚刷新
  - 可在墙上爬（方案 B：贴墙检测 + 持续上爬）
- 掉落：
  - 蜘蛛丝（`string`）：1–2
  - 蜘蛛眼（`spider_eye`）：0–1

## 约束与原则

- 保持单文件原型风格：所有逻辑仍在 `index.html` 内
- 保持“资源优先级”一致：
  - 优先 `assets/vanilla/...`（用户本机提取）
  - 缺失时自动回退到开源贴图/程序生成贴图
- 不引入新的外部依赖

## 数据模型

- `ANIMAL_STATS.spider = { hp: 16, drops: [...] }`
- `createAnimal("spider", x)` 返回的实体字段沿用现有结构，必要时新增：
  - `attackCooldown`（复用僵尸/骷髅已有字段模式）
  - `climbGrace`（可选，减少贴墙抖动；短时间内允许继续爬）

## 刷新规则

- 新增 `SPIDER_SPAWN = { timer, interval, maxSpiders }`
- 夜晚刷新函数：
  - `updateSpiderSpawning(dt)`：仅在 `isNightTime()` 时计时刷新
  - `countSpiders()`：统计 `animal.kind === "spider"`
  - `spawnSpiderNearPlayer()`：参考 `spawnZombieNearPlayer()` / `spawnSkeletonNearPlayer()`，在玩家附近陆地表面生成
- 在主循环中调用 `updateSpiderSpawning(dt)`

## AI 与爬墙（方案 B）

### 追击与攻击

- 与僵尸类似：朝玩家方向移动（水平追击）
- 若与玩家 AABB 重叠且冷却结束：对玩家造成 3 点伤害

### 贴墙检测

使用方块查询 `getTile/isSolid` 在蜘蛛身体中段附近检测左右侧是否存在实心方块：

- 当蜘蛛向右移动时：
  - 检测右侧一列（`txRight = floor((x + w) / TILE_SIZE)`）
  - 检测高度范围：从 `y + h*0.25` 到 `y + h*0.85` 覆盖几格 `ty`
  - 若任一格 `isSolid(getTile(txRight, ty))`，视为“贴右墙”
- 向左移动同理检测“贴左墙”

### 上爬规则

- 当“贴墙”且蜘蛛仍在朝墙方向移动时：
  - 将 `vy` 维持为负值（例如 `vy = Math.min(vy, -CLIMB_SPEED)`）
  - 同时保持水平速度不变，让其沿墙向上移动
- 仍使用现有 `applyPhysicsAndCollisions` 处理碰撞，保持物理一致性

## 渲染与资源

- 实体贴图：
  - `VANILLA_ENTITY_TEXTURE_PATHS.spider = ${VANILLA_TEXTURE_BASE}/entity/spider/spider.png`
  - 缺失则使用程序生成占位精灵（新增 `generateTextureForKey` 的 `entity:spider` 分支或复用 base sprite 生成逻辑）
- 物品：
  - 新增 `string`、`spider_eye` 的显示名、颜色、贴图映射与生成回退

## 验收标准

- 夜晚能在玩家附近刷新蜘蛛，白天不再新刷
- 蜘蛛血量为 16，近战命中玩家造成 3 点伤害
- 蜘蛛遇到垂直墙面能向上爬，能绕过 1–2 格高度的阻挡追击玩家
- 击杀蜘蛛会掉落蜘蛛丝与蜘蛛眼（数量符合设定）
- 不提供 `assets/vanilla` 资源时依旧可运行，贴图缺失能正常回退渲染

