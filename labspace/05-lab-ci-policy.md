# Lab 3 — CI Policy Enforcement with GitHub Actions

> **Goal:** Wire Docker Scout into a GitHub Actions pipeline. Watch a build fail
> because of missing attestations and CVEs, then fix it with one Dockerfile change.

## Step 1 — Fork the repo and add secrets

Fork `github.com/docker/workshop-agentic-security` to your GitHub account.

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
