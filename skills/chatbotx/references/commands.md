# HiTechCloudOmnichannel CLI command catalog

Companion to `../SKILL.md`. Read the rules, workflow, and command-name collisions there first.

Commands are generated at runtime from the connected workspace's OpenAPI spec, so this catalog
can lag behind the live CLI. `<command> --help` is the authoritative source for flags. Wherever
`<identifier>` appears, the value must carry an `id:`, `email:`, or `phone:` prefix.

## Contacts

```bash
hitechcloudomnichannel contacts list                               # [--page --perPage --sort --keyword --contactFilter]
hitechcloudomnichannel contacts count                               # Count matching filter [--page --perPage --sort --keyword --contactFilter]
hitechcloudomnichannel contacts create --email <email>              # [--phoneNumber --contactId --firstName --lastName]
hitechcloudomnichannel contacts get <identifier>
hitechcloudomnichannel contacts update <identifier>
hitechcloudomnichannel contacts delete <identifier>
hitechcloudomnichannel contacts upsert add <identifier>             # Insert or update by identifier
hitechcloudomnichannel contacts block <identifier>
hitechcloudomnichannel contacts unblock <identifier>
hitechcloudomnichannel contacts filter-fields                       # Field/operator reference for --contactFilter

hitechcloudomnichannel contacts import --fileId <fileId> --channel <channel> --inboxId <inboxId>
hitechcloudomnichannel contacts export --fields <fields>            # [--contactIds --exportAll --filter]

hitechcloudomnichannel contacts bulk-tags --contactIds <contactIds> --tags <tags>
hitechcloudomnichannel contacts bulk-delete --contactIds <contactIds>
hitechcloudomnichannel contacts bulk-sequences --contactIds <contactIds> --sequenceIds <sequenceIds>

hitechcloudomnichannel contacts tags list <identifier>
hitechcloudomnichannel contacts tag add <identifier> --tagIds <tagIds>
hitechcloudomnichannel contacts custom-fields update <identifier> --operations <operations>  # batch set/append/prepend/increase/decrease
hitechcloudomnichannel contacts notes list <identifier>
hitechcloudomnichannel contacts note add <identifier> --text <text>
hitechcloudomnichannel contacts sequences list <identifier>
hitechcloudomnichannel contacts sequence add <identifier> --sequenceIds <sequenceIds>
hitechcloudomnichannel contacts inboxes list <identifier>            # per-channel contact-inbox connections

hitechcloudomnichannel contacts messages list <identifier>           # [--perPage --cursor]
hitechcloudomnichannel contacts message send <identifier>            # [--text --files --mediaFile --flowId --nodeId --inboxId ...]
hitechcloudomnichannel contacts flow add <identifier> --flowId <flowId>
hitechcloudomnichannel contacts coupons list <identifier>            # coupons issued to the contact
```

## Conversations

```bash
hitechcloudomnichannel conversations list                            # [--botCategory --assignedId --channel --status --keyword --tags ...]
hitechcloudomnichannel conversations get <id>
hitechcloudomnichannel conversations assign add <id> --assignedId <assignedId>   # null clears
hitechcloudomnichannel conversations archive add <id>
hitechcloudomnichannel conversations enable-bot add <id>
hitechcloudomnichannel conversations disable-bot add <id>

hitechcloudomnichannel conversations messages list <conversationId>  # [--perPage --cursor]
hitechcloudomnichannel conversations message send <conversationId>   # same options as contacts message send
hitechcloudomnichannel conversations message delete <conversationId> <messageId> --createdAt <createdAt>
```

## Broadcasts

```bash
hitechcloudomnichannel broadcasts list
hitechcloudomnichannel broadcasts get <idOrName>
hitechcloudomnichannel broadcasts audience list <idOrName>            # [--page --perPage]
hitechcloudomnichannel broadcasts create --channel <channel> --subaction <subaction> --schedulesType <schedulesType> \
  --schedulesAt <schedulesAt> --contactFilter <contactFilter>
  # either flowId or templateId required (not both); schedulesAt required when schedulesType=future
hitechcloudomnichannel broadcasts schedule add <id> --schedulesType <schedulesType>  # [--schedulesAt]
hitechcloudomnichannel broadcasts stop add <id>
hitechcloudomnichannel broadcasts resume add <id>
hitechcloudomnichannel broadcasts resend add <id>                     # clone a sent/failed broadcast
hitechcloudomnichannel broadcasts duplicate add <id>
hitechcloudomnichannel broadcasts delete <id>                         # fails while status is sending
```

## Flows and automation

```bash
hitechcloudomnichannel flows list                                     # [--page --perPage --active]  active defaults true
hitechcloudomnichannel flows get <id>
hitechcloudomnichannel flows create --name <name>                     # [--folderId --spec --nodes --edges --publish]
hitechcloudomnichannel flows validate --spec <spec>                    # compile/validate flow-spec DSL without publishing
hitechcloudomnichannel flows publish add <id>                          # [--spec | --nodes --edges]
hitechcloudomnichannel flows draft update <id>                          # overwrite the draft in place
hitechcloudomnichannel flows versions list <id>
hitechcloudomnichannel flows import --fileId <fileId>                   # [--folderId] async import of an exported flow file
hitechcloudomnichannel schemas flow-spec                                # JSON Schema for the flow-spec DSL

hitechcloudomnichannel sequences list / get / create / update / delete
hitechcloudomnichannel sequences steps update <id> --order <order>       # create/update one step
hitechcloudomnichannel keywords list                                      # automated keyword responses [--type inbound|comment]
hitechcloudomnichannel triggers list / create / update / delete
hitechcloudomnichannel webhooks list / create / delete
hitechcloudomnichannel external-webhooks list / create / delete            # [--provider make|n8n]
```

## Team and workspace admin

```bash
hitechcloudomnichannel members list / get
hitechcloudomnichannel teams list / get / create / update / delete
hitechcloudomnichannel teams member add <id> --userIds <userIds>
hitechcloudomnichannel tags list / create / get / update / delete
hitechcloudomnichannel custom-fields list / create / get / update / delete
hitechcloudomnichannel bot-fields list / get / delete
hitechcloudomnichannel bot-fields update --fields <fields>                 # JSON array of {id,value} or {name,value}
hitechcloudomnichannel folders list --folderType <tag|customField>          # [--parentId]
hitechcloudomnichannel inboxes list
hitechcloudomnichannel capabilities list                                     # [--include] discover ids/names an agent needs
hitechcloudomnichannel token list                                             # calling token's workspace/permission/scopes
```

## Analytics

Time-range commands take `--from --to --timezone`. Some also take `--granularity`.

```bash
hitechcloudomnichannel analytics contact-counts-per-day
hitechcloudomnichannel analytics new-contacts-count
hitechcloudomnichannel analytics active-contacts-count
hitechcloudomnichannel analytics contacts-by-dimension --dimension <country|channel|source>
hitechcloudomnichannel analytics bot-messages-by-result              # [--granularity]
hitechcloudomnichannel analytics broadcasts-stats <broadcastId>
hitechcloudomnichannel analytics flows-stats <flowId>
hitechcloudomnichannel analytics sequences-steps-stats <sequenceId> <stepId>
hitechcloudomnichannel analytics mac-active-count                     # no time range — current billing period
```

## AI

```bash
hitechcloudomnichannel ai-agents list / get / create / update / delete
hitechcloudomnichannel ai-files list / get / create / delete           # knowledge-base files [--file --url]
hitechcloudomnichannel ai-functions list / get / create / update / delete
hitechcloudomnichannel ai-mcp-servers list / get / create / update / delete
```

## Commerce and engagement

```bash
hitechcloudomnichannel products list / get / create / update / delete
hitechcloudomnichannel product-categories list / create / update / delete
hitechcloudomnichannel coupon-topics list / get / create / update / archive / unarchive / delete
hitechcloudomnichannel coupon-topics issue add <id> --contactId <contactId>
hitechcloudomnichannel coupons list
hitechcloudomnichannel minigames list / get / create / update / delete / bulk-delete
hitechcloudomnichannel minigames plays list <id> --contactId <contactId>
hitechcloudomnichannel minigames players list <id>
hitechcloudomnichannel questionnaires list / get / create / update / delete / duplicate
hitechcloudomnichannel questionnaires submissions list <id>
hitechcloudomnichannel appointment-calendars list / get / create / update / delete
hitechcloudomnichannel appointments list / get / create / cancel / delete
hitechcloudomnichannel ref-links list / get / create / update / delete
hitechcloudomnichannel qr-codes list / get / create / update / delete
```

## Integrations and channels

```bash
hitechcloudomnichannel integrations list / get
hitechcloudomnichannel integrations status-token-errors
hitechcloudomnichannel whatsapp templates                              # [--inboxId --integrationWhatsappId --status]
hitechcloudomnichannel smtp-integrations list / get / create / update / delete
hitechcloudomnichannel spreadsheets list / get / create / update / delete   # connected Google Sheets
hitechcloudomnichannel facebook-lead-ads list / get / create / update / delete
hitechcloudomnichannel fb-comments list / get / create / update / delete
hitechcloudomnichannel ig-comments list / get / create / update / delete
hitechcloudomnichannel ig-stories list / get / create / update / delete
hitechcloudomnichannel contact-scans status --inboxId <inboxId>
hitechcloudomnichannel messenger-personas list
hitechcloudomnichannel messenger-channels tag-sync update <id> --enabled <enabled>
hitechcloudomnichannel zalo-channels tag-sync update <id> --enabled <enabled>
hitechcloudomnichannel webchats list / get / create / update / delete
hitechcloudomnichannel user-persistent-menus list / get / create / update / delete
hitechcloudomnichannel dynamic-images list / get / create / update / delete
hitechcloudomnichannel email-topics list / get / create / update / delete
hitechcloudomnichannel media-library files-upload-url --fileName <fileName> --mimeType <mimeType>
hitechcloudomnichannel media-library files-move --fileIds <fileIds>       # [--folderId]
```

## Ads

```bash
hitechcloudomnichannel ads conversion-rules                              # list rules; create via same command, see command-name collisions in SKILL.md
hitechcloudomnichannel ads funnel / funnel-timeseries / analytics-overview / analytics-timeseries
hitechcloudomnichannel ads capi-delivery
hitechcloudomnichannel ads conversions-export                             # [--allChannels]
hitechcloudomnichannel ads ad-accounts list <channel>
hitechcloudomnichannel ads campaigns                                       # list/create messaging ad campaigns, see command-name collisions in SKILL.md
hitechcloudomnichannel ads campaigns-publish <id> / campaigns-pause <id> / campaigns-retry <id>
hitechcloudomnichannel ads campaigns-insights                               # POST, adIds up to 500
```

## Misc

```bash
hitechcloudomnichannel error-logs list                                     # [--page --perPage --sort --keyword]
hitechcloudomnichannel appointment-external-calendars list / delete <integrationId>
hitechcloudomnichannel appointment-reminders list
```
