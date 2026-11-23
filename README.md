# Chatbot scoping coach

A free web coach from the workshop "Building chatbots for the people you serve" (ILSS masterclass). It asks a non-profit leader questions, one at a time, across 12 stages, and ends with a scoping report: where a chatbot makes sense, where it doesn't, and what to work out before building.

- `index.html` is the page, served by GitHub Pages.
- `worker/` is a small Cloudflare Worker that calls the Claude API. The API key lives only there, as a secret. The coach's instructions live there too, so the endpoint can't be used as a general Claude proxy.

Until the worker is connected (`API_URL` in `index.html` is empty, or the worker is down), the page runs in guided mode: a fixed set of questions with the answers given back as notes.

## Connect the AI coach

You need Node.js, a free Cloudflare account and an Anthropic API key (console.anthropic.com).

1. In the Anthropic Console, set a monthly spend limit for the key before you share the link.
2. In `worker/`, run `npm install`, then `npx wrangler login`.
3. Run `npx wrangler secret put ANTHROPIC_API_KEY` and paste the key when asked.
4. Run `npx wrangler deploy`. It prints the worker's URL.
5. Put that URL in `API_URL` near the top of the script in `index.html`, and push.

## Cost

Each reply calls `claude-opus-5-5` at low effort, with the instructions and conversation history cached. A full 12-stage conversation is roughly US$0.20–0.40. To make it much cheaper, change `MODEL` in `worker/src/index.js` to `claude-haiku-5-5` and redeploy.

## Guardrails

- Only the page's own address may call the worker (`ALLOWED_ORIGINS` in `worker/wrangler.toml`).
- At most 60 messages per conversation, and 3,000 characters per message.
- Nothing is stored: the page keeps the conversation in the browser tab only.
