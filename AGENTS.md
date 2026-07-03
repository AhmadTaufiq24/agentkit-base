# AGENTS.md

## Cursor Cloud specific instructions

This repo is a single Next.js 14 (App Router, TypeScript) app — `workshop-agent-kit` — an onchain AI agent chat UI built on Coinbase AgentKit. There is no separate backend, database, or containerized dependency; the frontend and the `/api/agent` route run together in one Next.js process. Package manager is npm (`package-lock.json`).

Standard commands live in `package.json` (`dev`, `build`, `start`, `lint`). The dev server runs on `http://localhost:3000`; the agent API is `POST /api/agent`.

### Running / testing caveats
- The chat UI loads and the user→`/api/agent`→UI round trip works without any credentials, but the agent itself will only reply with a config error until secrets are set. `POST /api/agent` throws (returned as a JSON `error`, rendered as an agent bubble) when `GOOGLE_API_KEY` is missing (`app/api/agent/create-agent.ts`), and `prepareAgentkitAndWalletProvider` throws without `CDP_API_KEY_ID` + `CDP_API_KEY_SECRET` (`app/api/agent/prepare-agentkit.ts`). `CDP_WALLET_SECRET` is also needed in practice for wallet/signing.
- To exercise the real onchain agent, provide env vars in a `.env` file (gitignored; not present in repo despite README's `mv .env.local .env`, which does not apply here): `GOOGLE_API_KEY`, `CDP_API_KEY_ID`, `CDP_API_KEY_SECRET`, `CDP_WALLET_SECRET`. Optional: `NETWORK_ID` (defaults to `base-sepolia`), `OPENAI_API_KEY`, `RPC_URL`, `PAYMASTER_URL`, `IDEMPOTENCY_KEY`. These are external SaaS credentials (Google Gemini + Coinbase Developer Platform), so end-to-end agent testing needs them supplied as secrets.
- In development mode the agent persists its generated smart wallet to `wallet_data.txt` (gitignored) so the same wallet is reused across restarts; delete it to force a new wallet.
- `next lint` currently reports two pre-existing unused-import errors (`ChatOpenAI`, `generatePrivateKey`) left in as commented-out-LLM scaffolding. These are not from setup; do not "fix" them unless the task calls for it.
- The `bigint: Failed to load bindings, pure JS will be used` message during `build`/runtime is a harmless native-binding fallback warning.
