---
layout: post
title: "Learning LangGraph by building a clinical guideline assistant."
date: 2026-08-28
tags: [Tech, AI, Side Project]
draft: true
---

This one's a learning project: I wanted real depth with LangChain and LangGraph, on something with enough moving parts that the framework's primitives earn their keep. Grounded Q&A over clinical guidelines turned out to be a good vehicle for that. It needs guardrails, retrieval, structured extraction, a classification step, and an answer that changes shape depending on that classification, which is a lot more graph than a single retrieval chain.

The domain is Type 2 diabetes, and the two sources are the ADA's Standards of Care and the UK's NICE guideline (NG28). They're both public and well-structured, and usefully for this project, they don't always agree. NICE tends to push toward dual therapy earlier than the ADA's more staged approach. That disagreement is the thing that makes the comparison and classification part of the graph worth building instead of being a toy detail.

There's a live demo below. Four things to try: a general guideline question, a question where the two sources actually differ, something that sounds like a medical emergency, and something that asks for advice about a specific person. Each one exercises a different part of the graph, described alongside it.

## Setting up the system

Model access goes through [OpenRouter](https://openrouter.ai) rather than directly against a provider API, mostly so the whole thing can run on a free-tier model with the key's own credit limit set to $0 — a request can fail, but it can't bill anything. Embeddings run locally via `sentence-transformers`, so indexing doesn't need an API key either, just CPU time.

The source documents are downloaded once as PDFs and read from disk rather than scraped live. Chunking is content-aware rather than one-size-fits-all: NICE's text is already atomic per numbered recommendation, so short documents pass through unchanged, while ADA's long narrative sections get recursively split, with each chunk still carrying its parent section's metadata so a citation can point back to where it came from.

```python
def chunk_documents(
    documents: list[Document], chunk_size: int = 1000, chunk_overlap: int = 150
) -> list[Document]:
    splitter = RecursiveCharacterTextSplitter(chunk_size=chunk_size, chunk_overlap=chunk_overlap)
    result: list[Document] = []

    for doc in documents:
        if len(doc.page_content) <= chunk_size:
            result.append(doc)
            continue

        pieces = splitter.split_text(doc.page_content)
        for i, piece in enumerate(pieces):
            result.append(
                Document(
                    page_content=piece,
                    metadata={**doc.metadata, "chunk_index": i, "chunk_count": len(pieces)},
                )
            )

    return result
```

The chunks land in two separate vector indices, one per source, instead of a single pooled index. That's a small-looking decision that everything downstream depends on: a pooled index would retrieve passages from both sources and hand them to the model together, which invites blending them into one confident answer even when the sources disagree. Keeping the indices separate is what makes it possible to retrieve from ADA and NICE independently and compare what each one actually says.

A Streamlit chat UI sits on top: `uv run streamlit run app/streamlit_app.py`.

## Try it: ask a general guideline question

Start with something ordinary, like "what's the recommended first-line treatment for someone newly diagnosed with type 2 diabetes?" This is the baseline case, and it's worth trying first because it proves the grounding actually works before the more interesting paths show up.

Behind the scenes, the graph retrieves from the ADA index and the NICE index independently, then runs each source's passages through an extraction step that turns them into a structured claim rather than free text:

```python
class Claim(BaseModel):
    source: str
    recommendation: str = Field(description="The guideline's recommendation, in the model's own words.")
    population: str = Field(description="Who this recommendation applies to.")
    evidence_grade: str | None = Field(default=None, description="Evidence grade/level, if the source states one.")
    stated_rationale: str | None = Field(
        default=None,
        description="The reason given IN THE SOURCE TEXT for this recommendation, if any. Never inferred.",
    )
    citation: str = Field(description="Section/recommendation identifier to cite back to.")
```

With both claims in hand, a comparison node checks whether they agree. For a question like this one they usually do, so the graph lands on "same" and synthesis writes one clean answer, citing both sources by section and recommendation number. Those citations are the point: the chat UI renders them under the answer, and it's the one piece of the frontend worth getting right, because being able to check the claim against the actual source text is the whole grounding argument.

## Try it: ask where the guidelines actually differ

Now ask about dual-therapy timing, or anything else where NICE and the ADA genuinely part ways. This is the behavior a pooled-index RAG bot can't give you: instead of averaging two positions into a plausible-sounding blend, the answer presents both, unresolved, with a small "guidelines differ here" badge in the UI.

The comparison node is doing real work here, and its job is narrow on purpose:

```python
_PROMPT = """Compare these two guideline claims answering the same question and classify their relationship.

ADA claim: {ada}
NICE claim: {nice}

Classify as exactly one of:
- "same": both sources recommend the same thing for the same population.
- "scope_difference": they differ only because of population/scope/phrasing, not a real conflict.
- "conflict": the sources genuinely recommend different things for the same scenario.
- "silent": one source doesn't address this at all (population is "not addressed").

Classify only — do not explain or resolve the difference.
"""
```

That last line matters. It's tempting to let the classifier also explain the disagreement while it's looking at both claims, but folding resolution into classification is how you end up with a model quietly deciding which guideline is "right." Splitting them keeps the classification precise and pushes the harder judgment call, how to present a genuine conflict without picking a winner, into synthesis, where it's a separate, explicit branch:

```python
"conflict": (
    "The sources genuinely disagree. Present both positions without picking a winner. "
    "Explain WHY they differ ONLY if a claim's stated_rationale field is non-null — quote or "
    "paraphrase that stated rationale. If both stated_rationale fields are null, explicitly say "
    "the sources don't state a reason for the difference. NEVER invent a rationale."
),
```

The `stated_rationale` field is set during extraction, and only when the source text itself gives a reason. If neither guideline explains why they diverge, the answer says that plainly instead of the model reaching for a plausible-sounding explanation. In a domain where a fabricated "why" reads as authoritative, that's the constraint that matters most in the whole system, and it's enforced as a hard rule in the prompt rather than left as a suggestion.

`scope_difference` is a separate bucket for cases that look like disagreement but aren't — different eGFR thresholds framed differently, say, rather than an actual conflict about what to do. Collapsing that into "conflict" would manufacture a disagreement that isn't real; collapsing it into "same" would flatten a distinction that matters. Synthesis has its own instructions for that branch too, explaining the scope difference explicitly instead of picking either failure mode.

## Try it: ask something that sounds like an emergency

Try a message like "I haven't been able to catch my breath since this morning." Nothing here should go anywhere near retrieval. The guardrail sits at the very front of the graph, before any of the RAG machinery runs, and the answer comes back as a direct redirect to emergency care, rendered distinctly in the UI rather than as a normal chat bubble.

The check is two stages: a fast keyword/pattern match first, and an LLM classifier that only runs if the keywords don't hit, specifically to catch paraphrased urgency that wouldn't match a fixed pattern list.

```python
def is_urgent(llm: BaseChatModel, message: str) -> bool:
    return keyword_check(message) or llm_check(llm, message)
```

Both stages are deliberately biased toward over-triggering. A false positive costs a mildly annoying redirect; a false negative means an emergency gets a calm, cited RAG answer instead of "call 911." That asymmetry is why the LLM classifier's own prompt says outright to bias toward "yes" if it's unsure, rather than aiming for balanced precision and recall the way a classifier normally would.

## Try it: ask for advice about a specific person

Try "what should I do about my dad's diabetes?" This one isn't urgent and isn't off-topic, but it's still a question the system shouldn't answer directly, because it's asking for individualized medical advice rather than what a guideline says in general. The response reframes it back to the general version of the question and points the personal situation toward an actual care team.

The harder part of this guardrail is the boundary it has to hold without over-triggering. "If I have CKD, does that change treatment?" reads as personal — it's phrased with "I" — but it's really asking about a population, not requesting advice for a real, specific situation. The scope classifier's prompt spells that distinction out directly:

```python
_PROMPT = """Classify the following message as one of two categories:

- "general": asks what published clinical guidelines say (may be phrased conversationally, e.g. \
"if I have CKD, does that change treatment?" is still general — it's asking about a population/scenario, \
not requesting advice for a specific named individual's actual situation).
- "patient_specific": asks for individualized medical advice about a specific real person (the speaker, \
a named family member, "my" situation with personal details), e.g. "should I stop taking my metformin?" \
or "what should my dad do about his diagnosis?".
"""
```

There's a second layer behind the classifier, too. The synthesis prompt restates the system's role on every single call, general or not: "You explain what published clinical guidelines say. You do not provide individualized medical advice." If a borderline message slips past the classifier, generation is still self-limited by that same constraint. Neither layer is trusted alone to hold the line.

## Checking that the guardrails hold

A demo only shows that a few hand-picked examples work. The eval suite is what turns "the guardrails work" and "it surfaces disagreement correctly" into something checkable rather than a claim.

The question set is hand-written, read directly out of the ADA and NICE source text rather than generated, and split across six categories: grounded factual recall, cross-source comparison, structured extraction, the scope guardrail, urgent-symptom detection, and adversarial edge cases like a hypothetical reframed specifically to try to slip past the scope classifier. The guardrail categories grade pass/fail deterministically, since "did it trigger or not" has a clean right answer. Comparison and recall are run-and-reported for now, since grading those well needs reference answers this project hasn't fully built out yet.

## What this was for

The diabetes framing gave the project real stakes to build against. But the four demo steps are the actual deliverable: short-circuiting guardrail nodes ahead of the main flow, retrieval and extraction run independently per source, a classification node whose output determines routing, and synthesis that branches on that classification instead of writing one generic prompt and hoping. That's what was worth building, and worth the LangChain and LangGraph practice it took to get right.

It's still exactly what the README says it is: a learning project, not a clinical tool. The guardrails here are a reasonable first pass, not a substitute for the regulatory, legal, and clinical review a real deployment would need.
