---
layout: post
title: "Toothpicks, sticks, logs."
date: 2026-07-27
category: Software Engineering
draft: true
---

<figure class="diagram-figure">
  <svg viewBox="0 0 640 220" role="img" aria-labelledby="tsl-title tsl-desc" style="width:100%;height:auto;font-family:var(--font-sans);">
    <title id="tsl-title">Three panels: a hand picking up toothpicks, a hand carrying a bundle of sticks, a crane lifting a log</title>
    <desc id="tsl-desc">As scale grows, the right tool changes: toothpicks are gathered by hand, sticks are bundled and carried, and a log needs a crane.</desc>

    <!-- Panel 1: toothpicks by hand -->
    <g transform="translate(10,20)">
      <rect x="0" y="0" width="180" height="160" rx="3" fill="none" stroke="var(--color-hairline)" stroke-width="1.5" />
      <line x1="55" y1="95" x2="90" y2="80" stroke="var(--color-ink-mid)" stroke-width="2" />
      <line x1="60" y1="100" x2="95" y2="88" stroke="var(--color-ink-mid)" stroke-width="2" />
      <line x1="50" y1="105" x2="88" y2="95" stroke="var(--color-ink-mid)" stroke-width="2" />
      <line x1="65" y1="92" x2="100" y2="78" stroke="var(--color-ink-mid)" stroke-width="2" />
      <path d="M110,70 C120,60 135,60 140,75 C145,90 130,100 120,95 L100,110" fill="none" stroke="var(--color-accent)" stroke-width="2.5" stroke-linecap="round" />
      <text x="90" y="200" fill="var(--color-muted)" font-size="12" text-anchor="middle">by hand</text>
    </g>

    <!-- Panel 2: sticks bundled and carried -->
    <g transform="translate(230,20)">
      <rect x="0" y="0" width="180" height="160" rx="3" fill="none" stroke="var(--color-hairline)" stroke-width="1.5" />
      <line x1="60" y1="60" x2="75" y2="120" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="72" y1="55" x2="87" y2="118" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="84" y1="58" x2="99" y2="120" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="96" y1="60" x2="108" y2="118" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="58" y1="95" x2="112" y2="90" stroke="var(--color-accent)" stroke-width="2.5" />
      <path d="M120,60 C132,52 145,55 148,70 C151,86 136,96 126,90 L108,105" fill="none" stroke="var(--color-accent)" stroke-width="2.5" stroke-linecap="round" />
      <text x="90" y="200" fill="var(--color-muted)" font-size="12" text-anchor="middle">bundled, carried</text>
    </g>

    <!-- Panel 3: log lifted by crane -->
    <g transform="translate(450,20)">
      <rect x="0" y="0" width="180" height="160" rx="3" fill="none" stroke="var(--color-hairline)" stroke-width="1.5" />
      <line x1="20" y1="150" x2="20" y2="30" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="20" y1="30" x2="130" y2="45" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="20" y1="150" x2="60" y2="150" stroke="var(--color-ink-mid)" stroke-width="3" />
      <line x1="115" y1="47" x2="115" y2="90" stroke="var(--color-accent)" stroke-width="2" stroke-dasharray="3,3" />
      <rect x="90" y="90" width="60" height="26" rx="6" fill="none" stroke="var(--color-accent)" stroke-width="2.5" />
      <text x="90" y="200" fill="var(--color-muted)" font-size="12" text-anchor="middle">by crane</text>
    </g>
  </svg>
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">The right tool changes with scale. You wouldn't crane toothpicks, and you can't hand-carry logs.</figcaption>
</figure>
