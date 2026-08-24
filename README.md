# Brainwave Audio API — Public Reference

This public repository intentionally contains no source code, model files, test data, configuration files, credentials, internal reports, or build artefacts. Its only media asset is the explicitly provided five-minute MP3 demonstration listed below.

# PART I — ENGLISH

## 1. Project: capabilities, methods, modes, and advantages

Brainwave Audio Engine is an offline-first audio-processing system for carefully weaving timing- and phase-based listening layers into an existing musical work. It is designed for musical material, not only for generated tones: the engine first measures the source, builds a time-indexed musical workmap, plans where a method can be placed, renders the selected layers, and produces machine-readable evidence of what was rendered.

The project is an audio-production tool. It does not diagnose, treat, prevent, or guarantee any neurological, medical, psychological, sleep, meditation, learning, or consciousness-related outcome. A detected modulation is evidence of an acoustic pattern, not proof of origin, intent, efficacy, or listener perception.

### 1.1 What the engine can do

| Capability | What it provides |
|---|---|
| Input handling | Imports mono or stereo WAV, FLAC, and MP3 from an uploaded audio body; input files are never addressed by a client file path. |
| Output handling | Publishes verified WAV 24-bit PCM, FLAC 24-bit lossless, or MP3 CBR 256 kb/s output. MP3 publication is checked by `ffprobe`. |
| Source preservation | Keeps the dry source as a distinct path; rendering does not reconstruct the musical source through a lossy analysis/synthesis chain. |
| Musical workmap | Extracts timing, loudness, spectrum, transients, chroma, harmonic/key hypotheses, tempo hypotheses, phrases, sections, macro form, recurring motifs, stereo coherence, ILD/IPD, and safe/protected regions. |
| Scene-aware placement | Scores musical opportunities per time and frequency band, determines permitted methods and depth, and can hard-bypass protected regions. |
| Carrier planning | Chooses harmonically quantized carriers under frequency bounds, preferred range, carrier-slew, minimum-hold, whistle-avoidance, and key-lock constraints. |
| Whole-composition automation | Supports a rate, carrier, depth, and pattern program across ordered composition sections; the rate and carrier can change during the track. |
| Three-bus rendering | Separates music spatial treatment, speaker-safe common-room treatment, and strict/direct headphone binaural treatment. |
| Multiple techniques | Supports strict binaural, monaural, isochronic, AM, FM, PM, native-music binaural, partial twin, and push-pull ILD methods. |
| Playback modes | Supports `headphones`, `speakers`, and `hybrid`, with incompatible headphone-only methods rejected for speaker-only output. |
| Layer verification | Produces aggregate and leave-one-out render comparisons, scene-guard results, plan and scene hashes, routing assertions, decoded-output checks, and safety metrics. |
| Reproducibility | A render plan and its hash record the selected topology, timelines, carrier decisions, and guard state before rendering. |

### 1.2 Processing philosophy

The engine treats an existing composition as an active musical canvas. It does not assume that a fixed oscillator at a fixed gain is appropriate throughout the work. Its analysis and planning pipeline can adapt the placement depth, selected frequency band, carrier, and rate to musical form and local texture.

The intended hierarchy is:

1. Preserve the musical source and its intelligibility.
2. Respect hard protected regions and low-confidence areas.
3. Place the selected mechanisms where the source has adequate masking or musical support.
4. Keep route-specific processing separate until the intended final bus point.
5. Verify that the decoded published file contains the requested causal contribution.

For instrumental music, the caller may request full-canvas eligibility. For vocal material, conservative vocal and transient guards can be enabled; strict vocal bypass means an eligible protected frame receives exactly zero added contribution from guarded scene-aware methods.

### 1.3 Audio-analysis features

The scene analyzer uses one sample-index timebase and computes a layered description rather than a single loudness curve. Available features include short-time spectral information, ERB-style bands, chroma, onset and transient evidence, mid/side information, local energy, tempo and local-tempo hypotheses, harmonic/key hypotheses, phrase and section boundaries, repeated motifs, stereo coherence, interaural level difference (ILD), interaural phase difference (IPD), common-envelope candidates, and estimated instrumental roles.

The analyzer produces the following control concepts for each relevant time/band region:

| Control concept | Meaning |
|---|---|
| Opportunity score | Conservative estimate of whether a region can accept a subtle layer. |
| Protected region | Region guarded because of vocal/unknown probability, transient risk, or another configured protection rule. |
| Allowed methods | Methods that the scene policy permits in that local region. |
| Allowed depth | Upper depth allowed by the scene policy in that local region. |
| Band activity | Per-band eligibility; it does not mean every musical frequency is modified. |
| Carrier candidates | Musically constrained carrier choices offered to the global planner. |

The built-in vocal/unknown score is a conservative guard, not a source-separation system and not an identity or provenance detector. An external local vocal mask may be supplied only in a trusted deployment context.

### 1.4 Rendering architecture

The renderer uses three logical buses:

| Bus | Purpose | Typical methods |
|---|---|---|
| `music_spatial_bus` | Dry music plus optional spatial/mid-side treatment. | Spatial treatment and the preserved musical source. |
| `speaker_safe_bus` | Common-mode, room-compatible contribution for loudspeakers. | Monaural, isochronic, AM, FM, PM, and supported push-pull ILD behaviour. |
| `direct_binaural_bus` | Direct, stereo-dependent contribution added after spatial treatment. | Strict binaural, native-music binaural, partial twin, headphone-specific push-pull behaviour. |

The direct binaural bus is deliberately inserted after spatial processing. This preserves the intended left/right relationship of strict or native binaural methods. The render manifest and proof report expose this routing decision.

### 1.5 Available methods

All method instances have a shared control vocabulary: enabled state, depth, spectral band, gain, rate, and optional section-level scheduling. The actual valid controls are exposed by the live schema endpoint.

| Method | Acoustic mechanism | Intended playback target |
|---|---|---|
| `binaural` | Opposed fractional-delay/phase residual around a carrier pair. The two channels represent carrier minus half the beat rate and carrier plus half the beat rate. | `headphones`, `hybrid` |
| `monaural` | Common, selected-band envelope target; no precise stereo dependency. | `speakers`, `hybrid`, `headphones` |
| `isochronic` | Zero-mean RC/Gaussian/smoothed-rectified pulse-like envelope in a selected band. | All targets |
| `am` | Zero-mean amplitude modulation in a selected band. | All targets |
| `fm` | Fractional-delay micro-FM with configured deviation in hertz. | All targets |
| `pm` | Fractional-delay phase modulation with configured phase delta in radians. | All targets |
| `native_binaural` | Analytic music-derived material shifted by exactly minus/plus one half of the requested rate. | `headphones`, `hybrid` |
| `partial_twin` | Envelope-shaped harmonic carrier pairs with a constant left/right difference. | `headphones`, `hybrid` |
| `push_pull_ild` | Constant-power interaural level movement plus common slow AM; stereo mode is channel-dependent and mono input is safely downgraded to common AM. | All targets |

For a `speakers` target, the API rejects the stereo-dependent `binaural`, `native_binaural`, and `partial_twin` methods instead of silently pretending that they are speaker-equivalent. `push_pull_ild` on a mono source is downgraded to its common-mode component.

### 1.6 Natural material modes

The music-derived methods are not limited to a static pure tone. Native material can be selected from `analytic_texture`, `late_reverb`, or `noise_residual` modes. `late_reverb` is stateful and uses a short feedback-tail process; `noise_residual` uses decorrelated prediction residual material. The Hilbert/analytic control quality can be selected from supported tap lengths. Partial-twin layers support a configurable harmonic count and envelope attack/release behaviour.

This does not make the layer inaudible or universally transparent. It is a controllable way of deriving a contribution whose texture may be more compatible with an existing musical surface than a single exposed sine carrier.

### 1.7 Carrier, rate, and section design

The engine can use a fixed carrier/rate or a composition program. A section program is an ordered, non-overlapping sequence of absolute timeline spans. Each span can specify rate, carrier, depth, and pattern values. The global scene planner then constrains carrier choice by the configured policy:

| Carrier-policy control | Role |
|---|---|
| Minimum / maximum carrier | Keeps the carrier inside a permitted frequency range. |
| Preferred carrier | Biases planning toward a chosen musical range. |
| Harmonic key lock | Prefer harmonically quantized candidates when a key hypothesis is usable. |
| Whistle avoidance | Penalizes or avoids conspicuous carrier choices. |
| Maximum slew | Limits carrier movement over time. |
| Minimum hold | Prevents rapid carrier hopping. |
| Semitone and exposure penalties | Discourage musically implausible or overly exposed changes. |

The planner uses bounded global optimization across the whole track rather than independent frame-by-frame choice. Hard protection always wins. A low-confidence `hold` policy can retain a previous decision only where doing so does not violate a hard bypass.

### 1.8 Quality, safety, and proof features

The engine applies one fixed peak-gain decision against the worst configured render variant. It does not use a post-render compressor, dynamic limiter, or waveshaper as a substitute for safe planning. Safety measurements use internal gated K-weighted loudness and oversampled, band-limited peak estimates; they are useful engineering metrics, not a claim of formal BS.1770 certification.

Verification can include:

| Verification item | What it establishes |
|---|---|
| Decoded output checks | The published artefact decodes and matches the requested output format. |
| Aggregate comparison | Difference between the fully enabled weave and an identically mastered all-weave-off variant. |
| Leave-one-out comparison | A causal contribution for every individually enabled method, with that method disabled while other settings remain fixed. |
| Scene guard | Protected-frame count, residual maximum, RMS/peak information, zero-bypass assertion, scene hashes, plan hash, and frame count. |
| Routing assertion | The direct binaural path was placed after spatial processing. |
| Plan manifest | Selected topology, method settings, scheduled values, and plan hash. |
| Master safety | Peak/headroom and render-variant safety information. |

Codec differences alone do not prove a requested method survived. Aggregate comparison can show a shared causal ambiguity. For strong per-method evidence, request leave-one-out verification and preserve the output, QA report, and manifest together.

### 1.9 Demonstration audio

The repository includes one five-minute MP3 demo, prepared from an audio source explicitly supplied for this public demonstration. It is encoded as stereo MP3 CBR 256 kb/s at 44.1 kHz and is provided solely as a listening example of the delivery format. It does not establish listener outcomes or replace the API proof workflow.

Demo file: [`demo/leberch-fantasy-ambient-5min-demo-256kbps.mp3`](demo/leberch-fantasy-ambient-5min-demo-256kbps.mp3)

## 2. API v2 — complete reference

### 2.1 Service contract

| Item | Value |
|---|---|
| Default base URL | `http://127.0.0.1:8765` |
| API version | `v2` |
| Transport | HTTP JSON, raw uploaded audio body, binary audio downloads, and SSE job snapshots. |
| Default network boundary | Loopback-only service. |
| Authentication | None is built into the local service. Do not expose it directly to an untrusted network; put authentication and transport protection in a trusted reverse proxy if remote access is required. |
| CORS | Controlled by the deployment's `BRAINWAVE_CORS_ORIGIN` setting. |
| Data root | Controlled by `BRAINWAVE_DATA_DIR`; it remains a server-side deployment concern, not an API client path. |
| External media tools | `FFMPEG_BIN` and `FFPROBE_BIN` may be configured by the operator. |

Every JSON error has the shape `{"error":"human-readable message"}`. Common status codes are `200` success, `201` source created, `202` asynchronous job accepted, `206` byte-range response, `400` invalid request/configuration, `404` unknown resource, and `416` invalid byte range.

### 2.2 Discovery and service endpoints

| Method and path | Purpose | Successful response |
|---|---|---|
| `GET /api/v2/health` | Service liveness and basic readiness. | `200` health object. |
| `GET /api/v2/capabilities` | Supported input/output formats, methods, targets, limits, and feature availability. | `200` capabilities object. |
| `GET /api/v2/schema/session` | Canonical live `SessionConfig` schema, defaults, enum values, and validation constraints. | `200` schema object. |

Clients should read capabilities and schema at startup instead of hard-coding a potentially stale feature list.

### 2.3 Source upload and source retrieval

#### Create a source

| Method and path | Request | Response |
|---|---|---|
| `POST /api/v2/sources` | Raw WAV, FLAC, or MP3 bytes in the HTTP body; this is not multipart form upload. Include `X-Filename`. The maximum accepted upload size is 2 GiB. | `201` source record containing `source_id`, technical metadata, `content_sha256`, and normalized render-source identity. |

The service accepts mono or stereo files. WAV and FLAC are handled as lossless source material. MP3 is decoded once into verified render WAV; the original content hash and the render-source hash remain distinguishable. Metadata preservation is conservative and format-dependent.

#### Read a source

| Method and path | Purpose |
|---|---|
| `GET /api/v2/sources` | Lists stored source records. |
| `GET /api/v2/sources/{source_id}` | Returns source metadata and identities. |
| `GET /api/v2/sources/{source_id}/audio` | Streams the uploaded/normalized source audio; supports HTTP byte ranges and may return `206`. |
| `GET /api/v2/sources/{source_id}/analysis` | Returns cached analysis when available. |
| `GET /api/v2/sources/{source_id}/music-map` | Returns the time-indexed musical workmap. |
| `GET /api/v2/sources/{source_id}/music-scene` | Returns the higher-level scene graph and guard decisions. |
| `GET /api/v2/sources/{source_id}/modulation-detection` | Returns measured candidate modulation evidence already present in the source. |

Detection results must be treated as candidates and measurements, not as attribution or proof that a historical file was intentionally produced with a specific method.

### 2.4 Analyze a source

| Method and path | Required field | Optional scene-analysis object | Response |
|---|---|---|---|
| `POST /api/v2/analyze` | `source_id` | `scene_analysis` | `202` analysis job or accepted analysis state. |

The `scene_analysis` object supports the following fields. Exact accepted values and defaults are always available from `GET /api/v2/schema/session`.

| Field | Meaning |
|---|---|
| `enabled` | Enables scene-aware analysis. |
| `profile` | Analysis workload profile: `fast`, `quality`, or `ultra`. |
| `analysis_hop_ms` | Analysis hop in milliseconds. |
| `protect_vocals` | Enables vocal-probability protection. |
| `strict_vocal_bypass` | Forces guarded scene-aware methods to zero in protected frames. |
| `protect_transients` | Enables transient guards. |
| `vocal_probability_threshold` | Threshold for vocal protection. |
| `unknown_probability_threshold` | Threshold for uncertain/unknown protection. |
| `transient_probability_threshold` | Threshold for transient protection. |
| `pre_guard_ms`, `post_guard_ms` | Guard expansion before and after a protected event. |
| `transition_ms` | Transition duration around eligible regions. |
| `minimum_opportunity_score` | Minimum local opportunity score needed for a scene-aware method. |
| `full_canvas_depth_floor` | Minimum permitted depth when full-canvas instrumental eligibility is selected. |
| `external_vocal_mask_path` | Trusted-local-deployment path to an NPZ mask. It is not suitable for an untrusted remote client. |

An external NPZ mask must contain `vocal_probability` and may contain `unknown_probability`. The engine interpolates and clips it to the analysis timeline and caches its hash. A remote deployment should replace this local-path field with an authenticated server-side upload/reference mechanism; it must never accept arbitrary file paths from untrusted clients.

For predominantly instrumental content, a caller can use `protect_vocals=false`, `protect_transients=false`, `minimum_opportunity_score=0`, and a positive `full_canvas_depth_floor` such as `0.06`. Full eligibility is visible only when `active_fraction=1` and every reported `band_active_fraction=1`.

### 2.5 Build a preflight render plan

| Method and path | Request | Response |
|---|---|---|
| `POST /api/v2/plan` | A valid `RenderRequest` containing a `SessionConfig`. | `200` `RenderPlan`, `MusicMap`, `MusicScene`, and `SceneEmbeddingPlan`. |

Preflight does not publish audio. It records intended topology, selected methods, schedule, scene guard state, carrier decisions, and `plan_hash`. Persist the hash with a production job so the rendered artefact can be matched to the intended plan.

### 2.6 Start a render or preview

| Method and path | Request | Result |
|---|---|---|
| `POST /api/v2/render` | A valid `RenderRequest`. | `202` render job. The job analyses when required, renders, runs mandatory QA, and atomically publishes only on success. |
| `POST /api/v2/preview` | A valid `RenderRequest`. | `202` preview job. Preview uses up to 96,000 source frames and creates original/delta preview artefacts. |

The service allows at most two active renders and two active analyses. A process restart marks active jobs `interrupted`; it does not resume them automatically.

### 2.7 Job lifecycle, status, evidence, and artefacts

| Method and path | Purpose |
|---|---|
| `GET /api/v2/jobs/{job_id}` | Current job state, progress, result fields, and terminal error when present. |
| `GET /api/v2/jobs/{job_id}/events` | SSE endpoint; emits a current snapshot and is safe to reconnect to. Clients may also poll the status endpoint. |
| `DELETE /api/v2/jobs/{job_id}` | Requests cancellation. |
| `GET /api/v2/jobs/{job_id}/analysis` | Analysis attached to the job/source. |
| `GET /api/v2/jobs/{job_id}/proof` | Structured verification/proof report. |
| `GET /api/v2/jobs/{job_id}/artifacts/output` | Published audio only after successful mandatory QA. |
| `GET /api/v2/jobs/{job_id}/artifacts/qa` | QA report; failed mandatory QA is retained as a failed QA record. |
| `GET /api/v2/jobs/{job_id}/artifacts/manifest` | Render manifest including plan and routing information. |
| `GET /api/v2/jobs/{job_id}/artifacts/preview/original` | Preview original artefact, when a preview was requested. |
| `GET /api/v2/jobs/{job_id}/artifacts/preview/delta` | Preview difference artefact, when a preview was requested. |

Job states are `queued`, `running`, `complete`, `failed`, `cancelled`, and `interrupted`. A render request can be accepted with `202` and later end in `failed`. If mandatory QA fails, publication is false: there is no final output or final manifest, but the failed QA record remains available. A failure before QA produces the job error instead.

### 2.8 RenderRequest and SessionConfig

`RenderRequest` contains a `SessionConfig`. The canonical schema is live at `GET /api/v2/schema/session`; use it as the authoritative contract. The root configuration fields are:

| Field | Purpose |
|---|---|
| `schema_version` | Current API configuration version, currently `1.0`. |
| `source_id` | Server-issued identifier returned by source upload. |
| `output_format` | Requested published format: WAV 24-bit PCM, FLAC 24-bit lossless, or MP3 CBR 256 kb/s. |
| `source_gain_db`, `output_gain_db` | Explicit gain controls. |
| `binaural`, `binaural_automation`, `carrier_automation` | Strict binaural controls and its time automation. |
| `monaural` | Monaural/common-mode method settings. |
| `isochronic`, `isochronic_automation` | Isochronic method and its automation. |
| `noise`, `noise_automation` | Noise/material layer settings and its automation. |
| `ambient` | Ambient layer settings. |
| `mixer` | Relative mix and routing controls. |
| `spatial` | Music spatial/mid-side controls. |
| `safety` | Peak/headroom and safety policy. |
| `multimodal` | Multi-method scene-aware program and verification policy. |
| `output_metadata` | Output metadata policy. |
| `module_instances` | Reserved for a future module model; non-empty values are rejected. |

Automation points must be strictly increasing in time and must satisfy the applicable Nyquist and range constraints. The strict binaural relation is fixed: left carrier equals carrier minus half the beat rate; right carrier equals carrier plus half the beat rate. The direct binaural component is mixed after spatial treatment.

### 2.9 Multimodal configuration

`multimodal` is active only when `enabled=true` and at least one nested method is enabled. It contains these major groups:

| Group | Purpose |
|---|---|
| `playback_target` | `headphones`, `speakers`, or `hybrid`. |
| `carrier_policy` | Minimum, maximum, preferred carrier, key lock, whistle avoidance, maximum slew, and hold/planning constraints. |
| `section_program` | Ordered non-overlapping timeline spans for rate, carrier, depth, and pattern values. |
| `multiband` | Five-band crossovers and filter order. |
| `scene_analysis` | Analyzer and guard settings described above. |
| `scene_planner` | Global carrier and scene-placement policy. |
| `maximum_composite_delta_db` | Bound on combined intervention depth. |
| `fade_seconds` | Render-edge fade setting. |
| `verification` | Verification enabled state and `aggregate` or `leave_one_out` mode. |

Available logical bands are `sub`, `low`, `mid`, `presence`, `air`, and `full`. A method consumes its assigned band; it does not claim to alter all frequency content merely because the overall scene is full-canvas eligible.

### 2.10 Method-specific controls

| Method | Key controls beyond the shared vocabulary |
|---|---|
| `binaural` | Carrier, beat/rate, direct mix, and automation under strict left/right carrier relation. |
| `monaural` | Common selected-band envelope target and smoothing/shape controls exposed by schema. |
| `isochronic` | Pulse shape, envelope depth, and rate/automation controls. |
| `am` | Modulation depth, shape, rate, selected band, and gain. |
| `fm` | Frequency deviation in hertz, rate, selected band, and gain. |
| `pm` | Phase delta in radians, rate, selected band, and gain. |
| `native_binaural` | Material mode, analytic/Hilbert tap quality, strict minus/plus half-rate shift, selected band, and gain. |
| `partial_twin` | Harmonic count from 1 through 8, envelope attack/release, carrier relation, selected band, and gain. |
| `push_pull_ild` | ILD amount from 0 through 3 dB, constant-power movement, and common slow-AM controls. |

### 2.11 Section programs and time-varying design

A section program uses absolute composition time, not a relative loop counter. Spans must be ordered and must not overlap. More transitions can be distributed across the whole work by defining more consecutive spans. Each span can independently supply a target rate, carrier range/value, depth, and pattern choice, while the carrier policy still limits illegal or overly abrupt carrier movement.

This makes it possible to begin with one rate and finish with another without forcing a final reorientation segment. It also permits carrier ranges to change with the musical sections so that a layer remains lower, less exposed, or more harmonically compatible with the current material.

### 2.12 Proof and quality contract

The proof object can contain aggregate comparison, leave-one-out per-method results, scene guard, render-plan evidence, routing assertion, decoder/codec verification, and master-safety metrics. Interpret them as follows:

| Proof item | Correct interpretation |
|---|---|
| Aggregate | The full enabled weave differs from an identically mastered disabled weave; it does not isolate each method. |
| Leave-one-out | Disabling one enabled method gives evidence for that individual method's causal contribution. |
| Scene guard | Confirms guard accounting and zero residual where strict bypass applies; it does not certify artistic transparency. |
| Decoded codec verification | Confirms the published output survives decoding and meets the requested technical format. |
| Plan hash and manifest | Tie output evidence to a specific planned topology and schedule. |
| Loudness/peak metrics | Engineering safety indicators, not a substitute for formal broadcast conformance, PEAQ, MUSHRA, or ABX listening tests. |

For production acceptance, use leave-one-out verification when individual-method evidence matters, listen on the intended playback target, and archive the audio with its QA report and manifest.

### 2.13 Compatibility endpoints

The service also maintains compatibility routes under `/api/...` for existing clients, including legacy render, preview, source, project/catalog, graph, migration, validation, and compile workflows. New integrations should use `/api/v2/...` and the live schema/capabilities endpoints.

### 2.14 Recommended client workflow

1. Query `GET /api/v2/capabilities` and `GET /api/v2/schema/session`.
2. Upload one WAV, FLAC, or MP3 file with `POST /api/v2/sources`; retain the returned `source_id` and hashes.
3. Run `POST /api/v2/analyze` when you need scene-aware placement, then inspect the source analysis, workmap, scene, and modulation-detection results.
4. Choose `headphones`, `speakers`, or `hybrid`; do not request headphone-only methods for a speaker-only target.
5. Configure strict protection before natural/music-derived methods when vocals or transients must remain untouched.
6. For instrumental full-canvas work, explicitly select the associated scene policy; do not infer full eligibility from a single global flag.
7. Send `POST /api/v2/plan`, review the plan and store `plan_hash`.
8. Send `POST /api/v2/render` or `POST /api/v2/preview`, then poll job status or reconnect to SSE.
9. Wait for a terminal state; never treat `202` as proof that audio was published.
10. On `complete`, archive output, QA report, manifest, and proof together. Use leave-one-out verification when needed.
11. Use FLAC/WAV when no additional delivery-codec loss is acceptable; select MP3 only when the verified CBR 256 kb/s deliverable is desired.

### 2.15 Contacts

Telegram: [@ctepx](https://t.me/ctepx)  
Email: [ctepx2001@gmail.com](mailto:ctepx2001@gmail.com)

---

# ЧАСТЬ II — РУССКИЙ

## 1. Проект: возможности, методы, режимы и преимущества

Brainwave Audio Engine — офлайн-ориентированная система обработки аудио для аккуратного вплетения во временную и фазовую структуру уже существующего музыкального произведения. Она рассчитана на работу именно с музыкальным полотном, а не только с отдельно сгенерированными тонами: сначала система измеряет исходник, строит привязанную ко времени музыкальную рабочую карту, планирует места применения метода, рендерит выбранные слои и формирует машиночитаемые доказательства того, что было отрендерено.

Проект является инструментом аудиопроизводства. Он не диагностирует, не лечит, не предотвращает и не гарантирует неврологический, медицинский, психологический эффект, эффект сна, медитации, обучения или изменения состояния сознания. Найденная модуляция является свидетельством акустического паттерна, но не доказательством происхождения, намерения автора, эффективности или восприятия слушателем.

### 1.1 Что умеет движок

| Возможность | Что она даёт |
|---|---|
| Приём входа | Импортирует моно- или стереофайлы WAV, FLAC и MP3 из загруженного аудиотела; клиент не передаёт путь к файлу. |
| Выходные форматы | Публикует проверенный WAV PCM 24-bit, FLAC lossless 24-bit или MP3 CBR 256 кбит/с. Перед публикацией MP3 проверяется через `ffprobe`. |
| Сохранность исходника | Хранит сухой источник как отдельный путь; рендер не пересобирает музыкальный источник через цепочку с потерями анализа/синтеза. |
| Музыкальная рабочая карта | Извлекает тайминг, громкость, спектр, транзиенты, хрому, гипотезы тональности/гармонии, гипотезы темпа, фразы, секции, макроформу, повторяющиеся мотивы, стереокогерентность, ILD/IPD и безопасные/защищённые области. |
| Размещение по сцене | Оценивает музыкальные возможности для каждого времени и частотной полосы, определяет разрешённые методы и глубину и может жёстко обходить защищённые области. |
| Планирование несущей | Выбирает гармонически квантизованные несущие с учётом границ частоты, предпочитаемого диапазона, скорости изменения несущей, минимального удержания, избегания свиста и привязки к тональности. |
| Автоматизация всей композиции | Поддерживает программу скорости, несущей, глубины и паттерна для упорядоченных секций композиции; скорость и несущая могут меняться по ходу трека. |
| Рендер с тремя шинами | Разделяет пространственную обработку музыки, безопасную для колонок общекомнатную обработку и строгую/прямую наушниковую бинауральную обработку. |
| Несколько техник | Поддерживает строгую бинауральную, моноауральную, изохронную, AM, FM, PM, native-music binaural, partial twin и push-pull ILD методы. |
| Режимы воспроизведения | Поддерживает `headphones`, `speakers` и `hybrid`; несовместимые наушниковые методы отклоняются для режима только колонок. |
| Проверка слоёв | Формирует aggregate и leave-one-out сравнения рендера, результаты scene guard, хеши плана и сцены, утверждения маршрутизации, проверки декодированного вывода и метрики безопасности. |
| Воспроизводимость | План рендера и его хеш фиксируют выбранную топологию, временные линии, решения по несущей и состояние защит до рендера. |

### 1.2 Принцип обработки

Движок рассматривает существующую композицию как активное музыкальное полотно. Он не предполагает, что фиксированный осциллятор с фиксированным усилением одинаково уместен во всём произведении. Анализ и планирование могут адаптировать глубину размещения, выбранную частотную полосу, несущую и скорость к музыкальной форме и локальной фактуре.

Предполагаемая иерархия такова:

1. Сохранить музыкальный источник и его разборчивость.
2. Уважать жёстко защищённые области и области с низкой уверенностью.
3. Размещать выбранные механизмы там, где в источнике достаточно маскирования или музыкальной опоры.
4. Держать обработку, зависящую от маршрута воспроизведения, раздельной до нужной финальной точки шины.
5. Проверять, что декодированный опубликованный файл содержит требуемый причинный вклад.

Для инструментальной музыки вызывающий клиент может запросить допустимость работы по всему полотну. Для вокального материала можно включить консервативные защиты вокала и транзиентов; строгий обход вокала означает, что подходящий защищённый кадр получает ровно нулевой добавленный вклад от защищаемых сценозависимых методов.

### 1.3 Возможности аудиоанализа

Анализатор сцены использует одну временную шкалу в индексах сэмплов и рассчитывает многослойное описание вместо одной кривой громкости. Доступны признаки кратковременного спектра, полосы в стиле ERB, хрома, признаки onset и транзиентов, информация mid/side, локальная энергия, гипотезы темпа и локального темпа, гипотезы гармонии/тональности, границы фраз и секций, повторяющиеся мотивы, стереокогерентность, межканальная разность уровней (ILD), межканальная разность фаз (IPD), кандидаты общей огибающей и оценённые инструментальные роли.

Для каждой значимой области времени/полосы анализатор создаёт следующие управляющие понятия:

| Управляющее понятие | Значение |
|---|---|
| Opportunity score | Консервативная оценка того, может ли область принять тонкий добавочный слой. |
| Protected region | Область, защищённая из-за вероятности вокала/неизвестного материала, риска транзиента или другого правила защиты. |
| Allowed methods | Методы, которые политика сцены разрешает в данной локальной области. |
| Allowed depth | Максимальная глубина, разрешённая политикой сцены в данной локальной области. |
| Band activity | Допустимость по отдельной полосе; это не означает, что изменяется каждая музыкальная частота. |
| Carrier candidates | Музыкально ограниченные варианты несущей, предложенные глобальному планировщику. |

Встроенная оценка вокала/неизвестного материала является консервативной защитой, а не системой разделения источников и не детектором личности или происхождения. Внешняя локальная маска вокала может применяться только в доверенном контексте развёртывания.

### 1.4 Архитектура рендера

Рендерер использует три логические шины:

| Шина | Назначение | Типичные методы |
|---|---|---|
| `music_spatial_bus` | Сухая музыка плюс необязательная пространственная/mid-side обработка. | Пространственная обработка и сохранённый музыкальный источник. |
| `speaker_safe_bus` | Общемодовый, совместимый с помещением вклад для громкоговорителей. | Monaural, isochronic, AM, FM, PM и поддерживаемое push-pull ILD поведение. |
| `direct_binaural_bus` | Прямой стереозависимый вклад, добавляемый после пространственной обработки. | Строгая binaural, native-music binaural, partial twin, наушниковое push-pull поведение. |

Прямая бинауральная шина намеренно вставляется после пространственной обработки. Это сохраняет нужное отношение левого/правого канала для строгих или native бинауральных методов. Манифест рендера и отчёт proof показывают это решение маршрутизации.

### 1.5 Доступные методы

Все экземпляры методов имеют общий словарь управления: состояние включения, глубина, спектральная полоса, усиление, скорость и необязательное расписание по секциям. Точные допустимые параметры публикуются живой схемой API.

| Метод | Акустический механизм | Целевой режим воспроизведения |
|---|---|---|
| `binaural` | Противоположный остаточный эффект дробной задержки/фазы вокруг пары несущих. Каналы представляют несущую минус половина beat rate и несущую плюс половина beat rate. | `headphones`, `hybrid` |
| `monaural` | Общая целевая огибающая выбранной полосы без зависимости от точного стерео. | `speakers`, `hybrid`, `headphones` |
| `isochronic` | Импульсоподобная огибающая с нулевым средним RC/Gaussian/smoothed-rectified в выбранной полосе. | Все режимы |
| `am` | Амплитудная модуляция с нулевым средним в выбранной полосе. | Все режимы |
| `fm` | Микро-FM на дробной задержке с заданной девиацией в герцах. | Все режимы |
| `pm` | Фазовая модуляция на дробной задержке с заданным фазовым сдвигом в радианах. | Все режимы |
| `native_binaural` | Аналитический материал, полученный из музыки и сдвинутый ровно на минус/плюс половину требуемой скорости. | `headphones`, `hybrid` |
| `partial_twin` | Огибающие гармонических пар несущих с постоянной разностью левого/правого канала. | `headphones`, `hybrid` |
| `push_pull_ild` | Межканальное перемещение уровня с постоянной мощностью плюс общая медленная AM; стереорежим зависит от каналов, а моноисточник безопасно понижается до общей AM. | Все режимы |

Для цели `speakers` API отклоняет стереозависимые методы `binaural`, `native_binaural` и `partial_twin`, а не делает вид, что они эквивалентны работе через колонки. `push_pull_ild` на моноисточнике понижается до его общемодовой части.

### 1.6 Режимы естественного материала

Методы, производные от музыки, не ограничены статичным чистым тоном. Native-материал может быть выбран из режимов `analytic_texture`, `late_reverb` или `noise_residual`. `late_reverb` хранит состояние и использует процесс с коротким feedback-tail; `noise_residual` использует декоррелированный остаточный материал предсказания. Качество управления Hilbert/analytic можно выбрать из поддерживаемых длин фильтра. Слои partial-twin поддерживают настраиваемое число гармоник и поведение attack/release огибающей.

Это не делает слой неслышимым или универсально прозрачным. Это управляемый способ получить вклад, фактура которого может быть более совместима с существующей музыкальной поверхностью, чем одиночная явно слышимая синусоидальная несущая.

### 1.7 Несущая, скорость и проектирование секций

Движок может использовать фиксированную несущую/скорость либо программу композиции. Программа секций — это упорядоченная непересекающаяся последовательность абсолютных временных интервалов. Каждый интервал может задавать скорость, несущую, глубину и паттерн. Затем глобальный планировщик сцены ограничивает выбор несущей заданной политикой:

| Управление политикой несущей | Роль |
|---|---|
| Минимальная / максимальная несущая | Удерживает несущую внутри разрешённого частотного диапазона. |
| Предпочитаемая несущая | Смещает планирование к выбранному музыкальному диапазону. |
| Harmonic key lock | Предпочитает гармонически квантизованные варианты, когда гипотеза тональности пригодна. |
| Whistle avoidance | Штрафует или избегает заметно свистящих вариантов несущей. |
| Maximum slew | Ограничивает изменение несущей во времени. |
| Minimum hold | Не допускает быстрых скачков несущей. |
| Штрафы за полутоны и экспозицию | Сдерживают музыкально неубедительные или чрезмерно заметные изменения. |

Планировщик использует ограниченную глобальную оптимизацию по всему треку, а не независимый выбор кадр за кадром. Жёсткая защита всегда имеет приоритет. Политика `hold` при низкой уверенности может удерживать предыдущее решение только там, где это не нарушает жёсткий обход.

### 1.8 Качество, безопасность и доказательства

Движок применяет одно фиксированное решение по пиковому усилению относительно худшего настроенного варианта рендера. Он не использует после рендера компрессор, динамический лимитер или waveshaper как замену безопасному планированию. Измерения безопасности используют внутреннюю gated K-weighted громкость и передискретизированные, ограниченные по полосе оценки пика; это полезные инженерные метрики, но не заявление о формальной сертификации BS.1770.

Проверка может включать:

| Пункт проверки | Что он устанавливает |
|---|---|
| Проверки декодированного вывода | Опубликованный артефакт декодируется и соответствует запрошенному выходному формату. |
| Aggregate comparison | Разницу между полностью включённым weave и идентично смастеренным вариантом со всеми weave-слоями выключенными. |
| Leave-one-out comparison | Причинный вклад каждого отдельно включённого метода: метод выключается при неизменных остальных настройках. |
| Scene guard | Число защищённых кадров, максимум остатка, RMS/peak информацию, утверждение нулевого обхода, хеши сцены, хеш плана и число кадров. |
| Routing assertion | Прямой бинауральный путь помещён после пространственной обработки. |
| Plan manifest | Выбранную топологию, настройки методов, запланированные значения и хеш плана. |
| Master safety | Информацию о пике/запасе и безопасности вариантов рендера. |

Одни только различия кодека не доказывают сохранность требуемого метода. Aggregate comparison может иметь общую причинную неоднозначность. Для сильного доказательства по каждому методу запрашивайте leave-one-out проверку и храните вместе выходной файл, QA-отчёт и манифест.

### 1.9 Демонстрационное аудио

Репозиторий содержит один пятиминутный MP3-демо, подготовленный из аудиоисточника, явно предоставленного для этой публичной демонстрации. Он закодирован как stereo MP3 CBR 256 кбит/с при 44,1 кГц и предоставляется только как пример прослушивания формата доставки. Он не устанавливает эффект для слушателя и не заменяет workflow доказательств API.

Демо-файл: [`demo/leberch-fantasy-ambient-5min-demo-256kbps.mp3`](demo/leberch-fantasy-ambient-5min-demo-256kbps.mp3)

## 2. API v2 — полная справка

### 2.1 Контракт сервиса

| Элемент | Значение |
|---|---|
| Базовый URL по умолчанию | `http://127.0.0.1:8765` |
| Версия API | `v2` |
| Транспорт | HTTP JSON, сырое тело загруженного аудио, бинарное скачивание аудио и SSE-снимки задач. |
| Сетевая граница по умолчанию | Сервис только для loopback. |
| Аутентификация | В локальный сервис не встроена. Не публикуйте его напрямую в недоверенную сеть; для удалённого доступа добавьте аутентификацию и защиту транспорта в доверенном reverse proxy. |
| CORS | Управляется настройкой развёртывания `BRAINWAVE_CORS_ORIGIN`. |
| Корень данных | Управляется `BRAINWAVE_DATA_DIR`; это серверная задача развёртывания, а не путь клиента API. |
| Внешние медиасредства | Оператор может настроить `FFMPEG_BIN` и `FFPROBE_BIN`. |

Любая JSON-ошибка имеет форму `{"error":"human-readable message"}`. Обычные статусы: `200` — успех, `201` — источник создан, `202` — асинхронная задача принята, `206` — ответ диапазона байт, `400` — неверный запрос/конфигурация, `404` — неизвестный ресурс и `416` — неверный диапазон байт.

### 2.2 Обнаружение возможностей и сервисные маршруты

| Метод и путь | Назначение | Успешный ответ |
|---|---|---|
| `GET /api/v2/health` | Жизнеспособность сервиса и базовая готовность. | Объект health с `200`. |
| `GET /api/v2/capabilities` | Поддерживаемые входные/выходные форматы, методы, цели, ограничения и доступность функций. | Объект capabilities с `200`. |
| `GET /api/v2/schema/session` | Каноническая живая схема `SessionConfig`, значения по умолчанию, enum и ограничения валидации. | Объект schema с `200`. |

Клиенты должны читать capabilities и schema при запуске, а не жёстко прошивать потенциально устаревший список функций.

### 2.3 Загрузка и получение источника

#### Создание источника

| Метод и путь | Запрос | Ответ |
|---|---|---|
| `POST /api/v2/sources` | Сырые байты WAV, FLAC или MP3 в HTTP-теле; это не multipart form upload. Добавьте `X-Filename`. Максимальный принимаемый размер загрузки — 2 GiB. | Запись источника с `201`, содержащая `source_id`, технические метаданные, `content_sha256` и нормализованную идентичность источника рендера. |

Сервис принимает моно- и стереофайлы. WAV и FLAC обрабатываются как источники без потерь. MP3 один раз декодируется в проверенный WAV для рендера; исходный content hash и render-source hash остаются различимыми. Сохранение метаданных консервативно и зависит от формата.

#### Чтение источника

| Метод и путь | Назначение |
|---|---|
| `GET /api/v2/sources` | Возвращает список сохранённых записей источников. |
| `GET /api/v2/sources/{source_id}` | Возвращает метаданные и идентичности источника. |
| `GET /api/v2/sources/{source_id}/audio` | Передаёт загруженное/нормализованное аудио источника; поддерживает HTTP byte ranges и может вернуть `206`. |
| `GET /api/v2/sources/{source_id}/analysis` | Возвращает кэшированный анализ, когда он доступен. |
| `GET /api/v2/sources/{source_id}/music-map` | Возвращает привязанную ко времени музыкальную рабочую карту. |
| `GET /api/v2/sources/{source_id}/music-scene` | Возвращает высокоуровневый граф сцены и решения защит. |
| `GET /api/v2/sources/{source_id}/modulation-detection` | Возвращает измеренные кандидаты модуляции, уже присутствующие в источнике. |

Результаты детектирования следует рассматривать как кандидаты и измерения, а не как атрибуцию или доказательство того, что исторический файл был намеренно создан конкретным методом.

### 2.4 Анализ источника

| Метод и путь | Обязательное поле | Необязательный объект scene-analysis | Ответ |
|---|---|---|---|
| `POST /api/v2/analyze` | `source_id` | `scene_analysis` | Задача анализа с `202` или принятое состояние анализа. |

Объект `scene_analysis` поддерживает следующие поля. Точные допустимые значения и значения по умолчанию всегда доступны через `GET /api/v2/schema/session`.

| Поле | Значение |
|---|---|
| `enabled` | Включает анализ с учётом сцены. |
| `profile` | Профиль нагрузки анализа: `fast`, `quality` или `ultra`. |
| `analysis_hop_ms` | Шаг анализа в миллисекундах. |
| `protect_vocals` | Включает защиту по вероятности вокала. |
| `strict_vocal_bypass` | Принудительно обнуляет защищаемые сценозависимые методы в защищённых кадрах. |
| `protect_transients` | Включает защиту транзиентов. |
| `vocal_probability_threshold` | Порог защиты вокала. |
| `unknown_probability_threshold` | Порог защиты неопределённого/неизвестного материала. |
| `transient_probability_threshold` | Порог защиты транзиентов. |
| `pre_guard_ms`, `post_guard_ms` | Расширение защиты до и после защищаемого события. |
| `transition_ms` | Длительность перехода вокруг допустимых областей. |
| `minimum_opportunity_score` | Минимальный локальный opportunity score для сценозависимого метода. |
| `full_canvas_depth_floor` | Минимальная разрешённая глубина при выборе инструментальной допустимости по всему полотну. |
| `external_vocal_mask_path` | Путь к NPZ-маске в доверенном локальном развёртывании. Он не подходит для недоверенного удалённого клиента. |

Внешняя NPZ-маска должна содержать `vocal_probability` и может содержать `unknown_probability`. Движок интерполирует и ограничивает её по временной шкале анализа и кэширует её хеш. Удалённое развёртывание должно заменить поле локального пути на аутентифицированную серверную загрузку/ссылку; оно никогда не должно принимать произвольные пути от недоверенных клиентов.

Для преимущественно инструментального материала клиент может использовать `protect_vocals=false`, `protect_transients=false`, `minimum_opportunity_score=0` и положительный `full_canvas_depth_floor`, например `0.06`. Полная допустимость видна только при `active_fraction=1` и при каждом сообщённом `band_active_fraction=1`.

### 2.5 Построение предварительного плана рендера

| Метод и путь | Запрос | Ответ |
|---|---|---|
| `POST /api/v2/plan` | Валидный `RenderRequest`, содержащий `SessionConfig`. | `RenderPlan`, `MusicMap`, `MusicScene` и `SceneEmbeddingPlan` с `200`. |

Предварительное планирование не публикует аудио. Оно фиксирует предполагаемую топологию, выбранные методы, расписание, состояние защит сцены, решения по несущей и `plan_hash`. Сохраняйте хеш вместе с production-задачей, чтобы опубликованный артефакт можно было сопоставить с предполагаемым планом.

### 2.6 Запуск рендера или preview

| Метод и путь | Запрос | Результат |
|---|---|---|
| `POST /api/v2/render` | Валидный `RenderRequest`. | Задача рендера с `202`. Задача анализирует при необходимости, рендерит, выполняет обязательный QA и атомарно публикует результат только при успехе. |
| `POST /api/v2/preview` | Валидный `RenderRequest`. | Задача preview с `202`. Preview использует до 96 000 кадров источника и создаёт оригинальный/delta preview-артефакты. |

Сервис допускает не более двух активных рендеров и двух активных анализов. Перезапуск процесса помечает активные задачи как `interrupted`; автоматически они не продолжаются.

### 2.7 Жизненный цикл задачи, статус, доказательства и артефакты

| Метод и путь | Назначение |
|---|---|
| `GET /api/v2/jobs/{job_id}` | Текущее состояние задачи, прогресс, поля результата и финальная ошибка при наличии. |
| `GET /api/v2/jobs/{job_id}/events` | SSE-маршрут; передаёт актуальный снимок и безопасен для переподключения. Клиент также может опрашивать маршрут статуса. |
| `DELETE /api/v2/jobs/{job_id}` | Запрашивает отмену. |
| `GET /api/v2/jobs/{job_id}/analysis` | Анализ, прикреплённый к задаче/источнику. |
| `GET /api/v2/jobs/{job_id}/proof` | Структурированный отчёт verification/proof. |
| `GET /api/v2/jobs/{job_id}/artifacts/output` | Опубликованное аудио только после успешного обязательного QA. |
| `GET /api/v2/jobs/{job_id}/artifacts/qa` | QA-отчёт; проваленный обязательный QA сохраняется как failed QA record. |
| `GET /api/v2/jobs/{job_id}/artifacts/manifest` | Манифест рендера, включая информацию о плане и маршрутизации. |
| `GET /api/v2/jobs/{job_id}/artifacts/preview/original` | Оригинальный preview-артефакт, когда запрашивался preview. |
| `GET /api/v2/jobs/{job_id}/artifacts/preview/delta` | Разностный preview-артефакт, когда запрашивался preview. |

Состояния задачи: `queued`, `running`, `complete`, `failed`, `cancelled` и `interrupted`. Запрос рендера может быть принят с `202`, а позднее завершиться как `failed`. Если обязательный QA провален, публикация имеет значение false: нет финального output или финального manifest, но failed QA record остаётся доступным. При ошибке до QA вместо этого доступна ошибка задачи.

### 2.8 RenderRequest и SessionConfig

`RenderRequest` содержит `SessionConfig`. Каноническая схема доступна через `GET /api/v2/schema/session`; используйте её как авторитетный контракт. Корневые поля конфигурации:

| Поле | Назначение |
|---|---|
| `schema_version` | Текущая версия конфигурации API, сейчас `1.0`. |
| `source_id` | Серверный идентификатор, возвращённый загрузкой источника. |
| `output_format` | Запрошенный опубликованный формат: WAV PCM 24-bit, FLAC lossless 24-bit или MP3 CBR 256 кбит/с. |
| `source_gain_db`, `output_gain_db` | Явные регуляторы усиления. |
| `binaural`, `binaural_automation`, `carrier_automation` | Управление строгой binaural и её автоматизацией во времени. |
| `monaural` | Настройки моноаурального/общемодового метода. |
| `isochronic`, `isochronic_automation` | Изохронный метод и его автоматизация. |
| `noise`, `noise_automation` | Настройки слоя шума/материала и его автоматизация. |
| `ambient` | Настройки ambient-слоя. |
| `mixer` | Управление относительным миксом и маршрутизацией. |
| `spatial` | Управление пространственной/mid-side обработкой музыки. |
| `safety` | Политика пиков/запаса и безопасности. |
| `multimodal` | Многометодная программа с учётом сцены и политика проверки. |
| `output_metadata` | Политика метаданных вывода. |
| `module_instances` | Зарезервировано для будущей модели модулей; непустые значения отклоняются. |

Точки автоматизации должны строго возрастать во времени и удовлетворять применимым ограничениям Nyquist и диапазона. Строгое binaural-соотношение фиксировано: левая несущая равна несущей минус половина beat rate; правая несущая равна несущей плюс половина beat rate. Прямой binaural-компонент смешивается после пространственной обработки.

### 2.9 Multimodal-конфигурация

`multimodal` активна только если `enabled=true` и включён хотя бы один вложенный метод. В ней есть следующие основные группы:

| Группа | Назначение |
|---|---|
| `playback_target` | `headphones`, `speakers` или `hybrid`. |
| `carrier_policy` | Минимальная, максимальная, предпочитаемая несущая, привязка к тональности, избегание свиста, maximum slew и ограничения удержания/планирования. |
| `section_program` | Упорядоченные непересекающиеся интервалы временной шкалы для значений скорости, несущей, глубины и паттерна. |
| `multiband` | Кроссоверы пяти полос и порядок фильтра. |
| `scene_analysis` | Настройки анализатора и защит, описанные выше. |
| `scene_planner` | Политика глобальной несущей и размещения по сцене. |
| `maximum_composite_delta_db` | Ограничение совокупной глубины вмешательства. |
| `fade_seconds` | Настройка fade на границах рендера. |
| `verification` | Состояние проверки и режим `aggregate` или `leave_one_out`. |

Доступные логические полосы: `sub`, `low`, `mid`, `presence`, `air` и `full`. Метод использует назначенную полосу; он не утверждает, что изменяет всё частотное содержимое только потому, что общая сцена допустима по всему полотну.

### 2.10 Параметры конкретных методов

| Метод | Основные управления сверх общего словаря |
|---|---|
| `binaural` | Несущая, beat/rate, direct mix и автоматизация при строгом отношении левой/правой несущей. |
| `monaural` | Общая целевая огибающая выбранной полосы и параметры сглаживания/формы, опубликованные схемой. |
| `isochronic` | Форма импульса, глубина огибающей и управление скоростью/автоматизацией. |
| `am` | Глубина модуляции, форма, скорость, выбранная полоса и усиление. |
| `fm` | Девиация частоты в герцах, скорость, выбранная полоса и усиление. |
| `pm` | Фазовый сдвиг в радианах, скорость, выбранная полоса и усиление. |
| `native_binaural` | Режим материала, качество analytic/Hilbert по числу taps, строгий сдвиг минус/плюс половины скорости, выбранная полоса и усиление. |
| `partial_twin` | Число гармоник от 1 до 8, attack/release огибающей, отношение несущих, выбранная полоса и усиление. |
| `push_pull_ild` | Величина ILD от 0 до 3 dB, перемещение с постоянной мощностью и управление общей медленной AM. |

### 2.11 Программы секций и изменяемая во времени схема

Программа секций использует абсолютное время композиции, а не относительный счётчик цикла. Интервалы должны быть упорядочены и не должны пересекаться. Большее число переходов можно равномерно распределить по всей работе, определив больше последовательных интервалов. Каждый интервал может независимо задавать целевую скорость, диапазон/значение несущей, глубину и выбор паттерна, при этом политика несущей по-прежнему запрещает недопустимое или слишком резкое движение несущей.

Это позволяет начать с одной скорости, а завершить другой без принудительного финального участка реориентации. Это также позволяет менять диапазоны несущей вместе с музыкальными секциями, чтобы слой оставался ниже, менее заметным или более гармонически совместимым с текущим материалом.

### 2.12 Контракт proof и качества

Объект proof может содержать aggregate comparison, результаты leave-one-out по каждому методу, scene guard, доказательства render-plan, routing assertion, проверку decoder/codec и метрики master-safety. Их следует интерпретировать так:

| Элемент proof | Корректная интерпретация |
|---|---|
| Aggregate | Полностью включённый weave отличается от идентично смастеренного выключенного weave; он не изолирует каждый метод. |
| Leave-one-out | Отключение одного включённого метода даёт доказательство причинного вклада этого отдельного метода. |
| Scene guard | Подтверждает учёт защит и нулевой остаток там, где применяется строгий обход; он не сертифицирует художественную прозрачность. |
| Проверка декодированного кодека | Подтверждает, что опубликованный вывод переживает декодирование и соответствует запрошенному техническому формату. |
| Хеш плана и манифест | Привязывают доказательства вывода к конкретной запланированной топологии и расписанию. |
| Метрики громкости/пика | Инженерные индикаторы безопасности, а не замена формальному соответствию вещанию, PEAQ, MUSHRA или ABX-тестам прослушивания. |

Для production-приёмки используйте leave-one-out проверку, когда важны доказательства по отдельному методу, слушайте на предполагаемой цели воспроизведения и архивируйте аудио вместе с QA-отчётом и манифестом.

### 2.13 Маршруты совместимости

Сервис также сохраняет маршруты совместимости под `/api/...` для существующих клиентов, включая устаревшие процессы render, preview, source, project/catalog, graph, migration, validation и compile. Новые интеграции должны использовать `/api/v2/...` и живые маршруты schema/capabilities.

### 2.14 Рекомендуемый сценарий клиента

1. Запросите `GET /api/v2/capabilities` и `GET /api/v2/schema/session`.
2. Загрузите один WAV, FLAC или MP3 через `POST /api/v2/sources`; сохраните возвращённые `source_id` и хеши.
3. Выполните `POST /api/v2/analyze`, когда нужно размещение с учётом сцены, затем просмотрите анализ источника, рабочую карту, сцену и результаты modulation-detection.
4. Выберите `headphones`, `speakers` или `hybrid`; не запрашивайте наушниковые методы для цели только колонок.
5. Настройте строгую защиту до natural/music-derived методов, когда вокал или транзиенты должны остаться нетронутыми.
6. Для инструментальной работы по всему полотну явно выберите соответствующую политику сцены; не делайте вывод о полной допустимости по одному глобальному флагу.
7. Отправьте `POST /api/v2/plan`, проверьте план и сохраните `plan_hash`.
8. Отправьте `POST /api/v2/render` или `POST /api/v2/preview`, затем опрашивайте статус задачи или переподключайтесь к SSE.
9. Дождитесь финального состояния; никогда не считайте `202` доказательством публикации аудио.
10. При `complete` архивируйте вместе output, QA-отчёт, manifest и proof. Используйте leave-one-out verification, когда это необходимо.
11. Используйте FLAC/WAV, когда нежелательны дополнительные потери кодека доставки; выбирайте MP3 только когда нужен проверенный deliverable CBR 256 кбит/с.

### 2.15 Контакты

Telegram: [@ctepx](https://t.me/ctepx)  
Email: [ctepx2001@gmail.com](mailto:ctepx2001@gmail.com)
