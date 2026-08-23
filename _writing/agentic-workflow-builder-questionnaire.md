---
layout: post
title: "Agentic Workflow Builder: Questionnaire"
date: 2026-08-16
tags: [Tech, AI, Side Project]
---

Before a workflow gets built, someone has to decide it's worth building. This is a look at a short, structured intake for that decision, a specialized version of "what do you want to build?" that turns a rough idea into requirements the [canvas](/writing/agentic-workflow-builder-canvas-prototype/) can start from.

## The concept

*[Building AI-Powered Products](https://www.oreilly.com/library/view/building-ai-powered-products/9781098152697/)* raises a handful of questions worth asking before any AI feature gets scoped. A few of them map cleanly onto a workflow-builder intake:

- **0 → 1 or 1 → n?** Is this making something possible that wasn't before, or improving something that already exists? A 0 → 1 product has no baseline to compare against, so "done" is fuzzier and there's more room to iterate. A 1 → n product is judged against what it's replacing, so the bar is concrete from day one.
- **What does AI actually enable here?** Learning from data, personalizing an experience, automating judgment or creative work, handling unstructured input — naming which one applies keeps the workflow honest about *why* it needs a probabilistic step at all, rather than reaching for an LLM call where a deterministic one would do.
- **RICE — Reach, Impact, Confidence, Effort.** Once there's more than one candidate workflow, RICE gives a way to rank them against each other. The score only means something in comparison to other ideas in the same backlog. It's not an absolute bar to clear.

Asking these up front is what turns "we should build an agent for this" into something with enough shape to hand to the canvas.

## The intake

A short questionnaire generates a starting brief: what kind of product this is, why AI is the right tool, a RICE score, and a suggested first pass at which step types to drop onto the canvas.

<div id="awq" class="awb-panel">
  <div class="awb-toolbar">
    <span class="awb-hint">Answer each step, then see the brief it produces.</span>
  </div>
  <div class="awq-progress">
    <span class="awq-dot active" data-step="0"></span>
    <span class="awq-dot" data-step="1"></span>
    <span class="awq-dot" data-step="2"></span>
    <span class="awq-dot" data-step="3"></span>
  </div>

  <div class="awq-step" data-step="0">
    <div class="awq-q">What kind of product is this?</div>
    <label class="awq-option">
      <input type="radio" name="awq-type" value="0to1" checked>
      <span>Something new — this wouldn't be possible without AI</span>
    </label>
    <label class="awq-option">
      <input type="radio" name="awq-type" value="1ton">
      <span>An upgrade — this improves something that already exists today</span>
    </label>
  </div>

  <div class="awq-step" data-step="1" hidden>
    <div class="awq-q">What is AI doing here? (pick any that apply)</div>
    <label class="awq-option">
      <input type="checkbox" name="awq-driver" value="data" checked>
      <span>Learning from data — personalization, insight, pattern-finding</span>
    </label>
    <label class="awq-option">
      <input type="checkbox" name="awq-driver" value="judgment">
      <span>Automating judgment or creative work a person used to do</span>
    </label>
    <label class="awq-option">
      <input type="checkbox" name="awq-driver" value="unstructured">
      <span>Handling unstructured or ambiguous input</span>
    </label>
    <label class="awq-option">
      <input type="checkbox" name="awq-driver" value="realtime">
      <span>Reacting in real time to something changing</span>
    </label>
  </div>

  <div class="awq-step" data-step="2" hidden>
    <div class="awq-q">Size up the idea</div>
    <div class="awq-rice-grid">
      <label class="awq-field">
        <span class="awq-label">How many people would use it?</span>
        <select class="awq-select" data-field="reach">
          <option value="lt100">A handful of people</option>
          <option value="100to1k">A few hundred</option>
          <option value="1kto10k" selected>A few thousand</option>
          <option value="gt10k">Tens of thousands or more</option>
        </select>
      </label>
      <label class="awq-field">
        <span class="awq-label">How big a difference would it make for them?</span>
        <select class="awq-select" data-field="impact">
          <option value="massive">Massive — changes what they can do</option>
          <option value="high" selected>High — meaningfully better</option>
          <option value="medium">Medium — a nice improvement</option>
          <option value="low">Low — a small tweak</option>
          <option value="minimal">Minimal — barely noticeable</option>
        </select>
      </label>
      <label class="awq-field">
        <span class="awq-label">How sure are you this will work?</span>
        <select class="awq-select" data-field="confidence">
          <option value="high">Very sure</option>
          <option value="medium" selected>Somewhat sure</option>
          <option value="low">Mostly a guess</option>
        </select>
      </label>
      <label class="awq-field">
        <span class="awq-label">How much work would it take to build?</span>
        <select class="awq-select" data-field="effort">
          <option value="tiny">A few days</option>
          <option value="small">A couple weeks</option>
          <option value="medium" selected>A month or so</option>
          <option value="large">A few months</option>
          <option value="huge">Half a year or more</option>
        </select>
      </label>
    </div>
  </div>

  <div class="awq-step" data-step="3" hidden>
    <div class="awq-brief"></div>
  </div>

  <div class="awq-nav">
    <button type="button" class="awq-back" disabled>Back</button>
    <button type="button" class="awq-next">Next</button>
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
  .awb-toolbar { margin-bottom: 0.75rem; }
  .awb-hint { color: #999; font-size: 0.8rem; max-width: 60ch; }
  .awq-progress { display: flex; gap: 0.4rem; margin-bottom: 1rem; }
  .awq-dot { width: 24px; height: 4px; border-radius: 999px; background: #2a2a2e; }
  .awq-dot.active { background: #4a9eff; }
  .awq-dot.done { background: #3a5f8a; }
  .awq-q { font-size: 0.95rem; font-weight: 600; margin-bottom: 0.9rem; }
  .awq-option {
    display: flex; align-items: flex-start; gap: 0.5rem;
    background: #141416; border: 1px solid #232327; border-radius: 6px;
    padding: 0.6rem 0.7rem; margin-bottom: 0.5rem;
    font-size: 0.8rem; color: #ddd; cursor: pointer;
  }
  .awq-option input { margin-top: 0.2rem; flex-shrink: 0; }
  .awq-rice-grid {
    display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
    gap: 0.7rem;
  }
  .awq-field { display: flex; flex-direction: column; gap: 0.3rem; }
  .awq-label {
    text-transform: uppercase; font-size: 0.62rem;
    letter-spacing: 0.06em; color: #888;
  }
  .awq-select {
    background: #17171a; color: #fff;
    border: 1px solid #333; border-radius: 6px;
    padding: 0.4rem 0.5rem; font-size: 0.8rem; font-family: inherit;
  }
  .awq-nav {
    display: flex; justify-content: space-between; gap: 0.75rem;
    margin-top: 1rem; padding-top: 0.9rem;
    border-top: 1px solid #1e1e22;
  }
  .awq-nav button {
    background: #fff; color: #000; border: 0;
    border-radius: 999px; padding: 0.45rem 1.1rem;
    font-weight: 600; font-size: 0.85rem;
    cursor: pointer; font-family: inherit;
  }
  .awq-back { background: transparent !important; color: #999 !important; border: 1px solid #444 !important; }
  .awq-back:disabled { opacity: 0.4; cursor: default; }
  .awq-brief { font-size: 0.85rem; line-height: 1.6; color: #ddd; }
  .awq-brief h4 {
    font-size: 0.68rem; text-transform: uppercase; letter-spacing: 0.06em;
    color: #888; margin: 1rem 0 0.4rem;
  }
  .awq-brief h4:first-child { margin-top: 0; }
  .awq-chips { display: flex; flex-wrap: wrap; gap: 0.4rem; }
  .awq-chip {
    display: inline-block; font-size: 0.75rem;
    padding: 0.25rem 0.65rem; border-radius: 999px;
    border: 1px solid var(--accent, #444); color: #fff;
  }
  .awq-score { font-size: 1.4rem; font-weight: 700; }
  .awq-score-note { font-size: 0.72rem; color: #888; }
</style>

<script>
(function() {
  const root = document.getElementById('awq');
  const steps = root.querySelectorAll('.awq-step');
  const dots = root.querySelectorAll('.awq-dot');
  const backBtn = root.querySelector('.awq-back');
  const nextBtn = root.querySelector('.awq-next');
  const briefEl = root.querySelector('.awq-brief');

  const STEP_LABELS = {
    trigger: 'Trigger', llm: 'LLM Call', tool: 'Tool Call',
    human: 'Human Review', branch: 'Branch', output: 'Output'
  };
  const STEP_COLORS = {
    trigger: '#4a9eff', tool: '#ff9f43', llm: '#b967ff',
    human: '#2ecc71', branch: '#f1c40f', output: '#8892a0'
  };

  const REACH = { lt100: 50, '100to1k': 500, '1kto10k': 5000, gt10k: 20000 };
  const IMPACT = { massive: 3, high: 2, medium: 1, low: 0.5, minimal: 0.25 };
  const CONFIDENCE = { high: 1.0, medium: 0.8, low: 0.5 };
  const EFFORT = { tiny: 0.25, small: 0.5, medium: 1, large: 3, huge: 6 };

  let current = 0;

  function goTo(step) {
    current = step;
    steps.forEach(s => { s.hidden = Number(s.dataset.step) !== step; });
    dots.forEach(d => {
      const n = Number(d.dataset.step);
      d.classList.toggle('active', n === step);
      d.classList.toggle('done', n < step);
    });
    backBtn.disabled = step === 0;
    nextBtn.textContent = step === steps.length - 1 ? 'Start over' : (step === steps.length - 2 ? 'See brief' : 'Next');
    if (step === steps.length - 1) renderBrief();
  }

  function readState() {
    const type = root.querySelector('input[name="awq-type"]:checked').value;
    const drivers = Array.from(root.querySelectorAll('input[name="awq-driver"]:checked')).map(el => el.value);
    const reach = root.querySelector('[data-field="reach"]').value;
    const impact = root.querySelector('[data-field="impact"]').value;
    const confidence = root.querySelector('[data-field="confidence"]').value;
    const effort = root.querySelector('[data-field="effort"]').value;
    return { type, drivers, reach, impact, confidence, effort };
  }

  function priorityScore(state) {
    return (REACH[state.reach] * IMPACT[state.impact] * CONFIDENCE[state.confidence]) / EFFORT[state.effort];
  }

  function suggestSteps(state) {
    const chips = ['trigger'];
    if (state.drivers.includes('data')) chips.push('tool');
    if (state.drivers.includes('judgment') || state.drivers.includes('unstructured')) chips.push('llm');
    if (state.confidence === 'low' || state.effort === 'large' || state.effort === 'huge') chips.push('human');
    chips.push('output');
    return [...new Set(chips)];
  }

  function chipHtml(type) {
    return '<span class="awq-chip" style="--accent:' + STEP_COLORS[type] + '">' + STEP_LABELS[type] + '</span>';
  }

  function renderBrief() {
    const state = readState();
    const score = priorityScore(state);
    const driverLabels = {
      data: 'learning from data', judgment: 'automating judgment or creative work',
      unstructured: 'handling unstructured input', realtime: 'reacting in real time'
    };
    const driverText = state.drivers.length
      ? state.drivers.map(d => driverLabels[d]).join(', ')
      : 'no clear AI-specific driver selected — worth revisiting whether this needs AI at all';
    const chips = suggestSteps(state);

    briefEl.innerHTML =
      '<h4>What this is</h4>' +
      '<div>' + (state.type === '0to1'
        ? 'Something new — there\'s no existing baseline, so success criteria will need to be defined as you go.'
        : 'An upgrade — judged against what it replaces, so the bar is concrete from day one.') + '</div>' +
      '<h4>Why AI</h4>' +
      '<div>' + driverText + '</div>' +
      '<h4>Priority score</h4>' +
      '<div class="awq-score">' + Math.round(score).toLocaleString() + '</div>' +
      '<div class="awq-score-note">Only means something compared to other ideas on your list, not as a bar to clear on its own.</div>' +
      '<h4>Suggested starting point for the canvas</h4>' +
      '<div class="awq-chips">' + chips.map(chipHtml).join('') + '</div>';
  }

  backBtn.addEventListener('click', () => { if (current > 0) goTo(current - 1); });
  nextBtn.addEventListener('click', () => {
    if (current === steps.length - 1) { goTo(0); return; }
    goTo(current + 1);
  });

  goTo(0);
})();
</script>
