# Advice on application deployment

**Author:** Daan Eggen  
**Date:** 19/09/2026  
**Version:** 0.1 — proposed advice **Version:** 1.0

---

## Recommendation

- Place a managed content delivery network (CDN) with reverse proxy and DDoS
  protection in front of the public Visioball platform. Restrict direct access
  to the origin server, which runs the backend and serves the original content.
- Cache explicitly public, reusable assets at the edge. Forward API requests to
  the backend without caching for the first prototype.
- Cloudflare is a suitable candidate to evaluate for this role. The provider,
  plan, hosting environment and backend stack remain unselected.
- This advice is based on project documents and technical documentation.
  Performance gains, cost savings and stakeholder acceptance are not yet
  validated. It recommends an architecture, not a completed deployment.

## Project context

- The challenge requires an app, safe communication with the ball, sound and
  settings control, and advice on further development.
- The server platform is a proposed extension. Reliable delivery of shared
  content could support users using the app, once that scope is confirmed. A CDN
  does not handle the local connection between the app and ball.
- The analysis identifies performance, availability, security, scalability, cost
  and maintenance as comparison criteria.
- The game API design provides game creation and retrieval plus audio and image
  uploads. Uploads return object URLs for use in games. A CDN can deliver these
  objects when their delivery host is configured for it; placing only the API
  domain behind a CDN does not cache files on other hosts.

## Research basis

- Library research: review the project analysis, API design and official CDN
  documentation to compare a directly reachable origin with a protected origin.
  This follows the DOT framework.
- Edge caching can serve reusable content near users and reduce requests and
  transferred bytes at the origin. These are documented capabilities, not
  measured project results. See
  [Cloudflare Cache](https://developers.cloudflare.com/cache/).
- Cloudflare documents automatic DDoS detection and mitigation. This supports
  evaluating it as a protective entry point for public HTTP traffic. Caching
  alone is not DDoS protection; the selected service must provide both. See
  [DDoS protection](https://developers.cloudflare.com/ddos-protection/).
- Cloudflare does not cache JSON by default. Its caching behaviour depends on
  request characteristics, response headers and configured rules. Therefore, the
  current API does not establish a strong caching benefit by itself. See
  [default cache behaviour](https://developers.cloudflare.com/cache/concepts/default-cache-behavior/).

## Alternatives and trade-offs

| Criterion    | Direct public origin                                                                                    | CDN with a restricted origin                                                                                                        |
| ------------ | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Performance  | Every request reaches the origin. Simple routing may suit a small local audience.                       | Cache hits can reduce delivery time; misses and uncached API requests still reach the origin. Improvement requires measurement.     |
| Security     | Relies on hosting protections and server configuration. Existing host DDoS protection may already help. | Adds upstream traffic filtering, provided requests cannot bypass it. Application security remains necessary.                        |
| Scalability  | The origin handles all requests and asset transfers.                                                    | Cached assets reduce origin work. Database writes and uncached reads still need sufficient backend capacity.                        |
| Availability | Depends on the origin and hosting network.                                                              | May keep eligible cached assets available during some failures. It does not keep the API or database working when the origin fails. |
| Cost         | Fewer services, but all delivery uses origin capacity and bandwidth.                                    | May reduce origin transfer costs, but introduces possible subscription, request or delivery charges. Savings are unproven.          |
| Maintenance  | Fewer components to configure and troubleshoot.                                                         | Adds cache rules, invalidation, proxy configuration and provider dependency.                                                        |

- A direct origin remains reasonable for a controlled development environment.
  For public deployment, the proposed protective layer offers a useful boundary
  before traffic reaches the project's backend.
- Compare total hosting, bandwidth and CDN costs using expected traffic and the
  current provider terms before choosing a plan. No budget or traffic estimate
  has been confirmed, so this document does not claim a cheapest option.

## Proposed setup

- Route public app requests through the CDN to the origin using HTTPS, with
  certificate validation on the origin connection.
- Make the origin reachable through a private tunnel, or restrict incoming web
  traffic to the proxy and authenticate its origin connections. Select the
  mechanism after the hosting environment is known. Hiding an IP address in DNS
  is insufficient. See
  [origin protection](https://developers.cloudflare.com/fundamentals/security/protect-your-origin-server/).
- Keep `POST /games`, `GET /games/{id}`, `POST /audio` and `POST /image` outside
  shared caching initially. Configure an explicit API cache bypass and return
  `Cache-Control: no-store`. A random game ID does not establish that its
  content is public.
- For uploaded audio and images approved for public delivery, define cache paths
  and expiry periods. Use versioned filenames for changed assets and document
  how cached content is removed. Avoid a blanket rule that caches all responses.
- Keep private and user-specific content outside shared caching. Review headers
  and rules together, because cache rules can override origin behaviour. See
  [default cache behaviour](https://developers.cloudflare.com/cache/concepts/default-cache-behavior/).
- Retain backend validation, access controls, patching and backups. Evaluate
  request limits and filtering against legitimate app traffic; browser challenge
  pages can interrupt an API client.
- Record who manages DNS, certificates, cache rules and incidents. Consider the
  provider's access to proxied traffic and logs when agreeing data handling.
- Monitor origin errors, API latency, cache hit ratio and origin traffic.
  Document recovery steps without automatically reopening public origin access.

## Validation and next steps

- Field research, planned: confirm expected users, asset types, privacy needs,
  budget and acceptable response times with stakeholders. This determines
  whether the proposed benefits address actual needs.
- Lab research, planned: compare direct and proxied delivery in a controlled
  environment using identical assets, request volumes and client locations.
  Separate cold-cache, warm-cache and uncached API measurements. Record response
  times, errors, cache status, origin requests and transferred bytes.
- Check that normal app requests succeed through the proxy, direct external
  origin access fails, and API or private responses are never shared from cache.
  Verify asset updates and removal if asset hosting is included.
- Test origin unavailability and record which operations fail. Use configuration
  review to assess DDoS coverage; ordinary load tests do not prove resistance to
  a distributed attack.
- Current validation: project scope and provider documentation were reviewed. No
  deployment, benchmark, security test or stakeholder review has taken place.

## Portfolio evidence

- Primary outcome: **Advice**, as described in the learning outcomes. This
  document connects analysis questions to a recommendation and explains its
  quality and cost trade-offs for a project decision.
- Result: a sourced recommendation, comparison of alternatives and proposed
  validation activities. It is provisional evidence, not a claim that the
  learning outcome has been achieved.
- Add supporting experiment results, reviewer feedback, the final decision and
  my reflection on how the evidence affected my recommendation.
- Analysis is supported by the source comparison; Design can use the proposed
  setup. Realization and Manage and control still need implementation, tests and
  operational records. Professional Standard needs stakeholder communication
  evidence; Personal Leadership needs my own goals and reflection.
- The assessment level for Professional Standard and Personal Leadership remains
  to be confirmed. This document does not replace wider portfolio evidence for
  those outcomes.

## Open questions

- Which uploaded assets may be delivered publicly, and what retention and cache
  expiry policies should apply?
- What protection is already included in the eventual hosting environment?
- Which provider and plan meet the agreed budget, privacy and operational needs?
- Who will review and approve the recommendation on behalf of the project?
