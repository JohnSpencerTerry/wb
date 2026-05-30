---
layout: default
title: "Train to Train"
date: 2026-05-30
---

A subway trivia game I built with my wife. She came up with the concept and did the UI; I wrote the data layer and the game loop. We finished a v1 and shelved it, but I think it deserves a write-up here.

<figure class="media-figure">
  <img src="/assets/photos/train2train/185%C2%B0%2040%C2%B0%2035%C2%B0.png" alt="Train to Train home screen on an iPhone, dark MTA-style UI with the Train 2 Train logo and difficulty buttons." />
</figure>

## The concept

You ride these lines every day. How many stops do you know? Pick a route bullet, look at five stops with one missing, and pick the right name from four choices. Easy is same-borough wrong answers. Normal is same-line wrong answers. Difficult is wrong answers from the stops directly next to the answer.

## The wireframe

We sketched a route bullet, a five-stop strip with the middle one replaced by a `?`, and four answer tiles. This means the game is multiple-choice with map context, which keeps the implementation simple.

<figure class="media-figure">
  <img src="/assets/photos/train2train/Start%20Page-1.png" alt="Early grayscale wireframe of the Train to Train home screen." />
  <figcaption>Early home-screen sketch.</figcaption>
</figure>

## The data

NY State Open Data hosts [MTA Subway Stations](https://data.ny.gov/Transportation/MTA-Subway-Stations/39hk-dx4f/about_data). One GET, JSON back:

```bash
curl 'https://data.ny.gov/resource/39hk-dx4f.json?$limit=2000' | jq '.[0]'
```

```json
{
  "stop_name": "Astor Pl",
  "gtfs_stop_id": "128",
  "daytime_routes": "6",
  "borough": "M",
  "gtfs_latitude": "40.730054",
  "gtfs_longitude": "-73.991070",
  "north_direction_label": "Uptown & The Bronx",
  "south_direction_label": "Downtown & Brooklyn"
}
```

The dataset is a list of *stations*. The game needs ordered stops per *route*. So we split `daytime_routes` on whitespace, push each station into one bucket per token, and sort each bucket along whichever axis has the bigger range — latitude for the N–S lines (1, 4, A, etc.), longitude for the L and the 7.

```js
function buildRoutes(stations) {
  const routes = {};
  for (const s of stations) {
    if (!s.daytime_routes) continue;
    for (const r of s.daytime_routes.trim().split(/\s+/)) {
      (routes[r] ||= []).push(s);
    }
  }
  for (const r in routes) {
    const stops = routes[r];
    const lats = stops.map(s => +s.gtfs_latitude);
    const lngs = stops.map(s => +s.gtfs_longitude);
    const latRange = Math.max(...lats) - Math.min(...lats);
    const lngRange = Math.max(...lngs) - Math.min(...lngs);
    stops.sort(latRange >= lngRange
      ? (a, b) => +b.gtfs_latitude - +a.gtfs_latitude
      : (a, b) => +a.gtfs_longitude - +b.gtfs_longitude);
    const seen = new Set();
    routes[r] = stops.filter(s => !seen.has(s.stop_name) && seen.add(s.stop_name));
  }
  return routes;
}
```

## The mockups

Between sketch and ship, the route bullet got the real MTA color, the answer tiles got tightened and rounded, and the prompt became literal: "What is the Missing Stop?" Each change is a readability fix.

<figure class="media-figure">
  <img src="/assets/photos/train2train/Start%20Page.png" alt="Final mockup of the Train to Train home screen on iPhone." />
  <figcaption>Final home screen.</figcaption>
</figure>

<figure class="media-figure">
  <img src="/assets/photos/train2train/EasyQ1.png" alt="Final mockup of a Train to Train question screen for the L line." />
  <figcaption>Final question screen.</figcaption>
</figure>

## Playable

Live API, no backend. Fetches stations once and caches the cleaned route map in `localStorage` so reloads are free.

<div id="t2t" class="t2t-panel" data-stage="loading">
  <div class="t2t-status">Loading subway stations…</div>
  <div class="t2t-game" hidden>
    <div class="t2t-bullet" aria-label="Route"></div>
    <div class="t2t-prompt">What is the Missing Stop?</div>
    <div class="t2t-strip" role="list"></div>
    <div class="t2t-choices"></div>
    <div class="t2t-feedback" aria-live="polite"></div>
    <button class="t2t-next" type="button" hidden>Next →</button>
  </div>
  <div class="t2t-error" hidden></div>
</div>

<style>
  .t2t-panel {
    background: #0e0e10;
    color: #fff;
    border-radius: 10px;
    padding: 1.5rem 1.25rem;
    margin: 1.5rem 0;
    font-family: system-ui, sans-serif;
  }
  .t2t-status, .t2t-error { text-align: center; color: #bbb; padding: 1rem 0; }
  .t2t-error { color: #f8b4b4; }
  .t2t-bullet {
    width: 64px; height: 64px; border-radius: 50%;
    background: #A7A9AC; color: #fff;
    display: flex; align-items: center; justify-content: center;
    font-size: 2rem; font-weight: 700; margin: 0.5rem auto 1rem;
  }
  .t2t-bullet.dark-text { color: #000; }
  .t2t-prompt { text-align: center; font-size: 1.15rem; margin-bottom: 1.25rem; }
  .t2t-strip {
    display: grid; grid-template-columns: repeat(5, 1fr);
    align-items: end; gap: 4px;
    padding: 0.5rem 0 1.25rem; margin: 0 auto 1rem;
    max-width: 360px;
    position: relative;
  }
  .t2t-strip::after {
    content: ""; position: absolute; left: 8%; right: 8%; bottom: 1.15rem;
    height: 4px; background: var(--t2t-line, #A7A9AC); z-index: 0;
  }
  .t2t-strip > div {
    display: flex; flex-direction: column; align-items: center;
    font-size: 0.7rem; color: #ddd;
    position: relative; z-index: 1;
  }
  .t2t-strip .t2t-label {
    min-height: 2.4em;
    line-height: 1.15;
    text-align: center;
    margin-bottom: 0.35rem;
    word-break: break-word;
  }
  .t2t-strip .t2t-dot {
    width: 14px; height: 14px; border-radius: 50%;
    background: #fff; border: 3px solid var(--t2t-line, #A7A9AC);
  }
  .t2t-strip .t2t-missing .t2t-dot { background: #0e0e10; border-style: dashed; }
  .t2t-strip .t2t-missing .t2t-label { font-weight: 700; color: #fff; font-size: 1rem; }
  .t2t-choices {
    display: grid; grid-template-columns: 1fr 1fr;
    gap: 10px; margin-bottom: 1rem;
  }
  .t2t-choices button {
    background: #000; color: #fff;
    border: 2px solid #fff; border-radius: 999px;
    padding: 0.75rem 0.5rem; font-size: 0.95rem;
    cursor: pointer; font-family: inherit;
    min-height: 3rem;
  }
  .t2t-choices button:disabled { cursor: default; }
  .t2t-choices button.correct { border-color: #2ecc71; }
  .t2t-choices button.wrong { border-color: #e74c3c; }
  .t2t-feedback { text-align: center; min-height: 1.4em; margin-bottom: 0.5rem; }
  .t2t-feedback.correct { color: #2ecc71; }
  .t2t-feedback.wrong { color: #f08; }
  .t2t-next {
    display: block; margin: 0 auto;
    background: #fff; color: #000; border: 0;
    border-radius: 999px; padding: 0.6rem 1.5rem;
    font-weight: 600; cursor: pointer; font-family: inherit;
  }
</style>

<script>
(function() {
  const API_URL = 'https://data.ny.gov/resource/39hk-dx4f.json?$limit=2000';
  const CACHE_KEY = 't2t-stations-v1';
  const ROUTE_COLORS = {
    '1':'#EE352E','2':'#EE352E','3':'#EE352E',
    '4':'#00933C','5':'#00933C','6':'#00933C',
    '7':'#B933AD',
    'A':'#0039A6','C':'#0039A6','E':'#0039A6',
    'B':'#FF6319','D':'#FF6319','F':'#FF6319','M':'#FF6319',
    'N':'#FCCC0A','Q':'#FCCC0A','R':'#FCCC0A','W':'#FCCC0A',
    'G':'#6CBE45',
    'J':'#996633','Z':'#996633',
    'L':'#A7A9AC',
    'S':'#808183'
  };
  const LIGHT_BULLET_ROUTES = new Set(['N','Q','R','W']);

  const root = document.getElementById('t2t');
  const statusEl = root.querySelector('.t2t-status');
  const gameEl = root.querySelector('.t2t-game');
  const errEl = root.querySelector('.t2t-error');
  const bulletEl = root.querySelector('.t2t-bullet');
  const stripEl = root.querySelector('.t2t-strip');
  const choicesEl = root.querySelector('.t2t-choices');
  const feedbackEl = root.querySelector('.t2t-feedback');
  const nextEl = root.querySelector('.t2t-next');

  let stations = null;
  let routes = null;

  async function loadStations() {
    try {
      const cached = localStorage.getItem(CACHE_KEY);
      if (cached) return JSON.parse(cached);
    } catch (e) { /* ignore */ }
    const res = await fetch(API_URL);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();
    try { localStorage.setItem(CACHE_KEY, JSON.stringify(data)); } catch (e) { /* quota */ }
    return data;
  }

  function buildRoutes(stations) {
    const routes = {};
    for (const s of stations) {
      if (!s.daytime_routes) continue;
      for (const r of s.daytime_routes.trim().split(/\s+/)) {
        (routes[r] = routes[r] || []).push(s);
      }
    }
    for (const r in routes) {
      const stops = routes[r];
      const lats = stops.map(s => +s.gtfs_latitude);
      const lngs = stops.map(s => +s.gtfs_longitude);
      const latRange = Math.max.apply(null, lats) - Math.min.apply(null, lats);
      const lngRange = Math.max.apply(null, lngs) - Math.min.apply(null, lngs);
      if (latRange >= lngRange) {
        stops.sort((a, b) => +b.gtfs_latitude - +a.gtfs_latitude);
      } else {
        stops.sort((a, b) => +a.gtfs_longitude - +b.gtfs_longitude);
      }
      const seen = new Set();
      routes[r] = stops.filter(s => {
        if (seen.has(s.stop_name)) return false;
        seen.add(s.stop_name);
        return true;
      });
    }
    return routes;
  }

  function shuffle(arr) {
    const a = arr.slice();
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [a[i], a[j]] = [a[j], a[i]];
    }
    return a;
  }

  function pickQuestion() {
    const valid = Object.keys(routes).filter(r => routes[r].length >= 5 && ROUTE_COLORS[r]);
    const route = valid[Math.floor(Math.random() * valid.length)];
    const stops = routes[route];
    const idx = 2 + Math.floor(Math.random() * (stops.length - 4));
    const correct = stops[idx];
    const window = stops.slice(idx - 2, idx + 3);
    const winNames = new Set(window.map(s => s.stop_name));
    const pool = stations.filter(s =>
      s.borough === correct.borough &&
      !winNames.has(s.stop_name) &&
      s.stop_name !== correct.stop_name
    );
    const distractors = [];
    const used = new Set();
    const shuffledPool = shuffle(pool);
    for (const d of shuffledPool) {
      if (distractors.length >= 3) break;
      if (used.has(d.stop_name)) continue;
      used.add(d.stop_name);
      distractors.push(d);
    }
    if (distractors.length < 3) return pickQuestion();
    return {
      route,
      stops: window,
      missingIndex: 2,
      correct,
      choices: shuffle([correct, ...distractors])
    };
  }

  function renderQuestion(q) {
    const color = ROUTE_COLORS[q.route] || '#A7A9AC';
    bulletEl.textContent = q.route;
    bulletEl.style.background = color;
    bulletEl.classList.toggle('dark-text', LIGHT_BULLET_ROUTES.has(q.route));

    stripEl.style.setProperty('--t2t-line', color);
    stripEl.innerHTML = '';
    q.stops.forEach((s, i) => {
      const cell = document.createElement('div');
      if (i === q.missingIndex) cell.classList.add('t2t-missing');
      const label = document.createElement('div');
      label.className = 't2t-label';
      label.textContent = i === q.missingIndex ? '?' : s.stop_name;
      const dot = document.createElement('div');
      dot.className = 't2t-dot';
      cell.appendChild(label);
      cell.appendChild(dot);
      stripEl.appendChild(cell);
    });

    choicesEl.innerHTML = '';
    q.choices.forEach(c => {
      const btn = document.createElement('button');
      btn.type = 'button';
      btn.textContent = c.stop_name;
      btn.addEventListener('click', () => onAnswer(btn, c, q));
      choicesEl.appendChild(btn);
    });

    feedbackEl.textContent = '';
    feedbackEl.className = 't2t-feedback';
    nextEl.hidden = true;
  }

  function onAnswer(btn, choice, q) {
    Array.from(choicesEl.querySelectorAll('button')).forEach(b => {
      b.disabled = true;
      if (b.textContent === q.correct.stop_name) b.classList.add('correct');
    });
    const isCorrect = choice.stop_name === q.correct.stop_name;
    if (!isCorrect) btn.classList.add('wrong');
    feedbackEl.textContent = isCorrect
      ? 'Correct.'
      : 'Nope — it was ' + q.correct.stop_name + '.';
    feedbackEl.classList.add(isCorrect ? 'correct' : 'wrong');
    nextEl.hidden = false;
  }

  nextEl.addEventListener('click', () => renderQuestion(pickQuestion()));

  loadStations().then(data => {
    stations = data;
    routes = buildRoutes(stations);
    statusEl.hidden = true;
    gameEl.hidden = false;
    renderQuestion(pickQuestion());
  }).catch(err => {
    statusEl.hidden = true;
    errEl.hidden = false;
    errEl.textContent = 'Could not load station data: ' + err.message;
  });
})();
</script>

