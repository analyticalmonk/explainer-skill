# Code-Specific Figure Archetypes

Figures for code explainers. The mechanics - the DPR-aware `initCanvas`, the self-contained IIFE pattern, `requestAnimationFrame` loops, and the base archetypes (stepped simulation, continuous animation, tabbed comparison, 3D, parameter exploration) - all come from the base `figure-archetypes.md` in `creating-explainers`. This file only covers how to apply them to code, plus the one pattern (the architecture diagram) specific enough to need its own example.

As always: a figure must illustrate something the prose just set up. No decorative diagrams.

## Architecture / Module Diagram

**Shows:** the modules of a system as boxes, their dependencies or calls as arrows. **When:** an overview article, or to orient the reader before a deep-dive. **Interaction:** click a module to highlight what it connects to.

This is the most code-specific pattern, so here is a worked sketch (adapt freely):

```javascript
(function() {
  const { canvas, ctx, w, h } = initCanvas('canvas-arch');
  const accent = '#2563eb'; // the article's accent color
  // modules: {id, label, x, y} as fractions of the canvas, laid out by hand
  const mods = [
    { id: 'router',  label: 'Router',  x: 0.5,  y: 0.15 },
    { id: 'handler', label: 'Handler', x: 0.5,  y: 0.45 },
    { id: 'store',   label: 'Store',   x: 0.25, y: 0.78 },
    { id: 'cache',   label: 'Cache',   x: 0.75, y: 0.78 },
  ];
  const edges = [['router','handler'],['handler','store'],['handler','cache']];
  const box = { w: 110, h: 44 };
  let active = null;

  const center = m => ({ x: m.x * w, y: m.y * h });
  const neighbors = id => edges.filter(e => e.includes(id)).flat().filter(x => x !== id);

  function draw() {
    ctx.clearRect(0, 0, w, h);
    const near = active ? new Set(neighbors(active)) : null;
    edges.forEach(([a, b]) => {
      const lit = active && (a === active || b === active);
      ctx.strokeStyle = lit ? accent : '#d4cfc6';
      ctx.lineWidth = lit ? 2 : 1;
      const A = center(mods.find(m => m.id === a));
      const B = center(mods.find(m => m.id === b));
      ctx.beginPath(); ctx.moveTo(A.x, A.y); ctx.lineTo(B.x, B.y); ctx.stroke();
    });
    mods.forEach(m => {
      const c = center(m);
      const lit = !active || m.id === active || (near && near.has(m.id));
      ctx.globalAlpha = lit ? 1 : 0.35;
      ctx.fillStyle = m.id === active ? '#1a1a1a' : '#ffffff';
      ctx.strokeStyle = '#1a1a1a';
      ctx.fillRect(c.x - box.w/2, c.y - box.h/2, box.w, box.h);
      ctx.strokeRect(c.x - box.w/2, c.y - box.h/2, box.w, box.h);
      ctx.fillStyle = m.id === active ? '#ffffff' : '#1a1a1a';
      ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
      ctx.fillText(m.label, c.x, c.y);
      ctx.globalAlpha = 1;
    });
  }
  canvas.addEventListener('click', e => {
    const r = canvas.getBoundingClientRect();
    const mx = e.clientX - r.left, my = e.clientY - r.top;
    const hit = mods.find(m => {
      const c = center(m);
      return Math.abs(mx - c.x) < box.w/2 && Math.abs(my - c.y) < box.h/2;
    });
    active = hit ? hit.id : null;
    draw();
  });
  draw();
})();
```

Set `accent` to the article's real accent color, and use the actual module names and edges from the codebase, not these placeholders.

## Data-Flow / Sequence Diagram

**Shows:** an item (a request, a message, a token, an event) moving through the stages that process it. **Builds on:** the continuous-animation archetype, where a `requestAnimationFrame` loop advances the item's position along a path of stages. **When:** explaining a pipeline or a request lifecycle. Label each stage; let the reader play, pause, and step. Keep the stages to the real ones in the code.

## Execution-Trace Stepper

**Shows:** an algorithm running, one step at a time, with the relevant state drawn on the canvas and the current line of code highlighted beside it. **Builds on:** the stepped-simulation archetype, where each Step advances one operation. **When:** a mechanism deep-dive where the order of operations is the point (a sort, a graph traversal, a training loop). Pair the canvas with a code panel: keep an array of steps, each naming the active line and the state to render, and highlight that line as you draw the state. This is the highest-value figure for a deep-dive; the reader sees the code and its effect at the same time.

## Annotated Code Walkthrough

**Shows:** a block of real code whose lines reveal short annotations as the reader steps or hovers. **When:** the code is compact enough to show and the insight lives in specific lines. **Build:** render the snippet as styled HTML (one element per line, each with a `data-note` attribute), plus a small controller that reveals the note for the active line and advances on click. Often needs no canvas at all. Keep annotations to one sentence each, on the lines that matter.

## Dependency Graph

**Shows:** modules or packages as nodes, dependencies as edges, for a system whose shape is the story. **Builds on:** the architecture diagram above (for a small fixed set) or a continuous-animation force layout (for many nodes). Prefer a hand-placed layout when you can; auto-layout is rarely worth the code. Use this when the coupling between parts is itself the thing you are explaining.
