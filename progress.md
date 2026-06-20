# Empire Wars — Upgrade Progress

Single self-contained file: `index.html` (vanilla JS + Tailwind CDN, no build step).
All tunable numbers live in the `CONFIG` object at the top of the `<script>`.

## Done

### Tier 0 — Foundation
- **Persistence**: versioned save (`SAVE_VERSION = 2`) to `localStorage` storing `globalCoins`,
  `playerTech`, `maxUnlockedLevel`, `settings`, and lifetime `stats`. Auto-saves on every
  War Room purchase, match end, settings change, and boot. Feature-detects `localStorage`
  (`storageOK`) and falls back to in-memory `memSave` when blocked (e.g. claude.ai artifacts).
- **Save migration**: `migrate()` upgrades pre-v2 saves in place (adds `settings`/`stats`,
  backfills missing unit levels) so meta-progression never wipes. Verified with a headless test.
- **Fixed dead/broken code**:
  - `boss` unit now actually spawns (milestone campaigns + endless boss waves).
  - Boss render branch rewritten under a real `shape === 'boss'` case (old unreachable
    `isBoss` branch removed; champion vs boss art separated).
  - Milestone campaigns (5/10/15/20) now play differently — they are boss battles.
- **Performance**:
  - Minimap rendered to a small `<canvas>` (`updateMinimap`) instead of rebuilding DOM dots
    every frame.
  - HUD DOM updates throttled to ~12 fps (`hudTimer`), not every frame.
  - Object pools for particles (`particlePool`) and projectiles (`projectilePool`).
  - Live unit counts (`counts.player/cpu`) replace per-frame `units.filter()`.

### Tier 1 — Combat depth
- **Damage types** (slashing / piercing / magic / siege) × **armor classes**
  (light / heavy / shielded) resolved through `CONFIG.counter` matrix in `computeDamage()`.
- **Distinct mechanics**: splash (pyro), cleave (champion), armor-pen (crossbow),
  charge bonus (knight/spearman), anti-hero (spearman vs heroes/bosses), stealth + backstab
  (assassin ignores aggro until close, bonus vs ranged/healers), shield-wall (shield grants
  damage reduction to allies behind it via `updateShieldWalls()`).
- **Targeting priorities**: ranged → backline, melee → nearest frontline,
  assassin → dive healers/ranged, healer → lowest-HP-ratio ally.

### Tier 2 — Player agency
- **Spell bar** with cooldowns + gold cost, cast by tapping the battlefield:
  Meteor (AoE burst), Freeze (stun zone), Rally (heal + buff), Reinforce (instant squad),
  Gold Surge (economy spike). Double-tap battlefield to jump to the front line.

### Tier 3 — Boss & milestone battles
- Boss spawns at campaigns 5/10/15/20 after a dramatic entrance delay, scaled by tier.
  Phases/specials: ground-slam AoE, summon adds, enrage below 33% HP. Dedicated boss HP banner.

### Tier 4 — Smarter AI + difficulty
- 4 tiers (Easy / Normal / Hard / Brutal) in `CONFIG.difficulties` scaling AI cadence,
  counter-pick chance, advanced-unit/champion usage, and gold — not just raw stats.
- AI reads player army composition (`aiPickUnit`) and counter-picks armor/role.

### Tier 5 (partial) — Modes & stats
- **Endless / Survival** mode with escalating waves, biome shifts, and wave bosses.
- **Lifetime stats**: wins, losses, best campaign, total kills, games played, most-used unit.

### Tier 6 — Game feel & audio
- Screen shake on big hits, hit-stop on hero/boss kills, ring/zone FX, styled floating text.
- Synth background-music loop (toggle), layered SFX. Pause menu, 1×/2×/3× speed, restart.

### Tier 7 — UX / mobile / accessibility
- Settings menu: sound, music, screen shake, colorblind markers, default speed, reset
  (with confirm). Colorblind team markers (▲ player / ■ enemy). Off-screen action arrows.
  Bigger tap targets on the troop bar.

### Tier 8 — Visual identity
- Biome variation per campaign tier: grassland → snow → volcanic → void.

## Session 3 — feedback fixes
- **Upgrades now improve every combat stat**, not just HP/DMG. `getStat` scales range
  (+3%/lvl), move speed (+2%/lvl) and attack rate (cooldown −2.5%/lvl, floored) via
  `CONFIG.statGrowth`; the unit constructor reads these scaled values. Tooltips and War Room
  cards now show Range / Fire-rate / Speed climbing with level. (Match gold cost still rises
  with level by design — it's the economy lever, not a combat stat; easy to flatten if wanted.)

## Session 2 — feedback fixes
- **Richer unit tooltips**: now show level, HP, DMG, range, fire rate (attacks/sec),
  move speed, melee/ranged/support, damage type, armor class, and a one-line special
  ability (`UNIT_ABILITY`). Roster rebuilds on match start so values reflect upgrades.
- **Assassin targeting fixed**: it no longer sprints across the whole map to a far backline
  target and dies. It now only dives healers/ranged within ~700px (`pri*400 - d`) and
  otherwise fights the nearest enemy, so it actually connects.
  - **Follow-up (root cause):** that still let it walk *past* a frontline blocker to reach
    a nearby archer, so it never fought the soldiers in front of it. Now it targets the
    nearest enemy outright (only lunging to a squishy within ~70px), keeping its ×2 backstab
    bonus in `computeDamage`. Verified headlessly: it engages a knight blocker (280→92 HP)
    instead of walking past it.
- **Scroll while a spell is armed**: removed `touch-action:none` on the casting canvas and
  added tap-vs-drag detection (`pressMoved`) — a clean tap casts, a drag scrolls.
- **Visual upgrade tiers** (`tierFor` / `TIER`): units and fortresses change appearance as
  they level — glowing rank outline (bronze→silver→gold→crystal), brighter weapon metal,
  aura ring at tier 2, crown at tier 3; castles gain taller corner towers, flags, gold
  trim, crenellations, a glow, and rank stars.

## Next / deferred (nice-to-have, not in Definition of Done)
- Loadout/deck (pick 6 of N) and unlock-gating of units.
- Talent-tree branches / prestige in the War Room (currently linear +25% upgrades).
- Seeded daily challenge.
- Per-unit charge VFX, parallax background.

## Known issues / notes
- Particles & floating text update once per render frame, so at 2×/3× they animate at 1×
  speed (cosmetic only; simulation itself is correctly stepped N times).
- Tailwind via CDN requires network on first load; gameplay logic is fully offline.

## Balance notes
- CPU economy: `startGold 150 + 50/level`, income `10 + 3/level`, base HP `3000 + 1000/level`.
- Counter matrix favors: piercing→heavy, magic→shielded, siege→structures/shielded,
  slashing→light. Spearman gets ×2.2 vs heroes; crossbow armor-pen 0.6; assassin ×2 backstab.
- Spell costs/cooldowns tuned in `CONFIG.spells`; meteor 320 dmg / 14s, freeze 3s / 18s.
- Testing target: Campaign 1 winnable in <5 min on Normal; Campaign 20 hard-but-fair.
  Re-tune `CONFIG.cpu.*` and `CONFIG.difficulties.*` if playtests skew.
