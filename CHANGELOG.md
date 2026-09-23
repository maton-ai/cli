# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.3] - 2026-09-23

### Added

- Agent detection now falls back to the process ancestry when no known environment variable is set, walking up to 16 parents and matching their executable names (or, for `node`, `bun`, `deno`, `python`, and `ruby`, the script they run) against a fixed list of agent CLIs. Only names on that list are ever reported, so nothing about the user's other processes leaves the machine. Before the ancestry walk, marker files are checked too, starting with Devin's `/opt/.devin`.
- Newly detected agents: Aider, Augment (Auggie), Bolt, Claude Cowork, Devin, Droid, GitHub Copilot agent mode in VS Code, Jules, Octofriend, Pi, Replit, Warp's Oz agent, Windsurf, and Zed. Existing detections for Amp, Antigravity, Copilot CLI, OpenCode, Cursor, and Claude Code recognise more of the variables those tools set.
- `X-Maton-Client-User-Agent` now carries `is_tty`, `ci`, and `terminal`. `is_tty` is true only when both stdin and stdout are terminals; `ci` names the CI provider (`github-actions`, `gitlab`, `circleci`, and so on), `unknown` for CI detected only through generic variables, and is omitted outside CI; `terminal` names the terminal emulator or editor from `TERM_PROGRAM`, with Cursor and Windsurf told apart from VS Code by their own variables.

### Changed

- `token` help no longer suggests passing its output to `curl`, and points at `maton api` instead, which attaches the credential itself so the token never lands in shell history, process listings, or logs.
- The agent is detected once per invocation and shared between telemetry and the client user agent, since the fallback now walks the process tree.

## [0.3.2] - 2026-09-18

### Added

- `login --device`: sign in with the device authorization grant (RFC 8628) on a host with no usable browser, such as a container, an SSH session, or a CI runner. The CLI prints a one-time code and sends you to `https://accounts.maton.ai/device?user_code=<code>`, which signs you in before handing off to the approval page, so the code can be approved from any other device while the CLI polls the token endpoint. It yields the same renewable OAuth token as `--oauth`, so `whoami`, automatic refresh, and `logout` revocation all behave identically.
- `login --oauth` now falls back to the one-time code flow by itself when no browser can be opened, instead of hanging until its timeout on a callback that this host will never receive. A browser is considered unavailable when `CI` is set, when the session is over SSH with no forwarded display, when a Linux or BSD host has neither `DISPLAY` nor `WAYLAND_DISPLAY`, or when no opener is on `PATH`; `BROWSER` names one explicitly and skips those checks. If the detector is wrong and the launch itself fails, the CLI switches to the one-time code at that point rather than waiting on the callback. The timeout message now also points at `--device`.
- `login --interactive` now composes with `--device`, so a host that has an opener on `PATH` but no browser worth opening can still take the one-time code without a launch attempt. `--oauth` and `--device` are rejected together instead of silently resolving to `--device`.
- `login --api-key` runs the paste-an-API-key flow.

### Changed

- **Breaking:** The CLI no longer displays any part of an API key. `whoami` drops the `api_key` field from `--json` and the `API key:` line from its text output, and `login --api-key` no longer echoes the pasted key back. Scripts that read `api_key` to tell how a profile is authenticated should read `auth_type` instead, which is unchanged. The `redacted_api_key` field is also gone from `config.toml`; existing entries are dropped the next time anything writes the file, and no migration is needed.
- `whoami` now always reports how the profile is authenticated, as `Signed in with: API key` or `Signed in with: OAuth`. Previously the line appeared only for OAuth profiles, with the `API key:` fingerprint implicitly marking the others. Relatedly, `auth_type` now reports `api_key` when `MATON_API_KEY` is set, since that variable overrides the stored credential without moving the active profile pointer; it previously reported the profile's own type while a different key was in use.
- **Breaking:** `maton login` now signs in through the browser and stores a renewable OAuth token, which is what `maton login --oauth` used to do; `--oauth` still names that flow explicitly. The old default, pasting a long-lived API key, moved to `maton login --api-key`. Existing API key profiles keep working and `MATON_API_KEY` is unaffected; only what a bare `maton login` does has changed.

## [0.3.1] - 2026-09-06

### Added

- `X-Maton-Client-User-Agent` now carries `command`, `flags`, `install_method`, and `credential_source`, and `platform` gained the OS version. `command` is the full command path and `flags` the sorted names of the flags actually passed — names only, never values, so nothing a user typed is sent. `install_method` is inferred from the resolved executable path (`npm`, `homebrew`, `script`, `go`, `package`), with `MATON_MANAGED_BY_NPM`/`MATON_MANAGED_BY_BUN` taking precedence; it is empty rather than guessed when the path matches nothing known. `credential_source` says where the token came from and is resolved per request, so it reports `env` for any `MATON_*` variable instead of naming the variable.
- OpenClaw is detected as the invoking agent via `OPENCLAW_SHELL`, which it sets on every command spawned by its exec tool. Skills installed from ClawHub run under OpenClaw, so this also identifies ClawHub-distributed traffic.

### Changed

- The client user agent is now built from the factory and encoded per request rather than once at client construction, which is what lets `credential_source` reflect the credential a given request actually used.
- Go toolchain pinned to 1.27.1.

## [0.3.0] - 2026-08-31

### Added

- `function`: manage serverless functions — `list`, `search`, `get`, `deploy`, `create`, `update`, `delete`, plus `code {get,download}` for a version's source bundle, `version {list,get}`, `env {list,create,update,delete}`, and `run {list,get}` with `run log {list,tail}` for a run's log events. Apart from `deploy`, one command per API operation; path parameters are flags, so sub-resources take `--function <id>`.
- `function deploy` is the command to reach for when shipping code: `cd my-fn && maton function deploy` creates the function on its first run and publishes a new version on every run after that. It records the function ID in `.maton/state.json` inside the deployed directory, so no ID is ever typed and no name is ever matched against the server — a name is only a URL-slug seed, and renaming reallocates the URL, so a name-keyed upsert could silently retarget the wrong function. The target resolves as `--function` > `MATON_FUNCTION_ID` > the link file > create a new function. The link file is dot-prefixed, so the bundler's existing dot-skip rule keeps it out of every upload; it is per-account, and deploy prints a reminder to gitignore it rather than editing `.gitignore` itself. `--name` and `--runtime` are refused against a linked function, so deploy can never move a live URL or attempt an immutable runtime change.
- `function create --dir <dir>` bundles a local source tree, inferring the handler and runtime (`main.py` → `main.handler` on `python3.12`, `index.js` → `index.handler` on `nodejs22.x`). A handler is written `<module>.<symbol>` — `main.handler` is `handler` in `main.py` — and the module carries no extension because the runtime supplies it, so `--handler` is not guessable from a file name and is checked locally before upload. Dot-prefixed paths, a default ignore list (`node_modules`, `__pycache__`, `venv`, `env`, `site-packages`, `dist`, `build`, `target`, `vendor`), symlinks, and non-text files are skipped; `--ignore`, `--no-default-ignore`, and `--dry-run` control and preview what is uploaded. `deploy` shares all of it.
- `function search <query>` carries its own mode syntax: a bare query ranks over names and descriptions, `"…"` greps code as a fixed string, and `/…/` greps it as a regex. Code results render ripgrep-style with `--context` lines.
- `function code get` returns the presigned envelope (`download_url`, `sha256`, `size`) in one request; `function code download` follows it and writes the tree, sending the presigned hop unauthenticated because S3 rejects a bearer alongside a signature.
- `function run log list` and `function run log tail` both default to the function's newest run, and both take `--run`/`-r` to name one. `list` takes `--since`/`--until` as RFC3339 or a duration; `tail` polls until the run ends, reading the end from the run's `ended_at` rather than the terminal `REPORT RunId:` line, which a `--filter-pattern` can hide. A failed poll is logged and retried rather than ending the tail — a blip mid-run should not discard a follow that is otherwise working, and a dropped poll never counts toward the two quiet polls that terminate it — with `--exit-status` to make the first one fatal instead, matching `trigger event watch`.

To invoke a function, point `maton api` at its URL: `maton api https://my-fn-3k9xq2v.maton.app`. The handler is called as `handler(event, context)` — the request arrives whole as a Maton event (version 1) with the body as a string on `event.get("body")`, absent when the request carries none, not as arguments bound by name — and the response carries the run's ID in `X-Function-Run-Id`, so `-i` names the run for `function run get` and `function run log list --run`.

`create` and `update` still accept `--dir`/`--file` and remain one-to-one with POST and PATCH, which is what a script wants. `update` also owns the two operations `deploy` deliberately declines: renaming (`--name`) and rolling back (`--version`). `update --handler` on its own takes no files: it republishes the current version's code under a new handler, which is a new version.

### Fixed

- `api`: print the response body when a failing response declares JSON but does not carry it, instead of replacing it with a decoder error. A function whose handler raises answers `500` with the plain body `Internal Server Error` under `content-type: application/json`, so the most common way to see a crash used to report `invalid character 'I' looking for beginning of value` and discard the response entirely. Such a body is now passed through verbatim and left unformatted, `--jq` and `--template` are skipped rather than run against a non-JSON body, and the exit status is unchanged.

## [0.2.0] - 2026-08-21

### Added

- `login --oauth`: sign in through the browser (authorization code + PKCE on a loopback listener) and store a renewable token instead of an API key. Access tokens refresh automatically; with `--interactive` the authorization URL is printed instead of opening a browser.
- `token`: print a valid access token for the active profile, renewing it first if expired, for programs calling the API directly. Exits non-zero for API-key profiles.
- `whoami`: `auth_type` field (`oauth` or `api_key`) in JSON output, and a `Signed in with:` line in text output.
- `MATON_API_URL` to override the API base URL.

### Changed

- Config, state, and data directories now resolve per-platform: `XDG_STATE_HOME`/`XDG_DATA_HOME` when set, and `%LocalAppData%`/`%AppData%\Maton CLI` on Windows. An existing `~/.config/maton` is still used when present.
- `logout` revokes OAuth refresh tokens upstream before clearing them locally; revocation failure is a warning, not an error.
- `login list` combines profile tags, e.g. `* name (active, oauth)`.
- `whoami`: `api_key` is omitted for OAuth profiles.
- `login switch` to an OAuth profile now requires a stored session.

## [0.1.6] - 2026-07-08

### Added

- Notion: `page_size` defaults to 100 when unset.
- GitHub: `duplicate` issue state and `duplicate_of` param.
- Google Ads: optional `--contains-eu-political-advertising` flag; `--bidding-strategy` on `campaign create` (default `MANUAL_CPC`).
- Trigger: `since_id` param for `event list`.
- HubSpot: valueless filter operators.

### Changed

- **Breaking:** inline value and from-file flags are now mutually exclusive (e.g. `--body`/`--text-file`); passing both errors instead of silently using one.
- **Breaking:** Salesforce `record create --all-or-none` with a single record now errors; pass a JSON array to create multiple records.
- **Breaking:** `--limit 0` / `--page-size 0` are now forwarded as a literal `0` instead of being dropped; for `connection list` this no longer means unlimited.
- **Breaking:** on `update`/`edit` commands, passing an explicit empty value (e.g. `--body ""`) now clears the field instead of being ignored.
- Renamed `repo search --topic` to `--topics` and `google-mail --fetch-format` to `--format` (old names kept as hidden aliases).
- GitHub: `issue close` reason limited to `completed` or `not_planned`.
- Salesforce: reject empty array for `composite`.
- Flags: use enum-aware and nil-pointer flag types based on param types.

### Fixed

- Align write payloads with SDK semantics across GitHub, Slack, Jira, HubSpot, Asana, Trello, Linear, Outlook, Notion, and Google apps — forward explicit empty strings and `0`/`false` values instead of dropping them.
- Google Mail: route ASCII names through `Mailbox.FormatHeader`.
- Google Calendar: interpolate path IDs raw to match SDKs.
- Google Drive: suppress permission notification on `--send-notification-email=false`.
- Google Ads: default empty `--date-range` to `LAST_30_DAYS`.
- Google Sheets: distinguish unset vs empty `--sheet-title`; default input option to `USER_ENTERED`.
- Google Tasks: allow `task list --show-completed=false` to hide completed tasks.
- Jira: omit `comment` `startAt` when unset.
- Issue/PR search: require a real qualifier flag.
- GitHub: allow `repo edit --topics ""` to clear topics; cross-repo issue duplicates; resolve `@me` for `pr edit` assignees.
- Asana: allow `task create` with workspace alone.

## [0.1.5] - 2026-06-24

### Added

- Maton: `trigger` commands — `create`, `get`, `list`, `update`, `delete`, plus `event` (with `list`, `watch`, and `--exec` flag with checkpointing) and `destination` (`create`, `list`, `watch`, etc.).
- `whoami` commands across apps.

### Changed

- Renamed `view` commands to `get` (`view` retained as a backward-compatible alias).
- `connection create` now takes the app as a positional argument.
- JSON output fields are now snake_case; connection commands still accept camelCase for backward compatibility.

### Deprecated

- Stripe: `stripe balance` in favor of `stripe balance get`.
- Linear and Notion: `user me` in favor of `whoami`.
- YouTube: `--mine` flag on `video`/`search`/`playlist list` commands.
- Slack: `--me`, `--username`, `--icon-emoji`, and `--reply-broadcast` flags on the `message` command.

### Fixed

- `trigger destination create --method` defaults to POST.
- `trigger create --destination` now defaults a destination's `method` to POST when omitted, matching `trigger destination create`.

## [0.1.4] - 2026-05-19

### Fixed

- GitHub: use GraphQL endpoint for `issue develop`, `issue pin`, `issue unpin`, and `issue transfer`.

### Removed

- Slack: `user set-presence` command (connection lacks required scope).
- Asana: `user me` command; use `asana whoami` instead.

## [0.1.3] - 2026-05-07

### Added

- Gmail: `whoami` command.
- `gmail` alias for `google-mail` commands.

### Changed

- Renamed `stripe balance-transaction` to `stripe transaction` (with `balance-transaction` as an alias).
- `connection list` returns all connections by default.

## [0.1.2] - 2026-05-07

### Added

- Slack: `bookmark` commands.
- Trello: `label`, `checklist`, `checkitem`, `search` commands; `board update`/`delete`; `list view`/`update`; `card move`/`comment`/`assign`/`label`; `--board` filter on `card list`; `member list`.
- YouTube: `video-category list`; `comment create`/`delete`; `playlist update`/`delete`/`remove-video`; `subscription create`/`delete` and `--for-channel` filter; `channel view` by `--handle`/`--username`.
- Stripe: `product`, `price`, `coupon`, `payment-method`, `balance-transaction` commands; `invoice finalize`/`pay`/`void`; `subscription items update`; `--payment-method-types` on `payment create`.
- Salesforce: `composite` and `version` commands; batch `record create`/`delete`; `record list --recent`/`--time-window`; bumped API to v63.0.
- OneDrive: `drive list`/`view`; `item update`/`invite`; large-file upload session; special folders for `item view`.
- Outlook: `calendar`, `event`, `contact` subcommands; filled `folder` and `message` gaps.
- Jira: `org view`; `--title` filter for `issue list`; `project view`; `project issue delete`; `comment list`.
- HubSpot: list associations; batch `read`/`create`/`update`/`archive`; list properties; `gdpr-contact` for `contact`.
- Notion: `--icon` flag for `page update`.
- Google Tasks: `--replace` flag for `tasklist update`.
- Google Drive: resumable uploads and file content update; `--no-metadata` flag for `file upload`.
- Google Calendar: defaults to `primary` calendar when `--calendar` is omitted.
- Asana: `project update`; `task list --parent`; `task search`.

### Changed

- Standardized on `--json` flag and removed `--format` flag (backward incompatible).
- Renamed `--body-from-file`, `--data-from-file`, `--text-from-file` flags to `--body-file`, `--data-file`, `--text-file`.
- HubSpot: renamed `delete` to `archive`.
- Removed `--media-only`, `--resumable`, and `--chunk-size` flags from `google-drive file update`/`upload`.
- Bumped Google Ads API to v24.
- Various help text cleanups (alias, app, google-sheets, jira `--cloud-id`, `--json` flag formatting, single-line examples).

### Fixed

- LinkedIn: auto-inject required header and drop JSON field projection on gateway commands.
- Google Sheets: render all rows as data with no header for `values` commands.

### Removed

- LinkedIn commands temporarily removed (backward incompatible).

## [0.1.1] - 2026-04-29

### Added

- `upgrade` command for updating the CLI to the latest version.

### Changed

- Grouped `login`, `logout`, and `whoami` under the auth commands section in help output.

## [0.1.0] - 2026-04-29

### Added

- Initial public release of the Maton CLI.
