# World Height 262 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Increase world height to 262 tiles while adding the extra space above (more sky), not below.

**Architecture:** Introduce a `SKY_OFFSET` and shift terrain-related Y values (`surfaceY`, `seaLevel`, `LAYER_OFFSET_Y`) downward by that offset. Compute player spawn from the generated surface height.

**Tech Stack:** Single-file HTML5 Canvas game (`index.html`) with vanilla JavaScript.

---

### Task 1: Add constants for world height + vertical offset

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Update constants**
- Set `WORLD_HEIGHT = 262`
- Add `BASE_WORLD_HEIGHT = 180` and `SKY_OFFSET = WORLD_HEIGHT - BASE_WORLD_HEIGHT`
- Update `LAYER_OFFSET_Y` to include the offset

**Step 2: Manual verification**
- Page loads without runtime errors.

**Step 3: Commit**
```bash
git add index.html
git commit -m "feat: set world height to 262"
```

---

### Task 2: Shift terrain generation to add sky above

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Shift column terrain**
- In `getColumnTerrain(x)`:
  - Increase `seaLevel` by `SKY_OFFSET`
  - Increase biome bases (`base`) by `SKY_OFFSET`
  - Shift the clamp range by `SKY_OFFSET`

**Step 2: Manual verification**
- World renders correctly.
- Terrain is visibly lower with more air above.

**Step 3: Commit**
```bash
git add index.html
git commit -m "feat: shift terrain downward for higher sky"
```

---

### Task 3: Compute player spawn from surface height

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Make spawn dynamic**
- Change `PLAYER_SPAWN` from a fixed `const` to a `let`.
- After `createWorld()` (when `surfaceHeights` and `biomeTypes` exist), set:
  - `spawnTileX` = preferred tile X (fallback to nearby land if initial spot is sea)
  - `PLAYER_SPAWN.x = spawnTileX * TILE_SIZE`
  - `PLAYER_SPAWN.y = (surfaceHeights[spawnTileX] - 2) * TILE_SIZE`

**Step 2: Manual verification**
- Player spawns on/near land surface and does not fall from the sky.

**Step 3: Commit**
```bash
git add index.html
git commit -m "feat: compute player spawn from terrain"
```

---

### Task 4: Update README world height

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/README.md`

**Step 1: Update documentation**
- Update the “世界大小”描述，将高度写为 `262`。

**Step 2: Commit**
```bash
git add README.md
git commit -m "docs: update world height to 262"
```

