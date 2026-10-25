# DigitAlaya — fun entrance animations

Source of truth (kept identical):
- `/workspace/digital-india/index.html`
- `/workspace/digital-india/BalBharatiPublicSchoolRohini_NityantGupta.html`
- Deploy mirrors: `/workspace/digital-india-deploy/` (same filenames)

## Classes

| Class | Effect |
|-------|--------|
| `.reveal` | Base: fade + slight rise/scale → final |
| `.reveal--pop` | Scale ~0.88 → 1 |
| `.reveal--rise` | TranslateY ~28px → 0 |
| `.reveal--tilt` | Slight rotate + scale (desktop); phones drop rotate |
| `.reveal--left` / `--right` | Slide from side |
| `.reveal--soft` | Gentler rise, longer duration |
| `.reveal--fade` | Opacity only (sticky UPI stage, hero art) |
| `.reveal-stagger` | Children enter with uneven delays (~95–140ms organic) |
| `.pillars-cascade` | Nine Pillars only: per-card spring entrance when clearly on screen |

Grid children also get alternating transforms via `:nth-child(6n+…)` so a bento/services row does not feel samey.

**Nine Pillars (per-card spring):** `.pillars` has `reveal-stagger pillars-cascade`, but the tall parent is **not** used to unlock children. Each `.pillar` has its own IntersectionObserver (`threshold` 0.35/0.5/0.65, `rootMargin: -10% 0px -22% 0px`) so pillar N only gets `.is-in` when that card sits in the mid band — not when the section header peeks. `forceNearViewport` / `forceStuck` use the same mid-band gate (`r.top < vh*0.82 && r.bottom > vh*0.18`) and never bulk-mark off-screen pillars. Same-frame batch delay ~0–70ms only; CSS nth-child cascade delays removed. Spring keyframes `pillarIn` / `pillarInLeft` / `pillarInRight`, tiranga bar, number pop, `pillarEnterFlash` unchanged. Coarse/phone: lateral → `pillarIn`.

Cards / spots / stats / milestones / metrics / flips / svc tiles get a soft saffron→green **border flash** (`cardEnterFlash`) when `.is-in` lands.

Tab panels use `panelPop`; metrics inside an active panel use staggered `metricPop`.

## Phone / Android wake

1. Device gate still first; motion gate after **Continue anyway**.
2. `__digitalayaWake` → `wakeReveals` → `forceNearViewport` marks near items with **stagger** (CSS transitions play — no instant snap). Pillars: only cards already in the mid band; others stay hidden until scrolled into view.
3. IO (general): thresholds `[0, 0.12, 0.22, 0.35]`, `rootMargin: 0px 0px -8% 0px`. Pillars IO: `[0.35, 0.5, 0.65]`, `rootMargin: -10% 0px -22% 0px` (no threshold 0).
4. Re-observe after unlock for anything still pending.
5. Safety: after ~2.4s, only **near-viewport** leftover hidden nodes get `.is-in` + `.reveal-force` (below-fold scroll reveals stay intact).
6. Sticky UPI stage: **opacity-only** (transform would break `position:sticky`). Pillars: full spring cascade on all widths (no sticky stack).

## Reduced motion

`prefers-reduced-motion: reduce` (and existing `reduceMotion` JS): all reveals / panel pops / card flashes / pillar cascade snap to final with no transition or keyframes.

## What was wired

Hero (eyebrow, title, lead, CTAs, stats, art), mission, journey header + track, milestones (existing stagger), pillars (cascade), UPI header + fade stage + step cards, stack header/tabs/panels, UPI compare bars + card, impact header + bento, services track + nav, toolkit flips + quiz shell, vision header + cards + timeline items, pledge header + form, sources list, footer blocks.

## Kept intact

Counters (~4–7s calm), UPI bar JS grow, sticky UPI demo, marquee fixes, device + motion gates, accessibility focus, no new stats, no per-letter splits, no heavy blur on many items.

## Risks

- Per-card pillar springs (at most a few on screen) → mitigated with no rotate on coarse, will-change while entering, 2.4s near-viewport force safety, reduced-motion snap. Early-fire fixed by not observing the tall parent.
- Sticky pillar stack removed — desktop list has gaps like phones (intentional for motion quality).
- Nested `.reveal-stagger` (hero content → hero stats) relies on recursive `markIn`.
- Horizontal `.services` stagger reveals off-screen cards when the track enters — intentional.
