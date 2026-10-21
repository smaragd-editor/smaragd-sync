# smaragd-sync

Self-hostable, end-to-end-encrypted sync for [Smaragd](https://github.com/smaragd-editor/smaragd),
a native desktop authoring tool for writers.

- [`smaragd-sync-protocol/`](smaragd-sync-protocol) — wire types shared by the
  Smaragd desktop client and this server: ids, encrypted-envelope layout, the
  pairing ticket, HTTP API request/response types. Consumed by the main
  [smaragd](https://github.com/smaragd-editor/smaragd) repo as a git dependency.
- [`smaragd-sync-server/`](smaragd-sync-server) — the axum + SQLite server. See its
  own [self-hosting guide](smaragd-sync-server/README.md). Published as a Docker
  image: `ghcr.io/smaragd-editor/smaragd-sync-server`.

End-to-end tests against the real client live in a separate repo,
[smaragd-sync-e2e](https://github.com/smaragd-editor/smaragd-sync-e2e), since they
depend on both this repo and the main app.

## Development

```sh
cargo test --workspace --all-targets --all-features
docker build -f smaragd-sync-server/Dockerfile -t smaragd-sync-server .   # context: repo root
```

## License

GPL-3.0-or-later — see [LICENSE](LICENSE).
