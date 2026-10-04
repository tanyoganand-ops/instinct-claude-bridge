# Liked animation references: source and copy targets

Checked 4 October 2026.

The five ZIPs separate original material from licensed alternatives. We found authentic Oryzo 3D models, but no downloadable original animation code for any of the five clips. Do not describe the replacements as the creators' source.

| Reference | Original source status | Copyable material included | Best part to take |
|---|---|---|---|
| HinedFx / Clay SaaS | No source link in description or bio; bio is a project intake form | Magic UI animated-list.tsx + blur-fade.tsx and upstream usage examples, MIT | Clean UI reveals and timed spring arrivals |
| Visual Runtime / glass panels | No source link in description or channel About panel | Vanilla Tilt source + ready browser build; Magic Card pointer highlight; Drei transmission material, all MIT | Cursor tilt/glare for lightweight UI; transmission shader for stronger liquid glass |
| Credit cards slider | Actually Gleb Kuznetsov / Milkinside, curated by Ravi Joon. Original says all rights reserved | Swiper Cards effect TS/CSS, full source helpers and upstream demo, MIT | Depth stacking, swipe transforms, rotate/scale/shadow logic |
| Cartier / Immersive Garden | No public source found | Drei MotionPathControls + MeshPortalMaterial and API docs, MIT | Continuous camera paths and portal scene blending |
| Oryzo.AI / Lusion | Authentic MIT OBJ meshes are public; website animation is not in that repo | Six original OBJ meshes; Lusion's separate MIT WebGL scroll example; Drei 3D/glass/portal tools | Original model geometry, same-studio canvas/DOM sync, reconstruction tools |

## Recommendation

Use the Clay clip as the pacing/layout reference, Drei MeshTransmissionMaterial for the liquid-glass material, and MotionPathControls plus MeshPortalMaterial for continuous scene transitions. Swiper is the practical card-stack interaction base. This is an implementation route, not a promise these libraries reproduce the films without new design work.

Each ZIP includes exact source files, original licence text, pinned commit IDs, a file map and detailed TAKE-NOTES. The bundles are source extracts for inspection/adaptation, not verified ready-to-run replicas. Install dependencies through their packages, preserve licence notices, and supply your own art, branding, textures, music and fonts. The Lusion scroll demo's unverified photos/font/icons were excluded; replace their paths before running it.

MIT allows commercial use, modification and redistribution if the copyright and permission notices remain. It does not grant rights to the reference footage or branding. React Bits was not bundled: its current MIT + Commons Clause prohibits redistribution of its standalone components, including bundles. A third-party glass demo without a clear licence was also not included.

## Reference links

- Clay: https://www.youtube.com/watch?v=y7E7ZmwEWyM
- HinedFx bio destination (project enquiry, not source): https://tally.so/r/zxQDk0
- Glass panels: https://www.youtube.com/watch?v=d59LRvWA3BI
- Card compilation: https://www.youtube.com/watch?v=1aIrmtniS7M
- Card original: https://dribbble.com/shots/7040213-Credit-cards-slider
- Cartier reference: https://www.awwwards.com/inspiration/transition-through-univeses-cartier-watches-wonders-2024
- Cartier official project: https://immersive-g.com/projects/cartier-watches-and-wonders-24/
- Cartier backstage: https://immersive-g.com/projects/cartier-watches-and-wonders-24/backstage/
- Oryzo clip: https://www.youtube.com/watch?v=vYrSgm1WxU8
- Oryzo official case study: https://lusion.co/projects/oryzo_ai/

## Verified repositories

- https://github.com/magicuidesign/magicui/
- https://github.com/micku7zu/vanilla-tilt.js/
- https://github.com/nolimits4web/swiper
- https://github.com/pmndrs/drei
- https://github.com/lusionltd/ORYZO-1
- https://github.com/lusionltd/WebGL-Scroll-Sync

## Specific upstream files

These observed links locate the relevant files. Branch-tip links can change; the bundled manifests pin the downloaded commits.

- https://github.com/magicuidesign/magicui/blob/main/apps/www/registry/magicui/animated-list.tsx
- https://github.com/magicuidesign/magicui/blob/main/apps/www/registry/magicui/blur-fade.tsx
- https://magicui.design/docs/components/magic-card
- https://github.com/pmndrs/drei/blob/master/src/core/MeshTransmissionMaterial.tsx
- https://github.com/pmndrs/drei/blob/master/src/core/MotionPathControls.tsx
- https://github.com/pmndrs/drei/blob/master/src/core/MeshPortalMaterial.tsx

## Evidence and limits

Full descriptions for all four YouTube references were read. The card original and both creator About panels were read live; the card's rights notice was explicit. Studio case studies and repository READMEs were checked, and licence text was inspected in downloaded source. A targeted search finding no source is not proof that private project files do not exist. Ask the creators for permission or project files if exact assets are needed. Typical technique breakdowns in TAKE-NOTES are reconstruction advice, not verified statements about the studios' private pipelines.
