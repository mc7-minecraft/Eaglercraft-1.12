# Working in this repo

## What this app is
A single static page: `index.html` is the whole application (an Eagletcraft game
launcher). All logic runs in the browser — there is no backend, no database, no
package manager and no build step. Nothing needs credentials to boot.

Non-obvious runtime detail: the page loads its game payload at click time from a
public CDN (`cdn.jsdelivr.net`) directly in the visitor's browser. That request
never touches this sandbox, so a network-blocked browser will show the page's own
error alert instead of the game.

## How it runs here
`docker-compose.base44.yml` serves the repo root with nginx (`nginx:alpine`) on
host port 3000, with the repo bind-mounted read-only at `/usr/share/nginx/html`.
Because files are read from the bind mount with `Cache-Control: no-store`,
editing `index.html` takes effect on refresh — no rebuild, no restart.

Two non-obvious things were needed to make those files readable and reachable:

- `nginx/nginx.conf` is mounted over `/etc/nginx/nginx.conf` and uses
  `user root;` instead of the stock `user nginx;`. The sandbox checks out the repo
  root as mode 700 (root-only), so an unprivileged worker cannot traverse the bind
  mount and every request 403s. Do not "fix" this by loosening the repo directory
  permissions — the platform owns them.
- The healthcheck probes `http://127.0.0.1/`, not `localhost`: nginx listens on
  IPv4 only and busybox `wget` would otherwise try `::1` and report connection
  refused while the site is perfectly healthy.

This is a development setup only: nginx serves the unbuilt source directly, it is
not a production deployment.

## Commands
- Start: `docker compose -f docker-compose.base44.yml up -d`
- Logs: `docker compose -f docker-compose.base44.yml logs -f web`
- Status: `docker compose -f docker-compose.base44.yml ps`
- Stop: `docker compose -f docker-compose.base44.yml down`

## How to verify it works
`curl -s http://localhost:3000/ | grep -q 'PLAY REGULAR'` — the landing page
renders with its two buttons ("PLAY REGULAR" / "OPEN UNDETECTABLE"). The compose
healthcheck runs the same check against the page inside the container.
