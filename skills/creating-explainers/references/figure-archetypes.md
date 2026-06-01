# Figure Archetypes

Most interactive figures fall into one of five patterns. Pick the closest to your figure's purpose and adapt - don't reinvent. Each archetype below includes a minimal working snippet you can paste into the article's script block (after `initCanvas`) and modify.

All archetypes share the same scaffolding:

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-myfig');
  // state ...
  function draw() { /* render */ }
  // event listeners ...
  draw();
})();
```

State lives in closure variables. There are no globals beyond `initCanvas` and `dpr`.

---

## Archetype 1: Stepped Simulation

**When:** A discrete process advances one tick at a time - physics steps, agent rollouts, training rounds, search expansions. The reader presses Step (or Play for autoplay).

**Examples in the project:** `world-models/` (gravity simulation, Figure 1), `dspy/` (optimizer simulator), `pi-agent/` (agent rollout).

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-grav');
  let state = {
    step: 0,
    ball: { x: 100, y: 50, vy: 0 },
    playing: false,
    playTimer: null,
  };

  const G = 9.8;
  const DT = 0.1;
  const FLOOR_Y = h - 40;

  function advance() {
    if (state.ball.y >= FLOOR_Y) return; // settled
    state.ball.vy += G * DT;
    state.ball.y += state.ball.vy;
    if (state.ball.y > FLOOR_Y) {
      state.ball.y = FLOOR_Y;
      state.ball.vy = 0;
    }
    state.step += 1;
    draw();
  }

  function reset() {
    state = { step: 0, ball: { x: 100, y: 50, vy: 0 }, playing: false, playTimer: null };
    document.getElementById('grav-play').textContent = 'Play';
    draw();
  }

  function togglePlay() {
    const btn = document.getElementById('grav-play');
    if (state.playing) {
      clearInterval(state.playTimer);
      state.playing = false;
      btn.textContent = 'Play';
    } else {
      state.playTimer = setInterval(advance, 250);
      state.playing = true;
      btn.textContent = 'Pause';
    }
  }

  function draw() {
    ctx.clearRect(0, 0, w, h);
    // floor
    ctx.strokeStyle = 'rgba(0,0,0,0.2)';
    ctx.beginPath();
    ctx.moveTo(0, FLOOR_Y);
    ctx.lineTo(w, FLOOR_Y);
    ctx.stroke();
    // ball
    ctx.fillStyle = '#2563EB';
    ctx.beginPath();
    ctx.arc(state.ball.x, state.ball.y, 12, 0, Math.PI * 2);
    ctx.fill();
    // readout
    document.getElementById('grav-info').textContent =
      'Step: ' + state.step + '   y: ' + state.ball.y.toFixed(1);
  }

  document.getElementById('grav-step').addEventListener('click', advance);
  document.getElementById('grav-play').addEventListener('click', togglePlay);
  document.getElementById('grav-reset').addEventListener('click', reset);
  draw();
})();
```

**Tuning notes:**
- `setInterval` cadence around 250-600ms feels good for human-paced stepping. Faster than 200ms feels jittery; slower than 800ms feels slow.
- Auto-stop Play when the simulation reaches a terminal state. Otherwise the button gets confusing.
- Always render once on init so the figure isn't blank before the first click.

---

## Archetype 2: Continuous Animation

**When:** Something flows or pulses continuously - particles moving along a pipeline, a node breathing, a chart drawing in. No "step" semantics; the figure animates on its own.

**Examples:** Landing-page thumbnails (`index.html`), `pi-agent/` (animated tree, Figure 4).

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-flow');
  let time = 0;
  const particles = Array.from({ length: 20 }, (_, i) => ({
    offset: i / 20, // 0..1 along path
  }));

  function draw() {
    ctx.clearRect(0, 0, w, h);
    // pipe
    ctx.strokeStyle = 'rgba(0,0,0,0.1)';
    ctx.lineWidth = 24;
    ctx.beginPath();
    ctx.moveTo(40, h / 2);
    ctx.lineTo(w - 40, h / 2);
    ctx.stroke();
    // particles
    ctx.fillStyle = '#2563EB';
    for (const p of particles) {
      const t = (p.offset + time * 0.2) % 1;
      const x = 40 + t * (w - 80);
      const y = h / 2;
      ctx.beginPath();
      ctx.arc(x, y, 5, 0, Math.PI * 2);
      ctx.fill();
    }
  }

  function loop() {
    time += 0.016; // ~60fps delta
    draw();
    requestAnimationFrame(loop);
  }

  loop();
})();
```

**Tuning notes:**
- Use `time += 0.016` (a fixed delta) rather than wall-clock - simpler and stable across frame rates.
- `requestAnimationFrame` pauses when the tab is hidden, which is what you want.
- For pulsing values, `Math.sin(time * 2) * 0.5 + 0.5` gives 0..1 oscillation. Adjust the multiplier for speed.
- These figures usually have no controls. If they do, it's a Pause/Play and a Reset.

---

## Archetype 3: Tabbed Comparison

**When:** The figure shows several discrete variants the reader can switch between - different signature shapes, different optimizer algorithms, different model sizes. Reader clicks a tab and the canvas re-renders for that variant.

**Examples:** `dspy/` (signature expansion - 4 tabs).

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-sig');
  const variants = [
    { name: 'Basic QA', inline: 'question -> answer', /* ... */ },
    { name: 'RAG',       inline: 'context, question -> answer', /* ... */ },
    { name: 'CoT',       inline: 'question -> reasoning, answer', /* ... */ },
  ];
  let current = 0;

  function draw() {
    ctx.clearRect(0, 0, w, h);
    const v = variants[current];
    // render v ...
    ctx.font = '20px serif';
    ctx.fillStyle = '#000';
    ctx.fillText(v.name + ': ' + v.inline, 40, 40);
  }

  // Tab buttons live in the figure's controls bar
  const tabs = document.querySelectorAll('#sig-tabs .ctrl-btn');
  tabs.forEach((btn, i) => {
    btn.addEventListener('click', () => {
      current = i;
      tabs.forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      draw();
    });
  });
  tabs[0].classList.add('active');
  draw();
})();
```

**HTML for the tabs:**
```html
<div class="figure-controls" id="sig-tabs">
  <button class="ctrl-btn">Basic QA</button>
  <button class="ctrl-btn">RAG</button>
  <button class="ctrl-btn">CoT</button>
</div>
```

**Tuning notes:**
- Keep variants to 3-5. More than that and the tab bar wraps awkwardly.
- The currently-selected tab gets the `.active` class (dark background).
- If variants pulse or animate, run a `requestAnimationFrame` loop and use `current` to pick which variant to render each frame.

---

## Archetype 4: 3D Rotation

**When:** The figure is inherently spatial - a rotating model of a system, an architecture diagram with depth, a 3D scene the reader explores by dragging. Hand-coded projection, not a 3D library.

**Examples:** `world-models/` (Figure 5 - 3D rotating world), `index.html` thumbnails for World Models.

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-3d');

  let angleY = 0.5, angleX = 0.3;
  let zoom = 1.0;
  let dragging = false;
  let lastMouse = { x: 0, y: 0 };

  // Project a (x,y,z) point to 2D screen space.
  // - Rotate around Y first (yaw), then X (pitch)
  // - Apply weak perspective: nearer points are bigger
  function project(x, y, z) {
    // Rotate Y (yaw)
    const cy = Math.cos(angleY), sy = Math.sin(angleY);
    let rx = x * cy - z * sy;
    let rz = x * sy + z * cy;
    let ry = y;
    // Rotate X (pitch)
    const cx = Math.cos(angleX), sx = Math.sin(angleX);
    const ry2 = ry * cx - rz * sx;
    const rz2 = ry * sx + rz * cx;
    ry = ry2; rz = rz2;
    // Perspective
    const s = 80 * zoom;
    const p = 4 / (4 + rz);
    return { x: w / 2 + rx * s * p, y: h / 2 + ry * s * p, z: rz };
  }

  // Draw a unit cube centered at origin
  const verts = [
    [-1,-1,-1],[1,-1,-1],[1,1,-1],[-1,1,-1],
    [-1,-1, 1],[1,-1, 1],[1,1, 1],[-1,1, 1]
  ];
  // Each face is 4 vertex indices, in CCW order when viewed from outside
  const faces = [
    { idx: [0,1,2,3], color: '#2563EB' }, // back
    { idx: [5,4,7,6], color: '#1D4ED8' }, // front
    { idx: [4,0,3,7], color: '#3B82F6' }, // left
    { idx: [1,5,6,2], color: '#60A5FA' }, // right
    { idx: [4,5,1,0], color: '#93C5FD' }, // bottom
    { idx: [3,2,6,7], color: '#1E3A8A' }, // top
  ];

  function draw() {
    ctx.clearRect(0, 0, w, h);
    // Project all faces and sort back-to-front (painter's algorithm)
    const projected = faces.map(f => {
      const pts = f.idx.map(i => project(verts[i][0], verts[i][1], verts[i][2]));
      const avgZ = pts.reduce((s, p) => s + p.z, 0) / pts.length;
      return { pts, color: f.color, z: avgZ };
    }).sort((a, b) => b.z - a.z);

    for (const face of projected) {
      ctx.fillStyle = face.color;
      ctx.strokeStyle = 'rgba(0,0,0,0.4)';
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.moveTo(face.pts[0].x, face.pts[0].y);
      for (let i = 1; i < face.pts.length; i++) ctx.lineTo(face.pts[i].x, face.pts[i].y);
      ctx.closePath();
      ctx.fill();
      ctx.stroke();
    }
  }

  function clamp(v, lo, hi) { return Math.max(lo, Math.min(hi, v)); }

  function getPos(e) {
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  canvas.style.cursor = 'grab';
  canvas.addEventListener('mousedown', e => {
    dragging = true; lastMouse = getPos(e); canvas.style.cursor = 'grabbing';
  });
  canvas.addEventListener('mousemove', e => {
    if (!dragging) return;
    const m = getPos(e);
    angleY += (m.x - lastMouse.x) * 0.008;
    angleX += (m.y - lastMouse.y) * 0.008;
    angleX = clamp(angleX, -1.2, 1.2);
    lastMouse = m;
    draw();
  });
  canvas.addEventListener('mouseup', () => { dragging = false; canvas.style.cursor = 'grab'; });
  canvas.addEventListener('mouseleave', () => { dragging = false; });
  canvas.addEventListener('wheel', e => {
    e.preventDefault();
    zoom = clamp(zoom - e.deltaY * 0.001, 0.5, 2.5);
    draw();
  }, { passive: false });

  // Touch support - single finger drag, two-finger pinch zoom
  let lastTouchDist = 0;
  canvas.addEventListener('touchstart', e => {
    e.preventDefault();
    if (e.touches.length === 1) {
      dragging = true;
      lastMouse = { x: e.touches[0].clientX, y: e.touches[0].clientY };
    } else if (e.touches.length === 2) {
      const dx = e.touches[0].clientX - e.touches[1].clientX;
      const dy = e.touches[0].clientY - e.touches[1].clientY;
      lastTouchDist = Math.hypot(dx, dy);
    }
  }, { passive: false });
  canvas.addEventListener('touchmove', e => {
    e.preventDefault();
    if (e.touches.length === 1 && dragging) {
      const mx = e.touches[0].clientX, my = e.touches[0].clientY;
      angleY += (mx - lastMouse.x) * 0.008;
      angleX += (my - lastMouse.y) * 0.008;
      angleX = clamp(angleX, -1.2, 1.2);
      lastMouse = { x: mx, y: my };
      draw();
    } else if (e.touches.length === 2) {
      const dx = e.touches[0].clientX - e.touches[1].clientX;
      const dy = e.touches[0].clientY - e.touches[1].clientY;
      const dist = Math.hypot(dx, dy);
      zoom = clamp(zoom + (dist - lastTouchDist) * 0.005, 0.5, 2.5);
      lastTouchDist = dist;
      draw();
    }
  }, { passive: false });
  canvas.addEventListener('touchend', () => { dragging = false; });

  draw();
})();
```

**Tuning notes:**
- `angleX` (pitch) is clamped so the camera can't flip upside-down. `[-1.2, 1.2]` is comfortable.
- Painter's algorithm (sort by avg z, draw back-to-front) is fine for convex shapes. For non-convex or interpenetrating geometry you'd need per-pixel depth - don't go there. Keep geometry simple.
- The perspective term `4 / (4 + rz)` is a weak perspective - tweak the `4` to make perspective stronger (smaller) or weaker (larger).
- Always set the cursor to `grab`/`grabbing` so the affordance is obvious.
- Always call `e.preventDefault()` on touch events - otherwise the browser tries to scroll the page.

---

## Archetype 5: Parameter Exploration

**When:** A small number of knobs (sliders, buttons, tabs) re-tune a model and the figure re-renders to show the effect. Good for "what does temperature do" or "how does N change the result".

**Examples:** `autoresearch/` (parameter sliders), `artemis-ii/` (mission timeline scrubber).

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-bell');
  let mu = 0, sigma = 1;

  function draw() {
    ctx.clearRect(0, 0, w, h);
    ctx.strokeStyle = '#2563EB';
    ctx.lineWidth = 2;
    ctx.beginPath();
    for (let px = 0; px < w; px++) {
      const x = (px - w / 2) / 50;
      const y = Math.exp(-((x - mu) ** 2) / (2 * sigma * sigma)) /
                (sigma * Math.sqrt(2 * Math.PI));
      const py = h - 40 - y * 200;
      if (px === 0) ctx.moveTo(px, py); else ctx.lineTo(px, py);
    }
    ctx.stroke();

    document.getElementById('bell-info').textContent =
      'mu=' + mu.toFixed(2) + '  sigma=' + sigma.toFixed(2);
  }

  document.getElementById('bell-mu').addEventListener('input', e => {
    mu = parseFloat(e.target.value);
    draw();
  });
  document.getElementById('bell-sigma').addEventListener('input', e => {
    sigma = parseFloat(e.target.value);
    draw();
  });
  draw();
})();
```

**HTML for the controls:**
```html
<div class="figure-controls">
  <span class="ctrl-label">mu</span>
  <input id="bell-mu" type="range" min="-3" max="3" step="0.1" value="0">
  <div class="ctrl-separator"></div>
  <span class="ctrl-label">sigma</span>
  <input id="bell-sigma" type="range" min="0.2" max="3" step="0.1" value="1">
  <div class="ctrl-separator"></div>
  <span class="ctrl-label" id="bell-info">mu=0  sigma=1</span>
</div>
```

**Tuning notes:**
- Use the `input` event (not `change`) for sliders - it fires continuously as the user drags, which is what makes the figure feel responsive.
- Always show the current numeric value in a `.ctrl-label` so the slider is self-documenting.
- Keep slider ranges tight enough that every value produces a visibly different result. If the curve barely changes across the whole range, you have the wrong range.

---

## Combining Archetypes

Real figures often combine two patterns. Common combinations:

- **Stepped + 3D**: a rollout you can advance one step at a time, while also dragging to rotate the view (`world-models/` Figure 5).
- **Continuous + Parameter**: a constantly-flowing animation whose parameters the reader can tweak (`autoresearch/`).
- **Stepped + Tabbed**: pick a strategy from tabs, then step through it (`pi-agent/`).

When combining, keep state in one object and have one `draw()` function that reads all of it. Don't try to compose archetypes - just read what each does and write a single integrated figure.

---

## Choosing colors for figure rendering

The CSS `var(--accent)` is your primary brand color. Use it for the dominant moving element (the ball, the active node, the highlighted curve).

Secondary elements use grays:
- `rgba(0, 0, 0, 0.2)` for medium-emphasis lines
- `rgba(0, 0, 0, 0.1)` for low-emphasis grid/floor lines
- `#6B7280` (Tailwind gray-500) for secondary fills
- `#2C2C2C` (the code block background) for high-emphasis text

For multi-element figures (e.g., 3 different agents), use the per-concept palettes from `color-palettes.md` rather than picking colors freehand.

---

## Performance

These figures are small and Canvas 2D is fast - you almost never need to optimize. Two specific things to avoid:

1. **Don't re-create gradients/paths every frame if they don't change.** Build them once outside `draw()` and reuse.
2. **Don't `console.log` in the animation loop.** It silently slows things down and clutters DevTools.

If a figure feels sluggish, the cause is almost always too many particles or too-frequent state updates, not Canvas overhead. Reduce count first; profile second.
