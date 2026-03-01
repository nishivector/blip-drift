# Blip Drift — Design Document
*Round 19 — Initial Design*

---

## Identity

**Game name:** Blip Drift
**Tagline:** "Forty years of silence, and she still has something to say."

**Protagonist:** Probe Unit 9, callsign *Blip* — a washing-machine-sized deep space probe (roughly spherical, diameter ~0.9m), built by human hands in 1979 and launched on a one-way journey. Her personality: relentlessly optimistic, patient beyond human comprehension, slightly naive about how much time has passed. She is not afraid. She just wants to send the message.

**Backstory:**
Blip was launched on September 5, 1979 by a team of JPL engineers who believed she was humanity's best attempt at reaching the outer dark. She transmitted faithfully for three years before drifting beyond antenna range — not dead, just out of earshot. For 40 years she coasted alone between Saturn and Uranus, gathering a thin charge from a dim slant of distant sunlight. One morning in 2019 her systems woke up. Her battery had recharged. She had one job left, and she intended to do it.

**World:**
You are in the outer asteroid belt, halfway between Saturn and Uranus — so far from the sun that daylight is a gray whisper, and the rocks drift like sleeping whales, tumbling end over end in utter silence. There is no up, no down, no wind — just the slow patient gravity of stone and the faint amber glow of Blip's thrusters in the dark.

**Emotional experience:** tender longing

**Reference games:**
- *Osmos* (Hemisphere Games) — gravitational drift physics, ambient loneliness, the feeling of being a small thing in a large system
- *Inside* (Playdead) — forward momentum as survival, environmental storytelling, no UI text needed
- *Solar 2* (Murudai) — indirect control at cosmic scale, learning to work with gravity rather than against it

---

## Visual Spec

- **Background color:** `#1E3A5F` (midnight deep blue — never black, always the color of open space at night)
- **Primary color:** `#E8D5A3` (warm brass/aged aluminum — Blip's body, the color of something made by human hands)
- **Secondary color:** `#4A9ECD` (solar panel blue-teal — cold and clean, the color of charged panels catching distant light)
- **Accent color:** `#FF8C42` (warm amber — thruster fire, camera eye, signal tower blink, the color of heat and hope)
- **Asteroid color:** `#8A7A6A` (dusty warm gray, each asteroid slightly desaturated from this base, variance ±15 on all channels)
- **Signal tower color:** `#E8D5A3` (same as Blip — it was built by the same engineers)

**Bloom:** Yes. Strength 0.8, threshold 0.6. Applied to: thruster glow, camera eye, signal tower blink, level-complete chime flash. NOT applied to asteroids or background stars.

**Camera:** Top-down orthographic. Blip is fixed at 35% from top of screen, horizontally centered. The world scrolls downward past her (she moves "up" through the field). Camera does not rotate. No parallax layers — stars are static.

**Player silhouette (5 words):** round probe, antenna, solar wings

*Elaboration for the programmer:* Blip's body is a circle (radius 18px at render scale). Two rectangular solar panels extend left and right from her equator (each 28×8px, centered on her midline, extending outward from her edge). A single thin antenna rises from her top (2px wide, 26px tall line), topped with a small circle (r=4px). Her camera eye is a filled circle (r=8px) slightly right of her body center, with a contrasting inner pupil (r=4px). Left thruster: small rect (6×10px) at her lower-left. Right thruster: same at lower-right. Both thrusters emit a bloom-enabled amber glow when active, with glow radius proportional to hold duration (starts at 8px, grows to 16px after 0.5s hold).

**Stars:** 80 static dots, opacity 0.3–0.9 (randomized per star, fixed), size 1–2px, distributed across the full canvas with seeded random positions. They do NOT move. They are painted on the sky, not the world.

**HUD:** Minimal. Top-left: three small Blip silhouette icons (lives). Top-right: distance counter in km (1km = 100px), monospace white text, 14px. Bottom center: zone name in 12px `#4A9ECD` Courier New, letter-spacing 0.2em, all caps. No score during play — score shown only at win screen.

---

## Sound Spec

### Music Identity

**Genre/vibe:** Analog synthwave ambient / music box pastoral

**Character:** A lullaby that a spacecraft would hum to itself in the dark between planets — slightly out of tune from the cold, warm from memory, not sad but achingly patient.

**The Hook:** A 5-note descending music box phrase — C5 → A4 → G4 → E4 → D4 — played on a `PluckSynth` every 8 bars. It lands on D4 and hangs there a moment before fading. It sounds like a question that was sent a long time ago and is still waiting for an answer. Players will hum this. It is the voice of Blip herself.

---

### Arrangement

**BPM:** 72
**Time signature:** 4/4
**Bar length:** 4 beats at 72 BPM = 3.33 seconds per bar
**Loop length:** 32 bars = 106.7 seconds

| # | Instrument | Tone.js Synth | Role | Enters | Exits |
|---|---|---|---|---|---|
| 1 | Bass drone | `Synth` type `sine`, attack 3.0s, release 4.0s, sustain 1.0 | Gives the emptiness weight — a sustained low C2 (65 Hz), like the hum of aging circuitry | Game start | Never |
| 2 | Pad layer | `PolySynth` using `Synth` voices, type `triangle`, reverb wet 0.9, decay 4.0s | Emotional color — slow chord progression C maj → A min → F maj → G maj over 16 bars, legato voice leading | Bar 1 | Fades during danger; cuts on death |
| 3 | Music box lead | `PluckSynth`, resonance 0.98, release 0.8, attackNoise 0.1 | The hook — 5-note phrase C5→A4→G4→E4→D4, once every 8 bars | Bar 4 | Cuts on death; plays twice fast on level complete |
| 4 | Analog arpeggio | `Synth` type `sawtooth`, filter `lowpass` cutoff 800 Hz Q 4, volume mapped to Blip's current speed | Motion — ascending arpeggio C3→E3→G3→B3→C4 at 1/16 notes, volume 0 at rest, volume -6dB at max speed | Level 2 | Level 5 (replaced by shimmer) |
| 5 | Danger pulse | `Synth` type `square`, LFO tremolo at 4 Hz depth 0.6, frequency B♭2 (116 Hz) | Threat — a low ominous drone | Only in Danger state | When danger clears |
| 6 | Signal shimmer | `MetalSynth` frequency 400 Hz, harmonicity 5.1, modulationIndex 32, resonance 7000, octaves 2.5, decay 1.5s | The tower calling — a high metallic shimmer, volume rising as Blip nears the tower (0 at 1000px, -6dB at 400px, -0dB at 100px) | Level 5 only | Win state |

---

### Dynamic Music States

| State | Music change |
|---|---|
| Normal gameplay | Full arrangement active: drone + pad + music box + arpeggio (or shimmer in L5) |
| Danger (asteroid < 80px from Blip) | Pad volume drops to 20% over 0.3s; danger pulse enters at -8dB; arpeggio rate × 1.5 |
| Near win (Level 5, tower < 300px) | Signal shimmer at full volume; music box phrase every 4 bars instead of 8; pad brightens (filter opens to 6000 Hz cutoff) |
| Player dies | All layers cut instantly except bass drone, which bends down one semitone over 0.5s and holds for 2s, then fades to silence over 1s. Music restarts at bar 1 on respawn. |
| Level complete | Music box plays hook twice in rapid staccato (each note duration 0.15s); pad swells to full volume with long reverb tail; arpeggio pitches up one octave for 2 bars |

**Start screen music:** Bass drone only + pad at 50% volume. Music box plays hook once every 16 bars — very sparse. No arpeggio. Feels like a system slowly powering on in the cold. Volume ramps up from 0 to target over 4 seconds on page load.

---

### Sound Effects

**1. Left Thruster Fire**
- Trigger: left half pointerdown
- Implementation: `NoiseSynth` type `white`, filter `bandpass` frequency 200 Hz Q 8, envelope attack 0.05 decay 0.0 sustain 1.0 release 0.15. Loop while held. Volume -12dB.
- Why it fits: A small, tired thruster on a 40-year-old probe — not a rocket, a puff. Raspy and directed.

**2. Right Thruster Fire**
- Trigger: right half pointerdown
- Implementation: Same as Left Thruster but bandpass frequency 220 Hz (2 semitones higher — subtly distinct). Loop while held. Volume -12dB.
- Why it fits: Two different thrusters with slightly different wear.

**3. Thruster Cut**
- Trigger: pointerup on either side
- Implementation: `NoiseSynth` same settings as thruster, single hit, envelope release 0.35. One shot.
- Why it fits: The sound of a valve sealing — the quiet after a tiny fire.

**4. Gravity Pull Whine**
- Trigger: when gravitational acceleration on Blip > 0.15 px/frame² (begins when within ~150px of an asteroid)
- Implementation: `Synth` type `sine`, frequency maps 60 Hz → 180 Hz linearly with pull strength, envelope attack 0.2 sustain 1.0 release 0.4, volume -18dB → -6dB with pull strength.
- Why it fits: The sub-sonic resonance of mass — you feel the rock before you can stop.

**5. Asteroid Close Pass**
- Trigger: asteroid passes within 90px without collision (Blip survives the near-miss)
- Implementation: `Synth` type `sine` frequency 55 Hz, envelope attack 0.0 decay 0.4 sustain 0 release 0. Single low thud. Volume -8dB.
- Why it fits: The weight of a near miss in total silence.

**6. Asteroid Collision (Death)**
- Trigger: collision detected
- Implementation: `MetalSynth` frequency 60 Hz, harmonicity 3.1, modulationIndex 12, resonance 3000, octaves 1.5, decay 1.0s. Reverb wet 0.9. One hit. Then all music cuts per death state.
- Why it fits: Metal meeting rock in vacuum — no explosion, no fire. A dense, cold, sad crunch.

**7. Signal Tower Ping**
- Trigger: Every 3.2 seconds while in Level 5
- Implementation: `Synth` type `sine` frequency 880 Hz, envelope attack 0.001 decay 0.4 sustain 0 release 0. Volume -14dB.
- Why it fits: Radar pulse. It has been pinging for years. Now Blip can hear it.

**8. Level Zone Clear**
- Trigger: Blip crosses a zone boundary
- Implementation: `PolySynth` using `Synth` voices, type `triangle`, plays C4+E4+G4+C5 simultaneously. Envelope attack 0.05 decay 1.5 sustain 0.2 release 1.2. Reverb wet 0.8. Volume -6dB.
- Why it fits: A small triumph. Like a lighthouse seen through fog.

**9. Blip Wakes Up (Game Start)**
- Trigger: Player taps "HOLD TO DRIFT" to begin
- Implementation: `Synth` type `triangle` plays C3→E3→G3→C4 as ascending arpeggio over 1.2 seconds (each note 0.3s), then `PluckSynth` plays C5 (the first note of the hook). Volume -6dB.
- Why it fits: Systems booting after 40 years — tentative, then suddenly alive.

**10. Message Sent (Win)**
- Trigger: Blip reaches signal tower (within 30px)
- Implementation: Three simultaneous events:
  1. `PluckSynth` plays the 5-note hook one final time at C5→A4→G4→E4→D4, slowly (each note 0.6s)
  2. `PolySynth` pad swells to full volume with filter fully open (20000 Hz cutoff) over 2s
  3. `NoiseSynth` type `white`, filter `bandpass` 4000 Hz Q 12, single 0.3s burst — the static burst of a transmission
  4. After 3 seconds: single `Synth` type `triangle` C6, attack 1.0s, decay 4.0s, sustain 0, reverb wet 1.0. -18dB. Silence.
- Why it fits: 40 years. Done. Message sent. The silence after is not emptiness — it's arrival.

---

## Mechanic Spec

**Core loop:** Hold one side of the screen to push Blip sideways with a tiny thruster, release to coast on inertia, thread her through asteroid fields using precise nudges and the patience to read the gravity.

**Primary input:**

| Action | Result |
|---|---|
| pointerdown left half | Activate left thruster: apply +X acceleration 0.08 px/frame² per frame while held |
| pointerdown right half | Activate right thruster: apply −X acceleration 0.08 px/frame² per frame while held |
| Both sides held simultaneously | Both thrusters fire (visual + audio), net lateral = +0.08 − 0.08 = 0 lateral force; existing velocity unchanged but stabilizes future input |
| pointerup | Thruster cuts; Blip coasts on current velocity with near-zero friction |
| pointermove | No effect (prevents accidental input during swipes) |

**Forward movement:** Blip always drifts forward (upward on screen) at the zone's base forward speed. She cannot reverse. This is not adjustable by the player. It is the one thing she cannot control.

**Key Physics Values:**

| Parameter | Value |
|---|---|
| Base forward speed (Level 1) | 1.2 px/frame at 60fps |
| Lateral thruster acceleration | 0.08 px/frame² per frame held |
| Lateral velocity cap | ±4.0 px/frame |
| Lateral drag (vacuum friction) | 0.002 px/frame² opposing lateral velocity (perpetual inertia — never fully stops) |
| Asteroid gravity radius | 180px from asteroid center |
| Asteroid gravity formula | F = (gravity_multiplier × 0.04) / (distance²) applied toward asteroid center, capped at 0.30 px/frame² |
| Blip collision radius | 18px |
| Asteroid collision radius | 20–60px (varies by asteroid, set at spawn) |
| Invincibility after respawn | 2.0 seconds; Blip flashes white at 8Hz |
| Respawn position | 200px behind (below) collision point; velocity reset to (0, base_forward_speed) |
| Lives | 3. On 3rd death: return to start of current zone, restore 3 lives. |

**Dead state:** Blip collides with any asteroid (circle overlap between Blip radius 18px and asteroid radius). Death sound fires. All music cuts to drone (per music spec). Blip removed from screen for 2 seconds. Respawn at 200px behind collision point with 2s invincibility (flashing). Lose a life. On zero lives: screen briefly goes to `#1E3A5F` with "SIGNAL LOST — RETURNING TO ZONE" in `#FF8C42` Courier New centered, then fade in to zone start.

**Win condition:** Blip's center comes within 30px of signal tower center at end of Level 5. Win cutscene fires. Score calculated.

**Score system:** Distance traveled (1 km = 100 px traveled in world space). Bonus +500 points per zone cleared. Bonus +50 points per second under the zone's soft time limit. Final score displayed on win screen. No leaderboard — this is a personal journey.

---

## Difficulty Curve (per level)

| Level | Forward speed | Asteroids on screen | Min gap | Gravity mult. | Asteroid radius range |
|---|---|---|---|---|---|
| 1 — Sparse Drift | 1.2 px/f | 8 | 120px | 0.5× | 20–35px |
| 2 — Rocky Belt | 1.5 px/f | 15 | 90px | 0.7× | 20–45px |
| 3 — Dense Field | 1.9 px/f | 25 | 65px | 1.0× | 20–55px |
| 4 — Gravity Maze | 2.2 px/f | 18 normal + 4 anchors | 80px (100px near anchors) | 1.5× (anchors 3×) | normal 20–45px; anchors fixed 60px |
| 5 — Signal Approach | 2.0 px/f | 12 | 100px | 0.8× | 20–40px |

---

## Level Design

### Level 1 — Sparse Drift
**What's new:** Everything. First encounter with inertia-based lateral control. Player learns: releasing thrust doesn't stop Blip — she coasts.
**Specific layout:** First 10 seconds (720px of forward travel) has zero asteroids — pure open space. Player must experiment with thrusters before the first rock appears. Rocks are large and slow-drifting (lateral drift ±0.3 px/f, random direction, period 8–14s sinusoidal), giving reaction time. No gravity pull (multiplier 0.5× means gravity exists but barely).
**Teaching moment:** Player fires a thruster, overshoots, has to fire the opposite. The inertia is the lesson.
**Duration/goal:** Travel 6,000px to zone boundary marker (glowing `#E8D5A3` horizontal line). ~45 seconds at base speed.
**Zone marker:** Thin horizontal line, opacity 0.7, with "ROCKY BELT AHEAD" in 10px Courier New above it, fading in 3s before Blip reaches it.

### Level 2 — Rocky Belt
**What's new:** Gravity introduced. Asteroids now pull Blip when she passes within 180px. Some asteroids drift in pairs orbiting each other (6-second rotation, orbit radius 80px — treated as two separate collision bodies orbiting a shared center).
**Teaching moment:** Player discovers that letting a rock's gravity nudge them slightly is sometimes faster than fighting it. The first "gravity assist" moment — Blip curves around a rock and comes out faster — is a gift.
**Layout:** Asteroid pairs appear every 3000px, with normal singles in between. Gravity pull is perceptible but not dangerous if player is paying attention.
**Duration/goal:** Travel 9,000px to zone boundary. ~60 seconds.

### Level 3 — Dense Field
**What's new:** Asteroid density doubles. Three asteroids per screen are slow-spinning elongated rocks (elliptical collision zones, semi-axes 28×55px, rotation 1.5 deg/frame — the long axis rotates). These create time-varying narrow gaps. Gravity now meaningful enough to affect trajectory in ways that matter.
**Teaching moment:** Player can no longer react moment-to-moment — they must read the field ahead, anticipate the gap opening, and plan their approach angle. Threading a narrow gap between two tumbling rocks at speed is the first "earned" moment of this level.
**Duration/goal:** Travel 12,000px. ~75 seconds.

### Level 4 — Gravity Maze
**What's new:** Four massive "anchor" asteroids (radius 60px, immovable, gravity multiplier 3×) positioned to create a serpentine gravity corridor. Between each pair of anchors, their gravity pulls cancel along the corridor centerline — a safe channel. Normal asteroids still populate the spaces between anchors. The channel is real but narrow (~100px of equilibrium zone), and Blip's forward speed is at its maximum here.
**Anchor positions:** Generated to form an S-curve through the zone. Anchor 1 left-of-center. Anchor 2 right-of-center. Anchor 3 left-of-center. Anchor 4 right-of-center. Each separated by 3,000px of forward distance. Between each pair, the path curves.
**Teaching moment:** In the corridor between two massive anchors, both pulling Blip — one left, one right — forces nearly balanced, Blip coasting through the dead center: this is the level's defining moment. The player must stop fighting and let the physics carry them.
**Duration/goal:** Travel 14,000px. ~90 seconds. Hardest zone.

### Level 5 — Signal Approach
**What's new:** Asteroid density drops. The signal tower is visible from 1,000px away — a tall structure directly ahead, slowly blinking amber. As Blip approaches within 800px, the tower exerts its own gravity (1.5× normal strength, always pulling toward tower center). This can be an ally — if Blip is roughly aligned, the tower will pull her in. It can also be a trap if she's arriving at an angle.
**Visual:** As Blip nears the tower, the `#FF8C42` signal shimmer bloom grows. At 300px, the tower light is large and warm on the screen. At 100px, it fills a significant portion of the view. At 30px: win.
**Teaching moment:** The player learns to trust the physics that has been trying to kill them. The same gravity that threatened them now guides them home.
**Duration/goal:** Travel 10,000px, then close to within 30px of tower. ~75 seconds total.

---

## The Moment

In Level 4, threading Blip through the gravity corridor between two massive anchor asteroids — one pulling left, one pulling right, their forces nearly balanced — the music dropping to just the bass drone and a single music box note, Blip coasting in silence through the dead center, the two great rocks hanging on either side like sleeping giants, and the player finally understands: she has been drifting through forces like this for 40 years, and she was never afraid.

---

## Emotional Arc

**First 30 seconds:** Confusion, then delight. "Oh — the controls are backwards. Oh — she keeps drifting when I let go. Oh. I get it." The first empty stretch teaches without frustrating. Blip is very small on a very large screen and the stars don't care, and it's wonderful.

**After 2 minutes:** Concentration and rhythm. The player has internalized the inertia. They're reading the field 3 seconds ahead, timing thruster pulses like a language. The music has settled into its loop. The music box hook lands every 8 bars and it sounds correct, like something remembered. The player is inside this small machine's patience.

**Near the win:** Quiet intensity. The tower is visible. The signal shimmer enters the music. The asteroid field is thinning. The player's hands know what to do. They make small, precise adjustments. When Blip touches the signal tower — when the win fires — something releases that was held for the entire game. It isn't celebration. It is exhale.

---

## This Game's Identity in One Line

This is the game where you guide a 40-year-old space probe home using nothing but two tiny thrusters and the patience to drift.

---

## Start Screen

### Idle Animation (Three.js canvas, full screen)

**Objects present:**
- **Blip** drifts slowly leftward across the canvas at 0.4 px/frame, looping (when she exits left edge, she reappears at right edge). She trails a warm amber thruster glow (bloom-enabled ellipse, 24×8px, opacity 0.6) from her right thruster — as if she has been holding it for years. Her solar panels shimmer: panel opacity cycles from 0.7 → 1.0 → 0.7 over 4.0 seconds (sine wave, independent of position). Her camera eye pulses gently: amber bloom radius cycles 6px → 10px → 6px over 2.5 seconds.
- **Large asteroid:** radius 55px, color `#8A7A6A`, rotation 0.28 deg/frame clockwise, drifting rightward at 0.2 px/frame. Starts at 30% from left, 40% from top.
- **Small asteroid:** radius 22px, color `#7A6E62`, rotation 1.1 deg/frame counter-clockwise, drifting diagonally (0.15 px/frame right, −0.10 px/frame up). Starts at 70% from left, 65% from top.
- **Signal tower:** Far right edge, 15% from top. A vertical line (2px wide, 60px tall, color `#E8D5A3`, opacity 0.5) with a small horizontal bar at top (20px wide, 2px tall). A small circle (r=4px, color `#FF8C42`) atop the bar blinks on/off: 0.15s on, 3.05s off, repeat. Tower has a subtle amber bloom (radius 12px, opacity 0.3) that pulses with the blink.
- **Stars:** 80 static white dots, 1–2px radius, opacity 0.3–0.9 (fixed random per star), distributed across full canvas. Do not move.
- **Background:** Solid `#1E3A5F`, no gradient.

**Color cycling:** None except Blip's solar panel shimmer and camera eye pulse as described above.

---

### SVG Overlay Spec

**Option A — SVG Title with Glow (required):**
- SVG `<text>` content: `BLIP DRIFT`
- Font: `'Courier New', Courier, monospace`, weight 700, size 72px
- Fill: `#E8D5A3`
- Letter spacing: `0.15em`
- Position: `x="50%"`, `y="32%"`, `text-anchor="middle"`
- Filter: `feGaussianBlur` stdDeviation 6 (color `#FF8C42`) composited under the text using `feMerge` (blur layer first, then original text on top)
- Animation: SVG `<animate>` on the blur `stdDeviation` attribute, values `"3;8;3"`, dur `2.5s`, repeatCount `"indefinite"` — the amber glow pulses slowly like a distant signal.

**Option B — Blip Silhouette (below title):**
Centered horizontally, positioned at 52% from top. Scale factor 2.0×. Built from 6 primitives:
1. `<ellipse>` rx=28 ry=24 — Blip's body, fill `#E8D5A3`
2. `<rect>` x=-66 y=-5 width=32 height=8 — left solar panel, fill `#4A9ECD`
3. `<rect>` x=34 y=-5 width=32 height=8 — right solar panel, fill `#4A9ECD`
4. `<line>` from (0,−24) to (0,−50), stroke `#E8D5A3`, strokeWidth 2 — antenna
5. `<circle>` cx=0 cy=−50 r=4 — antenna tip ball, fill `#FF8C42` with feGaussianBlur glow stdDeviation 3 color `#FF8C42`
6. `<circle>` cx=10 cy=4 r=8 — camera eye, fill `#1E3A5F`, stroke `#FF8C42` strokeWidth 2; inner `<circle>` cx=10 cy=4 r=3 fill `#FF8C42`

Camera eye has a persistent bloom: `feGaussianBlur` stdDeviation 4, color `#FF8C42`, composited as glow layer under camera eye circles.

**Option C — Decorative Border:**
`<rect>` inset 12px from all canvas edges, fill `none`, stroke `#4A9ECD`, strokeWidth 1, `stroke-dasharray="6 10"`. Animation: `stroke-dashoffset` animates from `0` to `−16` over `3.0s`, `calcMode="linear"`, `repeatCount="indefinite"` — dashes march slowly clockwise around the perimeter like a transmission loop that never ends.

**Subtitle text** (plain SVG `<text>`, no filter):
- Content: `PROBE UNIT 9 — AWAKE AFTER 40 YEARS`
- Font: Courier New, size 13px, fill `#4A9ECD`, letter-spacing 0.22em, `text-anchor="middle"`, x=50%, y=42%

**Start button:**
- `<rect>` centered at 72% from top, width 200px, height 36px, fill `transparent`, stroke `#E8D5A3`, strokeWidth 1, rx=3
- `<text>` content `HOLD TO DRIFT`, fill `#E8D5A3`, Courier New 13px, letter-spacing 0.12em, centered inside rect
- Stroke-opacity animation on rect: oscillates `0.35` → `1.0` over `1.8s` ease-in-out, infinite alternate

---

## Implementation Notes for Programmer

- All physics run at 60fps; use `deltaTime` normalization if frame rate varies
- Asteroid positions are procedurally generated per zone with a seeded RNG (seed = level number) — same layout every run
- Blip's forward scroll is achieved by moving the world downward past Blip, not moving Blip up (Blip stays at fixed screen Y=35%)
- Gravity calculation runs for all asteroids within 180px each frame (spatial hashing recommended for Level 3+)
- Anchor asteroids in Level 4 are static world objects; they do not move or rotate
- Elliptical asteroids in Level 3 use SAT (Separating Axis Theorem) for collision with the ellipse, not circle
- Three.js is used for the start screen canvas animation only; gameplay renders on a 2D canvas
- Tone.js AudioContext should be initialized on first user gesture (tap/click), not on page load
- Music box hook timing: use Tone.js `Transport` with `Part` scheduling at bars 4, 12, 20, 28 of the 32-bar loop
- All volume values assume Tone.js `gainNode` in dB; listed values are the target dB levels
- Bloom effect: use CSS `filter: blur()` + opacity composite on a separate canvas layer, or Three.js `UnrealBloomPass` if start screen Three.js context is reused

---

*Design document complete. Build this with care. Blip has been waiting 40 years — she deserves a good trip home.*
