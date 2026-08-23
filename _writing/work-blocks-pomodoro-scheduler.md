---
layout: post
title: "Work Blocks: A Pomodoro Scheduler"
date: 2026-06-16
tags: [Tech, Side Project]
---

Early in covid, a lot of people were suddenly working remote with a workday that had bookends but no structure in between. My wife and I built Work Blocks, an app that scheduled pomodoro-style focus blocks around whatever was already fixed in your day.

## The concept

Set your work hours. The app reads Google Calendar for meetings and other unmovable blocks. Everything else is open for "workblocks" — focus chunks with breaks between them.

A task like "Write report" estimated at two and a half hours becomes six 25-minute workblocks with 10-minute breaks (both configurable), scheduled into whatever open time is left in the day. Reminders and start/stop handle the rest.

Built in Python with tkinter.

## The tech

[tkinter](https://docs.python.org/3/library/tkinter.html) is event-driven, which requires structuring the app around a `mainloop` dispatching to callbacks instead of running top-to-bottom. Storage was just pickled Python objects to disk; no database, since the app didn't need one. Calendar reads went through Google's OAuth, which meant registering Work Blocks as a Google app to get credentials.

## The scheduling logic

Given work hours, a set of fixed meetings, and a task with a duration, the scheduler finds the open gaps in the day and packs workblocks (plus breaks) into them in order. That's the core of the app, and the part worth demoing.

## Try it

This is a quick recreation in JS to demo the scheduling logic, not the original app. Set work hours, add or edit a couple of meetings, give it a task and a duration, and hit Schedule.

<div id="wb" class="wb-panel">
  <div class="wb-controls">
    <div class="wb-row">
      <label>Work hours
        <input type="time" class="wb-day-start" value="09:00">
        to
        <input type="time" class="wb-day-end" value="17:00">
      </label>
    </div>
    <div class="wb-meetings">
      <div class="wb-meeting-row">
        <input type="checkbox" class="wb-meeting-on" checked>
        <input type="text" class="wb-meeting-title" value="Standup">
        <input type="time" class="wb-meeting-start" value="09:00">
        <input type="time" class="wb-meeting-end" value="09:15">
      </div>
      <div class="wb-meeting-row">
        <input type="checkbox" class="wb-meeting-on" checked>
        <input type="text" class="wb-meeting-title" value="Planning">
        <input type="time" class="wb-meeting-start" value="13:00">
        <input type="time" class="wb-meeting-end" value="14:00">
      </div>
    </div>
    <div class="wb-row">
      <label>Task <input type="text" class="wb-task-name" value="Write report"></label>
      <label>Hours <input type="number" class="wb-task-hours" value="3" min="0.5" max="12" step="0.5"></label>
    </div>
    <div class="wb-row">
      <label>Block (min) <input type="number" class="wb-block-len" value="25" min="5" max="90" step="5"></label>
      <label>Break (min) <input type="number" class="wb-break-len" value="10" min="0" max="30" step="5"></label>
    </div>
    <button type="button" class="wb-schedule">Schedule</button>
  </div>
  <div class="wb-timeline-wrap">
    <div class="wb-timeline"></div>
    <div class="wb-ticks"></div>
  </div>
  <div class="wb-legend">
    <span><i class="wb-swatch wb-swatch-meeting"></i>Meeting</span>
    <span><i class="wb-swatch wb-swatch-block"></i>Workblock</span>
    <span><i class="wb-swatch wb-swatch-break"></i>Break</span>
    <span><i class="wb-swatch wb-swatch-free"></i>Free</span>
  </div>
  <div class="wb-status"></div>
</div>

Drag the slider to see what the app's current-block view would show at that point in the schedule.

<div id="wb-now" class="wb-panel">
  <div class="wb-row">
    <label class="wb-now-time-label">Simulated time: <strong class="wb-now-time-value"></strong>
      <input type="range" class="wb-now-time" min="0" max="1439" step="5" value="600">
    </label>
  </div>
  <div class="wb-now-card">
    <div class="wb-now-kind"></div>
    <div class="wb-now-detail"></div>
  </div>
</div>

<style>
  .wb-panel {
    background: #0e0e10;
    color: #fff;
    border-radius: 10px;
    padding: 1.25rem 1.25rem 1.5rem;
    margin: 1.5rem 0;
    font-family: system-ui, sans-serif;
  }
  .wb-controls { display: flex; flex-direction: column; gap: 0.65rem; }
  .wb-row { display: flex; flex-wrap: wrap; gap: 1rem; font-size: 0.85rem; color: #ccc; }
  .wb-row label { display: flex; align-items: center; gap: 0.4rem; }
  .wb-meetings { display: flex; flex-direction: column; gap: 0.4rem; }
  .wb-meeting-row {
    display: flex; align-items: center; gap: 0.5rem;
    font-size: 0.85rem; color: #ccc;
  }
  .wb-meeting-title { width: 8rem; }
  .wb-panel input[type="text"],
  .wb-panel input[type="number"],
  .wb-panel input[type="time"] {
    background: #17171a; color: #fff;
    border: 1px solid #333; border-radius: 6px;
    padding: 0.3rem 0.5rem; font-size: 0.82rem;
    font-family: inherit;
  }
  .wb-panel input[type="number"] { width: 4rem; }
  .wb-panel input[type="checkbox"] { width: 1rem; height: 1rem; }
  .wb-schedule {
    align-self: flex-start;
    background: #fff; color: #000; border: 0;
    border-radius: 999px; padding: 0.5rem 1.25rem;
    font-weight: 600; font-size: 0.85rem;
    cursor: pointer; font-family: inherit;
    margin-top: 0.25rem;
  }
  .wb-timeline-wrap { margin-top: 1.25rem; }
  .wb-timeline {
    position: relative;
    height: 48px;
    background: #1a1a1d;
    border-radius: 6px;
    overflow: hidden;
  }
  .wb-seg {
    position: absolute; top: 0; bottom: 0;
    border-right: 1px solid #0e0e10;
  }
  .wb-seg-meeting { background: #555b66; }
  .wb-seg-block { background: #2ecc71; }
  .wb-seg-break { background: #f1c40f; }
  .wb-ticks {
    position: relative; height: 1.1rem;
    font-size: 0.68rem; color: #888;
  }
  .wb-tick { position: absolute; transform: translateX(-50%); }
  .wb-legend {
    display: flex; flex-wrap: wrap; gap: 1rem;
    margin-top: 0.75rem; font-size: 0.75rem; color: #aaa;
  }
  .wb-legend span { display: flex; align-items: center; gap: 0.4rem; }
  .wb-swatch { width: 10px; height: 10px; border-radius: 2px; display: inline-block; }
  .wb-swatch-meeting { background: #555b66; }
  .wb-swatch-block { background: #2ecc71; }
  .wb-swatch-break { background: #f1c40f; }
  .wb-swatch-free { background: #1a1a1d; border: 1px solid #333; }
  .wb-status { margin-top: 0.75rem; font-size: 0.82rem; color: #9fd6ff; min-height: 1.2em; }
  .wb-now-marker {
    position: absolute; top: -5px; bottom: -5px; width: 2px;
    background: #fff; box-shadow: 0 0 5px rgba(255,255,255,0.7);
  }
  .wb-now-time-label {
    display: flex; align-items: center; gap: 0.6rem;
    font-size: 0.85rem; color: #ccc; width: 100%;
  }
  .wb-now-time { flex: 1; }
  .wb-now-card {
    margin-top: 1rem; padding: 0.85rem 1rem;
    background: #17171a; border: 1px solid #333; border-radius: 8px;
  }
  .wb-now-kind { font-size: 0.95rem; font-weight: 600; }
  .wb-now-detail { margin-top: 0.2rem; font-size: 0.82rem; color: #9fd6ff; }
  @media (max-width: 560px) {
    .wb-row { flex-direction: column; gap: 0.5rem; }
    .wb-meeting-row { flex-wrap: wrap; }
  }
</style>

<script>
(function() {
  const root = document.getElementById('wb');
  const dayStartEl = root.querySelector('.wb-day-start');
  const dayEndEl = root.querySelector('.wb-day-end');
  const meetingRows = Array.from(root.querySelectorAll('.wb-meeting-row'));
  const taskNameEl = root.querySelector('.wb-task-name');
  const taskHoursEl = root.querySelector('.wb-task-hours');
  const blockLenEl = root.querySelector('.wb-block-len');
  const breakLenEl = root.querySelector('.wb-break-len');
  const scheduleBtn = root.querySelector('.wb-schedule');
  const timelineEl = root.querySelector('.wb-timeline');
  const ticksEl = root.querySelector('.wb-ticks');
  const statusEl = root.querySelector('.wb-status');

  const nowRoot = document.getElementById('wb-now');
  const nowTimeEl = nowRoot.querySelector('.wb-now-time');
  const nowTimeValueEl = nowRoot.querySelector('.wb-now-time-value');
  const nowKindEl = nowRoot.querySelector('.wb-now-kind');
  const nowDetailEl = nowRoot.querySelector('.wb-now-detail');

  let schedule = { dayStart: 0, dayEnd: 0, meetings: [], blocks: [] };

  function toMinutes(hhmm) {
    const [h, m] = hhmm.split(':').map(Number);
    return h * 60 + m;
  }

  function fmt(mins) {
    const h = Math.floor(mins / 60);
    const m = mins % 60;
    const ampm = h >= 12 ? 'pm' : 'am';
    const h12 = ((h + 11) % 12) + 1;
    return h12 + (m ? ':' + String(m).padStart(2, '0') : '') + ampm;
  }

  function getMeetings() {
    return meetingRows
      .filter(row => row.querySelector('.wb-meeting-on').checked)
      .map(row => ({
        title: row.querySelector('.wb-meeting-title').value || 'Meeting',
        start: toMinutes(row.querySelector('.wb-meeting-start').value),
        end: toMinutes(row.querySelector('.wb-meeting-end').value)
      }))
      .filter(m => m.end > m.start)
      .sort((a, b) => a.start - b.start);
  }

  function freeGaps(dayStart, dayEnd, meetings) {
    const gaps = [];
    let cursor = dayStart;
    for (const m of meetings) {
      const start = Math.max(m.start, dayStart);
      const end = Math.min(m.end, dayEnd);
      if (start > cursor) gaps.push([cursor, start]);
      cursor = Math.max(cursor, end);
    }
    if (cursor < dayEnd) gaps.push([cursor, dayEnd]);
    return gaps;
  }

  function scheduleBlocks(gaps, totalMinutesNeeded, blockLen, breakLen) {
    const blocks = [];
    let remaining = totalMinutesNeeded;
    for (const [gapStart, gapEnd] of gaps) {
      let cursor = gapStart;
      while (remaining > 0 && cursor + Math.min(blockLen, remaining) <= gapEnd) {
        const len = Math.min(blockLen, remaining);
        blocks.push({ type: 'block', start: cursor, end: cursor + len });
        cursor += len;
        remaining -= len;
        if (remaining > 0 && cursor + breakLen <= gapEnd) {
          blocks.push({ type: 'break', start: cursor, end: cursor + breakLen });
          cursor += breakLen;
        } else {
          break;
        }
      }
      if (remaining <= 0) break;
    }
    return { blocks, unscheduled: Math.max(0, remaining) };
  }

  function render() {
    const dayStart = toMinutes(dayStartEl.value);
    const dayEnd = toMinutes(dayEndEl.value);
    timelineEl.innerHTML = '';
    ticksEl.innerHTML = '';

    if (dayEnd <= dayStart) {
      statusEl.textContent = 'Work hours need an end time after the start time.';
      return;
    }

    const meetings = getMeetings();
    const span = dayEnd - dayStart;
    const pct = (mins) => ((mins - dayStart) / span) * 100;

    meetings.forEach(m => {
      const seg = document.createElement('div');
      seg.className = 'wb-seg wb-seg-meeting';
      seg.style.left = pct(m.start) + '%';
      seg.style.width = (pct(m.end) - pct(m.start)) + '%';
      seg.title = m.title + ' (' + fmt(m.start) + '–' + fmt(m.end) + ')';
      timelineEl.appendChild(seg);
    });

    const blockLen = Math.max(5, Number(blockLenEl.value) || 25);
    const breakLen = Math.max(0, Number(breakLenEl.value) || 0);
    const taskHours = Math.max(0, Number(taskHoursEl.value) || 0);
    const totalMinutesNeeded = Math.round(taskHours * 60);
    const gaps = freeGaps(dayStart, dayEnd, meetings);
    const { blocks, unscheduled } = scheduleBlocks(gaps, totalMinutesNeeded, blockLen, breakLen);

    schedule = { dayStart, dayEnd, meetings, blocks };

    blocks.forEach(b => {
      const seg = document.createElement('div');
      seg.className = 'wb-seg ' + (b.type === 'block' ? 'wb-seg-block' : 'wb-seg-break');
      seg.style.left = pct(b.start) + '%';
      seg.style.width = (pct(b.end) - pct(b.start)) + '%';
      seg.title = (b.type === 'block' ? 'Workblock' : 'Break') + ' (' + fmt(b.start) + '–' + fmt(b.end) + ')';
      timelineEl.appendChild(seg);
    });

    [dayStart, dayEnd].forEach(t => {
      const tick = document.createElement('div');
      tick.className = 'wb-tick';
      tick.style.left = pct(t) + '%';
      tick.textContent = fmt(t);
      ticksEl.appendChild(tick);
    });

    const marker = document.createElement('div');
    marker.className = 'wb-now-marker';
    timelineEl.appendChild(marker);

    const blockCount = blocks.filter(b => b.type === 'block').length;
    const taskName = taskNameEl.value || 'Task';
    if (unscheduled > 0) {
      statusEl.textContent = taskName + ': scheduled ' + blockCount + ' workblock' + (blockCount === 1 ? '' : 's') +
        ', but ' + Math.round(unscheduled) + ' minutes didn’t fit in the day.';
    } else {
      statusEl.textContent = taskName + ': scheduled ' + blockCount + ' workblock' + (blockCount === 1 ? '' : 's') +
        ' of ' + blockLen + ' minutes, with ' + breakLen + '-minute breaks, around ' + meetings.length + ' meeting' + (meetings.length === 1 ? '' : 's') + '.';
    }

    renderNow();
  }

  function renderNow() {
    const t = Number(nowTimeEl.value);
    nowTimeValueEl.textContent = fmt(t);

    const marker = timelineEl.querySelector('.wb-now-marker');
    const { dayStart, dayEnd, meetings, blocks } = schedule;

    if (dayEnd <= dayStart) {
      if (marker) marker.style.display = 'none';
      nowKindEl.textContent = 'No schedule yet';
      nowDetailEl.textContent = 'Hit Schedule above first.';
      return;
    }

    if (t < dayStart || t > dayEnd) {
      if (marker) marker.style.display = 'none';
      nowKindEl.textContent = 'Outside work hours';
      nowDetailEl.textContent = 'Day runs ' + fmt(dayStart) + '–' + fmt(dayEnd) + '.';
      return;
    }

    if (marker) {
      marker.style.display = '';
      marker.style.left = (((t - dayStart) / (dayEnd - dayStart)) * 100) + '%';
    }

    const meeting = meetings.find(m => t >= m.start && t < m.end);
    if (meeting) {
      nowKindEl.textContent = 'In a meeting: ' + meeting.title;
      nowDetailEl.textContent = 'Until ' + fmt(meeting.end) + '.';
      return;
    }

    const block = blocks.find(b => t >= b.start && t < b.end);
    if (block) {
      const minsLeft = block.end - t;
      if (block.type === 'block') {
        nowKindEl.textContent = 'In a workblock';
        nowDetailEl.textContent = minsLeft + ' min left, then a break.';
      } else {
        nowKindEl.textContent = 'On break';
        nowDetailEl.textContent = minsLeft + ' min left.';
      }
      return;
    }

    nowKindEl.textContent = 'Free time';
    nowDetailEl.textContent = 'No workblock scheduled right now.';
  }

  scheduleBtn.addEventListener('click', render);
  nowTimeEl.addEventListener('input', renderNow);
  render();
})();
</script>

## The matching algorithm

Given the day's bounds and the fixed meetings, the first pass finds the free gaps by walking the sorted meetings and taking whatever's left over:

```python
def free_gaps(day_start, day_end, meetings):
    gaps = []
    cursor = day_start
    for start, end, _title in sorted(meetings):
        start = max(start, day_start)
        end = min(end, day_end)
        if start > cursor:
            gaps.append((cursor, start))
        cursor = max(cursor, end)
    if cursor < day_end:
        gaps.append((cursor, day_end))
    return gaps
```

The second pass walks those gaps and packs in blocks and breaks until the task's total minutes are used up or the day runs out of room:

```python
def schedule_blocks(gaps, total_minutes, block_len, break_len):
    blocks = []
    remaining = total_minutes
    for gap_start, gap_end in gaps:
        cursor = gap_start
        while remaining > 0 and cursor + min(block_len, remaining) <= gap_end:
            length = min(block_len, remaining)
            blocks.append(("block", cursor, cursor + length))
            cursor += length
            remaining -= length
            if remaining > 0 and cursor + break_len <= gap_end:
                blocks.append(("break", cursor, cursor + break_len))
                cursor += break_len
            else:
                break
        if remaining <= 0:
            break
    return blocks, max(0, remaining)
```

The scheduler skips a gap that's too small for even one block entirely, and a task that doesn't fit in the day comes back with `remaining > 0` so the app can flag it instead of silently dropping time.
