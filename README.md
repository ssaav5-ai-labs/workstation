# workstation

Base tooling for the lab: the LLM gateway and the base devcontainer for code repos.

## gateway

The **cluster gateway is primary**: it runs in k3s from
[`homelab-gitops/platform/llm-gateway/`](https://github.com/ssaav5-ai-labs/homelab-gitops/tree/main/platform/llm-gateway)
and answers at `http://llm.192.168.86.211.nip.io/v1`. Point `LLM_GATEWAY_URL` there.

This Compose gateway is the **fallback**, for when the cluster or a `k3s-*` node is down,
`llm.192.168.86.211.nip.io` does not answer, or nip.io is down. See
[`gateway/README.md`](gateway/README.md) for the switch steps.

LiteLLM proxy in front of OpenRouter, with model aliases (ADR-0008):
`fast`, `code`, `reason`, and `local` (fallback only; Ollama on the host).

```powershell
cd gateway
Copy-Item .env.example .env   # then fill in OPENROUTER_API_KEY and LITELLM_MASTER_KEY
docker compose up -d
```

Gateway endpoint: `http://localhost:4000` from Windows, `http://host.docker.internal:4000` from
containers. All requests need the master key: `Authorization: Bearer $env:LITELLM_MASTER_KEY`.
