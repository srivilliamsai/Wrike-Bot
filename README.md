## Installation Guide

1) **Create an account** (or) **Login to your Zoho Cliq account** : https://cliq.zoho.com/

2) **Now install using this link** : https://cliq.zoho.com/installapp.do?id=5962
   
![Wrike-for-Zoho-Cliq](/images/installation-page.png)

### OAuth 2.0 configuration

| Setting | Value |
| --- | --- |
| Authorization URL | `https://login.wrike.com/oauth2/authorize/v4` |
| Access Token URL | `https://login.wrike.com/oauth2/token` |
| Refresh Token URL | `https://login.wrike.com/oauth2/token` |
| Required Scopes | `Default wsReadWrite` |
| Token Transport | Header (handled automatically when using the Cliq connection) |

Configure a Cliq custom service named `wrikeconnection` with the values above and copy Cliq's generated redirect URL into Wrike's developer portal. Every Deluge script now references this service so the tokens are shared across bots, functions, schedulers, and widgets.

## Database Structure

The extension stores minimal configuration in three Zoho Cliq databases. Create the tables below (or update existing ones) so the scripts can read/write user preferences.

### `wrikedb`

| Field | Type | Mandatory | Unique | Hidden | Description / Sample |
| --- | --- | --- | --- | --- | --- |
| `userid` | Text | ✅ | ✅ | ✅ | Zoho user id, e.g. `1234567890` |
| `accountid` | Text | ✅ | ❌ | ❌ | Wrike account id, e.g. `IEAGV6SC` |
| `accountname` | Text | ✅ | ❌ | ❌ | Friendly name, e.g. `Wrike Demo Workspace` |
| `projectid` | Text | ✅ | ❌ | ❌ | Default Wrike folder id (`0` when unset) |
| `tasklistid` | Text | ✅ | ❌ | ❌ | Default Wrike task list id (`0` when unset) |

This table powers onboarding, command menus, widgets, and the scheduler.

### `wrikechannelmapping`

| Field | Type | Mandatory | Unique | Hidden | Description / Sample |
| --- | --- | --- | --- | --- | --- |
| `userid` | Number | ✅ | ❌ | ✅ | Owner user id (Zoho ids are numeric) |
| `projectid` | Text | ✅ | ❌ | ❌ | Wrike folder id |
| `projectname` | Text | ✅ | ❌ | ❌ | Project label (e.g. `Pitstop`) |
| `channelid` | Number | ✅ | ❌ | ❌ | Cliq channel id |
| `channeluniquename` | Text | ✅ | ✅ | ❌ | Channel unique name (`wrike-updates`) |
| `webhookid` | Text | ✅ | ✅ | ❌ | Wrike webhook id (`IEAGV6SCJAAB57FY`) |
| `events` | Large Text | ✅ | ❌ | ❌ | JSON array (`["TaskCreated","Comment"]`) |
| `threadid` | Number | ❌ (auto) | ❌ | ❌ | Message id for bot thread (`1678900000004567`) |

The webhook handler uses this table to route Wrike payloads into the correct channel.

### `wrikezapikey`

| Field | Type | Mandatory | Unique | Hidden | Description / Sample |
| --- | --- | --- | --- | --- | --- |
| `userid` | Text | ✅ | ✅ | ✅ | Zoho user id |
| `zapikey` | Encrypted Text | ✅ | ❌ | ✅ | Secure token (`ls7J8...VeA`) |

This table is only used when constructing the Wrike webhook callback URL (payloads hit `https://cliq.zoho.com/api/.../incoming?zapikey=<value>`).

> Tip: Zoho Cliq limits each table to 5 Text and 5 Number fields—by storing identifier columns such as `userid`, `channelid`, and `threadid` as Number fields we stay within that budget while still leveraging the other supported types (Text, Boolean, Encrypted Text, Large Text). Mark every column **mandatory/unique/hidden exactly as specified** so the Deluge scripts can safely rely on those constraints.
