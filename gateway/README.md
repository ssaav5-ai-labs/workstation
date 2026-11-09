# LLM gateway (fallback)

The **cluster gateway is primary**: it runs LiteLLM in k3s from
`homelab-gitops/platform/llm-gateway/` and answers at
`http://llm.192.168.86.210.nip.io/v1`. Every container should point `LLM_GATEWAY_URL` there.

This Compose gateway is the **fallback**. Use it when:

- the cluster or a `k3s-*` node is down,
- `llm.192.168.86.210.nip.io` does not answer, or
- nip.io is down.

## Start the fallback

From PowerShell:

```powershell
cd gateway
Copy-Item .env.example .env   # fill OPENROUTER_API_KEY and LITELLM_MASTER_KEY
docker compose up -d
```

Then point containers at it and recreate them:

```powershell
setx LLM_GATEWAY_URL "http://host.docker.internal:4000"
```

Close PowerShell and open a new window (`setx` only affects new windows), then recreate the container:

```powershell
devpod up "$env:USERPROFILE\code\github.com\ssaav5-ai-labs\homelab-gitops" --recreate --ide none
```

## Check which gateway a container uses

```bash
echo $LLM_GATEWAY_URL
```

`http://host.docker.internal:4000` is the fallback; `http://llm.192.168.86.210.nip.io` is the cluster.

## Verify

```bash
curl -s -o /dev/null -w '%{http_code}\n' \
  -H "Authorization: Bearer $LLM_GATEWAY_KEY" \
  "$LLM_GATEWAY_URL/v1/models"
```

`200`, and the alias list (`fast`, `code`, `reason`), means the gateway is serving.

## Go back to the cluster

```powershell
setx LLM_GATEWAY_URL "http://llm.192.168.86.210.nip.io"
```

Close and reopen PowerShell, recreate the container, and verify again. Then stop the fallback:

```powershell
docker stop llm-gateway
```

`docker start llm-gateway` brings it back. Other containers (`handbook`, `homelab-infra`) pick up the
new URL on their next `devpod up ... --recreate`.

## Config

The primary config is `homelab-gitops/platform/llm-gateway/config.yaml`. Keep the `fast`, `code` and
`reason` aliases identical here and there. `local` is fallback-only (Ollama on the host); the cluster
has no Ollama. The image is pinned (`v1.106.6` in both the cluster and this Compose file); bump
deliberately, it holds every key.
