# klickops marketplace

The blueprints behind the marketplace on [klickops](https://klickops.io). A
blueprint describes a whole project the platform installs in one go: the app
and its image, the database or bucket it needs, the address it answers on,
backups, and the settings a person fills in before installing.

klickops reads this repository every five minutes. A merged pull request is in
the marketplace within minutes, on the public catalog and on the Deploy page.

## Suggest or add a blueprint

- Want an app in the marketplace?
  [Suggest it](https://github.com/natrontech/klickops-marketplace/issues/new?template=suggest.yml)
  with the upstream project and what it needs to run.
- Want to add one yourself? Copy a file in `blueprints/`, name it after its
  `metadata.id`, and open a pull request. [CONTRIBUTING.md](CONTRIBUTING.md)
  has the review rules and the format.

## What a blueprint is

One YAML file per blueprint in `blueprints/`. The smallest useful one:

```yaml
apiVersion: klickops.io/v1alpha1
kind: Blueprint
metadata:
  id: uptime-kuma
  name: Uptime Kuma
  description: Self-hosted uptime monitoring and status dashboard.
  category: Observability
  icon: https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/uptime-kuma.svg
  homepage: https://uptime.kuma.pet
  sourceUrl: https://github.com/louislam/uptime-kuma
  maintainer: natrontech
spec:
  project:
    apps:
      - id: uptime-kuma
        spec:
          name: uptime-kuma
          image: louislam/uptime-kuma:2.5.3
          port: 3001
          volumes:
            - name: data
              size: 2Gi
              mountPath: /app/data
              backup: fs
          strategy: Recreate
    domains:
      - id: public
        app: uptime-kuma
    recovery:
      - id: daily-project-backup
        spec:
          enabled: true
          cadence: daily
          retention: 720h
          mode: auto
```

A blueprint composes what klickops already offers: apps with volumes, managed
PostgreSQL and Valkey, buckets, generated secrets, settings (`spec.inputs`),
bindings that hand a database address or the app's own URL to the app as an
environment key, a public address per app, firewall rules, scheduled jobs and
backup recommendations. [`blueprints/pocket-id.yaml`](blueprints/pocket-id.yaml)
uses most of them.

A blueprint is a starting point. Once installed, the project belongs to the
person who installed it: they change what they like, and the platform never
applies a newer blueprint to an existing project. A change here reaches future
installs only.

What the format leaves out on purpose: Helm charts, raw Kubernetes objects,
hooks, templating beyond `${inputs.<key>}` and `${project.name}`, hostnames
(the platform assigns one per domain), and secret values (declare a generated
secret instead).

## Checks

Every pull request runs the linter the platform runs when it loads this
catalog ([`.github/workflows/lint.yml`](.github/workflows/lint.yml)): a
blueprint that lints will install. Run it locally on Linux amd64:

```sh
base=https://app.klickops.io/api/klickops
version=$(curl -fsSL "$base/version" | jq -r .version)
curl -fsSL "$base/downloads/blueprintlint_${version}_linux_amd64.tar.gz" | tar -xz blueprintlint
./blueprintlint blueprints
```

Renovate opens a pull request when an upstream image publishes a new version
([`renovate.json`](renovate.json)). The bump is reviewed like any other change.

## License

[Apache-2.0](LICENSE). The apps a blueprint installs keep their own licenses.
