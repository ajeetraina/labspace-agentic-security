# Setup

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
