# Securing the Agentic Stack — Workshop Lab Repo

Hands-on lab materials for the **Securing the Agentic Stack** workshop.

## Structure

```
labspace-agentic-security/
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
- `cosign` CLI installed (`brew install cosign`)
- GitHub account (for Lab 3)
- Docker Scout enabled on your org

## Quick start

```bash
git clone https://github.com/ajeetraina/labspace-agentic-security
cd labspace-agentic-security
bash start-labspace.sh          # then open http://localhost:3030
```

Or run it manually:

```bash
docker login
docker scout config organization YOUR_ORG
```

The complete workshop is also hosted at **https://dockerworkshop.vercel.app/**.

## Workshop deck

The companion slide deck is **"Securing the Agentic Stack: Docker Hardened Images
and Supply Chain Security"** by Ajeet Raina. The six-part agenda maps directly to
the labspace sections:

| Deck section | Labspace |
|--------------|----------|
| 01 · Why supply chain security matters now | Introduction |
| 02 · Building blocks (SBOM · VEX · SLSA) | Lab 2 |
| 03 · Standard image vs DHI | Lab 1 |
| 04 · Securing your CI pipeline | Lab 3 |
| 05 · Securing the agentic stack | Lab 4 |
| 06 · Wrap up | Conclusion |

## Source material

Adapted from [`ajeetraina/labspace-container-security`](https://github.com/ajeetraina/labspace-container-security)
which covers 8 container security best practices using `catalog-service-node`.

## Resources

- [Docker Hardened Images](https://docs.docker.com/dhi/)
- [Docker Scout](https://docs.docker.com/scout/)
- [MCP Catalog on Docker Hub](https://hub.docker.com/mcp)
- [SLSA framework](https://slsa.dev)
- [Cosign — image signing](https://docs.sigstore.dev/cosign/)
- [State of Agentic AI Report](https://docker.com/resources/the-state-of-agentic-ai-white-paper)
