# Contributing

A pull request adds or changes one blueprint. A maintainer reviews it against
the rules below and merges; the platform picks it up within minutes.

## Review rules

1. **One maintained upstream image per app.** The image the project publishes
   itself, from its own registry namespace (`ghcr.io/pocket-id/pocket-id`,
   `louislam/uptime-kuma`). No Bitnami, no MinIO, no third-party rebuilds, no
   images that bundle a second service.
2. **Pinned versions.** An explicit version tag (`2.5.3`, `v2.18.0-distroless`)
   or a digest. No `latest`, no floating tag (`v2`, `stable`). Renovate opens
   the bump when upstream releases; review it there.
3. **No secret values.** Declare a generated secret under
   `spec.project.secrets` and reference its key from the app. A plain
   environment key whose name looks like a secret is rejected.
4. **Nothing the platform assigns itself.** A domain names the app and
   nothing else; the platform picks the host. Service and volume sizes stay
   small enough for a first install.
5. **The linter passes.** CI runs `blueprintlint` on every pull request; it
   renders the blueprint through the same code the install uses.
6. **Complete metadata.** `id` (equals the filename), `name`, a one-sentence
   `description`, `category`, an icon, `homepage`, `sourceUrl` (the upstream
   repository), and `maintainer` (the GitHub handle that answers for the
   blueprint).

## The format in short

| Key | What it holds |
|---|---|
| `metadata` | Identity shown in the marketplace: `id`, `name`, `description`, `category`, `icon` or `iconLight` + `iconDark`, `homepage`, `sourceUrl`, `maintainer` |
| `spec.inputs` | Settings a person fills in before installing: `string`, `boolean` or `enum`, with `label`, optional `description`, `default`, `required` |
| `spec.project.apps` | Apps: `name`, `image`, `port`, `env`, `volumes`, `strategy`, `projectSecretKeys` |
| `spec.project.services` | Managed services: `type` `postgresql` (with `version`), `valkey`, or `s3-bucket` (with `params.maxSize`) |
| `spec.project.secrets` | Generated secrets: `scope` `project` or `app`, `key`, `generate.bytes` (16 to 128) and `generate.encoding` (`base64url` or `hex`) |
| `spec.project.bindings` | A service field (`url`, `host`, `port`, `username`, `password`, `database`, `endpoint`, `region`) or a domain field (`url`, `host`) handed to an app as `outputKey` |
| `spec.project.domains` | One public address per app: `id` and `app` |
| `spec.project.firewalls`, `scheduledJobs`, `recovery` | Firewall rules, scheduled jobs, and the backup recommendation the install offers |

Every resource carries a lowercase logical `id`; references between resources
use ids, not names. The only placeholders are `${inputs.<key>}` and
`${project.name}`.

## Before you open the pull request

```sh
base=https://app.klickops.io/api/klickops
version=$(curl -fsSL "$base/version" | jq -r .version)
curl -fsSL "$base/downloads/blueprintlint_${version}_linux_amd64.tar.gz" | tar -xz blueprintlint
./blueprintlint blueprints
```

The linter prints one line per blueprint when everything passes, and names
the first problem otherwise. Then install the blueprint once on klickops and
open the app: the review asks for that.

Commit subjects follow conventional commits: `feat: add <name>`,
`fix(<id>): <what>`, `chore(<id>): bump to <version>`.
