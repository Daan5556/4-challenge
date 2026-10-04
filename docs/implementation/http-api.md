# HTTP API prototype

## Scope and choices

- I implemented the four operations in the Game HTTP API specification using Go,
  PostgreSQL for games and Alarik for object storage. This implements the proposed
  server extension; stakeholder approval and hardware integration remain open.
- I use the standard HTTP server, one database table and an S3 client. The MinIO
  Go client speaks S3 to Alarik; no MinIO server is used.
- I preserve JSON numbers using raw JSON and reject duplicate keys before storage.
- Uploaded objects use UUID keys and conditional writes. The dedicated demo bucket
  allows public reads. The API applies its read policy at startup.
- The Compose services bind to loopback and persist data in named volumes. Local
  credentials are demo defaults. The Alarik image is pinned to the tested digest.
- Alarik configuration follows its official
  [installation documentation](https://alarik.io/docs/installation).

## Run locally

- Requirements: Go 1.27.1 or later, Docker Compose and FFmpeg. Tests also need Bash,
  curl and jq.
- From the repository root, start storage, then start the API:

```bash
docker compose -p 4-challenge -f server/compose.yaml up -d
cd server
go run ./src
```

- Wait for PostgreSQL and Alarik to initialize before starting the API. Startup
  fails clearly if either dependency is unavailable; retry after it is ready.
- The API listens on localhost:8080. Alarik serves objects on localhost:9000.
- Configuration variables: DATABASE_URL, LISTEN_ADDR, S3_ENDPOINT, S3_ACCESS_KEY,
  S3_SECRET_KEY, S3_REGION, S3_BUCKET and S3_PUBLIC_URL. Defaults match Compose.
- Set matching application credentials if overriding Compose defaults. DB_PASSWORD
  changes the initial database password only when creating a new database volume.
- S3_PUBLIC_URL includes the bucket path. Object and image URLs accept HTTP or
  HTTPS without an environment flag. This updates the original HTTPS-only contract;
  hostname, credentials, whitespace and length validation remain in place.
- Stop services with `docker compose -p 4-challenge -f server/compose.yaml stop`.
  Removing volumes deletes stored games and uploads; ordinary restarts retain them.

## Demo requests

- Each script in requests contains one curl command and prints response headers
  and body. Upload samples are included in requests/files.
- Run these commands from the repository root while the API is running:

```bash
bash requests/create-game.sh
bash requests/get-game.sh <returned-game-id>
bash requests/upload-image.sh
bash requests/upload-audio.sh
```

- Replace `<returned-game-id>` with the ID from game creation. Set BASE_URL to
  target another API address. Upload scripts resolve samples relative to themselves.

## Validation

- My approach combines DOT library research of the API contract and Alarik setup
  with lab checks against real PostgreSQL and Alarik services.
- On 27 September 2026, tests/api.sh passed 47 HTTP checks plus byte comparisons
  for all four supported media types. Checks cover create/retrieve, validation,
  error formats, size boundaries, distinct uploads and saved media references.
- The Go build, go vet, Bash syntax checks and Compose configuration validation
  passed. All four demo requests returned their expected success status.
- tests/persistence.sh confirmed that a game and uploaded image survived API,
  database and object storage restarts.
- tests/unavailable.sh confirmed that stopped database and object storage services
  cause writes to return 503 without reporting success.
- Run the checks from the repository root:

```bash
EVIDENCE_DIR=/tmp/visioball-evidence tests/api.sh
# Restart the API and storage, then:
EVIDENCE_DIR=/tmp/visioball-evidence tests/persistence.sh
# With the API running, stop only the named dependency before each outage check:
tests/unavailable.sh db
tests/unavailable.sh alarik
```

- Tests create persistent demo data. The outage script does not stop or restart
  services itself. Restore dependencies after each outage check.

## Limits and evidence gaps

- Decoder defaults are prototype choices pending stakeholder agreement: images
  up to 16 megapixels, audio decoding up to 10 seconds of processing, a 64 MiB
  FFmpeg allocation limit and at most two simultaneous uploads. Audio is decoded
  fully; no resizing or transcoding is stored.
- This controlled demo has no authentication, listing, editing, deletion, CORS
  policy or CDN deployment. Uploaded files remain after a rejected game request.
- Exact upload byte limits are tested with corrupt bodies; acceptance of valid
  media at each maximum size, concurrency/load behaviour, backup recovery and
  stakeholder review remain unvalidated.
- Realization evidence: my implementation in server/src and Bash/curl checks in
  tests connect the design to observed runtime behaviour.
- Manage and control evidence: my persistent Compose setup, restart checks and
  outage checks support local operation; production monitoring and recovery need work.
- Analysis and Advice: my source review and implementation choices provide input;
  stakeholder research and substantiated deployment recommendations remain open.
- Design: the API contract now has a working prototype; stakeholder feedback and
  any resulting contract revisions still need recording.
- Professional Standard: method and validation records support the work; team
  review and the applicable assessment level remain open.
- Personal Leadership: personal goals, reflection and the applicable assessment
  level remain to be recorded. These artifacts do not establish outcome achievement.
