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
pi
/setup-vertexai your-project-id eu
/model anthropic-vertex/claude-sonnet-5
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
- `gcloud` CLI installed for Google Application Default Credentials.

Run the Pi setup command after installing the package:

```text
/setup-vertexai <google-cloud-project> [location]
```

Example:

```text
/setup-vertexai my-project-id eu
```

The command stores the project and location in Pi's user config directory and checks
Google Application Default Credentials. If credentials are missing in interactive mode,
it can run:

```bash
gcloud auth application-default login
```

Environment variables override the saved setup:

| Name | Description | Default |
|------|-------------|---------|
| `GOOGLE_CLOUD_PROJECT` | Google Cloud project ID used by Vertex AI. | Saved setup value |
| `GCLOUD_PROJECT` | Fallback project ID if `GOOGLE_CLOUD_PROJECT` is not set. | Optional |
| `GOOGLE_CLOUD_LOCATION` | Vertex AI region or multi-region. Use `eu` for the EU multi-region endpoint. | Saved setup value or `eu` |
| `CLOUD_ML_REGION` | Fallback region if `GOOGLE_CLOUD_LOCATION` is not set. | Optional |

## Run

Try the package without adding it to your settings:

```bash
pi -e npm:@nilskluewer/pi-anthropic-vertex-enterprise-agent-platform
```

Use one of the supported Claude models:

```bash
pi --provider anthropic-vertex --model claude-fable-5
pi --provider anthropic-vertex --model claude-sonnet-5
pi --provider anthropic-vertex --model claude-opus-4.8
```

The extension reuses model metadata from Pi's built-in Anthropic provider, but only
exposes the Claude models verified for this package.

For the `eu` and `us` multi-regions, Vertex AI uses `aiplatform.<region>.rep.googleapis.com`
endpoints. The package defaults to `eu`; set another location in `/setup-vertexai` or
`GOOGLE_CLOUD_LOCATION` when you need a specific supported region such as `us-east5`.

## License

MIT
