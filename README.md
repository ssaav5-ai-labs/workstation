# workstation

Base tooling for the lab: the LLM gateway and the base devcontainer for code repos.

## gateway

LiteLLM proxy in front of OpenRouter, with model aliases (ADR-0008):
`fast`, `code`, `reason`, `local`.

```powershell
cd gateway
Copy-Item .env.example .env   # then fill in OPENROUTER_API_KEY and LITELLM_MASTER_KEY
docker compose up -d
```

Gateway endpoint: `http://localhost:4000` from Windows, `http://host.docker.internal:4000` from containers.
All requests need the master key: `Authorization: Bearer $env:LITELLM_MASTER_KEY`.
