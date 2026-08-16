---
layout: post
title: "Agentic Workflow Builder: Canvas Prototype"
date: 2026-08-14
---

Agentic workflows are showing up everywhere, but most companies that want one don't have the in-house expertise to build it. This is an early concept for a tool that lets a team assemble an agentic workflow without writing the orchestration code themselves.

## The concept

The tool would walk a team through a short questionnaire about what they're trying to automate, then let them build the workflow visually — picking from premade steps or defining custom ones, wiring them into a visual graph, similar to a DAG. A few ideas so far:

- A questionnaire that turns answers about the task into a starting workflow
- Testing as a first-class citizen, not something bolted on after the workflow works
- A visual builder for the graph itself, with premade and custom steps
- "Agent as computer" steps, similar to Operator mode in ChatGPT, where a step controls a UI directly instead of calling an API
- Prebuilt UI components for different entry points (chat, form, webhook, scheduled run)

*[Building AI-Powered Products](https://www.oreilly.com/library/view/building-ai-powered-products/9781098152697/)* has some good material on this that's shaping how I'm thinking about it.

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
      <div class="awb-palette-item" data-type="trigger">Trigger</div>
      <div class="awb-palette-item" data-type="llm">LLM Call</div>
      <div class="awb-palette-item" data-type="tool">Tool Call</div>
      <div class="awb-palette-item" data-type="human">Human Review</div>
      <div class="awb-palette-item" data-type="branch">Branch</div>
      <div class="awb-palette-item" data-type="output">Output</div>
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
    touch-action: none;
  }
  .awb-palette-item:active { cursor: grabbing; }
  .awb-ghost {
    position: fixed; z-index: 9999;
    transform: translate(-50%, -50%);
    pointer-events: none;
    background: #2a2a2f; border: 1px solid #555;
    border-radius: 6px; padding: 0.4rem 0.7rem;
    font-size: 0.78rem; color: #fff;
    font-family: system-ui, sans-serif;
    box-shadow: 0 4px 12px rgba(0,0,0,0.4);
  }
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
    touch-action: none;
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
    width: 22px; height: 22px; border-radius: 50%;
    background: transparent; border: 0;
    cursor: crosshair;
    touch-action: none;
    display: flex; align-items: center; justify-content: center;
  }
  .awb-port::after {
    content: ""; width: 10px; height: 10px; border-radius: 50%;
    background: #555; border: 2px solid #888;
  }
  .awb-port.awb-in { left: -12px; }
  .awb-port.awb-out { right: -12px; }
  .awb-port:hover::after { background: #2ecc71; }
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
  @media (max-width: 560px) {
    .awb-body { flex-direction: column; }
    .awb-palette {
      flex: 0 0 auto;
      flex-direction: row;
      overflow-x: auto;
      -webkit-overflow-scrolling: touch;
    }
    .awb-palette-item { flex: 0 0 auto; white-space: nowrap; }
    .awb-canvas-wrap { height: 260px; }
    .awb-node { width: 92px; font-size: 0.72rem; }
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

    el.addEventListener('pointerdown', (e) => {
      if (e.target.closest('.awb-port') || e.target.closest('.awb-node-del')) return;
      const rect = canvasWrap.getBoundingClientRect();
      dragging = {
        pointerId: e.pointerId,
        offsetX: e.clientX - rect.left - el.offsetLeft,
        offsetY: e.clientY - rect.top - el.offsetTop
      };
      el.setPointerCapture(e.pointerId);
    });
    el.addEventListener('pointermove', (e) => {
      if (!dragging || dragging.pointerId !== e.pointerId) return;
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
    const endDrag = (e) => {
      if (dragging && dragging.pointerId === e.pointerId) dragging = null;
    };
    el.addEventListener('pointerup', endDrag);
    el.addEventListener('pointercancel', endDrag);

    el.querySelector('.awb-node-del').addEventListener('click', () => {
      nodes = nodes.filter(n => n.id !== id);
      edges = edges.filter(ed => ed.from !== id && ed.to !== id);
      el.remove();
      drawEdges();
    });

    el.querySelectorAll('.awb-port').forEach(port => {
      port.addEventListener('pointerdown', (e) => {
        e.stopPropagation();
        pendingConnect = { id, dir: port.dataset.dir };
        port.setPointerCapture(e.pointerId);
      });
      port.addEventListener('pointerup', (e) => {
        e.stopPropagation();
        port.releasePointerCapture(e.pointerId);
        const dropEl = document.elementFromPoint(e.clientX, e.clientY);
        const dropPort = dropEl ? dropEl.closest('.awb-port') : null;
        if (dropPort && pendingConnect) {
          let from = pendingConnect;
          let to = { id: dropPort.dataset.id, dir: dropPort.dataset.dir };
          if (to.id !== from.id) {
            if (from.dir === 'in') { const t = from; from = to; to = t; }
            if (from.dir === 'out' && to.dir === 'in') {
              edges.push({ from: from.id, to: to.id });
              drawEdges();
            }
          }
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
    let ghost = null;
    item.addEventListener('pointerdown', (e) => {
      const type = item.dataset.type;
      ghost = document.createElement('div');
      ghost.className = 'awb-ghost';
      ghost.textContent = LABELS[type] || type;
      ghost.style.left = e.clientX + 'px';
      ghost.style.top = e.clientY + 'px';
      document.body.appendChild(ghost);
      item.setPointerCapture(e.pointerId);
    });
    item.addEventListener('pointermove', (e) => {
      if (!ghost) return;
      ghost.style.left = e.clientX + 'px';
      ghost.style.top = e.clientY + 'px';
    });
    const drop = (e) => {
      if (!ghost) return;
      ghost.remove();
      ghost = null;
      const type = item.dataset.type;
      const rect = canvasWrap.getBoundingClientRect();
      if (e.clientX >= rect.left && e.clientX <= rect.right && e.clientY >= rect.top && e.clientY <= rect.bottom) {
        const x = Math.max(0, e.clientX - rect.left - 45);
        const y = Math.max(0, e.clientY - rect.top - 20);
        makeNode(type, x, y);
      }
    };
    item.addEventListener('pointerup', drop);
    item.addEventListener('pointercancel', drop);
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

  const w = canvasWrap.clientWidth;
  const h = canvasWrap.clientHeight;
  const margin = 16;
  const x1 = margin;
  const x3 = Math.max(margin, w - margin - 100);
  const xm = Math.round((x1 + x3) / 2);
  makeNode('trigger', x1, margin);
  makeNode('llm', xm, Math.max(margin, Math.round(h / 2) - 20));
  makeNode('output', x3, margin);
  edges.push({ from: 'n1', to: 'n2' }, { from: 'n2', to: 'n3' });
  drawEdges();
})();
</script>

## Step details

Some visual thoughts on what each step needs under the hood. Nothing wires into the canvas above, but set the fields to see how each step's config would read.

<div class="awbd-grid">
  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#4a9eff"></span>Trigger</div>
    <div class="awbd-field-form">
      <span class="awbd-label">Kind</span>
      <select class="awbd-select" data-field="kind">
        <option>Schedule</option>
        <option>Calendar</option>
        <option>Event</option>
        <option>API call</option>
      </select>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Cadence</span>
      <select class="awbd-select" data-field="cadence">
        <option>Every hour</option>
        <option>Daily</option>
        <option>Weekly</option>
        <option>Custom cron</option>
      </select>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Auth (if API)</span>
      <div class="awbd-toggle" data-field="auth">
        <button type="button" class="awbd-toggle-btn active" data-value="API key">API key</button>
        <button type="button" class="awbd-toggle-btn" data-value="OAuth">OAuth</button>
        <button type="button" class="awbd-toggle-btn" data-value="Service account">Service account</button>
      </div>
    </div>
    <div class="awbd-summary"></div>
  </div>

  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#ff9f43"></span>Tool Call</div>
    <div class="awbd-field-form">
      <span class="awbd-label">Connection</span>
      <div class="awbd-toggle" data-field="connection">
        <button type="button" class="awbd-toggle-btn active" data-value="REST/GraphQL API">REST/GraphQL API</button>
        <button type="button" class="awbd-toggle-btn" data-value="MCP server">MCP server</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Auth</span>
      <div class="awbd-toggle" data-field="auth">
        <button type="button" class="awbd-toggle-btn active" data-value="API key">API key</button>
        <button type="button" class="awbd-toggle-btn" data-value="OAuth">OAuth</button>
        <button type="button" class="awbd-toggle-btn" data-value="Service account">Service account</button>
        <button type="button" class="awbd-toggle-btn" data-value="None">None</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Reliability</span>
      <select class="awbd-select" data-field="reliability">
        <option>No retries</option>
        <option>Retry 3x, 30s timeout</option>
        <option>Retry 5x, 60s timeout</option>
        <option>Custom</option>
      </select>
    </div>
    <div class="awbd-summary"></div>
  </div>

  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#b967ff"></span>LLM Call</div>
    <div class="awbd-field-form">
      <span class="awbd-label">Model</span>
      <select class="awbd-select" data-field="model">
        <option>Claude Sonnet 5</option>
        <option>Claude Opus 5</option>
        <option>GPT-5</option>
        <option>Local / self-hosted</option>
      </select>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Output</span>
      <div class="awbd-toggle" data-field="output">
        <button type="button" class="awbd-toggle-btn active" data-value="Freeform text">Freeform text</button>
        <button type="button" class="awbd-toggle-btn" data-value="Structured / JSON">Structured / JSON</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Fallback model</span>
      <div class="awbd-toggle" data-field="fallback">
        <button type="button" class="awbd-toggle-btn active" data-value="Off">Off</button>
        <button type="button" class="awbd-toggle-btn" data-value="On">On</button>
      </div>
    </div>
    <div class="awbd-summary"></div>
  </div>

  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#2ecc71"></span>Human Review</div>
    <div class="awbd-field-form">
      <span class="awbd-label">Channel</span>
      <div class="awbd-toggle" data-field="channel">
        <button type="button" class="awbd-toggle-btn active" data-value="Slack">Slack</button>
        <button type="button" class="awbd-toggle-btn" data-value="Email">Email</button>
        <button type="button" class="awbd-toggle-btn" data-value="In-app">In-app</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Actions allowed</span>
      <div class="awbd-toggle awbd-multi" data-field="actions">
        <button type="button" class="awbd-toggle-btn active" data-value="Approve">Approve</button>
        <button type="button" class="awbd-toggle-btn active" data-value="Reject">Reject</button>
        <button type="button" class="awbd-toggle-btn" data-value="Edit">Edit</button>
        <button type="button" class="awbd-toggle-btn" data-value="Redirect">Redirect</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">On timeout</span>
      <select class="awbd-select" data-field="timeout">
        <option>Auto-approve</option>
        <option>Escalate</option>
        <option>Block</option>
      </select>
    </div>
    <div class="awbd-summary"></div>
  </div>

  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#f1c40f"></span>Branch</div>
    <div class="awbd-field-form">
      <span class="awbd-label">Condition</span>
      <div class="awbd-toggle" data-field="condition">
        <button type="button" class="awbd-toggle-btn active" data-value="Rule-based">Rule-based</button>
        <button type="button" class="awbd-toggle-btn" data-value="LLM-judged">LLM-judged</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Merge</span>
      <select class="awbd-select" data-field="merge">
        <option>Wait for all paths</option>
        <option>First to complete</option>
        <option>Independent (no merge)</option>
      </select>
    </div>
    <div class="awbd-summary"></div>
  </div>

  <div class="awbd-card">
    <div class="awbd-head"><span class="awbd-dot" style="background:#8892a0"></span>Output</div>
    <div class="awbd-field-form">
      <span class="awbd-label">Destination</span>
      <select class="awbd-select" data-field="destination">
        <option>Return to caller</option>
        <option>Write to store</option>
        <option>Send message</option>
        <option>Trigger another workflow</option>
      </select>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Format</span>
      <div class="awbd-toggle" data-field="format">
        <button type="button" class="awbd-toggle-btn active" data-value="Match caller schema">Match caller schema</button>
        <button type="button" class="awbd-toggle-btn" data-value="Freeform">Freeform</button>
      </div>
    </div>
    <div class="awbd-field-form">
      <span class="awbd-label">Trace</span>
      <div class="awbd-toggle" data-field="trace">
        <button type="button" class="awbd-toggle-btn" data-value="Off">Off</button>
        <button type="button" class="awbd-toggle-btn active" data-value="On">On</button>
      </div>
    </div>
    <div class="awbd-summary"></div>
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
  .awbd-field-form {
    display: flex; flex-direction: column; gap: 0.3rem;
    padding: 0.45rem 0;
    border-top: 1px solid #1e1e22;
  }
  .awbd-field-form:first-of-type { border-top: 0; }
  .awbd-label {
    text-transform: uppercase; font-size: 0.62rem;
    letter-spacing: 0.06em; color: #888;
  }
  .awbd-select {
    background: #17171a; color: #fff;
    border: 1px solid #333; border-radius: 6px;
    padding: 0.35rem 0.4rem; font-size: 0.78rem;
    font-family: inherit; width: 100%;
  }
  .awbd-select:focus { outline: 1px solid #666; }
  .awbd-toggle {
    display: flex; flex-wrap: wrap; gap: 0.35rem;
  }
  .awbd-toggle-btn {
    background: #17171a; color: #ccc;
    border: 1px solid #333; border-radius: 999px;
    padding: 0.25rem 0.6rem; font-size: 0.72rem;
    cursor: pointer; font-family: inherit;
  }
  .awbd-toggle-btn:hover { border-color: #555; }
  .awbd-toggle-btn.active {
    background: #fff; color: #000; border-color: #fff;
  }
  .awbd-multi .awbd-toggle-btn.active {
    background: #2ecc71; color: #04150a; border-color: #2ecc71;
  }
  .awbd-summary {
    margin-top: 0.6rem; padding-top: 0.5rem;
    border-top: 1px dashed #262626;
    font-size: 0.72rem; color: #9fd6ff;
    min-height: 1.6em;
  }
</style>

<script>
(function() {
  function summarize(card) {
    const parts = [];
    card.querySelectorAll('[data-field]').forEach(field => {
      if (field.tagName === 'SELECT') {
        if (field.value) parts.push(field.value);
      } else if (field.classList.contains('awbd-multi')) {
        const actives = Array.from(field.querySelectorAll('.awbd-toggle-btn.active')).map(b => b.dataset.value);
        if (actives.length) parts.push(actives.join('/'));
      } else if (field.classList.contains('awbd-toggle')) {
        const active = field.querySelector('.awbd-toggle-btn.active');
        if (active) parts.push(active.dataset.value);
      }
    });
    const summaryEl = card.querySelector('.awbd-summary');
    if (summaryEl) summaryEl.textContent = parts.join(' · ');
  }

  document.querySelectorAll('.awbd-card').forEach(card => {
    card.querySelectorAll('.awbd-select').forEach(sel => {
      sel.addEventListener('change', () => summarize(card));
    });
    card.querySelectorAll('.awbd-toggle').forEach(group => {
      const multi = group.classList.contains('awbd-multi');
      group.addEventListener('click', (e) => {
        const btn = e.target.closest('.awbd-toggle-btn');
        if (!btn) return;
        if (multi) {
          btn.classList.toggle('active');
        } else {
          group.querySelectorAll('.awbd-toggle-btn').forEach(b => b.classList.remove('active'));
          btn.classList.add('active');
        }
        summarize(card);
      });
    });
    summarize(card);
  });
})();
</script>
