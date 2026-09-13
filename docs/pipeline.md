# steam-monitor pipeline

Every 15 min CI polls the stable + beta win64 manifests. On a new build it
downloads the DLLs, re-resolves all hook signatures from scratch, and
publishes per-SHA TOMLs. Same entry schema as the legacy feed
(`[FNV-hash] / name / rva / sig`), so the client only needs a URL swap.

1. `tools/fetch_client.py` → `bins/<channel>/` (exit 2 = already published, skip)
2. `tools/resolve_patterns.py` → `pattern/<side>/<sha256>.toml` (any FAIL = nonzero exit)
3. Any FAIL → nothing published, an issue is filed, re-run via `workflow_dispatch`
4. On success: `README.md` is re-rendered (knap, do not hand-edit) and committed with `pattern/`

No TOMLs are shipped or fetched — every hook derives from the DLLs alone.
`tools/migrate_patterns.py` is a seed-era analysis aid, not in the pipeline.

## Local run

```sh
python3 -m unittest discover -s tests
python3 tools/fetch_client.py --out bins/ --channels stable beta
python3 tools/resolve_patterns.py resolve \
  --dll bins/stable/steamclient64.dll --side steamclient --out-dir pattern/
```
