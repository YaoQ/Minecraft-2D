# Spider Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a new monster “spider” (16 HP, 3 damage) that spawns at night, can climb walls, and drops string + spider eye.

**Architecture:** Extend the existing single-file entity system in `index.html`: add stats + spawn controller + AI update function. Implement wall-climb via side-tile adhesion checks that constrain vertical velocity while pushing into a wall.

**Tech Stack:** HTML5 Canvas + Vanilla JavaScript (single `index.html`), in-browser runtime, optional vanilla texture override via `assets/vanilla`.

---

### Task 1: Add spider data + drop items (string, spider_eye)

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Add stats and drops (minimal)**
- Add `ANIMAL_STATS.spider = { hp: 16, drops: [...] }`
- Decide drops:
  - `string` x (1–2) and `spider_eye` x (0–1)

**Step 2: Add item display + color**
- Add `DROP_COLORS.string`, `DROP_COLORS.spider_eye`
- Add `formatItemName()` mappings for `string` / `spider_eye`

**Step 3: Manual verification**
- Run `python3 -m http.server 8000` in project root.
- Open `http://127.0.0.1:8000/`
- Ensure inventory UI can render new drop items (even if not obtainable yet).

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add spider items and stats"
```

---

### Task 2: Add spider textures (vanilla override + fallback)

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Add texture path mappings**
- Add to `ITEM_TEXTURE_PATHS` (or generated key strategy) for `string`, `spider_eye` as generated items.
- Add to `VANILLA_ITEM_TEXTURE_PATHS`:
  - `string -> item/string.png`
  - `spider_eye -> item/spider_eye.png`
- Add to `VANILLA_ENTITY_TEXTURE_PATHS`:
  - `spider -> entity/spider/spider.png`

**Step 2: Add fallback sprite generation**
- Extend the generated sprite code so `entity:spider` renders as a distinct placeholder (wider body, darker tone).

**Step 3: Manual verification**
- Reload the game; confirm:
  - If vanilla assets are absent, the game still draws placeholders (no crashes).
  - Drops/UI show generated/fallback icons for the new items.

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add spider and item texture mapping"
```

---

### Task 3: Implement spider AI (chase + melee) without wall-climb

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Create `updateSpider(animal, dt)`**
- Behavior:
  - Move horizontally toward the player like zombie-style chase.
  - Optional small random drift to avoid perfect tracking.
  - When overlapping player AABB and cooldown is ready: `applyDamage(player, 3)` and set cooldown.

**Step 2: Wire into `updateAnimals(dt)`**
- Add a `spider` branch: `else if (animal.kind === "spider") updateSpider(animal, dt)`

**Step 3: Manual verification**
- Temporarily spawn one spider by:
  - Reusing an existing spawn function pattern, or
  - Temporarily calling `animals.push(createAnimal("spider", player.x + 200))` once at load.
- Confirm it chases and hits for 3 damage.

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add spider melee AI"
```

---

### Task 4: Implement wall-climb (Approach B: adhesion side checks)

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Add a helper `isSpiderTouchingWall(animal, dir)`**
- Use `getTile()` + `isSolid()` to sample tiles at the spider’s side.
- Sample multiple `ty` values from ~25% to ~85% of body height.

**Step 2: Apply climb constraint in `updateSpider`**
- When pushing into a wall in the movement direction and the side-check returns true:
  - Force/limit vertical velocity upward: `animal.vy = Math.min(animal.vy, -CLIMB_SPEED)`
  - Keep horizontal intent so it “sticks” to the wall while rising.
- Keep `applyPhysicsAndCollisions(animal, dt)` as the physics integrator.

**Step 3: Manual verification**
- Build a vertical wall (stack solid blocks).
- Spawn a spider at the base and stand above/behind the wall.
- Expected: spider climbs upward along the wall and can reach you over 1–2 block obstacles.

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add spider wall climbing"
```

---

### Task 5: Add night-only spider spawning

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Add spawn controller**
- Add `SPIDER_SPAWN = { timer, interval, maxSpiders }`
- Implement:
  - `countSpiders()`
  - `spawnSpiderNearPlayer()` (land only, surface spawn, like zombie/skeleton)
  - `updateSpiderSpawning(dt)` (night-only; reset timer on daytime)

**Step 2: Wire into main loop**
- Call `updateSpiderSpawning(dt)` in `gameLoop` alongside zombie/skeleton spawning.

**Step 3: Manual verification**
- Toggle to night:
  - spiders begin spawning near player.
- Toggle to day:
  - new spiders stop spawning.

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add night spider spawning"
```

---

### Task 6: Polish (drops + balance) and update README

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`
- Modify: `/Users/yao/project/Minecraft-2D/README.md`

**Step 1: Ensure drops match spec**
- On spider death, verify `spawnAnimalDrops` drops the configured items.
- Ensure both `string` and `spider_eye` appear and can be picked up.

**Step 2: Update README**
- Add spider to “生物种类/怪物” list.
- Mention spider wall-climb, night spawn, and drop items.

**Step 3: Manual verification**
- Kill multiple spiders; confirm drop rates and pickup.
- Confirm no console errors while assets are missing (fallback works).

**Step 4: Commit**
```bash
git add index.html README.md
git commit -m "docs: document spider monster"
```

---

## Execution Choice

Plan complete and saved to `docs/plans/2026-04-26-spider-plan.md`.

Two execution options:

1. **Subagent-Driven (this session)** — dispatch one focused subagent per task, review between tasks, fast iteration
2. **Parallel Session (separate)** — open a new session and execute the plan with checkpoints using `superpowers:executing-plans`

Which approach do you want?

