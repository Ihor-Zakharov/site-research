# WebGL/3D for Award-Winning Landing Pages — Practical Reference (September 2026)

Verified against npm registry, jsdelivr package metadata, official docs, and current Codrops/blog tutorials on 2026-09-15. Version numbers below were pulled live from `registry.npmjs.org` — treat anything not explicitly dated as accurate to that day.

---

## 0. TL;DR decision table

| You want... | Reach for | Why |
|---|---|---|
| A hero WebGL scene synced to DOM text/layout, agency-grade | Vanilla three.js + a scroll-rig pattern, or R3F + `@14islands/r3f-scroll-rig` | DOM stays real (SEO, accessibility, text reflow); canvas is just paint |
| A product-configurator / interactive 3D UI with state, physics | React Three Fiber + drei + Rapier | Declarative scene graph matches React's component model; physics, gizmos, HTML overlays all solved |
| One hero shot, static camera, no interactivity budget | A looping WebM/H.265 video or scrubbed image sequence | Zero WebGL init cost, trivial LCP, no shader compile jank, works everywhere |
| No-code / non-dev team shipping fast | Spline or Unicorn Studio embed | Runtime is a few hundred KB, visual editor, GLTF/code export if you outgrow it |
| Sub-50KB footprint, full shader control, no React | OGL or raw WebGL2 | Three.js's API shape without the ~600KB module payload |
| Photoreal captured scene (product photo, real space) | Gaussian splatting via `@sparkjsdev/spark` | Native three.js integration, phone-friendly, no polygon budget to manage |
| Cutting-edge shader work, WGSL/GLSL portability | Three.js **TSL** + `WebGPURenderer` | One JS shader graph compiles to both WGSL (WebGPU) and GLSL (WebGL2 fallback) |

---

## 1. Three.js today

### 1.1 Version and release cadence

- **Current npm `latest`: `three@0.186.0`**, published **2026-09-08**. Three.js version numbers map 1:1 to the "r" release name (`0.186.0` = **r186**). Confirmed live via `registry.npmjs.org/three`.
- Cadence is roughly monthly; there is no LTS branch — you pin a version and upgrade deliberately (bundle-size and shader-chunk internals shift release to release).
- **WebGPU became production-ready at r171** (~September 2025). With Safari 26 shipping WebGPU in 2026, all three evergreen engines (Chromium, Firefox, WebKit) now support it, so shipping `WebGPURenderer` to 100% of users (with its automatic WebGL2 fallback) is viable today.
- **TSL (Three.js Shading Language)** matured to documented/stable status around **r183–r184**. r183 also renamed the node-based post FX class `PostProcessing` → **`RenderPipeline`** (same class, identical API — both names circulate in tutorials written across that window; verified directly against `THREE.RenderPipeline` used in the live `webgpu_postprocessing_bloom.html` example on the `mrdoob/three.js` `dev` branch).
- Source: [npmjs.com/package/three](https://www.npmjs.com/package/three), [github.com/mrdoob/three.js](https://github.com/mrdoob/three.js), [Three.js Shading Language wiki](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language), [Field Guide to TSL and WebGPU — Maxime Heckel](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/).

### 1.2 The module/CDN story

Three.js **no longer ships a UMD global build**. This is a hard, verifiable fact as of r186 — pulled straight from the published `package.json` `exports` map:

```json
{
  ".":            { "import": "./build/three.module.js", "require": "./build/three.cjs" },
  "./tsl":        "./build/three.tsl.js",
  "./addons":     "./examples/jsm/Addons.js",
  "./addons/*":   "./examples/jsm/*",
  "./webgpu":     "./build/three.webgpu.js",
  "./src/*":      "./src/*"
}
```

The `build/` folder physically contains only: `three.cjs`, `three.core.js`, `three.module.js`, `three.tsl.js`, `three.webgpu.js`, `three.webgpu.nodes.js`. **There is no `three.min.js` any more.** If you find a tutorial with `<script src=".../three.min.js">`, it predates roughly r150 and is stale. The only way to use three.js from a plain `<script>` tag today is an **import map** pointing at the ES module build.

**No-build / CDN setup (importmap):**

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.186.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.186.0/examples/jsm/",
    "three/tsl": "https://cdn.jsdelivr.net/npm/three@0.186.0/build/three.tsl.js",
    "three/webgpu": "https://cdn.jsdelivr.net/npm/three@0.186.0/build/three.webgpu.js"
  }
}
</script>
<script type="module">
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
</script>
```

unpkg works identically (`https://unpkg.com/three@0.186.0/build/three.module.js`); jsdelivr is generally preferred for uptime/edge caching. **Always pin the exact version** in both the `three` and `three/addons/` entries — an unpinned `three` (bare `@latest`) and a pinned `addons` path (or vice versa) is the #1 cause of "works locally, breaks in prod" CDN bugs, because addon source imports `from 'three'` internally and a version mismatch between core and addons throws immediately (duplicate-class / `instanceof` failures).

**Bundled (npm + Vite/webpack) setup** — what you want for anything beyond a single page:

```bash
npm install three
npm install -D vite
```
```js
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
```
A bundler tree-shakes unused three.js classes and lets you code-split heavy addons (loaders, TSL nodes) behind dynamic `import()`.

**WebGPU/TSL specific entry point:**
```js
import * as THREE from 'three/webgpu';   // re-exports core THREE + WebGPURenderer
import { Fn, uniform, texture, vec3, color, uv, time } from 'three/tsl';
```

Source: [threejs.org/manual](https://threejs.org/manual/), [sbcode.net importmap tutorial](https://sbcode.net/threejs/importmap/), package metadata via `data.jsdelivr.com`/`registry.npmjs.org` (fetched live).

### 1.3 WebGLRenderer vs WebGPURenderer

| | `WebGLRenderer` | `WebGPURenderer` |
|---|---|---|
| Import | `import * as THREE from 'three'` | `import * as THREE from 'three/webgpu'` |
| Backend | WebGL2 only | WebGPU, with **automatic WebGL2 fallback** if the browser/GPU lacks WebGPU |
| Shader authoring | Raw GLSL strings (`ShaderMaterial`) or legacy `NodeMaterial` | **TSL** (compiles to WGSL *or* GLSL depending on active backend) — same source either way |
| Compute shaders | Not natively (GPGPU via render-to-texture hacks, `GPUComputationRenderer`) | Native compute via `.compute()` / `instancedArray` storage buffers |
| Init | Synchronous | **Async** — you must `await renderer.init()` before first render |
| Maturity | Battle-tested since ~2011 | Production-ready since r171 (Sept 2025); still the newer code path |

Minimal WebGPU setup (note the required `await`):

```js
import * as THREE from 'three/webgpu';

const renderer = new THREE.WebGPURenderer({ antialias: true });
await renderer.init();                       // <-- required, WebGPU adapter/device request is async
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);
```

In React Three Fiber, the `gl` prop on `<Canvas>` accepts a callback that **returns a promise**, specifically to support this async constructor (added for R3F v9, see §2).

**Rule of thumb:** if you're doing straightforward PBR meshes/lighting and don't need compute shaders, `WebGLRenderer` is still simpler and has zero async ceremony. Reach for `WebGPURenderer` + TSL when you want GPU compute (large particle systems), you want one shader source that degrades gracefully, or you're building net-new in 2026 and want to be on the currently-invested code path (TSL is where mrdoob's team is putting new features — `RenderPipeline`, new node-based postprocessing, retroreflective/iridescent material nodes, etc. all land there first).

Source: [Field Guide to TSL and WebGPU](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/), [ICS Media: Getting started with Three.js on WebGPU](https://ics.media/en/entry/250501/), [R3F WebGPU discussion](https://discourse.threejs.org/t/r3f-webgpu-webgl2-fallback-tree-shaking/87188).

### 1.4 TSL (Three.js Shading Language): what it is, when to use it

TSL is a **JavaScript node-graph API** that replaces both raw GLSL strings and the old imperative `NodeMaterial` system. You write shader logic as chained JS function calls; three.js compiles the graph to **GLSL** (WebGL2 backend) or **WGSL** (WebGPU backend) from the same source — no more maintaining two shader languages or losing work when you switch renderers.

Core building blocks (verified against the official wiki and a live Codrops TSL tutorial):

```js
import { Fn, uniform, texture, uv, vec2, vec3, vec4, color, time, mix, mul, add } from 'three/tsl';

// Functions: wrap logic in Fn(), destructure named args from the first array param
const oscSine = Fn(([t = time]) => {
  return t.add(0.75).mul(Math.PI * 2).sin().mul(0.5).add(0.5);
});

// Uniforms: uniform() wraps a JS/THREE value; mutate .value like old-school uniforms
const myColor = uniform(new THREE.Color(0x0066ff));
const uProgress = uniform(0);

// Texture sampling
const detail = texture(detailMap, uv().mul(10));

// Assigning to a NodeMaterial — no shader string ever gets written
const material = new THREE.MeshStandardNodeMaterial();
material.colorNode = texture(colorMap).mul(detail);
material.positionNode = positionLocal.add(vec3(0, oscSine(time).mul(uProgress), 0));
```

Operators are **method calls, not infix operators** — this is the single biggest mental shift coming from GLSL:

```js
// GLSL:  float red = (uv.x + 2.3) * 0.3;
// TSL:
const red = uv().x.add(2.3).mul(0.3);
```

GLSL ↔ TSL concept mapping (from the official wiki):

| GLSL | TSL |
|---|---|
| `vUv` | `uv()` |
| `vWorldPosition` | `positionWorld` |
| `gl_FragColor` | `material.fragmentNode` (or a node material's `colorNode` output) |
| `diffuseColor` | `material.colorNode` |
| varying + attribute plumbing | not needed — nodes resolve their own stage automatically |

**A complete raymarched SDF material in TSL** (adapted from Codrops' liquid raymarching tutorial — real, working structure):

```js
import { Fn, vec3, float, uniform, positionLocal, max, abs, min, normalize, sin, timerLocal } from 'three/tsl';

const timer = timerLocal();

const sdSphere = Fn(([p, r]) => p.length().sub(r));

const smin = Fn(([a, b, k]) => {
  const h = max(k.sub(abs(a.sub(b))), 0).div(k);
  return min(a, b).sub(h.mul(h).mul(k).mul(0.25));
});

const sdf = Fn(([pos]) => {
  const moved = pos.add(vec3(sin(timer), 0, 0));
  const sphereA = sdSphere(moved, 0.5);
  const sphereB = sdSphere(pos, 0.3);
  return smin(sphereB, sphereA, float(0.3));   // gloopy metaball blend
});

const raymarchMaterial = new THREE.MeshBasicNodeMaterial();
raymarchMaterial.colorNode = vec3(uv(), 1);     // placeholder — real version raymarches per-pixel
```

**When to use TSL today:**
- You want `WebGPURenderer` (TSL is the *only* shader path that targets it).
- You're building a reusable node material you want to keep working across WebGL2 and WebGPU without a rewrite.
- You want the built-in PBR lighting model "for free" while injecting custom logic (`colorNode`/`normalNode`/`positionNode` hooks), instead of string-patching via `onBeforeCompile`.

**When *not* to bother:** a one-off fullscreen `ShaderMaterial` quad for a small effect, or when you're copy-adapting a Shadertoy shader — raw GLSL `ShaderMaterial` is still less ceremony for throwaway work, and virtually every WebGL creative-coding reference (Shadertoy, Book of Shaders, most Codrops tutorials) is written in GLSL you'll be translating anyway.

Source: [threejs.org/docs/pages/TSL.html](https://threejs.org/docs/pages/TSL.html), [Three.js Shading Language wiki](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language), [Codrops: Liquid Raymarching Scene using TSL](https://tympanus.net/codrops/2024/07/15/how-to-create-a-liquid-raymarching-scene-using-three-js-shading-language/), [Field Guide to TSL and WebGPU](https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/).

### 1.5 Scene/camera/renderer boilerplate with correct color management

The #1 visual bug in amateur three.js scenes is **washed-out or oversaturated color** from missing/incorrect color management. As of modern three.js, `THREE.ColorManagement.enabled = true` by default; you must still set the renderer output space, tone mapping, and per-texture color space correctly:

```js
import * as THREE from 'three';

const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
renderer.outputColorSpace   = THREE.SRGBColorSpace;       // final pixels → display-ready sRGB
renderer.toneMapping        = THREE.ACESFilmicToneMapping; // filmic HDR->LDR rolloff (avoid blown highlights)
renderer.toneMappingExposure = 1.0;
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // DPR clamp, see §7
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const scene  = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(0, 1.5, 5);

// Color textures (anything meant to be seen as color: albedo/diffuse maps, backgrounds) need sRGB:
const colorTex = new THREE.TextureLoader().load('/albedo.jpg');
colorTex.colorSpace = THREE.SRGBColorSpace;

// Data textures (normal, roughness, metalness, AO — anything read as raw numbers) must NOT be sRGB-decoded:
const normalTex = new THREE.TextureLoader().load('/normal.jpg');
normalTex.colorSpace = THREE.NoColorSpace; // (or THREE.LinearSRGBColorSpace)
```

Getting the color-texture/data-texture distinction backwards is the classic "my normal map looks wrong / my colors look washed out" bug. `GLTFLoader` sets this correctly for you automatically on `.glb` imports — the manual cases above matter mainly for hand-authored `ShaderMaterial`/plane textures.

Source: [threejs.org Color Management manual](https://threejs.org/docs/#manual/en/introduction/Color-management).

### 1.6 Resize handling + DPR clamping

```js
function onResize() {
  const { innerWidth: w, innerHeight: h } = window;
  camera.aspect = w / h;
  camera.updateProjectionMatrix();
  renderer.setSize(w, h);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // never render at native 3x/4x DPR
}
window.addEventListener('resize', onResize);
```
Clamping DPR to **2** (not `window.devicePixelRatio` raw) is the universal agency convention: a 3x-DPR phone rendering full postprocessing at native resolution is 2.25x the fragment-shader cost of a clamped 2x for a difference most users can't see. Combine with `PerformanceMonitor`-style adaptive stepping (§7) to drop to 1x under load.

### 1.7 Raycasting

```js
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

window.addEventListener('pointermove', (e) => {
  pointer.x = (e.clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(e.clientY / window.innerHeight) * 2 + 1;
});

function tick() {
  raycaster.setFromCamera(pointer, camera);
  const hits = raycaster.intersectObjects(scene.children, true); // true = recursive
  if (hits.length) { /* hits[0].object, hits[0].point, hits[0].uv */ }
  renderer.render(scene, camera);
  requestAnimationFrame(tick);
}
```
For hover-heavy landing pages with many meshes, raycasting every mesh every frame is wasteful — restrict `intersectObjects` to an explicit interactive-objects array, or throttle the raycast to `pointermove` instead of the render loop.

### 1.8 `ShaderMaterial` vs `onBeforeCompile` vs `NodeMaterial`

| | Use when | Cost |
|---|---|---|
| **`ShaderMaterial`** (full custom GLSL) | Fullscreen quads, particle systems, anything with no built-in lighting model to preserve | You own 100% of the shader — no free PBR/shadows/fog unless you code it |
| **`onBeforeCompile`** | You need MeshStandardMaterial's PBR lighting/shadows *and* a small custom tweak (vertex displacement, custom fresnel tint) | Fragile: string-matches `#include <...>` chunks against three's internal shader source, which **changes between versions**. Must set `material.customProgramCacheKey = () => uniqueString` if uniforms vary per-material-instance, or three's shader cache silently reuses the wrong compiled program |
| **`NodeMaterial` / TSL** (`MeshStandardNodeMaterial` + `colorNode`/`normalNode`/`positionNode`) | Same intent as `onBeforeCompile` but future-proof and portable to WebGPU | The modern recommended default for "modify a standard material" as of r18x — no string patching, same API on both renderer backends |

`onBeforeCompile` example (classic pattern, still valid on `WebGLRenderer`):
```js
material.onBeforeCompile = (shader) => {
  shader.uniforms.uTime = { value: 0 };
  shader.vertexShader = shader.vertexShader.replace(
    '#include <begin_vertex>',
    `#include <begin_vertex>
     transformed.y += sin(position.x * 4.0 + uTime) * 0.1;`
  );
  material.userData.shader = shader; // stash so you can update uTime.value in the render loop
};
```

### 1.9 Instancing (`InstancedMesh`)

The standard way to draw thousands of copies of one mesh in a single draw call — essential for particle-like foliage/geo fields on landing pages:

```js
const geometry = new THREE.IcosahedronGeometry(0.2, 1);
const material = new THREE.MeshStandardMaterial();
const count = 2000;
const mesh = new THREE.InstancedMesh(geometry, material, count);

const dummy = new THREE.Object3D();
for (let i = 0; i < count; i++) {
  dummy.position.set(
    (Math.random() - 0.5) * 20,
    (Math.random() - 0.5) * 20,
    (Math.random() - 0.5) * 20
  );
  dummy.rotation.set(Math.random() * Math.PI, Math.random() * Math.PI, 0);
  dummy.updateMatrix();
  mesh.setMatrixAt(i, dummy.matrix);
}
mesh.instanceMatrix.needsUpdate = true; // MUST set this or GPU buffer never updates
scene.add(mesh);
```
Per-instance color: `mesh.setColorAt(i, color)` + `mesh.instanceColor.needsUpdate = true` (requires instantiating `mesh.instanceColor` first, or the material's `vertexColors` won't pick it up automatically on some three versions — check current docs at time of use). For instance counts that change every frame based on GPU compute, prefer TSL's `instancedArray`/storage-buffer path over CPU `setMatrixAt` loops.

### 1.10 GPGPU particles

Two eras coexist and both are current:

**Legacy/WebGL2: `GPUComputationRenderer`** (`three/addons/misc/GPUComputationRenderer.js`) — ping-pongs particle state (position, velocity) between two float render targets each frame, reading/writing via fragment shaders instead of JS loops. This is what almost every Codrops GPGPU particle tutorial (2019–2025) uses:

```js
import { GPUComputationRenderer } from 'three/addons/misc/GPUComputationRenderer.js';

const WIDTH = 128; // WIDTH*WIDTH particles
const gpuCompute = new GPUComputationRenderer(WIDTH, WIDTH, renderer);
const posTexture = gpuCompute.createTexture(); // fill with initial xyz in rgb
const posVariable = gpuCompute.addVariable('texturePosition', positionFragmentShaderGLSL, posTexture);
gpuCompute.setVariableDependencies(posVariable, [posVariable]);
gpuCompute.init();

function tick() {
  gpuCompute.compute();
  particlesMaterial.uniforms.texturePosition.value =
    gpuCompute.getCurrentRenderTarget(posVariable).texture;
}
```
Each particle's index maps to one pixel of the WIDTH×WIDTH texture; xyz position is packed into the RGB channels. See [Three.js Journey: GPGPU Flow Field Particles](https://threejs-journey.com/lessons/gpgpu-flow-field-particles-shaders) and [Codrops: Crafting a Dreamy Particle Effect with GPGPU](https://tympanus.net/codrops/2024/12/19/crafting-a-dreamy-particle-effect-with-three-js-and-gpgpu/).

**Modern/WebGPU: TSL compute** — native compute shaders via storage buffers, no render-target ping-pong hack needed:
```js
import { Fn, instancedArray, instanceIndex } from 'three/tsl';

const particleCount = 100000;
const positionBuffer = instancedArray(particleCount, 'vec3');

const computeUpdate = Fn(() => {
  const pos = positionBuffer.element(instanceIndex);
  pos.addAssign(vec3(0, 0.01, 0)); // trivial example: drift upward
})().compute(particleCount);

renderer.computeAsync(computeUpdate); // per frame
```
See [Wawa Sensei: GPGPU particles with TSL & WebGPU](https://wawasensei.dev/courses/react-three-fiber/lessons/tsl-gpgpu). Curl-noise driven motion (the classic "swirling flow field" look) is covered in the shader cheat-sheet, §5.

### 1.11 Postprocessing: three built-in paths

| Library | Renderer | API shape | Use when |
|---|---|---|---|
| **`EffectComposer`** (`three/addons/postprocessing/`) | `WebGLRenderer` | Linear chain of `Pass` objects (`RenderPass` → `UnrealBloomPass` → `ShaderPass` → `OutputPass`), each a full extra fragment pass | Legacy/stable; huge amount of existing tutorial code; fine for 1–3 effects |
| **`RenderPipeline`** (formerly `PostProcessing`, r183+) | `WebGPURenderer` (TSL) | Node graph — you wire `pass()`/`bloom()`/`dotScreen()` TSL nodes together like function composition, engine shares/merges buffers automatically | New WebGPU/TSL projects; verbose GLSL pass-chaining replaced by composable nodes |
| **`postprocessing` (pmndrs, standalone npm package)** | `WebGLRenderer` | `EffectComposer` + `EffectPass` — merges *multiple effects into a single shader pass* automatically for performance | The React Three Fiber ecosystem default (`@react-three/postprocessing` wraps exactly this library — not three.js's own `EffectComposer`) |

`RenderPipeline` bloom example — **verified directly from the live `mrdoob/three.js` `webgpu_postprocessing_bloom.html` example**:
```js
import * as THREE from 'three/webgpu';
import { pass } from 'three/tsl';
import { bloom } from 'three/addons/tsl/display/BloomNode.js';

const renderer = new THREE.WebGPURenderer({ antialias: true });
await renderer.init();
renderer.toneMapping = THREE.ReinhardToneMapping;

const renderPipeline = new THREE.RenderPipeline(renderer);
const scenePass = pass(scene, camera);
const scenePassColor = scenePass.getTextureNode('output');
const bloomPass = bloom(scenePassColor);

renderPipeline.outputNode = scenePassColor.add(bloomPass);

function animate() {
  renderPipeline.render();
}
renderer.setAnimationLoop(animate);
```

pmndrs `postprocessing` (vanilla, no React) — v6.39.5 as of 2026-09-09:
```js
import { BloomEffect, EffectComposer, EffectPass, RenderPass } from 'postprocessing';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new EffectPass(camera, new BloomEffect({ intensity: 1.5 })));
// render loop: composer.render(delta) instead of renderer.render(scene, camera)
```
Built-in effects: bloom, blur, antialiasing (SMAA), color grading (sepia, brightness/contrast, hue/saturation, LUT), depth of field, chromatic aberration, noise/glitch, god rays, pixelation, outline, shockwave, SSAO, tone mapping, dot-screen/grid/scanline patterns. Zlib-licensed, ~2.9k GitHub stars.

Source: [pmndrs/postprocessing](https://github.com/pmndrs/postprocessing), [Three.js Roadmap: post-processing guide](https://threejsroadmap.com/blog/the-complete-guide-to-threejs-post-processing-in-2026), raw example fetched from `mrdoob/three.js` `dev` branch.

---

## 2. React Three Fiber ecosystem

### 2.1 Version table (live from npm registry, 2026-09-15)

| Package | Version | Published | Peer deps |
|---|---|---|---|
| `@react-three/fiber` | **9.7.0** | 2026-07-31 | `react` `>=19 <19.3`, `three` `>=0.156` (v10 alpha line in parallel testing as of Sept 2026) |
| `@react-three/drei` | **10.7.8** | 2026-08-05 | `react` `^19`, `three` `>=0.159`, `@react-three/fiber` `^9.0.0` |
| `@react-three/postprocessing` | **3.1.1** | 2026-08-27 | `@react-three/fiber` `>=9.7.0`, `postprocessing` `^6.36.0`, `three` `>=0.156.0` |
| `@react-three/rapier` | **2.2.0** | 2025-11-03 | `react` `^19`, `three` `>=0.159.0`, `@react-three/fiber` `^9.0.4` |
| `leva` | **0.10.1** | 2025-10-31 | `react`/`react-dom` `^18 \|\| ^19` |
| `@react-three/offscreen` | **0.0.8** | 2023-05-11 | **Stale/experimental** — last publish 2023, treat as unmaintained; see §7 |
| `@react-spring/three` | **10.1.2** | 2026-06-24 | `@react-three/fiber` `>=6.0`, `three` `>=0.126` |
| `troika-three-text` | **0.52.5** | 2026-07-24 | `three` `>=0.125.0` |
| `postprocessing` (pmndrs, underlies `@react-three/postprocessing`) | **6.39.5** | 2026-09-09 | — |
| `@react-three/uikit` | **1.0.76** | 2026-09-01 | Flexbox-style HTML-like UI *inside* the WebGL scene |
| `three-stdlib` | **2.36.1** | 2025-11-10 | Framework-agnostic port of `three/examples/jsm` for bundlers that dislike deep-importing addons |
| `zustand` (R3F's state dep of choice) | **5.0.15** | 2026-08-13 | actively maintained |
| `gltfjsx` (CLI, not a runtime dep) | **6.5.3** | 2024-11-04 | Hasn't needed a release in ~2 years — feature-complete for its scope |
| `@react-three/rapier`, `drei`, `fiber` all require **React 19** now | — | — | React 18 support was dropped in the v9/v10 line |

R3F v9 is a **React 19 compatibility release** with async-`gl`-prop support (needed for `WebGPURenderer.init()`), an extended `extend()` that wraps individual three.js classes directly, and a breaking change: **automatic sRGB texture-prop conversion was removed** — you now set `texture.colorSpace = THREE.SRGBColorSpace` explicitly, same as vanilla three (§1.5). A v10 alpha line was active in parallel as of September 2026.

Source: live `registry.npmjs.org` queries, [r3f.docs.pmnd.rs v9 migration guide](https://r3f.docs.pmnd.rs/tutorials/v9-migration-guide), [pmndrs/react-three-fiber releases](https://github.com/pmndrs/react-three-fiber/releases).

### 2.2 Canvas boilerplate with correct color management + WebGPU option

```jsx
import { Canvas } from '@react-three/fiber';
import * as THREE from 'three/webgpu';

function App() {
  return (
    <Canvas
      dpr={[1, 2]}                              // clamp DPR — same rationale as §1.6
      gl={async (props) => {
        const renderer = new THREE.WebGPURenderer(props);
        await renderer.init();                  // gl prop may return a Promise since v9
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.outputColorSpace = THREE.SRGBColorSpace;
        return renderer;
      }}
      camera={{ position: [0, 1.5, 5], fov: 45 }}
    >
      <mesh>
        <boxGeometry />
        <meshStandardMaterial color="orange" />
      </mesh>
    </Canvas>
  );
}
```
For plain `WebGLRenderer`, drop the async `gl` factory entirely — R3F sets sane color-management defaults on the default renderer already; you generally only need `<Canvas gl={{ toneMapping: THREE.ACESFilmicToneMapping }}>`.

### 2.3 Most useful drei helpers for landing pages

| Helper | What it does |
|---|---|
| **`ScrollControls` / `Scroll` / `useScroll`** | Wraps the canvas in a virtual scroll track; `useScroll()` gives you `{offset, delta, velocity}` (0–1 progress + per-frame velocity) to drive any uniform or transform — the R3F equivalent of a GSAP ScrollTrigger, native to the render loop |
| **`Float`** | Wraps children in a subtle idle bob/rotate (sin-wave float) — the "hero object gently drifting" look, one line instead of hand-rolled `useFrame` math |
| **`MeshTransmissionMaterial`** | Glass/liquid material layered on top of `MeshPhysicalMaterial`; adds real-time screen-space refraction, `chromaticAberration`, `distortion`, `thickness`, `ior` — see §4.9 |
| **`Environment`** | HDRI-based image-based lighting; `<Environment preset="city" />` streams a pmndrs-hosted prefiltered HDRI, or `files="/my.hdr"` for a custom one; handles the PMREM prefilter step for you (§6) |
| **`Text` / `Text3D`** | `Text` = troika MSDF text (crisp at any scale, one draw call per string); `Text3D` = real extruded/beveled 3D typography geometry (much heavier, use sparingly) |
| **`Image`** | A `<mesh>` + built-in shader that handles cover/contain fitting and a grayscale-to-color hover transition out of the box |
| **`useTexture`** | Suspense-friendly texture loader (`const tex = useTexture('/img.jpg')`), supports naming multiple textures via an object map |
| **`Html`** | Renders real DOM inside the 3D scene, position-synced to a 3D anchor each frame — the inverse of the "DOM-synced plane" pattern in §4.20 |
| **`View`** | Multiple independent "viewports" rendered by one shared canvas/GL context — the standard way to get several small 3D islands on a long marketing page without N separate WebGL contexts (browsers cap concurrent contexts around 8–16) |
| **`PerformanceMonitor` / `AdaptiveDpr` / `AdaptiveEvents`** | Runtime FPS-based quality stepping — see §7 |
| **`Preload`** | Forces all `useLoader`/`useTexture` suspense resources to resolve before first paint of the scene, avoiding pop-in |
| **`shaderMaterial`** | Factory that turns a `{uniforms, vertexShader, fragmentShader}` triple into a proper JSX-usable, auto-uniform-updating material class in one call — the standard way to author custom shaders in R3F |

Source: [pmndrs/drei](https://github.com/pmndrs/drei), [drei.docs.pmnd.rs](https://drei.docs.pmnd.rs/).

### 2.4 When R3F beats vanilla three — and when it's overkill

**R3F wins when:**
- The page is already a React app — sharing state (scroll position, theme, cursor) between DOM and 3D via normal props/context beats hand-wiring a second event system.
- You need composability: swapping a hero model, reusing a `<GlassCard>` material across five sections, driving a scene from CMS data — R3F's declarative tree makes this trivial versus manually tracking disposal/re-creation in imperative three.js.
- Physics/interaction complexity (Rapier), or many small independent 3D "widgets" scattered through a long page — `<View>` solves the multi-canvas problem cleanly.

**Vanilla three.js (or OGL) wins when:**
- It's a single, self-contained hero/background effect with no React state coupling — the reconciler is pure overhead.
- Bundle size is the binding constraint (marketing pages judged on Lighthouse) — R3F + drei + three is a non-trivial payload versus a hand-rolled scene using only the three.js classes you need.
- You're hand-tuning a bespoke shader pipeline (custom multi-pass raymarching, GPGPU fluid) where R3F's per-frame reconciliation and `useFrame` scheduling add a layer you have to reason around rather than helping.
- The whole site is static HTML (marketing/agency no-build context) — pulling in React just to host one WebGL canvas is backwards.

---

## 3. Lightweight alternatives

### 3.1 OGL

`oframe/ogl` — **1.0.11** on npm (last publish 2025-01-27; repo last commit ~April 2025 — mature/stable, not abandoned, just feature-complete for its scope). Zero dependencies, ES6 modules, **~29KB minzipped total**. API shape deliberately mirrors three.js (`Renderer`, `Camera`, `Transform`, `Program`, `Mesh`) but stays thin — "the minimum abstraction necessary," explicitly positioned for developers who want to write their own shaders rather than lean on built-in materials.

```js
import { Renderer, Camera, Transform, Box, Program, Mesh } from 'ogl';

const renderer = new Renderer();
const gl = renderer.gl;
document.body.appendChild(gl.canvas);

const camera = new Camera(gl);
camera.position.z = 5;

const scene = new Transform();
const geometry = new Box(gl);
const program = new Program(gl, {
  vertex: /* glsl */ `
    attribute vec3 position;
    uniform mat4 modelViewMatrix, projectionMatrix;
    void main() { gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }
  `,
  fragment: /* glsl */ `void main() { gl_FragColor = vec4(1.0); }`,
});
const mesh = new Mesh(gl, { geometry, program });
mesh.setParent(scene);
renderer.render({ scene, camera });
```
Best fit: agency sites that want a *lot* of custom shader work (fluid sims, raymarching, particle fields) without three.js's ~600KB module weight or its built-in-material abstraction getting in the way. Used heavily by Codrops-style creative-dev tutorials as the "we don't need three.js's kitchen sink" option.

Source: [github.com/oframe/ogl](https://github.com/oframe/ogl).

### 3.2 curtains.js — status: effectively dormant

Latest npm publish **2 years stale** as of 2026 (`v8.1.6` from ~2024). Still functions, still a clean idea (turn DOM `<img>`/`<video>` elements directly into WebGL textured planes via CSS-defined size/position), but it predates and is now subsumed by the general "DOM-synced plane" pattern (§4.20) that most agencies hand-roll on top of plain three.js or `r3f-scroll-rig` instead. **Don't start a new 2026 project on curtains.js** — read its source for the *technique*, implement on three.js/OGL directly.

Source: [curtainsjs npm](https://www.npmjs.com/package/curtainsjs), [github.com/martinlaxenaire/curtainsjs](https://github.com/martinlaxenaire/curtainsjs).

### 3.3 three.js subset builds / tree-shaking

There's no official "lite" three.js distribution — the move instead is **bundler tree-shaking** (three.js's ES modules are shake-friendly if you import only the classes you use and avoid `import * as THREE`) plus manually excluding heavy `examples/jsm` addons you don't need (loaders, controls) via dynamic `import()`. `three-stdlib` (2.36.1) exists specifically to make the `examples/jsm` addons importable from a normal package for bundlers that choke on deep-path imports into `node_modules/three/examples/...`.

### 3.4 Raw WebGL2

Still the right call for a **single, extremely simple** effect (one fullscreen shader quad, no scene graph, no camera) — a hover-distortion image effect or a fullscreen gradient/noise background genuinely doesn't need a scene graph library at all. Boilerplate is ~40 lines: compile two shaders, one `gl.TRIANGLE_STRIP` quad, a couple of uniforms, done. The tradeoff is you re-derive matrix math, resize handling, and texture-loading plumbing by hand — worth it only when the *entire* WebGL usage on the page is that one quad.

### 3.5 Spline

Browser-based 3D design tool (spline.design) positioned as the no-code/agency default for one-off hero scenes.
- **Runtime**: editor and exports render on **WebGPU with WebGL fallback**; typical scenes load in under a second and hold 60fps on mobile.
- **Export**: JPG/PNG/MP4/GIF for static/video use; **GLTF/USDZ** for interop with other 3D tools; **React, vanilla JS, Swift, Kotlin** code export, plus a **Runtime API** for programmatic control of an embedded scene (set variables, trigger events from your own JS).
- **Pricing (2026)**: free tier with unlimited scenes/basic publishing; paid tiers (~$9–20/mo range at time of research) unlock private projects, more export formats, physics, custom lighting, animation timelines, and code export/unlimited scenes for native app use.
- **Pros**: fastest path from "designer has an idea" to "shipped 3D hero," visual and collaborative, exports real GLTF so you're not fully locked in.
- **Cons**: runtime is a black box you don't control shader-level; heavier scenes still pay real WebGL cost; the free tier brands the embed.

Source: [spline.design/3d-design](https://spline.design/3d-design), [Spline pricing roundups, Sept 2026](https://www.saasworthy.com/product/spline-tool/pricing).

### 3.6 Rive

Not a 3D engine — a **2D vector state-machine animation runtime** (competes with Lottie, not three.js), included here because agencies increasingly reach for it instead of WebGL for interactive *character/icon* work where true 3D isn't the point. Key differentiator vs Lottie: **state machines** — visual graphs of animation states + transition logic + live input binding — so a designer can build "hover → press → success" interaction logic without an engineer wiring timeline scrubbing by hand. Two-way data binding between app code and the animation graph. Web runtime is small (WASM), renders vector art crisply at any size. Use when the "3D-adjacent" ask is really "a polished, interactive mascot/icon/illustration," not a spatial scene.

Source: [rive.app](https://rive.app/), [Rive State Machine overview](https://help.rive.app/editor/state-machine).

### 3.7 Unicorn Studio

No-code WebGL design tool aimed squarely at the agency/portfolio crowd — "add motion and interactivity to web projects without custom WebGL development." Ships pre-built shader/particle/distortion effect blocks with a visual parameter editor; embeds via script tag + a JSON-configured `<div>`, similar deployment model to Spline. Per a widely-cited 2025 roundup (Adam Argyle, ex-Chrome DevRel), Unicorn Studio was one of the breakout no-code WebGL tools of 2025, and by 2026 has been expanding an "experts" network (via Contra) for hire-an-implementer workflows — a strong signal of real agency adoption, not just a toy. Best fit: teams that want award-site-grade background shader effects with zero shader authoring, and are fine with a runtime-script dependency.

Source: [unicorn.studio/docs](https://www.unicorn.studio/docs/), [uithings.com overview](https://uithings.com/what-is-unicorn-studio).

### 3.8 Vectary

Browser-based 3D design tool positioned more toward **product visualization / e-commerce / configurators** than pure landing-page hero art (compare: Spline skews creative/motion, Vectary skews "photoreal product on a shelf"). No-code, runs entirely in-browser, imports from Adobe/Autodesk/Blender/Figma/Rhino/SketchUp/SolidWorks. Ships **AR and VR viewing** (mobile AR quick-look, VR headsets) without a native app, and supports single-link embedding into decks/docs as well as web pages. Free tier + paid "Business Workspace" tiers for team/admin controls. Best fit: product pages needing a spinnable/configurable hero product rather than an abstract generative background.

Source: [vectary.com](https://www.vectary.com/).

### 3.9 Decision table

| Need | Pick |
|---|---|
| Abstract generative background, agency polish, zero shader code | Unicorn Studio |
| Designer-led hero scene, some code control, GLTF export path | Spline |
| Product configurator / AR try-before-you-buy | Vectary |
| Interactive mascot/icon, not spatial | Rive |
| Full shader control, tiny footprint, dev-led | OGL or raw WebGL2 |
| Full shader control + scene graph + huge ecosystem | three.js (vanilla or R3F) |

---

## 4. The winning effect catalogue

Each entry: technique in one line, a real shader core, and a reference implementation URL.

### 4.1 Hover image distortion / displacement

**Technique**: two textures (image A, image B or just image + a grayscale "displacement" texture) blended over a `progress` uniform tweened 0→1 on hover; the displacement texture's luminance offsets the UV lookup so the transition ripples/melts instead of cross-fading linearly.
```glsl
uniform sampler2D uTexture1, uTexture2, uDisplacement;
uniform float uProgress;
varying vec2 vUv;
void main() {
  vec4 disp = texture2D(uDisplacement, vUv);
  vec2 uv1 = vUv - uProgress * disp.rg * 0.3;
  vec2 uv2 = vUv + (1.0 - uProgress) * disp.rg * 0.3;
  vec4 t1 = texture2D(uTexture1, uv1);
  vec4 t2 = texture2D(uTexture2, uv2);
  gl_FragColor = mix(t1, t2, uProgress);
}
```
Reference: [Codrops — Motion Hover Effects with Image Distortions](https://tympanus.net/codrops/2019/10/21/how-to-create-motion-hover-effects-with-image-distortions-using-three-js/), [Codrops — WebGL Distortion Hover Effects](https://tympanus.net/codrops/2018/04/10/webgl-distortion-hover-effects/).

### 4.2 RGB-shift on scroll velocity

**Technique**: a persistent GPGPU "displacement" texture accumulates mouse/scroll delta each frame (with exponential decay so it relaxes back to zero), then each color channel samples the source image at a slightly different UV offset scaled by that displacement — classic chromatic-aberration-follows-motion look.
```glsl
// accumulate (in a ping-ponged compute pass):
color.rg += uDeltaMouse * dist;   // dist = falloff by distance to cursor
color.rg *= 0.965;                // relaxation/decay each frame

// sample (in the display pass):
vec2 shift = displacement.rg * 0.001;
float strength = clamp(length(displacement.rg), 0.0, 2.0);
float r = texture2D(uTexture, finalUv + shift * (1.0 + strength * 0.25)).r;
float g = texture2D(uTexture, finalUv + shift * (1.0 + strength * 2.00)).g;
float b = texture2D(uTexture, finalUv + shift * (1.0 + strength * 1.50)).b;
gl_FragColor = vec4(r, g, b, 1.0);
```
Reference: [Codrops — Grid Displacement Texture with RGB Shift using GPGPU](https://tympanus.net/codrops/2024/08/27/grid-displacement-texture-with-rgb-shift-using-three-js-gpgpu-and-shaders/) (code above is adapted directly from this tutorial's published shaders). For scroll-velocity (not just mouse-delta) driven variants, see [Codrops — Scroll-Reactive 3D Gallery with Velocity](https://tympanus.net/codrops/2026/03/09/building-a-scroll-reactive-3d-gallery-with-three-js-velocity-and-mood-based-backgrounds/).

### 4.3 Curl-noise particle fields

**Technique**: derive a divergence-free (non-converging, swirly) vector field from a scalar/vector noise potential by taking its numerical curl — particles advected by this never clump at a single point, unlike raw gradient-descent noise fields.
```glsl
vec3 snoiseVec3(vec3 x) {
  float s  = snoise(x);
  float s1 = snoise(vec3(x.y - 19.1, x.z + 33.4, x.x + 47.2));
  float s2 = snoise(vec3(x.z + 74.2, x.x - 124.5, x.y + 99.4));
  return vec3(s, s1, s2);
}
vec3 curlNoise(vec3 p) {
  const float e = 0.1;
  vec3 dx = vec3(e, 0.0, 0.0), dy = vec3(0.0, e, 0.0), dz = vec3(0.0, 0.0, e);
  vec3 p_x0 = snoiseVec3(p - dx), p_x1 = snoiseVec3(p + dx);
  vec3 p_y0 = snoiseVec3(p - dy), p_y1 = snoiseVec3(p + dy);
  vec3 p_z0 = snoiseVec3(p - dz), p_z1 = snoiseVec3(p + dz);
  float x = p_y1.z - p_y0.z - p_z1.y + p_z0.y;
  float y = p_z1.x - p_z0.x - p_x1.z + p_x0.z;
  float z = p_x1.y - p_x0.y - p_y1.x + p_y0.x;
  return normalize(vec3(x, y, z) / (2.0 * e));
}
```
Requires the 3D simplex `snoise` from §5.1. Reference: [cabbibo/glsl-curl-noise](https://github.com/cabbibo/glsl-curl-noise), used inside GPGPU position-update shaders (§1.10).

### 4.4 GPGPU flow fields

**Technique**: §1.10's `GPUComputationRenderer` (or TSL compute) position-update shader reads `curlNoise(position * frequency + time * speed)` each frame and integrates it into velocity/position — the "thousands of particles flowing like smoke/magnetic field lines" look.
Reference: [Three.js Journey — GPGPU Flow Field Particles Shaders](https://threejs-journey.com/lessons/gpgpu-flow-field-particles-shaders), [Medium — Chaotic Flow Fields with GPGPU in R3F](https://medium.com/@midnightdemise123/creating-chaotic-flow-fields-with-gpgpu-in-react-three-fiber-f9aad608c534).

### 4.5 Fluid simulation

**Technique**: real GPU Navier-Stokes solver — separate fragment shader passes for advection, curl, vorticity confinement, divergence, iterative pressure (Jacobi) solve, and gradient subtraction, each ping-ponging between double-buffered render targets. This is not an approximation — it's an actual incompressible-fluid PDE solver running entirely on the GPU at interactive framerates.
Reference implementation: **[PavelDoGreat/WebGL-Fluid-Simulation](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)** (the canonical mouse-trail fluid dye demo copied across thousands of sites) — vanilla WebGL, drop-in. Three.js port (deforms a plane / drives smoke-like material instead of raw canvas paint): [bandinopla/threejs-fluid-simulation](https://github.com/bandinopla/threejs-fluid-simulation) (has both WebGL and WebGPU variants).

### 4.6 Metaballs / gooey

**Technique**: raymarch a scene whose distance field is a **smooth-min blend** of N sphere SDFs positioned at particle/cursor-trail points — the smin blend radius controls how "gooey" the merge looks.
```glsl
float smoothMin(float d1, float d2, float k) {
  float h = exp(-k * d1) + exp(-k * d2);
  return -log(h) / k;
}
float map(vec3 p) {
  float d = 1e5;
  for (int i = 0; i < TRAIL_LENGTH; i++) {
    float s = sdSphere(p - vec3(uPointerTrail[i], 0.0), radius - baseRadius * float(i));
    d = smoothMin(d, s, 7.0);
  }
  return d;
}
```
A CSS-only "gooey" alternative for simple 2D blob merges exists too (`filter: blur() + contrast()` "gooey filter"), but for a real interactive 3D droplet look you want the raymarched version. Reference: [Codrops — Interactive Droplet-like Metaballs with Three.js and GLSL](https://tympanus.net/codrops/2025/06/09/how-to-create-interactive-droplet-like-metaballs-with-three-js-and-glsl/) (code above adapted from this tutorial), [Codrops — Making Gooey Image Hover Effects](https://tympanus.net/codrops/2019/10/23/making-gooey-image-hover-effects-with-three-js/), GPU marching-cubes alternative: [sjpt/metaballsWebgl](https://github.com/sjpt/metaballsWebgl).

### 4.7 Raymarched SDF scenes

**Technique**: no geometry at all — a single fullscreen triangle's fragment shader marches a ray from the camera through each pixel, stepping by the scene SDF's returned distance until it's ~0 (a hit) or a max-distance bailout, à la Inigo Quilez's Shadertoy demos.
```glsl
float map(vec3 p) { return sdSphere(p, 1.0); } // scene SDF

vec3 raymarch(vec3 ro, vec3 rd) {
  float t = 0.0;
  for (int i = 0; i < 80; i++) {
    vec3 p = ro + rd * t;
    float d = map(p);
    if (d < 0.001) return shade(p, calcNormal(p));
    t += d;
    if (t > 100.0) break;
  }
  return backgroundColor;
}
// normals via central-difference gradient of the SDF:
vec3 calcNormal(vec3 p) {
  vec2 e = vec2(0.0001, 0.0);
  return normalize(vec3(
    map(p + e.xyy) - map(p - e.xyy),
    map(p + e.yxy) - map(p - e.yxy),
    map(p + e.yyx) - map(p - e.yyx)
  ));
}
```
Reference: [Inigo Quilez — raymarching distance fields](https://iquilezles.org/articles/raymarchingdf/), [Maxime Heckel — Painting with Math: A Gentle Study of Raymarching](https://blog.maximeheckel.com/posts/painting-with-math-a-gentle-study-of-raymarching/), [Codrops — Liquid Raymarching Scene using TSL](https://tympanus.net/codrops/2024/07/15/how-to-create-a-liquid-raymarching-scene-using-three-js-shading-language/) for the TSL port of this exact pattern.

### 4.8 Fresnel / iridescent / holographic materials

**Technique**: classic fresnel = `pow(1.0 - dot(viewDir, normal), power)` used as a rim-light mix factor; "holographic" = fresnel term modulating a hue-shifting gradient (often UV- or normal-driven) plus scanline/noise. For physically-based iridescence, three.js's `MeshPhysicalMaterial` has built-in `iridescence`, `iridescenceIOR`, and `iridescenceThicknessRange` properties (thin-film interference model) — no custom shader needed for the "soap bubble / oil slick" look.
```glsl
float fresnel = pow(1.0 - max(dot(viewDir, normal), 0.0), 2.5);
vec3 holo = mix(baseColor, rainbow(vUv.y + uTime * 0.1), fresnel);
gl_FragColor = vec4(holo, fresnel * 0.8 + 0.2);
```
```js
// Built-in, no custom shader:
const mat = new THREE.MeshPhysicalMaterial({
  iridescence: 1,
  iridescenceIOR: 1.3,
  iridescenceThicknessRange: [100, 400],
});
```
Reference: [Maxime Heckel — On Shaping Light](https://blog.maximeheckel.com/posts/on-shaping-light/), three.js `MeshPhysicalMaterial` docs.

### 4.9 Glass & transmission

**Technique**: `MeshPhysicalMaterial.transmission` (built into three.js core) gives basic see-through refraction but requires three.js to render the scene to a texture behind the object first (one extra full-scene pass per transmissive object — expensive with many instances). drei's **`MeshTransmissionMaterial`** layers additional shaders on top for `chromaticAberration`, `distortion`/`distortionScale` (animatable glass-warp), `thickness`, `ior`, `roughness`. For chromatic **dispersion** specifically (light splitting into RGB by wavelength — different IOR per channel):
```glsl
vec3 refractVecR = refract(eyeVector, normal, 1.0 / uIorR);
vec3 refractVecG = refract(eyeVector, normal, 1.0 / uIorG);
vec3 refractVecB = refract(eyeVector, normal, 1.0 / uIorB);
float R = texture2D(uTexture, uv + refractVecR.xy).r;
float G = texture2D(uTexture, uv + refractVecG.xy).g;
float B = texture2D(uTexture, uv + refractVecB.xy).b;
```
**Performance warning**: every mesh using `transmission`/`MeshTransmissionMaterial` triggers a separate full-scene re-render — two or three glass objects on screen can double or triple your draw-call/fill-rate budget. Keep transmissive objects to a small, deliberate count (usually just the hero object).
Reference: [drei MeshTransmissionMaterial](http://drei.docs.pmnd.rs/shaders/mesh-transmission-material), [Codrops — Warping 3D Text Inside a Glass Torus](https://tympanus.net/codrops/2025/03/13/warping-3d-text-inside-a-glass-torus/), [Maxime Heckel — Refraction, dispersion, and other shader light effects](https://blog.maximeheckel.com/posts/refraction-dispersion-and-other-shader-light-effects/) (dispersion code above adapted from this article).

### 4.10 Caustics

**Technique**: not real photon tracing — a plausible-looking cheat. Render the refracting surface's **normals** to an offscreen texture, then in a second pass compare the surface area a light ray patch covers *before* vs *after* refraction (approximated via screen-space derivatives `dFdx`/`dFdy` on the refracted-ray texture): where rays converge (`oldArea/newArea > 1`), caustic intensity goes up. Composited onto a receiving surface (e.g. a "pool floor" plane) with `THREE.CustomBlending` (`OneFactor`/`SrcAlphaFactor`) and a chromatic-aberration-style multi-sample offset for the light-splitting look.
Reference: [Maxime Heckel — Shining a Light on Caustics with Shaders and React Three Fiber](https://blog.maximeheckel.com/posts/caustics-in-webgl/) (uses `useFBO` from drei for the render targets).

### 4.11 Liquid / blob morphing

**Technique**: same raymarched smooth-min SDF approach as metaballs (§4.6), but with the sphere/primitive positions driven by simplex-noise-perturbed animation rather than pointer position, plus a noise-based surface color/normal perturbation for a "mercury/liquid droplet" material look:
```glsl
float rnd3D(vec3 p) { return fract(sin(dot(p, vec3(12.9898, 78.233, 37.719))) * 43758.5453123); }
// trilinear-interpolated 3D value noise built on rnd3D, then:
vec3 dropletColor(vec3 normal, vec3 rayDir) {
  vec3 reflectDir = reflect(rayDir, normal);
  float n1 = noise3D(reflectDir * 2.0 + uTime);
  float n2 = noise3D(reflectDir * 2.0 - uTime);
  return (vec3(0.1765, 0.1255, 0.2275) * n1 + vec3(0.4118, 0.4118, 0.4157) * n2) * 2.3;
}
```
Reference: same as §4.6, [Codrops droplet metaballs tutorial](https://tympanus.net/codrops/2025/06/09/how-to-create-interactive-droplet-like-metaballs-with-three-js-and-glsl/) (code verbatim from its published shader).

### 4.12 Text as MSDF (troika-three-text)

**Technique**: Multi-channel Signed Distance Field text rendering — glyphs stay crisp at any scale/zoom with one texture atlas and one draw call per text block, versus blurry bitmap-font textures or expensive extruded-geometry 3D text. `troika-three-text` (**0.52.5**, updated 2026-07-24) parses `.ttf`/`.otf`/`.woff` directly (via Typr) and generates the SDF atlas on the fly, off the main thread (web worker), with full kerning/ligatures/bidi/fallback-font support.
```js
import { Text } from 'troika-three-text';
const myText = new Text();
scene.add(myText);
myText.text = 'Hello world!';
myText.fontSize = 0.2;
myText.color = 0x9966ff;
myText.anchorX = 'center';
myText.maxWidth = 4;
myText.sync(() => { /* fires once SDF layout is ready */ });
// always myText.dispose() on unmount to free the atlas
```
In R3F, drei's `<Text>` wraps this exact library. Reference: [protectwise/troika — troika-three-text](https://github.com/protectwise/troika/tree/main/packages/troika-three-text), [Codrops — Responsive and SEO-friendly WebGL Text](https://tympanus.net/codrops/2025/06/05/how-to-create-responsive-and-seo-friendly-webgl-text/) (covers keeping real, crawlable DOM text in sync with the WebGL rendition — see also §4.20).

### 4.13 Scroll-driven camera paths

**Technique**: author a `CatmullRomCurve3` through a handful of hand-placed keyframe points; each frame, sample `curve.getPointAt(scrollProgress)` for camera position and a second (slightly ahead) sample for the lookAt target, so the camera flies a smooth spline as the user scrolls instead of teleporting between fixed shots.
```js
const curve = new THREE.CatmullRomCurve3([
  new THREE.Vector3(0, 0, 10),
  new THREE.Vector3(5, 2, 5),
  new THREE.Vector3(0, 4, -5),
  new THREE.Vector3(-5, 1, -10),
]);
function updateCameraFromScroll(t) { // t = 0..1 scroll progress
  camera.position.copy(curve.getPointAt(t));
  const lookAtPoint = curve.getPointAt(Math.min(t + 0.01, 1));
  camera.lookAt(lookAtPoint);
}
```
In R3F, drei's `useScroll()` gives you `scroll.offset` directly for `t`. This is the backbone of nearly every "scroll through a 3D product story" award-site section.

### 4.14 Infinite tunnels / galleries

**Technique**: domain-repetition — `mod()` world-space position by a tile size and re-center, so a small tiled geometry/texture reads as an infinite repeating corridor without needing infinite actual geometry:
```glsl
vec3 repeat(vec3 p, vec3 spacing) {
  return mod(p + 0.5 * spacing, spacing) - 0.5 * spacing;
}
float map(vec3 p) {
  vec3 rp = repeat(p, vec3(4.0, 4.0, 4.0));
  return sdBox(rp, vec3(1.0));
}
```
For a *gallery* (not abstract tunnel): move the camera at constant Z-speed through a line of `InstancedMesh` image planes spaced evenly, with fog (`scene.fog = new THREE.Fog(...)`) hiding the pop-in/out distance to fake infinity cheaply.

### 4.15 Point-cloud & Gaussian splatting

**Point clouds**: plain `THREE.Points` + `BufferGeometry` (from a `.ply` via `PLYLoader`), `PointsMaterial` with `size`/`sizeAttenuation` — cheap, good for abstract data-viz-style backgrounds.

**Gaussian splatting** (photoreal captured scenes — a real space/object scanned, not a polygon model): the ecosystem's default is now **`@sparkjsdev/spark`** (**2.2.0**, published 2026-09-11 — very actively maintained), purpose-built for three.js integration and explicitly the pick "if you're delivering to phones." `@mkkellogg/gaussian-splats-3d` (0.4.7, Jan 2025) is the older/alternate renderer. Three.js itself has also gained a minimal **native** splat loader/renderer for the SPZ format (reportedly ~7KB on top of base three, three lines of code to display a splat) — check the current `three/examples/jsm/loaders/` for the latest state before reaching for a third-party dependency on a simple "just show me this splat" use case.
```js
import { SplatMesh } from '@sparkjsdev/spark';
const splat = new SplatMesh({ url: '/scene.spz' });
scene.add(splat);
```
(Exact `SplatMesh` API should be checked against [sparkjs.dev/docs](https://sparkjs.dev/docs/overview/) at implementation time — this is a fast-moving library.)
Source: [github.com/sparkjsdev/spark](https://github.com/sparkjsdev/spark), [radiancefields.substack.com — Gaussian Splatting in August 2026](https://radiancefields.substack.com/p/gaussian-splatting-in-august-2026).

### 4.16 Video-texture masks

**Technique**: `THREE.VideoTexture(videoEl)` feeds a `<video playsinline muted loop autoplay>` element's frames straight into a shader as `uVideoTexture`; used as a **luminance mask** multiplied against a gradient/pattern, or as the alpha channel of a reveal effect — much cheaper than a real-time particle/fluid sim for "liquid video reveal" hero sections while looking equally rich, because all the expensive simulation was pre-rendered to the video file.
```glsl
float lum = dot(texture2D(uVideoTexture, vUv).rgb, vec3(0.299, 0.587, 0.114));
vec3 color = mix(colorA, colorB, lum);
```

### 4.17 Matcap materials

**Technique**: `MeshMatcapMaterial` — lighting is entirely baked into one small texture, sampled by remapping the **view-space normal's xy** into a 0–1 UV. Zero real-time lighting computation, zero environment map, **one texture sample per fragment** — the cheapest good-looking "lit 3D object" material available, ideal for a hero object on a tight frame budget.
```js
const mat = new THREE.MeshMatcapMaterial({ matcap: matcapTexture });
```
Free matcap texture packs: three.js ships a small set under `examples/textures/matcaps/`; broader community sets exist (search "matcap pack CC0").

### 4.18 ASCII / halftone / dither post effects

**Technique — dithering**: ordered (Bayer matrix) dithering converts smooth tone gradients into stylized binary/limited-palette pixel patterns, faking more perceived colors/greys than are actually rendered — the retro-bitmap-display look.
**Technique — shape-aware ASCII** (2026 state of the art, beyond the classic luminance-ramp approach): a classic ASCII shader just buckets per-cell brightness into a character ramp (`.:-=+*#%@`), which can't distinguish a diagonal edge from a flat one at equal brightness. The shape-aware version runs three passes: (1) render scene to an offscreen target, (2) one fragment per character cell samples **16 spatial positions** (6 inner + 10 neighbor taps) to build a 6-value "shape vector," then linear-scans all 95 printable glyphs' precomputed shape vectors for the closest match:
```glsl
int best = 0; float bestD = 1e9;
for (int g = 0; g < uGlyphCount; g++) {
  float d = 0.0;
  for (int i = 0; i < 6; i++) {
    float diff = v[i] - texelFetch(tShapes, ivec2(i, g), 0).r;
    d += diff * diff;
  }
  if (d < bestD) { bestD = d; best = g; }
}
```
(3) a final pass stamps the winning glyph from a font atlas per cell. Reference: [Codrops — Beyond the Luminance Ramp: A Shape-Aware ASCII Renderer in Three.js](https://tympanus.net/codrops/2026/09/04/beyond-the-luminance-ramp-a-shape-aware-ascii-renderer-in-three-js/) (code above verbatim), [Codrops — Efecto: Real-Time ASCII and Dithering Effects](https://tympanus.net/codrops/2026/01/04/efecto-building-real-time-ascii-and-dithering-effects-with-webgl-shaders/), [Codrops — Building a Real-Time Dithering Shader](https://tympanus.net/codrops/2025/06/04/building-a-real-time-dithering-shader/), [Maxime Heckel — The Art of Dithering and Retro Shading for the Web](https://blog.maximeheckel.com/posts/the-art-of-dithering-and-retro-shading-web/), and three.js's own built-in example: [`webgl_effects_ascii.html`](https://threejs.org/examples/webgl_effects_ascii.html).

### 4.19 Particle text/logo morphs

**Technique**: sample target positions for "particle = text glyph pixel" either by rasterizing text/logo to an offscreen canvas and reading pixel alpha to build a target-position texture, or by sampling points across a loaded glyph/logo geometry; each particle's GPGPU update shader lerps from its current position toward the corresponding target position (often through a curl-noise "explosion" midpoint for the transition). Reference: [Three.js Journey — Particles Morphing Shader](https://threejs-journey.com/lessons/particles-morphing-shader), [Wawa Sensei — GPGPU particles with TSL & WebGPU](https://wawasensei.dev/courses/react-three-fiber/lessons/tsl-gpgpu) (modern TSL/compute version), CodePen: [Interactive particles text with three.js](https://codepen.io/sanprieto/pen/XWNjBdb).

### 4.20 "WebGL as a background layer under DOM text" — the dominant agency pattern

This is the single most common technique behind award-site hero sections, and it's worth explaining precisely because it's misunderstood as "render the text in WebGL" — **it's the opposite**: keep text as real, crawlable, accessible DOM; use WebGL purely as a synced paint layer.

**Why not render text in WebGL directly**: you lose SEO, text selection, accessibility, find-in-page, and font-loading/FOUT handling that the browser already solved — all to gain... nothing, if the text itself isn't what's being distorted.

**The technique, two flavors:**

**A) DOM element *is* the texture source** (curtains.js-style): grab the actual `<img>`/`<video>` element, upload its current frame as a WebGL texture, draw a plane at that element's exact screen rect, hide the original element visually (`opacity:0` or `color: transparent`, never `display:none` — keep it in the accessibility tree and layout flow). CSS still fully controls size/position/responsiveness; WebGL just reads `getBoundingClientRect()` each frame or on resize.

**B) Single fixed fullscreen canvas + proxy DOM elements** (the `lusion.co` / `14islands/r3f-scroll-rig` pattern — scales to many synced elements without per-element canvas/context overhead, and browsers cap concurrent WebGL contexts around 8–16 so this matters past a handful of elements):
1. One `<canvas>` fixed to the viewport, above or below the DOM.
2. Real (often invisible/opacity-0) DOM elements participate in normal document flow — this is what makes it "progressively enhanced": disable JS or fail WebGL detection and the DOM layout is still correct/complete.
3. Every `requestAnimationFrame`: read each proxy's `getBoundingClientRect()`, convert screen pixels → 3D world units, and position/scale a corresponding mesh to match.
4. **The critical desync bug to avoid**: native scroll events and `requestAnimationFrame` are not guaranteed to fire in the same tick, so reading `window.scrollY` directly inside rAF causes visible one-frame judder between the DOM (which the browser scrolls immediately/natively) and the WebGL plane (which only updates on the next rAF). The fix used by every serious implementation: **virtualize scroll** — either lock the body (`position: fixed`) and drive scroll via `transform: translateY()` computed by a smooth-scroll library (Lenis is the current standard), or otherwise ensure both DOM and WebGL read from one single authoritative scroll-progress number updated once per rAF tick.
5. Pixel-perfect camera math — size the camera frustum so 1 three.js unit = 1 CSS pixel at the z=0 plane:
```js
function syncCameraToPixels(camera, distance) {
  camera.fov = 2 * Math.atan(window.innerHeight / 2 / distance) * (180 / Math.PI);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.position.z = distance;
  camera.updateProjectionMatrix();
}
// then: mesh.position.set(rect.left - innerWidth/2 + rect.width/2, -(rect.top - innerHeight/2 + rect.height/2), 0);
//       mesh.scale.set(rect.width, rect.height, 1);
```
Reference: [r3f-scroll-rig (14islands)](https://github.com/14islands/r3f-scroll-rig) (code example in §2 above is close to its actual public API), [JOYCO — WebGL Scroll Sync devlog](https://hub.joyco.studio/logs/08-webgl-scroll-sync), [Lusion's live demo](https://webgl-scroll-sync.lusion.co/), [Codrops — Responsive and SEO-friendly WebGL Text](https://tympanus.net/codrops/2025/06/05/how-to-create-responsive-and-seo-friendly-webgl-text/). A full working minimal version of pattern (B) is in the starter file at the end of this document.

---

## 5. Shader fundamentals cheat-sheet

### 5.1 Noise

**Value noise (2D)** — hash + bilinear interpolation, the cheapest useful noise, classic Book of Shaders chapter 11 form:
```glsl
float random(vec2 st) {
  return fract(sin(dot(st.xy, vec2(12.9898, 78.233))) * 43758.5453123);
}
float noise(vec2 st) {
  vec2 i = floor(st);
  vec2 f = fract(st);
  float a = random(i);
  float b = random(i + vec2(1.0, 0.0));
  float c = random(i + vec2(0.0, 1.0));
  float d = random(i + vec2(1.0, 1.0));
  vec2 u = f * f * (3.0 - 2.0 * f); // smoothstep-style interpolant
  return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}
```
Source: [thebookofshaders.com/11/](https://thebookofshaders.com/11/).

**Simplex noise (3D)** — the industry-standard Ashima/McEwan implementation, MIT licensed, verified exact source below (this is *the* snippet pasted into a huge fraction of all creative-coding shaders on the web — get it right, don't hand-retype it):
```glsl
// Ashima Arts / Ian McEwan — MIT License — github.com/ashima/webgl-noise, github.com/stegu/webgl-noise
vec3 mod289(vec3 x) { return x - floor(x * (1.0/289.0)) * 289.0; }
vec4 mod289(vec4 x) { return x - floor(x * (1.0/289.0)) * 289.0; }
vec4 permute(vec4 x) { return mod289(((x*34.0)+10.0)*x); }
vec4 taylorInvSqrt(vec4 r) { return 1.79284291400159 - 0.85373472095314 * r; }

float snoise(vec3 v) {
  const vec2 C = vec2(1.0/6.0, 1.0/3.0);
  const vec4 D = vec4(0.0, 0.5, 1.0, 2.0);
  vec3 i  = floor(v + dot(v, C.yyy));
  vec3 x0 = v - i + dot(i, C.xxx);
  vec3 g = step(x0.yzx, x0.xyz);
  vec3 l = 1.0 - g;
  vec3 i1 = min(g.xyz, l.zxy);
  vec3 i2 = max(g.xyz, l.zxy);
  vec3 x1 = x0 - i1 + C.xxx;
  vec3 x2 = x0 - i2 + C.yyy;
  vec3 x3 = x0 - D.yyy;
  i = mod289(i);
  vec4 p = permute(permute(permute(
            i.z + vec4(0.0, i1.z, i2.z, 1.0))
          + i.y + vec4(0.0, i1.y, i2.y, 1.0))
          + i.x + vec4(0.0, i1.x, i2.x, 1.0));
  float n_ = 0.142857142857;
  vec3 ns = n_ * D.wyz - D.xzx;
  vec4 j = p - 49.0 * floor(p * ns.z * ns.z);
  vec4 x_ = floor(j * ns.z);
  vec4 y_ = floor(j - 7.0 * x_);
  vec4 x = x_ * ns.x + ns.yyyy;
  vec4 y = y_ * ns.x + ns.yyyy;
  vec4 h = 1.0 - abs(x) - abs(y);
  vec4 b0 = vec4(x.xy, y.xy);
  vec4 b1 = vec4(x.zw, y.zw);
  vec4 s0 = floor(b0) * 2.0 + 1.0;
  vec4 s1 = floor(b1) * 2.0 + 1.0;
  vec4 sh = -step(h, vec4(0.0));
  vec4 a0 = b0.xzyw + s0.xzyw * sh.xxyy;
  vec4 a1 = b1.xzyw + s1.xzyw * sh.zzww;
  vec3 p0 = vec3(a0.xy, h.x);
  vec3 p1 = vec3(a0.zw, h.y);
  vec3 p2 = vec3(a1.xy, h.z);
  vec3 p3 = vec3(a1.zw, h.w);
  vec4 norm = taylorInvSqrt(vec4(dot(p0,p0), dot(p1,p1), dot(p2,p2), dot(p3,p3)));
  p0 *= norm.x; p1 *= norm.y; p2 *= norm.z; p3 *= norm.w;
  vec4 m = max(0.5 - vec4(dot(x0,x0), dot(x1,x1), dot(x2,x2), dot(x3,x3)), 0.0);
  m = m * m;
  return 105.0 * dot(m*m, vec4(dot(p0,x0), dot(p1,x1), dot(p2,x2), dot(p3,x3)));
}
```
Source: [ashima/webgl-noise, noise3D.glsl](https://github.com/ashima/webgl-noise/blob/master/src/noise3D.glsl) (fetched verbatim).

**Curl noise**: see §4.3 — built on top of `snoise` above.

### 5.2 fbm (fractal Brownian motion)

Stack multiple octaves of noise at increasing frequency/decreasing amplitude for natural-looking detail (clouds, marble, terrain):
```glsl
#define NUM_OCTAVES 5
float fbm(vec2 st) {
  float v = 0.0;
  float a = 0.5;
  vec2 shift = vec2(100.0);
  mat2 rot = mat2(cos(0.5), sin(0.5), -sin(0.5), cos(0.5)); // rotate each octave to reduce axial bias
  for (int i = 0; i < NUM_OCTAVES; ++i) {
    v += a * noise(st);
    st = rot * st * 2.0 + shift;
    a *= 0.5;
  }
  return v;
}
```
Source: [thebookofshaders.com/13/](https://thebookofshaders.com/13/) (fbm/noise chapter).

### 5.3 SDF primitives + smooth-min

```glsl
float sdSphere(vec3 p, float r) { return length(p) - r; }
float sdBox(vec3 p, vec3 b) { vec3 q = abs(p) - b; return length(max(q,0.0)) + min(max(q.x,max(q.y,q.z)),0.0); }
float sdPlane(vec3 p, vec3 n, float h) { return dot(p, n) + h; }
float sdTorus(vec3 p, vec2 t) { vec2 q = vec2(length(p.xz) - t.x, p.y); return length(q) - t.y; }
```
Smooth-min variants (Inigo Quilez — polynomial is the most common default; use `k` in the 0.1–1.0 range to taste):
```glsl
// polynomial (quadratic) — cheapest, most common default
float smin(float a, float b, float k) {
  k *= 4.0;
  float h = max(k - abs(a - b), 0.0) / k;
  return min(a, b) - h*h*k*(1.0/4.0);
}
// exponential — smoother/rounder falloff, costs a log2/exp2
float smin_exp(float a, float b, float k) {
  float r = exp2(-a/k) + exp2(-b/k);
  return -k * log2(r);
}
// root/power — cheap, no transcendental calls
float smin_root(float a, float b, float k) {
  k *= 2.0;
  float x = b - a;
  return 0.5 * (a + b - sqrt(x*x + k*k));
}
```
Source: [iquilezles.org/articles/smin](https://iquilezles.org/articles/smin/) (all three verified exact from the source article; a cubic-polynomial variant and a full "normalized kernels" treatment are also there for smin nerds).

### 5.4 Easing in GLSL

Standard Penner-style curves, ported to GLSL (`t` in `[0,1]`):
```glsl
float easeInOutCubic(float t) {
  return t < 0.5 ? 4.0*t*t*t : 1.0 - pow(-2.0*t + 2.0, 3.0) / 2.0;
}
float easeOutElastic(float t) {
  float c4 = (2.0 * 3.14159265) / 3.0;
  return t == 0.0 ? 0.0 : t == 1.0 ? 1.0 :
    pow(2.0, -10.0*t) * sin((t*10.0 - 0.75) * c4) + 1.0;
}
float easeOutExpo(float t) {
  return t == 1.0 ? 1.0 : 1.0 - pow(2.0, -10.0 * t);
}
```
Full reference set (cubic/quart/quint/sine/circ/back/elastic/bounce × in/out/inOut): [easings.net](https://easings.net/) — every curve there ports to GLSL in 1–3 lines.

### 5.5 UV manipulation & aspect-correct UVs

**The single most common shader bug on the web**: sampling `vUv` directly on a non-square plane stretches noise/patterns to match the plane's aspect ratio instead of staying visually uniform. Fix by correcting UVs against `resolution`/plane aspect before using them for anything pattern-like:
```glsl
uniform vec2 uResolution; // canvas or plane pixel size
vec2 aspectCorrectedUv(vec2 uv, vec2 resolution) {
  vec2 ratio = vec2(
    min(resolution.x / resolution.y, 1.0),
    min(resolution.y / resolution.x, 1.0)
  );
  return vec2(
    (uv.x - 0.5) * ratio.x + 0.5,
    (uv.y - 0.5) * ratio.y + 0.5
  );
}
```
Other common UV tricks: `uv = uv * 2.0 - 1.0` (remap 0–1 → -1..1, centers origin for radial effects), `fract(uv * tiles)` (tiling), rotating UVs with a 2D rotation matrix around `(0.5,0.5)` before sampling.

### 5.6 `smoothstep` band tricks

`smoothstep(edge0, edge1, x)` is the shader Swiss army knife — anti-aliased thresholds, gradient bands, masks:
```glsl
float band = smoothstep(0.4, 0.5, x) - smoothstep(0.5, 0.6, x); // an anti-aliased stripe
float mask = smoothstep(0.0, 0.02, sdfValue);                    // anti-aliased SDF edge (crisp, no jaggies)
float glow = 1.0 - smoothstep(0.0, radius, dist);                 // soft radial falloff
```
Always prefer `smoothstep` edges over hard `step()`/`if` comparisons for anything meant to look clean at 1x DPR — it's free antialiasing.

### 5.7 Dithering

Ordered (Bayer) dithering — quantize color after adding a per-pixel threshold from a tiled Bayer matrix, turning banding into an intentional retro pattern instead of a smooth (but GPU-cheap ~8-bit banded) gradient:
```glsl
float bayerDither(vec2 fragCoord) {
  int x = int(mod(fragCoord.x, 4.0));
  int y = int(mod(fragCoord.y, 4.0));
  mat4 bayer = mat4(
     0.0,  8.0,  2.0, 10.0,
    12.0,  4.0, 14.0,  6.0,
     3.0, 11.0,  1.0,  9.0,
    15.0,  7.0, 13.0,  5.0
  ) / 16.0;
  return bayer[x][y];
}
// usage: color = floor(color * levels + bayerDither(gl_FragCoord.xy)) / levels;
```
Source: [Codrops — Building a Real-Time Dithering Shader](https://tympanus.net/codrops/2025/06/04/building-a-real-time-dithering-shader/), [Maxime Heckel — The Art of Dithering and Retro Shading for the Web](https://blog.maximeheckel.com/posts/the-art-of-dithering-and-retro-shading-web/).

### 5.8 Screen-space vs object-space

- **Object-space** (model-local coordinates, `position`/`normal` attributes before any matrix multiply): effect sticks to the mesh regardless of camera/world transform — correct for procedural surface texture (wood grain, marble veining) that shouldn't "swim" as the object rotates.
- **World-space** (`positionWorld` in TSL, or `(modelMatrix * vec4(position,1.0)).xyz` in GLSL): use when the effect should relate to a fixed point in the scene (e.g. a global light/wind direction) rather than the object's own orientation.
- **Screen-space** (`gl_FragCoord.xy / uResolution`, or `vUv` on a fullscreen postprocessing quad): use for postprocessing (dither, ASCII, chromatic aberration, vignette) and for effects that should read as "painted on the viewport" rather than "part of the object" (e.g. cursor-follow glow that ignores camera rotation).

### 5.9 Uniform naming convention

The de facto agency/Codrops convention — `u`-prefixed camelCase, consistent enough across tutorials/starters that reusing these names makes your shaders instantly legible to any collaborator:

| Uniform | Type | Meaning |
|---|---|---|
| `uTime` | `float` | Elapsed seconds, drives idle animation |
| `uMouse` | `vec2` | Normalized (or pixel) cursor position, usually smoothed/lerped, not raw |
| `uResolution` | `vec2` | Canvas or element pixel size, for aspect correction |
| `uProgress` | `float` | 0–1 tween/scroll/hover progress driving a transition |
| `uVelocity` | `float` or `vec2` | Scroll or pointer speed, decayed each frame, drives motion-reactive distortion |

### 5.10 Wiring scroll velocity / mouse inertia into uniforms

Both are the same pattern: **track a target value from the raw event, then ease your uniform toward it every rAF tick** (never assign raw input directly — that's what causes jittery, un-cinematic motion):
```js
let mouseTarget = new THREE.Vector2();
let mouseSmoothed = new THREE.Vector2();
window.addEventListener('pointermove', (e) => {
  mouseTarget.set((e.clientX / innerWidth) * 2 - 1, -(e.clientY / innerHeight) * 2 + 1);
});

let lastScrollY = window.scrollY;
let scrollVelocity = 0;

function tick() {
  // mouse inertia: ease 10% of the remaining distance to target each frame
  mouseSmoothed.lerp(mouseTarget, 0.1);
  material.uniforms.uMouse.value.copy(mouseSmoothed);

  // scroll velocity: delta since last frame, decayed so it settles back to 0 when scrolling stops
  const currentScrollY = window.scrollY;
  const rawVelocity = currentScrollY - lastScrollY;
  lastScrollY = currentScrollY;
  scrollVelocity = THREE.MathUtils.lerp(scrollVelocity, rawVelocity, 0.15);
  material.uniforms.uVelocity.value = scrollVelocity;

  requestAnimationFrame(tick);
}
```
This is exactly the "displacement decay" pattern from §4.2's GPGPU RGB-shift shader, generalized: raw input → target, target → smoothed value via `lerp`, smoothed value → uniform.

---

## 6. Asset pipeline

### 6.1 glTF/GLB

`.glb` (single binary file, embeds geometry+textures+animations) is the standard web-delivery format — always prefer it over loose `.gltf`+`.bin`+textures for production (one HTTP request instead of N, no relative-path breakage). Three.js's `GLTFLoader` is the standard import path and correctly sets texture `colorSpace` for you automatically.

### 6.2 Draco vs Meshopt vs KTX2/Basis — three different problems

| | Compresses | Mechanism | Decode cost | Use when |
|---|---|---|---|---|
| **Draco** | Geometry (vertices/indices) only | Entropy coding + quantization, CPU decode via WASM | Higher — WASM decoder is ~400KB–1MB to ship, decode itself is CPU-heavy, can be a mobile bottleneck | Very high compression ratio needed on large/complex static meshes (scanned/CAD-heavy models); ratio matters more than decode speed |
| **Meshopt** | Geometry + animation | Byte-level filters tuned for GPU-friendly re-ordering, decode via a small WASM | Much lighter (~16KB decoder), faster decode | The general-purpose default in 2026 for most real-time web scenes — good ratio, cheap decode, plays nicely with quantization |
| **KTX2 / Basis Universal** | **Textures**, not geometry | GPU-native compressed formats (transcoded at load time to ASTC/ETC2/BC7 depending on device) via `KTX2Loader` | Transcode is fast; the big win is texture data **stays compressed in VRAM** (unlike JPEG/PNG, which decode to full RGBA in GPU memory) | Always, for texture-heavy scenes — this is usually the single highest-leverage optimization available, bigger than geometry compression for most landing-page scenes |

KTX2 has two encode modes: **UASTC** (higher quality/larger, use for normal maps or hero close-up textures) and **ETC1S/basis** (smaller/more lossy, use for simple diffuse/background textures).

### 6.3 `gltf-transform` CLI

`@gltf-transform/cli` — **4.5.0**, published 2026-09-01, actively maintained. The standard command-line optimization pipeline:
```bash
npm install -g @gltf-transform/cli

# one-shot "optimize everything" pass:
gltf-transform optimize input.glb output.glb --compress draco --texture-compress webp
# or, targeting KTX2 GPU textures instead of WebP:
gltf-transform optimize input.glb output.glb --compress meshopt --texture-compress ktx2

# targeted individual commands:
gltf-transform draco input.glb output.glb --method edgebreaker   # geometry compression
gltf-transform meshopt input.glb output.glb --level medium       # alternative geometry compression
gltf-transform uastc input.glb output.glb --level 4 --rdo --rdo-lambda 4 --zstd 18  # KTX2/UASTC textures
gltf-transform resize input.glb output.glb --width 1024 --height 1024  # cap texture dimensions
gltf-transform prune input.glb output.glb    # strip unreferenced accessors/materials/nodes
gltf-transform dedup input.glb output.glb    # merge duplicate accessors/textures
gltf-transform inspect input.glb             # print a full report: verts, tris, textures, sizes
```
Always run `inspect` before and after to confirm the optimization actually helped and didn't silently blow up a texture you expected to shrink.
Source: [gltf-transform.dev/cli](https://gltf-transform.dev/cli), [github.com/donmccurdy/glTF-Transform](https://github.com/donmccurdy/glTF-Transform).

### 6.4 `gltfjsx` CLI

`gltfjsx` — **6.5.3** (last publish Nov 2024; stable/feature-complete, not abandoned). Converts a `.glb` into a typed, re-usable React Three Fiber JSX component:
```bash
npx gltfjsx model.glb --types --transform
# --types / -t     : emit TypeScript
# --transform       : ALSO run a draco+resize(1024)+dedupe+prune pass on the source glb (via gltf-transform under the hood)
# --meta / -m       : keep original node names/metadata as userData
# --shadows / -s    : meshes cast/receive shadows
# --draco / -d       : point at a specific Draco decoder path
# --precision / -p   : numeric precision in generated JSX (default 2)
```
The generated component still expects the `.glb` in `/public` — `gltfjsx` only generates the JSX wrapper (`<mesh geometry={nodes.Cube.geometry} material={materials.Metal} />` style), it doesn't inline geometry data.
Source: [github.com/pmndrs/gltfjsx](https://github.com/pmndrs/gltfjsx).

### 6.5 Polygon/texture budgets for the web (practitioner guidance)

- **Hero model total**: aim under ~5MB `.glb` after Draco/Meshopt+KTX2, ideally 1–2MB for above-the-fold content that gates LCP-adjacent rendering.
- **Triangle count**: tens of thousands, not millions — a close-up hero object rarely needs more than 50–150k triangles; anything denser, bake normal maps from a high-poly sculpt instead of shipping the dense mesh.
- **Textures**: 1K–2K max per map for most hero objects; reserve 4K for a single extreme-close-up hero material. KTX2 at 2K frequently beats PNG at 1K in both file size and VRAM footprint.
- **Draw calls**: budget in the low hundreds total for the page, not just the 3D — merge/instance aggressively (§1.9).

### 6.6 HDRI / environment maps

Poly Haven / most sources ship `.hdr` (Radiance RGBE) or `.exr` (OpenEXR) equirectangular panoramas — load with `RGBELoader`/`EXRLoader`, then **prefilter** before using as an environment map:
```js
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

const pmremGenerator = new THREE.PMREMGenerator(renderer);
pmremGenerator.compileEquirectangularShader();

new RGBELoader().load('/studio.hdr', (hdrTexture) => {
  const envMap = pmremGenerator.fromEquirectangular(hdrTexture).texture;
  scene.environment = envMap;   // lights all PBR materials
  scene.background = envMap;    // optional — use as visible background too
  hdrTexture.dispose();
  pmremGenerator.dispose();
});
```
Skipping the PMREM prefilter step (using the raw equirect texture directly as `scene.environment`) gives visibly wrong, non-roughness-aware specular reflections — this step is not optional for correct-looking PBR. drei's `<Environment preset="..."/>` / `<Environment files="..."/>` does this automatically and also offers a curated set of pmndrs-hosted prefiltered presets so you don't need to self-host an HDRI for common looks ("city", "sunset", "studio", "warehouse", etc.).

### 6.7 Baking

For scenes with a mostly-static camera and static lighting (the overwhelmingly common landing-page case), **bake** ambient occlusion/global illumination in Blender/other DCC tools into a lightmap or `aoMap`/`lightMap` texture channel rather than computing real-time shadows/GI. Dramatically cheaper at runtime, and looks *better* than real-time approximations for a fixed hero shot since you can spend offline render budget you'd never afford live.

### 6.8 When to use video or an image sequence instead of 3D

If the "3D" requirement is actually **one fixed camera angle with a simple, repeatable motion** (product slowly rotating, liquid swirling, particles drifting) and there's no user interactivity beyond maybe scroll-scrubbing, a **pre-rendered video loop (WebM/H.265) or a scroll-scrubbed image sequence** (the Apple product-page technique — decode frames to a `<canvas>` 2D context keyed to scroll position) is very often the *correct* engineering call, not a compromise:
- Zero WebGL context/shader-compile cost, zero risk of a `webglcontextlost` failure mode.
- Trivial, predictable LCP — it's just an image/video element.
- No GPU thermal/battery cost, no low-end-device fallback to design for.
- The "3D" can be rendered offline at unlimited quality/complexity (real path-traced fluid sim, film-quality lighting) — no real-time budget constraint at all.

Reach for real-time WebGL only when the scene genuinely needs to respond to input (mouse, scroll *direction* and not just progress, viewport size in a way frame-scrubbing can't cover, or live data).

---

## 7. Performance & fallbacks

### 7.1 Frame budget

60fps = 16.6ms/frame; a growing share of 2026 devices are 120Hz (ProMotion iPhones/iPads, most flagship Android) = 8.3ms/frame. Leave headroom for browser compositing and other tabs — target **~10ms** of actual JS+GPU work per frame as a practical ceiling, not the full 16.6ms.

### 7.2 Draw calls & overdraw

- Audit with `renderer.info.render.calls` (and `.triangles`, `.points`) — log it during development, budget low hundreds of draw calls total for the page.
- Merge static geometry (`BufferGeometryUtils.mergeGeometries`) or instance repeated meshes (`InstancedMesh`, §1.9) rather than adding N separate `Mesh` objects.
- **Overdraw** (many transparent/blended layers stacked, e.g. particle systems or multi-pass postprocessing chains) is disproportionately expensive on **mobile tile-based GPUs** — keep transparent object counts low and avoid chaining multiple full-screen blur/bloom passes on mobile specifically.

### 7.3 `powerPreference`

```js
new THREE.WebGLRenderer({ powerPreference: 'high-performance' }); // requests discrete GPU on dual-GPU laptops
```
`'high-performance'` trades battery for consistent frame times (use for a flagship hero scene); `'low-power'` explicitly asks for the integrated GPU (use for a background-only decorative effect where battery matters more than peak fidelity); `'default'` lets the user agent decide.

### 7.4 DPR strategy

Clamp, don't trust `window.devicePixelRatio` raw (§1.6): `Math.min(devicePixelRatio, 2)` as the ceiling, and be willing to drop to `1` under sustained load (§7.5). This single line is worth more to mobile framerate than most shader micro-optimizations.

### 7.5 `PerformanceMonitor`-style adaptive quality

drei's `<PerformanceMonitor>` watches rolling FPS and calls back when a device is struggling or has headroom, so quality adapts to the actual device instead of UA-sniffing:
```jsx
import { PerformanceMonitor, AdaptiveDpr } from '@react-three/drei';

function Scene() {
  const [dpr, setDpr] = useState(1.5);
  return (
    <>
      <PerformanceMonitor
        onIncline={() => setDpr(2)}
        onDecline={() => setDpr(1)}
        flipflops={3}                 // after 3 oscillations, stop adjusting — settle on a safe value
        onFallback={() => setDpr(1)}  // sustained bad perf — drop hard
      />
      <AdaptiveDpr pixelated />
      {/* ...also good decline targets: disable postprocessing, drop shadow map size, cut particle count */}
    </>
  );
}
```
Vanilla-three equivalent: track a rolling average of `performance.now()` deltas yourself and step down DPR/effect quality manually using the same thresholds.

### 7.6 Lazy-init on intersection

Don't construct the renderer or start the render loop for a below-the-fold WebGL section until it's about to be visible — this is a major, easy LCP/TBT win:
```js
const observer = new IntersectionObserver(([entry]) => {
  if (entry.isIntersecting && !initialized) initWebGLScene();
  else if (!entry.isIntersecting && initialized) renderer.setAnimationLoop(null); // pause, don't dispose
}, { rootMargin: '200px' });
observer.observe(canvasContainer);
```
Also pause on `document.visibilitychange` (`document.hidden`) — a backgrounded tab still burning GPU cycles is a common mobile battery-drain/thermal complaint.

### 7.7 OffscreenCanvas + worker

Moves the render loop off the main thread so scroll/input stay buttery even under heavy shader load:
```js
// main thread:
const offscreen = canvas.transferControlToOffscreen();
worker.postMessage({ canvas: offscreen, width: innerWidth, height: innerHeight }, [offscreen]);
// worker thread: construct THREE.WebGLRenderer({ canvas: offscreen }) — same API from here
```
Caveat: `OffscreenCanvas` doesn't natively receive DOM pointer/resize events — you must forward them manually via `postMessage`. **`@react-three/offscreen` (the R3F wrapper for this) has not published since 2023 (v0.0.8)** — treat it as experimental/unmaintained; for a 2026 project needing this, implement the vanilla three.js manual's `OffscreenCanvas` pattern directly rather than depending on that package.
Source: [threejs.org/manual — OffscreenCanvas](https://threejs.org/manual/en/offscreencanvas.html), [Evil Martians — Faster WebGL/Three.js with OffscreenCanvas and Web Workers](https://evilmartians.com/chronicles/faster-webgl-three-js-3d-graphics-with-offscreencanvas-and-web-workers).

### 7.8 Mobile behaviour, battery, thermal

- **iOS Safari** aggressively reclaims WebGL contexts under memory pressure or backgrounding — always handle `webglcontextlost`/`webglcontextrestored` (re-upload textures/geometry on restore, or degrade gracefully) rather than assuming the context is permanent.
- Texture VRAM ceilings are far lower than desktop — this is where KTX2 (§6.2) pays off most.
- **Thermal throttling** is real and fast: sustained 100% GPU load on a fanless phone can trigger clock throttling within roughly a minute or two of continuous heavy shader work — favor cheaper per-pixel shaders and smaller render-target resolutions specifically on mobile UA, not just a smaller DPR.
- There is **no reliable Battery Status API** left in mainstream browsers (Chrome/Firefox removed `navigator.getBattery()` years ago over fingerprinting concerns) — don't design a fallback strategy around detecting battery level; use measured frame-time/FPS (§7.5) as your only reliable adaptive-quality signal.

### 7.9 `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) { /* ... */ }
```
```js
const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```
For a WebGL hero: honor this by rendering exactly one static frame and calling `renderer.setAnimationLoop(null)` (stop the loop entirely) rather than merely slowing animations down — also disable camera autoplay/orbit and any scroll-parallax camera movement, since vestibular-motion triggers are about *movement*, not speed. Listen for `change` on the `MediaQueryList` too — a user can toggle this mid-session.

### 7.10 No-WebGL fallback

Always feature-detect before committing to a WebGL-dependent layout:
```js
function hasWebGL() {
  try {
    const canvas = document.createElement('canvas');
    return !!(window.WebGLRenderingContext &&
      (canvas.getContext('webgl2') || canvas.getContext('webgl')));
  } catch (e) { return false; }
}
```
If `false` (or a WebGL init throws — always wrap renderer creation in try/catch too), swap to a static poster image or CSS-only gradient/animation — **never leave a blank canvas**. This is a real, non-trivial slice of traffic: locked-down corporate machines, some in-app browsers/webviews, and older/budget Android devices with software rendering all still show up in analytics in 2026.

### 7.11 LCP impact and how agencies hide the cost

A `<canvas>` element is **never itself an LCP candidate** — browsers only count image/text elements for Largest Contentful Paint. This is why the standard agency pattern is:
1. A real `<img>` (or text block) is the visible, LCP-counted element on first paint — often literally a poster/first-frame screenshot of the WebGL scene.
2. WebGL initialization is deferred until after `load` (or `requestIdleCallback`, or the intersection-observer gate from §7.6) so it never competes with LCP-critical resources for bandwidth/main-thread time.
3. Once the first real WebGL frame is ready, crossfade the canvas in over the poster image.
4. Many agency sites additionally show a **full-page preloader** (progress bar/percentage) that holds the reveal until *all* heavy assets (textures, `.glb`, HDRI) are warm — this is a deliberate, debated trade-off: it actively **hurts** raw performance metrics (delays FCP/TTI/user interaction) purely for perceived-polish reasons (no visible pop-in/jank during the "hero reveal" moment). Treat it as a design decision to make consciously, not a performance best practice — plenty of award-winning sites skip the preloader entirely and just crossfade opportunistically.

---

## 8. Free asset sources

| Source | What | License | Notes |
|---|---|---|---|
| **[Poly Haven](https://polyhaven.com/)** | HDRIs, PBR textures, models | **CC0** | Has a public JSON API (`/our-api`) for programmatic access — good for build-time asset pipelines; the de facto default HDRI source, what drei's `Environment` presets are often built from |
| **[ambientCG](https://ambientcg.com/)** | PBR materials, HDRIs, models, decals, substances | **CC0** | 1M+ monthly downloads; founded 2018 specifically to guarantee no-attribution-required licensing unlike many texture sites |
| **[Sketchfab](https://sketchfab.com/)** | Huge scanned/user-uploaded model library | **Mixed** — filter explicitly for CC0/downloadable | Best source for scanned real-world objects; always double-check the specific model's license, not just "free to view" |
| **[Quaternius](https://quaternius.com/)** | Stylized low-poly game-ready packs (characters, environments, vehicles, props) | Generally free/CC0-style (verify per pack) | Great for playful/low-poly landing-page 3D, not photoreal |
| **[Kenney](https://kenney.nl/assets)** | 2D/3D/audio/UI asset packs | **CC0** | Enormous, extremely popular for prototyping; "All-in-1" bundle covers most needs at once |
| **[Pmndrs market](https://market.pmnd.rs/)** | Curated models/HDRIs packaged specifically for the R3F/drei ecosystem | Mixed free/paid | Assets are pre-optimized for real-time web use, not just "a glb that happens to exist" |
| **MSDF font tooling** | `msdfgen` (Chlumsky, the reference C++ generator) / `msdf-bmfont-xml` (JS wrapper, generates atlas + JSON) | — | Needed if you're hand-rolling SDF text outside `troika-three-text` (which generates its own atlases on the fly, no separate tool needed) |
| **Fonts** | Google Fonts (source TTFs for MSDF generation), [Fontsource](https://fontsource.org/) (self-hostable npm-packaged fonts) | Varies (mostly OFL) | |

---

## 9. No-build vanilla three.js starter

A complete, single-file, no-bundler starting point demonstrating: importmap CDN loading, correct color management, a DOM-synced plane (the §4.20 pattern — a real `<h1>`/`<img>` proxy tracked by a WebGL mesh), scroll-velocity + mouse uniforms, `prefers-reduced-motion` handling, and a no-WebGL fallback. Save as `index.html` and open directly (no dev server required — CDN-hosted modules work over `file://`... though a local static server is still recommended for CORS-safety with textures if you add any).

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>WebGL DOM-Synced Starter</title>
  <style>
    html, body { margin: 0; padding: 0; background: #0b0b0f; color: #f2f2f5; }
    body { font-family: -apple-system, system-ui, sans-serif; }

    #gl-canvas {
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      z-index: 0;
      pointer-events: none;
    }

    main {
      position: relative;
      z-index: 1;
      min-height: 220vh; /* fake scroll length */
    }

    .hero {
      height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 0 24px;
    }

    /* The DOM proxy element — WebGL tracks this rect exactly.
       Kept visually transparent but fully present in layout/a11y tree. */
    #hero-title {
      font-size: clamp(2.5rem, 8vw, 6rem);
      font-weight: 700;
      letter-spacing: -0.02em;
      margin: 0;
      color: transparent; /* text painted by WebGL layer instead of CSS */
      -webkit-text-stroke: 1px rgba(242,242,245,0.15); /* faint fallback outline if WebGL fails/is disabled */
    }

    .hero p {
      max-width: 46ch;
      opacity: 0.7;
      margin-top: 1rem;
    }

    /* No-WebGL / reduced-motion fallback: show a plain readable heading instead */
    .no-webgl #hero-title,
    .reduced-motion #hero-title {
      color: #f2f2f5;
      -webkit-text-stroke: 0;
    }
    .no-webgl #gl-canvas,
    .reduced-motion #gl-canvas {
      display: none;
    }

    section {
      max-width: 640px;
      margin: 0 auto;
      padding: 4rem 24px;
      opacity: 0.85;
      line-height: 1.6;
    }
  </style>
</head>
<body>
  <canvas id="gl-canvas"></canvas>

  <main>
    <div class="hero">
      <h1 id="hero-title">Depth, synced to scroll</h1>
      <p>A WebGL plane tracks this heading's exact screen position every frame — resize, reflow, and scroll all stay pixel-perfect, and the real heading underneath stays selectable, crawlable, and accessible.</p>
    </div>
    <section>
      <p>Scroll to drive the plane's velocity-reactive distortion. Everything above the canvas is real, unstyled-by-WebGL DOM — this page works with JavaScript disabled, WebGL unavailable, or reduced motion requested.</p>
    </section>
  </main>

  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.186.0/build/three.module.js"
    }
  }
  </script>
  <script type="module">
    import * as THREE from 'three';

    // ---------- 1. Feature detection & reduced-motion gate ----------
    function hasWebGL() {
      try {
        const c = document.createElement('canvas');
        return !!(window.WebGLRenderingContext &&
          (c.getContext('webgl2') || c.getContext('webgl')));
      } catch (e) { return false; }
    }

    const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

    if (!hasWebGL()) {
      document.documentElement.classList.add('no-webgl');
      // Bail out completely — the DOM fallback in CSS above already reads fine on its own.
    } else if (prefersReducedMotion) {
      document.documentElement.classList.add('reduced-motion');
      // Bail out — respect the user's OS-level preference, no WebGL animation at all.
    } else {
      initScene();
    }

    function initScene() {
      const canvas = document.getElementById('gl-canvas');
      const proxyEl = document.getElementById('hero-title');

      // ---------- 2. Renderer with correct color management ----------
      let renderer;
      try {
        renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
      } catch (e) {
        document.documentElement.classList.add('no-webgl');
        return;
      }
      renderer.outputColorSpace = THREE.SRGBColorSpace;
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      renderer.toneMappingExposure = 1.0;
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // DPR clamp

      canvas.addEventListener('webglcontextlost', (e) => e.preventDefault());
      canvas.addEventListener('webglcontextrestored', () => location.reload());

      const scene = new THREE.Scene();

      // Perspective camera whose FOV is recomputed on resize so that
      // 1 three.js unit == 1 CSS pixel at z=0. This is what makes DOM
      // rect -> mesh position/scale a direct, no-conversion mapping.
      const camera = new THREE.PerspectiveCamera(50, 1, 0.1, 2000);
      const CAMERA_DISTANCE = 600;

      function syncCameraToViewport() {
        const w = window.innerWidth;
        const h = window.innerHeight;
        camera.fov = 2 * Math.atan(h / 2 / CAMERA_DISTANCE) * (180 / Math.PI);
        camera.aspect = w / h;
        camera.position.z = CAMERA_DISTANCE;
        camera.updateProjectionMatrix();
        renderer.setSize(w, h);
      }

      // ---------- 3. The DOM-synced plane ----------
      const uniforms = {
        uTime:       { value: 0 },
        uMouse:      { value: new THREE.Vector2(0, 0) },
        uVelocity:   { value: 0 },
        uProgress:   { value: 0 }, // page scroll progress 0..1
        uResolution: { value: new THREE.Vector2(1, 1) },
      };

      const geometry = new THREE.PlaneGeometry(1, 1, 32, 32);
      const material = new THREE.ShaderMaterial({
        uniforms,
        transparent: true,
        vertexShader: /* glsl */ `
          uniform float uTime;
          uniform float uVelocity;
          varying vec2 vUv;
          void main() {
            vUv = uv;
            vec3 pos = position;
            // subtle velocity-reactive wave — the plane ripples while you scroll,
            // and settles flat when scrolling stops (uVelocity decays toward 0).
            float wave = sin(pos.x * 6.0 + uTime * 1.5) * 0.5
                       + sin(pos.y * 4.0 - uTime * 1.2) * 0.5;
            pos.z += wave * (4.0 + abs(uVelocity) * 40.0);
            gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
          }
        `,
        fragmentShader: /* glsl */ `
          precision highp float;
          uniform float uTime;
          uniform vec2 uMouse;
          uniform float uVelocity;
          uniform float uProgress;
          varying vec2 vUv;

          // cheap RGB-shift, magnitude driven by scroll velocity (see cheat-sheet 4.2 / 5.10)
          float rim(vec2 uv) {
            vec2 c = uv - 0.5;
            return 1.0 - smoothstep(0.0, 0.75, length(c));
          }

          void main() {
            float shift = clamp(abs(uVelocity) * 0.02, 0.0, 0.02);
            float r = rim(vUv + vec2(shift, 0.0));
            float g = rim(vUv);
            float b = rim(vUv - vec2(shift, 0.0));

            vec3 base = mix(vec3(0.55, 0.65, 1.0), vec3(1.0, 0.55, 0.75), uProgress);
            vec3 color = base * vec3(r, g, b) * 1.4;

            float alpha = g * 0.9; // fades at plane edges instead of a hard rectangle
            gl_FragColor = vec4(color, alpha);
          }
        `,
      });

      const plane = new THREE.Mesh(geometry, material);
      scene.add(plane);

      // ---------- 4. Sync loop: DOM rect -> mesh transform ----------
      function syncPlaneToProxy() {
        const rect = proxyEl.getBoundingClientRect();
        const w = window.innerWidth;
        const h = window.innerHeight;

        plane.scale.set(Math.max(rect.width, 1), Math.max(rect.height, 1), 1);
        plane.position.set(
          rect.left - w / 2 + rect.width / 2,
          -(rect.top - h / 2 + rect.height / 2),
          0
        );
      }

      // ---------- 5. Scroll velocity + mouse inertia uniforms ----------
      const mouseTarget = new THREE.Vector2();
      const mouseSmoothed = new THREE.Vector2();
      window.addEventListener('pointermove', (e) => {
        mouseTarget.set((e.clientX / innerWidth) * 2 - 1, -(e.clientY / innerHeight) * 2 + 1);
      }, { passive: true });

      let lastScrollY = window.scrollY;
      let scrollVelocity = 0;

      function scrollProgress() {
        const max = document.documentElement.scrollHeight - window.innerHeight;
        return max > 0 ? window.scrollY / max : 0;
      }

      // ---------- 6. Resize handling ----------
      function onResize() {
        syncCameraToViewport();
        uniforms.uResolution.value.set(window.innerWidth, window.innerHeight);
        syncPlaneToProxy();
      }
      window.addEventListener('resize', onResize);
      onResize();

      // ---------- 7. Lazy-pause when tab hidden (battery/thermal, §7.8) ----------
      document.addEventListener('visibilitychange', () => {
        if (document.hidden) renderer.setAnimationLoop(null);
        else renderer.setAnimationLoop(tick);
      });

      // ---------- 8. Render loop ----------
      const clock = new THREE.Clock();
      function tick() {
        const elapsed = clock.getElapsedTime();
        uniforms.uTime.value = elapsed;

        mouseSmoothed.lerp(mouseTarget, 0.1);
        uniforms.uMouse.value.copy(mouseSmoothed);

        const currentScrollY = window.scrollY;
        const rawVelocity = currentScrollY - lastScrollY;
        lastScrollY = currentScrollY;
        scrollVelocity = THREE.MathUtils.lerp(scrollVelocity, rawVelocity, 0.15);
        uniforms.uVelocity.value = scrollVelocity;
        uniforms.uProgress.value = scrollProgress();

        syncPlaneToProxy();
        renderer.render(scene, camera);
      }
      renderer.setAnimationLoop(tick);

      // React to a live OS-level reduced-motion toggle mid-session too:
      window.matchMedia('(prefers-reduced-motion: reduce)').addEventListener('change', (e) => {
        if (e.matches) {
          renderer.setAnimationLoop(null);
          document.documentElement.classList.add('reduced-motion');
        }
      });
    }
  </script>
</body>
</html>
```

Ties together §1.2 (importmap), §1.5 (color management), §1.6/§7.4 (DPR), §4.20 (DOM-synced plane), §5.10 (velocity/mouse uniforms), §7.9/§7.10 (reduced-motion + no-WebGL fallback), and §7.8 (context-loss handling) in one file. To sync a whole page, replace the single proxy element with a `querySelectorAll('[data-webgl-track]')` loop and one mesh per match.
