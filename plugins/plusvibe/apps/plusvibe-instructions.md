# PlusVibe

PlusVibe is a cold email outreach platform. The object hierarchy is
**workspace > campaign > sequence step > variation**, with **leads** attached to
campaigns and **email accounts** doing the sending.

## Always resolve the workspace first

Almost every operation requires `workspace_id`. Call `PLUSVIBE_GET_WORKSPACES`
before anything else. One API key spans every workspace on the account, so an
account commonly has several. When more than one exists and the user has not
named one, ask which workspace to work in rather than guessing.

Note the ID field names differ between endpoints: workspaces come back keyed
`_id`, campaigns come back keyed `id`. Reading the wrong key yields nothing.

## Cost control

`PLUSVIBE_LIST_ALL_CAMPAIGNS` returns every campaign in full, including the
complete sequence array with raw HTML email bodies. A single campaign can be
tens of kilobytes. Always pass `limit`, and filter with `campaign_id`,
`status`, or `campaign_type` when you can. When you only need one facet, prefer
the narrow reads: `PLUSVIBE_GET_CAMPAIGN_SUMMARY`,
`PLUSVIBE_GET_CAMPAIGN_STATUS`, `PLUSVIBE_GET_CAMPAIGN_NAME`.

## This platform sends real email to real people

Launching a campaign starts outbound cold email to everyone on its lead list.
Confirm with the user before `PLUSVIBE_LAUNCH_CAMPAIGN`,
`PLUSVIBE_REPLY_EMAIL`, `PLUSVIBE_FORWARD_EMAIL`, `PLUSVIBE_COMPOSE_EMAIL`, and
before any delete. Never chain a send off the back of a read. New campaigns are
created as drafts, which is the correct place to leave them for review.

Deletes are unrecoverable: `PLUSVIBE_DELETE_CAMPAIGN`, `PLUSVIBE_DELETE_LEADS`,
`PLUSVIBE_DELETE_EMAIL_ACCOUNT`, `PLUSVIBE_DELETE_TAGS`. Deleting a tag also
strips it from every email account and campaign that carried it.

Honor opt-out requests immediately by adding the address to the blocklist with
`PLUSVIBE_ADD_BLOCKLIST_ENTRIES`.

## Reading the numbers honestly

Campaigns with open tracking disabled (`is_emailopened_tracking: 0`) report
`open_rate: 0` and `opened_count: 0` no matter how the campaign is actually
performing. Check that flag before presenting open rate at all, and lead with
reply rate, which is always real.

`sent_count` counts messages, so it legitimately exceeds `lead_count` on a
multi-step sequence.

## Two tool surfaces

These `PLUSVIBE_*` operations are the native Caeros app provider, authenticated
by the API key in Settings > Apps > PlusVibe Auth. The same platform is also
reachable through PlusVibe's remote MCP server at `mcp.plusvibe.ai`, whose tools
are lower_snake_case (`list_campaigns`, `add_leads_to_campaign`). Either surface
works. The MCP additionally exposes inbox placement tests and custom holiday
calendars, which the native provider does not carry.
