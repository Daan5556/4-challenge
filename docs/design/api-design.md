# Game API functional design

**Date:** 19/09/2026  
**Version:** 0.1 — proposed prototype design

## Purpose and status

- Provide a small HTTP API through which the guardian's app can save a game
  definition and retrieve it using its identifier.
- Confirmed by Daan's request: exactly two operations, create and retrieve by ID;
  a non-incremental identifier; a title, optional picture, timestamp and JSON
  object describing the game. “GUI” is interpreted as UUID/GUID.
- The remaining choices below are proposed defaults, not stakeholder-approved
  requirements or implemented behaviour.
- The [standalone technical specification](api/game-api-spec.md) defines the
  external contract, payloads, validation and errors. It can be shared separately
  with an app developer without this document.
- This deliverable contains design only. No API, database or deployment has been
  implemented, and no runtime behaviour has been tested.

## Connection to the challenge

- The [original challenge](sources/challenge.md) asks for an app, communication
  with the ball, sound/settings control and supporting design and advice.
- Storing reusable game definitions could support the app's selection of
  activities and settings. This is a proposed extension, not a requirement in
  the original assignment. Its value needs confirmation with the app team and
  challenge stakeholders.
- The API stores information. The app and hardware team remain responsible for
  interpreting that information and communicating safely with the ball.
- A game here means a saved definition, not a played session, score or live match.
  The timestamp records creation of that definition, not when a game is played.
- The guardian-led app direction follows the [project analysis](analysis.md).
  Its relationship to the assignment's accessibility deliverables still needs
  stakeholder confirmation.

## Functional requirements

| ID  | Requirement                                              | Basis                                  | Acceptance criterion                                                            |
| --- | -------------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------- |
| F1  | Create a game through `POST /games`.                     | User request                           | A valid request creates one game and returns its complete representation.       |
| F2  | Retrieve a game through `GET /games/{id}`.               | User request                           | An existing ID returns the stored game; an unknown valid ID returns 404.        |
| F3  | Use non-incremental IDs.                                 | User request; UUIDv4 proposed          | Each creation receives a server-generated UUIDv4, never a sequential number.    |
| F4  | Store title, optional picture, timestamp and definition. | User request; field semantics proposed | Creation and retrieval follow the standalone specification.                     |
| F5  | Reject invalid input without creating a game.            | Proposed                               | Validation errors identify the relevant field and leave storage unchanged.      |
| F6  | Make a successful creation immediately retrievable.      | Proposed                               | After 201, a GET using the returned ID succeeds while the service is available. |

## User flows

- Create: the guardian enters a title and game settings in the app, optionally
  selects an already hosted picture, and saves. The app sends the definition;
  the server validates it, assigns the ID and creation time, stores it and
  returns the saved game. The app retains the ID for later retrieval.
- Retrieve: the app sends a known ID and displays the returned title, picture
  and settings. A missing picture or failed image download uses an app fallback
  and does not prevent retrieval of the game definition.
- Failure: the app keeps unsaved input on validation or connection failure and
  displays the error. An uncertain POST outcome requires care: a timeout may
  occur after the server has already saved the game.
- There is no list/search operation. If the app loses the ID, it cannot recover
  the game through this API. There are no edit, delete, upload, account, sharing,
  score or ball-control operations in this prototype.

## Choices and trade-offs

- HTTP with JSON keeps the interface small and usable from different app stacks.
  Use HTTPS for a hosted service; plain HTTP is acceptable for local development.
  The contract follows [HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html).
- Generate UUIDv4 IDs on the server using secure randomness. This satisfies the
  non-incremental requirement without exposing creation order. UUIDs are longer
  to copy than integers and random IDs have poorer index locality than ordered
  IDs. Enforce uniqueness and retry generation on collision; never overwrite an
  existing game. See [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html).
- Use `createdAt`, assigned by the server in UTC, to avoid relying on the app's
  clock. A scheduled or played-at timestamp would be a separate future concept.
- Represent the picture as an optional HTTPS URL. This avoids a third endpoint,
  file storage and upload handling. The app must already have a hosted URL;
  taking and uploading a new photo is not supported by this API.
- Treat `definition` as an opaque JSON object. This lets the app team experiment
  before agreeing on game rules. The server validates its shape and size, but
  cannot guarantee that its contents describe a playable or safe game. The app
  must validate settings before applying them to the ball.
- Games are immutable through this API. Saving a revision creates a new game
  and ID. This simplifies the prototype but leaves old definitions in storage.
- Use a maximum 64 KiB request body, title length of 1–120 Unicode code points,
  picture URL length of 2,048 characters and JSON nesting limit of 20 containers.
  These are provisional prototype limits to bound input, not measured capacity
  requirements. Confirm them using representative definitions.
- Reject unknown outer fields to catch integration mistakes while allowing
  arbitrary fields inside `definition`. Use consistent problem responses based
  on [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html).
- Start without application authentication only in a controlled demonstration
  using non-sensitive sample data. Anyone who can reach it can create games and
  retrieve a game whose ID they know. UUIDs do not provide authorization.
  Public release requires a separate access and abuse-control decision.

## Infrastructure and CDN

- Keep framework, database, hosting and provider selection open. The storage
  must retain successful creations across application restarts; its technology
  is outside this functional design.
- Return `Cache-Control: no-store` for API responses. Immediate retrieval then
  does not depend on cache freshness, and error responses must not be cached.
  A future CDN must respect this policy for the API routes.
- A CDN could deliver separately hosted pictures. The API only stores their
  URLs and does not fetch or proxy them. Image hosting, cache lifetime, removal
  and permitted origins remain separate infrastructure decisions.
- Cloudflare is a candidate alongside other providers; no service is selected.
  Later research should compare origin-only and CDN image delivery using the
  same assets, recording latency, availability, transfer volume and cost.

## Risks and responses

| Risk                                      | Prototype response and remaining limitation                                                                                                                                      |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Duplicate games after a POST retry        | Each accepted POST creates a new ID, even for identical content. The app must not silently retry an uncertain submission. Idempotency is a future extension.                     |
| Unauthorized access or excessive creation | Restrict the demo environment and use sample data. Before public use, decide authentication, ownership, rate limits and storage quotas.                                          |
| Invalid or dangerous game settings        | Store JSON as data; do not execute it. The app/hardware integration must enforce allowed settings before use. A shared definition schema remains open.                           |
| Untrusted or unavailable picture          | Validate URL syntax only; the API never fetches it. The app needs a fallback and must decide trusted image sources. External hosts may change content or observe image requests. |
| Lost IDs and accumulating data            | The app retains returned IDs. No in-API recovery or deletion exists; agree on demo cleanup and retention before collecting real content.                                         |
| Data loss or service failure              | Return 201 only after persistence succeeds; failed writes return an error. Backup, recovery and monitoring arrangements remain implementation work.                              |
| Incompatible future definitions           | Agree on a versioned game-definition format with the app team before multiple clients depend on it. No compatibility claim is made for arbitrary JSON.                           |

## Validation and research

- Library method, completed for this draft: review the challenge, project
  analysis, learning outcomes and linked HTTP, UUID and error-format standards.
  This separates assignment requirements from proposed interface conventions;
  it does not establish guardian needs.
- Workshop method, planned: walk through save/load and lost-ID flows with the
  app and hardware contributors. Check the meaning of a game, picture handling,
  timestamp and safe interpretation of settings; record decisions and revisions.
- Lab method, planned: implement the two operations and run the contract checks
  in the standalone specification. Record actual requests, responses and storage
  observations, including restart and failure cases.
- Showroom method, planned: ask a technical reviewer and challenge stakeholder
  to assess simplicity, risks and contribution to the challenge. Record feedback
  and changes rather than treating this draft as validated.
- These methods follow the [DOT framework](sources/dot-framework.md): standards
  review checks conventions, workshops check fit, and experiments check behaviour.

## Design learning outcome

- The primary purpose of this deliverable is the **Design**
  [learning outcome](sources/learning-outcomes.md): design part of a system,
  communicate its functional and technical requirements, and validate it early.
- The functional design explains the intended behaviour and trade-offs. The
  [standalone API specification](api/game-api-spec.md) communicates the interface
  to the app developer using HTTP conventions, payloads and concrete examples.
- The design approach moves from the requested scope to requirements F1–F6,
  create/retrieve flows, alternative choices, and an external contract. Examples
  include choosing UUIDs over sequential IDs, picture URLs over uploads, and a
  flexible definition over a fixed game schema. The choices section records
  the reasons and limitations so a reviewer can assess them.
- This is a backend interface design. App visual design is outside this artifact;
  do not claim that it demonstrates the aesthetic part of the wider outcome.

| Design requirement             | Design artifact                         | Early validation, before implementation                                                                                 |
| ------------------------------ | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| F1: create a game              | Create flow and POST contract           | App developer walks through a valid request and checks that the response contains everything needed to retain the game. |
| F2: retrieve by ID             | Retrieve flow and GET contract          | Walk through known, unknown and malformed IDs and sketch the app's response to each.                                    |
| F3: non-incremental ID         | UUIDv4 choice and response examples     | Reviewer checks that generation belongs to the server and that the app can retain and reuse the ID.                     |
| F4: game fields                | Payload tables and picture/time choices | App and hardware contributors compare representative game definitions with the contract and confirm field meanings.     |
| F5: invalid input              | Validation rules and problem responses  | Review invalid examples and check whether the app can explain each failure to a guardian.                               |
| F6: retrievable after creation | Persistence and success semantics       | Review the save/load sequence and failure cases, including a lost POST response.                                        |

- Use the JSON examples as a paper prototype of the interface. Ask the app
  developer to work through creating and loading a game without additional
  explanation, then record missing information and misunderstandings.
- Record each review with date, participant and role, scenario, expected result,
  feedback, decision, design revision and link to the revised artifact. Validate
  unresolved changes in a follow-up review. Runtime acceptance checks in the
  specification are later implementation work, not completed design validation.
- Current evidence: an AI-assisted functional design and external contract have
  been drafted. Local links and six JSON payload examples were checked for
  consistency/syntax. These document checks do not validate stakeholder fit.
- Daan's recorded contribution so far is defining the scope and game fields.
  His next contribution is to assess the alternatives, lead the design review,
  revise the contract and explain why the feedback changed or supported it.
- Stakeholder review, feedback, revisions and Daan's reflection are still missing.
  Do not claim the Design outcome has been achieved from this draft alone.

## Other outcome coverage

- This artifact targets Design. It is not intended to demonstrate all seven
  outcomes by itself; track the following gaps across the wider project.
- Analysis: source review and requirements provide supporting material; actual
  stakeholder research and triangulated findings remain to be gathered.
- Advice: the trade-offs are provisional input; substantiated recommendations
  and stakeholder responses remain to be recorded.
- Realization: implementation, code review and runtime test results are absent.
- Manage and control: deployment, monitoring, recovery and maintenance evidence
  are absent.
- Professional Standard: source use and explicit assumptions support the draft;
  Daan's methodological work and stakeholder communication need evidence.
- Personal Leadership: Daan's personal goals, actions and reflections are not
  supplied and must be recorded by him.

## Open questions

- Confirm the proposed picture URL and server creation-time interpretation.
- Which app-defined JSON structure represents a game and which settings are safe
  for the hardware? Who owns validation and future schema versions?
- Who will review the game-storage extension on behalf of the challenge?
- Which controlled environment, persistence approach and cleanup process should
  the eventual prototype use? Browser integration may also need a CORS policy.
- Confirm the assessment level for Professional Standard and Personal Leadership
  and any additional portfolio submission requirements.
