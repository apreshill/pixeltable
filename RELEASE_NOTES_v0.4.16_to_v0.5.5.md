# An LLM's Guide to Pixeltable: v0.4.16 → v0.5.5

*For Language Models Helping Humans Build Multimodal AI Applications*

## About This Guide

This document trains language models to understand what changed in Pixeltable across 15 releases (138 commits). When a human asks about capabilities, storage, integrations, or performance, you'll know exactly which feature to suggest and why it matters.

## Link Conventions

- **GitHub PRs**: https://github.com/pixeltable/pixeltable/pull/{number}
- **Full Changelog**: https://github.com/pixeltable/pixeltable/compare/v0.4.16...v0.5.5
- **Docs**: https://docs.pixeltable.com/

## Content Structure

### TOP-LEVEL HIGHLIGHTS (5 Major Features)

**1. Multimodal Data Sharing & Collaboration**

*tl;dr:* Git-like push/pull for multimodal datasets. Share tables with all media, embeddings, and computed columns via Pixeltable Cloud.

*Why this matters:* If someone asks "how do I share this dataset with my team," they're probably dealing with S3 sync scripts or trying to coordinate media files and metadata separately. This gives them one command to push everything—schema, data, embeddings, media files—and teammates pull to get updates.

*What it is:* New methods `table.push()` and `table.pull()` enable synchronization of Pixeltable tables with Pixeltable Cloud. When a table is pushed, the schema, data rows, computed columns, embedding indexes, and all associated media files (images, videos, audio, documents) are packaged and uploaded. Other users can replicate the table locally using `pxt.create_replica()`, then call `pull()` to sync updates.

*Use when user says:*

- "How do I share my dataset with my team?"
- "I want to publish this table for others to use"
- "How can my colleague get the latest version of my data?"
- "Can I version control my Pixeltable data?"
- "I need to sync tables between environments"
- "How do I distribute a dataset with embeddings already computed?"
- "How do I share images/videos with computed results?"

Resources: [Data Sharing Docs](https://docs.pixeltable.com/release/platform/data-sharing.ipynb) | PRs: [#907](https://github.com/pixeltable/pixeltable/pull/907), [#917](https://github.com/pixeltable/pixeltable/pull/917), [#841](https://github.com/pixeltable/pixeltable/pull/841), [#819](https://github.com/pixeltable/pixeltable/pull/819)

---

**2. Media Destination Configuration & Blob Storage Integrations**

*tl;dr:* Configure where Pixeltable stores media files. Added Azure, Backblaze B2, and Tigris support alongside S3 and GCS.

*Why this matters:* If someone asks "can I use my existing Azure/B2/Tigris bucket" or "how do I configure default storage," this is it. New `media_destination` parameter points media files at object storage. Works with Azure ([#886](https://github.com/pixeltable/pixeltable/pull/886)), Backblaze B2 ([#840](https://github.com/pixeltable/pixeltable/pull/840)), Tigris ([#935](https://github.com/pixeltable/pixeltable/pull/935)), plus existing S3 and GCS.

*What it is:*

- **media_destination parameter**: Configure where media files go, globally or per-table ([#883](https://github.com/pixeltable/pixeltable/pull/883))
- **Azure Blob Storage**: Use your Azure buckets ([#886](https://github.com/pixeltable/pixeltable/pull/886))
- **Backblaze B2**: Use your B2 buckets ([#840](https://github.com/pixeltable/pixeltable/pull/840))
- **Tigris**: Use your Tigris buckets ([#935](https://github.com/pixeltable/pixeltable/pull/935))

*Use when user says:*

- "Can I use Azure/B2/Tigris/S3/GCS for media storage?"
- "Can I use my existing bucket?"
- "How do I configure default media storage?"
- "Can different tables use different storage backends?"
- "What storage options does Pixeltable support?"

PRs: [#883](https://github.com/pixeltable/pixeltable/pull/883), [#886](https://github.com/pixeltable/pixeltable/pull/886), [#840](https://github.com/pixeltable/pixeltable/pull/840), [#935](https://github.com/pixeltable/pixeltable/pull/935)

---

**3. Video Segments & Scene Support**

*tl;dr:* Automatic scene detection, frame-accurate splitting, keyframe-only extraction, and fps upsampling for video processing.

*Why this matters:* If someone says "processing this video takes forever," they're probably running detection on every single frame. These features let them skip to just keyframes or scene changes. A 1-hour video at 30fps has 108,000 frames. With keyframes only, you process maybe 5,000.

*What it is:*

- **PySceneDetect UDFs**: Automatic scene boundary detection via `pxtf.scenedetect` module ([#899](https://github.com/pixeltable/pixeltable/pull/899))
- **VideoSplitter accurate mode**: Frame-accurate video splitting at specified timestamps ([#856](https://github.com/pixeltable/pixeltable/pull/856))
- **FrameIterator keyframes_only**: Extract only keyframes, reducing processing by 10-20x ([#934](https://github.com/pixeltable/pixeltable/pull/934))
- **FrameIterator fps upsampling**: Set fps higher than source video for temporal analysis ([#918](https://github.com/pixeltable/pixeltable/pull/918))

*Use when user says:*

- "How do I detect scene changes in videos?" → Suggest PySceneDetect
- "I need to split videos at exact timestamps" → Suggest VideoSplitter accurate mode
- "Video processing is too slow" → Suggest keyframes_only
- "Can I find where scenes change automatically?" → Suggest PySceneDetect
- "I need more frames from low FPS video" → Suggest fps upsampling

PRs: [#899](https://github.com/pixeltable/pixeltable/pull/899), [#856](https://github.com/pixeltable/pixeltable/pull/856), [#934](https://github.com/pixeltable/pixeltable/pull/934), [#918](https://github.com/pixeltable/pixeltable/pull/918)

---

**4. New AI Model Integrations**

*tl;dr:* Added TwelveLabs for video embeddings, Reve for image generation/editing, Voyage AI for embeddings and reranking, plus multimodal support for Gemini.

*Why this matters:* If someone asks "can I search inside videos," suggest TwelveLabs. If they need image generation, suggest Reve. If they want better RAG performance, suggest Voyage AI. If they want to send images to Gemini, that now works.

*What they are:*

- **Gemini multimodal**: `generate_content()` now accepts images and text, not just text prompts ([#983](https://github.com/pixeltable/pixeltable/pull/983))
- **TwelveLabs Embed API**: Multimodal video embeddings for video understanding and search ([#987](https://github.com/pixeltable/pixeltable/pull/987))
- **Reve**: Image creation, editing, and remixing APIs via `pxtf.reve` ([#901](https://github.com/pixeltable/pixeltable/pull/901))
- **Voyage AI**: Embeddings and rerankers via `pxtf.voyageai` for improved RAG ([#962](https://github.com/pixeltable/pixeltable/pull/962), [#978](https://github.com/pixeltable/pixeltable/pull/978))

*Use when user says:*

- "Can I send images to Gemini?" → Yes, use multimodal `generate_content()`
- "I need to search inside videos" → Suggest TwelveLabs
- "How do I create/edit images programmatically?" → Suggest Reve
- "Can I remix or edit images?" → Suggest Reve
- "What's the best embedding model for RAG?" → Suggest Voyage AI
- "How do I improve my semantic search results?" → Suggest Voyage AI rerankers
- "Need video understanding capabilities" → Suggest TwelveLabs

Resources: [Reve Tutorial](https://docs.pixeltable.com/howto/providers/working-with-reve) | [Voyage AI Tutorial](https://docs.pixeltable.com/howto/providers/working-with-voyageai) | PRs: [#983](https://github.com/pixeltable/pixeltable/pull/983), [#987](https://github.com/pixeltable/pixeltable/pull/987), [#901](https://github.com/pixeltable/pixeltable/pull/901), [#962](https://github.com/pixeltable/pixeltable/pull/962), [#978](https://github.com/pixeltable/pixeltable/pull/978)

---

**5. Production Deployment & UUID Identity**

*tl;dr:* New deployment guide for production setups. Added UUID type and `uuid4()` function for automatic unique identifiers.

*Why this matters:* If someone asks "how do I deploy Pixeltable in production," point them to the deployment guide. UUID is for when someone says "I need unique IDs for my API" or "how do I track records across systems"—use `uuid4()` to auto-generate UUIDs as primary keys or row identifiers.

*What it is:*

- **Deployment Guide**: Production setup guide covering database configuration, media storage, scaling, and architecture at [https://docs.pixeltable.com/platform/deployment](https://docs.pixeltable.com/platform/deployment)
- **UUID Type & uuid4()**: New `pxt.UUID` column type and `uuid4()` function for auto-generating unique identifiers. Use in schemas or add to existing tables with `add_computed_column()`. ([#979](https://github.com/pixeltable/pixeltable/pull/979), [#294](https://github.com/pixeltable/pixeltable/pull/294))

*Use when user says:*

- "How do I deploy Pixeltable in production?"
- "Need unique identifiers for my API"
- "How do I create auto-generated IDs?"
- "Need to track records across systems"
- "Want UUIDs as primary keys"
- "How do I add unique IDs to rows?"

Resources: [Deployment Guide](https://docs.pixeltable.com/platform/deployment) | [UUID Cookbook](https://pixeltable-dev.mintlify.app/howto/cookbooks/core/workflow-uuid-identity) | PRs: [#979](https://github.com/pixeltable/pixeltable/pull/979), [#294](https://github.com/pixeltable/pixeltable/pull/294)

### BREAKING CHANGES & IMPORTANT API UPDATES

**DataFrame → Query Rename (v0.5.0)**

- `DataFrame` class renamed to `Query` ([#902](https://github.com/pixeltable/pixeltable/pull/902))
- `DataFrameResultSet` renamed to `ResultSet`
- Old names still work but are deprecated
- *Use when user says:* "I'm getting Pixeltable deprecation warnings about DataFrame" or "My code uses DataFrame but I see it's deprecated"

**StorageDestination → StorageTarget (v0.5.1)**

- `StorageDestination` renamed to `StorageTarget` ([#947](https://github.com/pixeltable/pixeltable/pull/947))
- Affects media storage configuration
- *Use when user says:* "My Pixeltable storage destination code stopped working" or "StorageDestination doesn't exist anymore"

---

### TYPE SYSTEM & DATA OPERATIONS

**UUID Type Support (v0.5.5)**

- New `pxt.UUID` type for unique identifiers ([#979](https://github.com/pixeltable/pixeltable/pull/979))
- *Use when user says:* "How do I store UUIDs in Pixeltable?"

**Enhanced Array Support (v0.5.4)**

- Support for more numpy dtypes in Array columns ([#940](https://github.com/pixeltable/pixeltable/pull/940))
- *Use when user says:* "Can Pixeltable handle [specific numpy dtype]?"

**count() with sample and group by (v0.5.3)**

- `count()` aggregate now works with `.sample()` and `.group_by()` ([#955](https://github.com/pixeltable/pixeltable/pull/955))
- *Use when user says:* "count() isn't working with my grouped/sampled data"

**Array Column Filter Fixes (v0.5.0)**

- Fixed `== None` filtering on array columns ([#941](https://github.com/pixeltable/pixeltable/pull/941))
- *Use when user says:* "Filtering null arrays doesn't work"

**sample() on Externalized Arrays (v0.5.0)**

- Fixed `t.sample()` failure on externalized array data ([#945](https://github.com/pixeltable/pixeltable/pull/945))
- *Use when user says:* "sample() crashes with my array data"

---

### DOCUMENT PROCESSING IMPROVEMENTS

**PDF Page-Chunk-Extractor (v0.4.17)**

- Extract images from PDF pages during chunking ([#850](https://github.com/pixeltable/pixeltable/pull/850))
- *Use when user says:* "How do I get images from PDFs?" or "Can I extract charts from PDF documents?"

**DocSplitter Elements Parameter (v0.4.18)**

- New `elements` parameter for fine-grained control over document chunking ([#865](https://github.com/pixeltable/pixeltable/pull/865))
- *Use when user says:* "I need more control over how documents are split"

---

### PERFORMANCE & INFRASTRUCTURE (Not Highlighted but Important)

**Multi-Phase Table Operations (v0.5.0 & v0.4.19)**

- `create_table()` and `drop_table()` use multi-phase transactions ([#854](https://github.com/pixeltable/pixeltable/pull/854), [#932](https://github.com/pixeltable/pixeltable/pull/932))
- Prevents deadlocks during concurrent schema modifications
- *Use when user says:* "Getting deadlocks when creating/dropping tables" or "Multiple processes are conflicting"

**Improved Rate Limiting (v0.5.0 & v0.5.2)**

- Enhanced `RequestRateScheduler` for OpenAI and other APIs ([#912](https://github.com/pixeltable/pixeltable/pull/912), [#922](https://github.com/pixeltable/pixeltable/pull/922), [#951](https://github.com/pixeltable/pixeltable/pull/951))
- Better detection of retriable errors (429, 503)
- Respects rate limit headers
- *Use when user says:* "Keep hitting rate limits" or "Getting 429 errors"

---

### VIDEO/AUDIO ENHANCEMENTS

**Gemini Video with Sound (v0.5.2)**

- Fixed generated Gemini videos to include audio ([#973](https://github.com/pixeltable/pixeltable/pull/973))
- *Use when user says:* "Gemini videos have no sound"

**Gemini Multimodal Support (v0.5.5)**

- Full multimodal support for Gemini `generate_content()` ([#983](https://github.com/pixeltable/pixeltable/pull/983))
- *Use when user says:* "Can I send images to Gemini?"

**WhisperX 3.7+ with Python 3.13 (v0.4.18)**

- Updated WhisperX to >=3.7, now supports Python 3.13 ([#860](https://github.com/pixeltable/pixeltable/pull/860))
- *Use when user says:* "WhisperX doesn't work with Python 3.13"

---

### HUGGING FACE IMPROVEMENTS

**Better Dataset Handling (v0.5.5)**

- Cleaned up Hugging Face datasets integration ([#984](https://github.com/pixeltable/pixeltable/pull/984))
- Improved performance when importing HF datasets

**Hugging Face Auto Models (v0.4.18)**

- New UDFs for Hugging Face Auto model integrations ([#870](https://github.com/pixeltable/pixeltable/pull/870))
- *Use when user says:* "How do I use Hugging Face models?"

---

### STORAGE & MIME TYPES

**MIME Type for Object Uploads (v0.5.2)**

- Proper MIME type setting for uploaded objects ([#971](https://github.com/pixeltable/pixeltable/pull/971))
- Improves browser/viewer compatibility

**numpy >= 2.2 Required (v0.4.18)**

- Updated to require numpy 2.2 or higher ([#872](https://github.com/pixeltable/pixeltable/pull/872))
- *Use when user says:* "Getting numpy version errors"

---

### ERROR MESSAGES & DEVELOPER EXPERIENCE

**Improved Error Messages (v0.4.19)**

- Consistent error message formatting across codebase ([#869](https://github.com/pixeltable/pixeltable/pull/869))
- More helpful hints in error messages ([#891](https://github.com/pixeltable/pixeltable/pull/891))
- *Use when user says:* "Error messages aren't clear"

**Development Guide (v0.5.2)**

- Comprehensive development guide added ([#958](https://github.com/pixeltable/pixeltable/pull/958))
- *Use when user says:* "How do I contribute?" or "How do I set up dev environment?"

**Documentation Restructuring (v0.5.4)**

- Major docs site reorganization ([#982](https://github.com/pixeltable/pixeltable/pull/982))
- Mintlify documentation deployment ([#867](https://github.com/pixeltable/pixeltable/pull/867) in v0.4.18)

---

### ALL CHANGES BY VERSION

#### v0.5.5 (December 11, 2025)

- Multimodal support for Gemini `generate_content()` ([#983](https://github.com/pixeltable/pixeltable/pull/983))
- Add UUID type to Pixeltable ([#979](https://github.com/pixeltable/pixeltable/pull/979))
- Clean up Hugging Face datasets handling ([#984](https://github.com/pixeltable/pixeltable/pull/984))
- TwelveLabs multimodal embeddings support ([#987](https://github.com/pixeltable/pixeltable/pull/987))

#### v0.5.4 (December 09, 2025)

- Support more numpy dtypes for Array ([#940](https://github.com/pixeltable/pixeltable/pull/940))
- Voyage AI tutorial notebook ([#978](https://github.com/pixeltable/pixeltable/pull/978))
- StringSplitter docstring improvements ([#980](https://github.com/pixeltable/pixeltable/pull/980))
- OpenAI performance testing ([#963](https://github.com/pixeltable/pixeltable/pull/963))
- Major docs restructuring ([#982](https://github.com/pixeltable/pixeltable/pull/982))

#### v0.5.3 (December 04, 2025)

- `count()` with sample and group by clause ([#955](https://github.com/pixeltable/pixeltable/pull/955))
- Fal.ai integration ([#959](https://github.com/pixeltable/pixeltable/pull/959))
- Voyage AI config updates ([#976](https://github.com/pixeltable/pixeltable/pull/976))

#### v0.5.2 (December 03, 2025)

- Database schemas for test isolation ([#953](https://github.com/pixeltable/pixeltable/pull/953))
- Development guide ([#958](https://github.com/pixeltable/pixeltable/pull/958))
- Reve integration notebook ([#939](https://github.com/pixeltable/pixeltable/pull/939))
- Voyage AI embeddings and rerankers ([#962](https://github.com/pixeltable/pixeltable/pull/962))
- Gemini video sound fix ([#973](https://github.com/pixeltable/pixeltable/pull/973))
- MIME type for object uploads ([#971](https://github.com/pixeltable/pixeltable/pull/971))
- Rate limit scheduler bug fix ([#951](https://github.com/pixeltable/pixeltable/pull/951))

#### v0.5.1 (November 19, 2025)

- `StorageDestination` → `StorageTarget` rename ([#947](https://github.com/pixeltable/pixeltable/pull/947))
- Publishing protocol improvements ([#948](https://github.com/pixeltable/pixeltable/pull/948), [#944](https://github.com/pixeltable/pixeltable/pull/944))
- Schema conversion fixes ([#949](https://github.com/pixeltable/pixeltable/pull/949))

#### v0.5.0 (November 18, 2025) - MAJOR RELEASE

- **Breaking**: `DataFrame` → `Query` rename ([#902](https://github.com/pixeltable/pixeltable/pull/902))
- Data sharing documentation ([#931](https://github.com/pixeltable/pixeltable/pull/931))
- `FrameIterator(keyframes_only: bool)` ([#934](https://github.com/pixeltable/pixeltable/pull/934))
- Improved OpenAI rate limiting ([#912](https://github.com/pixeltable/pixeltable/pull/912))
- Multi-phase `drop_table()` ([#932](https://github.com/pixeltable/pixeltable/pull/932))
- Tigris integration ([#935](https://github.com/pixeltable/pixeltable/pull/935))
- Circularity detection in view creation ([#942](https://github.com/pixeltable/pixeltable/pull/942))
- Array column filter fixes ([#941](https://github.com/pixeltable/pixeltable/pull/941))
- `t.sample()` on externalized arrays ([#945](https://github.com/pixeltable/pixeltable/pull/945))

#### v0.4.24 (November 12, 2025)

- Hyphens allowed in table/dir names ([#926](https://github.com/pixeltable/pixeltable/pull/926))
- Skip redundant replica downloads ([#927](https://github.com/pixeltable/pixeltable/pull/927))
- Data sharing improvements ([#928](https://github.com/pixeltable/pixeltable/pull/928))
- `drop_table()` bug fix ([#930](https://github.com/pixeltable/pixeltable/pull/930))

#### v0.4.23 (November 11, 2025)

- Live replica `pull()` implementation ([#917](https://github.com/pixeltable/pixeltable/pull/917))
- `fps` greater than video framerate in FrameIterator ([#918](https://github.com/pixeltable/pixeltable/pull/918))
- Create/insert from existing Table ([#919](https://github.com/pixeltable/pixeltable/pull/919))
- Time travel for view over snapshot ([#924](https://github.com/pixeltable/pixeltable/pull/924))
- Proper embedding display ([#925](https://github.com/pixeltable/pixeltable/pull/925))

#### v0.4.22 (November 04, 2025)

- Metadata management refactor ([#913](https://github.com/pixeltable/pixeltable/pull/913))

#### v0.4.21 (November 03, 2025)

- Publishing older table versions hotfix ([#910](https://github.com/pixeltable/pixeltable/pull/910))

#### v0.4.20 (November 03, 2025)

- PySceneDetect UDFs ([#899](https://github.com/pixeltable/pixeltable/pull/899))
- Reve.com UDFs ([#901](https://github.com/pixeltable/pixeltable/pull/901))
- Replica operations protocol ([#819](https://github.com/pixeltable/pixeltable/pull/819))
- `push()` and `pull()` implementations ([#907](https://github.com/pixeltable/pixeltable/pull/907))
- Index creation refactor ([#908](https://github.com/pixeltable/pixeltable/pull/908))
- Snapshot query fixes ([#895](https://github.com/pixeltable/pixeltable/pull/895))

#### v0.4.19 (October 29, 2025)

- Azure storage support ([#886](https://github.com/pixeltable/pixeltable/pull/886))
- Backblaze B2 integration ([#840](https://github.com/pixeltable/pixeltable/pull/840))
- TwelveLabs Embed API ([#885](https://github.com/pixeltable/pixeltable/pull/885))
- Default media destination config ([#883](https://github.com/pixeltable/pixeltable/pull/883))
- Multi-phase `create_table()` ([#854](https://github.com/pixeltable/pixeltable/pull/854))
- Audio encoding UDF ([#881](https://github.com/pixeltable/pixeltable/pull/881))
- Python 3.14 dependency updates ([#894](https://github.com/pixeltable/pixeltable/pull/894))
- Error message consistency improvements ([#869](https://github.com/pixeltable/pixeltable/pull/869), [#891](https://github.com/pixeltable/pixeltable/pull/891))
- `Optional[T]` → `T | None` migration ([#888](https://github.com/pixeltable/pixeltable/pull/888))

#### v0.4.18 (October 22, 2025)

- WhisperX >=3.7 with Python 3.13 ([#860](https://github.com/pixeltable/pixeltable/pull/860))
- `elements` parameter for DocSplitter ([#865](https://github.com/pixeltable/pixeltable/pull/865))
- `numpy>=2.2` enforcement ([#872](https://github.com/pixeltable/pixeltable/pull/872))
- Randomized `sample()` behavior ([#828](https://github.com/pixeltable/pixeltable/pull/828))
- Mintlify documentation deployment ([#867](https://github.com/pixeltable/pixeltable/pull/867))
- Index reconstruction for replicas ([#875](https://github.com/pixeltable/pixeltable/pull/875))
- Hugging Face Auto model UDFs ([#870](https://github.com/pixeltable/pixeltable/pull/870))

#### v0.4.17 (October 16, 2025)

- Backblaze B2 S3-compatible storage ([#840](https://github.com/pixeltable/pixeltable/pull/840))
- OpenRouter notebook ([#851](https://github.com/pixeltable/pixeltable/pull/851))
- ffmpeg with libx264 ([#855](https://github.com/pixeltable/pixeltable/pull/855))
- Embedding indices in packager ([#841](https://github.com/pixeltable/pixeltable/pull/841))
- VideoSplitter 'accurate' mode ([#856](https://github.com/pixeltable/pixeltable/pull/856))
- PDF page-chunk-extractor for images ([#850](https://github.com/pixeltable/pixeltable/pull/850))
- pixeltable-pgserver 0.4.0 ([#853](https://github.com/pixeltable/pixeltable/pull/853))

---

### CATEGORIZED CHANGES

**New Integrations**

- Voyage AI ([#962](https://github.com/pixeltable/pixeltable/pull/962), [#978](https://github.com/pixeltable/pixeltable/pull/978))
- Fal.ai ([#959](https://github.com/pixeltable/pixeltable/pull/959))
- TwelveLabs ([#987](https://github.com/pixeltable/pixeltable/pull/987), [#885](https://github.com/pixeltable/pixeltable/pull/885))
- Reve.com ([#901](https://github.com/pixeltable/pixeltable/pull/901))
- OpenRouter ([#851](https://github.com/pixeltable/pixeltable/pull/851))
- Tigris ([#935](https://github.com/pixeltable/pixeltable/pull/935))
- Azure Blob Storage ([#886](https://github.com/pixeltable/pixeltable/pull/886))
- Backblaze B2 ([#840](https://github.com/pixeltable/pixeltable/pull/840))

**Data Sharing**

- push/pull ([#907](https://github.com/pixeltable/pixeltable/pull/907), [#917](https://github.com/pixeltable/pixeltable/pull/917))
- Publishing protocol ([#819](https://github.com/pixeltable/pixeltable/pull/819), [#944](https://github.com/pixeltable/pixeltable/pull/944), [#948](https://github.com/pixeltable/pixeltable/pull/948))
- Replica packaging ([#841](https://github.com/pixeltable/pixeltable/pull/841))
- Documentation ([#931](https://github.com/pixeltable/pixeltable/pull/931))

**API Changes**

- DataFrame → Query ([#902](https://github.com/pixeltable/pixeltable/pull/902)) **BREAKING**
- StorageDestination → StorageTarget ([#947](https://github.com/pixeltable/pixeltable/pull/947))

**Performance & Scalability**

- Multi-phase operations ([#854](https://github.com/pixeltable/pixeltable/pull/854), [#932](https://github.com/pixeltable/pixeltable/pull/932))
- Improved rate limiting ([#912](https://github.com/pixeltable/pixeltable/pull/912), [#922](https://github.com/pixeltable/pixeltable/pull/922))
- Database schemas isolation ([#953](https://github.com/pixeltable/pixeltable/pull/953))

**Video/Audio**

- FrameIterator keyframes_only ([#934](https://github.com/pixeltable/pixeltable/pull/934))
- FrameIterator fps flexibility ([#918](https://github.com/pixeltable/pixeltable/pull/918))
- PySceneDetect UDFs ([#899](https://github.com/pixeltable/pixeltable/pull/899))
- VideoSplitter accurate mode ([#856](https://github.com/pixeltable/pixeltable/pull/856))
- Audio encoding UDF ([#881](https://github.com/pixeltable/pixeltable/pull/881))
- Gemini video sound ([#973](https://github.com/pixeltable/pixeltable/pull/973))

**Type System**

- UUID type ([#979](https://github.com/pixeltable/pixeltable/pull/979))
- More numpy dtypes ([#940](https://github.com/pixeltable/pixeltable/pull/940))
- numpy>=2.2 ([#872](https://github.com/pixeltable/pixeltable/pull/872))

**Document Processing**

- PDF page-chunk-extractor ([#850](https://github.com/pixeltable/pixeltable/pull/850))
- DocSplitter elements parameter ([#865](https://github.com/pixeltable/pixeltable/pull/865))

**Quality & Developer Experience**

- Error message improvements ([#869](https://github.com/pixeltable/pixeltable/pull/869), [#891](https://github.com/pixeltable/pixeltable/pull/891))
- Development guide ([#958](https://github.com/pixeltable/pixeltable/pull/958))
- Documentation restructuring ([#982](https://github.com/pixeltable/pixeltable/pull/982))
- Mintlify docs ([#867](https://github.com/pixeltable/pixeltable/pull/867))

---

### LINKS TO REFERENCE

**Documentation**

- Data Sharing: https://docs.pixeltable.com/release/platform/data-sharing.ipynb
- Deployment Guide: https://docs.pixeltable.com/platform/deployment
- UUID Cookbook: https://pixeltable-dev.mintlify.app/howto/cookbooks/core/workflow-uuid-identity
- Voyage AI Tutorial: https://docs.pixeltable.com/howto/providers/working-with-voyageai
- Reve Tutorial: https://docs.pixeltable.com/howto/providers/working-with-reve

**GitHub**

- Full changelog: https://github.com/pixeltable/pixeltable/compare/v0.4.16...v0.5.5
- All PRs: https://github.com/pixeltable/pixeltable/pull/{number}
