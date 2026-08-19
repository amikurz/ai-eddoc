# AI-EDdoc

Emergency department clinical decision support. The system proposes; a physician
signs. Nothing it produces becomes an active order, a prescription, or a
discharge without a clinician's signature.

**Status: in development against synthetic data only.** No real patient
encounter has been processed by this software.

---

## What it does

1. **Structured voice intake at arrival.** A multilingual kiosk conversation
   captures the patient's own words. Ten languages, with the native transcript
   shown alongside an English translation so an accompanying family member can
   verify what was heard before the patient confirms it. A qualified medical
   interpreter is one press away at every point, and machine translation is
   never presented as a substitute for one.

2. **A triage note and a differential.** Can't-miss diagnoses are named
   explicitly rather than reached by exclusion.

3. **A proposed diagnostic workup.** Every proposed order carries a resolvable
   link to the guideline it rests on, so the clinician can independently verify
   the basis for each recommendation.

4. **A physician signature.** The difference between what was proposed and what
   was actually signed is recorded, and is the system's primary quality signal.

## Integration posture

**Reads over FHIR R4. No FHIR writes.** The system reads demographics, vitals,
allergies, problems, medications and results. It has no write path to the chart
and is not designed to grow one.

**Orders reach the clinician one of two ways.**

- *Primary — pended orders.* The AI note is filed to the chart and the order set
  is pended under a dedicated service account holding a security class that
  permits pending an order and forbids signing one. The physician reviews and
  signs in Hyperspace, where Epic's own allergy, interaction and duplicate
  checks run at signature. The guardrail lives in Epic's permission model, not
  in our code, and is auditable in Epic's own log.
- *Fallback — CDS Hooks.* Suggestion cards for sites without a service account
  or interface engine. `ServiceRequest.intent` is always `proposal`, never
  `order`: the clinician's acceptance is what creates an order.

**Authentication.** SMART Backend Services, RS384-signed client assertion. No
user is present at 3 a.m., which is why this is a backend integration rather
than a SMART launch.

## Safety properties

- Contraindication checks fail **closed**. Unknown renal function blocks a
  contrast study; it does not assume normal.
- An unconfirmed transcript is never reasoned about. A speech-recognition error
  the patient has not seen must not become a diagnosis.
- The report narrative governs interpretation, not the interface flag. A study
  reading "free intraperitoneal air" flagged normal still raises a critical
  finding.
- The system performs no physical examination and cannot. Any generated text
  asserting an examination finding is removed and surfaced to the physician as
  an unsupported claim.
- Provenance travels with everything: whether a model response came from a live
  endpoint, and whether a result was resulted by the EMR or entered by an
  operator during a demonstration.

## Regulatory

Designed against the FDA non-device clinical decision support criteria: the
basis for every recommendation is surfaced so that a clinician can independently
review it rather than relying on the software's output. Production inference on
identified data requires a Business Associate Agreement. A prospective clinical
pilot is human-subjects research and requires IRB review.

## This repository

Contains the public JWK Set used to verify this application's signed client
assertions, and this document. **It contains no source code and no private key
material.** `jwks.json` holds only public key parameters and is published
deliberately.

## Contact

Ami Kurzweil, MD — kurzweilconsulting@gmail.com
