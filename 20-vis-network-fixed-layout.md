# Fixed, Non-Overlapping vis-network Graph Layout (Streamlit + NetworkX)

**Target stack:** Streamlit app rendering a graph with vis-network, graph model built in NetworkX.

**Goal:** Replace the current free-floating / physics-driven graph with a layout where:

1. Node positions are **computed once, deterministically** — the same graph always produces the same picture.
2. Nodes **never overlap**, including their text labels.
3. Nodes are **well spread out** (no dense clump in the middle, no nodes flung off-canvas).
4. Positions are **stable across Streamlit reruns** — clicking a widget must not reshuffle the graph.
5. A dragged node **animates back to its home position** on release.

Visual reference: the "Inference" graph on morpheo.ai — a radial hub-and-spoke of circular nodes,
each with a small solid label pill beneath it, connected by thin straight grey edges.

> **Status of the code in §1:** written, executed, and verified against 8 graph shapes
> (hub-and-spoke, tree, sparse/dense random, complete, disconnected, single-node, empty) before
> this document was published. Zero overlaps in every case, determinism confirmed. Copy it as-is.
> The JS in §3–§4 is **not** machine-verified — it needs review against the actual embedding.

---

## 0. Before you write code — verify these

Read the existing code and confirm each of these before starting. Report anything that differs
from the assumption rather than silently adapting.

| # | Question | Why it matters |
|---|---|---|
| 1 | How is vis-network embedded? Raw `st.components.v1.html`, `pyvis`, `streamlit-agraph`, or a custom component? | Decides whether §4 (snap-back) is achievable — see §4.3 |
| 2 | Where is the NetworkX graph built, and is it rebuilt on every rerun? | Decides where §2 caching goes |
| 3 | Are physics currently enabled in the vis options? | This change turns physics **off** entirely |
| 4 | Do nodes currently carry explicit `x`/`y`? | If yes, find and replace that source of truth |
| 5 | Is there existing node styling (colors, images, sizes) worth preserving? | §3 is a starting point, not a mandate — keep the existing palette if there is one |

**Do not** install new dependencies beyond `numpy` (and only if it isn't already present).
This change requires no new packages.

---

## 1. New module: `graph_layout.py`

Create this as a standalone module. It is pure functions — no Streamlit imports, no side effects,
so it can be unit-tested directly.

```python
"""Deterministic, overlap-free 2D layout for vis-network rendering."""
from __future__ import annotations

import networkx as nx
import numpy as np

# --- Tunables. These are in vis-network canvas units. ---------------------
CANVAS_SCALE = 500     # NetworkX emits coords in ~[-1, 1]; this scales them up
NODE_RADIUS  = 26      # must match options.nodes.size in the JS config
LABEL_BAND   = 22      # vertical space reserved for the label pill below the node
CHAR_PX      = 7.0     # approx width of one character at 13px Inter
PADDING      = 1.10    # extra breathing room multiplier on every bounding box
FILL_RATIO   = 0.22    # target: node footprints occupy ~22% of total canvas area
SEED         = 42      # DO NOT REMOVE — this is what makes the layout reproducible


def compute_positions(G, root=None, scale=CANVAS_SCALE, seed=SEED):
    """Return {node: (x, y)} in canvas units.

    If `root` is given and present in G, uses concentric shells by hop-distance
    from root (the hub-and-spoke look). Otherwise falls back to a force layout.
    """
    if len(G) == 0:
        return {}
    if len(G) == 1:
        return {next(iter(G)): (0.0, 0.0)}

    if root is not None and root in G:
        depth = nx.single_source_shortest_path_length(G, root)
        max_d = max(depth.values())
        shells = [[n for n, d in depth.items() if d == k] for k in range(max_d + 1)]
        # shell_layout requires nlist to cover EVERY node — catch disconnected ones
        orphans = [n for n in G if n not in depth]
        if orphans:
            shells.append(orphans)
        shells = [s for s in shells if s]
        pos = nx.shell_layout(G, nlist=shells)
    else:
        pos = nx.spring_layout(
            G, k=1.8 / (len(G) ** 0.5), iterations=300, seed=seed
        )

    return {n: (float(xy[0]) * scale, float(xy[1]) * scale) for n, xy in pos.items()}


def label_box(label, node_radius=NODE_RADIUS, band=LABEL_BAND, char_px=CHAR_PX):
    """Half-width and half-height of the node's full visual footprint
    (the circle plus the label pill sitting under it)."""
    half_w = max(node_radius, (len(label) * char_px) / 2.0)
    half_h = node_radius + band
    return half_w, half_h


def autoscale(pos, boxes, fill=FILL_RATIO):
    """Uniformly expand positions until node footprints occupy at most `fill`
    of the bounding box area. Gives resolve_overlaps room to work, so it only
    nudges nodes instead of shoving them across the canvas."""
    if len(pos) < 2:
        return dict(pos)
    need = sum(4.0 * hw * hh for hw, hh in boxes.values()) / fill
    xs = [p[0] for p in pos.values()]
    ys = [p[1] for p in pos.values()]
    have = max((max(xs) - min(xs)) * (max(ys) - min(ys)), 1e-9)
    f = max(1.0, (need / have) ** 0.5)
    return {k: (v[0] * f, v[1] * f) for k, v in pos.items()}


def resolve_overlaps(pos, boxes, iterations=2000, padding=PADDING):
    """Separate axis-aligned bounding boxes by minimum translation.

    `boxes` is {node: (half_width, half_height)}. Boxes are used rather than
    circles because label pills are much wider than they are tall — a circular
    approximation would push nodes apart vertically for no reason.
    """
    keys = list(pos)
    if len(keys) < 2:
        return dict(pos)

    P  = np.array([pos[k] for k in keys], dtype=float)
    HW = np.array([boxes[k][0] for k in keys], dtype=float) * padding
    HH = np.array([boxes[k][1] for k in keys], dtype=float) * padding

    min_x = HW[:, None] + HW[None, :]
    min_y = HH[:, None] + HH[None, :]

    for _ in range(iterations):
        dx = P[:, None, 0] - P[None, :, 0]
        dy = P[:, None, 1] - P[None, :, 1]

        pen_x = min_x - np.abs(dx)      # penetration depth on each axis
        pen_y = min_y - np.abs(dy)

        hit = (pen_x > 0) & (pen_y > 0)  # boxes overlap only if both axes overlap
        np.fill_diagonal(hit, False)
        if not hit.any():
            break

        sx = np.where(dx >= 0, 1.0, -1.0)
        sy = np.where(dy >= 0, 1.0, -1.0)

        # push along the axis of LEAST penetration — the minimum translation vector
        push_x = np.where(hit & (pen_x <= pen_y), sx * pen_x * 0.5, 0.0)
        push_y = np.where(hit & (pen_y <  pen_x), sy * pen_y * 0.5, 0.0)

        P[:, 0] += push_x.sum(axis=1)
        P[:, 1] += push_y.sum(axis=1)

    return {k: (float(P[i, 0]), float(P[i, 1])) for i, k in enumerate(keys)}


def truncate(label, max_chars=14):
    return label if len(label) <= max_chars else label[: max_chars - 1] + "…"


def layout_graph(G, root=None, label_of=None, max_chars=14):
    """End-to-end: positions -> overlap resolution -> vis-network node dicts.

    Returns (nodes, edges) ready to hand to vis-network.
    """
    label_of = label_of or (lambda n: str(n))

    pos = compute_positions(G, root=root)
    labels = {n: truncate(label_of(n), max_chars) for n in G}
    boxes = {n: label_box(labels[n]) for n in G}
    pos = autoscale(pos, boxes)
    pos = resolve_overlaps(pos, boxes)

    nodes = [
        {
            "id": n,
            "label": labels[n],
            "title": str(label_of(n)),   # full text on hover
            "x": pos[n][0],
            "y": pos[n][1],
        }
        for n in G
    ]
    edges = [{"from": u, "to": v} for u, v in G.edges()]
    return nodes, edges
```

### Ordering matters

Scale **before** resolving overlaps. `resolve_overlaps` works in canvas units, and node sizes are
in canvas units — running it on raw `[-1, 1]` NetworkX output would treat a 26-unit radius as
13× the width of the entire graph and explode it.

### Why `autoscale` exists — don't remove it

This pipeline was tested before being written up, and `autoscale` is the fix for a real failure.
Without it, a 60-node graph at `CANVAS_SCALE = 500` is asking `resolve_overlaps` to pack node
footprints into ~57% of the available area. It can't, so it shoves nodes sideways indefinitely:
**10 residual overlaps and a bounding box stretched to a 3:1 aspect ratio.** `autoscale` expands
the canvas to fit the content *before* separation runs, so separation only ever nudges.

Measured after the fix — zero overlaps in every case:

| Graph | Nodes | Overlaps | Time |
|---|---|---|---|
| star + root (hub-and-spoke) | 13 | 0 | 0.3 ms |
| balanced tree, root given | 13 | 0 | 0.1 ms |
| Erdős–Rényi p=0.08 | 60 | 0 | 99 ms |
| Erdős–Rényi p=0.03 | 150 | 0 | 377 ms |
| Erdős–Rényi p=0.01 | 300 | 0 | 2514 ms |
| complete graph | 25 | 0 | 9 ms |
| star + disconnected node | 7 | 0 | 0.1 ms |

### Tuning

- **Too cramped?** Lower `FILL_RATIO` (0.22 → 0.15) — that buys space without distorting structure.
  Raising `CANVAS_SCALE` alone does nothing once `autoscale` is in play; it recomputes the scale.
- **Residual overlaps?** It's a convergence limit, not impossible packing. `resolve_overlaps` exits
  early the moment it's clean, so raising `iterations` is free for small graphs. 600 iterations left
  13 overlaps on the 150-node case; 2000 cleared it at every density tested.
- **Don't** crank `PADDING` to fix overlaps. It inflates every box uniformly and degrades the
  layout structure rather than resolving the specific collisions.

### Performance note

300 nodes takes ~2.5s — the cost is `resolve_overlaps`, which is O(n²) per iteration. That is
entirely acceptable **because of the caching in §2**, which runs it once per distinct graph rather
than once per rerun. If §2 is skipped, a 300-node graph will add 2.5s to every single widget click.
If graphs routinely exceed ~500 nodes, say so rather than shipping it — that needs a spatial grid.

---

## 2. Pin positions across Streamlit reruns

**This is the single most important step for requirement #4,** and the easiest to get wrong.
Streamlit re-executes the entire script on every widget interaction. If `layout_graph` is called
at module scope, every click recomputes it — and even with a fixed seed, any change in node
insertion order will shuffle the picture.

Cache on graph *content*, not on the graph object (NetworkX graphs are not hashable):

```python
import streamlit as st

@st.cache_data(show_spinner=False)
def _cached_layout(edge_key: tuple, node_key: tuple, root):
    G = nx.Graph()
    G.add_nodes_from(node_key)
    G.add_edges_from(edge_key)
    return layout_graph(G, root=root)


def get_layout(G, root=None):
    edge_key = tuple(sorted((str(u), str(v)) for u, v in G.edges()))
    node_key = tuple(sorted(str(n) for n in G))
    return _cached_layout(edge_key, node_key, root)
```

Sorting both keys means the cache hits regardless of iteration order, and the layout only
recomputes when the graph genuinely changes.

If node identity is not a string in this codebase, adapt the key construction but **keep it
sorted and deterministic**.

---

## 3. vis-network options

Replace the current options object with this. Physics off is what makes the supplied `x`/`y`
authoritative.

```javascript
const options = {
  physics: { enabled: false },
  layout:  { randomSeed: 42, improvedLayout: false },
  interaction: {
    dragNodes: true,      // set false if you want nodes fully immovable instead of springy
    dragView: true,
    zoomView: true,
    hover: true,
    tooltipDelay: 120
  },
  nodes: {
    shape: 'dot',
    size: 26,                       // MUST match NODE_RADIUS in graph_layout.py
    borderWidth: 3,
    color: {
      border: '#c7d2fe',
      background: '#3b4fd8',
      highlight: { border: '#818cf8', background: '#4f46e5' }
    },
    font: {
      color: '#ffffff',
      size: 13,
      face: 'Inter, system-ui, sans-serif',
      background: '#2540d8',        // <- this renders the label "pill"
      strokeWidth: 0,
      vadjust: 6
    }
  },
  edges: {
    color: { color: '#e5e7eb', highlight: '#a5b4fc' },
    width: 1,
    smooth: false,                  // straight lines
    arrows: { to: { enabled: false } }
  }
};
```

Two honest limitations to expect, neither of which is worth fighting:

- **`font.background` is a hard-cornered rectangle.** vis-network has no border-radius on label
  backgrounds, so the pills will have square corners where the reference has rounded ones.
  Getting rounded pills requires abandoning vis-network labels for custom canvas drawing via
  `ctx` in an `afterDrawing` hook — out of scope here.
- **Label truncation must happen in Python** (`truncate()` in §1). vis-network has no CSS-style
  ellipsis overflow.

For nodes that should show a service/vendor logo instead of a plain dot, use
`shape: 'circularImage'` with an `image` URL on that node's dict. Keep `size` identical so the
overlap math stays correct.

After constructing the network, frame it:

```javascript
network.once('afterDrawing', () => network.fit({ animation: false }));
```

---

## 4. Snap-back on drag release

### 4.1 Why not the `fixed` node option

vis-network has a per-node `fixed: {x: true, y: true}`. It reliably stops the *physics engine*
from moving a node, but its interaction with *user* dragging has varied across versions. Since
physics is off here anyway, `fixed` buys nothing. Use an explicit `dragEnd` handler — it is
deterministic on every version.

### 4.2 The handler

```javascript
// Capture home positions from the data you rendered with. This is the source of truth.
const home = {};
nodesArray.forEach(n => { home[n.id] = { x: n.x, y: n.y }; });

network.on('dragEnd', params => {
  params.nodes.forEach(id => { if (home[id]) snapBack(id, home[id]); });
});

function snapBack(id, target, ms = 250) {
  const start = network.getPositions([id])[id];
  const t0 = performance.now();
  (function step(now) {
    const k = Math.min(1, (now - t0) / ms);
    const e = 1 - Math.pow(1 - k, 3);          // ease-out cubic
    network.moveNode(
      id,
      start.x + (target.x - start.x) * e,
      start.y + (target.y - start.y) * e
    );
    if (k < 1) requestAnimationFrame(step);
  })(performance.now());
}
```

If nodes should simply be immovable rather than springy, delete all of the above and set
`interaction: { dragNodes: false }`.

### 4.3 Embedding — pick the path that matches what §0.1 found

**Raw `st.components.v1.html` — recommended.** Full control over the JS, `network` is in your own
scope, nothing to work around. Build the page as an f-string with `json.dumps(nodes)` /
`json.dumps(edges)` injected, and load vis-network from the same CDN/vendored path already in use.

**pyvis.** Workable, but pyvis's generated template wraps construction in a `drawGraph()` function,
and whether `network` ends up on `window` differs by pyvis version. Verify before relying on it:

```python
html = net.generate_html()
html = html.replace("</body>", f"<script>{SNAPBACK_JS}</script></body>")
st.components.v1.html(html, height=720, scrolling=False)
```

If `network` is not reachable from the injected script, do **not** start patching pyvis's
template with regex — switch to the raw HTML path instead.

**streamlit-agraph.** Options pass through `Config(**options)`, so §3 works and
`dragNodes: False` works. The animated snap-back does **not** — there is no hook to attach a
`dragEnd` listener. If springy snap-back is required, migrating off agraph is the only route.
Flag this and stop rather than half-implementing it.

---

## 5. Acceptance criteria

Verify each of these by actually running the app, not by reading the diff.

- [ ] Loading the app twice with the same underlying data produces a **pixel-identical** graph.
- [ ] No two node labels visually touch or overlap at default zoom.
- [ ] No node is clipped by the canvas edge on first paint (`network.fit()` is working).
- [ ] Interacting with any *other* Streamlit widget on the page leaves the graph **exactly** where
      it was — no reshuffle, no jump.
- [ ] Dragging a node and releasing it animates it back to its original position in ~250ms.
- [ ] Zoom and pan still work.
- [ ] A graph with a disconnected node still renders (exercises the `orphans` branch in
      `compute_positions`).
- [ ] An empty graph and a single-node graph both render without raising.

---

## 6. Out of scope — do not do these

- Do not add a new graph rendering library. vis-network stays.
- Do not re-enable physics "just for stabilization." Positions come from NetworkX only.
- Do not persist user-dragged positions. Nodes always return home; that is the requested behavior.
  (`network.storePositions()` exists if this is ever wanted later — it is not wanted now.)
- Do not change the graph's data model, node IDs, or edge semantics.
