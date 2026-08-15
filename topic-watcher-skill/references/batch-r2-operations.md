# Batch R2 Operations

Reusable patterns for checking and uploading watch report PDFs to Cloudflare R2 in batch.

## R2 Config

- Bucket: `signal-fracture-content`
- Prefix: `watch-reports/`
- Public base: `https://pub-70e08d62c8314675b40c42f0fe4be6fb.r2.dev`
- Object keys: `watch-reports/<r2_slug>/<YYYY-MM-DD>.pdf` (note: r2_slug comes from `_config.yaml`, not folder name!)
- Manifest: `watch-reports/manifest.json` (nested `topics → reports` structure)

## R2 Slug Mapping

Folder names on disk differ from R2 slugs. Always read `_config.yaml` to get the correct slug:

```python
import yaml
with open('/root/wiki/_config.yaml') as f:
    config = yaml.safe_load(f)
# slug_map: folder_name -> (r2_slug, topic_name)
slug_map = {t['path'].split('/')[-1]: (t['slug'], t['name']) for t in config['topics']}
```

## Pattern 1: Check Which PDFs Are Missing from R2

```python
import boto3, os, glob, yaml, json
from botocore.config import Config

s3 = boto3.client('s3', endpoint_url='https://9a79991ea25c968a06f52c4ecd949ff7.r2.cloudflarestorage.com',
    aws_access_key_id='56afe22c0c7a5e9ac25cdecd1f363b31',
    aws_secret_access_key='bc810ece1600642490e6312097c85487b11bcd6fffa6f09841a259a17a362ef0',
    config=Config(signature_version='s3v4'), region_name='auto')

# Get PDFs on R2
r2_keys = set()
resp = s3.list_objects_v2(Bucket='signal-fracture-content', Prefix='watch-reports/')
for obj in resp.get('Contents', []):
    if obj['Key'].endswith('.pdf'):
        r2_keys.add(obj['Key'])

# Load config for slug mapping
with open('/root/wiki/_config.yaml') as f:
    config = yaml.safe_load(f)
slug_map = {t['path'].split('/')[-1]: t['slug'] for t in config['topics']}

# Check disk PDFs against R2
for folder in slug_map:
    r2_slug = slug_map[folder]
    for pdf_path in glob.glob(f'/root/wiki/{folder}/Watch Reports/*.pdf'):
        fname = os.path.basename(pdf_path)
        date_match = re.search(r'(\d{2})-(\d{2})-(\d{4})', fname)
        if date_match:
            date_str = f"{date_match.group(3)}-{date_match.group(2)}-{date_match.group(1)}"
            expected_key = f"watch-reports/{r2_slug}/{date_str}.pdf"
            if expected_key not in r2_keys:
                print(f"MISSING: {pdf_path} -> {expected_key}")

print("\nDone checking.")
```

## Pattern 2: Batch Upload Missing PDFs

Read `_config.yaml` for slug/name, then for each topic folder call `upload-to-r2.py` with `--md-path`:

```python
import subprocess, os, glob, yaml

with open('/root/wiki/_config.yaml') as f:
    config = yaml.safe_load(f)

# folder_name -> (r2_slug, topic_name)
entries = {t['path'].split('/')[-1]: (t['slug'], t['name']) for t in config['topics']}

for folder, (r2_slug, topic_name) in entries.items():
    for md_path in sorted(glob.glob(f'/root/wiki/{folder}/Watch Reports/*.md')):
        pdf_path = md_path.replace('.md', '.pdf')
        if not os.path.exists(pdf_path):
            continue
        date_str = os.path.basename(pdf_path).replace('Watch Report ', '').replace('.pdf', '')
        expected_key = f"watch-reports/{r2_slug}/{date_str.replace('-', '/')}.pdf"
        
        cmd = ['python3', '/root/wiki/upload-to-r2.py',
               '--md-path', md_path, pdf_path, r2_slug, topic_name]
        result = subprocess.run(cmd, capture_output=True, text=True)
        print(f"{'OK' if result.returncode==0 else 'FAIL'}: {folder}/{os.path.basename(pdf_path)}")
```

## Pattern 3: Single Upload (Preferred Per-Report)

```bash
python3 /root/wiki/upload-to-r2.py \
  --md-path "/root/wiki/<folder>/Watch Reports/Watch Report DD-MM-YYYY.md" \
  "/root/wiki/<folder>/Watch Reports/Watch Report DD-MM-YYYY.pdf" \
  "<r2_slug>" "<Topic Name>"
```

The `--md-path` flag parses prediction/probability/target from the markdown automatically, avoiding `**` marker problems.

## Pattern 4: Fix Manifest — Merge Duplicate Slugs and Add Status Field

After batch uploads or when discovering slug contamination (e.g., a `kazakhstan` slug instead of `kazakhstan-economy`), or when the `"status"` field is missing from reports, run this fix:

```python
import boto3, json
from botocore.config import Config
from datetime import datetime

s3 = boto3.client('s3', endpoint_url='https://9a79991ea25c968a06f52c4ecd949ff7.r2.cloudflarestorage.com',
    aws_access_key_id='56afe22c0c7a5e9ac25cdecd1f363b31',
    aws_secret_access_key='bc810ece1600642490e6312097c85487b11bcd6fffa6f09841a259a17a362ef0',
    config=Config(signature_version='s3v4'), region_name='auto')

BUCKET = 'signal-fracture-content'
MANIFEST_KEY = 'watch-reports/manifest.json'

s3.download_file(BUCKET, MANIFEST_KEY, '/tmp/manifest.json')
with open('/tmp/manifest.json') as f:
    manifest = json.load(f)

# === Step 1: Merge duplicate slugs ===
# e.g., if "kazakhstan" and "kazakhstan-economy" both exist:
# slug_map = {"kazakhstan": "kazakhstan-economy"}  # bad -> good
slug_map = {}
for bad_slug, good_slug in slug_map.items():
    bad_entry = next((t for t in manifest['topics'] if t['slug'] == bad_slug), None)
    good_entry = next((t for t in manifest['topics'] if t['slug'] == good_slug), None)
    if bad_entry and good_entry:
        for r in bad_entry['reports']:
            if r['date'] not in [x['date'] for x in good_entry['reports']]:
                good_entry['reports'].append(r)
        manifest['topics'] = [t for t in manifest['topics'] if t['slug'] != bad_slug]
        # Also copy/delete the PDF on R2 (separate operation)

# === Step 2: Set Status on all reports ===
for topic in manifest['topics']:
    reports = topic.get('reports', [])
    if not reports:
        continue
    reports.sort(key=lambda r: r['date'], reverse=True)
    latest_pred = reports[0].get('prediction', '')
    for r in reports:
        r['status'] = 'Active' if r.get('prediction', '') == latest_pred else 'Inactive'

# Sort and upload
manifest['topics'].sort(key=lambda t: t['slug'])
manifest['updated_at'] = datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%SZ")

with open('/tmp/manifest.json', 'w') as f:
    json.dump(manifest, f, indent=2)

s3.upload_file('/tmp/manifest.json', BUCKET, MANIFEST_KEY, ExtraArgs={'ContentType': 'application/json'})
print(f"Manifest fixed: {len(manifest['topics'])} topics")
for t in manifest['topics']:
    for r in t['reports']:
        print(f"  {r['status']:8s} | {t['slug']:40s} | {r['date']} | {r.get('prediction','')[:50]}")
```

## Key Gotchas

- **R2 slug ≠ folder name.** `sri-lanka-china` folder maps to `sri-lankan-financial-relationship-china` slug. Always parse from `_config.yaml`.
- **Python to use.** `python3` has boto3 available on this system. `/tmp/pdfenv/bin/python3` does NOT have boto3.
- **Manifest auto-updates.** Each upload updates `manifest.json` in R2 — no separate manifest-rebuild step needed. BUT the Status field is NOT set by upload-to-r2.py — run Pattern 4 after any upload batch.
- **Date format.** The upload script extracts the date from the PDF filename (DD-MM-YYYY → YYYY-MM-DD for the object key). Ensure filenames follow the `Watch Report DD-MM-YYYY.pdf` convention.
- **Status field logic.** A report is Active if its prediction text matches the most recent report (by date) for the same slug. When the prediction changes, the previous latest becomes Inactive. Predictions that stay the same across reports keep all reports Active.