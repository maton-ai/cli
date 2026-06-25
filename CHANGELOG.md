# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
