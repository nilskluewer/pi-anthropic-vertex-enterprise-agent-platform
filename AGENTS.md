# Agent Notes: Anthropic Vertex AI for Pi

This repo publishes the Pi extension
`@nilskluewer/pi-anthropic-vertex-enterprise-agent-platform`.

Use these notes when maintaining the extension. Keep the public package small and avoid
committing local plans, sync folders, credentials, logs, or generated dependency folders.

## Fast workflow

```bash
cd <repo>
npm install --omit=dev --registry=https://registry.npmjs.org/
pi -e . --list-models | rg 'anthropic-vertex'
pi -e . --provider anthropic-vertex --model claude-sonnet-5 --thinking off -p --no-session "Say exactly: OK"
rm -rf node_modules package-lock.json .DS_Store
npm pack --dry-run --registry=https://registry.npmjs.org/
```

Before publishing, the npm dry-run tarball should normally contain only:

```text
LICENSE
README.md
extensions/anthropic-vertex-enterprise-agent-platform.ts
package.json
```

`AGENTS.md` is intentionally not in `package.json.files`, so it is useful for repo
maintenance but not shipped in the npm package.

## Public package constraints

- Do not commit `node_modules/`, `package-lock.json`, `.DS_Store`, `.pi/`, `.vscode/`,
  sync folders, notes, local logs, or private plans.
- Do not commit real project IDs, internal org names, service-account JSON, access tokens,
  API keys, or local user paths in shipped files.
- Keep `.npmrc` and `publishConfig.registry` pointed at npmjs.org.
- The human runs `npm publish` because npm 2FA may require an OTP.

## Configuration model

The extension supports two setup paths:

1. Pi command:

   ```text
   /setup-vertexai <google-cloud-project> [location]
   ```

   This writes:

   ```text
   ~/.pi/agent/anthropic-vertex-enterprise-agent-platform.json
   ```

2. Environment variables, including Claude Code compatible names:

   ```text
   GOOGLE_CLOUD_PROJECT
   ANTHROPIC_VERTEX_PROJECT_ID
   GCLOUD_PROJECT
   GOOGLE_CLOUD_LOCATION
   CLOUD_ML_REGION
   VERTEX_REGION
   ```

Precedence for project ID:

```text
GOOGLE_CLOUD_PROJECT -> ANTHROPIC_VERTEX_PROJECT_ID -> GCLOUD_PROJECT -> saved setup
```

Precedence for region:

```text
GOOGLE_CLOUD_LOCATION -> CLOUD_ML_REGION -> VERTEX_REGION -> saved setup -> eu
```

When a saved setup or fallback env var is used, the extension also populates
`GOOGLE_CLOUD_PROJECT` and `GOOGLE_CLOUD_LOCATION` in-process so the Anthropic Vertex
SDK sees the same values.

## EU endpoint behavior

Our verified working region is the Vertex AI EU multi-region:

```text
location: eu
endpoint: https://aiplatform.eu.rep.googleapis.com
```

Multi-region hosts are special:

```text
eu     -> https://aiplatform.eu.rep.googleapis.com
us     -> https://aiplatform.us.rep.googleapis.com
global -> https://aiplatform.googleapis.com
other  -> https://<region>-aiplatform.googleapis.com
```

Do not regress to `https://eu-aiplatform.googleapis.com`; that host is wrong for the
`eu` multi-region and causes HTML 404 responses.

## Model exposure

The extension should expose Claude models from Pi's built-in Anthropic model list, not
hard-code only a tiny subset. Vertex AI availability still depends on the user's Google
Cloud project and enabled Model Garden / Gemini Enterprise Agent Platform models.

Models verified end-to-end with this extension in the EU multi-region:

```text
anthropic-vertex/claude-fable-5
anthropic-vertex/claude-sonnet-5
anthropic-vertex/claude-opus-4-8
```

Use hyphenated `claude-opus-4-8` for Pi and Vertex. A dotted `claude-opus-4.8` may be
accepted by custom-model fallback in some paths, but it should not be documented as the
canonical model ID for this package.

Mini-test commands:

```bash
pi -e . --provider anthropic-vertex --model claude-fable-5 --thinking off -p --no-session "Say exactly: OK_FABLE_5"
pi -e . --provider anthropic-vertex --model claude-sonnet-5 --thinking off -p --no-session "Say exactly: OK_SONNET_5"
pi -e . --provider anthropic-vertex --model claude-opus-4-8 --thinking off -p --no-session "Say exactly: OK_OPUS_4_8"
```

If Opus returns 429, treat it as quota, not as a model/endpoint regression.

## Implementation approach

Do not reimplement Anthropic streaming. The key design is:

- Register provider `anthropic-vertex`.
- Create an `AnthropicVertex` client from `@anthropic-ai/vertex-sdk`.
- Delegate to Pi's built-in `anthropic-messages` API provider via
  `getApiProvider("anthropic-messages").stream(...)`.
- Patch the delegated model's `api` to `anthropic-messages` before calling the built-in
  stream, because Pi's registry guard expects the delegated API to match.

This keeps Pi responsible for:

- message transformation
- prompt caching
- tool call normalization
- thinking block replay
- partial JSON streaming
- usage tracking and cost calculation

The extension only owns:

- setup command and config resolution
- Vertex region/base URL handling
- AnthropicVertex client construction
- SimpleStreamOptions to AnthropicOptions mapping needed for direct `stream(...)`

## Local validation checklist

Run this before committing changes:

```bash
cd <repo>
npm install --omit=dev --registry=https://registry.npmjs.org/
npm audit --omit=dev --registry=https://registry.npmjs.org/
pi -e . --list-models | rg 'anthropic-vertex.*claude-(fable-5|sonnet-5|opus-4-8)'
pi -e . --provider anthropic-vertex --model claude-sonnet-5 --thinking off -p --no-session "Say exactly: OK"
rm -rf node_modules package-lock.json .DS_Store
npm pack --dry-run --registry=https://registry.npmjs.org/
```

Security scan:

```bash
rg -n --hidden -S \
  '(api[_-]?key|secret|token|password|credential|private[_-]?key|GOOGLE_APPLICATION_CREDENTIALS|PROJECT_ID|VERTEX_PROJECT_ID|internal-project|internal-org|/Users/|AIza|ya29\.|ghp_|gho_|sk-[A-Za-z0-9]|-----BEGIN)' \
  . -g '!node_modules' -g '!package-lock.json' -g '!.git'
```

Benign hits usually include documentation strings mentioning credentials or env-var names.
Do not ignore actual values.
