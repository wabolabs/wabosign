# Changelog

All notable changes to WaboSign are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.6.0] — 2026-08-03

Feature and upstream-sync release. Brings the fork from upstream DocuSeal 3.0.2 to
**3.1.7** (161 upstream commits), carrying the template documents editor, mobile-first
settings and filters, view-only parties, the new submission-level `completed_at`
model, and the paywall-free MCP/SMTP/SSO surfaces that resurfaced in the merge.

### Added
- **Template documents editor** — crop, redact, and replace document pages from the template builder, including touchscreen redact/crop, dynamic documents, and a Google Drive "replace" option.
- **Mobile-first settings** — settings navigation, filters, and modals now render as pages on small screens.
- **View-only parties** — recipients can be marked view-only and sent a read-only copy instead of a signing invitation.
- **HTML email editor** — compose invitation emails with rich HTML and edit the message per group of 10 parties.
- **Submission `completed_at` model** — completion state is now tracked on the submission itself (populated when the last non-viewer submitter completes) rather than derived on the fly from submitters.
- **"Completed at" signature timestamp** — new `with_signature_id_completed_at` account config stamps the signature ID with the submitter's completion time instead of attachment creation time.
- **SMTP reset action** — reset configured SMTP settings from `/settings/email`.
- **Shared-templates index** (`templates/shared`) and `GET /api/events/submission/:type` submission-events endpoint, both synced from upstream.

### Changed
- Gemfile.lock regenerated against the updated dependency set (Rails 8.1.3.1, sqlite3 2.9.5, nokogiri 1.19.4, oj 3.17.5) via a Ruby 4.0.5 container — the host toolchain can no longer load this Gemfile.
- Signing-form and template-builder JS refreshed from upstream (signature steps, area clamping, cmd+scroll zoom, redact/crop).
- All upstream Pro/Console/Plans gates that resurfaced in the merge (SMS, bulk-send, embedding, logo upload, reminders, SSO, user-role picker, AATL trusted-signature) were stripped back out, consistent with fork policy.

### Fixed
- [bin/sync-upstream](bin/sync-upstream) — `git fetch upstream --tags` failed once WaboSign's own 1.x release tags collided with DocuSeal's old tags of the same name; the fetch now targets just the tag being synced.
- [bin/rebrand-sync](bin/rebrand-sync) — preserved the literal `DocuSeal <lang> v` SDK user-agent regex in [lib/detect_browser_device.rb](lib/detect_browser_device.rb) (published `@docuseal/*` clients send it) and deny-listed [.githooks/pre-push](.githooks/pre-push), whose prose names the upstream repo it blocks.
- [lib/ability.rb](lib/ability.rb) — pre-existing `ArgumentError` in the Viewer-role grant: `TemplateConditions.collection(user, ability: :read)` was called with a keyword the method never accepted.
- [Dockerfile.test](Dockerfile.test) — added the `liblept5` system dependency upstream 3.1.7's `lib/leptonica.rb` requires at boot.
- CI — RSpec fixtures updated for the new `submission.completed_at` model (`api_missing_endpoints` documents/events endpoints, second-signer system spec); Brakeman ignore refreshed (swapped an obsolete entry for the weak-confidence false positive on the MCP instructions block).

### Notes
- Released image: `ghcr.io/wabolabs/wabosign:1.6.0` (also tagged `:latest`).
- Sync reference tag: `wabosign-synced-with-3.1.7`.

[1.6.0]: https://github.com/wabolabs/wabosign/releases/tag/1.6.0

## [1.5.0] — 2026-06-05

Fork-hardening and security release. Consolidates the upstream-merge guardrails
onto `master` and clears the outstanding Dependabot alerts. (Supersedes the
divergent, never-documented `1.4.0` sync line.)

### Security
- Bumped dependencies to clear 21 of 24 open Dependabot alerts (11 of 12 highs), including the only browser-shipped runtime package (`uuid` 9 → 11.1.1) and the Babel build chain (→ 7.29.x). The remaining 3 are unreachable or unfixable without breaking tooling (a transitive `vue` 2 inside a third-party widget, the glob CLI-only advisory, and eslint's ajv 6). Verified by compiling the container `webpack` stage. See [package.json](package.json).

### Added
- **Fork-invariants CI guard** ([bin/fork-check](bin/fork-check) + [config/fork_invariants.yml](config/fork_invariants.yml)) — fails the build if an upstream merge re-introduces a Pro gate, deletes fork code, overwrites a brand asset, or drops AGPL §7(b) attribution. The executable form of the post-merge checklist.
- **Brand-asset checksum baseline** and **`bin/sync-upstream` gates** to make upstream merges safe and repeatable.
- **Push guard against the fork parent** — [.githooks/pre-push](.githooks/pre-push) + [bin/install-push-guard](bin/install-push-guard) refuse pushes to `docusealco/*` (wired in via [bin/setup](bin/setup)), so an accidental `git push upstream` can't open a branch/PR on the upstream repo.

### Changed
- **Release images now publish to GHCR**, matching the documentation. [.github/workflows/docker.yml](.github/workflows/docker.yml) builds `linux/amd64` + `linux/arm64` and pushes the `MAJOR.MINOR.PATCH` tag plus `:latest` to `ghcr.io/wabolabs/wabosign` (previously Docker Hub, inherited un-rebranded from upstream). A fork invariant now keeps the registry on GHCR across upstream syncs.
- Made the DocuSeal fork relationship explicit in the UI and email attribution.

### Notes
- Released image: `ghcr.io/wabolabs/wabosign:1.5.0` (also tagged `:latest`).

[1.5.0]: https://github.com/wabolabs/wabosign/releases/tag/1.5.0

## [1.3.2] — 2026-05-20

CI green-up patch. No functional or security changes.

### Fixed
- [app/models/user.rb](app/models/user.rb) — `Style/RedundantRegexpEscape` (Rubocop): removed unnecessary `\-` escapes inside the `FULL_EMAIL_REGEXP` character classes (`[.'+\-]` → `[.'+-]`, `[.\-]` → `[.-]`). Semantics unchanged.
- [config/brakeman.ignore](config/brakeman.ignore) — added fingerprint for the `LinkToHref` XSS warning on `submissions_filters/_filter_modal.html.erb`: Brakeman tracks `params[:path]` taint through the `filter_path` conditional assignment introduced in 1.3.1; the `start_with?('/')` guard is the actual mitigation.
- [.github/workflows/ci.yml](.github/workflows/ci.yml) — replaced `docusealco/pdfium-binaries` (deleted repo, returns 404) with [`bblanchon/pdfium-binaries`](https://github.com/bblanchon/pdfium-binaries) as the pdfium binary source for the RSpec job. Same tarball layout (`lib/libpdfium.so`), no other changes.

### Notes
- Released image: `ghcr.io/wabolabs/wabosign:1.3.2` (also tagged `:latest`).

[1.3.2]: https://github.com/wabolabs/wabosign/releases/tag/1.3.2

## [1.3.1] — 2026-05-20

Security-focused patch addressing the alerts surfaced by the repo's first CodeQL scan (run against the 1.3.0 tag, commit [34250ac3](https://github.com/wabolabs/wabosign/commit/34250ac3)). No functional changes.

### Security
- [app/views/submissions_filters/_filter_modal.html.erb](app/views/submissions_filters/_filter_modal.html.erb) — reflected XSS (`rb/reflected-xss`): `params[:path]` flowed unsanitised into both the form `action` and the "remove filter" link `href`. Now constrained via a `filter_path` local that defaults to `/` unless the supplied value starts with `/`, blocking `javascript:` and absolute-URL payloads.
- [app/controllers/start_form_controller.rb](app/controllers/start_form_controller.rb) — column-name injection (`rb/sql-injection`, two sites): `find_by!` / `find_or_initialize_by` were keyed by `required_params.except('name')`, whose keys derive from the template-owner-controlled `link_form_fields` preference. Replaced with `required_params.slice('email', 'phone')` so only the columns actually permitted by `submitter_params` can reach the SQL builder.
- [app/models/user.rb](app/models/user.rb) — ReDoS (`rb/redos`): the local-part of `FULL_EMAIL_REGEXP` used a nested quantifier (`(?:(?:[a-z0-9_-]+[.+'])*[a-z0-9_-]+)*`) that backtracks exponentially on adversarial input. Rewritten as `[a-z0-9_]+(?:[.'+\-][a-z0-9_]+)*` — same accepted set, linear matching.
- [app/controllers/mcp_controller.rb](app/controllers/mcp_controller.rb) — polynomial ReDoS (`rb/polynomial-redos`): Bearer-token extraction used `\ABearer\s+(.+)\z`, which CodeQL flags as polynomial on long Authorization headers. Replaced with a `start_with?('Bearer ')` check plus a string slice.
- [app/javascript/submission_form/dropzone.vue](app/javascript/submission_form/dropzone.vue), [initials_step.vue](app/javascript/submission_form/initials_step.vue), [signature_step.vue](app/javascript/submission_form/signature_step.vue) — insecure randomness (`js/insecure-randomness`): attachment-correlation UUIDs were generated with `Math.random().toString()`. Swapped to `crypto.randomUUID()`. The IDs are UI-only, but the change matches the secure default and clears the alerts.
- [.github/workflows/ci.yml](.github/workflows/ci.yml) — missing-workflow-permissions (`actions/missing-workflow-permissions`, six jobs): added a single workflow-level `permissions: read-all` block. All six CI jobs are read-only (lint/test/scan); none publish artefacts or post statuses that need write access.

### Notes
- The following CodeQL alerts on the 1.3.0 commit are false positives in context and are not addressed by this release; they should be dismissed in the GitHub Security tab:
  - `rb/insecure-mass-assignment` on the five settings controllers (`user_configs`, `storage_settings`, `email_smtp_settings`, `account_configs`, `account_custom_fields`) — every call site uses `params.require(...).permit(...)` strong-parameters before `update!`.
  - `rb/csrf-protection-disabled` on `users/omniauth_callbacks_controller.rb` (OAuth provider callbacks legitimately can't carry a CSRF token) and `send_submission_email_controller.rb` (intentional public endpoint, rate-limited).
  - `rb/weak-sensitive-data-hashing` on `preview_document_page_controller.rb`, `config/dotenv.rb`, `lib/puma/plugin/redis_server.rb` — SHA-1 is used only as a non-cryptographic identifier (tempfile path, cache key) and is not protecting sensitive data.
  - `rb/clear-text-storage-sensitive-data` on `sso_settings_controller.rb` — the target column is on [`EncryptedConfig`](app/models/encrypted_config.rb), which declares `encrypts :value`, so the SSO `client_secret` is stored encrypted at rest.
- Released image: `ghcr.io/wabolabs/wabosign:1.3.1` (also tagged `:latest`).

[1.3.1]: https://github.com/wabolabs/wabosign/releases/tag/1.3.1

## [1.3.0] — 2026-05-19

Adds three new SMS providers alongside the existing BulkVS integration.

### Added
- [Twilio](lib/sms/providers/twilio.rb) — form-encoded POST to the Messages API; Basic Auth with `SID:Token`; treats a `201` response carrying an `error_code` as a failure.
- [VoIP.ms](lib/sms/providers/voipms.rb) — query-string-auth GET to `sendSMS`; treats `status != "success"` as a failure even on HTTP 200; enforces the API's 160-byte hard cap before dispatch.
- [SignalWire](lib/sms/providers/signalwire.rb) — Twilio-shaped client targeting the per-account Space URL host; strips `https://` and any trailing `/` from the user-supplied space URL.
- [/settings/sms](app/views/sms_settings/index.html.erb) — dynamic provider select driven by `Sms::SUPPORTED_PROVIDERS`, per-provider field blocks toggled by a nonce'd inline script (the app's CSP requires nonces on inline JS).
- [SMS.md](SMS.md) — per-provider "Configuring …" sections, wire-format quick-reference table, updated extension and status-code map sections.

### Changed
- [lib/sms.rb](lib/sms.rb) dispatches via per-provider classes and delegates the "is this configured" check to each provider — replaces the BulkVS-only hardcoded gate in `enabled_for?`.
- [app/controllers/sms_settings_controller.rb](app/controllers/sms_settings_controller.rb) extends the preserve-secret-on-blank-edit pattern (used for BulkVS) to all four providers' password/token fields via a `SECRET_KEYS` array.
- Existing BulkVS configs keep working unchanged — credentials remain in their existing keys; the `provider` key defaults to `bulkvs` when absent.

### Notes
- Released image: `ghcr.io/wabolabs/wabosign:1.3.0` (also tagged `:latest`).
- This release is a fast-follow on 1.2.0 — same upstream-sync state, plus the SMS providers.

[1.3.0]: https://github.com/wabolabs/wabosign/releases/tag/1.3.0

## [1.2.0] — 2026-05-19

Synced with upstream [DocuSeal 3.0.0](https://github.com/docusealco/docuseal/releases/tag/3.0.0) and added scripted-sweep tooling so future upstream merges are reproducible.

### Added
- [bin/rebrand-sync](bin/rebrand-sync) — idempotent Ruby script that performs the DocuSeal → WaboSign rename sweep across the working tree. Sentinel-protects AGPL §7(b) attribution phrases, the `<docuseal-form>` / `<docuseal-builder>` SDK custom elements, the `@docuseal/*` npm packages, and the `github.com/docusealco/{fields-detection,pdfium-binaries,turbo}` binary URLs. Pulls `PRODUCT_NAME` / `AATL_CERT_NAME` from [lib/wabosign.rb](lib/wabosign.rb) so a future brand change only touches one file.
- [bin/rebrand-check](bin/rebrand-check) — CI gate that fails on accidental DocuSeal survivors. Wired in as the new `Rebrand check` job in [.github/workflows/ci.yml](.github/workflows/ci.yml).
- "Sync workflow" section in [REBRANDING.md](REBRANDING.md) documenting the per-sync workflow.
- Upstream resend-emails feature: `app/controllers/submissions_resend_email_controller.rb` plus a new `resources :resend_email` route. English UI strings fall back to the key name until 14-language i18n is added.

### Changed
- Synced with upstream DocuSeal 3.0.0 (15 upstream commits, merge-base `528a1216`):
  - PDF image optimization, signing-form completion-button refactor.
  - Vue area-box clamping; percent format support; validation message improvements.
  - Defensive blank-check for `X-Wabosign-Signature` — caller-supplied signature headers are no longer overridden ([upstream a7891f89](https://github.com/docusealco/docuseal/commit/a7891f89)).
  - Belt-and-suspenders `authorize!(:update, @submitter)` on `submitters_send_email#create` ([upstream e52830c9](https://github.com/docusealco/docuseal/commit/e52830c9)).
- `git rerere` enabled (`rerere.enabled = true`, `rerere.autoupdate = true`) so semantic conflict resolutions are cached across syncs.
- [.gitattributes](.gitattributes) marks `Gemfile.lock` and `yarn.lock` as `-merge` (regenerate post-merge rather than diff).
- Webhook `User-Agent` continues to be `'WaboSign Webhook'` (upstream renamed theirs to `'WaboSign.com Webhook'`; the fork's name is preserved).
- `lib/docuseal.rb` upstream → `lib/wabosign.rb` rename is now performed by the script rather than by hand.

### Fixed
- [public/service-worker.js](public/service-worker.js) — the install/activate listeners now log `'WaboSign App installed/activated'` (latent rebrand survivor from 1.0.0).
- [.dockerignore](.dockerignore) and [.gitignore](.gitignore) — runtime data-dir entries now point at `/wabosign` instead of the stale `/docuseal`.

### Notes
- AGPL §7(b) "based on DocuSeal" attribution intact in [_powered_by](app/views/shared/_powered_by.html.erb), [_email_attribution](app/views/shared/_email_attribution.html.erb), [completed.vue](app/javascript/submission_form/completed.vue), [NOTICE](NOTICE), [LICENSE_ADDITIONAL_TERMS](LICENSE_ADDITIONAL_TERMS), and [README.md](README.md).
- Released image: `ghcr.io/wabolabs/wabosign:1.2.0` (also tagged `:latest`).
- Sync reference tag: `wabosign-synced-with-3.0.0` marks the merged tree as a known-good base for the next upstream pull.

[1.2.0]: https://github.com/wabolabs/wabosign/releases/tag/1.2.0

## [1.1.0] — 2026-05-18

### Added
- Per-account product-name branding. Account admins can replace "WaboSign" in the UI, emails, audit-trail PDFs, signing-form headers, page titles, PWA manifest, social-share `og:title`, and authenticator-app issuer with their own product name. Configurable from `/settings/personalization` above the logo upload. Leave blank to fall back to the default.

### Changed
- Resolution flows through a new `Wabosign.branded_product_name(account = nil)` helper. When no account is in scope (landing page, PWA manifest, OAuth chrome), the deployment's oldest non-archived account's brand is used.

[1.1.0]: https://github.com/wabolabs/wabosign/releases/tag/1.1.0

## [1.0.0] — 2026-05-17

First WaboSign release. Forked from [DocuSeal](https://github.com/docusealco/docuseal) 2.5.3.

### Added
- Google Workspace SSO via `omniauth-google-oauth2`, configurable from `/settings/sso` with ENV + DB fallback. See [GOOGLE_SSO.md](GOOGLE_SSO.md).
- SMS invitations via BulkVS, configurable from `/settings/sms`. See [SMS.md](SMS.md).
- Custom account logo upload with server-side SVG sanitization. The logo renders on the sign-in page, signing flow, dashboard navbar, share-link QR page, and audit-trail PDFs.
- Editor and Viewer user roles alongside Admin. Editors get CRUD on templates and submissions; Viewers get read-only access. Self-service profile management is preserved for every role.
- OCI image labels (`org.opencontainers.image.*`) and multi-arch (linux/amd64 + linux/arm64) Docker builds wired via `.github/workflows/docker.yml`.
- [CHANGELOG.md](CHANGELOG.md) and a Releases section in [README.md](README.md).

### Changed
- Removed the upstream "Pro" feature paywall — multi-account, SSO, SMS, audit trail, and timestamping all work out of the box on a self-hosted deployment.
- Rebranded all UI surfaces, emails, and asset paths from DocuSeal to WaboSign while preserving AGPL §7(b) upstream attribution in [NOTICE](NOTICE), [REBRANDING.md](REBRANDING.md), [LICENSE_ADDITIONAL_TERMS](LICENSE_ADDITIONAL_TERMS), and the in-app "Powered by" footer.
- Default container image is now `ghcr.io/wabolabs/wabosign` (public).
- Security contact in [SECURITY.md](SECURITY.md) now routes to `wabosign@wabo.cc`.

### Removed
- Developer Newsletter step from the initial-setup flow (was a DocuSeal mailing-list signup).
- Console-redirect endpoints (`/upgrade`, `/manage`, `/console_redirect`) and the enquiries form — only made sense for DocuSeal's hosted multitenant SaaS.
- Upstream API-docs language stubs at `docs/api/` (10 files referencing `api.docuseal.com`). The OpenAPI spec at `docs/openapi.json` and the embedding/webhook guides remain (URLs rewritten to `sign.wabo.cc`).
- The "Upgrade to Pro" fallback markup served by the embed-script controller — replaced with a neutral "embed assets not loaded" message.

### Security
- Account-logo SVG uploads are sanitized via Nokogiri before storage (strips `<script>`, `<foreignObject>`, `on*` attributes, and external `href` / `xlink:href` values).

[1.0.0]: https://github.com/wabolabs/wabosign/releases/tag/1.0.0
