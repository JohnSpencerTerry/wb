---
layout: post
title: "DoggieDo: A Dog Owner Discovery App"
date: 2026-07-05
---

My wife and I built DoggieDo, an app for dog owners to find dog-friendly places nearby and keep a profile for their dog. We shelved it before it had real users because we didn't think we could put together a viable business model.

## The concept

Two pieces on the consumer side. The first is a map of dog-friendly parks, vets, cafes, and groomers. You can filter it by category. Each place shows tips and activity counts from other owners. The second is a pet profile builder. It's a short wizard. You enter your dog's name, breed, and age. Then you add personality and health notes. Then you add a photo and bio.

Alongside that, we built out DoggieDo for Business. The idea was to make DoggieDo a two-sided platform: dog owners get a free, useful app, and pet care businesses (vets, groomers, cafes, boarding) pay to advertise directly to the owners already using it in their neighborhood. The consumer app was the audience; the business side was the revenue.

## The tech

We used FastAPI on the backend because it paired well with SQLModel, which let us define a table once and get both the Postgres schema and the API's request/response types from it. Postgres also gave us PostGIS for location queries, which the map needed. On the frontend we used React with TanStack Router and Tailwind, and ran the whole stack locally with Docker Compose.

## Where it fell short

DoggieDo is a "telephone" app: the map and tips only get good once enough dog owners in an area are using it. Without that density, there's not much for a new owner to see, and not much for a business to advertise against.

One option was to geo-restrict launch to NYC to build density faster. But a smaller footprint meant DoggieDo for Business was competing for the same pet care advertising dollars as every other local marketing channel in one of the most contested markets around, with none of the reach that would make it worth a business's while.

## Try it

These are quick recreations of the two features in plain JS, not the original app.

<div id="dd-places" class="dd-panel">
  <div class="dd-filters"></div>
  <div class="dd-list"></div>
</div>

<div id="dd-pet" class="dd-panel">
  <div class="dd-steps"></div>
  <div class="dd-step-body"></div>
  <div class="dd-nav">
    <button type="button" class="dd-back">Back</button>
    <button type="button" class="dd-next">Next</button>
  </div>
  <div class="dd-card" style="display:none;"></div>
</div>

<style>
  .dd-panel {
    background: #0e0e10;
    color: #fff;
    border-radius: 10px;
    padding: 1.25rem 1.25rem 1.5rem;
    margin: 1.5rem 0;
    font-family: system-ui, sans-serif;
  }
  .dd-filters { display: flex; flex-wrap: wrap; gap: 0.5rem; margin-bottom: 1rem; }
  .dd-filter-btn {
    background: #17171a; color: #ccc; border: 1px solid #333;
    border-radius: 999px; padding: 0.35rem 0.9rem; font-size: 0.8rem;
    cursor: pointer; font-family: inherit;
  }
  .dd-filter-btn.active { background: #fff; color: #000; border-color: #fff; }
  .dd-list { display: flex; flex-direction: column; gap: 0.6rem; }
  .dd-place {
    display: flex; justify-content: space-between; align-items: center;
    background: #17171a; border: 1px solid #333; border-radius: 8px;
    padding: 0.65rem 0.9rem;
  }
  .dd-place-name { font-weight: 600; font-size: 0.9rem; }
  .dd-place-meta { font-size: 0.78rem; color: #9fd6ff; margin-top: 0.15rem; }
  .dd-place-tag {
    font-size: 0.7rem; color: #aaa; border: 1px solid #333;
    border-radius: 999px; padding: 0.15rem 0.6rem; text-transform: capitalize;
  }
  .dd-steps { display: flex; gap: 0.5rem; margin-bottom: 1rem; font-size: 0.78rem; color: #666; }
  .dd-step { display: flex; align-items: center; gap: 0.4rem; }
  .dd-step.active { color: #fff; font-weight: 600; }
  .dd-dot {
    width: 8px; height: 8px; border-radius: 50%; background: #333; display: inline-block;
  }
  .dd-step.active .dd-dot { background: #2ecc71; }
  .dd-step-body { display: flex; flex-direction: column; gap: 0.65rem; font-size: 0.85rem; color: #ccc; }
  .dd-step-body label { display: flex; flex-direction: column; gap: 0.3rem; }
  .dd-panel input[type="text"],
  .dd-panel input[type="number"],
  .dd-panel textarea {
    background: #17171a; color: #fff;
    border: 1px solid #333; border-radius: 6px;
    padding: 0.4rem 0.6rem; font-size: 0.85rem;
    font-family: inherit;
  }
  .dd-panel textarea { resize: vertical; min-height: 3.5rem; }
  .dd-check-row { display: flex; flex-wrap: wrap; gap: 0.5rem; }
  .dd-check-row label {
    flex-direction: row; align-items: center; gap: 0.4rem;
    background: #17171a; border: 1px solid #333; border-radius: 999px;
    padding: 0.3rem 0.75rem; font-size: 0.8rem; cursor: pointer;
  }
  .dd-emoji-row { display: flex; flex-wrap: wrap; gap: 0.5rem; }
  .dd-emoji-btn {
    background: #17171a; border: 1px solid #333; border-radius: 8px;
    font-size: 1.4rem; padding: 0.3rem 0.6rem; cursor: pointer;
  }
  .dd-emoji-btn.active { border-color: #2ecc71; background: #1a2e22; }
  .dd-nav { display: flex; justify-content: space-between; margin-top: 1.1rem; }
  .dd-nav button {
    background: #fff; color: #000; border: 0;
    border-radius: 999px; padding: 0.5rem 1.25rem;
    font-weight: 600; font-size: 0.85rem;
    cursor: pointer; font-family: inherit;
  }
  .dd-back { background: transparent !important; color: #ccc !important; border: 1px solid #333 !important; }
  .dd-nav button:disabled { opacity: 0.4; cursor: default; }
  .dd-card {
    margin-top: 1rem; padding: 1rem; background: #17171a;
    border: 1px solid #333; border-radius: 8px;
  }
  .dd-card-name { font-size: 1.1rem; font-weight: 700; }
  .dd-card-emoji { font-size: 2rem; margin-right: 0.5rem; }
  .dd-card-row { font-size: 0.82rem; color: #9fd6ff; margin-top: 0.3rem; }
  .dd-card-bio { font-size: 0.82rem; color: #ccc; margin-top: 0.5rem; }
</style>

<script>
(function() {
  const places = [
    { name: "Tompkins Square Dog Run", category: "park", tips: 42, distance: "0.3 mi" },
    { name: "East River Off-Leash Area", category: "park", tips: 18, distance: "0.6 mi" },
    { name: "Lucky Paws Vet Clinic", category: "vet", tips: 27, distance: "0.4 mi" },
    { name: "Downtown Animal Hospital", category: "vet", tips: 15, distance: "1.1 mi" },
    { name: "Barkery Cafe", category: "cafe", tips: 33, distance: "0.2 mi" },
    { name: "Muddy Paws Grooming", category: "groomer", tips: 21, distance: "0.5 mi" }
  ];
  const categories = ["all", "park", "vet", "cafe", "groomer"];

  const placesRoot = document.getElementById('dd-places');
  const filtersEl = placesRoot.querySelector('.dd-filters');
  const listEl = placesRoot.querySelector('.dd-list');
  let activeCategory = "all";

  categories.forEach(cat => {
    const btn = document.createElement('button');
    btn.type = 'button';
    btn.className = 'dd-filter-btn' + (cat === activeCategory ? ' active' : '');
    btn.textContent = cat === 'all' ? 'All' : cat.charAt(0).toUpperCase() + cat.slice(1);
    btn.addEventListener('click', () => {
      activeCategory = cat;
      filtersEl.querySelectorAll('.dd-filter-btn').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      renderPlaces();
    });
    filtersEl.appendChild(btn);
  });

  function renderPlaces() {
    listEl.innerHTML = '';
    const filtered = places.filter(p => activeCategory === 'all' || p.category === activeCategory);
    filtered.forEach(p => {
      const row = document.createElement('div');
      row.className = 'dd-place';
      row.innerHTML =
        '<div>' +
          '<div class="dd-place-name">' + p.name + '</div>' +
          '<div class="dd-place-meta">' + p.tips + ' tips &middot; ' + p.distance + ' away</div>' +
        '</div>' +
        '<span class="dd-place-tag">' + p.category + '</span>';
      listEl.appendChild(row);
    });
    if (!filtered.length) {
      listEl.innerHTML = '<div class="dd-place-meta">No places in this category yet.</div>';
    }
  }
  renderPlaces();

  const petRoot = document.getElementById('dd-pet');
  const stepsEl = petRoot.querySelector('.dd-steps');
  const bodyEl = petRoot.querySelector('.dd-step-body');
  const backBtn = petRoot.querySelector('.dd-back');
  const nextBtn = petRoot.querySelector('.dd-next');
  const cardEl = petRoot.querySelector('.dd-card');

  const stepNames = ['Basics', 'Personality & health', 'Photo & bio'];
  const traits = ['Friendly', 'Energetic', 'Shy', 'Good with kids', 'Reactive'];
  const emojis = ['🐕', '🐩', '🐕‍🦺', '🐶'];

  let step = 0;
  const pet = { name: '', breed: '', age: '', traits: [], emoji: emojis[0], bio: '' };

  function renderSteps() {
    stepsEl.innerHTML = '';
    stepNames.forEach((name, i) => {
      const el = document.createElement('div');
      el.className = 'dd-step' + (i === step ? ' active' : '');
      el.innerHTML = '<span class="dd-dot"></span>' + name;
      stepsEl.appendChild(el);
    });
  }

  function renderStepBody() {
    if (step === 0) {
      bodyEl.innerHTML =
        '<label>Name <input type="text" class="dd-name" value="' + pet.name + '"></label>' +
        '<label>Breed <input type="text" class="dd-breed" value="' + pet.breed + '"></label>' +
        '<label>Age <input type="number" class="dd-age" min="0" max="25" value="' + pet.age + '"></label>';
    } else if (step === 1) {
      bodyEl.innerHTML =
        '<div>Personality &amp; health notes</div>' +
        '<div class="dd-check-row">' +
        traits.map(t =>
          '<label><input type="checkbox" class="dd-trait" value="' + t + '"' +
          (pet.traits.includes(t) ? ' checked' : '') + '>' + t + '</label>'
        ).join('') +
        '</div>';
    } else {
      bodyEl.innerHTML =
        '<div>Pick a photo</div>' +
        '<div class="dd-emoji-row">' +
        emojis.map(e =>
          '<button type="button" class="dd-emoji-btn' + (pet.emoji === e ? ' active' : '') + '" data-emoji="' + e + '">' + e + '</button>'
        ).join('') +
        '</div>' +
        '<label>Bio <textarea class="dd-bio">' + pet.bio + '</textarea></label>';
    }
    wireStepBody();
    updateCard();
  }

  function wireStepBody() {
    if (step === 0) {
      bodyEl.querySelector('.dd-name').addEventListener('input', e => { pet.name = e.target.value; updateCard(); });
      bodyEl.querySelector('.dd-breed').addEventListener('input', e => { pet.breed = e.target.value; updateCard(); });
      bodyEl.querySelector('.dd-age').addEventListener('input', e => { pet.age = e.target.value; updateCard(); });
    } else if (step === 1) {
      bodyEl.querySelectorAll('.dd-trait').forEach(cb => {
        cb.addEventListener('change', () => {
          pet.traits = Array.from(bodyEl.querySelectorAll('.dd-trait:checked')).map(el => el.value);
          updateCard();
        });
      });
    } else {
      bodyEl.querySelectorAll('.dd-emoji-btn').forEach(btn => {
        btn.addEventListener('click', () => {
          pet.emoji = btn.dataset.emoji;
          bodyEl.querySelectorAll('.dd-emoji-btn').forEach(b => b.classList.remove('active'));
          btn.classList.add('active');
          updateCard();
        });
      });
      bodyEl.querySelector('.dd-bio').addEventListener('input', e => { pet.bio = e.target.value; updateCard(); });
    }
  }

  function updateCard() {
    cardEl.style.display = '';
    cardEl.innerHTML =
      '<span class="dd-card-emoji">' + pet.emoji + '</span>' +
      '<span class="dd-card-name">' + (pet.name || 'Unnamed dog') + '</span>' +
      '<div class="dd-card-row">' + (pet.breed || 'Unknown breed') + (pet.age ? ', ' + pet.age + ' yrs' : '') + '</div>' +
      (pet.traits.length ? '<div class="dd-card-row">' + pet.traits.join(', ') + '</div>' : '') +
      (pet.bio ? '<div class="dd-card-bio">' + pet.bio + '</div>' : '');
  }

  function renderNav() {
    backBtn.disabled = step === 0;
    nextBtn.textContent = step === stepNames.length - 1 ? 'Done' : 'Next';
  }

  backBtn.addEventListener('click', () => {
    if (step > 0) { step -= 1; renderSteps(); renderStepBody(); renderNav(); }
  });
  nextBtn.addEventListener('click', () => {
    if (step < stepNames.length - 1) {
      step += 1;
      renderSteps(); renderStepBody(); renderNav();
    } else {
      nextBtn.textContent = 'Saved';
      nextBtn.disabled = true;
    }
  });

  renderSteps();
  renderStepBody();
  renderNav();
})();
</script>
