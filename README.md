# Distributed LLM Inference with Ray Serve — Ray Summit 2026

© 2026, Anyscale. All Rights Reserved

## Provenance

Ray Summit 2026 Anyscale course material, republished with permission. Original content
copyright 2026 Anyscale.

Serve high-throughput, low-latency inference on Ray Serve, from a classic ML model to a
distributed LLM. Each notebook is self-contained and defines every helper it needs, so you can
open any one of them and run it top to bottom.

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

### Tested configuration

Nothing in this repository was executed to produce this table. Every value below is the
configuration the repo *declares*, read out of the `image_uri` lines in the service YAMLs and the
pins in `requirements.txt`. Read it as the target environment, not as a matrix that anyone ran
here.

| Component | Version |
|---|---|
| Ray | `ray[serve]==2.56.0` |
| LLM image, for `code/llm/**` and notebooks 03 to 05 | `anyscale/ray-llm:2.56.0-py312-cu130` |
| CPU image, for `code/classic/**` and `code/ops/tracing/` | `anyscale/ray:2.56.0-slim-py312` |
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

## Also here

- **`code/`** holds deployable versions of the apps the notebooks build, grouped by concept under
  `llm/`, `classic/`, and `ops/`. Each concept directory is self-contained: run or deploy from inside
  it. See [code/README.md](code/README.md).
