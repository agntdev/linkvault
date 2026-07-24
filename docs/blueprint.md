# Secure File Share Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for admins to upload files and generate private download links with configurable expiry/usage limits. Recipients get single-use or time-limited links without needing authentication. Admins manage files, links, and access via commands and inline actions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- admins managing file sharing
- recipients with private download links

## Success criteria

- Admin receives unique download link with metadata after file upload
- Recipient can download file using link before expiry/max_uses
- Admin can list, revoke, and track file usage via commands
- Link auto-expires or becomes invalid after configured limits

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open admin dashboard with file management options
- **Upload File** (button, actor: user, callback: upload:init) — Initiate file upload flow with optional metadata input
  - inputs: file, title, expiry, max_uses, recipient_label
  - outputs: download_link, file_summary

## Flows

### File Upload
_Trigger:_ upload:init

1. Admin selects file and provides metadata
2. Bot stores file and generates link
3. Bot returns link with summary

_Data touched:_ FileRecord, DownloadLink

### Link Management
_Trigger:_ /list

1. Show active files/links with inline controls
2. Admin selects action (revoke, view stats)

_Data touched:_ FileRecord, DownloadLink

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **AdminAccount** _(retention: persistent)_ — Authorized admin identities with access rights
  - fields: telegram_id, permissions, created_at
- **FileRecord** _(retention: persistent)_ — Metadata and storage reference for uploaded files
  - fields: filename, size, mime_type, storage_pointer, uploader_id, upload_timestamp, ttl
- **DownloadLink** _(retention: persistent)_ — Secure access token with usage constraints
  - fields: token, file_id, recipient_identifier, max_uses, download_count, created_at, expires_at

## Integrations

- **Telegram** (required) — Bot API messaging and file storage
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/remove admin Telegram IDs
- Configure default expiry/max_uses
- Set file retention TTL

## Notifications

- Upload confirmation with download link
- Link expiry warnings
- Download usage alerts

## Permissions & privacy

- Only admins can upload/manage files
- Download links are unguessable tokens
- No user authentication required for downloads

## Edge cases

- Expired link access attempt
- Max usage exceeded
- Invalid token
- File deletion before download

## Required tests

- End-to-end upload → link distribution → download → expiry flow
- Command-based link revocation and stats verification

## Assumptions

- Single admin by default
- Files stored in platform-controlled storage
- Default 7-day expiry and 1-use limit
