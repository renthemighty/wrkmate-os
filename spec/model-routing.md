# Model Routing Spec (draft)

This is a spec draft. The format below is a working proposal, not a finalized config schema. Expect field names and structure to change before implementation.

## Purpose

The activation engine is model agnostic. It does not call one large language model, it calls whichever provider is configured for the intent, the region, and the current fallback state. This document describes the config that drives that routing.

## Provider routing config

A routing config lists every available provider, a fallback order, and the device-side resolver's parameter budget.

```json
{
  "providers": [
    {
      "name": "claude",
      "endpoint": "https://api.anthropic.com/v1/messages",
      "region": "global",
      "priority": 1,
      "allowed_intents": ["compose_message", "tool_assembly", "general_query"]
    },
    {
      "name": "gpt",
      "endpoint": "https://api.openai.com/v1/responses",
      "region": "global",
      "priority": 2,
      "allowed_intents": ["compose_message", "tool_assembly", "general_query"]
    },
    {
      "name": "grok",
      "endpoint": "https://api.x.ai/v1/chat/completions",
      "region": "global",
      "priority": 3,
      "allowed_intents": ["general_query"]
    },
    {
      "name": "gemini",
      "endpoint": "https://generativelanguage.googleapis.com/v1/models",
      "region": "global",
      "priority": 4,
      "allowed_intents": ["compose_message", "tool_assembly", "general_query"]
    },
    {
      "name": "llama",
      "endpoint": "https://engine.internal/llama/v1/chat",
      "region": "self-hosted",
      "priority": 5,
      "allowed_intents": ["compose_message", "general_query"]
    },
    {
      "name": "deepseek",
      "endpoint": "https://api.deepseek.com/v1/chat/completions",
      "region": "cn",
      "priority": 1,
      "allowed_intents": ["compose_message", "tool_assembly", "general_query"]
    },
    {
      "name": "qwen",
      "endpoint": "https://dashscope.aliyuncs.com/api/v1/services/aigc/text-generation/generation",
      "region": "cn",
      "priority": 2,
      "allowed_intents": ["compose_message", "tool_assembly", "general_query"]
    },
    {
      "name": "kimi",
      "endpoint": "https://api.moonshot.cn/v1/chat/completions",
      "region": "cn",
      "priority": 3,
      "allowed_intents": ["compose_message", "general_query"]
    },
    {
      "name": "glm",
      "endpoint": "https://open.bigmodel.cn/api/paas/v4/chat/completions",
      "region": "cn",
      "priority": 4,
      "allowed_intents": ["compose_message", "general_query"]
    }
  ],
  "fallback_order": ["claude", "gpt", "gemini", "grok", "llama"],
  "device_model": {
    "parameter_budget": {
      "min": 20000,
      "max": 160000,
      "selection": "chosen_at_install_from_device_benchmark"
    }
  },
  "data_boundary": {
    "leaves_device": "intent_frames_only",
    "never_leaves_device": ["raw_audio", "full_transcript"],
    "region_pinning": "enabled"
  }
}
```

## Field notes

**`providers[].name`**: Provider identifier. Anything the engine has an adapter for, including Claude, GPT, Grok, Gemini, self-hosted Llama, and the Chinese providers DeepSeek, Qwen, Kimi, and GLM.

**`providers[].endpoint`**: The API endpoint the engine calls for that provider. Self-hosted models point at an internal address instead of a public API.

**`providers[].region`**: The jurisdiction the provider serves, or `global` if it serves all regions. A region value of `cn` groups the providers that route requests inside mainland China, so a region blocked from one global provider still resolves to a working provider in the same routing pass.

**`providers[].priority`**: Order of preference within a region. Lower number tried first.

**`providers[].allowed_intents`**: Which candidate actions this provider is permitted to handle. Not every provider needs to handle every intent. A provider can be scoped down to simple intents only, or opened up to full tool assembly.

**`fallback_order`**: The sequence the engine walks if the top-priority provider for a region fails, times out, or returns an error. This is a global default; a region can override it by relying on its own `priority` ordering among providers scoped to that region.

**`device_model.parameter_budget`**: Not a provider setting. This describes the resolver, not the engine. The resolver's parameter count is chosen once, at install time, from a benchmark of the device it is installed on. Range is 20,000 parameters at the low end up to 160,000 at the high end. This value never changes after install unless the app is reinstalled or the device benchmark is re-run.

**`data_boundary`**: What is allowed to leave the device and what is not. Raw audio and full transcripts stay on the device under all configurations. Only structured intent frames are sent to the activation engine, and `region_pinning` determines whether those frames can be routed to a provider outside the user's jurisdiction.

## Open questions

- Whether `allowed_intents` should support wildcard matching for providers that handle everything.
- Whether fallback should be per-region or a single global list, as drafted above.
- How the engine surfaces a provider outage to the shell without exposing which provider failed.
