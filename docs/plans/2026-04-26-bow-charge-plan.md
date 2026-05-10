# Bow Charge Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Make player bow arrows gain higher damage the longer right-click is held, firing on release.

**Architecture:** Add a small “bow charging” state machine on the player, intercept right mouse down/up to start/finish charging, and compute arrow damage from held duration at release.

**Tech Stack:** Single-file HTML5 Canvas game (`index.html`) with vanilla JavaScript.

---

### Task 1: Add player bow charging state + input hooks

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Add player fields**
- Extend `createPlayer()` to include:
  - `isBowCharging`
  - `bowChargeStartMs`
  - `bowChargeTargetX`
  - `bowChargeTargetY`

**Step 2: Add event listeners**
- Add `mousemove` listener (canvas) to update charge target while charging.
- Add `mouseup` listener (window) to release the shot when charging.

**Step 3: Manual verification**
- Run a local server and ensure no console errors on load.

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add bow charging state"
```

---

### Task 2: Start charging on right mouse down (bow selected)

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Intercept right-click**
- In the canvas `mousedown` path:
  - If inventory is not open, button is right-click, and selected hotbar type is `bow`:
    - Start charging (set start time + target)
    - Do not execute the existing right-click actions (food/place)

**Step 2: Manual verification**
- Holding right-click does not fire instantly anymore.
- Releasing right-click without movement still fires toward the initial cursor position.

**Step 3: Commit**
```bash
git add index.html
git commit -m "feat: start bow charge on right click"
```

---

### Task 3: Release shot on mouse up with damage scaling (unbounded)

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/index.html`

**Step 1: Implement release logic**
- On right `mouseup` when `player.isBowCharging`:
  - `holdSec = (nowMs - bowChargeStartMs) / 1000`
  - `damage = 4 + Math.floor(holdSec / 0.3)` (unbounded)
  - In survival: consume `arrow` on release; if no arrow, do not fire.
  - Fire via `shootArrow({ from:"player", ... damage })`
  - Apply `player.bowCooldown` after firing.

**Step 2: Manual verification**
- Quick tap does ~4 damage.
- Holding longer increases damage noticeably.
- In survival, no arrows => no shot.

**Step 3: Commit**
```bash
git add index.html
git commit -m "feat: scale bow arrow damage by charge time"
```

---

### Task 4: Update README controls text (optional but recommended)

**Files:**
- Modify: `/Users/yao/project/Minecraft-2D/README.md`

**Step 1: Document bow charging**
- In the right-click section, mention:
  - Bow: hold right-click to charge, release to shoot; longer hold = higher damage.

**Step 2: Commit**
```bash
git add README.md
git commit -m "docs: document bow charging"
```

