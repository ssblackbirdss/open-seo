# OpenSEO (custom model endpoint fork)

> Open source alternative to Semrush and Ahrefs, with support for any OpenAI-compatible model endpoint such as [freellmapi](https://github.com/tashfeenahmed/freellmapi).

## About this fork

This is a fork of [every-app/open-seo](https://github.com/every-app/open-seo). It adds one thing: an `OPENROUTER_BASE_URL` setting that lets OpenSEO's **in-app AI agent** talk to any OpenAI-compatible endpoint (a local proxy, a self-hosted gateway, and so on) instead of only OpenRouter. When the variable is not set, behavior is identical to upstream.

Everything else is upstream's work. Upstream's hosted version, pricing and community links live in the original repository.

> If you use OpenSEO through its MCP server from your own agent (Claude Code, OpenCode, and so on), you do **not** need this fork's feature. Your agent brings its own model.

<img width="100%" alt="openseo-keyword-research" src="https://github.com/user-attachments/assets/8ebdc439-3e72-41ab-8bde-8cda771ef2e8" />

## What OpenSEO does

- Keyword research
- Rank tracking
- Competitor insights
- Backlinks
- Site audits
- AI visibility

SEO data comes from [DataForSEO](https://dataforseo.com/). You bring your own API key and pay them directly for what you use.

## MCP and agent skills

OpenSEO exposes an MCP server so AI agents can use your SEO data directly, and ships Agent Skills that guide an agent through common SEO tasks. See upstream's docs:

- [Set up OpenSEO MCP](https://openseo.so/docs/mcp)
- [Set up OpenSEO Agent Skills](https://openseo.so/docs/skills/setup)

## Self-hosting

Upstream documents two self-hosting paths:

- **Docker** (personal use on your own machine): [`docs/SELF_HOSTING_DOCKER.md`](./docs/SELF_HOSTING_DOCKER.md)
- **Cloudflare** (internet-facing, multiple devices or a team): [`docs/SELF_HOSTING_CLOUDFLARE.md`](./docs/SELF_HOSTING_CLOUDFLARE.md)

Either way you need a DataForSEO API key: [`docs/DATAFORSEO_API_KEY.md`](./docs/DATAFORSEO_API_KEY.md).

**This fork's custom endpoint feature is intended for the Docker path.** On Cloudflare, a proxy running on your own machine is not reachable.

## Using a custom model endpoint (Docker)

### 1. Clone and create your `.env`

```sh
git clone https://github.com/ssblackbirdss/open-seo.git
cd open-seo
```

Create your env file:

```sh
# macOS / Linux
cp .env.example .env

# Windows (cmd)
copy .env.example .env
```

### 2. Add these lines to `.env`

Keep the variable names exactly as written (the `OPENROUTER_` prefix is intentional, it is what the app reads). Only change the values.

```env
# Required for SEO data
DATAFORSEO_API_KEY=your-dataforseo-key

# Your endpoint's API key (for example your freellmapi key)
OPENROUTER_API_KEY=your-endpoint-api-key

# Your endpoint's address. Change the port to match your proxy.
OPENROUTER_BASE_URL=http://host.docker.internal:31415/v1

# A model name your endpoint lists at /v1/models
OPENROUTER_MODEL=auto

# Must be different from your proxy's port
PORT=3002

# Name of the image you build in the next step
OPEN_SEO_IMAGE=open-seo-freellm
```

### 3. Build and run

```sh
docker build -f deploy/docker/Dockerfile -t open-seo-freellm .
docker compose up -d --force-recreate
```

The first start builds the app inside the container and can take a minute or two. Follow progress with `docker compose logs -f`. After any change to `.env`, run the `docker compose up -d --force-recreate` command again.

### Notes

- **Only the in-app agent uses these variables.** SEO data features need `DATAFORSEO_API_KEY` regardless.
- **`host.docker.internal`** works out of the box on Docker Desktop (Windows and macOS). On Linux, add this to the `open-seo` service in `compose.yaml`:
  ```yaml
  extra_hosts:
    - "host.docker.internal:host-gateway"
  ```
- **Your endpoint must be reachable from inside the container.** A proxy bound only to `127.0.0.1` on the host may not be.
- **Port conflicts:** OpenSEO defaults to port 3001, and many local proxies use it too. That is why the example sets `PORT=3002`.
- **Model names** differ between endpoints. Use one your endpoint actually lists.
- **With a custom endpoint**, OpenRouter-specific request options (usage accounting and reasoning settings) are not sent, so cost tracking in the app shows 0.
- **Model quality matters.** The agent calls many tools and handles long contexts. Small or heavily rate-limited free models can be unreliable at this.

### Troubleshooting

| Symptom | Likely cause and fix |
| --- | --- |
| Container keeps restarting with `set: Illegal option -` | The entrypoint script has Windows (CRLF) line endings. Make sure the repo contains a `.gitattributes` with `*.sh text eol=lf`, re-clone, and rebuild the image. |
| Agent errors or returns nothing | Check that the endpoint is running, the port in `OPENROUTER_BASE_URL` is right, and the model name exists at `/v1/models`. |
| Requests still go to OpenRouter | The image was built before the custom endpoint code was included. Rebuild with the `docker build` command above. |
| App not reachable on the expected port | Check `PORT` in `.env` and recreate the container. |

## Costs

OpenSEO needs a [DataForSEO](https://dataforseo.com/) API key for SEO data. When self-hosting, you pay DataForSEO directly. Keyword research is cheap; rank tracking costs grow with how many results pages you track and how often.

## Local development

See [`docs/LOCAL_DEVELOPMENT.md`](./docs/LOCAL_DEVELOPMENT.md).

## Contributing

Issues and pull requests for the custom endpoint feature are welcome here. For anything else about OpenSEO itself, see upstream and [`docs/CONTRIBUTING.md`](./docs/CONTRIBUTING.md).

## License

MIT, same as upstream. See [`LICENSE`](./LICENSE). Original work is copyright its upstream author.