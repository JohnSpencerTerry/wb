---
layout: post
title: "Agentic Workflow Builder: Canvas Prototype"
date: 2026-08-16
---

Agentic workflows are going to show up everywhere, but most companies that want one won't have the in-house expertise to build it. This is an early concept for a tool that lets a team assemble an agentic workflow without writing the orchestration code themselves.

## The concept

The tool would walk a team through a short questionnaire about what they're trying to automate, then let them build the workflow visually — picking from premade steps or defining custom ones, wiring them into a graph similar to a Dagster or Airflow DAG. A few ideas so far:

- A questionnaire that turns answers about the task into a starting workflow
- Testing as a first-class citizen, not something bolted on after the workflow works
- A visual builder for the graph itself, with premade and custom steps
- "Agent as computer" steps, similar to Operator mode in ChatGPT, where a step controls a UI directly instead of calling an API
- Prebuilt UI components for different entry points (chat, form, webhook, scheduled run)

*Building AI-Driven Products* has some good material on this that's shaping how I'm thinking about it.

## The canvas

The visual builder is the part I wanted to prototype first. Drag a step from the palette onto the canvas, connect steps by dragging from one port to another, then run a test pass to watch execution move through the graph in order.

<div id="awb" class="awb-panel">
  <div class="awb-toolbar">
    <span class="awb-hint">Drag a step onto the canvas, then drag from a step's right dot to another step's left dot to connect them.</span>
    <div class="awb-actions">
      <button type="button" class="awb-run">Run Test</button>
      <button type="button" class="awb-reset">Reset</button>
    </div>
  </div>
  <div class="awb-body">
    <div class="awb-palette">
      <div class="awb-palette-item" draggable="true" data-type="trigger">Trigger</div>
      <div class="awb-palette-item" draggable="true" data-type="llm">LLM Call</div>
      <div class="awb-palette-item" draggable="true" data-type="tool">Tool Call</div>
      <div class="awb-palette-item" draggable="true" data-type="human">Human Review</div>
      <div class="awb-palette-item" draggable="true" data-type="branch">Branch</div>
      <div class="awb-palette-item" draggable="true" data-type="output">Output</div>
    </div>
    <div class="awb-canvas-wrap">
      <svg class="awb-edges"></svg>
      <div class="awb-canvas"></div>
    </div>
  </div>
  <div class="awb-log"></div>
</div>

<style>
  .awb-panel {
    background: #0e0e10;
    color: #fff;
    border-radius: 10px;
    padding: 1rem 1.1rem 1.25rem;
    margin: 1.5rem 0;
    font-family: system-ui, sans-serif;
  }
  .awb-toolbar {
    display: flex; flex-wrap: wrap; gap: 0.75rem;
    align-items: center; justify-content: space-between;
    margin-bottom: 0.75rem;
  }
  .awb-hint { color: #999; font-size: 0.8rem; max-width: 60ch; }
  .awb-actions { display: flex; gap: 0.5rem; flex-shrink: 0; }
  .awb-actions button {
    background: #fff; color: #000; border: 0;
    border-radius: 999px; padding: 0.45rem 1rem;
    font-weight: 600; font-size: 0.85rem;
    cursor: pointer; font-family: inherit;
  }
  .awb-actions .awb-reset { background: transparent; color: #999; border: 1px solid #444; }
  .awb-body {
    display: flex; gap: 0.75rem;
    border: 1px solid #26262a; border-radius: 8px;
    overflow: hidden;
  }
  .awb-palette {
    flex: 0 0 120px;
    background: #17171a;
    padding: 0.6rem;
    display: flex; flex-direction: column; gap: 0.5rem;
  }
  .awb-palette-item {
    background: #232327; border: 1px solid #333;
    border-radius: 6px; padding: 0.5rem 0.4rem;
    font-size: 0.78rem; text-align: center;
    cursor: grab;
  }
  .awb-palette-item:active { cursor: grabbing; }
  .awb-canvas-wrap {
    position: relative; flex: 1;
    height: 340px;
    background:
      linear-gradient(90deg, #1a1a1d 1px, transparent 1px),
      linear-gradient(#1a1a1d 1px, transparent 1px);
    background-size: 20px 20px;
    background-color: #0b0b0d;
    overflow: hidden;
  }
  .awb-edges, .awb-canvas {
    position: absolute; inset: 0; width: 100%; height: 100%;
  }
  .awb-edges { pointer-events: none; }
  .awb-edges path { fill: none; stroke: #555; stroke-width: 2; }
  .awb-edges path.awb-active { stroke: #2ecc71; }
  .awb-node {
    position: absolute;
    width: 110px;
    background: #1f1f23; border: 1px solid #3a3a40;
    border-radius: 6px;
    padding: 0.4rem 0.5rem;
    font-size: 0.78rem;
    cursor: move;
    user-select: none;
  }
  .awb-node.awb-running { border-color: #2ecc71; box-shadow: 0 0 0 2px rgba(46,204,113,0.3); }
  .awb-node .awb-node-del {
    position: absolute; top: -6px; right: -6px;
    width: 16px; height: 16px; border-radius: 50%;
    background: #333; color: #ccc; font-size: 0.65rem;
    display: flex; align-items: center; justify-content: center;
    cursor: pointer; border: 1px solid #555;
  }
  .awb-port {
    position: absolute; top: 50%; transform: translateY(-50%);
    width: 10px; height: 10px; border-radius: 50%;
    background: #555; border: 2px solid #888;
    cursor: crosshair;
  }
  .awb-port.awb-in { left: -6px; }
  .awb-port.awb-out { right: -6px; }
  .awb-port:hover { background: #2ecc71; }
  .awb-palette-item[data-type="trigger"], .awb-node[data-type="trigger"] { border-left: 3px solid #4a9eff; }
  .awb-palette-item[data-type="tool"], .awb-node[data-type="tool"] { border-left: 3px solid #ff9f43; }
  .awb-palette-item[data-type="llm"], .awb-node[data-type="llm"] { border-left: 3px solid #b967ff; }
  .awb-palette-item[data-type="human"], .awb-node[data-type="human"] { border-left: 3px solid #2ecc71; }
  .awb-palette-item[data-type="branch"], .awb-node[data-type="branch"] { border-left: 3px solid #f1c40f; }
  .awb-palette-item[data-type="output"], .awb-node[data-type="output"] { border-left: 3px solid #8892a0; }
  .awb-log {
    margin-top: 0.6rem; font-size: 0.78rem; color: #999;
    min-height: 1.2em;
  }
</style>

<script>
(function() {
  const root = document.getElementById('awb');
  const canvasWrap = root.querySelector('.awb-canvas-wrap');
  const canvas = root.querySelector('.awb-canvas');
  const svg = root.querySelector('.awb-edges');
  const logEl = root.querySelector('.awb-log');
  const runBtn = root.querySelector('.awb-run');
  const resetBtn = root.querySelector('.awb-reset');

  const LABELS = {
    trigger: 'Trigger', llm: 'LLM Call', tool: 'Tool Call',
    human: 'Human Review', branch: 'Branch', output: 'Output'
  };

  let nodes = [];
  let edges = [];
  let nextId = 1;
  let pendingConnect = null;

  function makeNode(type, x, y) {
    const id = 'n' + (nextId++);
    const el = document.createElement('div');
    el.className = 'awb-node';
    el.style.left = x + 'px';
    el.style.top = y + 'px';
    el.dataset.id = id;
    el.dataset.type = type;
    el.innerHTML =
      '<div class="awb-node-label">' + (LABELS[type] || type) + '</div>' +
      '<div class="awb-port awb-in" data-id="' + id + '" data-dir="in"></div>' +
      '<div class="awb-port awb-out" data-id="' + id + '" data-dir="out"></div>' +
      '<div class="awb-node-del">&times;</div>';
    canvas.appendChild(el);
    nodes.push({ id, type, x, y });
    wireNode(el);
    return el;
  }

  function wireNode(el) {
    const id = el.dataset.id;
    let dragging = null;
    el.addEventListener('mousedown', (e) => {
      if (e.target.classList.contains('awb-port') || e.target.classList.contains('awb-node-del')) return;
      const rect = canvasWrap.getBoundingClientRect();
      dragging = {
        offsetX: e.clientX - rect.left - el.offsetLeft,
        offsetY: e.clientY - rect.top - el.offsetTop
      };
    });
    document.addEventListener('mousemove', (e) => {
      if (!dragging) return;
      const rect = canvasWrap.getBoundingClientRect();
      const x = Math.max(0, e.clientX - rect.left - dragging.offsetX);
      const y = Math.max(0, e.clientY - rect.top - dragging.offsetY);
      el.style.left = x + 'px';
      el.style.top = y + 'px';
      const node = nodes.find(n => n.id === id);
      node.x = x;
      node.y = y;
      drawEdges();
    });
    document.addEventListener('mouseup', () => { dragging = null; });

    el.querySelector('.awb-node-del').addEventListener('click', () => {
      nodes = nodes.filter(n => n.id !== id);
      edges = edges.filter(ed => ed.from !== id && ed.to !== id);
      el.remove();
      drawEdges();
    });

    el.querySelectorAll('.awb-port').forEach(port => {
      port.addEventListener('mousedown', (e) => {
        e.stopPropagation();
        pendingConnect = { id, dir: port.dataset.dir };
      });
      port.addEventListener('mouseup', (e) => {
        e.stopPropagation();
        if (!pendingConnect || pendingConnect.id === id) { pendingConnect = null; return; }
        let from = pendingConnect, to = { id, dir: port.dataset.dir };
        if (from.dir === 'in') { const t = from; from = to; to = t; }
        if (from.dir === 'out' && to.dir === 'in') {
          edges.push({ from: from.id, to: to.id });
          drawEdges();
        }
        pendingConnect = null;
      });
    });
  }

  function portPos(id, dir) {
    const node = nodes.find(n => n.id === id);
    if (!node) return { x: 0, y: 0 };
    const el = canvas.querySelector('[data-id="' + id + '"]');
    const width = el ? el.offsetWidth : 110;
    const height = el ? el.offsetHeight : 40;
    return {
      x: node.x + (dir === 'out' ? width : 0),
      y: node.y + height / 2
    };
  }

  function drawEdges(activeSet) {
    svg.innerHTML = '';
    edges.forEach(ed => {
      const p1 = portPos(ed.from, 'out');
      const p2 = portPos(ed.to, 'in');
      const midX = (p1.x + p2.x) / 2;
      const d = 'M ' + p1.x + ' ' + p1.y + ' C ' + midX + ' ' + p1.y + ', ' + midX + ' ' + p2.y + ', ' + p2.x + ' ' + p2.y;
      const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
      path.setAttribute('d', d);
      if (activeSet && activeSet.has(ed.from + '>' + ed.to)) path.classList.add('awb-active');
      svg.appendChild(path);
    });
  }

  root.querySelectorAll('.awb-palette-item').forEach(item => {
    item.addEventListener('dragstart', (e) => {
      e.dataTransfer.setData('text/plain', item.dataset.type);
    });
  });

  canvasWrap.addEventListener('dragover', (e) => e.preventDefault());
  canvasWrap.addEventListener('drop', (e) => {
    e.preventDefault();
    const type = e.dataTransfer.getData('text/plain');
    if (!type) return;
    const rect = canvasWrap.getBoundingClientRect();
    const x = Math.max(0, e.clientX - rect.left - 55);
    const y = Math.max(0, e.clientY - rect.top - 20);
    makeNode(type, x, y);
  });

  function topoOrder() {
    const incoming = {};
    nodes.forEach(n => incoming[n.id] = 0);
    edges.forEach(ed => incoming[ed.to] = (incoming[ed.to] || 0) + 1);
    const queue = nodes.filter(n => incoming[n.id] === 0).map(n => n.id);
    const order = [];
    const seen = new Set();
    while (queue.length) {
      const id = queue.shift();
      if (seen.has(id)) continue;
      seen.add(id);
      order.push(id);
      edges.filter(ed => ed.from === id).forEach(ed => {
        incoming[ed.to]--;
        if (incoming[ed.to] <= 0) queue.push(ed.to);
      });
    }
    nodes.forEach(n => { if (!seen.has(n.id)) order.push(n.id); });
    return order;
  }

  runBtn.addEventListener('click', () => {
    if (!nodes.length) { logEl.textContent = 'Add some steps first.'; return; }
    const order = topoOrder();
    const labels = order.map(id => {
      const node = nodes.find(n => n.id === id);
      return node ? (LABELS[node.type] || node.type) : null;
    }).filter(Boolean);
    logEl.textContent = 'Running: ' + labels.join(' → ');
    canvas.querySelectorAll('.awb-node').forEach(el => el.classList.remove('awb-running'));
    const activeEdges = new Set();
    order.forEach((id, i) => {
      setTimeout(() => {
        canvas.querySelectorAll('.awb-node').forEach(el => el.classList.remove('awb-running'));
        const el = canvas.querySelector('[data-id="' + id + '"]');
        if (el) el.classList.add('awb-running');
        if (i > 0) {
          const prev = order[i - 1];
          activeEdges.add(prev + '>' + id);
        }
        drawEdges(activeEdges);
        if (i === order.length - 1) {
          setTimeout(() => logEl.textContent = 'Done: ' + labels.join(' → '), 500);
        }
      }, i * 550);
    });
  });

  resetBtn.addEventListener('click', () => {
    nodes = [];
    edges = [];
    canvas.innerHTML = '';
    svg.innerHTML = '';
    logEl.textContent = '';
  });

  makeNode('trigger', 20, 40);
  makeNode('llm', 180, 140);
  makeNode('output', 340, 40);
  edges.push({ from: 'n1', to: 'n2' }, { from: 'n2', to: 'n3' });
  drawEdges();
})();
</script>

## Step details

What each step actually needs under the hood:

<div class="awbd-grid">
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#4a9eff"></span>Trigger</div>
    <div class="awbd-field"><span class="awbd-label">Kind</span><span class="awbd-val">Schedule · Calendar · Event · API call</span></div>
    <div class="awbd-field"><span class="awbd-label">Schedule</span><span class="awbd-val">Cron-style cadence</span></div>
    <div class="awbd-field"><span class="awbd-label">Event</span><span class="awbd-val">Topic/queue name, payload shape</span></div>
    <div class="awbd-field"><span class="awbd-label">API</span><span class="awbd-val">Auth method, input schema</span></div>
  </div>
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#ff9f43"></span>Tool Call</div>
    <div class="awbd-field"><span class="awbd-label">Connection</span><span class="awbd-val">REST/GraphQL API · MCP server</span></div>
    <div class="awbd-field"><span class="awbd-label">Auth</span><span class="awbd-val">API key · OAuth · service account · none</span></div>
    <div class="awbd-field"><span class="awbd-label">Schema</span><span class="awbd-val">Input params, output shape</span></div>
    <div class="awbd-field"><span class="awbd-label">Reliability</span><span class="awbd-val">Retry policy, timeout</span></div>
  </div>
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#b967ff"></span>LLM Call</div>
    <div class="awbd-field"><span class="awbd-label">Model</span><span class="awbd-val">Provider + model</span></div>
    <div class="awbd-field"><span class="awbd-label">Prompt</span><span class="awbd-val">Template, variables from upstream steps</span></div>
    <div class="awbd-field"><span class="awbd-label">Output</span><span class="awbd-val">Freeform text or structured/JSON schema</span></div>
    <div class="awbd-field"><span class="awbd-label">Fallback</span><span class="awbd-val">Secondary model on failure</span></div>
  </div>
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#2ecc71"></span>Human Review</div>
    <div class="awbd-field"><span class="awbd-label">Channel</span><span class="awbd-val">Slack · email · in-app queue</span></div>
    <div class="awbd-field"><span class="awbd-label">Shown to reviewer</span><span class="awbd-val">Proposed action, not raw state</span></div>
    <div class="awbd-field"><span class="awbd-label">Actions</span><span class="awbd-val">Approve · reject · edit · redirect</span></div>
    <div class="awbd-field"><span class="awbd-label">Timeout</span><span class="awbd-val">Auto-approve, escalate, or block</span></div>
  </div>
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#f1c40f"></span>Branch</div>
    <div class="awbd-field"><span class="awbd-label">Condition</span><span class="awbd-val">Rule-based comparison or LLM-judged classification</span></div>
    <div class="awbd-field"><span class="awbd-label">Paths</span><span class="awbd-val">Named outputs + default fallback</span></div>
    <div class="awbd-field"><span class="awbd-label">Merge</span><span class="awbd-val">How state reconciles if paths rejoin</span></div>
  </div>
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#8892a0"></span>Output</div>
    <div class="awbd-field"><span class="awbd-label">Destination</span><span class="awbd-val">Return to caller · write to store · send message · trigger another workflow</span></div>
    <div class="awbd-field"><span class="awbd-label">Format</span><span class="awbd-val">Matches caller's expected shape, or freeform</span></div>
    <div class="awbd-field"><span class="awbd-label">Trace</span><span class="awbd-val">Attach run metadata for debugging</span></div>
  </div>
</div>

<style>
  .awbd-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 0.75rem;
    margin: 1.5rem 0;
  }
  .awbd-card {
    background: #0e0e10;
    border: 1px solid #26262a;
    border-radius: 8px;
    padding: 0.8rem 0.9rem;
    font-family: system-ui, sans-serif;
    color: #fff;
  }
  .awbd-head {
    display: flex; align-items: center; gap: 0.5rem;
    font-size: 0.85rem; font-weight: 600;
    margin-bottom: 0.6rem;
  }
  .awbd-dot { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
  .awbd-field {
    display: flex; flex-direction: column; gap: 0.1rem;
    padding: 0.35rem 0;
    border-top: 1px solid #1e1e22;
  }
  .awbd-field:first-of-type { border-top: 0; }
  .awbd-label {
    text-transform: uppercase; font-size: 0.62rem;
    letter-spacing: 0.06em; color: #888;
  }
  .awbd-val { font-size: 0.78rem; color: #ddd; }
</style>
