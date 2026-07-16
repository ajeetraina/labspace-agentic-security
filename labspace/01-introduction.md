# Introduction

👋 Welcome to **Securing the Agentic Stack**!

This workshop walks you through supply chain security fundamentals and applies
them to real workloads — both a traditional Node.js service and a Python MCP server
that an AI agent calls at runtime.

---

## Why this matters now

Agentic AI systems pull dependencies, invoke tools, and build environments
**autonomously** — without a human reviewing each step. One compromised base image
or MCP server can give an attacker access to production credentials, filesystems,
and external APIs with no human in the loop.

> *"The better the agent, the bigger the blast radius."*

**Docker's own data (State of Agentic AI Report, 800+ respondents):**

| Stat | Figure |
|------|--------|
| Organizations with AI agents in production | 60% |
| Cite security as #1 challenge scaling agentic AI | 40% |
| Familiar with MCP but can't deploy it securely | 85% |
| Use containers for agent dev or production | 94% |

---

## The three standards you need to know

| Standard | What it solves |
|----------|---------------|
| **SBOM** | Know every component in every image |
| **VEX** | Cut CVE noise — know which ones actually affect you |
| **SLSA** | Prove an artifact's provenance wasn't tampered with |

Docker Hardened Images ship all three, plus a digital signature, out of the box.

---

## What you will do in this lab

| Lab | Task |
|-----|------|
| **Lab 1** | Migrate `catalog-service-node` from `node:20` to DHI — see 0 CVEs |
| **Lab 2** | Generate an SBOM, inspect attestations, verify the digital signature |
| **Lab 3** | Wire Docker Scout into GitHub Actions — watch a build fail, then pass |
| **Lab 4** | Build a Python MCP server on `hardened-python`, run it hardened, verify it |

> **Key numbers you will reproduce:**
> - `node:20` → ~193 CVEs, 700+ packages
> - DHI node runtime → **0 critical/high CVEs, ~12 packages**
> - Image size: **10× smaller**
