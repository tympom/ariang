# aria2 + AriaNg

Own Dockerfiles, built from official upstream sources only:

- `Dockerfile.aria2` — Alpine + aria2 package from Alpine's own repo
- `Dockerfile.ariang` — Nginx Alpine + AriaNg release zip from mayswind/AriaNg on GitHub

GitHub Actions builds both images on every push to `master` and publishes them
to GitHub Container Registry (GHCR), so `docker compose` can pull prebuilt
images instead of rebuilding on every host.

## One-time repo setup

```bash
git init
git add .
git commit -m "Initial commit: aria2 + AriaNg stack"
git branch -M master
git remote add origin git@github.com:YOUR_GITHUB_USERNAME/aria2-stack.git
git push -u origin master
```

Replace `YOUR_GITHUB_USERNAME` in `docker-compose.yml` (two places) with your
actual GitHub username/org, matching the repo owner GitHub Actions publishes
under.

No extra secrets needed for the build — `GITHUB_TOKEN` is provided
automatically by Actions and already has `packages: write` permission
(granted explicitly in the workflow).

## Making the images pullable

By default, GHCR packages are private. Either:

- Go to your GitHub profile → **Packages** → select `aria2` / `ariang` →
  **Package settings** → **Change visibility** → Public, or
- Keep them private and `docker login ghcr.io` on any machine that needs to
  pull them (a GitHub PAT with `read:packages` scope works as the password).

## Config file (not committed)

`aria2.conf` contains your real `rpc-secret` and is gitignored. On any new
checkout, create it from the template:

```bash
cp aria2.conf.example aria2.conf
```

Then edit `rpc-secret` to a real value:

```bash
openssl rand -hex 24
```

## Local build (dev/testing)

```bash
docker compose build
docker compose up -d
```

## Using the prebuilt GHCR images (after CI has run at least once)

```bash
docker compose pull
docker compose up -d
```

`docker compose up -d` without `--build` will pull the `image:` from GHCR if
not present locally, rather than build. `docker compose build` explicitly
rebuilds and tags locally under the same image name.

## First-run config

Open `http://<host>:8080`, go to **Settings → RPC**, set:

- Protocol: `HTTP`
- Host: your server's IP/hostname
- Port: `6800`
- Interface: `jsonrpc`
- Secret: the `rpc-secret` value from `aria2.conf`

Or use the URL-based shortcut:

```
http://<ariang-host>:<port>/#!/settings/rpc/set/http/<aria2-host>/6800/jsonrpc/<your-secret>
```

## Layout

```
.
├── .github/workflows/build.yml   # CI: builds + pushes both images to GHCR
├── .gitignore
├── Dockerfile.aria2
├── Dockerfile.ariang
├── aria2.conf.example            # committed template
├── aria2.conf                    # your real config, gitignored
├── docker-compose.yml
├── downloads/                    # gitignored, aria2 download dir
└── README.md
```

## Notes

- Keep RPC/web UI ports off public exposure where possible (e.g. behind
  Tailscale), especially on NAT-only VPS setups without a dedicated public
  IPv4.
- Update `ARIANG_VERSION` in both `docker-compose.yml` build args and the
  workflow's `build-args` when a newer AriaNg release comes out:
  https://github.com/mayswind/AriaNg/releases
