# ServerlessInbox

**A complete JMAP email server that runs serverless in your own AWS account — modelled at under $1 a month.**

ServerlessInbox is email the way serverless should work: no servers to patch, no idle compute to pay for, nothing running when nobody is mailing. Lambda, DynamoDB, SQS and SES do the work; you pay for what you use. For a small company the modelled AWS bill is about $0.77 a month, and an idle deployment is the price of your Route 53 hosted zone — roughly $0.50. Those are [calculated figures with the workload and unit prices shown](https://docs.serverlessinbox.com/tutorials/#cost-estimate), not measurements from a production fleet.

It speaks [JMAP](https://jmap.io) (RFC 8620 / 8621) — a modern, stateless, JSON email API — so it is as much an email *engine* for your own tooling and integrations as it is a mailbox.

> **Status: public beta.** Expect rough edges and breaking changes between releases. Provided as-is, without warranty — see the [software terms](TERMS.md).

## What you get

The goal from the start has been a first stable release covering core mail, and that is what exists today. Core mail here means the whole path a message takes and the machinery around it: JMAP mail — email, mailboxes, threads, contacts, address books, sharing, identities — webmail and an admin UI, an admin API and a blob API, generated DNS records, SES bounce and complaint handling with suppression, multiple domains, aliases, shared mailboxes with delegated access, groups, plus-addressing, full-text search, and push. That is a working mail server, not a preview of one.

What is not there is in the [roadmap](#roadmap) below — import and export first.

One deploy gives you:

- **JMAP API** — email, mailboxes, threads, contacts, address books, sharing, identities, WebSocket push
- **Webmail** and an **admin UI**
- **Admin API** and a **blob API** for attachments
- **DNS records generated for you** (MX, SPF, DKIM, DMARC) — you apply them to Route 53 with one click in the admin UI, and you choose the SPF and DMARC policy
- **SES reputation handling** built in — bounce and complaint processing and suppression, so your SES account stays healthy
- **More than one domain**, several addresses per person through aliases, **shared mailboxes with delegated access**, and groups — all managed from the admin UI or the Admin API
- **Plus-addressing** works out of the box: `you+anything@your-domain.com` lands in your inbox, no configuration
- Cognito as the default identity provider, with per-deployment configuration

One deployment serves one organisation. Tenant isolation is enforced in the data layer, but running several organisations on one deployment is not available yet.

## Is this for you yet?

**Try it if** you're comfortable in your own AWS account, want a cheap JMAP mailbox on a domain you can experiment with, and don't mind reporting a rough edge instead of hitting a support line.

**Not yet if** you need to move existing mail in, run several organisations from one deployment, or have someone to call at 3am.

## Install

Deploy from a CloudFormation template or as CDK constructs. You need an AWS account and a Route 53 hosted zone for your mail domain. The beta supports **eu-west-1** only.

👉 **[Install guide](https://docs.serverlessinbox.com/tutorials/install-mailbox-standard/)** · [Documentation](https://docs.serverlessinbox.com)

New AWS accounts start in the SES sandbox. The docs include a page written for AWS reviewers that explains how ServerlessInbox handles bounces and complaints, to support your sandbox-exit request.

## Roadmap

Core mail is the finished part. Everything below is not, listed in the order it is queued — import and export first, and that is what is being worked on now.

| On the way | Where it stands |
|---|---|
| **Import and export** | Being worked on now. There is no migration tool yet; until it lands, mail moves in and out through the JMAP API. |
| **Personal access tokens** | API access today goes through OAuth, not long-lived tokens. |
| **MCP server** | Letting an AI assistant work with your mailbox over the Model Context Protocol is largely built, but unusable until personal access tokens land. Those are the blocker, not the MCP side. |
| **Malware and authentication verdicts affecting routing** | SES scans inbound mail and records its verdict on the message, but placement does not use it yet: mail SES flags as carrying malware is delivered to the inbox rather than to junk. SPF, DKIM and DMARC results are recorded and visible in the same way, and likewise do not affect routing yet. The spam verdict is the one that does — spam goes to junk today. |
| **Sieve scripts** | No server-side filtering rules yet. |
| **Date-range search** | Full-text search works; filtering results by "before" and "after" does not yet. |
| **AI-assisted categorisation** | Planned, using the cheapest Bedrock model that does the job. |
| **Calendar** | Planned, not built. |
| **Support conversations in-product** | You can open a support case and grant time-boxed, read-only access from the admin UI, but the discussion itself still happens over your normal support channel. |
| **IMAP/SMTP bridge** | Only if enough people ask. It needs always-on compute, which breaks the cost model, so it would be opt-in and cost extra. |
| **Multi-tenancy, and regions beyond eu-west-1** | One deployment serves one organisation, and the beta runs in eu-west-1 only. |

Three things about a deployment's terms, rather than its features:

- **No lock-in.** Any subscription will always be endable each month — no annual commitment, no term to sit out. And when a paid subscription ends, mail accounts keep working on the free tier; only extended features switch off. You should keep paying because you like the product, not because leaving is expensive. Import and export are on the roadmap above for the same reason.
- **The free tier will shrink after the beta.** A free tier always exists, and we expect it to come down to one mailbox for new deployments once the beta ends. During the beta it allows 10 mail accounts across up to 3 domains, and beta participants keep what they are running then — those 10 accounts and 3 domains included — indefinitely, so testing now is what secures multi-account use later. Mail never stops because of licensing — see the [licence model](https://docs.serverlessinbox.com/explanation/license-model/).
- **Updates arrive on their own by default.** The `artifactTracking` parameter defaults to `minor`, so a deployment picks up patch releases within its current minor version automatically (fixes and maintenance, no new features). Set it to `pinned` at install time if you'd rather decide yourself — worth doing during a beta.

The [discussions](../../discussions) are the place to argue for what should come first.

## How you can help

Three things are worth more to this project than a star:

1. **Your actual AWS bill**, broken down by service, after a month of real use. The [cost estimate](https://docs.serverlessinbox.com/tutorials/#cost-estimate) is calculated, not measured — help turn it into measured data.
2. **Where the install tripped you up**, even if you worked it out. Especially anything the docs told you that turned out to be wrong.
3. **Which JMAP client you got working**, and which you couldn't. No third-party client has been formally verified yet.

Post any of it in [Discussions](../../discussions), or open an [issue](../../issues/new/choose).

## Open shell, closed core

**The mail engine itself is closed.** The Go Lambdas that process, store and send your mail ship as pre-compiled binaries, and so does the webmail UI for now. Everything around them is open, so you can audit what the closed parts are allowed to do:

| Open source (Apache-2.0) | Closed |
|---|---|
| CDK constructs and CloudFormation templates, IDL and generated SDKs, admin UI, JMAP server library, documentation | **The Go Lambda implementations — the entire data plane** · the webmail UI (for now) |

The closed Lambdas are pre-compiled and **signed**: each binary verifies its own signature at startup. The infrastructure around them — every IAM permission, every resource — is open and yours to inspect.

**The Lambdas holding the most privileged IAM roles are the open ones.** The control-plane TypeScript Lambdas — the pieces that can change your infrastructure — ship as readable source precisely so you can audit what they are allowed to do. The closed binaries are the mail engine, and the open CDK around them defines exactly what that engine may touch.

The Lambdas check their licence with the ServerlessInbox licence server. It never receives email content, contacts or user names — the [software terms](TERMS.md) list exactly what it does receive.

## Repositories

| Repo | What it is |
|---|---|
| [mailbox-cdk](https://github.com/serverlessinbox/mailbox-cdk) | CDK constructs — the infrastructure |
| [mailbox-apps](https://github.com/serverlessinbox/mailbox-apps) | Ready-to-deploy app / CloudFormation templates |
| [mailbox-idl](https://github.com/serverlessinbox/mailbox-idl) | JMAP JSON schemas, admin API protobuf — source of the generated SDKs |
| [jmap-server-go](https://github.com/serverlessinbox/jmap-server-go) | JMAP server library for Go |
| [admin-ui](https://github.com/serverlessinbox/admin-ui) | Admin frontend |
| [artifact-registry](https://github.com/serverlessinbox/artifact-registry) | Release artifact resolution |

## Issues, questions, feedback

**All issues live here** — whatever the component. You don't need to know which repo a problem is in.

- 🐛 [Report a bug or install problem](../../issues/new/choose)
- 💬 [Questions, ideas and show-and-tell](../../discussions)
- 🔒 Security issues: please don't open a public issue — see [SECURITY.md](https://github.com/serverlessinbox/.github/blob/main/SECURITY.md)

## License

The open-source repositories are licensed under [Apache-2.0](LICENSE). The pre-compiled Lambda binaries are covered by the [ServerlessInbox Software Terms](TERMS.md).
