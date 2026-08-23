---
layout: post
title: "FHIR and BCDA."
date: 2026-05-29
tags: [Tech]
draft: true
---

<!--
OUTLINE

1. What FHIR is, and where it came from
   - REST-based standard: healthcare concepts (Patient, Claim, ExplanationOfBenefit) modeled
     as discrete resources, fetched over HTTPS, JSON or XML
   - Brief history: HL7 started work on it in 2011, revised through Release 5 (2023); built to
     be easier to implement than the older HL7 v2/v3 document-based standards it's succeeding
   - FHIR is one standard among many in healthcare data — link the xkcd "standards" comic as
     the acknowledgment that FHIR doesn't make the proliferation problem go away, it's just
     the current widely-adopted answer

2. CMS runs more than one feed, and they're not interchangeable
   - CCLF: monthly, fixed-width tabular files, sourced from the Integrated Data Repository,
     accessed manually/via command line
   - BCDA: FHIR-formatted, API-based, adjudicated claims weekly and partially adjudicated
     claims daily, sourced from CCW plus FISS/MCS
   - Ground this in the reason for building around BCDA at work: it's a materially lower
     latency source than CCLF for the same underlying claims

3. What partially adjudicated data actually is, and the pipeline that produces it
   - Recreate the CMS pipeline diagram as an inline SVG: Provider submission → CMS validates/
     aggregates → Shared Systems Software → Common Working File → Chronic Conditions
     Warehouse → BCDA (partially adjudicated at 2-4 days via the Beneficiary FHIR Data
     Server; fully adjudicated at 8-14 days)
   - The two FHIR resource types here are Claim and ClaimResponse, covering Medicare Parts
     A and B

4. Use cases this actually unlocks
   - Transition of care: ACOs spot recent hospital discharges within days instead of weeks,
     to coordinate follow-up and cut readmissions
   - Intervention opportunities: procedures flagged for review get caught early enough for a
     provider to intervene before unnecessary treatment happens
   - Care coordination: building a fuller patient history by tracking services across
     providers faster than the monthly CCLF cadence allows

5. The resources aren't vanilla FHIR (brief, not the main point)
   - CMS's BCDA responses follow the CARIN Blue Button Implementation Guide's EOB profile,
     which adds required fields and constraints on top of the base FHIR ExplanationOfBenefit
     resource (status/use restrictions, a focal-coverage invariant, required identifiers)
   - The practical point: build against the profile a source actually uses, not the base spec

6. Closing
   - Tie back to the opening: FHIR gives healthcare data a common shape, but "FHIR-compliant"
     is a spectrum defined by whichever implementation guide a source has layered on top

7. Update — BCDA v3
   - Dated note: BCDA v3 launched July 1, 2026, after this piece was originally written
   - Key change: BCDA v3 lets claims be linked back to their CCLF counterparts, closing the
     gap between the two feeds this piece opens with
   - Also new: the _typeFilter parameter, consolidated FHIR resources for partially
     adjudicated claims, new endpoints/mapping; v1/v2 access ends July 30, 2027
-->

At work, we built an ingest pipeline against CMS's Beneficiary Claims Data API (BCDA), pulling ExplanationOfBenefit, Claim, and Patient data to give downstream systems a lower-latency source than CMS's older claims feed. Getting that pipeline right meant understanding both FHIR, the format BCDA speaks, and the specific way CMS implements it.

## What FHIR is, and where it came from

FHIR (Fast Healthcare Interoperability Resources) models healthcare concepts, a patient, a claim, an explanation of benefits, as discrete resources. Each resource has its own URL, and you fetch or manipulate it over a standard REST API, in JSON or XML.

HL7 International, the standards body behind the older HL7 v2 and v3 healthcare messaging formats, started work on FHIR in 2011 and has been revising it since, most recently Release 5 in 2023. Where HL7 v2 and v3 relied on complex document-based messaging, FHIR's pitch was a standard healthcare engineers could work with using the same REST conventions they'd use anywhere else.

FHIR isn't the only standard in healthcare data (see [xkcd 927](https://xkcd.com/927/)). It's the current widely-adopted one.

## CMS runs more than one feed, and they're not interchangeable

CMS exposes Medicare claims data through more than one channel, and the two most relevant here don't behave the same way.

CCLF (Claim and Claim Line Feed) is CMS's older channel: fixed-width tabular files, generated monthly, sourced from CMS's Integrated Data Repository, and pulled down manually or by script.

BCDA is newer and FHIR-native: claims come back as FHIR resources over a REST API, adjudicated claims refresh weekly, and partially adjudicated claims refresh daily. It's sourced from the Chronic Conditions Warehouse for adjudicated data and from CMS's FISS and MCS systems for partially adjudicated data.

That latency gap is the whole reason we built around BCDA. A monthly file is fine for retrospective reporting. BCDA's weekly-to-daily refresh is what makes quick response to claim updates possible, against the same underlying Medicare data CCLF eventually reports too.

## The pipeline behind partially adjudicated data

Partially adjudicated claims exist because CMS's full adjudication process takes time, and BCDA exposes an earlier snapshot of that process instead of waiting for it to finish.

<figure class="media-figure">
  <img src="/assets/photos/bcda/adjudication-data-flow.svg" alt="CMS diagram of the claims pipeline from provider submission through CMS to BCDA and CCLF, showing partially adjudicated data shared at 2-4 days and fully adjudicated data at 8-14 days." />
  <figcaption>CMS's diagram of the BCDA and CCLF pipelines.</figcaption>
</figure>

A provider submits a claim after delivering care. A Medicare Administrative Contractor validates and aggregates it. From there the claim moves through CMS's Shared Systems Software and into the Common Working File, which feeds both channels: CCLF through the Integrated Data Repository, and BCDA through the Chronic Conditions Warehouse and the Beneficiary FHIR Data Server. BCDA shares partially adjudicated data 2 to 4 days after submission, and fully adjudicated data 8 to 14 days after. The two resource types behind partially adjudicated data are `Claim` and `ClaimResponse`, covering Medicare Parts A and B.

## Use cases this unlocks

A few days of lead time changes what's practical to act on. ACOs use partially adjudicated data to spot a recent hospital discharge within days instead of weeks, in time to coordinate follow-up care and reduce readmissions. Procedures flagged for review show up early enough for a provider to intervene before an unnecessary treatment happens, instead of finding out after the fact. And because the data refreshes faster than CCLF, it supports building a fuller patient history sooner, tracking services across providers on a timeline that's actually current.

## The resources aren't vanilla FHIR

BCDA's `ExplanationOfBenefit` responses don't follow the base FHIR `ExplanationOfBenefit` resource. They follow the [CARIN Blue Button Implementation Guide's C4BB profile](https://build.fhir.org/ig/HL7/carin-bb/en/StructureDefinition-C4BB-ExplanationOfBenefit.html), which adds its own requirements on top: required claim identifiers, patient and insurer references, at least one coverage entry, `.status` restricted to `active` or `cancelled`, `.use` required to be `claim`, and at most one coverage entry marked `focal = true`.

None of that is visible if you build a parser against the base FHIR spec and assume it covers what BCDA sends back. The practical rule is to build against the profile a source actually implements, CARIN in this case, not the base resource definition it's layered on top of.

## Closing

FHIR gives healthcare data a common shape: resources, a REST API, a consistent way to model a claim or a patient. What "FHIR-compliant" means in practice depends on which implementation guide a given source has built on top of that shape. BCDA speaks FHIR, but the resource you get back is CARIN's version of it, produced by a pipeline with its own timing, feeding into a use case that only works because that pipeline is faster than the alternative.

## Update: BCDA v3

*This section was added after the piece was originally written.* BCDA v3 launched July 1, 2026. The change most relevant to everything above is that v3 lets claims be linked back to their CCLF counterparts, closing part of the gap between the two feeds described in this piece. Also new: the `_typeFilter` parameter for filtering claims data, consolidated FHIR resources for partially adjudicated claims, and new endpoints and mappings for teams migrating over. Access to v1 and v2 ends July 30, 2027.