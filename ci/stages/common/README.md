GitHub Actions reusable composite actions mirroring `ci/stages/common`.

Usage examples (in a workflow under `.github/workflows/*.yml`):

```yaml
name: CI Example
on: [push]
jobs:
  tools:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install tools
        uses: ./github/ci/stages/common/StageInstallTools

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and push image
        uses: ./github/ci/stages/common/StageDockerfileBuild
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          repository: myorg/myapp
          dockerfile: ./Dockerfile
          context: .
          tags: latest

  scans:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run repo scans
        uses: ./github/ci/stages/common/StageScansScript

  image-scans:
    runs-on: ubuntu-latest
    steps:
      - name: Scan built image
        uses: ./github/ci/stages/common/StageDockerfileScans
        with:
          registry: ghcr.io
          image: myorg/myapp
          tag: latest

  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate SBOM for workspace
        uses: ./github/ci/stages/common/StageScansSBOM

  dast:
    runs-on: ubuntu-latest
    steps:
      - name: ZAP Baseline
        uses: ./github/ci/stages/common/StageDASTScans
        with:
          target-url: https://example.com/

  consolidate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Consolidate SARIF
        uses: ./github/ci/stages/common/StageConsolidateSarif

  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: .NET publish and test
        uses: ./github/ci/stages/common/StageTests
```

Notes:
- Reusable workflows must live in `.github/workflows`. Here we provide composite actions so you can call them locally via `uses: ./github/...`.
- Each action may require additional secrets (e.g., registry credentials, `SONAR_TOKEN`, Dependency-Track API key). Supply them via workflow `with` and `secrets`.

