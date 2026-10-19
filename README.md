# Deploy and Host Langflow on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/langflow-production?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=langflow)

[Langflow](https://www.langflow.org) is a visual builder for AI agents, RAG pipelines and MCP servers. You drag components onto a canvas (models from any provider, prompts, vector stores, tools, other agents), connect them, test them in the playground, and every flow becomes an API endpoint and an MCP tool. It is open source (MIT) and written in Python, so you can drop into code for any component.

## About Hosting Langflow

This template runs Langflow 1.12.6 with PostgreSQL on Railway, set up the way you'd run it for a team rather than on a laptop.

- **The login is on.** Langflow's own default is auto-login, which means no password: anyone who finds the URL is admin. Here auto-login is off, an `admin` account with a generated password is created on first boot, and public sign-up is closed.
- **Encryption that works.** The API keys and global variables you store in Langflow are encrypted with `LANGFLOW_SECRET_KEY`, generated in the 43-character format Langflow's encryption needs. Get that format wrong and storing a key fails with "Fernet key must be 32 url-safe base64-encoded bytes".
- **Pinned version.** The image is `langflowai/langflow:1.12.6`, not `latest`, so a redeploy never upgrades you by surprise. Upgrade by changing the tag (see below).
- **Nothing lost on redeploy.** Flows, users, messages and run history are in Postgres. Uploaded files and knowledge bases are on a Railway volume.

## Common Use Cases

- Agents and RAG chatbots over your own documents, published as an API or embedded chat widget
- MCP servers: every Langflow project is an MCP server, so Claude, Cursor and other MCP clients can call your flows as tools
- Prototyping and comparing LLM workflows across providers (OpenAI, Anthropic, Google, Mistral, Groq, Ollama and more) with a team

## Dependencies for Langflow Hosting

- PostgreSQL 17 (included), on its own volume
- An API key from the model provider you want to use, added inside Langflow

### Deployment Dependencies

- [Langflow documentation](https://docs.langflow.org)
- [Langflow on GitHub](https://github.com/langflow-ai/langflow)
- [Template source on GitHub](https://github.com/two-workspaces/langflow)

### Implementation Details

**Sign in.** Open the Langflow service's Railway domain and sign in as `admin` with the `LANGFLOW_SUPERUSER_PASSWORD` value from the service's Variables tab. You can change the password under Settings → General; the variable is only used to create the account on first boot.

**Add teammates.** Open-source Langflow has no user-management screen (that's in the Enterprise edition), so use sign-up for a few minutes:

1. On the Langflow service, set `LANGFLOW_ENABLE_SIGNUP=true` and add `LANGFLOW_NEW_USER_IS_ACTIVE=true`, then deploy.
2. Teammates register with the **Sign Up** link on the login page. Each person gets their own flows.
3. Set `LANGFLOW_ENABLE_SIGNUP` back to `false` and deploy. The Sign Up link stays visible, but registration is refused.

To script it instead, `POST /api/v1/users/` with the admin's token creates a user, and `PATCH /api/v1/users/<id>` with `{"is_active": true}` activates it.

**Add your model keys.** Paste an API key into a model component, or save it once under Settings → Global Variables so every flow can use it. Either way it's stored encrypted in Postgres.

**Call a flow from your app.** Create an API key under Settings → Langflow API Keys, then:

```bash
curl -X POST "https://<your-app>.up.railway.app/api/v1/run/<flow-id>" \
  -H "x-api-key: <your-langflow-api-key>" -H "Content-Type: application/json" \
  -d '{"input_value": "Hello", "input_type": "chat", "output_type": "chat"}'
```

The flow's **Share → API access** panel shows ready-made snippets in Python, JavaScript and curl. Webhook triggers need an API key too.

**Use flows as MCP tools.** Open a project's **MCP Server** tab for the server URL and a client config to paste into Claude Desktop, Cursor and other MCP clients.

**Upgrade.** In the Langflow service's Settings → Source, change the image tag (for example `langflowai/langflow:1.13.0`) and deploy. Database migrations run on boot. Read the [release notes](https://github.com/langflow-ai/langflow/releases) first, since agent components change between minor versions.

**Variables you might change.**
- `LANGFLOW_SECRET_KEY`: never change it after the first deploy, or stored keys can't be decrypted.
- `LANGFLOW_RATE_LIMIT_PER_MINUTE` (default 20): login attempts per minute. Railway's proxy makes every visitor look like the same IP, so this limit is shared by everyone.
- `LANGFLOW_WORKERS`: add it to run more worker processes on bigger plans.

**Memory.** Measured on Railway: Langflow idles at about 1.3 GB and Postgres at about 180 MB, more while flows run. Use the Hobby plan or above; it won't fit Trial's 512 MB.

## Why Deploy Langflow on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Langflow on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
