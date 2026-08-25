# Distributed LLM Inference with Ray Serve — Ray Summit 2026

© 2026, Anyscale. All Rights Reserved

## Provenance

Ray Summit 2026 Anyscale course material, republished with permission. Original content
copyright 2026 Anyscale.

Serve high-throughput, low-latency inference on Ray Serve, from a classic ML model to a
distributed LLM. Each notebook is self-contained and defines every helper it needs, so you can
open any one of them without loading the others first. None of them ship executed, and none
were run here, so treat top-to-bottom execution as the intended design rather than an
observed result. See Committed notebook state below.

## Agenda

| # | Notebook | What you learn |
|---|----------|----------------|
| 01 | [Ray and Ray Serve fundamentals](01_ray_serve_fundamentals.ipynb) | Deployments, request routing and admission control, FastAPI ingress, composing several deployments |
| 02 | [Architecture, fault tolerance, and scaling](02_architecture_fault_tolerance_scaling.ipynb) | The control plane behind an app: health checks, self-healing, and how the autoscaling loop works |
| 03 | [Ray Serve LLM](03_ray_serve_llm_framework.ipynb) | An OpenAI-compatible endpoint with `ray.serve.llm`, and scaling at the framework and engine levels |
| 04 | [Inside the inference engine](04_inside_the_inference_engine.ipynb) | TTFT and ITL, the KV cache, paged attention, prefix caching, continuous batching, chunked prefill |
| 05 | [Scaling LLM inference](05_scaling_llm_inference.ipynb) | KV-cache-aware routing, prefill/decode disaggregation, mixture-of-experts at scale |

Notebooks 03, 04 and 05 all import `ray.serve.llm`, so each one needs the `anyscale/ray-llm`
image, which carries vLLM. The plain `anyscale/ray` image fails at the import. 03 and 04 run
on one GPU; 05 covers multi-GPU routing and disaggregation. Every example under `code/llm/`
needs that same image. Notebooks 01 and 02 run on CPU. [code/README.md](code/README.md)
describes what each example does and gives a GPU count where the example needs more than the
Tier-1 default of one to two GPUs. Runnable cells use a small model, while the multi-node production configs appear as
read-only blocks and as `code/**/service_prod*.yaml`.

Cost warning: the `service_prod.yaml` files for serving, disaggregation, and expert parallelism
request p5.48xlarge fleets (8x H100 per node, up to 24 nodes for disaggregation). They are
read-only references, not configs to deploy casually.

## Environment

### Declared configuration

Nothing in this repository was executed to produce this table. Every value below is the
configuration the repo *declares*, read out of the `image_uri` lines in the service YAMLs and the
pins in `requirements.txt`. Read it as the target environment, not as a matrix that anyone ran
here.

| Component | Version |
|---|---|
| Ray | `ray[serve]==2.56.0` |
| LLM image, for `code/llm/**` and notebooks 03 to 05 | `anyscale/ray-llm:2.56.0-py312-cu130` |
| CPU image, for `code/classic/**` other than `stream_processing/`, and for `code/ops/tracing/` | `anyscale/ray:2.56.0-slim-py312` |
| GPU image, for `code/classic/stream_processing/` | `anyscale/ray:2.56.0-py312-cu125` |
| Python | 3.12, read from the `py312` suffix that every image tag carries |
| CUDA | not written as a number anywhere in the repo. The tags end in `cu130` for `ray-llm` and `cu125` for the stream-processing image |
| FastAPI | `>=0.133.0,<0.140` |
| PyTorch | unpinned in `requirements.txt`. Pinned to `torch==2.13.0` in `code/classic/ranking/service.yaml` |
| Pydantic | unpinned in `requirements.txt`. Pinned to `pydantic==2.13.4` in `code/classic/ranking/service.yaml` |
| httpx, openai, numpy, requests, psutil, locust, boto3, diffusers | present in `requirements.txt`, all unpinned |

Where each value came from: `requirements.txt` for the Ray, FastAPI, and unpinned entries,
`code/classic/ranking/service.yaml` for the torch and pydantic pins, and the `image_uri` line of
each of the thirteen service YAMLs for the image tags. No lockfile is committed, so a fresh
install resolves the unpinned entries to whatever is current on the day you run it.

The FastAPI ceiling is the one pin with a recorded reason. `requirements.txt` states that from
FastAPI 0.140.0 the app object holds a `_thread.lock` that Ray 2.56.0's `serve.ingress` cannot
cloudpickle, so every `@serve.ingress` module fails at import, and that the boundary was found by
bisection at 0.139.0 working and 0.140.0 failing. That bisection was not reproduced for this
README.

## Committed notebook state

All five notebooks ship completely unexecuted. Every code cell in the repository has
`execution_count: null` and an empty `outputs` list, so no cell was run before commit and no
recorded output survives in the files. You get the code and the prose around it. You do not get
evidence that any of it ran.

| Notebook | Code cells | Cells with `execution_count` set | Cells with stored output |
|---|---|---|---|
| `01_ray_serve_fundamentals.ipynb` | 36 | 0 | 0 |
| `02_architecture_fault_tolerance_scaling.ipynb` | 24 | 0 | 0 |
| `03_ray_serve_llm_framework.ipynb` | 7 | 0 | 0 |
| `04_inside_the_inference_engine.ipynb` | 6 | 0 | 0 |
| `05_scaling_llm_inference.ipynb` | 3 | 0 | 0 |

Those counts come from parsing the five `.ipynb` files as JSON. Re-check them yourself, from the
repository root:

```bash
python - <<'PY'
import glob, json

for path in sorted(glob.glob("*.ipynb")):
    cells = json.load(open(path))["cells"]
    code = [c for c in cells if c["cell_type"] == "code"]
    ran = [c for c in code if c.get("execution_count") is not None]
    out = [c for c in code if c.get("outputs")]
    print(f"{path}: code={len(code)} executed={len(ran)} with_output={len(out)}")
PY
```

Two smaller facts from the same parse. All five files are nbformat 4.5, and all five point at the
kernel named `python3`, though notebook 02 gives that kernel the display name `base` while the
other four call it `Python 3`.

## Also here

- **`code/`** holds deployable versions of the apps the notebooks build, grouped by concept under
  `llm/`, `classic/`, and `ops/`. Each concept directory is self-contained: run or deploy from inside
  it. See [code/README.md](code/README.md).

## A note on scope

What this repository is, and what it is not, so you know what you are picking up.

**The notebooks are unexecuted.** See [Committed notebook state](#committed-notebook-state) above.
Reading them teaches the material. Running them needs the cluster described below, and nobody ran
them before they were committed.

**The `code/llm/` examples need the `ray-llm` image.** `ray.serve.llm` runs on vLLM, which the
plain `anyscale/ray` image does not carry, so the import fails there before a deployment ever
starts. The files that reach for the LLM stack are `serving/app.py`, `serving/multimodel.py`,
`composition/app.py`, `routing/app.py`, `routing/verify.py`, `disaggregation/app.py`,
`expert_parallel/app.py`, and `engine/reset_cache.py` under `code/llm/`, plus
`offline_inference/batch_infer.py`, which imports `ray.data.llm` rather than `ray.serve.llm`. The
two client scripts in that tree, `serving/query.py` and `engine/load_test.py`, only speak HTTP to
an endpoint, so they run anywhere.

**The `service_prod.yaml` files are reference material, not configs to deploy casually.** Each
asks for a fleet of `p5.48xlarge` workers, described in the files' own comments as 8x H100 per
node:

| Config | Worker nodes | Model it names | Parallelism it declares |
|---|---|---|---|
| `code/llm/serving/service_prod.yaml` | `p5.48xlarge`, 1 to 4 | `meta-llama/Llama-3.1-70B-Instruct` | tensor parallel 8 |
| `code/llm/disaggregation/service_prod.yaml` | `p5.48xlarge`, 6 to 24 | `moonshotai/Kimi-K2-Instruct` | pipeline parallel 2, tensor parallel 8, KV moved over NIXL |
| `code/llm/expert_parallel/service_prod.yaml` | `p5.48xlarge`, 16 fixed | `deepseek-ai/DeepSeek-V3` | data parallel 16, tensor parallel 8, expert parallel on |

Read those numbers before you run `anyscale service deploy` on any of them. The disaggregation
config starts at six nodes rather than scaling up from one. The expert-parallel config sets
`min_nodes` equal to `max_nodes` at 16, so it never scales down. None of the three was deployed to
produce this README, and the repository records no cost figure for any of them.

**`code/llm/engine/reset_cache.py` depends on private APIs.** It imports `build_dev_openai_app`
from `ray.llm._internal.serve.core.ingress.dev_ingress` and `broadcast` from
`ray.llm._internal.serve.utils.broadcast`. The file's own header says those carry no
compatibility guarantee and that their behavior is pinned to Ray 2.56.0, so treat the script as
tied to that one release. Nothing else in the repository reaches into `ray.llm._internal`.

**There is no test suite, no CI configuration, and no lockfile.** The tracked files are the five
notebooks, the `code/` tree, this README and `code/README.md`, `requirements.txt`, `.gitignore`,
and `.anyscaleignore`. Nothing in the repository verifies itself, so every claim in it, including
the ones on this page, rests on reading the source.
