# MiroFish local research setup

Status: preparation only. This configuration has not been built or run, and the
Codex Security scan is pending. Do not treat it as a production deployment.

Upstream: https://github.com/666ghj/MiroFish
Base commit: 7657031ac01184afe2cb220f5ee3545573b5e843
Fork: https://github.com/wmmalcolm/MiroFish
License: AGPL-3.0; retain the repository license and notices.

## Before running

1. Complete the Codex Security review on the intended deployment commit. Resolve
   applicable findings, including origin/CORS protection, authorization, upload
   handling, subprocess/tool boundaries, logging, and dependencies.
2. Install Docker with Compose on the machine that will host the research tool.
3. Select an OpenAI-compatible model provider and model, obtain a Zep Cloud key,
   and approve a spending limit with those providers. Never put credentials in a
   chat, Git commit, or screenshot.
4. Use synthetic or public input only. The model provider and Zep Cloud receive
   data during processing. No real applicant records or private documents.
5. Keep this service separate from any application that prohibits external
   network calls. It is a standalone research tool, not an embedded application
   integration.

## Configure

From the checked-out setup branch:

```bash
umask 077
cp .env.local.example .env.local
chmod 600 .env.local
python3 -c "import secrets; print(secrets.token_hex(32))"
```

Edit .env.local locally. Fill all five values; use the generated value for
SECRET_KEY. Do not paste it into chat. The Compose configuration rejects empty
required values. Do not use `docker compose config` in a shared log because its
normal output includes resolved environment values. Validate quietly:

```bash
docker compose --env-file .env.local -f compose.local.yaml config --quiet
```

The local environment file is ignored by Git and excluded from Docker's build
context. Optional boost credentials are intentionally absent.

## Start after review and configuration

```bash
docker compose --env-file .env.local -f compose.local.yaml up --build -d
docker compose --env-file .env.local -f compose.local.yaml ps
curl --fail --silent http://127.0.0.1:5001/health
```

Open http://127.0.0.1:3000. The health response should report status `ok` and
service `MiroFish Backend`. This checks startup, not provider credentials or
simulation correctness. Verify a small synthetic end-to-end experiment separately.

This file is standalone: use `-f compose.local.yaml` exactly as shown, rather
than merging it with upstream docker-compose.yml. Both published ports bind to
host loopback. The services bind to 0.0.0.0 only inside the container. The frontend
command explicitly sets its container host, and Flask debug is disabled.

Uploads and logs persist in named Docker volumes. Normal shutdown preserves them:

```bash
docker compose --env-file .env.local -f compose.local.yaml down
```

Do not add `--volumes` unless deletion of that research data is intended.

## Pilot limits and interpretation

Start with a small fictional audience and at most five rounds. The environment
sets a five-round default; it does not enforce a hard round or monetary cap.
Graph preparation, persona creation, reports, and conversations can also incur
charges. Use provider-side controls and monitor usage.

Label outputs SYNTHETIC. Use them to generate interview questions and alternative
copy, not to claim measured demand or make decisions about real individuals.

## Scope and remaining limitations

The configuration builds the checked-out source instead of pulling the floating
upstream application image. The upstream base image and OS packages are still
mutable; this is not a fully reproducible or audited supply chain. It uses the
upstream development servers, not a production web stack.

Loopback bindings do not provide authentication or protect against every browser
origin attack. Review those controls before loading credentials or sensitive
material. No TLS, public URL, application integration, live model execution, or
production readiness is claimed.

Validation completed during preparation: YAML parsing and configuration checks
for loopback port bindings, required environment values, disabled debug,
frontend container host, preserved data volumes, and environment-file exclusion.
Docker was unavailable in the preparation environment; build, startup, provider
authentication, and end-to-end simulations remain untested.
