# Prerequisites & Setup

> **Lab 1 (15–20 min).** Get Docker Desktop, Scout, Model Runner, and the MCP
> Toolkit ready, then launch the labspace.

## Prerequisite #1 — Install Docker Desktop

Install (or update to) Docker Desktop 4.30+.

👉 https://www.docker.com/products/docker-desktop/

## Prerequisite #2 — Enable Scout Image Analysis

In **Docker Desktop → Settings → Features in development** (or **Docker Scout**),
enable **Image Analysis** so Scout can inspect images locally.

## Prerequisite #3 — Enable background Scout SBOM indexing & Model Runner

- Enable **background Scout SBOM indexing** so SBOMs are generated as you build.
- Enable **Docker Model Runner**.
- Enable the **Docker MCP Toolkit** (used in Lab 4).

---

## Set up the labspace

A labspace is a fully-packaged playground environment for labs, workshops, and
trainings. It runs locally.

**Install Git**, then clone this repo and start local development mode:

```bash terminal-id=main
git clone https://github.com/ajeetraina/labspace-agentic-security
cd labspace-agentic-security
bash start-labspace.sh
```

Then **access the lab via port 3030**:

👉 http://localhost:3030

The full workshop is also hosted at **https://dockerworkshop.vercel.app/**.

> Labspace infra: https://github.com/dockersamples/labspace-infra

---

## Configure your Docker org & registry

## 1. Docker org setup

::variableDefinition[org]{prompt="What is your Docker Organization?"}

## 2. DHI tier selection

::variableSetButton[Use the free tier (dhi.io)]{variables="tier=free,dhiPrefix=dhi.io/"}

::variableSetButton[Use the paid tier ($$org$$)]{variables="tier=paid,dhiPrefix=$$org$$/dhi-"}

> **Free tier:** images at `dhi.io` — no Docker Hub subscription required.
> **Paid tier:** images mirrored into your org on Docker Hub.

## 3. Docker login

```bash terminal-id=main
docker login
```

:::conditionalDisplay{variable="tier" requiredValue="free"}
Also log in to the `dhi.io` registry:

```bash terminal-id=main
docker login dhi.io
```
:::

## 4. Configure Docker Scout

```bash terminal-id=main
docker scout config organization $$org$$
```

## 5. Clone and bootstrap the sample app

```bash terminal-id=main
git clone https://github.com/dockersamples/catalog-service-node
cd catalog-service-node
```

Pre-build the baseline image so Scout has something to scan:

```bash terminal-id=build
cd catalog-service-node
docker build -t catalog-service:baseline --sbom=true --provenance=mode=max .
```

## 6. Verify Notation is installed

```bash terminal-id=main
notation version
```

If not installed:
```bash terminal-id=main
# macOS
brew install notation

# Linux
curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/latest/download/notation_linux_amd64.tar.gz
tar xzf notation.tar.gz && sudo mv notation /usr/local/bin/
```

## 7. Check prerequisites

```bash terminal-id=main
docker --version       # Docker Desktop 4.30+ recommended
docker scout version   # Scout CLI
notation version       # Notation
git --version
```

All four commands should return a version. You are ready.
