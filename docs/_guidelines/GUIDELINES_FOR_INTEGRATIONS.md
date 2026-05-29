# Guidelines for Pixeltable Integrations

This guide covers how to add a new AI provider or external-service integration to Pixeltable as a set of UDFs in `pixeltable/functions/`. It complements [`CONTRIBUTING.md`](../../CONTRIBUTING.md), which covers the dev environment and PR workflow.

## Audience and scope

**This guide is for:** external contributors adding a new provider (a new chat API, embedding API, image/video generation service, model-hosting platform, etc.) as a Pixeltable integration. Most integrations live in a single file under `pixeltable/functions/<provider>.py`.

**This guide is not for:** changes to the Pixeltable core (new column types, new iterator base classes, query-engine work). Those involve API-shape decisions that need maintainer discussion before any code is written. Open a [GitHub Discussion](https://github.com/orgs/pixeltable/discussions) first.

If you are using an AI coding assistant, point it at this file and at the canonical reference for your integration shape (see the next section). Tell it to mirror the structure of that reference file rather than invent a new shape.

## Pick a canonical reference and mirror it

Pixeltable has 40+ integrations in `pixeltable/functions/`. Some are older than others and don't all follow every current convention. Use the table below to pick the file your integration should most closely resemble. When in doubt, mirror its structure.

| Your integration looks like... | Mirror this file |
|---|---|
| Multimodal provider with file uploads (audio, video, large files) | [`pixeltable/functions/twelvelabs.py`](../../pixeltable/functions/twelvelabs.py) |
| Chat / vision / embedding API (image inputs, JSON outputs) | [`pixeltable/functions/openai.py`](../../pixeltable/functions/openai.py), [`pixeltable/functions/anthropic.py`](../../pixeltable/functions/anthropic.py) |
| Generic model-hosting platform with a `run(input, ref)` shape | [`pixeltable/functions/replicate.py`](../../pixeltable/functions/replicate.py), [`pixeltable/functions/fal.py`](../../pixeltable/functions/fal.py) |
| Image or video generation provider | [`pixeltable/functions/bfl.py`](../../pixeltable/functions/bfl.py), [`pixeltable/functions/reve.py`](../../pixeltable/functions/reve.py) |
| Embedding-only provider | [`pixeltable/functions/voyageai.py`](../../pixeltable/functions/voyageai.py), [`pixeltable/functions/jina.py`](../../pixeltable/functions/jina.py) |

`twelvelabs.py` is the current gold standard for multimodal integrations. It demonstrates async client registration, parallel file uploads with cleanup, base64-vs-upload size branching, overloads for multiple input modalities, conditional return types, and `pxt.ExternalServiceError` usage in under 270 lines. Read it before you write anything.

## What to create, and where

| File | Purpose |
|---|---|
| `pixeltable/functions/<provider>.py` | Your UDF module |
| `pixeltable/functions/__init__.py` | Add `from . import <provider>` |
| `pixeltable/env.py` | Add `self.__register_package('<sdk>')` to `Env.__register_packages` |
| `tests/functions/test_<provider>.py` | Tests, marked `@pytest.mark.remote_api` |
| `docs/release/howto/providers/working-with-<provider>.ipynb` | Tutorial notebook |
| `docs/release/integrations/frameworks.mdx` | Add a row for your provider |
| `pyproject.toml` | Add the SDK to optional dependencies |

The `pixeltable/env.py` edit is one line, but it's mandatory if you want `Env.get().require_package('<sdk>')` to work in your UDFs. Look for `Env.__register_packages` (around line 743) and add your SDK alphabetically alongside `openai`, `anthropic`, `replicate`, etc. If the import name differs from the pip name, pass `library_name=` (e.g. `self.__register_package('fal_client', library_name='fal-client')`).

## UDF signature rules

This section is the one most external contributions get wrong. Read it carefully.

### Accept image/media columns directly, never expose `.localpath`

The invariant: **users call your UDF by passing the column directly**, e.g. `provider.do_thing(t.image)`. Users should never have to write `t.col.localpath`. That's an internal accessor.

Two valid annotation styles for image inputs:

- **`pxt.Image`**: resolves to a `PIL.Image.Image` at runtime. Used in [`twelvelabs.py:71`](../../pixeltable/functions/twelvelabs.py).
- **`PIL.Image.Image`**: also accepted; the runtime resolves a `pxt.Image` column to a PIL object before invoking the UDF. Used in [`bfl.py:250`](../../pixeltable/functions/bfl.py), [`reve.py:177`](../../pixeltable/functions/reve.py), [`gemini.py:366`](../../pixeltable/functions/gemini.py), [`runwayml.py:185`](../../pixeltable/functions/runwayml.py) (and `runwayml.py:58` for the `list[PIL.Image.Image]` variant). This is the more common pattern in newer integrations.

For non-image media, the column type resolves to a local file path string:

- `pxt.Video` → file path string
- `pxt.Audio` → file path string
- `pxt.Document` → file path string

```python
# Right: image input as PIL.Image (most common in newer integrations)
@pxt.udf(is_deterministic=False, resource_pool='request-rate:provider')
async def edit_image(image: PIL.Image.Image, prompt: str) -> PIL.Image.Image: ...

# Right: image input as pxt.Image (twelvelabs.py:71)
@pxt.udf(resource_pool='request-rate:twelvelabs')
async def embed(text: str, image: pxt.Image | None = None, *, model_name: str) -> ...: ...

# Right: video overload (twelvelabs.py:179)
@embed.overload
async def _(video: pxt.Video, *, model_name: str, ...) -> ...: ...
```

**Do not** declare `file_path: str` parameters and ask users to pass `t.col.localpath` in their call sites. That breaks the column-typed API surface and doesn't match any other integration in `pixeltable/functions/`. Use `file_path: str` only in *internal helpers* (see `_embed_av_content` at [`twelvelabs.py:200`](../../pixeltable/functions/twelvelabs.py) for an example).

### Use overloads for multi-modality UDFs

A single UDF name can accept multiple input modalities (text, image, audio, video) by registering overloads. See [`twelvelabs.py:134-197`](../../pixeltable/functions/twelvelabs.py): the base `embed` UDF is text+image, with overloads for image-only, audio, and video. Each overload gets its own signature and parameters.

### Declare a `resource_pool` for rate limiting

Every UDF that hits an external API must declare a resource pool:

```python
@pxt.udf(resource_pool='request-rate:<provider>')
async def my_udf(...): ...
```

This lets the runtime apply the rate limit configured under `[<provider>] rate_limit = N` (default 600 RPM). UDFs without a resource pool are unthrottled, and heavy workloads will hammer the provider. References: [`fal.py:30`](../../pixeltable/functions/fal.py), [`replicate.py:30`](../../pixeltable/functions/replicate.py), [`twelvelabs.py:70`](../../pixeltable/functions/twelvelabs.py).

### Async by default

UDFs that make external requests should be `async`, calling the provider's async SDK client. If the provider only ships a sync SDK, wrap blocking calls with `asyncio.to_thread`. Do not introduce a separate sync code path that runs alongside an async one in the same module.

### Mark generative UDFs non-deterministic

For UDFs whose output varies between calls with the same input (chat completions, image generation, video generation, annotation, anything LLM-backed), you **must** set `is_deterministic=False`:

```python
@pxt.udf(is_deterministic=False, resource_pool='request-rate:<provider>')
async def my_generative_udf(...): ...
```

Without it, Pixeltable will cache results assuming determinism and serve stale outputs on retries or recomputes. This is a real correctness bug, not a perf hint. References:

- Generation: [`replicate.py:30`](../../pixeltable/functions/replicate.py), [`fal.py:30`](../../pixeltable/functions/fal.py), [`reve.py:139`](../../pixeltable/functions/reve.py)
- Chat / responses / TTS / vision: [`openai.py:242`](../../pixeltable/functions/openai.py) (`speech`), [`openai.py:446`](../../pixeltable/functions/openai.py) (`chat_completions`), [`openai.py:572`](../../pixeltable/functions/openai.py) (`responses`), [`anthropic.py:146`](../../pixeltable/functions/anthropic.py), [`gemini.py:96`](../../pixeltable/functions/gemini.py)

Embedding UDFs that are model-deterministic (same input, same vector for a fixed model) are intentionally omitted, since they're safe to cache. The OpenAI embedding UDFs further down `openai.py` are a good example. `bfl.py` is an outlier that doesn't set `is_deterministic=False` on its generation UDFs; treat that as an exception, not the rule.

## File handling: inline vs. upload

Two patterns, picked by file size:

### Inline base64 for small images

Use `pixeltable.utils.image.to_base64()`:

```python
from pixeltable.utils.image import to_base64

b64_str = to_base64(image, format=('png' if image.has_transparency_data else 'jpeg'))
```

Reference: [`twelvelabs.py:118`](../../pixeltable/functions/twelvelabs.py).

### Upload + cleanup for large or non-image files

Use an async context manager that uploads files in parallel and deletes them on exit. The canonical shape is `_asset_uploads()` in [`twelvelabs.py:41-67`](../../pixeltable/functions/twelvelabs.py):

```python
@asynccontextmanager
async def _asset_uploads(input_type, files: list[str]) -> AsyncIterator[list[str]]:
    if len(files) == 0:
        yield []
        return
    client = _provider_client()
    uploaded: list[str] = []
    try:
        tasks = [client.upload(file_path=f, file_type=input_type) for f in files]
        results = await asyncio.gather(*tasks)
        uploaded = [r.asset_id for r in results]
        yield uploaded
    finally:
        await asyncio.gather(
            *[client.assets.delete(aid) for aid in uploaded],
            return_exceptions=True,
        )
```

Note the `return_exceptions=True` on the cleanup path. Cleanup failures shouldn't mask the original error.

### Size-based branching

When the provider supports both inline and upload paths, branch on file size: inline for small, upload for large. The threshold is provider-specific; check the provider's API docs for any documented inline payload limit and use that. TwelveLabs uses 2 MB for its own API, but the constant isn't universal. Reference: [`twelvelabs.py:215-241`](../../pixeltable/functions/twelvelabs.py) for the branching pattern.

## Client registration and credentials

### Register the client at module top

```python
from pixeltable.env import register_client

@register_client('<provider>')
def _(api_key: str) -> '<provider>.AsyncClient':
    import <provider_sdk>
    return <provider_sdk>.AsyncClient(api_key=api_key)
```

Lazy-import the SDK inside the function. Reference: [`twelvelabs.py:30-34`](../../pixeltable/functions/twelvelabs.py).

### Look up the client at call time

```python
from pixeltable.runtime import get_runtime

def _provider_client() -> '<provider>.AsyncClient':
    return get_runtime().get_client('<provider>')
```

Use `get_runtime().get_client(...)`, *not* `Env.get().get_client(...)`. `Env` does not expose `get_client`; the registered-client lookup lives on the runtime. References: [`twelvelabs.py:18, 37-38`](../../pixeltable/functions/twelvelabs.py), [`replicate.py:26-27`](../../pixeltable/functions/replicate.py), [`fal.py:26-27`](../../pixeltable/functions/fal.py).

### Check the SDK is installed at call time

Inside each UDF body, call `env.Env.get().require_package('<sdk>')` before you use the SDK. This makes missing-install errors surface at the right moment with a clear message instead of a raw `ImportError`. Reference: [`twelvelabs.py:107`](../../pixeltable/functions/twelvelabs.py).

For `require_package` to work, **your SDK must be registered** in `Env.__register_packages` in [`pixeltable/env.py`](../../pixeltable/env.py) (around line 743). Add a line:

```python
self.__register_package('<sdk>')
# or, if the import name differs from the pip name:
self.__register_package('<sdk_import>', library_name='<pip-name>')
```

Insert it alphabetically alongside the existing entries.

### Credentials

Pixeltable loads provider credentials from the environment variable `<PROVIDER>_API_KEY` and from the `[<provider>]` section of the user's config file. You don't need to write any credential-handling code; `@register_client` wires this for you. Do not invent a new credential mechanism for your provider.

## Error handling

### Raise `pxt.ExternalServiceError` for upstream failures

When the provider returns an unexpected response (empty data, malformed payload, an explicit error code you can identify), raise:

```python
raise pxt.ExternalServiceError(
    pxt.ErrorCode.PROVIDER_ERROR,
    f"Provider returned unexpected response: {response}",
    provider='<provider>',
)
```

Reference: [`twelvelabs.py:127`](../../pixeltable/functions/twelvelabs.py). Do not let raw SDK exceptions bubble up unfiltered; they confuse users and lose the structured `provider=` annotation that Pixeltable serializes in error metadata.

### Use exponential backoff for retries

For transient failures (network blips, 429s, 503s), use `pixeltable.utils.http.exponential_backoff()`. No fixed-interval polling loops; they don't degrade gracefully under load.

## Return types

### Static return types

Just annotate the function:

```python
async def my_udf(...) -> str: ...
async def my_udf(...) -> pxt.Image: ...
async def my_udf(...) -> pxt.Array[np.float32] | None: ...
```

### Variable return types

If the return shape depends on an argument (e.g. embedding dimension depends on the model name), register a `conditional_return_type` callback:

```python
@my_udf.conditional_return_type
def _(model_name: str) -> ts.ArrayType:
    if model_name == 'small':
        return ts.ArrayType(shape=(512,), dtype=np.dtype('float32'))
    if model_name == 'large':
        return ts.ArrayType(shape=(1024,), dtype=np.dtype('float32'))
    return ts.ArrayType(dtype=np.dtype('float32'))
```

Reference: [`twelvelabs.py:253-259`](../../pixeltable/functions/twelvelabs.py).

### Generated media outputs

For generated images/audio/video, return `pxt.Image`, `pxt.Audio`, `pxt.Video` directly. If the provider hosts the result at a URL, return the URL string and let users cast it in their schema: `t.add_computed_column(image=t.response.output[0].astype(pxt.Image))`. Reference: [`replicate.py:71`](../../pixeltable/functions/replicate.py).

## Module footer

Every integration module ends with:

```python
from pixeltable.utils.code import local_public_names

__all__ = local_public_names(__name__)


def __dir__() -> list[str]:
    return __all__
```

This is what controls what `from pixeltable.functions.<provider> import *` exposes. Reference: every file in `pixeltable/functions/`.

## Tests

- Place tests in `tests/functions/test_<provider>.py`.
- Mark every test that calls the live API with `@pytest.mark.remote_api`. These are excluded from default CI runs.
- Mark slow tests `@pytest.mark.expensive`.
- Use the existing test fixtures from `tests/utils.py`: `get_image_files()`, `get_video_files()`, `get_audio_files()`. Do not add new test assets unless no existing fixture covers your case.
- Verify both shape and content where possible. For embedding UDFs, assert dimensionality: `assert res['embed'][0].shape == (1024,)`.
- For tests requiring API credentials, use the `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `<PROVIDER>_API_KEY` env var convention.

Reference: [`tests/functions/test_twelvelabs.py`](../../tests/functions/test_twelvelabs.py), [`tests/functions/test_openai.py`](../../tests/functions/test_openai.py).

## Tutorial notebook

- Path: `docs/release/howto/providers/working-with-<provider>.ipynb`.
- **Use public URLs for sample data**, never local repo paths. `https://raw.githubusercontent.com/pixeltable/pixeltable/main/tests/data/...` resolves anywhere. Paths like `../../../../tests/data/...` only work inside the repo and break for everyone else.
- Follow [`GUIDELINES_FOR_NOTEBOOKS.md`](GUIDELINES_FOR_NOTEBOOKS.md): YAML frontmatter in a raw cell, no H1 headers, `##` for main sections, `###` for subsections, clear outputs before commit.
- Demonstrate the UDF on a real column directly: `t.add_computed_column(result=<provider>.<udf>(t.image))`. Do not show `.localpath` in your examples.

## Docstrings

Follow [`GUIDELINES_FOR_DOCSTRINGS.md`](GUIDELINES_FOR_DOCSTRINGS.md):

- Examples use `>>>` prompts, not fenced code blocks.
- Backticks must be properly paired.
- HTML tags must be self-closing.
- Include a `__Requirements:__` block with the `pip install` line.
- Include a one-line summary, then `Args:` / `Returns:` / `Examples:` sections.

Reference: [`twelvelabs.py:72-106`](../../pixeltable/functions/twelvelabs.py) for a complete, conformant docstring.

## Pre-PR checklist

Before opening a PR, walk through this list:

- [ ] UDFs accept image/media columns directly (`pxt.Image` or `PIL.Image.Image`, `pxt.Video`, `pxt.Audio`, `pxt.Document`); never `file_path: str` in public signatures
- [ ] Notebook examples pass column references directly (no `.localpath`)
- [ ] Every external-call UDF declares `resource_pool='request-rate:<provider>'`
- [ ] Generative UDFs declare `is_deterministic=False`
- [ ] `@env.register_client('<provider>')` at module top with lazy SDK import
- [ ] SDK registered in `Env.__register_packages` in `pixeltable/env.py`
- [ ] `Env.get().require_package('<sdk>')` inside each UDF body
- [ ] `pxt.ExternalServiceError(pxt.ErrorCode.PROVIDER_ERROR, ..., provider='<provider>')` for upstream failures
- [ ] No fixed-interval polling; retries use exponential backoff
- [ ] Module ends with `__all__ = local_public_names(__name__)` and `__dir__`
- [ ] `conditional_return_type` registered if the return shape depends on arguments
- [ ] Tests in `tests/functions/test_<provider>.py`, marked `@pytest.mark.remote_api`
- [ ] Tests reuse existing fixtures from `tests/utils.py` rather than adding new assets
- [ ] Notebook first cell is a raw cell with YAML frontmatter; no H1 markdown headers
- [ ] Notebook uses `raw.githubusercontent.com` URLs, never local paths
- [ ] Docstrings follow [`GUIDELINES_FOR_DOCSTRINGS.md`](GUIDELINES_FOR_DOCSTRINGS.md)
- [ ] `make format` and `make check` pass

## Working with an AI coding assistant

If you're using Claude, Cursor, Copilot, etc. to draft your integration:

1. Open this file and the canonical reference for your integration shape (table at the top) in your editor.
2. Tell your assistant: "Mirror the structure of `pixeltable/functions/twelvelabs.py` (or whichever you picked). Read this guidelines file and that reference file before writing anything."
3. After it drafts the module, walk through the pre-PR checklist above with the assistant. Each item maps to a concrete file/line reference in this doc; verify each one.
4. If your assistant suggests a pattern that isn't in any existing `pixeltable/functions/*.py` file, ask it to find the closest existing example and mirror that instead.
