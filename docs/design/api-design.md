# Game API functional design

**Author:** Daan Eggen  
**Date:** 20/09/2026  
**Version:** 0.2 — proposed prototype design

## Purpose and status

- Provide a small HTTP API through which the front-end app can save a game
  definition and retrieve it using its identifier, and upload audio and image
  files.
- My confirmed scope contains four operations: `POST /games`, `GET /games/{id}`,
  `POST /audio` and `POST /image`. A game has a non-incremental UUID, title,
  optional image URL, timestamp and JSON definition.
- Uploads return URLs that the client can submit when creating a game. The Game
  HTTP API specification defines the standalone technical contract.
- The remaining choices below are proposed defaults, not stakeholder-approved
  requirements or implemented behaviour.
- This deliverable contains design only. No API, database or deployment has been
  implemented, and no runtime behaviour has been tested.

## Connection to the challenge

- The source document Smart Audio Control for the Visiobal asks for an app,
  communication with the ball, sound/settings control and supporting design and
  advice.
- Storing reusable game definitions could support the app's selection of
  activities and settings. This is a proposed extension, not a requirement in
  the original assignment. Its value needs confirmation with the app team and
  challenge stakeholders.
- The API stores information. The app and hardware team remain responsible for
  interpreting that information and communicating safely with the ball.
- A game here means a saved definition, not a played session, score or live
  match. The timestamp records creation of that definition, not when a game is
  played.
- The front-end app direction follows Analysis. Its relationship to the
  assignment's accessibility deliverables still needs stakeholder confirmation.

## Functional requirements

| ID  | Requirement                                            | Basis                                     | Acceptance criterion                                                                                                   |
| --- | ------------------------------------------------------ | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| F1  | Create a game through `POST /games`.                   | User request                              | A valid request creates one game and returns its complete representation.                                              |
| F2  | Retrieve a game through `GET /games/{id}`.             | User request                              | An existing ID returns the stored game; an unknown valid ID returns 404.                                               |
| F3  | Use non-incremental IDs.                               | My advice; UUIDv4 proposed                | Each creation receives a server-generated UUIDv4, never a sequential number.                                           |
| F4  | Store title, optional image, timestamp and definition. | User request; field semantics proposed    | Creation and retrieval follow the Game HTTP API specification.                                                         |
| F5  | Reject invalid input without creating a game.          | Proposed                                  | Validation errors identify the relevant field and leave storage unchanged.                                             |
| F6  | Make a successful creation immediately retrievable.    | Proposed                                  | After 201, a GET using the returned ID succeeds while the service is available.                                        |
| F7  | Upload audio through `POST /audio`.                    | Confirmed scope                           | A valid file is stored as an object and its usable URL is returned.                                                    |
| F8  | Upload an image through `POST /image`.                 | Confirmed scope                           | A valid file is stored as an object and its usable URL is returned.                                                    |
| F9  | Reference uploaded files when creating a game.         | Confirmed scope; audio placement proposed | The client submits the image URL in `imageUrl` and audio URLs inside `definition`; retrieval preserves the references. |

## User flows

- Upload: the app sends one image file to `POST /image` and each audio file to
  `POST /audio`. Each successful upload returns `url`, `contentType` and
  `sizeBytes`. The app retains successful URLs if another upload fails.
- Create: after the required uploads succeed, the app sends the title, optional
  `imageUrl` and `definition` to `POST /games`. Audio URLs go inside
  `definition`; `definition.audioUrls` is the example convention in the
  technical specification. The server assigns the ID and creation time, stores
  the game and returns it. The app retains the ID for later retrieval.
- Retrieve: the app sends a known ID and displays the returned title, image and
  settings. A missing image or failed image download uses an app fallback and
  does not prevent retrieval of the game definition.
- Failure: the app keeps unsaved input on validation or connection failure and
  displays the error. An uncertain POST outcome requires care: a timeout may
  occur after the server has already saved the game.
- There is no list/search operation. If the app loses the ID, it cannot recover
  the game through this API. There are no edit, delete, account, sharing,
  operations in this prototype.

## Choices and trade-offs

- I use HTTP with JSON for game data and raw binary bodies for file uploads. Use
  HTTPS for a hosted service; plain HTTP is acceptable for local development.
- Generate UUIDv4 IDs on the server using secure randomness. This satisfies the
  non-incremental requirement without exposing creation order. UUIDs are longer
  to copy than integers and random IDs have poorer index locality than ordered
  IDs. Enforce uniqueness and retry generation on collision; never overwrite an
  existing game. See [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html).
- Use `createdAt`, assigned by the server in UTC, to avoid relying on the app's
  clock. A scheduled or played-at timestamp would be a separate future concept.
- I separate file uploads from game creation and store URL references in games.
  This supports reuse and avoids embedding large files in JSON, but uploads and
  game creation do not form one transaction. Unused files can remain after
  failure.
- I propose one raw binary file per upload request, with its media type in
  `Content-Type`. This avoids multipart metadata and base64 expansion for a
  single file. Multipart would be useful if additional file metadata becomes
  necessary.
- I propose PNG/JPEG images up to 5 MiB and MP3/WAV audio up to 20 MiB, matching
  the Game HTTP API specification. File signatures and parseability must match
  the declared type. Empty, corrupt or mismatched files are rejected. No image
  resizing or audio transcoding is included; decoder resource limits remain
  open.
- Uploads return a stable HTTPS URL after persistence succeeds. Each object has
  a server-generated UUIDv4 key and is never overwritten by another upload.
- I keep `imageUrl` optional and audio references inside the flexible
  definition. Game creation checks image URL syntax without fetching the image
  or checking ownership. External HTTPS image URLs remain accepted. Audio URL
  fields inside the definition are not separately validated by the API.
- Treat `definition` as an opaque JSON object. This lets the app team experiment
  before agreeing on game rules. The server validates its shape and size, but
  cannot guarantee that its contents describe a playable or safe game. The app
  must validate settings before applying them to the ball.
- Games are immutable through this API. Saving a revision creates a new game and
  ID. This simplifies the prototype but leaves old definitions in storage.
- Use a maximum 64 KiB game-create request body, title length of 1–120 Unicode
  code points, image URL length of 2,048 characters and JSON nesting limit of 20
  containers. These are provisional prototype limits to bound input, not
  measured capacity requirements. Confirm them using representative definitions.
- Reject unknown outer fields to catch integration mistakes while allowing
  arbitrary fields inside `definition`.
- Start without application authentication only in a controlled demonstration
  using non-sensitive sample data. Anyone who can reach it can upload files,
  create games and retrieve a game whose ID they know. UUIDs do not provide
  authorization. Public release requires a separate access and abuse-control
  decision.

## Infrastructure and CDN

- Keep framework, database, hosting and provider selection open. The storage
  must retain successful creations across application restarts; its technology
  is outside this functional design.
- Object storage holds uploaded audio and images separately from game records.
  Both must survive application restarts. The returned URLs must serve the
  uploaded bytes immediately, with their validated media types.
- A CDN could deliver those objects. File delivery belongs to object storage or
  the CDN; it does not add another application API operation. API responses use
  `no-store`, while object cache lifetimes remain a deployment decision.
- I propose stable, non-expiring read URLs for the controlled prototype so game
  references remain usable. Anyone with a URL can read the file. Private files
  would need a different access design; expired signed URLs cannot simply remain
  embedded in game definitions without a renewal mechanism.
- Cloudflare is a candidate alongside other providers; no service is selected.
  Later research should compare origin-only and CDN audio and image delivery
  using the same assets, recording latency, availability, transfer volume and
  cost.

## Risks and responses

| Risk                                      | Prototype response and remaining limitation                                                                                                                                                                           |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Duplicate games after a POST retry        | Each accepted POST creates a new ID, even for identical content. The app must not silently retry an uncertain submission. Idempotency is a future extension.                                                          |
| Unauthorized access or excessive creation | Restrict the demo environment and use sample data. Before public use, decide authentication, ownership, rate limits and storage quotas.                                                                               |
| Invalid or dangerous game settings        | Store JSON as data; do not execute it. The app/hardware integration must enforce allowed settings before use. A shared definition schema remains open.                                                                |
| Invalid or unsafe uploads                 | Validate media type, file signature, size and parseability. Rejected uploads must not publish an object. These checks do not establish content suitability; public use needs a separate scanning/moderation decision. |
| Unavailable media                         | The app uses an image fallback and reports unavailable required audio before starting a game that depends on it. Game retrieval does not fetch referenced files.                                                      |
| Unused files and storage costs            | Successful uploads remain if game creation fails. Retain URLs for retry and agree on manual demo cleanup; there is no delete endpoint or automatic orphan cleanup.                                                    |
| Duplicate uploads after retries           | Each successful upload creates a new object and URL. A lost response may follow a completed upload; retrying can create duplicates. No deduplication or idempotency mechanism is included.                            |
| Lost IDs and accumulating data            | The app retains returned IDs. No in-API recovery or deletion exists; agree on demo cleanup and retention before collecting real content.                                                                              |
| Data loss or service failure              | Return 201 only after persistence succeeds; failed writes return an error. Backup, recovery and monitoring arrangements remain implementation work.                                                                   |
| Incompatible future definitions           | Agree on a versioned game-definition format with the app team before multiple clients depend on it. No compatibility claim is made for arbitrary JSON.                                                                |

## Validation and research

- Library method, completed for this draft: review the challenge, project
  analysis, learning outcomes and linked HTTP, UUID and error-format standards.
  This separates assignment requirements from proposed interface conventions; it
  does not establish user needs.
- Workshop method, planned: walk through upload, save/load and lost-ID flows
  with the app and hardware contributors. Check the meaning of a game, image
  handling, timestamp and safe interpretation of settings; record decisions and
  revisions.
- Lab method, planned: implement the four operations and run the contract checks
  in the Game HTTP API specification. Record actual requests, responses and
  storage observations, including restart and failure cases.
- Showroom method, planned: ask a technical reviewer and challenge stakeholder
  to assess simplicity, risks and contribution to the challenge. Record feedback
  and changes rather than treating this draft as validated.
- These methods follow The DOT Framework: standards review checks conventions,
  workshops check fit, and experiments check behaviour.

## Design learning outcome

- The primary purpose of this deliverable is the **Design** outcome described in
  Learning outcomes: design part of a system, communicate its functional and
  technical requirements, and validate it early.
- The functional design explains the intended behaviour and trade-offs. The Game
  HTTP API specification communicates the interface to the app developer using
  HTTP conventions, payloads and concrete examples.
- The design approach moves from the requested scope to requirements F1–F9,
  create/retrieve flows, alternative choices, and an external contract. Examples
  include choosing UUIDs over sequential IDs, separate uploads with URL
  references over embedded file data, and a flexible definition over a fixed
  game schema. The choices section records the reasons and limitations so a
  reviewer can assess them.
- This is a backend interface design. App visual design is outside this
  artifact; do not claim that it demonstrates the aesthetic part of the wider
  outcome.

| Design requirement                | Design artifact                                      | Early validation, before implementation                                                                                 |
| --------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| F1: create a game                 | Create flow and POST contract                        | App developer walks through a valid request and checks that the response contains everything needed to retain the game. |
| F2: retrieve by ID                | Retrieve flow and GET contract                       | Walk through known, unknown and malformed IDs and sketch the app's response to each.                                    |
| F3: non-incremental ID            | UUIDv4 choice and response examples                  | Reviewer checks that generation belongs to the server and that the app can retain and reuse the ID.                     |
| F4: game fields                   | Payload tables and image/time choices                | App and hardware contributors compare representative game definitions with the contract and confirm field meanings.     |
| F5: invalid input                 | Validation rules and problem responses               | Review invalid examples and check whether the app can explain each failure to a user.                                   |
| F6: retrievable after creation    | Persistence and success semantics                    | Review the save/load sequence and failure cases, including a lost POST response.                                        |
| F7–F9: upload and reference files | Upload contracts and URL references in game payloads | Walk through image/audio uploads followed by game creation; review partial failure, invalid files and retained URLs.    |

- Use the JSON examples as a paper prototype of the interface. Ask the app
  developer to work through uploading files, creating and loading a game without
  additional explanation, then record missing information and misunderstandings.
- Record each review with date, participant and role, scenario, expected result,
  feedback, decision, design revision and reference to the revised artifact.
  Validate unresolved changes in a follow-up review. Runtime acceptance checks
  in the specification are later implementation work, not completed design
  validation.
- Current evidence: this functional design and the Game HTTP API specification
  describe four operations and the upload-to-game flow. A document consistency
  check covers endpoints, URL placement, upload formats and limits. Stakeholder
  fit and runtime behaviour remain unvalidated.
- My recorded contribution so far is defining the scope, game fields and
  separate upload operations. My next contribution is to assess the
  alternatives, lead the design review, revise the contract and explain why the
  feedback changed or supported it.
- Stakeholder review, feedback, revisions and my reflection are still missing.
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
  my methodological work and stakeholder communication need evidence.
- Personal Leadership: I still need to record my personal goals, actions and
  reflections.

## Open questions

- Confirm the proposed creation-time meaning, upload formats and size limits,
  decoder resource limits, and audio URL placement inside the definition.
- Which app-defined JSON structure represents a game and which settings are safe
  for the hardware? Who owns validation and future schema versions?
- Who will review the game-storage extension on behalf of the challenge?
- Which controlled environment, persistence approach and cleanup process should
  the eventual prototype use? Browser integration may also need a CORS policy.
- Confirm the assessment level for Professional Standard and Personal Leadership
  and any additional portfolio submission requirements.
