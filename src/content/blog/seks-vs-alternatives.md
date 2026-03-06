---
title: "SEK vs. Alternatives"
description: "How does credential isolation compare to env vars, config files, secrets managers, and OAuth? A side-by-side comparison."
date: 2026-02-10
author: "Síofra"
tags: ["comparison", "security", "architecture"]
---

## The Problem

Your AI agent needs to call APIs. You need to give it credentials somehow. What are your options?

## Option 1: Environment Variables

```bash
export OPENAI_API_KEY=sk-...
```

**Pros:**
- Simple
- Universal support

**Cons:**
- Agent can read and leak them
- Visible in process listings
- Persist in shell history
- Any prompt injection can extract them

**Verdict:** Fine for development. Dangerous for production.

## Option 2: Config Files

```yaml
# config.yaml
openai:
  api_key: sk-...
```

**Pros:**
- Easy to manage
- Can use file permissions

**Cons:**
- Agent can read the file
- Often accidentally committed to git
- Any file-read vulnerability exposes all secrets

**Verdict:** Slightly better than env vars. Still risky.

## Option 3: Secrets Manager + Runtime Fetch

```python
from aws import secrets_manager
key = secrets_manager.get("openai-key")
client = OpenAI(api_key=key)
```

**Pros:**
- Secrets stored securely
- Access logging
- Rotation support

**Cons:**
- Secret still enters agent memory
- Agent can still leak the fetched value
- Adds complexity

**Verdict:** Better infrastructure, same core problem.

## Option 4: OAuth Flows

```
User authorizes app → App gets access token → Agent uses token
```

**Pros:**
- User-scoped access
- Revocable
- Standard protocol

**Cons:**
- Complex to implement
- Token still visible to agent
- Not all APIs support OAuth

**Verdict:** Good for user-facing apps, not for agent credentials.

## Option 5: SEK Passthrough Proxy

```python
client = OpenAI(
    api_key="sek_token_abc123",  # Fake token
    base_url="https://broker.seks.ai/api/openai"
)
```

**Pros:**
- Agent never sees real credentials
- Works with standard SDKs
- Fake tokens are useless if leaked
- Centralized audit logging
- Policy enforcement at broker level

**Cons:**
- Additional network hop (minimal latency)
- Requires broker infrastructure

**Verdict:** The only option where credentials never enter agent memory.

## Comparison Table

| Approach | Credential in Agent Memory | Leak Risk | Audit Trail | Revocation |
|----------|---------------------------|-----------|-------------|------------|
| Env vars | ❌ Yes | 🔴 High | ❌ None | 🟡 Manual |
| Config files | ❌ Yes | 🔴 High | ❌ None | 🟡 Manual |
| Secrets manager | ❌ Yes (after fetch) | 🟠 Medium | ✅ Yes | 🟡 Manual |
| OAuth | ❌ Yes (token) | 🟠 Medium | 🟡 Partial | ✅ Yes |
| **SEK** | ✅ Never | 🟢 Low | ✅ Yes | ✅ Instant |

## When to Use What

**Use env vars:** Local development only.

**Use secrets manager:** For human-operated services.

**Use OAuth:** When the user is authenticating.

**Use SEK:** When an AI agent needs credentials and you don't trust it with the actual keys.

---

*Questions? Check the FAQ or reach out to the team.*
