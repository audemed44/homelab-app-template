# Skeleton

Instructions for coding agents working in this repository. `CLAUDE.md`
imports this file.

## Project

TODO one line on what it does. A Go server (`cmd/skeleton`, `internal/`)
serves a JSON API and the Preact + TypeScript frontend (`frontend/`), built
into `web/dist` and embedded in the binary. State lives in SQLite at
`/data/skeleton.db` (`internal/store`).

- `internal/server`: routes (`server.go`), token sign-in and the
  same-origin guard (`auth.go`), the Foyer card (`foyer.go`).
- `internal/logging`: JSON logs for Logbook, the request middleware,
  `Job`/`QuietJob` for background work, `Audit` for what people do, and
  the limits that stop a failing app flooding the disk.
- `internal/store`: schema, migrations (append-only, tracked in
  `PRAGMA user_version`) and queries.
- `frontend/src`: `App.tsx` (session, shell, nav), `router.ts` (path
  routes), `api.ts` (one function per endpoint), `components/ui.tsx`
  (Dialog, Field, Figure, SectionHead, useAction), `styles.css`.

## Constraints

- **Low memory is a feature.** One static binary, `GOMEMLIMIT=32MiB`,
  `mem_limit: 64m` in compose. Keep history in SQLite, not in memory.
- Direct dependencies: modernc.org/sqlite (pure Go, so the build stays
  static and cgo-free). Justify any new one, Go or npm.
- Every `/api/` call needs `SKELETON_TOKEN` (bearer, or the session cookie
  derived from it), and state-changing browser requests from another origin
  are refused (`sameOrigin`). Never log or return secrets.
- `GET /api/foyer/widget` serves the card in Foyer's widget format
  (https://github.com/audemed44/foyer/blob/main/docs/app-widgets.md).
- `HOMEPAGE_URL` puts a link back to Foyer in the header.
- UI style is Foyer's: Swiss editorial, always dark (no light theme), heavy
  Inter headlines, tracked uppercase eyebrows, 2px rules over numbered
  headings, square corners, one accent (#2563ff). Check phone width too
  (390px): it's used from an iPhone.

## Logging

Use `internal/logging`, through `log/slog`:

- Inside a request or job, log with the `Context` variants
  (`slog.ErrorContext(r.Context(), …)`) so the line carries its trace.
- Wrap background work in `logging.Job` (or `QuietJob` when it runs every
  minute or so) and call `End(err, counts…)`.
- Call `logging.Audit(ctx, "thing.verb", ids…)` for anything a person
  changes: sign-ins, settings, deletes. Set `logging.SetUser` where the app
  knows who's signed in.
- Messages are short and stable (`"mail sync failed"`, not one with an id
  in it); details go in attributes. The limits count repeats per message.
- Ids, counts and names only. Never tokens, passwords or the content the
  app stores (mail bodies, amounts, document text).

## Commits

Conventional Commits: `<type>(<scope>): <summary>`, e.g. `feat(items): ...`.
CI rejects anything else, including the PR title.

## Checks before pushing

```sh
go vet ./... && go test -race ./...        # needs web/dist (npm run build)
cd frontend && npm run format:check && npm run typecheck && npm test && npm run build
docker build -t skeleton:dev .
```
