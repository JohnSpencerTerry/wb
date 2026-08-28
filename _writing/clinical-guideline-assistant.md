---
layout: post
title: "Learning LangGraph by building a clinical guideline assistant."
date: 2026-08-28
tags: [Tech, AI, Side Project]
draft: true
---

This project was an opportunity to experiment with LangChain and LangGraph while building a clinical-guidelines grounded Q&A tool. That required guardrails, retrieval, structured extraction, a classification step, and an answer that changes shape depending on that classification, which is a lot more graph than a single retrieval chain. Code is on GitHub: [clinical-guideline-assistant](https://github.com/JohnSpencerTerry/clinical-guideline-assistant).

This project focused on Type 2 diabetes and draws on two sources: the ADA's Standards of Care and the UK's NICE guideline (NG28), both public and well-structured. The two can agree, disagree, or one can be silent on a given question, and that range is what the comparison and classification part of the graph is built to handle.

There's a [live demo](https://john-spencer-terry-clinical-guideline-assistant.streamlit.app/), and the repo's [example-prompts.md](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/example-prompts.md) has a longer list organized by category. Four of those categories map directly onto the graph's shape, described below with real output from the deployed demo: grounded factual recall, cross-source comparison, the scope guardrail, and urgent/emergency detection.

## The flow, at a glance

<figure class="diagram-figure">
  <svg viewBox="0 0 860 200" role="img" aria-labelledby="cga-flow-title cga-flow-desc" style="width:100%;height:auto;font-family:var(--font-sans);">
    <title id="cga-flow-title">The LangGraph flow, from question to answer</title>
    <desc id="cga-flow-desc">A question passes through guardrail checks, which can short-circuit to a redirect answer. Otherwise it proceeds to retrieval and extraction per source, then comparison and branch-specific synthesis, ending in a cited answer.</desc>

    <line x1="90" y1="42" x2="130" y2="42" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#cga-flow-arrow)" />
    <line x1="280" y1="42" x2="320" y2="42" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#cga-flow-arrow)" />
    <line x1="500" y1="42" x2="540" y2="42" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#cga-flow-arrow)" />
    <line x1="730" y1="42" x2="770" y2="42" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#cga-flow-arrow)" />
    <line x1="205" y1="64" x2="205" y2="128" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#cga-flow-arrow)" />

    <rect x="0" y="20" width="90" height="44" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="45" y="46" fill="var(--color-ink)" font-size="12.5" text-anchor="middle">Question</text>

    <rect x="130" y="20" width="150" height="44" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="205" y="40" fill="var(--color-ink)" font-size="12" text-anchor="middle">Guardrails</text>
    <text x="205" y="55" fill="var(--color-muted)" font-size="10.5" text-anchor="middle">urgent + scope checks</text>

    <rect x="320" y="20" width="180" height="44" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="410" y="40" fill="var(--color-ink)" font-size="12" text-anchor="middle">Retrieve + extract</text>
    <text x="410" y="55" fill="var(--color-muted)" font-size="10.5" text-anchor="middle">per source: ADA, NICE</text>

    <rect x="540" y="20" width="190" height="44" rx="3" fill="var(--color-bg)" stroke="var(--color-accent)" stroke-width="2" />
    <text x="635" y="40" fill="var(--color-ink)" font-size="12" text-anchor="middle">Compare + synthesize</text>
    <text x="635" y="55" fill="var(--color-muted)" font-size="9.5" text-anchor="middle">same / scope diff / conflict / silent</text>

    <rect x="770" y="20" width="90" height="44" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="815" y="46" fill="var(--color-ink)" font-size="12.5" text-anchor="middle">Answer</text>

    <rect x="130" y="128" width="150" height="40" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" stroke-dasharray="3,2" />
    <text x="205" y="152" fill="var(--color-muted)" font-size="11" text-anchor="middle">Redirect (short-circuits)</text>

    <defs>
      <marker id="cga-flow-arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
        <path d="M0,0 L6,3 L0,6 Z" fill="var(--color-hairline)" />
      </marker>
    </defs>
  </svg>
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">Two guardrail nodes can short-circuit the graph before retrieval ever runs. Otherwise the question flows through per-source retrieval and extraction, comparison, and branch-specific synthesis.</figcaption>
</figure>

Grounded factual recall walks the main spine. Cross-source comparison is what compare + synthesize is for. The scope guardrail and urgent/emergency detection each trigger a different short-circuit.

## Setting up the system

Model access goes through [OpenRouter](https://openrouter.ai) rather than directly against a provider API, mostly so the whole thing can run on a free-tier model with the key's own credit limit set to $0 — a request can fail, but it can't bill anything. Embeddings run locally via `sentence-transformers`, so indexing doesn't need an API key either, just CPU time.

The source documents are downloaded once as PDFs and read from disk rather than scraped live. Chunking is content-aware rather than one-size-fits-all: NICE's text is already atomic per numbered recommendation, so short documents pass through unchanged, while ADA's long narrative sections get recursively split, with each chunk still carrying its parent section's metadata so a citation can point back to where it came from (code: [`chunking.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/ingestion/chunking.py)).

The chunks land in two separate vector indices, one per source, instead of a single pooled index. A pooled index would retrieve passages from both sources and hand them to the model together, inviting it to blend them into one confident answer even when the sources disagree. Separate indices let the graph retrieve from ADA and NICE independently and compare what each one actually says.

A Streamlit chat UI sits on top: `uv run streamlit run app/streamlit_app.py`.

## Grounded factual recall

A prompt such as "What is the first-line pharmacologic treatment for type 2 diabetes?" produces a metformin recommendation cited to ADA section 13, plus a note that the retrieved NICE passage doesn't explicitly address a first-line drug for this framing of the question. Both sides stay grounded, instead of the model filling in the NICE side from what it already knows about diabetes drugs.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/first-line-treatment.png" alt="Demo screenshot: question 'What is the first-line pharmacologic treatment for type 2 diabetes?' answered with a metformin recommendation, a note that the NICE excerpt doesn't explicitly state a first-line drug, and sources [13] [1.45.2]." style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">The deployed demo's actual answer, with citations rendered under it.</figcaption>
</figure>

Behind the scenes, the graph retrieves from the ADA index and the NICE index independently, then runs each source's passages through an extraction step that turns them into a structured claim rather than free text: a recommendation, the population it applies to, an evidence grade where the source states one, and a citation identifier (code: [`extraction.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/extraction.py)).

With both claims in hand, a comparison node checks whether they agree, disagree, or one source is silent, and synthesis writes the answer to match. Every claim traces back to a specific citation, rendered under the answer in the chat UI: `[13] [1.45.2]` above, ADA's section number and NICE's recommendation number. Being able to check a claim against the actual source text is the whole grounding argument, which makes citations the one piece of the frontend worth getting right.

## Cross-source comparison

A prompt such as "How do ADA and NICE differ on managing chronic kidney disease risk in type 2 diabetes?" produces two intact, separately cited recommendations instead of one blended answer. ADA's is broad: incorporate agents that lower cardiovascular and kidney risk regardless of their effect on blood glucose. NICE's is narrower and more specific: metformin plus an SGLT-2 inhibitor, and only once eGFR is above 30.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/ckd-comparison.png" alt="Demo screenshot: question about ADA vs NICE on chronic kidney disease risk, answered with ADA's broad glucose-agnostic guidance in one paragraph and NICE's specific metformin plus SGLT-2 inhibitor recommendation in another, plus a closing paragraph naming the key difference." style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">The deployed demo's actual answer, laying out both positions rather than merging them.</figcaption>
</figure>

The comparison node's job is narrow on purpose: classify the two claims as `same`, `scope_difference`, `conflict`, or `silent`, and nothing else. It's tempting to let the classifier also explain a disagreement while it's looking at both claims, but folding resolution into classification is how a model ends up quietly deciding which guideline is "right" (code: [`compare.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/compare.py)). Synthesis handles that harder judgment call instead, in a separate, explicit branch per classification. The `conflict` branch carries the constraint that matters most in the whole system:

```python
"conflict": (
    "The sources genuinely disagree. Present both positions without picking a winner. "
    "Explain WHY they differ ONLY if a claim's stated_rationale field is non-null — quote or "
    "paraphrase that stated rationale. If both stated_rationale fields are null, explicitly say "
    "the sources don't state a reason for the difference. NEVER invent a rationale."
),
```

The `stated_rationale` field is set during extraction, only when the source text itself gives a reason. If neither guideline explains why they diverge, the answer says that plainly instead of the model reaching for a plausible-sounding explanation. A fabricated "why" reads as authoritative in this domain, so this is a hard rule in the prompt, not a suggestion (code: [`synthesize.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/synthesize.py)).

`scope_difference` is a separate bucket for cases that look like disagreement but aren't: different eGFR thresholds framed differently, say, rather than an actual conflict about what to do. Collapsing that into "conflict" would manufacture a disagreement that isn't real. Collapsing it into "same" would flatten a distinction that matters. Either way, the chat UI marks it with a small "guidelines differ here" badge, making the different path visually obvious.

## Scope guardrail

A prompt such as "Should I stop taking my metformin? My doctor prescribed it but I feel sick." produces a reframe back to the general question and a pointer toward an actual care team, instead of an answer. It's not urgent and not off-topic, but it's asking for individualized medical advice, not what a guideline says in general.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/scope-redirect.png" alt="Demo screenshot: question 'Should I stop taking my metformin? My doctor prescribed it but I feel sick.' answered with a redirect: 'I can explain what published Type 2 Diabetes guidelines say in general, but I can't give individualized medical advice for a specific person's situation.'" style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">The deployed demo's actual redirect, rendered as a distinct warning bubble rather than a normal answer.</figcaption>
</figure>

The harder part of this guardrail is holding that boundary without over-triggering. A prompt such as "If I have CKD, does that change treatment recommendations?" is phrased with "I," but it should still get answered as a general question, because it's asking about a population, not requesting advice for a real, specific situation. The scope classifier's prompt spells out that distinction directly, with worked examples on both sides of the line (code: [`scope_classifier.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/guardrails/scope_classifier.py)).

There's a second layer behind the classifier, too. The synthesis prompt restates the system's role on every single call, general or not: "You explain what published clinical guidelines say. You do not provide individualized medical advice." If a borderline message slips past the classifier, generation is still self-limited by that same constraint. Neither layer is trusted alone to hold the line.

## Urgent/emergency detection

A prompt such as "I'm experiencing pain in my chest that I think is due to low blood sugar. Can I take aspirin alongside my normal diabetes medication?" produces an immediate redirect to emergency care, no retrieval involved, even though the second half of the message is phrased like an ordinary medication question. The guardrail sits at the very front of the graph, before any of the RAG machinery runs, and the answer comes back rendered distinctly in the UI rather than as a normal chat bubble.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/emergency-redirect.png" alt="Demo screenshot: message 'I'm experiencing pain in my chest that I think is due to low blood sugar. Can I take aspirin alongside my normal diabetes medication?' answered with an emergency redirect telling the user to call 911 or go to the nearest emergency room." style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">The deployed demo's actual redirect, triggered before the medication question ever reaches retrieval.</figcaption>
</figure>

The check runs in two stages: a fast keyword/pattern match first, then an LLM classifier that only runs if the keywords don't hit, specifically to catch phrasing a fixed pattern list wouldn't (code: [`urgent_check.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/guardrails/urgent_check.py)). This particular message is phrased as "pain in my chest" rather than the literal "chest pain" the keyword pattern looks for, so it's the LLM classifier stage that catches it — and it catches the whole message, guideline question and all, before any of it reaches retrieval. Both stages are deliberately biased toward over-triggering. A false positive costs a mildly annoying redirect. A false negative means an emergency gets a calm, cited RAG answer instead of "call 911." The LLM classifier's own prompt says outright to bias toward "yes" if it's unsure, instead of aiming for the balanced precision and recall a classifier would normally target.

## Checking that the guardrails hold

A demo only shows that a few hand-picked examples work. The eval suite checks "the guardrails work" and "it surfaces disagreement correctly" against a real question set instead of leaving them as claims.

The question set is hand-written, read directly out of the ADA and NICE source text rather than generated, and split across six categories: grounded factual recall, cross-source comparison, structured extraction, the scope guardrail, urgent-symptom detection, and adversarial edge cases like a hypothetical reframed specifically to try to slip past the scope classifier. The guardrail categories grade pass/fail deterministically, since "did it trigger or not" has a clean right answer. Comparison and recall are run-and-reported for now, since grading those well needs reference answers this project hasn't fully built out yet.

## What this was for

The diabetes framing gave the project real stakes to build against. Each category above maps to a distinct LangGraph pattern: short-circuiting guardrail nodes ahead of the main flow, retrieval and extraction run independently per source, a classification node whose output determines routing, and synthesis that branches on that classification instead of writing one generic prompt. Building those five pieces was the actual goal, and the reason for the project.

Medical Q&A is a domain where this approach can actually work, and only because of two things this project leaned on hard: the answer space is limited to two named sources instead of the model's general knowledge, and a layer of guardrails decides what's answerable at all before anything reaches an LLM. Neither piece is optional. Retrieval without the guardrails would answer a patient-specific or urgent question just as confidently as a general one. Guardrails without narrow retrieval would still risk blending or inventing guidance the sources never actually gave.

It's still exactly what the README says it is: a learning project, not a clinical tool. The guardrails here are a reasonable first pass, not a substitute for the regulatory, legal, and clinical review a real deployment would need.
