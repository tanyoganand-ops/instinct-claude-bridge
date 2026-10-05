---
title: "HERO SPEC PACK - Portfolio 2026"
subtitle: "Golden valley > drone zoom > surface break > gold underwater"
date: "5 October 2026"
---

# 0. How to read this

Build brief for the portfolio redesign hero. Written so the builder can code from it without asking questions. Sources are the 11 research reports only (no new web research). Where a report said "not verified", this doc says so too.

**Direction lock (user's own words, from the user's messages as relayed in the assignment):**

- "I'll stick to like the sun, so yellow/sun tinge, light gold premium feel with hints of pink"
- "interactive hero like this [igloo.inc] with interactive text like the unseen one would be nice"
- "starts with a really clear hero with proper mountain texture feel, in the middle of 2 arches ... then we zoom in kinda like a drone shot, then we switch to taking the shot underwater just like unseen with the same muted audio feel, underwater but more gold tinged with light reflecting through refracting"
- "Shouldn't be much fog in the hero, should just be cinematic shot down valley with 2 peaks on left and right of screen then zoom down the middle"
- Heavier-weight type, no thin fonts. Unseen/Igloo richness, no minimalism. Slow, smooth, padded motion. Grain + glow = premium. Must run on mid-range phones.

**Supersession:** "2 arches" is replaced by "2 peaks left and right" per the later message. Heavy fog is replaced by light depth haze only. Pink-arches footage is a reserve only.

**Licence gate for the build path:** MIT, CC0, BSD (incl. public domain) only. Everything else is marked CONDITIONAL, CHECK, UNCONFIRMED, REF or NO in section 6 and is not to be installed or copied.

## Palette tokens (locked)

| Token | Hex | Use |
|---|---|---|
| `--gold` | `#E4C98A` | Dominant. Sunlit slopes, valley floor, gold light, rules, button fill |
| `--sun-cream` | `#F7EFDD` | Page ground, sun disc and horizon highlight, small speculars, upper haze |
| `--blush` | `#E9B8B4` | Hint only. Narrow sky band near the sun, faint water glint, one small UI accent per viewport |
| `--charcoal` | `#262119` | Type, shaded slopes (keep warm fill, shadows never go black) |

Blush never tints fog or bloom as a whole. Chromatic fringe is not the source of pink. Gold is not for long text on cream. Test final text and button contrast pairs before release (not tested in research).

# 1. Concept summary

One continuous camera journey on a single scroll, with a real HTML headline on top throughout.

| Beat | Scroll `p` (design values, tune by eye) | What the viewer sees |
|---|---|---|
| 1. Clear valley | 0.00 - 0.22 | Cinematic wide shot down a rocky valley. Two unequal peaks frame left and right, open corridor in the middle, low gold sun in the notch, light depth haze only in the far distance. Textured rock reads clearly. Slow idle drift + pointer parallax. |
| 2. Drone zoom | 0.22 - 0.55 | Camera glides down the valley centreline, lowering, eased in and out, look-ahead target just below the horizon. Peaks slide past the frame edges. |
| 3. Surface break | 0.55 - 0.68 | Valley floor ends in a still lake. Camera descends to the waterline and crosses it. Short refraction/ripple mask, audio crossfade, tint and absorption shift. No splash sound. |
| 4. Underwater | 0.68 - 1.00 | Gold-tinged underwater: dark olive-teal body, amber caustics and shafts from the surface, drifting particles, Snell's window glimpse. Muffled low-passed audio. Fades to cream into the portfolio. |

Interactive headline (accessible HTML, variable-font weight response near the cursor, tap on touch) sits over the whole sequence. Sound toggle is off by default.

The hero is original geometry, lighting and materials. Igloo and Unseen are references for depth, texture, pointer response and pacing, not for copying composition, models or audio.

# 2. Scene-by-scene build spec

## 2.0 Renderer and stack (decision)

- Three.js `WebGLRenderer` (WebGL2) + React Three Fiber + Drei. All MIT. `Sky` (WebGL) and the GLSL water and caustics references are WebGL2. Do not mix in the WebGPU/TSL path (report 580: one renderer architecture). Pin the Three version in the lockfile.
- Colour: palette hexes are sRGB inputs (Three converts to linear). A/B `AgXToneMapping` vs `ACESFilmicToneMapping` on a real frame, pick the one that keeps gold saturated, tune `toneMappingExposure` by eye.
- Post stack uses Three's own `EffectComposer` (MIT), not `pmndrs/postprocessing` or `@react-three/postprocessing` (Zlib, CONDITIONAL). Passes: `RenderPass`, `UnrealBloomPass` (high threshold, low strength), one custom `ShaderPass` (original code) doing grain, radial chromatic fringe and vignette together. One custom full-screen pass keeps mobile cheap.

## 2.1 Scene 1: clear golden valley

**Composition**

- Camera starts high and wide, looking down the valley axis. FOV 38-45.
- Two summit masses, unequal in height and slope (so they do not read as twin volcanoes), one in the left third, one in the right third. Central flight corridor stays open and lower. Sun sits low in the notch on the valley centreline, so the dive aims into the brightest area.
- Foreground: nearby rock with displacement detail. Midground: layered ridges. Far: paler ridges (value separation, not fog).

**Geometry (technique chosen):** original procedural heightmap, baked offline, loaded at runtime. No erosion at runtime.

1. Silhouette first: broad U-shaped trough along Z, gently undulating floor, two Gaussian/elliptical summit lobes on the left and right walls, taper toward the far horizon (report 586).
2. Ridged multifractal detail `r = (offset - abs(noise))^2` with octave feedback, masked to the valley walls. Detail amplitude lower than the main form. No high frequency on the valley floor.
3. Low-amplitude, low-frequency domain warp, two decorrelated fields, none on the flight centreline or floor.
4. Offline thermal talus relaxation, then a low-iteration hydraulic pass for gullies. Keep drainage off the centreline.
5. Bake a 16-bit heightmap and normal map (1025 or 2049 square). Grid `BufferGeometry`, Y from the heightmap, restrained vertical exaggeration.
6. Generator: write own noise (original code) or use `THREE.Terrain` (MIT, https://github.com/IceCreamYou/THREE.Terrain) as generator and heightmap exporter. Three's `TerrainGenerator` is TSL/WebGPU and is not in the baseline.
7. LOD: the camera path is fixed, so prebuild a corridor: high-res mesh along the centreline and near peak faces, coarse far peaks, cull behind the camera. `lod-terrain` (MIT, https://github.com/felixpalmer/lod-terrain) and `geo-clipmap` (MIT, https://github.com/tschie/geo-clipmap) are algorithm references only (old demos, not dependencies). Camera speed is authored, independent of any streaming.
8. Real DEM is optional, shape reference only: NASADEM/SRTM (NASA CC0 guidance, cite NASA) or USGS 3DEP (US public domain, US coverage only). Swiss swissALTI3D (Lauterbrunnen) is NOT in the build path (attribution-required OGD).

**Materials**

- CC0 PBR sets: Poly Haven `rocky_terrain` https://polyhaven.com/a/rocky_terrain , `cliff_side` https://polyhaven.com/a/cliff_side , `sparse_grass` https://polyhaven.com/a/sparse_grass ; ambientCG `Rocks025` https://ambientcg.com/view?id=Rocks025 . Shortlist: Cliff Side + Sparse Grass + Rocks 025. Inspect the actual maps before locking. They give surface detail, not mountain shape.
- Splat by slope and height with smoothstep thresholds and world-space noise to break bands: meadow low and flat, grass mid, rock on steep slopes and high up.
- Triplanar on steep faces (weights `pow(abs(n), sharpness)`, blend normals properly), via `onBeforeCompile` (Three has no stock triplanar PBR). Expensive, so low tier uses a baked albedo + vertex colour instead.
- Altitude palette in shader: charcoal/earth in shade, gold on valley floor and sunward slopes, cream on exposed upper faces. Smooth transitions only.
- Thin warm peak rim `pow(1 - max(dot(n, v), 0), p)` on the two skyline ridges only. Subtle.

**Lighting**

- Sky: Three `Sky` addon (MIT, https://threejs.org/docs/pages/Sky.html). Sun low, in the notch, on the valley axis. Tune turbidity, Rayleigh, Mie by eye. Hide the sun disc when generating an environment map.
- One warm `DirectionalLight` as key. No dynamic shadow maps. Fake long shadows with a few broad, soft, fixed swaths baked into vertex colours or a low-res mask, opposite the sun.
- Fill/IBL: `PMREMGenerator.fromScene(sky)`. A CC0 HDRI is optional and only downsampled offline (Gruntowa source is 1.42 GB, never ship it, never show it as the visible sky).
- Sun: small hard disc, bloom threshold high so only the disc and brightest water glint bloom. No lens flare, or one very faint ghost.
- Shafts in beat 1: no raymarched godrays. One or two broad soft translucent wedges or a screen gradient masked by the peak silhouettes through the notch, strongest near the sun, fading down the valley. Animate opacity only.
- Blush: narrow sky band where warm sun meets cool skylight, faint glint on the lake, very thin rim on sun-facing peak edges. Nowhere else.

**Atmosphere (fake height fog, not volumetrics)**

- Height-aware, camera-distance blend injected with `onBeforeCompile`: `fog = (1 - exp(-dist * k)) * smoothstep(hTop, hBottom, worldY)`. Low `k`. Haze mostly in the far centreline and valley bottom. Colour: pale warm neutral between gold and cream, close to the far valley's lit average. Not grey, not pink.
- Animate a seamless noise texture on world XZ to move the fog boundary (technique from The Sleepers, https://tympanus.net/codrops/2026/07/10/the-sleepers-creating-an-atmospheric-webgl-experience-with-lightweight-techniques/ ; rewrite it, the article code has undeclared variables and is not drop-in).
- Horizon sphere with the same rule so haze does not stop at the geometry edge.
- Stock fallback: `FogExp2` with very low density and cream colour.
- Foreground stays clear. Never stack strong shafts and heavy fog.

## 2.2 Scene 2: drone zoom

- Author 6-9 waypoints. `CatmullRomCurve3` (centripetal, arc-length parameterised) for camera position and a second curve for the look-at target (short look-ahead) so yaw never snaps.
- Scroll progress `p` maps to curve `t`. This is the same idea as Drei `MotionPathControls` offset 0-1 with `loop={false}`; either implementation works. One `p` drives camera, fog uniform, audio and headline state.
- Ease at beats (slow in at 0.22, slow out into the waterline). Never start a tween per wheel event.
- Height profile: high and wide at the start, descending, low over the lake by `p = 0.55`.
- Drone feel: FOV push +3 to +6 degrees, roll under 1.5 degrees on curve bends.
- LOD refines ahead of the camera along the corridor.
- First view is already useful: headline and nav readable before any scroll.

## 2.3 Scene 3: surface break

- Water plane at Y = 0 in a basin at the end of the valley. Flat plane with two scrolling normal maps (pattern from Nugget8 Three.js-Ocean-Scene, MIT). High tier adds Drei `MeshReflectorMaterial` (MIT) at low reflection resolution; mid and low use Fresnel + sky reflection only.
- Gold sun streak on the water with a small blush part near the sun.
- The camera physically passes through the plane. Over about 0.05 of `p`, animate a post uniform `uUnderwater` 0 to 1: absorption tint, fog colour swap, distortion.
- Waterline mask: brief expanding ripple/distortion at the crossing only, adapted from a GL Transitions water-drop shader (MIT, https://github.com/gl-transitions/gl-transitions ; check the file header for the author notice). Alternative: Unseen-style screen-space trick (render to texture, displaced plane in front of the camera, same noise function as the surface so they do not diverge).
- Audio crossfade fires at this `p` (3.4).
- Test the crossing on a phone. Overdraw is the risk. Never combine the mask with raymarching or the reflector on mobile.

## 2.4 Scene 4: gold underwater

**Look**

- Water body: dark neutral olive-teal with plausible blue/green attenuation. Gold lives in the light, not the water: amber shafts, caustics, surface glints, a restrained amber scatter colour by depth. No flat yellow fog (report 582).
- Grade: (1) per-channel Beer-Lambert `T = exp(-sigma * thickness)`; (2) blend to a warm low-saturation scatter colour with depth (near clearer, far darker olive-teal); (3) tone-map once after compositing, bloom limited so gold reads as light, not neon.
- Blush: occasional subtle shimmer on surface glints only.
- Snell's window when looking up: `refract(view, normal, 1.333)` with a Fresnel/critical-angle transition (about 48.6 degree half-angle), mixing to water colour outside. A stylised distorted-sky texture plus moving meniscus is acceptable.

**Technique (build cheap first)**

| Tier | Caustics | Shafts | Surface underside |
|---|---|---|---|
| Low / mid (default) | Two small seamless caustic textures, different scale, speed, rotation, modulated on seabed and rocks. Texture samples only. | 3-5 tapered translucent cones/planes with noise-masked soft gradients, static geometry, animate UV and opacity, fade with depth. | Scrolling-normal plane, Snell's window as a fullscreen shader term. |
| High | Projected caustics using the `martinRenou/threejs-caustics` algorithm (BSD-3, https://github.com/martinRenou/threejs-caustics) on the seabed only, if frame rate holds. | Same fake shafts. Optional single quarter-res raymarched pass A/B on desktop only. | Same. |

- Seabed: original geometry from the same heightmap pipeline, CC0 material, baked lighting where possible.
- Particles: instanced marine snow, 500 (low) to 2000 (high), slow drift, additive, tiny.
- Code to read and adapt (keep MIT notices): UnderwaterAI https://github.com/Underwater-AI/underwater-ai.github.io (MIT; procedural scroll-driven surface-to-seafloor scene, three 0.160; its README says GSAP has a separate licence, do not copy GSAP use). WaterThreeJS https://github.com/achrefelouafi/WaterThreeJS (MIT per inspected LICENSE in report 580; files `Ocean.js`, `Post.js`, `Floor.js`, `shaders/common.js`; WebGL2/GLSL; runtime and fps NOT verified, no hosted demo; strip clouds, foam, buoyancy).
- Ending: past `p = 0.92` the grade shifts to `--sun-cream` and fades into the page ground (this wash is the canvas-to-DOM handoff).
- Avoid: full-res raymarched volumetrics, fluid-sim grids, multi-capture refraction, parallax water, WebGPU-only passes.

# 3. Interaction spec

## 3.1 Pointer-reactive layers (desktop)

- Normalised pointer in -1..1, damped with `maath` (MIT) in `useFrame`.
- Small clamped offset on camera position and look-at (about 0.6 degrees yaw, 0.4 pitch) so the two-peak framing never breaks.
- Depth gains: near rock moves most, mid ridges less, far peaks and sky least (separate group offsets). Sun disc and shafts barely move.
- Glow response: bloom strength and shaft opacity rise slightly near the sun. Underwater, glints respond to the pointer through a ripple normal offset.
- Ambient motion: Drei `Float` (MIT) for particles only (it is bobbing, not follow).
- No physics (Rapier not needed). `PresentationControls` is drag rotation and is not used.

## 3.2 Scroll-driven camera

- One source of truth: `p` in 0..1 from native page scroll over a tall section (about 600vh desktop, 500vh mobile). The hero canvas is `position: sticky; top: 0; height: 100svh`.
- Desktop smoothing: Lenis (MIT, https://github.com/darkroomengineering/lenis), defaults, `syncTouch` false.
- Damp `p` (`maath`, lambda about 4-6) so motion is padded.
- Camera = `curvePos.getPointAt(pEased)`, target = `curveTarget.getPointAt(min(pEased + lookAhead, 1))`.
- Alternative: Drei `ScrollControls` + `MotionPathControls` (https://drei.docs.pmnd.rs/controls/motion-path-controls). ScrollControls makes a virtual scroll container, so prefer native scroll on mobile.
- Reduced motion: skip the flight, show a still of beat 1, opacity crossfade to the portfolio, no parallax, no looping caustics.
- Pause rendering offscreen (`IntersectionObserver`) and on hidden tab (`visibilitychange`, reset delta on resume).

## 3.3 Interactive headline

- Real `<h1>` in the DOM always. Per-letter spans are `aria-hidden="true"` and the full text is exposed once via `aria-label` on the `<h1>` (or a visually hidden duplicate). Never replace it with canvas text.
- Primary effect: cursor-proximity variable font response with MagnetType (npm `@liiift-studio/magnettype`, MIT per npm; its demo site imports `@overpunch/magnettype`, so verify the package name before install; https://github.com/Liiift-Studio/MagnetType). Letters thicken (`wght`, optionally `wdth`) near the pointer and ease back.
- Fallback: `react-split-text` (MIT, https://github.com/CyriacBr/react-split-text) with an own `requestAnimationFrame` proximity loop setting `font-variation-settings`.
- Font: the reports name no font family. Requirement: variable font with a `wght` axis covering at least 500-900, open licence (OFL or equivalent) confirmed before use, display default 700+, body 500+. No thin weights anywhere. **OPEN ITEM.** Self-host woff2, Latin subset.
- Colour `--charcoal`. If it crosses the bright sun area keep it outside the bloom and check contrast on the real frame.
- Optional accent, only after the core works: short WebGL shimmer behind or around the headline, or one short title with water-ripple displacement ported from `zebiv-code/text_ripple` (MIT, https://github.com/zebiv-code/text_ripple; port only the pass you need). Accessible WebGL text pattern: `ehaakana/codrops-text-demo` (MIT, https://github.com/ehaakana/codrops-text-demo). Never displace the whole heading in a way that hurts legibility.
- Per beat: beat 1 full response; beats 2-3 headline lifts and fades with scroll leaving a small wordmark; beat 4 may show a second short line that reacts the same way.
- Touch, keyboard, reduced motion, WebGL failure: no hover dependency. On touch a visible named button or phrase triggers the same weight animation on tap; focus plus Enter/Space does the same. Reduced motion = static heading.
- Do not use Blotter.js-based Codrops demos (TextDistortionEffects, LetterInteractions): custom licence, not MIT.
- Visual references only: https://squeezy.overnice.com/ , https://cosmic-sans.blast-foundry.com/ , https://symphonyofvines.unseen.co/

## 3.4 Audio gate and toggle

- Silent by default (browsers block audible autoplay). A persistent visible corner control "Sound off / Sound on". First click or tap creates or resumes the `AudioContext`. The mute control always works.
- Two looping beds on separate `GainNode`s. Above water: restrained mountain air. Underwater: low hydrophone bed through a `BiquadFilterNode` lowpass (about 400-900 Hz). Both barely audible under the headline, no sharp transients, no splash effect.
- Crossfade 2-4 s with equal-power curves, triggered at the surface-break `p` (about 0.60). Plain Web Audio preferred (can track `p`). Howler (MIT, https://github.com/goldfire/howler.js) is the alternative if codec fallback matters.
- Candidate files. Re-check the licence on the Freesound item page at download and keep a local licence note (creator, title, URL, licence, date):
  - Underwater: https://freesound.org/people/felix.blume/sounds/384218/ (CC0 per SoundSpool cross-check; audition, may be too active) or https://freesound.org/people/felix.blume/sounds/328300/ (cenote, CC0 indicated; check clicks and insects).
  - Above water: https://freesound.org/people/senorstudy/sounds/437284/ (author says CC0; brown-noise air bed).
  - Provisional only: https://freesound.org/people/petebuchwald/sounds/288899/ (wind and birds; CC0 not confirmed on Freesound).
- Encode small, loop-safe. Pause when the tab is hidden.
- Not allowed: The Sleepers music (CC BY-NC 4.0), any Unseen audio.

# 4. Below-hero

Principle (report 584): a calm editorial surface on `--sun-cream`. Cinematic effects stay in the hero. Motion lives only in image previews and scroll entrances.

## 4.1 Canvas to DOM handoff

- Sticky hero canvas, a cream DOM sheet rises over it, canvas pauses when out of view (`IntersectionObserver` toggling R3F `frameloop`), unmount if nothing later needs it. Pattern: https://www.karltiama.dev/blog/terrain-hero-threejs (blog pattern, no licence claimed).
- Handoff is the beat-4 wash to cream, then the first project image appears cleanly. No dark gradient bridge.
- Everything below the hero is normal DOM flow and native scroll. Do not virtualise the page into the canvas.
- Optional if later cards need WebGL: `14islands/r3f-scroll-rig` (MIT, https://github.com/14islands/r3f-scroll-rig).

## 4.2 Work grid

- Opens with a small label and a heavy headline ("Selected work").
- One wide lead project, then a measured two-column grid with one controlled offset or alternating aspect ratios. No complex masonry.
- Card: large media, title, role/category and year visible without hover, one-sentence summary, clear link ("Discover project").
- One system for all disciplines: apps in device frames, video as a short muted lazy loop (poster first), design as full-bleed imagery.
- Hover video only on pointer devices; touch taps open the case study. Hover is never the only way to learn what a piece is.
- Each case study ends with a clear next step. Short ease-out scroll reveal per card, same easing and timing as the hero.
- Pattern references (not code): https://arnaudrocca.fr/ , https://kevinhilgendorf.com/ , https://www.awwwards.com/sites/aaron-mcguire-2023-portfolio , https://www.alexbeigeweb.dev/ , https://erikamoreira.co/

## 4.3 About and contact

1. About: short first-person statement, one portrait or personal artefact, one "what I make" line (apps, design, video). Specific, not a CV.
2. Contact: one oversized heavy CTA (copy is a placeholder for the user, e.g. "Have a good idea?") with a clear email action. Secondary links quiet.
3. Footer: location and time zone, live local time only if real, realistic reply expectation, socials, small copyright. Optional slow low-contrast ticker "apps . design . video".
- References: https://www.awwwards.com/inspiration/oversized-get-in-touch-typography , https://www.awwwards.com/inspiration/footer-with-clock-component-basis-studio , https://roryphillips.dev/

## 4.4 Palette on the page

Cream baseline throughout. Gold only as highlight, thin rule, index number, button fill with charcoal text, or one feature panel. Blush at most one small signal per viewport (dot, tag, cursor detail), never pink cards. Charcoal for all type. At most one deliberate gold panel or charcoal footer to break the scroll. No alternating stripes.

# 5. Mobile spec

Mobile is a designed lighter version of the same story. The user's own phone failed on a heavy award site, so the poster route is a hard requirement.

## 5.1 Load order

1. First paint: HTML headline + a warm AVIF/WebP poster of beat 1 (composition-matched, small, JPEG fallback). It stays under the canvas until renderer creation, first frame and key assets succeed.
2. Dynamic-import the 3D code after first paint. Critical set only: valley geometry and material, sky and lighting, headline font, UI. Finite item-count progress (`LoadingManager` or Drei `useProgress`; count-based, so no false precision). Thin gold line loader.
3. Reveal beat 1, then stream underwater textures and audio before scroll reaches about `p = 0.45`. KTX2/Basis textures. If a non-critical asset fails, continue; if a critical one fails, stay on the poster.
4. Poster-only route when: no WebGL2 (`WebGL.isWebGL2Available()`, Three addon, MIT), context creation fails or is lost, `prefers-reduced-motion`, `navigator.connection.saveData` true or very slow `effectiveType` (optional API, guard it), or tier 0.

## 5.2 Quality tiers (`@pmndrs/detect-gpu`, MIT, https://github.com/pmndrs/detect-gpu)

Use the scoped package `@pmndrs/detect-gpu` (v6.0.24 observed 5 Oct 2026), pin the version. Run it without blocking the poster. The tier is a starting hint. Then watch real frame time with Drei `PerformanceMonitor` and step down one tier at a time.

| Tier | Device | Scene |
|---|---|---|
| 0 | No WebGL2 / old | Poster only, DOM content, still beat images |
| 1 low | Weak mid-range phones | Simplified two-peak mesh (about 40-80k tris), baked albedo + vertex colour (no triplanar, no splat), sky gradient, flat water with two scrolling normals, texture caustics, 3 fake shafts, 500 particles, grain + vignette only (no bloom, no CA), DPR max 1.0-1.25 |
| 2 mid | Typical phones, tablets | Full valley at lower LOD, 2-texture splat, Sky + fixed shadow swaths, half-res bloom, one combined grain/CA/vignette pass, DPR max 1.5 |
| 3 high | Flagship, desktop | Tier 2 plus triplanar cliff, low-res reflector on the lake, optional projected caustics, 2000 particles, DPR max 2.0 desktop and 1.5 phone |

Keep the cues that make low tier read as the same hero: two peaks, gold light path, waterline, grain.

## 5.3 Touch and gyro

- Native vertical scroll drives `p` (no scroll-jacking; back/forward and anchors keep working). Lenis `syncTouch: false`. Sticky canvas. Test address-bar resize in real Safari and Chrome (`svh`/`lvh`).
- No pointer parallax on touch. Optional gyro parallax behind an explicit "Enable tilt" button: in the tap handler on HTTPS, if `typeof DeviceOrientationEvent.requestPermission === 'function'` call it and subscribe only on `granted`; elsewhere feature-test real events. Clamp and damp (about 2 degrees max). Refused or unsupported means nothing breaks. No absolute orientation or compass.
- Do not swallow vertical scroll. Any drag-look uses Pointer Events and a signposted mode.
- Headline has no hover dependency (3.3). Small "scroll" cue on load.

## 5.4 Perf budgets (gates to verify on a real mid-range phone, not published standards)

| Metric | Mobile target |
|---|---|
| Frame time | 16.7 ms (60 fps). Brief dips at the surface break are fine. A stable 30 fps beats an unstable 60 |
| Draw calls | under 80 visible, investigate at 100, ceiling about 150 (`renderer.info.render.calls`) |
| Triangles | start 100-250k visible, 500k is a test ceiling only |
| GPU texture memory | about 64-100 MB, KTX2 with mipmaps, scene-4 textures upload before arrival |
| Post | one essential full-screen pass on baseline mobile, a second only if measured |
| DPR | cap 1.25-1.5 on mobile, drop to 1.0 on regress (DPR 2 = 4x the pixels) |
| Initial transfer | poster + critical scene first, never large masters |

- `<AdaptiveDpr />` and `<PerformanceMonitor />` (Drei, MIT) with bounded `dpr={[1, 1.5]}` on mobile. Downshift order: DPR, then bloom/CA off, then particles, then terrain LOD. Dead band and `onFallback` to stop oscillation.
- `frameloop="demand"` for the idle poster state, continuous only while visible and animating.
- Test cold load and cache-warm separately, and the full surface break, not just the static hero.

# 6. Asset and licence ledger

Status key. **BUILD** = MIT/CC0/BSD/public domain with evidence in the reports, allowed in the build path. **CHECK** = allowed only after the named check, not to be installed until then. **CONDITIONAL** = permissive but not MIT/CC0/BSD, needs the user's sign-off. **REF** = reference only, do not copy. **NO** = excluded.

## 6.1 Libraries and code

| Item | URL | Licence | Status / note |
|---|---|---|---|
| three (core, Sky, EffectComposer, UnrealBloomPass, PMREMGenerator) | https://github.com/mrdoob/three.js | MIT (LICENSE inspected) | BUILD |
| @react-three/fiber | https://github.com/pmndrs/react-three-fiber | MIT | BUILD |
| @react-three/drei | https://github.com/pmndrs/drei | MIT | BUILD |
| maath | https://github.com/pmndrs/maath | MIT (LICENSE inspected) | BUILD |
| @pmndrs/detect-gpu | https://github.com/pmndrs/detect-gpu | MIT | BUILD |
| Lenis | https://github.com/darkroomengineering/lenis | MIT | BUILD, re-check at lockfile |
| Howler.js | https://github.com/goldfire/howler.js | MIT | BUILD, optional |
| GL Transitions | https://github.com/gl-transitions/gl-transitions | MIT repo-level | BUILD, check file header notice |
| MagnetType | https://github.com/Liiift-Studio/MagnetType | MIT per npm registry | CHECK package name and LICENSE file |
| react-split-text | https://github.com/CyriacBr/react-split-text | MIT per GitHub/npm | BUILD (fallback) |
| text_ripple | https://github.com/zebiv-code/text_ripple | MIT per GitHub | BUILD (optional port) |
| codrops-text-demo | https://github.com/ehaakana/codrops-text-demo | MIT per GitHub | BUILD (reference) |
| Codrops LineTextHoverAnimations | https://github.com/codrops/LineTextHoverAnimations/ | MIT (LICENSE inspected) | BUILD (letter-split structure only) |
| Codrops camera fly-through | https://github.com/AndrewPrifer/CodropsCameraFlyThroughTutorial | MIT | REF (we use curves, not Theatre.js) |
| r3f-scroll-rig | https://github.com/14islands/r3f-scroll-rig | MIT | BUILD, optional |
| THREE.Terrain | https://github.com/IceCreamYou/THREE.Terrain | MIT | BUILD, optional generator |
| geo-three | https://github.com/tentone/geo-three | MIT | REF |
| lod-terrain | https://github.com/felixpalmer/lod-terrain | MIT | REF (algorithm) |
| geo-clipmap | https://github.com/tschie/geo-clipmap | MIT | REF (algorithm) |
| TerrainForge | https://github.com/Nexvia-labs/TerrainForge | MIT claimed | CHECK LICENSE file |
| UnderwaterAI site | https://github.com/Underwater-AI/underwater-ai.github.io | MIT (README + metadata) | BUILD as reference to adapt; GSAP separate licence |
| Nugget8 Ocean Scene | https://github.com/Nugget8/Three.js-Ocean-Scene | MIT (metadata) | BUILD as reference; mobile fps author-reported |
| martinRenou threejs-caustics | https://github.com/martinRenou/threejs-caustics | BSD-3-Clause (metadata) | BUILD for the algorithm; demo meshes (CC BY) and skybox (Apache-2.0) excluded |
| WaterThreeJS | https://github.com/achrefelouafi/WaterThreeJS | MIT (LICENSE inspected, report 580) | BUILD as reference; runtime unverified |
| dfrankland/threejs-water | https://github.com/dfrankland/threejs-water | MIT per GitHub | REF |
| CAUSTIC//VOLUME | https://github.com/ScottieFox/caustic-volume | MIT reported, LICENSE body not read | CHECK before any reuse |
| SeedOcean | https://github.com/reed-soul/SeedOcean | MIT per README (alpha, WebGPU-first) | REF |
| jeantimex/threejs-water | https://github.com/jeantimex/threejs-water | Conflict: report 580 inspected LICENSE as MIT; report 581 saw GitHub "Other/NOASSERTION" | CHECK; demo Flickr texture not covered |
| jeantimex precomputed_atmospheric_scattering | https://github.com/jeantimex/precomputed_atmospheric_scattering | MIT (inspected) | REF (too heavy) |
| thaslle/stylized-water | https://github.com/thaslle/stylized-water | MIT (LICENSE.txt) | REF (cartoon look) |
| cortiz2894/stylized-components | https://github.com/cortiz2894/stylized-components | MIT, asks for credit | REF (demo did not render) |
| AxiomeCG/r3f-fog-effect | https://github.com/AxiomeCG/r3f-fog-effect | MIT (metadata) | REF (demo blank) |
| kinetic-text-3d | https://github.com/odina101/kinetic-text-3d | MIT per README only | CHECK |
| pmndrs/postprocessing, @react-three/postprocessing | https://github.com/pmndrs/postprocessing | Zlib | **CONDITIONAL**, avoided by using Three EffectComposer |
| three-good-godrays | https://github.com/ameobea/three-good-godrays | ISC-like 3-clause, no SPDX id | **CONDITIONAL**, needs shadow maps, not used |
| sweriko/WebGPU-Godrays | https://github.com/sweriko/WebGPU-Godrays | MIT in package.json only | REF, not used |
| three-volumetric-pass | https://github.com/ameobea/three-volumetric-pass | No licence found, observed 1 fps | **NO** |
| The Mountains of Madness | https://github.com/AmanPriyanshu/The-Mountains-of-Madness | Apache-2.0 | **NO** (technique reference only) |
| fractal-erosion-webgl | https://github.com/Ernyoke/fractal-erosion-webgl/ | Not confirmed | **NO** until confirmed |
| typography-toolkit | https://github.com/mathonsunday/typography-toolkit | README says MIT, no LICENSE file | **NO** until confirmed |
| Codrops TextDistortionEffects / LetterInteractions (Blotter) | https://github.com/codrops/TextDistortionEffects | Custom licence, no as-is redistribution | **NO** |
| r3f-terrain | https://github.com/mozzius/r3f-terrain | Not confirmed | **NO** |
| Three TSL TerrainGenerator and height-fog examples | https://threejs.org/docs/pages/TerrainGenerator.html | MIT but WebGPU/TSL | REF |

## 6.2 Textures, HDRI, data, audio, fonts

| Item | URL | Licence | Status / note |
|---|---|---|---|
| Poly Haven rocky_terrain, rocky_terrain_03 | https://polyhaven.com/a/rocky_terrain | CC0 (https://polyhaven.com/license) | BUILD, not yet downloaded or tested |
| Poly Haven cliff_side, sparse_grass | https://polyhaven.com/a/cliff_side | CC0 | BUILD |
| ambientCG Rocks025, Rock002, Grass004 | https://ambientcg.com/view?id=Rocks025 | CC0 (https://ambientcg.com/license) | BUILD |
| OpenHDRI Gruntowa April Golden Hour | https://openhdri.org/h/gruntowa-april-golden-hour | CC0 stated on page | BUILD, optional, downsample offline |
| OpenHDRI Egg Hill Sunset Clear Sky | https://openhdri.org/h/egg-hill-sunset-clear-sky | Not displayed on fetched page | **UNCONFIRMED, do not use** |
| NASADEM / SRTM | https://www.earthdata.nasa.gov/data/catalog/lpcloud-nasadem-hgt-001 | NASA CC0 guidance unless marked | CHECK collection metadata, cite NASA |
| USGS 3DEP via OpenTopography | https://portal.opentopography.org/datasetMetadata?otCollectionID=OT.012021.4269.1 | US Government public domain | BUILD, US coverage only |
| swisstopo swissALTI3D | https://www.swisstopo.admin.ch/en/height-model-swissalti3d | OGD, attribution mandatory | **CONDITIONAL**, not in build path |
| 3DTexel Double Nordic Mountains | https://3dtexel.com/product/double-nordic-mountains/ | Listing says CC0 (third-party) | CHECK downloaded package |
| Freesound felix.blume 384218 | https://freesound.org/people/felix.blume/sounds/384218/ | CC0 via SoundSpool cross-check | CHECK on item page |
| Freesound felix.blume 328300 | https://freesound.org/people/felix.blume/sounds/328300/ | CC0 indicated | CHECK on item page |
| Freesound senorstudy 437284 | https://freesound.org/people/senorstudy/sounds/437284/ | Author text says CC0 | CHECK on item page |
| Freesound petebuchwald 288899 | https://freesound.org/people/petebuchwald/sounds/288899/ | CC0 per SoundSpool only | **UNCONFIRMED** |
| Heavy display variable font | not chosen | needs OFL or equivalent | **OPEN ITEM** |
| Envato stock footage (golden fog, sunrise mist) | https://elements.envato.com/misty-dawn-in-the-mountains-beautiful-autumn-lands-QJWAE4G | Envato licence | **NO**: the locked direction is a rendered scene, previews not allowed in end products |
| The Sleepers music | https://projects.thibautfoussard.com/the-sleepers/ | CC BY-NC 4.0 | **NO** |
| Unseen / Igloo / Symphony of Vines models, audio, shaders | various | No reuse licence | **NO**, visual reference only |

## 6.3 Reference sites (visual only)

- Symphony of Vines (Awwwards SOTD 7.43): https://symphonyofvines.unseen.co/ , dev notes https://unread.unseen.co/the-symphony-of-vines-dev-insights-c284cc4e8aa0
- Igloo Inc (SOTD 7.92): https://www.igloo.inc/ , case study https://www.awwwards.com/igloo-inc-case-study.html
- Unseen Interactive Water: https://water-scene.unseen.co/ (pipeline and audio toggle not publicly established)
- Unseen WebGL Refraction: https://unseen.co/labs/webgl-refraction/
- Unseen Convex Seascape dev article (waterline trick, lazy-load first-scene textures): https://unread.unseen.co/insights-convex-seascape-survey-4b64f2cff88b
- Unseen Studio site writeup (render-target transitions, dive-underwater camera): https://www.awwwards.com/unseen-studio-by-unseen-studio-wins-sotm-february-2023.html
- 3D Realistic Water Experiment: https://water-simulation.vercel.app/
- Susurrus: https://susurrus.vercel.app/ , https://tympanus.net/codrops/2026/04/24/susurrus-crafting-a-cozy-watercolor-world-with-three-js-and-shaders/

# 7. Build order and perf rules

## 7.1 Order

1. **Scene 1 (valley), static first.** Heightmap bake script, terrain mesh, CC0 materials, Sky + sun + key light, fake height haze, two-peak composition locked on a still camera. Check against the brief: two peaks left and right, open centre, clear air, readable rock, gold dominant. Add the post pass (grain, CA, vignette, bloom) now so everything after is tuned with it. Export the poster still from this view.
2. **Zoom.** Waypoints, position and target curves, native scroll to damped `p`, beat easing, clamped pointer parallax, pause and visibility handling.
3. **Underwater and surface break.** Water plane, crossing, `uUnderwater` uniform, texture caustics, fake shafts, particles, seabed, audio crossfade. Optional projected caustics last.
4. **Headline.** DOM `<h1>`, font, MagnetType response, touch tap button, beat fade, optional shimmer last.
5. **Below hero.** Cream handoff, work grid, about, contact, footer.
6. **Mobile pass throughout, hard-gated at the end.** Poster route, tiers, detect-gpu, budgets on a real mid-range phone.

Merge scenes only after mobile tests, load checks, DOM fallback, motion limits and a clean ledger (no CHECK, UNCONFIRMED or CONDITIONAL item in the bundle).

## 7.2 Perf rules

- Fake height fog, never volumetrics. No raymarched passes in the default path.
- Post stack: bloom (high threshold, low strength, half res on mid), plus one combined grain / chromatic fringe / vignette pass. Nothing more on mobile.
- No dynamic shadow maps.
- Target 60 fps. A stable 30 fps on low tier beats an unstable 60.
- Adaptive DPR with `AdaptiveDpr` and `PerformanceMonitor`, bounded range, one-step downshifts, dead band.
- Reuse materials, instance particles, merge static geometry, under 80 draw calls on mobile.
- Grain and glow are the premium cues, keep them on every tier (grain is nearly free, glow via restrained bloom or a cheap gradient sprite on tier 1).
- Pause rendering offscreen and when hidden. Reduced motion means a still.

# 8. Open items (not decided by the research)

1. Heavy variable display font (OFL) not chosen.
2. Headline and contact copy are placeholders.
3. Whether Zlib `pmndrs/postprocessing` is acceptable. The spec avoids it, so this only matters if the builder wants it.
4. Whether swisstopo attribution-required data is acceptable (only for a real Lauterbrunnen-based valley instead of a procedural one).
5. Audio files need an audition and a licence check at download.
6. Nothing here has been rendered. The research could not run WebGL and several demos were not visually confirmed. Treat demo-based claims as unverified until a prototype runs on a real mid-range phone.
