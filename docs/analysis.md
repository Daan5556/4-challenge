# Analysis

**Author:** Daan Eggen  
**Date:** 12/09/2026  
**Version:** 1.0

---

## Project context

- The [challenge](sources/challenge.md) concerns the Visioball, a ball developed
  for Koninklijke Visio through Industrial Design Engineering at Fontys Venlo.
- The ball produces sound to help blind and partially sighted users locate it
  and receive feedback about its movement.
- The challenge identifies a need for a user-friendly way to select sounds and
  manage settings, alongside safe communication between an app and the ball.
- My role is IT infrastructure and backend web developer. My intended
  contribution is a server platform with social features and CDN integration.
- The current project direction is that guardians use the app. Direct use of
  the app by blind and partially sighted users is outside my current focus.
- The original challenge includes accessibility deliverables. The guardian-led
  direction and its effect on these deliverables need stakeholder confirmation.

## Problem and objective

- The challenge does not define a server platform or social features. Their
  value and connection to sound selection and settings management must first
  be established.
- The analysis should determine which guardian needs justify shared online
  functionality and what infrastructure is needed to support it.
- The desired result is a supported set of requirements, a clear description of
  information flows, and evidence for later architecture and technology advice.
- Cloudflare is a candidate for CDN integration. No provider, backend stack or
  hosting environment has been selected.

## Research questions

- Main question: What requirements should a server platform with social
  features and CDN integration meet to support guardians using the Visioball?
- RQ1: What do the client and guardians need, and which social features would
  provide value within the challenge?
- RQ2: How should information move between guardians, the app, the backend,
  content storage and the ball, including when connectivity is unavailable?
- RQ3: What can comparable products and existing technical approaches teach us
  about shared content, account management and platform operation?
- RQ4: Which requirements for security, availability, performance, scalability,
  cost and maintainability follow from the intended use?
- RQ5: Where would a CDN provide measurable value, and how do suitable options,
  including Cloudflare, compare against the requirements?

## Scope

- Investigate guardian workflows, backend responsibilities, infrastructure and
  the interfaces with the app.
- Explore social features with stakeholders. Sharing sound presets, profiles
  and groups are discussion examples, not agreed requirements.
- Identify which actions require a server and which belong to local app-to-ball
  communication. Do not assume that real-time ball control passes through the
  backend or CDN.
- Include the hardware and app teams when identifying integration constraints.
- Detailed app interface design and ball electronics development fall outside
  my contribution, but their constraints can affect the platform requirements.

## Research methods

- Field: interview the client and representative guardians about current sound
  selection, settings management and possible shared activities (RQ1, RQ2).
  This establishes actual needs before selecting features. Record examples,
  priorities and disagreements, then ask participants to check the summary.
- Library: compare relevant products and review official technical documentation
  for backend, storage and CDN approaches (RQ3, RQ5). This identifies existing
  solutions and constraints. Record sources, dates and consistent comparison
  criteria; distinguish documented claims from measured behaviour.
- Workshop: map guardian workflows and information flows with the app and
  hardware contributors (RQ2, RQ4). This exposes unclear responsibilities and
  dependencies. Produce annotated process and data-flow diagrams, including
  failure cases, and record participant feedback.
- Lab: run a small content-delivery experiment once representative assets and
  quality targets are known (RQ4, RQ5). Compare an origin-only baseline with
  CDN delivery using the same assets and conditions. Record cache state,
  response times, errors, transferred data and configuration so results can be
  reproduced. Keep findings limited to the tested conditions.
- Showroom: review the proposed requirements and comparison findings with a
  relevant technical expert and stakeholders (RQ1, RQ4, RQ5). This checks their
  relevance and feasibility. Record feedback and resulting revisions.
- Combine stakeholder findings with literature and experiments. Investigate
  conflicting evidence rather than treating a single method as sufficient.

## Processes and information flows

- Document how a guardian currently selects a sound and changes settings, who
  is involved, and where problems occur.
- For each proposed social workflow, identify the initiating user, permissions,
  information created, intended recipients and expected result.
- Map the app, API, storage, possible CDN and ball as separate components, with
  the direction and purpose of each information exchange.
- Identify which information is public, private or shared with a specific group,
  if those categories are needed. Establish ownership and access rules before
  deciding what may be cached.
- Investigate what happens when the ball disconnects, the backend is unavailable,
  content changes or a download fails.
- Identify whether sound files or other assets are transferred at all, their
  expected sizes and update frequency, and the devices that consume them.

## Requirements and comparison criteria

- Give each requirement an identifier, source, rationale, priority and acceptance
  criterion. Mark it as proposed or stakeholder-confirmed.
- Derive measurable performance and availability targets from expected use;
  values are not yet agreed.
- Establish expected user numbers, simultaneous activity, content volume and
  budget before assessing capacity and cost.
- Investigate account permissions, protection of private information and content
  management responsibilities for the selected social features.
- Compare suitable technical options against the same criteria: functional fit,
  performance, availability, security, scalability, cost and maintenance effort.
- Record limitations and missing evidence. Keep final technology recommendations
  traceable to findings for the Advice learning outcome.

## Portfolio evidence

- Keep an evidence record for each research activity with its date, research
  question, method, my contribution, findings, limitations and artifact links.
- Collect interview summaries and confirmed priorities as evidence of client and
  target-audience analysis.
- Collect a sourced product and technology comparison as evidence of research
  into existing work and the market.
- Collect process and data-flow diagrams, including revisions, as evidence of
  analysis of workflows and information exchanges.
- Collect the experiment setup, raw measurements and interpretation as evidence
  of technical investigation.
- Collect the requirements and review feedback as evidence that findings were
  checked and translated into useful project input.
- Add my own reflection after each activity: what I learned, what remains
  uncertain and how the findings changed my next steps.
- Current coverage: the assignment and learning-outcome sources have been
  reviewed. The research artifacts above are planned and are not yet linked as
  completed evidence.

## Open questions

- Who can confirm the scope on behalf of IDE and Koninklijke Visio, and which
  guardians can participate in research?
- Which social use case should the first prototype support?
- How does the guardian-led app direction align with the original deliverables?
- What interfaces and limitations does the existing ball or hardware mockup have?
- What budget, hosting constraints, expected usage and quality targets apply?
- Is Cloudflare being considered only for CDN delivery or also for backend
  hosting and storage?

## Next steps

- Confirm the research scope and arrange stakeholder interviews.
- Use the findings to select a first social workflow and document its information
  flows with the app and hardware contributors.
- Establish comparison criteria and quality targets, then perform the library
  research and content-delivery experiment.
- Review the resulting requirements, record revisions and link the completed
  evidence in this document.
