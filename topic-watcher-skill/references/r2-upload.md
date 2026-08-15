# R2 Upload Reference

## Single Upload (Preferred: --md-path mode)

```bash
python3 /root/wiki/upload-to-r2.py \
  --md-path "<report.md>" \
  "<pdf_path>" "<slug>" "<topic_name>"
```

The `--md-path` flag tells the upload script to parse the Prediction section directly from the markdown file. This is the **preferred** approach — it avoids manual extraction errors with markdown bold markers (`**Probability:**`, `**Target:**`).

## Single Upload (Manual mode)

```bash
python3 /root/wiki/upload-to-r2.py \
  "<pdf_path>" "<slug>" "<topic_name>" "<prediction>" <probability> "<target_date>"
```

**WARNING:** When passing prediction/probability/target_date manually, you MUST strip `**` markdown bold markers from the values. The upload script does NOT strip them in manual mode. If you pass `** June 2027` as the target date, that's exactly what ends up in the manifest.

Uploads one PDF to `watch-reports/<slug>/<date>.pdf` and updates `watch-reports/manifest.json`.

## Batch Upload (Multiple PDFs)

There is **no** `upload-all-r2.py` script. When regenerating multiple PDFs (e.g., after a PDF script style change), write an inline script that:

1. Reads `/root/wiki/_config.yaml` using `yaml.safe_load` — maps topic paths to slugs and names
2. Walks `/root/wiki/` to find all `Watch Report *.md` files
3. Maps each MD to its topic via the parent directory → `_config.yaml` lookup
4. Generates each PDF using `/root/wiki/watch-report-to-pdf.py`
5. Calls `upload-to-r2.py` per PDF using `--md-path` mode with slug, name from `_config.yaml`

**Critical:** Folder names on disk (e.g., `sri-lanka-china`) differ from config slugs (e.g., `sri-lankan-financial-relationship-china`). Never derive slugs from folder names — always read from `_config.yaml`.

**Python for PDF/upload scripts:** `/usr/local/lib/hermes-agent/venv/bin/python`

**R2 public base URL:** `https://pub-70e08d62c8314675b40c42f0fe4be6fb.r2.dev`

Object keys: `watch-reports/<slug>/<YYYY-MM-DD>.pdf` (no leading slash).

## Manifest Structure

Nested `topics → reports`. Auto-migrates from old flat format.

## Common Issues

- **Slug mismatch:** Always read slugs from `_config.yaml`, never from folder names
- **Bold markers in manifest fields:** If you see `**` in probability or target_date in the manifest, the values were passed manually without stripping markdown. Use `--md-path` mode to avoid this.
- **Notes format:** The `watch-report-to-pdf.py` parser returns `notes` as a plain string. The `generate_pdf` function renders it directly as `<p>{notes}</p>`. If you see Python dict repr in the Notes section of a PDF, the parser and generator are out of sync.
- **Order:** Generate all PDFs first, then run uploads
- **R2_PUBLIC_BASE:** Hardcoded in `upload-to-r2.py` as `https://pub-70e08d62c8314675b40c42f0fe4be6fb.r2.dev` — no bucket name in path
