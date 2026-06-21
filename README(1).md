# Pathfinding Visualizer

An interactive grid where you draw walls, then watch Dijkstra's algorithm search outward from a start node, find the shortest route to a finish node, and animate the result.

Built with React (class components) and plain CSS — no other UI libraries.

## How it works

### The algorithm

The grid is a graph: every cell is a node connected to its direct up/down/left/right neighbors, and every connection costs exactly 1 step. `src/algorithms/dijkstra.js` runs Dijkstra's algorithm over that graph:

1. The start node's distance is set to `0`; every other node starts at `Infinity` (unknown).
2. Repeatedly, the algorithm grabs the closest unvisited node, marks it visited, and records it in a list (`visitedNodesInOrder`) — this list is what drives the rippling blue animation, since its order *is* the order the algorithm actually explored cells in.
3. For each neighbor of that node (skipping walls and already-visited cells), it sets `neighbor.distance = currentNode.distance + 1` and remembers `neighbor.previousNode = currentNode` — a breadcrumb trail back to where it came from.
4. This repeats until the finish node is reached (success) or there are no more reachable nodes (the finish is walled off).
5. Once the finish node is found, `getNodesInShortestPathOrder` walks backward through the `previousNode` breadcrumbs from finish to start, then reverses the order, producing the actual shortest path.

Because every step on this grid costs the same, this is mathematically equivalent to breadth-first search — Dijkstra's algorithm is built to also handle *weighted* graphs (where some steps cost more than others), but that flexibility isn't exercised here since every edge weighs 1.

### The animation

`animateDijkstra` and `animateShortestPath` in `PathfindingVisualizer.jsx` don't use React state to drive the animation. Instead, they schedule a staggered sequence of `setTimeout` calls and reach directly into the DOM (`document.getElementById(...).className = ...`) to flip each cell's CSS class. The moment a class containing `animation-name` (defined in `Node.css`) gets applied, the browser plays that animation automatically. This bypasses React's normal render cycle, which is a deliberate performance choice — animating hundreds of cells through full React re-renders would visibly lag, while direct DOM updates don't.

### The component structure

```
App
 └─ PathfindingVisualizer   (owns state: the grid, whether the mouse is held down)
     └─ Node × 750           (one per cell; receives data as props, reports clicks via callback props)
```

State flows down from `PathfindingVisualizer` to each `Node` as props (`isWall`, `isStart`, `isFinish`, etc.). Mouse events flow back up: a `Node` never changes itself — it calls a callback function it was handed (`onMouseDown`, `onMouseEnter`, `onMouseUp`), which updates `PathfindingVisualizer`'s state, which re-renders the grid with the new wall layout.

## Features

- Click and drag across the grid to draw or erase walls
- Animated visualization of the algorithm's search order (blue ripple)
- Animated highlight of the final shortest path (yellow trail)
- Pure CSS keyframe animations, triggered via class changes for performance

## Tech stack

- React 18 (class components, not hooks — the original tutorial predates hooks, but class components are still fully supported)
- Create React App / `react-scripts` 5 for the build tooling
- Plain CSS (no CSS-in-JS, no component libraries)

## Project structure

```
pathfinding-visualizer/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
├── src/
│   ├── algorithms/
│   │   └── dijkstra.js              # Pathfinding logic, no React
│   ├── PathfindingVisualizer/
│   │   ├── PathfindingVisualizer.jsx  # Owns the grid + animation logic
│   │   ├── PathfindingVisualizer.css
│   │   └── Node/
│   │       ├── Node.jsx             # A single grid cell
│   │       └── Node.css             # Cell styling + animations
│   ├── App.js                       # Root component
│   ├── App.css
│   ├── App.test.js
│   ├── index.js                     # Entry point — mounts App into the page
│   ├── index.css
│   ├── logo.svg
│   └── serviceWorker.js
├── .gitignore
├── .prettierrc.json
├── package.json
├── package-lock.json
└── README.md
```

## Getting started

Requires [Node.js](https://nodejs.org/) (which includes `npm`).

```bash
npm install   # downloads React and other dependencies into node_modules
npm start     # starts a dev server at http://localhost:3000
```

Open the URL in your browser, draw some walls by clicking and dragging, then click "Visualize Dijkstra's Algorithm."

## Known limitations / ideas for future work

- Start and finish positions are hard-coded (`START_NODE_ROW/COL`, `FINISH_NODE_ROW/COL` in `PathfindingVisualizer.jsx`) rather than draggable.
- All edges are weight-1 — there's no concept of "slow terrain," so Dijkstra's weighted-graph capability isn't really being used.
- The unvisited-node list is fully re-sorted every loop iteration (`sortNodesByDistance`), which is simple but not as fast as a proper priority queue — fine at this grid size, but worth knowing if you scale it up.
- Only one algorithm is implemented. Comparing it against breadth-first search, A*, or a maze-generation mode would be natural next steps.
- No "clear path" vs. "clear everything" distinction — running the visualization again requires clearing walls first if you want a clean slate.

## Credits

This project is based on the [Pathfinding Visualizer tutorial](https://github.com/clementmihailescu/Pathfinding-Visualizer) by [Clément Mihailescu](https://www.youtube.com/channel/UCaO6VoaYJv4kS-TQO_M-N_g). The core logic in `src/PathfindingVisualizer/` and `src/algorithms/` follows his original implementation.
