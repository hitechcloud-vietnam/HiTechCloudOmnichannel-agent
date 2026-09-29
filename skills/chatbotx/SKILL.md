---
name: hitechcloudomnichannel
description: Manage contacts, conversations, broadcasts, flows, sequences, appointments, minigames, and every other hitechcloudomnichannel workspace resource from the command line.
allowed-tools: Bash(hitechcloudomnichannel:*)
version: 1.1.1
homepage: https://github.com/hitechcloud-vietnam/HiTechCloudOmnichannel-agent/tree/main/skills/hitechcloudomnichannel
emoji: "🤖"
metadata:
  openclaw:
    requires:
      bins:
        - hitechcloudomnichannel
    os:
      - macos
      - linux
      - windows
    primaryEnv: HITECHCLOUDOMNICHANNEL_API_KEY
    envVars:
      - name: HITECHCLOUDOMNICHANNEL_API_KEY
        required: true
        description: HiTechCloudOmnichannel workspace API key (HiTechCloudOmnichannel Settings → Developer → API Keys).
      - name: HITECHCLOUDOMNICHANNEL_API_URL
        required: true
        description: Base API URL of the HiTechCloudOmnichannel instance, e.g. https://app.hitechcloud.vn/api.
      - name: HITECHCLOUDOMNICHANNEL_ALLOW_SELF_SIGNED_CERT
        required: false
        description: Set to "true" to skip TLS certificate validation, e.g. for a local/self-signed instance.
    install:
      - kind: node
        package: hitechcloudomnichannel
        bins: [hitechcloudomnichannel]
---

# hitechcloudomnichannel

Use the `hitechcloudomnichannel` CLI to manage a HiTechCloudOmnichannel workspace: contacts, conversations, broadcasts, flows,
sequences, appointments, minigames, analytics, and the rest of the workspace API. Commands are
generated at runtime from the connected workspace's OpenAPI spec, so `--help` on the live CLI is
the authoritative reference and this document can lag behind it.

## Rules for agents

1. Confirm credentials and workspace scope before anything else. Run `hitechcloudomnichannel token list`. A `401`
   means the user must set `HITECHCLOUDOMNICHANNEL_API_KEY` and `HITECHCLOUDOMNICHANNEL_API_URL`, or run
   `hitechcloudomnichannel config set --apiKey <key> --apiUrl <url>`. Do not run any other command until this
   returns a workspace, permission, and scope payload.
2. Discover before mutating. Run `hitechcloudomnichannel capabilities list` and the relevant `list` or `get`
   command to resolve names, ids, permissions, and current state before any write. Run
   `<command> --help` when the exact flags are unknown.
3. Messages, broadcasts, bulk operations, deletes, and flow publishing reach real customers.
   Confirm the exact recipients, filters, payloads, and schedules. There is no dry-run flag, so
   count the audience first with `contacts count --contactFilter <filter>` or
   `broadcasts audience list`.
4. Verify every write with the matching `get` or `list` command. An exit code of `0` is not proof:
   a command affected by a name collision (see below) can do nothing and still exit `0`.
5. Never expose API keys, saved config, or unredacted contact data in responses.
   `~/.hitechcloudomnichannel/config.json` holds plaintext credentials. Treat it as a secret file.

## Setup

Requires Node.js 24 or newer. Documented against `hitechcloudomnichannel` 1.8 or newer.

```bash
npm install -g hitechcloudomnichannel

# Save credentials once
hitechcloudomnichannel config set --apiKey <yourApiKey> --apiUrl <yourApiUrl>
# --apiUrl example: https://app.hitechcloud.vn/api

# Or via environment variables (no config file written)
export HITECHCLOUDOMNICHANNEL_API_KEY="your_api_key"
export HITECHCLOUDOMNICHANNEL_API_URL="https://app.hitechcloud.vn/api"

# Local dev / self-signed cert
hitechcloudomnichannel config set --allowSelfSignedCert true
```

Global options work on every command. `--apiKey`, `--apiUrl`, and `--allowSelfSignedCert` each
override the saved config for one run. `--refresh-spec` re-fetches the OpenAPI spec and clears the
1-hour cache at `~/.hitechcloudomnichannel/openapi-cache.json`. Use it, or set `HITECHCLOUDOMNICHANNEL_SPEC_CACHE_TTL_SECONDS`,
when a command is missing after a workspace API upgrade.

## Output and errors

Every command prints JSON, so output can be piped into `jq`. Add `--pretty` for indented output.
Errors come back as `{"error": true, "message": "...", "status": <httpStatus>}`. Branch on
`status`, not on `message`.

| Status | Meaning |
|---|---|
| `401` | Invalid or missing API key |
| `402` | Add-on required |
| `403` | Plan limit or missing permission |
| `404` | Not found |
| `429` | Rate limited |

## Workflow

```bash
# 1. Discover what a workspace token can see
hitechcloudomnichannel capabilities list
hitechcloudomnichanneloudomnichannel token list

# 2. Look up the ids you need — most write commands take an id, not a name
hitechcloudomnichanneloudomnichanneloudomnichanneloudomnichannel inboxes list
hitechcloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichannel contacts list --keyword "jane"
hitechcloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichanneloudomnichannel tags list

# 3. Act
hitechcloudomnichannel contacts message send email:jane@example.com --text "Hi Jane!" --inboxId <inboxId>

# 4. Verify
hitechcloudomnichannel contacts messages list email:jane@example.com --perPage 5
```

The same shape applies to broadcasts and flows:

```bash
# Broadcasts: count the audience, create, verify, stop if needed
hitechcloudomnichannel contacts count --contactFilter <filter>
hitechcloudomnichannel broadcasts create --channel <channel> --subaction <subaction> \
  --schedulesType <schedulesType> --schedulesAt <schedulesAt> --contactFilter <filter>
hitechcloudomnichannel broadcasts get <idOrName>
hitechcloudomnichannel broadcasts stop add <id>

# Flows: validate the spec, then publish
hitechcloudomnichannel flows validate --spec <spec>
hitechcloudomnichannel flows publish add <id> --spec <spec>
```

Help is available at every depth:

```bash
hitechcloudomnichannel --help                          # every command group
hitechcloudomnichannel contacts --help                 # actions in a group
hitechcloudomnichannel contacts message --help         # subactions
hitechcloudomnichannel contacts message send --help    # options for one action
```

## Contact identifiers

Wherever `<identifier>` appears in a command, the value must carry a prefix. A bare value returns
`404 Invalid identifier format`.

| Format | Example | Lookup by |
|---|---|---|
| `id:<value>` | `id:123456789` | Contact ID |
| `email:<value>` | `email:user@example.com` | Email address |
| `phone:<value>` | `phone:+84708123123` | Phone number |

## Command groups

Every group supports `--help` for its exact flags. The full per-command catalog with flags is in
[`references/commands.md`](references/commands.md). Read it when you need the command shape for a
group before calling `--help`; for most tasks the Workflow examples above are enough.

| Group | Commands |
|---|---|
| `contacts` | list, count, create, get, update, delete, upsert, block, import, export, bulk-tags, bulk-delete, tags, custom-fields, notes, sequences, messages, message send, flow add, coupons |
| `conversations` | list, get, assign, archive, enable-bot, disable-bot, messages, message send/delete |
| `broadcasts` | list, get, audience, create, schedule, stop, resume, resend, duplicate, delete |
| `flows`, `sequences`, `keywords`, `triggers`, `webhooks`, `external-webhooks`, `schemas` | flow create/validate/publish/draft/versions/import, sequence steps, automation triggers |
| `members`, `teams`, `tags`, `custom-fields`, `bot-fields`, `folders`, `inboxes`, `capabilities`, `token` | workspace admin and discovery |
| `analytics` | contact counts, contacts-by-dimension, bot-messages-by-result, broadcasts/flows/sequences stats, mac-active-count |
| `ai-agents`, `ai-files`, `ai-functions`, `ai-mcp-servers` | AI agent configuration and knowledge base |
| `products`, `product-categories`, `coupon-topics`, `coupons`, `minigames`, `questionnaires`, `appointment-calendars`, `appointments`, `ref-links`, `qr-codes` | commerce and engagement |
| `integrations`, `whatsapp`, `smtp-integrations`, `spreadsheets`, `facebook-lead-ads`, `fb-comments`, `ig-comments`, `ig-stories`, `contact-scans`, `messenger-personas`, `messenger-channels`, `zalo-channels`, `webchats`, `user-persistent-menus`, `dynamic-images`, `email-topics`, `media-library` | integrations and channels |
| `ads` | conversion-rules, funnel, analytics, capi-delivery, conversions-export, ad-accounts, campaigns |
| `error-logs`, `appointment-external-calendars`, `appointment-reminders` | misc |

## Command-name collisions

Command names are derived from the API path and method alone. When two operations under one
resource reduce to the same name, the CLI registers the first and skips the second. It prints
`Warning: duplicate command name "..." — skipping` on stderr but still exits `0`. Verified cases:

- `bot-fields update <idOrName> --value <value>` (single field) is unreachable. Use
  `bot-fields update --fields <fields>`, which updates by id or name in batch.
- `contacts custom-fields update <identifier> <idOrName> --value <value>` (single-field PUT) is
  unreachable. Use `contacts custom-fields update <identifier> --operations '[{"customFieldId":"...","operation":"set","value":"..."}]'`
  for a single field too.
- `contacts custom-field delete <identifier>` clears every custom field on the contact, not one.
  The per-field delete has no CLI command.
- `integrations find-by-ai --provider <provider>` is GET only. Connecting or disconnecting an AI
  provider has no CLI command; use the API directly.
- `ads conversion-rules`, `ads find-by-conversion-rules`, `ads campaigns`,
  `media-library folders`, `media-library find-by-folders`, `media-library files`, and
  `media-library find-by-files` each have duplicated generated names. Only the first-registered
  operation is reachable.
- `analytics flows-stats <flowId>`: GET (fetch) wins. The DELETE (reset stats) variant has no CLI
  command.
- `minigames update <id>`: only one of PUT (full replace) and PATCH (partial) is reachable.

When a documented action returns `404` or silently does nothing, assume a collision and call the
workspace REST API directly instead of trying other flag combinations.

## Notes

- `contacts message send` and `conversations message send` accept either `--flowId` or free text
  with `--text`. Check `--help` before sending.
- `broadcasts create` needs exactly one of `--flowId` or `--templateId`. `--schedulesAt` is
  required only when `--schedulesType future` and the broadcast is not saved as a draft.
- Filter on the server with `--contactFilter` instead of filtering results client side.
  `hitechcloudomnichannel contacts filter-fields` documents every supported field and operator.

## MCP alternative

Agents in MCP-capable IDEs can use the `hitechcloudomnichannel-mcp` server instead of the CLI. It exposes the
same workspace API as MCP tools, filtered by the token's scopes. Setup and the default tool list
are in `skills/hitechcloudomnichannel-mcp/SKILL.md` of this repository.
