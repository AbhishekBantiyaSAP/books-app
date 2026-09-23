# Saving Build Artifacts in GitHub Actions

Persisting build output (binaries, archives, packages) as a GitHub Actions artifact lets downstream jobs or humans download the exact file produced by the build — no rebuild needed.

## Why This Approach

Using `actions/upload-artifact` directly in the build job is the simplest option:

- No extra job, no custom scripts, no storage setup
- GitHub hosts and retains the file automatically

## Steps

### 1. Produce a file in a known path

Your build step must writes output of mbt build to mta_archives/*.mtar:

```yaml
- name: Build MTA
  run: mbt build
# Output lands in: mta_archives/*.mtar
```

### 2. Add the upload step immediately after

```yaml
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: <artifact-name>          # logical name used to download later
    path: <path/to/output>         # file, directory, or glob pattern
    if-no-files-found: error       # fail fast if build produced nothing
```

## Example

From `.github/workflows/build.yaml` in this repo:

```yaml
- name: Upload MTAR artifact
  uses: actions/upload-artifact@v4
  with:
    name: bookshop
    path: mta_archives/*.mtar
    if-no-files-found: error
```

`bookshop` is the artifact name.

## Key Parameters

| Parameter | Purpose |
|---|---|
| `name` | Identifier for the artifact within the workflow run |
| `path` | File, directory, or glob pointing to build output |
| `if-no-files-found` | `error` (fail), `warn`, or `ignore` — use `error` to surface broken builds immediately |
| `retention-days` | Optional. How long GitHub keeps the artifact (default: 90 days) |
