# Lab 2 — SBOM Generation & Signature Verification

> **Goal:** Attach a full SBOM and provenance to your image, inspect all attestations,
> and verify the digital signature.

## Step 1 — Build with full attestations

```bash terminal-id=build
cd catalog-service-node
docker buildx build \
  --sbom=true \
  --provenance=mode=max \
  -t $$org$$/catalog-service:v1.0 \
  --push \
  .
```

`--sbom=true` generates a CycloneDX + SPDX bill of materials and attaches it to
the image manifest. `--provenance=mode=max` records the full build environment,
source inputs, and build parameters as a signed SLSA envelope.

## Step 2 — List all attestations

```bash terminal-id=build
docker scout attest list $$org$$/catalog-service:v1.0
```

| Attestation | What it is |
|-------------|------------|
| CycloneDX SBOM | Components, libraries, versions |
| SPDX SBOM | SBOM in SPDX format |
| Scout SBOM | SBOM generated and signed by Docker Scout |
| OpenVEX | Non-applicable CVEs with explanations |
| SLSA provenance | Source, build params, environment |
| SLSA verification summary | SLSA compliance level |

## Step 3 — View the SBOM

```bash terminal-id=build
docker scout sbom $$org$$/catalog-service:v1.0
```

Pipe to jq for a clean package list:

```bash terminal-id=build
docker scout sbom $$org$$/catalog-service:v1.0 \
  --format list
```

## Step 4 — Inspect DHI attestations (pre-built image)

DHI ships attestations that your own build inherits through the base image chain.
Inspect the upstream DHI node image directly:

```bash terminal-id=build
docker scout attest list $$dhiPrefix$$node:24-debian13
```

You will see entries including `OpenVEX`, `FIPS attestation`, `STIG attestation`,
and `Scout health` — attestations Docker generates and signs for every DHI release.

## Step 5 — View the SLSA provenance

```bash terminal-id=build
docker buildx imagetools inspect $$org$$/catalog-service:v1.0 \
  --format '{{json .Provenance}}'
```

Note the `builder.id`, `materials` (source repo + commit SHA), and
`invocation.parameters` fields. This is the non-falsifiable record of what
produced this artifact.

## Step 6 — Sign the image with Notation

```bash terminal-id=build
notation sign $$org$$/catalog-service:v1.0
```

```bash terminal-id=build
notation list $$org$$/catalog-service:v1.0
```

## Step 7 — Verify the signature

```bash terminal-id=build
notation verify $$org$$/catalog-service:v1.0
```

```none no-copy-button
Successfully verified signature for $$org$$/catalog-service:v1.0
```

## Step 8 — Export VEX for use with external scanners

DHI ships VEX documents that tell external scanners which CVEs are non-applicable.
This eliminates false positives in Trivy, Grype, Wiz, and Snyk automatically.

```bash terminal-id=build
docker scout vex get $$org$$/catalog-service:v1.0 --output vex.json
cat vex.json | head -40
```

Pass to Grype:
```bash no-copy-button
grype $$org$$/catalog-service:v1.0 --vex vex.json
```

Pass to Trivy:
```bash no-copy-button
trivy image --vex vex.json $$org$$/catalog-service:v1.0
```

> **Key insight:** VEX is the signal-to-noise solution. Without it, a scanner reports
> every CVE in every package even if the vulnerable code path is never reachable.
> With VEX, you see only what actually matters.
