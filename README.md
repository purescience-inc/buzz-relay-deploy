# buzz-relay-deploy

The compose template for a [Buzz](https://github.com/block/buzz) relay behind PureDesktop's shared books. It is set up the way it runs on a Hostinger VPS:

- Docker Manager runs the project;
- the server's Traefik gives it an HTTPS address, `<project>.<TRAEFIK_HOST>`.

**No secrets live in this repository.** Every password and key comes from the project's environment; the variables are listed in [`.env.example`](.env.example).

## How it is used

[PureRelays](https://github.com/Nikau-Dev/purerelays), a PureDesktop app, opens a relay for a new account in five steps:

1. It generates fresh secrets and stores them in PureDesktop's encrypted secret store.
2. It deploys this repository through Hostinger's Docker Manager. Docker Manager takes a GitHub URL and reads `docker-compose.yaml` from the `master` branch.
3. It passes the environment: the secrets, `TRAEFIK_HOST`, and `BUZZ_IMAGE`.
4. For the prototype image, it has already logged the server in to ghcr.io, using a post-install script at setup.
5. It waits for the relay to come on air.

Changing this file changes what **new** relays get. A running relay is only updated when someone redeploys it.

## Services

| Service | What it does |
| --- | --- |
| `relay` | The Buzz relay. Stock `ghcr.io/block/buzz:main`, or `ghcr.io/nikau-dev/buzz:proto-collab`, which adds document-sync event kinds 4470/24470/34470 and their own rate limit. |
| `relay-init` | Prepares the git data volume. |
| `postgres`, `redis` | The relay's database and cache. |
| `minio`, `minio-init` | Media storage and its bucket. |

## Deploying by hand

```bash
cp .env.example .env    # then replace every CHANGE_ME
docker compose -p buzz-<name> up -d
```

The server needs a Traefik on the `websecure` entrypoint with a `letsencrypt` resolver, as Hostinger's Docker template provides.
