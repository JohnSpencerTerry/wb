---
layout: post
title: "Agentic Workflow Builder: Test Bench"
date: 2026-08-16
---

An LLM call doesn't always return the same output twice, so you can't assert on specific outputs like in traditional software testing. As a follow up to the [canvas prototype](/side-projects/agentic-workflow-builder-canvas-prototype/), here are some thoughts on what testing a single step might look like.

## The concept

Deterministic steps such as a tool call or a trigger firing can be tested the way you'd test any function: run it, assert the output matches exactly. LLM steps are probabilistic, so the assertion has to change shape too: does the output *contain* the right information, does it match a schema, is it *semantically* correct?

That's the core idea for a test bench: each step type gets assertion types that fit whether its output is deterministic or probabilistic.

## The bench

Pick a step type below, edit its mocked output, then run the assertions against it. LLM Call is the only type with a "Semantic match" option — a real system would use an AI model to judge meaning; this demo just checks for overlapping keywords as a rough stand-in.

<div id="awt" class="awb-panel">
  <div class="awb-toolbar">
    <span class="awb-hint">Pick a step type, edit its mocked output, then run the test suite against it.</span>
  </div>
  <div class="awt-types">
    <button type="button" class="awt-type-btn active" style="--accent:#4a9eff" data-type="trigger">Trigger</button>
    <button type="button" class="awt-type-btn" style="--accent:#b967ff" data-type="llm">LLM Call</button>
    <button type="button" class="awt-type-btn" style="--accent:#ff9f43" data-type="tool">Tool Call</button>
    <button type="button" class="awt-type-btn" style="--accent:#2ecc71" data-type="human">Human Review</button>
    <button type="button" class="awt-type-btn" style="--accent:#f1c40f" data-type="branch">Branch</button>
    <button type="button" class="awt-type-btn" style="--accent:#8892a0" data-type="output">Output</button>
  </div>
  <div class="awt-panel-inner">
    <label class="awt-field">
      <span class="awt-label">Mocked output</span>
      <textarea class="awt-output" rows="3"></textarea>
    </label>
    <div class="awt-tests"></div>
    <button type="button" class="awt-add">+ Add assertion</button>
    <div class="awt-actions">
      <button type="button" class="awt-run">Run Tests</button>
      <span class="awt-summary"></span>
    </div>
  </div>
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
  .awb-toolbar { margin-bottom: 0.9rem; }
  .awb-hint { color: #999; font-size: 0.8rem; max-width: 60ch; }
  .awt-types { display: flex; flex-wrap: wrap; gap: 0.5rem; margin-bottom: 1rem; }
  .awt-type-btn {
    background: #17171a; color: #ccc;
    border: 1px solid var(--accent, #333);
    border-radius: 999px;
    padding: 0.35rem 0.75rem; font-size: 0.78rem;
    cursor: pointer; font-family: inherit;
  }
  .awt-type-btn.active {
    background: var(--accent); color: #0c0c0e;
    border-color: var(--accent); font-weight: 600;
  }
  .awt-panel-inner {
    border: 1px solid #26262a; border-radius: 8px;
    padding: 0.9rem 1rem;
    background: #0b0b0d;
  }
  .awt-field { display: flex; flex-direction: column; gap: 0.3rem; margin-bottom: 0.9rem; }
  .awt-label, .awt-row-label {
    text-transform: uppercase; font-size: 0.62rem;
    letter-spacing: 0.06em; color: #888;
  }
  .awt-output {
    background: #17171a; color: #fff;
    border: 1px solid #333; border-radius: 6px;
    padding: 0.5rem 0.6rem; font-size: 0.8rem;
    font-family: ui-monospace, monospace;
    resize: vertical;
  }
  .awt-tests { display: flex; flex-direction: column; gap: 0.5rem; margin-bottom: 0.6rem; }
  .awt-row {
    display: flex; flex-wrap: wrap; align-items: center; gap: 0.4rem;
    background: #141416; border: 1px solid #232327; border-radius: 6px;
    padding: 0.4rem 0.5rem;
  }
  .awt-row select, .awt-row input[type="text"] {
    background: #1e1e22; color: #fff;
    border: 1px solid #333; border-radius: 5px;
    padding: 0.3rem 0.4rem; font-size: 0.76rem; font-family: inherit;
  }
  .awt-row input[type="text"] { flex: 1; min-width: 120px; }
  .awt-row-remove {
    background: transparent; color: #888; border: 0;
    cursor: pointer; font-size: 0.9rem; padding: 0 0.3rem;
  }
  .awt-row-remove:hover { color: #f08; }
  .awt-row-badge {
    font-size: 0.68rem; font-weight: 600;
    padding: 0.1rem 0.5rem; border-radius: 999px;
    min-width: 3.6em; text-align: center;
  }
  .awt-row-badge.pass { background: rgba(46,204,113,0.15); color: #2ecc71; }
  .awt-row-badge.fail { background: rgba(231,76,60,0.15); color: #e74c3c; }
  .awt-add {
    background: transparent; color: #999; border: 1px dashed #444;
    border-radius: 6px; padding: 0.35rem 0.7rem; font-size: 0.76rem;
    cursor: pointer; font-family: inherit; align-self: flex-start;
  }
  .awt-add:hover { color: #fff; border-color: #666; }
  .awt-actions {
    display: flex; align-items: center; gap: 0.75rem;
    margin-top: 0.9rem; padding-top: 0.75rem;
    border-top: 1px solid #1e1e22;
  }
  .awt-run {
    background: #fff; color: #000; border: 0;
    border-radius: 999px; padding: 0.45rem 1.1rem;
    font-weight: 600; font-size: 0.85rem;
    cursor: pointer; font-family: inherit;
  }
  .awt-summary { font-size: 0.8rem; color: #999; }
</style>

<script>
(function() {
  const root = document.getElementById('awt');
  const typeBtns = root.querySelectorAll('.awt-type-btn');
  const outputEl = root.querySelector('.awt-output');
  const testsEl = root.querySelector('.awt-tests');
  const addBtn = root.querySelector('.awt-add');
  const runBtn = root.querySelector('.awt-run');
  const summaryEl = root.querySelector('.awt-summary');

  const ASSERTION_LABELS = {
    equals: 'Equals (exact)',
    contains: 'Contains',
    schema: 'Has JSON keys',
    semantic: 'Semantic match (simulated)'
  };

  const STEP_DEFAULTS = {
    trigger: {
      output: '{"event":"order.created","order_id":"A1029"}',
      types: ['equals', 'contains', 'schema'],
      tests: [
        { type: 'schema', value: 'event, order_id' },
        { type: 'contains', value: 'order.created' }
      ]
    },
    llm: {
      output: 'Thanks for reaching out! I can help you track order A1029 — it shipped yesterday and should arrive by Friday.',
      types: ['contains', 'schema', 'semantic'],
      tests: [
        { type: 'contains', value: 'A1029' },
        { type: 'semantic', value: 'when will the order arrive' }
      ]
    },
    tool: {
      output: '{"status":200,"tracking":"1Z999AA10123456784"}',
      types: ['equals', 'contains', 'schema'],
      tests: [
        { type: 'schema', value: 'status, tracking' },
        { type: 'contains', value: '200' }
      ]
    },
    human: {
      output: 'approved',
      types: ['equals', 'contains'],
      tests: [
        { type: 'equals', value: 'approved' }
      ]
    },
    branch: {
      output: 'route: escalate',
      types: ['equals', 'contains'],
      tests: [
        { type: 'contains', value: 'escalate' }
      ]
    },
    output: {
      output: '{"reply":"Your order ships Friday.","status":"sent"}',
      types: ['equals', 'contains', 'schema'],
      tests: [
        { type: 'schema', value: 'reply, status' },
        { type: 'contains', value: 'sent' }
      ]
    }
  };

  let currentType = 'trigger';
  let tests = [];

  function evaluate(assertionType, output, value) {
    switch (assertionType) {
      case 'equals':
        return output.trim() === value.trim();
      case 'contains':
        return output.toLowerCase().includes(value.toLowerCase());
      case 'schema': {
        try {
          const obj = JSON.parse(output);
          return value.split(',').map(s => s.trim()).filter(Boolean).every(k => k in obj);
        } catch (e) {
          return false;
        }
      }
      case 'semantic': {
        const words = s => new Set((s.toLowerCase().match(/[a-z0-9]+/g) || []).filter(w => w.length > 2));
        const have = words(output), want = words(value);
        if (!want.size) return false;
        let overlap = 0;
        want.forEach(w => { if (have.has(w)) overlap++; });
        return overlap / want.size >= 0.35;
      }
      default:
        return false;
    }
  }

  function renderTests() {
    const typeOptions = STEP_DEFAULTS[currentType].types;
    testsEl.innerHTML = '';
    tests.forEach((t, i) => {
      const row = document.createElement('div');
      row.className = 'awt-row';
      const select = document.createElement('select');
      typeOptions.forEach(opt => {
        const o = document.createElement('option');
        o.value = opt;
        o.textContent = ASSERTION_LABELS[opt];
        if (opt === t.type) o.selected = true;
        select.appendChild(o);
      });
      select.addEventListener('change', () => { tests[i].type = select.value; });

      const input = document.createElement('input');
      input.type = 'text';
      input.value = t.value;
      input.placeholder = 'expected value';
      input.addEventListener('input', () => { tests[i].value = input.value; });

      const badge = document.createElement('span');
      badge.className = 'awt-row-badge';

      const remove = document.createElement('button');
      remove.type = 'button';
      remove.className = 'awt-row-remove';
      remove.innerHTML = '&times;';
      remove.addEventListener('click', () => {
        tests.splice(i, 1);
        renderTests();
        summaryEl.textContent = '';
      });

      row.appendChild(select);
      row.appendChild(input);
      row.appendChild(badge);
      row.appendChild(remove);
      testsEl.appendChild(row);
    });
  }

  function selectType(type) {
    currentType = type;
    typeBtns.forEach(b => b.classList.toggle('active', b.dataset.type === type));
    const def = STEP_DEFAULTS[type];
    outputEl.value = def.output;
    tests = def.tests.map(t => ({ ...t }));
    renderTests();
    summaryEl.textContent = '';
  }

  typeBtns.forEach(btn => {
    btn.addEventListener('click', () => selectType(btn.dataset.type));
  });

  addBtn.addEventListener('click', () => {
    const types = STEP_DEFAULTS[currentType].types;
    tests.push({ type: types[0], value: '' });
    renderTests();
  });

  runBtn.addEventListener('click', () => {
    const output = outputEl.value;
    const badges = testsEl.querySelectorAll('.awt-row-badge');
    let passCount = 0;
    tests.forEach((t, i) => {
      const ok = evaluate(t.type, output, t.value);
      if (ok) passCount++;
      const badge = badges[i];
      badge.textContent = ok ? 'PASS' : 'FAIL';
      badge.className = 'awt-row-badge ' + (ok ? 'pass' : 'fail');
    });
    summaryEl.textContent = tests.length
      ? passCount + ' / ' + tests.length + ' passing'
      : 'No assertions yet.';
    summaryEl.style.color = (tests.length && passCount === tests.length) ? '#2ecc71' : '#999';
  });

  selectType('trigger');
})();
</script>
