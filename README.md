# KM NetKit

**Version: 21R4.6.1** (based on 4D NetKit 21 R4)

> [!IMPORTANT]
> **KM NetKit is an unofficial, patched build of [4D NetKit](https://github.com/4d/4D-NetKit).** It is **not** published, maintained, or supported by 4D SAS. It exists to ship fixes ahead of the official release (originally for a presentation on 2026-10-21). For production use, prefer the official 4D NetKit that comes with 4D, and report issues in the official product to 4D, not to this repository.

KM NetKit is a fork of 4D NetKit, the 4D component that lets you connect your applications to third-party web services and consume their REST APIs directly from 4D code. It handles the OAuth 2.0 authentication flows for you and provides high-level, object-oriented clients for the most common [Microsoft Graph](https://docs.microsoft.com/en-us/graph/overview) and [Google Workspace](https://developers.google.com/workspace) services.

## Differences from the official 4D NetKit

| Area | Change |
|------|--------|
| Microsoft Graph mail notifier, pull mode | `onModify` and `onDelete` now fire. Previously only `onCreate` fired. Upstream polled mail with three separate delta streams, one per change type, but Graph only reports updates and deletes for messages a stream has already returned. Mail now uses one delta stream and sorts changes client-side, the same way calendar events already worked. See [`GraphNotification`](KM-NetKit/Project/Sources/Classes/GraphNotification.4dm). |
| Packaging | The project lives in [`KM-NetKit/`](KM-NetKit) and is built, signed, and published as GitHub releases by the [Publish](.github/workflows/publish.yml) workflow. Version numbers follow `package.json`. |

Everything else is identical to upstream 4D NetKit 21 R4. The class store namespace is still **`NetKit`** (`cs.NetKit.*`), so existing code runs unchanged.

## Installation

Download the component from the [releases](https://github.com/miyako/KM-NetKit/releases) page, or declare it as a GitHub dependency in your project's `Project/Sources/dependencies.json`:

```json
{
  "dependencies": {
    "KM-NetKit": {
      "github": "miyako/KM-NetKit"
    }
  }
}
```

> [!WARNING]
> KM NetKit and the official 4D NetKit both expose the `NetKit` namespace. Load only one of them in a given project.

## Overview

With KM NetKit you can:

* **Authenticate** to Microsoft and Google identity platforms using OAuth 2.0, in both `signedIn` (Authorization Code, interactive) and `service` (Client Credentials / JWT Bearer, unattended) modes — including PKCE, refresh tokens, and certificate-based client assertions.
* **Send and manage emails**: send, reply, append, move, copy, update, and delete messages; manage folders and labels; read messages in MIME, JMAP (4D mail object), or native Microsoft format.
* **Manage calendars and events**: list calendars, and create, read, update, and delete events, including attachments and attendees.
* **Read user profiles**: query individual users or paginated user directories.
* **Organize items**: read Outlook master categories (Microsoft).
* **Receive change notifications**: monitor mailbox and calendar changes through push/pull notifiers.

**Warning:** Shared objects are not supported by the NetKit API.

## Getting started

Authentication is always the first step. You create an [`OAuth2Provider`](KM-NetKit/Documentation/Classes/OAuth2Provider.md) object that holds your credentials and retrieves access tokens, then pass it to a service client (`Office365` or `Google`).

```4d
// 1. Create an OAuth2 provider
var $oauth2 : cs.NetKit.OAuth2Provider
$oauth2:=New OAuth2 provider({\
  name: "Microsoft"; \
  permission: "signedIn"; \
  clientId: "your-client-id"; \
  redirectURI: "http://127.0.0.1:50993/authorize/"; \
  scope: "https://graph.microsoft.com/.default"})

// 2. Create a service client and use it
var $office365 : cs.NetKit.Office365
$office365:=New Office365 provider($oauth2)

$office365.mail.send($mail)
```

For a complete walkthrough in **service mode**, see the [Tutorial: Authenticate to the Microsoft Graph API in service mode](KM-NetKit/Documentation/Tutorial.md).

## Documentation

### Authentication

| Page | Description |
|------|-------------|
| [New OAuth2 provider](KM-NetKit/Documentation/Methods/New-OAuth2-provider.md) | Method that instantiates an `OAuth2Provider` object. |
| [OAuth2Provider](KM-NetKit/Documentation/Classes/OAuth2Provider.md) | Core OAuth 2.0 client: token acquisition, refresh, PKCE, and JWT generation. |
| [JWT](KM-NetKit/Documentation/Classes/JWT.md) | Create, sign, and verify JSON Web Tokens. |

### Microsoft 365 (Microsoft Graph)

| Page | Description |
|------|-------------|
| [New Office365 provider](KM-NetKit/Documentation/Methods/New-Office365-provider.md) | Method that instantiates a `cs.Netkit.Office365` object |
| [Office365](KM-NetKit/Documentation/Classes/Office365.md) | Entry-point interface exposing the `mail`, `calendar`, `user`, and `category` clients (see below). |
| [Office365Mail](KM-NetKit/Documentation/Classes/Office365Mail.md) | `$office365.mail`: Send, read, move, copy, reply, update, and delete messages; manage folders. |
| [Office365Calendar](KM-NetKit/Documentation/Classes/Office365Calendar.md) | `$office365.calendar`: Manage calendars and events. |
| [Office365User](KM-NetKit/Documentation/Classes/Office365User.md) | `$office365.user`: Read Azure AD user profiles. |
| [Office365Category](KM-NetKit/Documentation/Classes/Office365Category.md) | `$office365.category`: Read Outlook master categories. |

### Google Workspace

| Page | Description |
|------|-------------|
| [Google](KM-NetKit/Documentation/Classes/Google.md) | Entry-point interface exposing the `mail`, `calendar`, and `user` clients (see below). |
| [GoogleMail](KM-NetKit/Documentation/Classes/GoogleMail.md) | `$google.mail`: Send, read, and manage Gmail messages and labels. |
| [GoogleCalendar](KM-NetKit/Documentation/Classes/GoogleCalendar.md) | `$google.calendar`: Manage Google calendars and events. |
| [GoogleUser](KM-NetKit/Documentation/Classes/GoogleUser.md) | `$google.user`: Read Google user profiles (People API). |

### Notifications

| Page | Description |
|------|-------------|
| [GraphNotification](KM-NetKit/Documentation/Classes/GraphNotification.md) / [GraphNotificationHandler](KM-NetKit/Documentation/Classes/GraphNotificationHandler.md) | Microsoft Graph change notifications for mail and calendar. |
| [GoogleNotification](KM-NetKit/Documentation/Classes/GoogleNotification.md) / [GoogleNotificationHandler](KM-NetKit/Documentation/Classes/GoogleNotificationHandler.md) | Google change notifications for mail and calendar. |


---

4D NetKit is developed by 4D SAS. KM NetKit is an independent, unofficial derivative of 4D NetKit and is not endorsed by 4D SAS.

(c) Microsoft, Microsoft Office, Microsoft 365, Microsoft Graph are trademarks of the Microsoft group of companies.

(c) Google, Gmail are trademarks of the Alphabet, Inc.
