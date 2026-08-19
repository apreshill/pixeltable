# Integrations docs: routing rules

Scope: the Integrations tab in `docs/release/docs.json`, the pages in this directory, and the
provider notebooks in `docs/release/howto/providers/`.

Read this before adding an integration page or editing the Integrations tab. It exists because
the tab previously mixed several classification axes at once, which left shipped features
undiscoverable and gave new pages no obvious home.

## Where does it go

Answer in order. The first match wins.

1. **Does Pixeltable call it for inference?** → `Models`
   - Needs an API key and a vendor account → **Hosted providers**
   - An engine that runs whatever model you point it at → **Local runtimes**
     (Ollama, llama.cpp, vLLM, Hugging Face)
   - A specific named model with its own udfs → **Open models**
     (SAM 3, CLIP, DETR, ViT, Whisper, YOLOX)
2. **Does Pixeltable read from it or write to it?** → `Data`
   (object stores, file formats, databases, dataset tools)
3. **Does it run, expose, or observe Pixeltable in an application?** → `Serving & ops`
   (FastAPI, MCP, OpenTelemetry)
4. **None of the above** → it is probably not an integration. See below.

## Capability is a column, never a category

Do not create a group named for a capability: no "Embedding providers", no "Rerankers", no
"Vision models". Vendors combine capabilities differently. `functions/voyageai.py` ships
`embeddings`, `rerank`, and `multimodal_embed`; `functions/jina.py` ships `embeddings` and
`rerank`. A capability-named group forces those vendors into two places at once with no
cross-link, which is the duplication this structure exists to prevent.

Capability belongs in the matrix columns on `docs/release/integrations.mdx`. To surface a
capability view, add a table that re-cuts the same integrations. Do not add a nav group.

## Models are not integrations, and channels are not origins

CLIP, SAM 3, DETR, ViT, sentence transformers, and cross-encoders are udfs inside
`pixeltable/functions/huggingface.py`. They belong on `integrations/open-models.mdx`, not in the
Hosted providers list.

Hugging Face is a distribution channel. CLIP is an OpenAI model, SAM 3 and DETR are Meta models,
ViT is a Google model. Name the origin on the model row. A vendor legitimately appears in two
places when it both hosts an API and originates an open model: OpenAI is a hosted provider and
the origin of CLIP and Whisper. That is provenance, not duplication.

## What is not an integration

- Utility modules in `pixeltable/functions/` that wrap no external service: `string`, `math`,
  `json`, `date`, `timestamp`, `image`, `video`, `audio`, `array`, `uuid`, `vision`.
- Core API surface. Pandas and PyArrow interop is the data model, not a partnership.
- Pixeltable's own libraries. Those live under `docs/release/libraries/`.

## Adding an integration

1. Module exists at `pixeltable/functions/<name>.py` with `@pxt.udf` functions.
2. Notebook at `docs/release/howto/providers/working-with-<name>.ipynb`. Keep this path even
   though the page appears in the Integrations tab. Slugs are decoupled from nav grouping, and
   moving one costs a redirect.
3. Frontmatter in a **raw cell at index 0**, with a bare vendor name as the title:

   ```
   ---
   title: "Anthropic"
   icon: "notebook"
   description: "[Open in Kaggle](...) | [Open in Colab](...) | [View on GitHub](...)"
   ---
   ```

   Bare vendor name, not "Working with X in Pixeltable". The sidebar shows this string, and a
   shared prefix across every entry buries the word that distinguishes them.
4. Register the page id in `docs/release/docs.json` under the right group from the decision list
   above.
5. Add a row to the capability matrix in `docs/release/integrations.mdx`. Source every mark from
   the udf names in the module. Never copy marks from another page; the older cards drifted from
   the code and that is what the matrix replaced.
6. Record the credential. Pixeltable derives the environment variable from the client
   registration as `{NAME}_{CREDENTIAL_PARAM}` uppercased, where both come from the
   `@env.register_client(...)` decorator. These do not always match the vendor name:
   `mistralai.py` registers `mistral`, so the variable is `MISTRAL_API_KEY`; `voyageai.py`
   registers `voyage`; Replicate uses `api_token`; RunwayML uses `api_secret`. Read the decorator
   rather than guessing.

## Naming

Use one canonical vendor string across the notebook title, the capability matrix row, and the SDK
reference page. These three surfaces have drifted before (`working-with-hugging-face` /
`huggingface` / "Hugging Face Hub"), which makes the pages impossible to match by string.

## Links

Internal links are `/{page-id}` built from `docs.json`, never relative paths and never
`/docs/release/...`. Page ids are the path under `docs/release/` without the extension.

## Before you finish

Run `make docs` and `make linkscheck`. Never change `integrations.telemetry.enabled` in
`docs/release/docs.json`; it must stay `true`.
