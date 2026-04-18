# SHOOT A DM — Handoff Document
**Project:** Edgar Cielos interactive contact section  
**Status:** Feature complete. Ready for deployment. One item to verify in browser: crosshair magnetism (math confirmed correct, needs live mouse test).

---

## What Was Built

A self-contained browser mini-game (`index.html` + `assets/`) for Edgar Cielos' Readymag portfolio. Users hover over the game to open steel gate doors, then shoot bat-winged social media icons flying around a gothic 8-bit background. Clicking an icon opens that social link. Built in vanilla HTML/CSS/JS — no frameworks, no build tools.

---

## File Structure

```
edgar_cielos_shootdm/
├── index.html              ← entire game (edit this)
├── SHOOTDM_SPEC.md         ← original spec
├── HANDOFF.md              ← this file
├── .claude/launch.json     ← python http.server config (port 3456)
├── watersocks/             ← original asset source, IGNORE (already copied to assets/)
└── assets/
    ├── border total.gif    ← 1116×240px — topmost decorative frame overlay
    ├── background.gif      ← 800×336px natural — gothic graveyard scene
    ├── white cross.png     ← default crosshair
    ├── yellow cross.png    ← proximity/shot crosshair
    ├── border.png          ← unused
    ├── edgar.gif           ← unused
    ├── ekg.gif             ← unused
    ├── frames.png          ← unused
    ├── socials/
    │   ├── email.gif       ← 400×270px natural (bat-winged email icon)
    │   ├── insta.gif       ← 400×270px natural
    │   ├── linkin.gif      ← 400×270px natural
    │   └── tiktok.gif      ← 400×270px natural
    └── Wall Opening/
        ├── L Gate.png      ← 432×287px natural (left steel door, "SHOOT A")
        └── R Gate.png      ← 696×287px natural (right steel door, "DM")
```

---

## Key Measurements (pixel-analyzed from actual asset files — do not guess)

| Value | Measurement |
|---|---|
| `border total.gif` natural size | **1116 × 240px** (spec said 1182×255 — that was wrong) |
| Game viewport width | **817px** — right panel starts at x=817 |
| Right panel width | **299px** (x=817 to x=1116) |
| Border frame top/bottom | **13px** each — visible game interior is y=13 to y=227 (214px) |
| L Gate natural size | **432 × 287px** — rendered **361 × 240px** |
| R Gate natural size | **696 × 287px** — rendered **534 × 240px** (width capped at game boundary) |
| L Gate solid content ends | **~302px** screen x (right ~59px of image are transparent) |
| R Gate solid content starts | **~17px** from its left edge → screen x ≈ 300px (seamless join) |
| Social icon natural size | **400 × 270px** — rendered `height:110px, width:auto` (~163px wide) |

---

## HTML Layer Stack (z-index)

```
z=1  #bg               — background.gif, 817×240, object-fit:cover
z=3  #targets-layer    — container for 4 flying .target imgs (pointer-events:none)
z=4  #game-clip        — overflow:hidden container 817×240 (clips doors to game area)
       #door-left      — L Gate.png, left:0, width:361px, height:240px
       #door-right     — R Gate.png, left:283px, width:534px, height:240px
z=6  #crosshair        — 40×40 div with crosshair PNG as background-image
z=7  #border-overlay   — border total.gif, 1116×240, pointer-events:none (critical)
```

(`#miss-canvas` at z=2 is hidden/display:none — kept in DOM per original spec.)

---

## Critical Architecture Details

### Door seam
Both gate PNGs have transparent inner edges by design. L Gate solid ends at ~302px, R Gate solid starts at ~300px. The seam is invisible.

- **`#door-right` must stay at `left: 283px`** — this is the exact position where R Gate's solid content aligns with L Gate's. Change it and the door gap reappears.
- **`#game-clip` must stay `overflow:hidden` at 817px** — R Gate is 534px wide starting at x=283 = reaches 817px exactly. Without this clip, R Gate would bleed into Edgar's panel.
- **`void el.offsetHeight` in `openDoors()`** — forces browser reflow after removing the shake CSS animation so the slide transition fires correctly. Remove it and doors snap instead of slide.

### Door open animation
`translateX(-100%)` on `#door-left` = −361px (exits left).  
`translateX(100%)` on `#door-right` = +534px (moves from x=283 to x=817, clips at game boundary).

---

## JavaScript Constants

```js
GAME_VIEWPORT_WIDTH = 817
GAME_HEIGHT         = 240
TARGET_H            = 110        // rendered icon height px
TARGET_W            = 163        // rendered icon width px (400/270 × 110)
MAGNET_RADIUS       = 160        // px from icon center where magnetism starts
MAGNET_STRENGTH     = 0.70       // pull factor — 28px shift at 70px distance
DOOR_HOVER_GRACE_MS = 800        // ms delay before doors close on mouseleave
DOOR_SHAKE_DURATION_MS = 1200   // must match CSS animation duration
```

### Social Links (PLACEHOLDERS — update before ship)
```js
'target-email':    'mailto:placeholder@email.com'   // ← needs Edgar's real email
'target-insta':    'https://www.instagram.com/ed.herndez/'  // ← confirmed
'target-linkedin': 'https://www.linkedin.com/'      // ← needs Edgar's real LinkedIn URL
'target-tiktok':   'https://www.tiktok.com/'        // ← needs Edgar's real TikTok URL
```

---

## What Works ✅

- Doors shake on hover → slide open → slide closed 800ms after mouse leaves
- 4 bat-winged icons fly on independent sine-wave paths, bounce off walls
- Crosshair turns yellow when within 80px of any icon center (proximity)
- Crosshair flashes yellow + scales 1.4× on any click (shot feedback)
- Click on icon → opens social link in new tab → icon flashes white → respawns after 1s
- Miss click → background + icons instantly desaturate to greyscale → ease back to color
- Crosshair magnetism: 28px pull at 70px from icon center (verified by math, needs live browser test to confirm feel)
- Mobile: IntersectionObserver for door open/close, touchmove/touchend for aim and shoot
- Border overlay frames everything, pointer-events:none so clicks pass through

---

## Remaining Before Ship

1. **Live-test magnetism** — hover over game at `localhost:3456`, open doors, move mouse near a flying icon. Should feel a gentle pull. If not visible enough, increase `MAGNET_STRENGTH` toward `0.85` in the constants at top of the `<script>` block.

2. **Update social links** — swap the 3 placeholder URLs above with Edgar's real links.

3. **Deploy to GitHub Pages:**
   - Create new PUBLIC repo named `edgar-shootdm` on GitHub
   - Push this entire folder (`index.html` + `assets/` — nothing else needed)
   - Repo Settings → Pages → Source: Deploy from branch → Branch: `main` → folder: `/ (root)` → Save
   - Wait ~60 seconds → site live at `https://yourusername.github.io/edgar-shootdm/`
   - Test that URL in browser before giving to Edgar

4. **Embed in Readymag:**
   - Add an "Embed" widget in Readymag editor
   - Paste the GitHub Pages URL
   - Set width: **1116**, height: **240**, scrolling: no, frameborder: 0
   - ⚠️ Do NOT use 1182×255 — the actual GIF is 1116×240

---

## Local Dev Server

```bash
cd "C:\Users\qlx32\Desktop\edgar_cielos_shootdm"
python -m http.server 3456
# open http://localhost:3456
```
(The `.claude/launch.json` automates this if using Claude Code.)

---

## Do NOT Change Without Good Reason

| Thing | Why |
|---|---|
| `left: 283px` on `#door-right` | Changes this reopens the door center gap |
| `width: 534px` on `#door-right` | 283+534=817 exactly — change clips yellow text or bleeds into right panel |
| `void el.offsetHeight` in `openDoors()` | Removing causes doors to snap instead of slide |
| `overflow:hidden` on `#game-clip` | Removing causes R Gate to bleed into Edgar's panel when open |
| `pointer-events: none` on `#border-overlay` | Removing blocks all game clicks |
| `width: 1116px` everywhere | Do not revert to 1182px — the border GIF is 1116px wide |
