# Lab 3 — Securing Your CI Pipeline

> **Goal:** Wire Docker Scout into a GitHub Actions pipeline. Watch a build fail
> because of missing attestations and CVEs, then fix it with one Dockerfile change.

## Docker Scout build policies — security as code

Define rules that **automatically fail the build** before anything insecure reaches
your registry or production.

**Policies to enforce today:**

- ✅ Block images with critical CVEs
- ✅ Require a valid SBOM attestation
- ✅ Require SLSA provenance
- ✅ Block unsigned images from promotion

> Only signed, attested images reach production.

```yaml no-copy-button
# docker-scout-policy.yaml
version: "1"
policies:
  - name: no-critical-cves
    type: vulnerability
    severity: critical
    action: fail
  - name: require-sbom
    type: attestation
    attestation: sbom
    action: fail
  - name: require-provenance
    type: attestation
    attestation: slsa-provenance
    action: fail
```

## Image signing with Notation

1. Build the image **with attestations**
2. **Sign** with Notation after push
3. Signature is stored as an **OCI referrer**
4. **Verify at deploy** — CI gate or admission controller

```bash no-copy-button
# Sign the image after push
notation sign myorg/myapp:v1.0

# Verify before deploy
notation verify myorg/myapp:v1.0
```

Works with Docker Hub · AWS ECR · Azure ACR · GitHub GHCR · any OCI registry
supporting referrers.

## The complete secure CI pipeline

```text no-copy-button
1. CHECKOUT      2. BUILD (DHI)     3. ATTEST         4. POLICY        5. SIGN          6. PUSH
actions/         FROM docker/       --sbom=true       docker/scout     notation sign    docker/build-
checkout@v4      hardened-node:20   --provenance      -action@v1       myorg/myapp      push-action@v6
                                    =mode=max         compare          :v1.0
```

You'll build exactly this pipeline below and watch the **POLICY** gate do its job.

---

## Step 1 — Fork the repo and add secrets

Fork `github.com/ajeetraina/labspace-agentic-security` to your GitHub account.

Add these secrets in **Settings → Secrets and variables → Actions**:

| Secret | Value |
|--------|-------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token (read/write) |

## Step 2 — Review the pipeline file

Open :fileLink[.github/workflows/secure-build.yml]{path=".github/workflows/secure-build.yml"}.

The pipeline has six jobs in sequence:

```yaml no-copy-button
checkout → build (DHI base) → attest → scout-policy → sign → push
```

The Scout gate step:

```yaml no-copy-button
- name: Docker Scout policy check
  uses: docker/scout-action@v1
  with:
    command: compare
    image: ${{ steps.build.outputs.imageid }}
    to-env: production
    ignore-unchanged: true
    only-severities: critical,high
    exit-code: true        # ← fails the build if policy not met
```

`exit-code: true` turns Scout into a hard gate — the pipeline stops and no image
reaches the registry if policies fail.

## Step 3 — Trigger with a non-DHI base (will fail)

Edit `lab/03-policy/Dockerfile` — comment out the DHI base and uncomment the standard one:

```dockerfile no-copy-button
# Round 1: standard base — will fail Scout gate
FROM node:20

WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

```bash terminal-id=main
git add lab/03-policy/Dockerfile
git commit -m "test: standard base image — should fail"
git push
```

Watch the **Actions** tab in GitHub. The pipeline fails at the `scout-policy` step:

```none no-copy-button
✗  No fixable critical or high vulnerabilities    (2C 26H found)
✗  Supply chain attestations                      (SBOM missing)

Error: scout compare failed — policy not met
```

## Step 4 — Fix it: switch to DHI (will pass)

Edit `lab/03-policy/Dockerfile` again:

```yaml save-as=lab/03-policy/Dockerfile
###########################################################
# Stage: build
###########################################################
FROM $$dhiPrefix$$node:24-debian13-dev AS build

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --production --ignore-scripts && npm cache clean --force

###########################################################
# Stage: final — distroless runtime
###########################################################
FROM $$dhiPrefix$$node:24-debian13 AS final

ENV NODE_ENV=production
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY ./src ./src
EXPOSE 3000
CMD ["node", "src/index.js"]
```

```bash terminal-id=main
git add lab/03-policy/Dockerfile
git commit -m "fix: switch to Docker Hardened Images"
git push
```

Watch Actions again. All six steps go green:

```none no-copy-button
✓  No fixable critical or high vulnerabilities
✓  Supply chain attestations
✓  No unapproved base images
✓  Default non-root user

Image signed with Notation ✓
Pushed to registry ✓
```

## Step 5 — Verify the pushed image locally

```bash terminal-id=build
docker pull $$org$$/catalog-service:latest
docker scout quickview $$org$$/catalog-service:latest --org $$org$$
notation verify $$org$$/catalog-service:latest
```

All three commands confirm: zero CVEs, full attestations, valid signature.

## What the full workflow YAML looks like

```yaml no-copy-button
name: Secure Build

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build with SBOM and provenance
        id: build
        uses: docker/build-push-action@v6
        with:
          context: lab/03-policy
          push: false
          load: true
          sbom: true
          provenance: mode=max
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/catalog-service:${{ github.sha }}

      - name: Docker Scout policy check
        uses: docker/scout-action@v1
        with:
          command: compare
          image: ${{ steps.build.outputs.imageid }}
          to-env: production
          ignore-unchanged: true
          only-severities: critical,high
          exit-code: true

      - name: Sign with Notation
        run: |
          notation sign \
            ${{ secrets.DOCKERHUB_USERNAME }}/catalog-service:${{ github.sha }}

      - name: Push signed image
        uses: docker/build-push-action@v6
        with:
          context: lab/03-policy
          push: true
          sbom: true
          provenance: mode=max
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/catalog-service:latest
```
