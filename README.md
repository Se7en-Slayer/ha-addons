# Se7en-Slayer Home Assistant add-ons

A Home Assistant add-on repository. Add it under
**Settings → Add-ons → Add-on Store → ⋮ → Repositories**.

## Why this repository is public

Supervisor clones add-on repositories with `git.Repo.clone_from(url)` and no
credential handling whatsoever, so a private repository cannot be installed —
it fails at "Add repository", and the error presents as a bad URL rather than a
permissions problem.

Nothing here is sensitive by construction: a Dockerfile, a manifest, and this
file. Every secret Hermes uses lives in the add-on's `/data` volume on the
Home Assistant machine, which never touches this repository.

## hermes

A thin wrapper around the official `nousresearch/hermes-agent` image. It changes
exactly three things and nothing else:

| | |
|---|---|
| `HERMES_HOME=/data` | Hermes stores state in `/opt/data`; an add-on's persistent volume is `/data`, and that path is fixed. |
| `HERMES_WRITE_SAFE_ROOT=/data` | The agent's write sandbox, moved to match. Both must move together or the sandbox guards an empty directory. |
| `CMD ["gateway", "run"]` | Runs the Telegram gateway instead of the interactive CLI. |

Telegram runs in **polling** mode. The upstream image ships without
`python-telegram-bot[webhooks]`, so webhook mode fails at gateway start, and
polling needs no inbound port.

### Updating

Bump the `FROM` tag in `hermes/Dockerfile` and `version` in `hermes/config.yaml`
to the same upstream release. Home Assistant then shows an update button.

### Rollback

Uninstall the add-on and remove this repository; Supervisor deletes the locally
built image with it. The add-on's `/data` is removed on uninstall, so take a
copy of it first if the state matters.
