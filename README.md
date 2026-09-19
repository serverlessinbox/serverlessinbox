# ServerlessInbox

**A complete JMAP email server that runs serverless in your own AWS account — for about $1 a month.**

ServerlessInbox is email the way serverless should work: no servers to patch, no idle compute to pay for, nothing running when nobody is mailing. Lambda, DynamoDB, SQS and SES do the work; you pay for what you use. For a small company that is roughly a dollar a month in AWS runtime.

It speaks [JMAP](https://jmap.io) (RFC 8620 / 8621) — a modern, stateless, JSON email API — so it is as much an email *engine* for your own tooling and integrations as it is a mailbox.

> **Status: public beta.** It runs real mail today, but expect rough edges and breaking changes between releases. Provided as-is, without warranty — see the [software terms](TERMS.md).

## What you get

One deploy gives you:

- **JMAP API** — email, mailboxes, threads, contacts, address books, sharing, identities, WebSocket push
- **Webmail** and an **admin UI**
- **Admin API** and a **blob API** for attachments
- **DNS automation** for your mail domain (SPF, DKIM, DMARC)
- **SES reputation handling** built in — bounce and complaint processing and suppression, so your SES account stays healthy
- Multi-tenant from the ground up; Cognito as the default identity provider

## Install

Deploy from a CloudFormation template or as CDK constructs. You need an AWS account and a Route 53 hosted zone for your mail domain. The beta supports **eu-west-1** only.

👉 **[Install guide](https://docs.serverlessinbox.com/tutorials/install-mailbox-standard/)** · [Documentation](https://docs.serverlessinbox.com)

New AWS accounts start in the SES sandbox. The docs include a page written for AWS reviewers that explains how ServerlessInbox handles bounces and complaints, to support your sandbox-exit request.

## Open shell, closed core

Everything you deploy is auditable:

| Open source (Apache-2.0) | Closed |
|---|---|
| CDK constructs and CloudFormation templates, IDL and generated SDKs, admin UI, JMAP server library, documentation | The Go Lambda implementations, webmail UI (for now) |

The closed Lambdas are pre-compiled and **signed**: each binary verifies its own signature at startup. The infrastructure around them — every IAM permission, every resource — is open and yours to inspect.

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
