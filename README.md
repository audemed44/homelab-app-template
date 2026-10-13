# Skeleton

> The homelab app template: see [TEMPLATE.md](TEMPLATE.md).

TODO one line on what it does.

One small Go binary with the web UI built in, data in SQLite.

## Run it

```yaml
services:
  skeleton:
    image: ghcr.io/audemed44/skeleton:latest
    restart: unless-stopped
    user: "1000:1000"
    environment:
      - SKELETON_TOKEN=${SKELETON_TOKEN} # openssl rand -hex 32
    volumes:
      - ./skeleton:/data
    ports:
      - "8089:8080"
```

See [docker-compose.example.yml](docker-compose.example.yml) for every option.

| Variable | Default | |
|---|---|---|
| `SKELETON_TOKEN` | (required) | What you sign in with; also Foyer's widget key |
| `SKELETON_DATA_DIR` | `/data` | Where the database lives |
| `SKELETON_PORT` | `8080` | Port inside the container |
| `HOMEPAGE_URL` | | Foyer's address, linked from the header |
| `LOG_LEVEL` | `info` | `debug`, `info`, `warn` or `error` |
| `LOG_FORMAT` | `json` | `text` for reading in a terminal |

## Logs

One JSON object per line on stderr, for
[Logbook](https://github.com/audemed44/logbook) to collect: every request
(failures as errors, changes at info, plain reads at debug), background
jobs with how long they took, and an audit line for what people do. A
message repeated more than 20 times a minute is held back and counted, at
most 600 lines a minute get out, and long values are cut at 4 KB, so a
failing app can't flood the disk. Docker's own copy is capped in compose
(`logging:`, 3 × 10 MB).

## Foyer

Skeleton serves a [Foyer](https://github.com/audemed44/foyer) card at
`/api/foyer/widget`:

```yaml
      - name: Skeleton
        url: https://skeleton.example.com
        container: skeleton
        widget:
          type: app
          url: http://skeleton:8080/api/foyer/widget
          key: ${SKELETON_TOKEN}
```

## Development

```sh
cd frontend && npm install && npm run build && cd ..
SKELETON_TOKEN=dev SKELETON_DATA_DIR=./data go run ./cmd/skeleton
# or, with hot reload: run the binary, then `npm run dev` in frontend/
```
