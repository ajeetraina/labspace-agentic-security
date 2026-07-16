# Conclusion & Next Steps

🎉 You have completed all four labs. Here is what you just did:

---

## What you built

| Lab | What you proved |
|-----|-----------------|
| **Lab 1** | One FROM change eliminates 190+ CVEs and 595 packages |
| **Lab 2** | SBOM, VEX, SLSA provenance, and digital signature are inspectable and verifiable |
| **Lab 3** | A CI pipeline that blocks any image not meeting all seven Scout policies |
| **Lab 4** | A fully attested, signed MCP server running with zero-write, zero-capability isolation |

---

## Your security framework

```
1. Know what is in your images          →  SBOM + VEX
2. Verify where they came from          →  SLSA provenance + image signing
3. Start from a trusted base            →  Docker Hardened Images
4. Enforce at the pipeline              →  Docker Scout build policies
5. Isolate your agents                  →  MCP servers in hardened containers
```

---

## Key commands cheat sheet

```bash
# Scan an image
docker scout quickview IMAGE --org YOUR_ORG
docker scout cves IMAGE --only-severity critical,high

# Compare two images
docker scout compare --ignore-unchanged --to BASELINE IMAGE

# Inspect attestations
docker scout attest list IMAGE
docker scout sbom IMAGE --format list

# Sign and verify
notation sign IMAGE
notation verify IMAGE

# Build with full attestations
docker buildx build --sbom=true --provenance=mode=max -t IMAGE --push .
```

---

## Resources

| Resource | URL |
|----------|-----|
| Docker Hardened Images | hub.docker.com → Trusted Content → Hardened Images |
| Docker Scout docs | docs.docker.com/scout |
| MCP Catalog | hub.docker.com/mcp |
| SLSA framework | slsa.dev |
| VEX specification | openvex.dev |
| Notation (image signing) | notaryproject.dev |
| MCP specification | modelcontextprotocol.io |
| State of Agentic AI Report | docker.com/resources/the-state-of-agentic-ai-white-paper |

---

> *Securing your containers is step one.*
> *Securing your supply chain is the rest.*
>
> [Get started with DHI →](https://docs.docker.com/dhi/get-started/)
