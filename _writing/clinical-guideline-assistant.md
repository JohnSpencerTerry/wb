---
layout: post
title: "Learning LangGraph by building a clinical guideline assistant."
date: 2026-08-28
tags: [Tech, AI, Side Project]
draft: true
mermaid: true
---

This project was an opportunity to experiment with LangChain and LangGraph while building a clinical-guidelines grounded Q&A tool. That required guardrails, retrieval, structured extraction, a classification step, and an answer that changes shape depending on that classification, which is a lot more graph than a single retrieval chain. Code is on GitHub:

<div style="position:relative;display:flex;gap:14px;align-items:flex-start;border:1px solid var(--color-hairline);border-radius:8px;padding:16px 18px;margin:1.25rem 0;">
  <a href="https://github.com/JohnSpencerTerry/clinical-guideline-assistant" target="_blank" rel="noopener" aria-label="View JohnSpencerTerry/clinical-guideline-assistant on GitHub" style="position:absolute;inset:0;z-index:1;"></a>
  <svg viewBox="0 0 16 16" width="28" height="28" fill="var(--color-ink-mid)" style="flex:none;margin-top:2px;position:relative;" aria-hidden="true"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.01 8.01 0 0 0 16 8c0-4.42-3.58-8-8-8Z"/></svg>
  <div style="position:relative;">
    <div style="font-weight:600;font-size:15px;color:var(--color-ink);">JohnSpencerTerry/clinical-guideline-assistant</div>
    <div style="font-size:13.5px;color:var(--color-muted);margin-top:4px;line-height:1.45;">Grounded Q&amp;A over Type 2 Diabetes clinical guidelines (ADA, NICE) &mdash; a LangChain/LangGraph learning project.</div>
    <div style="font-size:12.5px;color:var(--color-muted);margin-top:10px;display:flex;align-items:center;gap:6px;">
      <span style="width:10px;height:10px;border-radius:50%;background:#3572A5;display:inline-block;"></span> Python
    </div>
  </div>
</div>

This project focused on Type 2 diabetes and draws on two sources: the ADA's Standards of Care and the UK's NICE guideline (NG28), both public and well-structured. The two sources can agree, disagree, or stay silent on a question. That range is what the comparison and classification part of the graph handles.

The live demo is embedded below. The repo's [example-prompts.md](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/example-prompts.md) has a longer prompt list by category. Four categories map directly onto the graph's shape: grounded factual recall, cross-source comparison, the scope guardrail, and urgent/emergency detection.

<div style="margin:1.5rem 0;">
  <iframe src="https://john-spencer-terry-clinical-guideline-assistant.streamlit.app/?embed=true" style="width:100%;height:600px;border:1px solid var(--color-hairline);border-radius:8px;" loading="lazy" title="Clinical Guideline Assistant &mdash; live demo"></iframe>
</div>

<p style="font-size:13px;color:var(--color-muted);margin-top:-0.75rem;">If the embed doesn't load, open the <a href="https://john-spencer-terry-clinical-guideline-assistant.streamlit.app/" target="_blank" rel="noopener">demo</a> directly.</p>

## The flow, at a glance

This is the actual graph, straight out of the project's design doc ([mvp.md](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/mvp.md)):

<pre class="mermaid">
flowchart TD
    Start([User Question]) --> KW[urgent_check_keyword]
    KW -->|hit| ER[emergency_redirect]
    KW -->|no hit| LLMU[urgent_check_llm]
    LLMU -->|hit| ER
    LLMU -->|no hit| SC[scope_classifier]
    SC -->|patient-specific| SR[scope_redirect]
    SC -->|general question| RET[retrieve_per_source]

    RET --> RA[retrieve: ADA]
    RET --> RN[retrieve: NICE]

    RA --> EA[extract_structured_claim: ADA]
    RN --> EN[extract_structured_claim: NICE]

    EA --> CMP[compare_claims]
    EN --> CMP

    CMP -->|same| SYN1[synthesize: unified answer]
    CMP -->|scope difference| SYN2[synthesize: explain scope]
    CMP -->|conflict| SYN3[synthesize: present both + grounded rationale if stated]
    CMP -->|silent| SYN4[synthesize: note gap]

    SYN1 --> End([Answer + Citations])
    SYN2 --> End
    SYN3 --> End
    SYN4 --> End
    ER --> End
    SR --> End
</pre>

Grounded factual recall walks the main spine down through `compare_claims`. Cross-source comparison is what the four `synthesize` branches are for. The scope guardrail and urgent/emergency detection each trigger a different redirect, short-circuiting straight to `End`.

## Setting up the system

Model access goes through [OpenRouter](https://openrouter.ai) rather than directly against a provider API, mostly so the whole thing can run on a free-tier model with the key's own credit limit set to $0 — a request can fail, but it can't bill anything. Embeddings run locally via `sentence-transformers`, so indexing doesn't need an API key either, just CPU time.

The source documents are downloaded once as PDFs and read from disk rather than scraped live. Chunking is content-aware rather than one-size-fits-all: NICE's text is already atomic per numbered recommendation, so short documents pass through unchanged, while ADA's long narrative sections get recursively split, with each chunk still carrying its parent section's metadata so a citation can point back to where it came from (code: [`chunking.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/ingestion/chunking.py)).

The chunks land in two separate vector indices, one per source, instead of a single pooled index. A pooled index would retrieve passages from both sources and hand them to the model together, inviting it to blend them into one confident answer even when the sources disagree. Separate indices let the graph retrieve from ADA and NICE independently and compare what each one actually says.

A Streamlit chat UI sits on top: `uv run streamlit run app/streamlit_app.py`.

## Grounded factual recall

A prompt such as "What is the first-line pharmacologic treatment for type 2 diabetes?" produces a metformin recommendation cited to ADA section 13, plus a note that the retrieved NICE passage doesn't explicitly address a first-line drug for this framing of the question. Both sides stay grounded, instead of the model filling in the NICE side from what it already knows about diabetes drugs.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/first-line-treatment.png" alt="Demo screenshot: question 'What is the first-line pharmacologic treatment for type 2 diabetes?' answered with a metformin recommendation, a note that the NICE excerpt doesn't explicitly state a first-line drug, and sources [13] [1.45.2]." style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;text-align:center;">Citations rendered under the answer.</figcaption>
</figure>

Behind the scenes, the graph retrieves from the ADA index and the NICE index independently, then runs each source's passages through an extraction step that turns them into a structured claim rather than free text: a recommendation, the population it applies to, an evidence grade where the source states one, and a citation identifier (code: [`extraction.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/extraction.py)).

With both claims in hand, a comparison node checks whether they agree, disagree, or one source is silent, and synthesis writes the answer to match. Every claim traces back to a specific citation, rendered under the answer: `[13] [1.45.2]` above, ADA's section number and NICE's recommendation number. Checking a claim against the source text is the whole grounding argument. That makes citations the one piece of the frontend worth getting right.

## Cross-source comparison

A prompt such as "How do ADA and NICE differ on managing chronic kidney disease risk in type 2 diabetes?" produces two intact, separately cited recommendations instead of one blended answer. ADA's is broad: incorporate agents that lower cardiovascular and kidney risk regardless of their effect on blood glucose. NICE's is narrower and more specific: metformin plus an SGLT-2 inhibitor, and only once eGFR is above 30.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/ckd-comparison.png" alt="Demo screenshot: question about ADA vs NICE on chronic kidney disease risk, answered with ADA's broad glucose-agnostic guidance in one paragraph and NICE's specific metformin plus SGLT-2 inhibitor recommendation in another, plus a closing paragraph naming the key difference." style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;text-align:center;">Both positions kept intact, not merged.</figcaption>
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
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;text-align:center;">Rendered as a distinct warning bubble, not a normal answer.</figcaption>
</figure>

The harder part of this guardrail is holding that boundary without over-triggering. A prompt such as "If I have CKD, does that change treatment recommendations?" is phrased with "I," but it should still get answered as a general question, because it's asking about a population, not requesting advice for a real, specific situation. The scope classifier's prompt spells out that distinction directly, with worked examples on both sides of the line (code: [`scope_classifier.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/guardrails/scope_classifier.py)).

There's a second layer behind the classifier, too. The synthesis prompt restates the system's role on every single call, general or not: "You explain what published clinical guidelines say. You do not provide individualized medical advice." If a borderline message slips past the classifier, generation is still self-limited by that same constraint. Neither layer is trusted alone to hold the line.

## Urgent/emergency detection

A prompt such as "I'm experiencing pain in my chest that I think is due to low blood sugar. Can I take aspirin alongside my normal diabetes medication?" produces an immediate redirect to emergency care, no retrieval involved, even though the second half of the message is phrased like an ordinary medication question. The guardrail sits at the very front of the graph, before any of the RAG machinery runs, and the answer comes back rendered distinctly in the UI rather than as a normal chat bubble.

<figure class="diagram-figure">
  <img src="/assets/photos/clinical-guideline-assistant/emergency-redirect.png" alt="Demo screenshot: message 'I'm experiencing pain in my chest that I think is due to low blood sugar. Can I take aspirin alongside my normal diabetes medication?' answered with an emergency redirect telling the user to call 911 or go to the nearest emergency room." style="max-width:100%;height:auto;border-radius:6px;border:1px solid var(--color-hairline);" />
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;text-align:center;">Triggered before the medication question reaches retrieval.</figcaption>
</figure>

The check runs in two stages: a fast keyword/pattern match first, then an LLM classifier that only runs if the keywords don't hit, specifically to catch phrasing a fixed pattern list wouldn't (code: [`urgent_check.py`](https://github.com/JohnSpencerTerry/clinical-guideline-assistant/blob/main/src/cga/graph/guardrails/urgent_check.py)). This particular message is phrased as "pain in my chest" rather than the literal "chest pain" the keyword pattern looks for, so it's the LLM classifier stage that catches it — and it catches the whole message, guideline question and all, before any of it reaches retrieval. Both stages are deliberately biased toward over-triggering. A false positive costs a mildly annoying redirect. A false negative means an emergency gets a calm, cited RAG answer instead of "call 911." The LLM classifier's own prompt says outright to bias toward "yes" if it's unsure, instead of aiming for the balanced precision and recall a classifier would normally target.

## Checking that the guardrails hold

A demo only proves a handful of hand-picked examples work. The eval suite checks the same guardrail and disagreement-detection claims against a full question set.

The question set is hand-written, read directly out of the ADA and NICE source text rather than generated. It's split across six categories: grounded factual recall, cross-source comparison, structured extraction, the scope guardrail, urgent-symptom detection, and adversarial edge cases like a hypothetical reframed specifically to try to slip past the scope classifier. The guardrail categories grade pass/fail deterministically, since "did it trigger or not" has a clean right answer. Comparison and recall are run-and-reported for now, since grading those well needs reference answers this project hasn't fully built out yet.

## What this was for

The diabetes framing gave the project real stakes to build against. Each category above maps to a distinct LangGraph pattern: short-circuiting guardrail nodes ahead of the main flow, retrieval and extraction run independently per source, a classification node whose output determines routing, and synthesis that branches on that classification instead of writing one generic prompt. Building those five pieces was the goal.

Medical Q&A works here for two reasons. The answer space is limited to two named sources, not the model's general knowledge. A layer of guardrails decides what's answerable at all before anything reaches an LLM. Neither piece works alone. Retrieval without guardrails answers a patient-specific or urgent question as confidently as a general one. Guardrails without narrow retrieval still risk blending or inventing guidance the sources never gave.

This is a learning project, not a clinical tool. The guardrails are a reasonable first pass, not a substitute for the regulatory, legal, and clinical review a real deployment would need.
