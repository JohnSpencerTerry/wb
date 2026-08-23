---
layout: post
title: "A feature request for the Brooklyn Public Library hold shelf."
date: 2026-08-22
category: Software Engineering
draft: false
---

I use the Brooklyn Public Library a lot. When you request an item and it becomes available, it gets pulled and held on a physical shelf inside the branch, with a **Pickup By** date. If you miss that date, the hold is canceled and it goes back into circulation.

BPL branches keep limited hours: closed Sundays, shorter days midweek. A trip out of town or a stretch of busy days can make it difficult to get there in time.

**Pause Hold** is only available before an item reaches the shelf. Once it's on the shelf, the only options are to pick it up or cancel the hold outright.

<figure class="media-figure">
  <img src="/assets/photos/bpl/pause-hold-error.png" alt="BPL error message: Cannot modify hold because the item is already on the holdshelf or in transit." />
</figure>

Canceling sends you back to the end of the list, waiting for the copy to come around again.

## The feature: Extend Hold

An **Extend Hold** button, next to Pause Hold and Cancel Hold on any item marked "Ready for Pickup":

- Clicking it pushes the **Pickup By** date out by 5 days.
- It's only available if no other patron is waiting on the item, that is, there's no one behind you in line for that copy. If someone's waiting, extending your hold delays their pickup, so the button shouldn't be actionable.
- The 5-day count skips days the library is closed, Sundays and holidays, so a patron actually gets 5 open days to make it in, not 5 calendar days that might include two closed ones.
- It can only be used once per item. Hold shelves have finite physical space, and letting the same item get extended over and over would be burdensome.

## Try it

A rough recreation of the pickup card, built to match BPL's own UI.

<div id="bpl-card" class="bpl-panel">
  <div class="bpl-toggle-row">
    <label class="bpl-toggle">
      <input type="checkbox" id="bpl-waitlist-toggle">
      <span>Simulate another patron waiting on this item</span>
    </label>
  </div>

  <div class="bpl-card">
    <div class="bpl-card-header">Ready for Pickup at Leonard</div>
    <div class="bpl-item-row">
      <div class="bpl-cover">DVD</div>
      <div class="bpl-item-info">
        <div class="bpl-item-title">Good Time</div>
        <div class="bpl-item-meta">DVD &middot; 2 copies owned</div>
        <div class="bpl-pickup-row">
          <span class="bpl-pickup-label">Pickup By:</span>
          <span class="bpl-badge" id="bpl-pickup-date"></span>
        </div>
      </div>
    </div>
    <button type="button" class="bpl-btn bpl-btn-primary" id="bpl-extend-btn">Extend Hold</button>
    <div class="bpl-extend-note" id="bpl-extend-note"></div>
    <button type="button" class="bpl-btn bpl-btn-primary bpl-btn-disabled" id="bpl-pause-btn">Pause Hold</button>
    <button type="button" class="bpl-btn-link" id="bpl-cancel-btn">Cancel Hold</button>
  </div>
</div>

<style>
  .bpl-panel {
    --bpl-primary: #4b4fa0;
    --bpl-primary-dark: #2f2d6e;
    --bpl-badge-bg: #e3e5fb;
    --bpl-badge-text: #2f2d6e;
    --bpl-surface: #ffffff;
    --bpl-page: #eef0f4;
    --bpl-muted: #6b6b76;
    --bpl-link: #2f3fae;
    background: var(--bpl-page);
    border-radius: 20px;
    padding: 1.75rem 1.25rem;
    margin: 1.5rem 0;
    font-family: system-ui, sans-serif;
  }
  .bpl-toggle-row { display: flex; justify-content: center; margin-bottom: 1.25rem; }
  .bpl-toggle {
    display: flex; align-items: center; gap: 0.5rem;
    font-size: 0.8rem; color: var(--bpl-muted); font-weight: 600;
    cursor: pointer;
  }
  .bpl-card {
    background: var(--bpl-surface);
    border-radius: 18px;
    padding: 1.75rem 1.5rem;
    max-width: 340px;
    margin: 0 auto;
    box-shadow: 0 2px 10px rgba(47, 45, 110, 0.08);
    text-align: center;
  }
  .bpl-card-header {
    color: var(--bpl-primary-dark);
    font-weight: 800;
    font-size: 0.85rem;
    text-transform: uppercase;
    letter-spacing: 0.03em;
    padding-bottom: 1rem;
    margin-bottom: 1.25rem;
    border-bottom: 1px solid #e6e6ea;
  }
  .bpl-item-row { display: flex; gap: 1rem; text-align: left; margin-bottom: 1.5rem; }
  .bpl-cover {
    flex: none;
    width: 64px; height: 90px;
    background: #1a1a1d;
    color: #fff;
    border-radius: 6px;
    display: flex; align-items: center; justify-content: center;
    font-size: 0.7rem; font-weight: 700;
  }
  .bpl-item-title { color: var(--bpl-link); font-weight: 700; font-size: 1.05rem; }
  .bpl-item-meta { color: var(--bpl-muted); font-size: 0.82rem; margin-top: 0.3rem; }
  .bpl-pickup-row { margin-top: 0.7rem; font-size: 0.85rem; }
  .bpl-pickup-label { color: #1a1a1d; font-weight: 700; margin-right: 0.4rem; }
  .bpl-badge {
    display: inline-block;
    background: var(--bpl-badge-bg);
    color: var(--bpl-badge-text);
    font-weight: 700;
    font-size: 0.8rem;
    padding: 0.25rem 0.7rem;
    border-radius: 999px;
  }
  .bpl-btn {
    display: block;
    width: 100%;
    border: 0;
    border-radius: 999px;
    padding: 0.8rem 1rem;
    font-weight: 700;
    font-size: 0.95rem;
    cursor: pointer;
    font-family: inherit;
    margin-bottom: 0.75rem;
  }
  .bpl-btn-primary { background: var(--bpl-primary); color: #fff; }
  .bpl-btn-primary:hover:not(:disabled) { background: var(--bpl-primary-dark); }
  .bpl-btn:disabled, .bpl-btn-disabled {
    background: #cfd0e0 !important;
    color: #7f7f8c !important;
    cursor: not-allowed;
  }
  .bpl-btn-link {
    background: none; border: 0; color: var(--bpl-link);
    font-weight: 700; font-size: 0.9rem;
    text-decoration: underline; cursor: pointer; font-family: inherit;
  }
  .bpl-extend-note {
    font-size: 0.78rem; color: var(--bpl-muted);
    margin: -0.35rem 0 0.9rem;
    min-height: 1.4em;
  }
</style>

<script>
(function() {
  const HOLIDAYS = ['2026-09-07']; // Labor Day, for demo purposes

  function isClosed(date) {
    const iso = date.toISOString().slice(0, 10);
    return date.getDay() === 0 || HOLIDAYS.includes(iso);
  }

  function addOpenDays(date, days) {
    const result = new Date(date);
    let added = 0;
    while (added < days) {
      result.setDate(result.getDate() + 1);
      if (!isClosed(result)) added++;
    }
    return result;
  }

  function formatDate(date) {
    return (date.getMonth() + 1) + '/' + date.getDate() + '/' + date.getFullYear();
  }

  const waitlistToggle = document.getElementById('bpl-waitlist-toggle');
  const extendBtn = document.getElementById('bpl-extend-btn');
  const pauseBtn = document.getElementById('bpl-pause-btn');
  const extendNote = document.getElementById('bpl-extend-note');
  const pickupDateEl = document.getElementById('bpl-pickup-date');

  // start one open day out, matching a freshly-arrived hold
  let pickupDate = addOpenDays(new Date(), 1);
  let extended = false;

  function render() {
    pickupDateEl.textContent = formatDate(pickupDate);
    const hasWaitlist = waitlistToggle.checked;

    if (extended) {
      extendBtn.disabled = true;
      extendBtn.textContent = 'Hold Extended';
      extendNote.textContent = 'This item can only be extended once.';
    } else if (hasWaitlist) {
      extendBtn.disabled = true;
      extendBtn.textContent = 'Extend Hold';
      extendNote.textContent = 'Unavailable — another patron is waiting for this item.';
    } else {
      extendBtn.disabled = false;
      extendBtn.textContent = 'Extend Hold';
      extendNote.textContent = '';
    }
  }

  extendBtn.addEventListener('click', () => {
    if (extendBtn.disabled) return;
    pickupDate = addOpenDays(pickupDate, 5);
    extended = true;
    render();
  });

  waitlistToggle.addEventListener('change', render);
  pauseBtn.addEventListener('click', () => {
    extendNote.textContent = 'Cannot modify hold because the item is already on the holdshelf or in transit.';
  });

  render();
})();
</script>
