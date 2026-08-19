# Provider notebooks

These notebooks appear in the **Integrations** tab, not under How-to, even though they live here.
Slugs stay at `howto/providers/` because moving one costs a redirect; grouping happens in
`docs/release/docs.json`.

Before adding or editing one, read
[`docs/release/integrations/AGENTS.md`](../../integrations/AGENTS.md). It owns the rules for which
group a new integration belongs to, the frontmatter shape, the capability matrix row, and how the
credential environment variable is derived.

Two things that are wrong often enough to repeat here:

- **Title is the bare vendor name.** `title: "Anthropic"`, not "Working with Anthropic in
  Pixeltable". Frontmatter goes in a raw cell at index 0. Without it, Mintlify falls back to the
  notebook's H1 and every sidebar entry starts with the same 13 characters.
- **The credential variable is derived from `@env.register_client(...)`, not the vendor name.**
  `mistralai.py` registers `mistral`, so it reads `MISTRAL_API_KEY`. `voyageai.py` registers
  `voyage`. Replicate uses `api_token`, RunwayML uses `api_secret`. Read the decorator.
