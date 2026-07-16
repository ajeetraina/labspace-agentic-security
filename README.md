# Securing the Agentic Stack — Workshop Lab Repo

Hands-on lab materials for the **Securing the Agentic Stack** workshop.

## Structure

```
workshop-agentic-security/
├── compose.yaml                     # Labspace runtime
├── compose.override.yaml
├── docker-scout-policy.yaml         # Scout build policies
├── .github/workflows/
│   └── secure-build.yml             # Complete CI pipeline
├── labspace/                        # Step-by-step lab guides
│   ├── labspace.yaml                # Labspace manifest
│   ├── 01-introduction.md
│   ├── 02-setup.md
│   ├── 03-lab-migrate-dhi.md        # Lab 1: Migrate to DHI
│   ├── 04-lab-sbom-attestations.md  # Lab 2: SBOM + signatures
│   ├── 05-lab-ci-policy.md          # Lab 3: GitHub Actions
│   ├── 06-lab-mcp-dhi.md            # Lab 4: MCP server on DHI
│   └── 07-conclusion.md
└── lab/
    ├── 01-migrate/Dockerfile        # Lab 1 starting point
    ├── 02-attest/Dockerfile         # Lab 2 starting point
    ├── 03-policy/Dockerfile         # Lab 3 — standard base (will fail)
    └── 04-mcp/
        ├── Dockerfile               # MCP server on DHI
        ├── docker-compose.yml       # Hardened runtime config
        ├── requirements.txt
        └── src/mcp_server.py        # Sample MCP server

```

## Prerequisites

- Docker Desktop 4.30+
- Docker Hub account (free)
- `notation` CLI installed (`brew install notation`)
- GitHub account (for Lab 3)
- Docker Scout enabled on your org

## Quick start

```bash
git clone https://github.com/docker/workshop-agentic-security
cd workshop-agentic-security
docker login
docker scout config organization YOUR_ORG
```

## Workshop deck

The companion slide deck is at [`Workshop Deck.dc.html`](../Workshop%20Deck.dc.html)
(29 slides, 4 live demo sections).

## Source material

Adapted from [`ajeetraina/labspace-container-security`](https://github.com/ajeetraina/labspace-container-security)
which covers 8 container security best practices using `catalog-service-node`.

## Resources

- [Docker Hardened Images](https://docs.docker.com/dhi/)
- [Docker Scout](https://docs.docker.com/scout/)
- [MCP Catalog on Docker Hub](https://hub.docker.com/mcp)
- [SLSA framework](https://slsa.dev)
- [Notation — image signing](https://notaryproject.dev)
- [State of Agentic AI Report](https://docker.com/resources/the-state-of-agentic-ai-white-paper)
