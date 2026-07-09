# @nilskluewer/pi-anthropic-vertex-enterprise-agent-platform

[![npm](https://img.shields.io/npm/v/@nilskluewer/pi-anthropic-vertex-enterprise-agent-platform)](https://www.npmjs.com/package/@nilskluewer/pi-anthropic-vertex-enterprise-agent-platform)

## Overview

Use Anthropic Claude models through Google Cloud Vertex AI / Gemini Enterprise Agent
Platform in [pi](https://github.com/earendil-works/pi).

The extension creates an `AnthropicVertex` client and delegates streaming to Pi's
built-in `anthropic-messages` implementation, so Pi still handles tools, prompt
caching, thinking blocks, partial JSON streaming, usage tracking, and cost calculation.

## Getting Started

```bash
pi install npm:@nilskluewer/pi-anthropic-vertex-enterprise-agent-platform
gcloud auth application-default login
export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_LOCATION=eu
pi --provider anthropic-vertex --model claude-sonnet-4-6
```

## Structure

```text
.
├── extensions/
│   └── anthropic-vertex-enterprise-agent-platform.ts  # Pi extension entry point
├── LICENSE
├── README.md
└── package.json                                      # npm and Pi package manifest
```

## Setup

Requirements:

- Google Cloud project with the Vertex AI API enabled.
- Claude models enabled in Model Garden / Gemini Enterprise Agent Platform.
- Google Application Default Credentials configured locally.

Environment variables:

| Name | Description | Default |
|------|-------------|---------|
| `GOOGLE_CLOUD_PROJECT` | Google Cloud project ID used by Vertex AI. | Required |
| `GCLOUD_PROJECT` | Fallback project ID if `GOOGLE_CLOUD_PROJECT` is not set. | Optional |
| `GOOGLE_CLOUD_LOCATION` | Vertex AI region or multi-region. Use `eu` for the EU multi-region endpoint. | `eu` |
| `CLOUD_ML_REGION` | Fallback region if `GOOGLE_CLOUD_LOCATION` is not set. | Optional |

## Run

Try the package without adding it to your settings:

```bash
pi -e npm:@nilskluewer/pi-anthropic-vertex-enterprise-agent-platform
```

Use any Claude model available in your Pi installation:

```bash
pi --provider anthropic-vertex --model claude-sonnet-4-6
pi --provider anthropic-vertex --model claude-opus-4-6
```

The extension registers Claude model definitions from Pi's built-in Anthropic provider
at runtime, so newly supported Claude models are picked up when Pi updates.

For the `eu` and `us` multi-regions, Vertex AI uses `aiplatform.<region>.rep.googleapis.com`
endpoints. The package defaults to `eu`; set `GOOGLE_CLOUD_LOCATION` when you need a
specific supported region such as `us-east5`.

## License

MIT
